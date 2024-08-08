## Delayed result gathering
Примером является карта экспозиции и избегание экспозиции. У нас есть сетка, которая показывает, подвергается ли каждая ячейка воздействию «стража», и у нас есть персонаж, который пытается переместиться и спрятаться в близлежащей ячейке, которая не подвергается воздействию. Когда страж перемещается, персонаж убегает, как только его текущая ячейка подвергается воздействию.

![[exposure-avoidance-compressed.mp4]]

Вот шаги в цикле обновления этой механики:
1. Обновить карту экспозиции на основе текущего положения дозорного.  
    Пропустить луч от дозорного к каждой ячейке. Если луч чистый, ячейка экспонирована.
2. Вычислите длину пути от персонажа до каждой ячейки сетки.
3. Если текущая ячейка персонажа открыта, обновите его пункт назначения на соседнюю ячейку, которая не открыта, и переместите персонажа к пункту назначения.

```cs
void Update()
{
  UpdateExposureMap();
  UpdatePathLengths();
  UpdateDestination();
}
```

```cs
void UpdateExposureMap()
{
  for (int iCell = 0; iCell < NumCells; ++iCell)
  {
    Vector3 cellCenter = Grid[iCell];
    Vector3 vec = cellCenter - EyePos;
    Vector3 rayDir = vec.normalized;
    float rayLen = vec.magnitude;
 
    bool exposed = !Physics.Raycast(EyePos, rayDir, rayLen);
    ExposureMap[iCell] = exposed;
  }
}
```

Для приведенной выше реализации профайлер показывает следующее для 10 тыс. лучей на кадр:

![[single-threaded-raycasts.png]]

При дальнейшем увеличении мы видим, что широкий оранжевый прямоугольник на самом деле представляет собой множество крошечных секций raycast, по одной на каждый raycast.

![[single-threaded-raycasts-enlarged.png]]

Возникает немедленная проблема: raycasts в больших количествах не совсем дешевы. Вызовы этих функций и ожидание их последовательного возврата довольно сильно тормозят основной поток. Оптимизация в порядке.

## Многопоточность
Серия блокирующих вызовов функций raycast в основном потоке — это очень плохо. Raycasts относятся к семейству распараллеливаемых операций. Каждый raycast делает одно и то же: он запрашивает пространственные данные и находит, где луч сталкивается со столкновением, и эта операция может быть самодостаточной в отдельном потоке. Вместо того чтобы заполнять основной поток последовательными вызовами функций raycast, первое, что нужно попробовать, — это переложить работу на другие ядра ЦП, т. е. воспользоваться преимуществами многопоточности. Если у нас 10 рабочих ядер, то в качестве первого прохода мы можем попробовать разделить работу raycast на 10 потоков.

Мы определяем две структуры заданий: `RaycastSetupJob` и `RaycastGatherJob`. Задание настройки устанавливает команды raycast перед заданием raycast, которое выполняет фактическую работу raycast (вызванную вызовом Unity `RaycastCommand.ScheduleBatch`). Задание сбора собирает результаты raycast и использует их для обновления карты экспозиции. Вот цепочка зависимостей: задание raycast зависит от задания настройки, а задание gahter зависит от задания raycast.

```cs
struct RaycastSetupJob : IJobParallelFor
{
  public Vector3 EyePos;
 
  [ReadOnly]
  public NativeArray<Vector3> Grid;
 
  [WriteOnly]
  public NativeArray<RaycastCommand> Commands;
 
  public void Execute(int index)
  {
    Vector3 cellCenter = Grid[index];
    Vector3 vec = cellCenter - EyePos;
    Commands[index] = 
      new RaycastCommand(EyePos, vec.normalized, vec.magnitude);
  }
}
 
struct RaycastGatherJob : IJobParallelFor
{
  [ReadOnly]
  public NativeArray<RaycastHit> Results;
 
  [WriteOnly]
  public NativeArray<bool> ExposureMap;
 
  public void Execute(int index)
  {
    bool exposed = (Results[index].distance <= 0.0f);
    ExposureMap[index] = exposed;
  }
}
```

`UpdateExposureMap` обретает следующий вид:
```cs
void UpdateExposureMap()
{
  // allocate data shared across jobs
  var allocator = Allocator.TempJob;
  var commands = 
    new NativeArray<RaycastCommand>(NumCells, allocator);
  var results = 
    new NativeArray<RaycastHit>(NumCells, allocator);
 
  // create setup job
  var setupJob = new RaycastSetupJob();
  setupJob.EyePos = EyePos;
  setupJob.Grid = Grid;
  setupJob.Commands = commands;
 
  // create gather job
  var gatherJob = new RaycastGatherJob();
  gatherJob.Results = results;
  gatherJob.ExposureMap = ExposureMap;
 
  // schedule setup job
  var hSetupJob = setupJob.Schedule(NumCells, JobBatchSize);
 
  // schedule raycast job
  // specify dependency on setup job
  var hRaycastJob = 
    RaycastCommand.ScheduleBatch
    (
      commands, 
      results, 
      JobBatchSize, 
      hSetupJob
    );
 
  // schedule gather job
  // specify dependency on raycast job
  var hGatherJob = 
    gatherJob.Schedule(NumCells, JobBatchSize, hRaycastJob);
 
  // kick jobs
  JobHandle.ScheduleBatchedJobs();
 
  // wait for completion
  hGatherJob.Complete();
 
  // dispose of job data
  commands.Dispose();
  results.Dispose();
}
```

Вот что показывает профайлер:
![[multi-threaded-raycats-same-frame-gather.png]]
Время ЦП, затрачиваемое на шаг raycast в основном потоке, сокращается. Однако основной поток все еще останавливается, ожидая завершения многопоточного задания raycast в рабочих потоках. Можем ли мы улучшить это еще больше?

## Отсроченный сбор результатов
Если система может позволить себе задержку в один кадр, то почему бы не выключить работу и не собрать результаты в следующем кадре? Таким образом, после выключения работы ядро ​​ЦП может быть немедленно освобождено для работы над чем-то другим.

Обратите внимание, что если результаты нужны в том же кадре, все равно стоит попытаться запустить работу пораньше, выполнить какую-то другую работу и собрать результаты позже в том же кадре. Суть в том, чтобы избежать остановки основного потока в ожидании завершения работы.

В следующей итерации нашего примера я собираюсь заставить основной поток выкинуть задание raycast в одном кадре и отложить сбор результатов raycast до следующего кадра. Теперь фактически не осталось ничего, что могло бы занять процессорное время в основном потоке, кроме сбора результатов raycast из предыдущего кадра и запустить задание, которое настраивает задание raycast для следующего кадра.

Задание по сбору данных теперь ожидает завершения в начале функции `UpdateExposureMap`. Карта экспозиции теперь имеет двойную буферизацию, поэтому задания могут работать с задним буфером, в то время как остальная часть кода может обращаться к переднему буферу, чтобы избежать условий гонки. Кроме того, дескриптор задания по сбору данных и общие данные между заданиями теперь продвигаются из локальных переменных в поля, поскольку их время жизни должно охватывать два кадра.

```cs
void UpdateExposureMap()
{
  // wait for jobs from last frame to complete
  hGatherJob.Complete();
 
  // double-buffering
  SwapExposureBackBuffer();
 
  // dispose of job data allocated from last frame
  if (Commands.IsCreated)
    Commands.Dispose();
  if (Results.IsCreated)
    Results.Dispose();
 
  // allocate data shared across jobs
  var allocator = Allocator.TempJob;
  Commands =
    new NativeArray<RaycastCommand>(NumCells, allocator);
  Results =
    new NativeArray<RaycastHit>(NumCells, allocator);
 
  // create setup job
  var setupJob = new RaycastSetupJob();
  setupJob.EyePos = EyePos;
  setupJob.Grid = Grid;
  setupJob.Commands = Commands;
 
  // create gather job
  var gatherJob = new RaycastGatherJob();
  gatherJob.Results = Results;
  gatherJob.ExposureMap = ExposureMap;
 
  // schedule setup job
  var hSetupJob = setupJob.Schedule(NumCells, JobBatchSize);
 
  // schedule raycast job
  // specify dependency on setup job
  var hRaycastJob = 
    RaycastCommand.ScheduleBatch
    (
      Commands, 
      Results, 
      JobBatchSize, 
      hSetupJob
    );
 
  // schedule gather job
  // specify dependency on raycast job
  hGatherJob = 
    gatherJob.Schedule(NumCells, JobBatchSize, hRaycastJob);
 
  // kick jobs
  JobHandle.ScheduleBatchedJobs();
}
```

И вот что показывает профайлер сейчас:
![[multi-threaded-raycasts-next-frame-gather.png]]
Задание raycast выполняется на рабочих потоках за пределами диапазона `UpdateExposureMap`. Результаты не нужны до вызова `UpdateExposureMap` в следующем кадре. Больше не нужно останавливать основной поток!

## Time slicing
Что делать, если работа занимает слишком много времени, чтобы уместиться в один кадр? Или что делать, если работа занимает больше времени, чем нам бы хотелось? Тогда мы можем разделить работу на несколько кадров. Это основная идея временного среза, еще одного метода оптимизации, который я подхватил на работе и который является одним из моих давних фаворитов.

Если подумать, то нарезка времени повсюду. Потоковая передача текстур, бесшовная загрузка и т. д. — работа, которая происходит «в фоновом режиме», не снижая частоту кадров игры. Не можете сделать все это в одном кадре? Тогда сделайте это в нескольких! Это простая, но очень эффективная идея.

Теперь давайте посмотрим, как мы можем еще больше сократить время ЦП, затрачиваемое на работу raycast с помощью временного среза. Вот шаблон, который я обычно использую для временного среза:

1. Объявите индекс (или итератор), который будет отслеживать текущий ход выполнения временного среза.
2. Инициализируйте индекс в начале нового пакета работы.
3. Повышайте индекс по мере выполнения дополнительной части работы в каждом временном интервале.
4. Выполнить логику окончательной обработки после завершения последней части работы.
5. Вернитесь к шагу 2, чтобы начать новый этап работы.

Давайте изменим наши структуры заданий так, чтобы они обрабатывали только часть карты экспозиции в одном кадре. Вот изменения:
- Добавьте поле `TimeSliceBaseIndex` (инициализированное значением 0) для отслеживания хода временного среза.
- Добавьте поле индекса в структуру задания, чтобы обозначить начало диапазона ячеек карты экспозиции для обработки в каждом кадре.
- Измените атрибут `WriteOnly` в поле `ExposureMap` структуры `RaycastGatherJob`на `NativeDisableParallelForRestriction`чтобы удалить защиту Unity, которая ограничивает доступ к нативным массивам только индексом, переданным в функции заданий `Execute`.

```cs
struct RaycastSetupJob : IJobParallelFor
{
  public Vector3 EyePos;
 
  [ReadOnly]
  public NativeArray<Vector3> Grid;
 
  [ReadOnly]
  public NativeArray<RaycastCommand> Commands;
 
  public int TimeSliceBaseIndex;
 
  public void Execute(int localIndex)
  {
    int globalIndex = localIndex + TimeSliceBaseIndex;
    Vector3 cellCenter = Grid[globalIndex];
    Vector3 vec = cellCenter - EyePos;
    Commands[localIndex] = 
      new RaycastCommand(EyePos, vec.normalized, vec.magnitude);
  }
}
 
struct RaycastGatherJob : IJobParallelFor
{
  [ReadOnly]
  public NativeArray<RaycastHit> Results;
 
  [NativeDisableParallelForRestriction]
  public NativeArray<bool> ExposureMap;
 
  public int TimeSliceBaseIndex;
 
  public void Execute(int localIndex)
  {
    int globalIndex = localIndex + TimeSliceBaseIndex;
    bool exposed = (Results[localIndex].distance <= 0.0f);
    ExposureMap[globalIndex] = exposed;
  }
}
```

```cs
void UpdateExposureMap(int numRaysPerTimeSlice)
{
  // wait for jobs from last frame to complete
  hGatherJob.Complete();
 
  // trim excess ray count for the last time slice of batch
  int numExcessRays = 
    TimeSliceBaseIndex + numRaysPerTimeSlice - NumCells;
  numRaysPerTimeSlice -= Mathf.Max(0, numExcessRays);
 
  // batch ended?
  if (TimeSliceBaseIndex < 0)
  {
    // double-buffering
    SwapExposureBackBuffer();
 
    // reset time slicing index
    TimeSliceBaseIndex = 0;
  }
 
  // dispose of job data allocated from last frame
  if (Commands.IsCreated)
    Commands.Dispose();
  if (Results.IsCreated)
    Results.Dispose();
 
  // allocate data shared across jobs
  var allocator = Allocator.TempJob;
  Commands =
    new NativeArray<RaycastCommand>
    (
      numRaysPerTimeSlice, 
      allocator
    );
  Results =
    new NativeArray<RaycastHit>
    (
      numRaysPerTimeSlice, 
      allocator
    );
   
  // create setup job
  var setupJob = new RaycastSetupJob();
  setupJob.EyePos = EyePos;
  setupJob.Grid = Grid;
  setupJob.Commands = Commands;
  setupJob.TimeSliceBaseIndex = TimeSliceBaseIndex;
   
  // create gather job
  var gatherJob = new RaycastGatherJob();
  gatherJob.Results = Results;
  gatherJob.ExposureMap = ExposureMap;
  gatherJob.TimeSliceBaseIndex = TimeSliceBaseIndex;
   
  // schedule setup job
  var hSetupJob = 
    setupJob.Schedule
    (
      numRaysPerTimeSlice, 
      JobBatchSize
    );
   
  // schedule raycast job
  // specify dependency on setup job
  var hRaycastJob = 
    RaycastCommand.ScheduleBatch
    (
      Commands, 
      Results, 
      JobBatchSize, 
      hSetupJob
    );
   
  // schedule gather job
  // specify dependency on raycast job
  hGatherJob = 
    gatherJob.Schedule
    (
      numRaysPerTimeSlice, 
      JobBatchSize, 
      hRaycastJob
    );
 
  // advance time slice index
  TimeSliceBaseIndex += numRaysPerTimeSlice;
 
  // end of batch?
  if (TimeSliceBaseIndex >= NumCells)
  {
    // signal end of batch
    TimeSliceBaseIndex = -1;
  }
 
  // kick jobs
  JobHandle.ScheduleBatchedJobs();
}
```

Если мы установим количество лучей на временной срез равным 10% от общего количества лучей на пакет, то для завершения всех raycasts для пакета и запуска замены буфера потребуется 10 кадров. На видео ниже вы можете видеть, как буфер карты фронтальной экспозиции обновляется каждый раз, когда все raycasts для пакета завершаются. В этом видео используется более низкая частота кадров, чтобы отладочная отрисовка длилась дольше на кадр, поэтому при полной частоте кадров задержка обновления карты экспозиции не будет такой большой.

![[exposure-avoidance-compressed.mp4]]
Если можно гарантировать, что не возникнет никаких условий гонки при использовании собранных результатов raycast для обновления карты экспозиции, можно отказаться от двойной буферизации и напрямую обновлять один буфер карты экспозиции по мере поступления результатов raycast. Конечным результатом является то, что обновленная часть карты экспозиции отображается в каждом кадре, а общая система становится более отзывчивой.