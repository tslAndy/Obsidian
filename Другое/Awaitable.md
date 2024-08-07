#other 
До версии Unity 2023.1 в Unity было несколько вариантов асинхронного кода, но у каждого были свои недостатки. У нас всегда были сопрограммы, которые можно было использовать для асинхронного кодирования, но у них были свои ограничения, такие как привязка к `Monobehaviour`, который запускает сопрограмму, и их неспособность возвращать результаты напрямую.

У нас также была возможность использовать `Task`, но поскольку он управляется из среды .NET, при использовании в Unity возникали свои проблемы, например, необходимость вручную контролировать его жизненный цикл.

Начиная с версии Unity 2023.1, у нас есть класс `Awaitable` из Unity, который наконец-то дает хорошее решение для использования асинхронного кода.

## Что это
`Awaitable` — это механизм, предоставляемый Unity для помощи в реализации асинхронных операций. Асинхронность не обязательно означает параллельность. У нас может быть часть нашего кода, которая будет ждать выполнения другой части нашего кода, прежде чем продолжить, но не будет блокировать никакой код, который выполняется в этот момент.

Например, без асинхронного кода цикл while, который повторяется до тех пор, пока метод не вернет true внутри одного из наших методов Update, будет выполнять код другого метода до тех пор, пока он не вернет true, а затем продолжит выполнение остальной части кода после истечения времени while, но также заблокирует выполнение всех других методов Update, которые у нас есть.

Чтобы избежать этого, можно использовать сопрограмму, но, как уже упоминалось, она имеет свои ограничения.

Класс Awaitable может решить эту проблему, но также может использоваться для выполнения кода в фоновом потоке, если у нас есть дорогостоящие операции, которые мы не хотим влиять на производительность основного потока.

## Простое использование
Статические методы `EndOfFrameAsync`, `FixedUpdateAsync`, `NextFrameAsync`и `WaitForSecondsAsync` класса Awaitable позволяют нам выполнить фрагмент кода, а затем передать выполнение остальной части нашей программы на заранее определенное время. Вот пример их использования:

```cs
public class AwaitableExample : MonoBehaviour
{
   private void Start()
   {
      for (int i = 0; i < 10; i++)
      {
         Debug.Log($"i Equals to: {i}, on frame: {Time.frameCount}");
      }
      
      Debug.Log("Outside the for loop");
   }

   private void Update() => Debug.Log($"Inside the Update, frame: {Time.frameCount}");
}
```

Результат будет следующим:
```
i Equals to: 0, on frame: 1
i Equals to: 1, on frame: 1
.
.
.
i Equals to: 9, on frame: 1
Outside the for loop
Inside the Update, frame: 1
Inside the Update, frame: 2
.
.
.

```

Цикл for будет выполнен до любого другого кода, а затем начнет выполняться код в Update. Это может быть проблемой для нас. Мы хотим, чтобы код в цикле for выполнялся, но мы не хотим, чтобы наша игра ждала завершения кода, вместо этого мы хотели бы, чтобы код выполнялся во время выполнения нашего Update, чтобы наша игра не зависала каждый раз, когда создается новый игровой объект. С помощью Awaitable мы можем сделать это:

```cs
public class AwaitableExample : MonoBehaviour
{
   private async void Start()
   {
      for (int i = 0; i < 10; i++)
      {
         Debug.Log($"i Equals to: {i}, on frame: {Time.frameCount}");
         await UnityEngine.Awaitable.EndOfFrameAsync();
      }
      
      Debug.Log("Outside the for loop");
   }

   private void Update() => Debug.Log($"Inside the Update, frame: {Time.frameCount}");
}

```

Теперь вывод будет таким:

```cs
i Equals to: 0, on frame: 1
Inside the Update, frame: 1
i Equals to: 1, on frame: 1
Inside the Update, frame: 2
i Equals to: 2, on frame: 2
.
.
.
Inside the Update, frame: 9
i Equals to: 9, on frame: 9
Inside the Update, frame: 10
Outside the for loop
Inside the Update, frame: 11
Inside the Update, frame: 12
```

Обратите внимание, что в первом кадре у нас будет два выполнения цикла for. Это происходит, потому что первая операция выполняется в то время, когда `Start`должен запуститься метод, а после этого выполнение возобновляется в конце каждого кадра из-за метода `EndOfFrameAsync`. Если бы вместо этого мы использовали , `NextFrameAsync`вывод был бы следующим:

```cs
i Equals to: 0, on frame: 1
Inside the Update, frame: 1
Inside the Update, frame: 2
i Equals to: 1, on frame: 2
Inside the Update, frame: 3
i Equals to: 2, on frame: 3
.
.
.
Inside the Update, frame: 10
i Equals to: 9, on frame: 10
Inside the Update, frame: 11
Outside the for loop
Inside the Update, frame: 12
```

И с `FixedUpdateAsync`выходом может быть:
```cs
i Equals to: 0, on frame: 1
i Equals to: 1, on frame: 1
Inside the Update, frame: 1
i Equals to: 2, on frame: 2
Inside the Update, frame: 2
i Equals to: 3, on frame: 3
i Equals to: 4, on frame: 3
i Equals to: 5, on frame: 3
i Equals to: 6, on frame: 3
i Equals to: 7, on frame: 3
Inside the Update, frame: 3
```

«может быть», потому что это зависит от того, как часто выполняется FixedUpdate по сравнению с тем, как часто выполняется Update. С помощью `FixedUpdateAsync`мы откладываем выполнение до следующего FixedUpdate

Все эти методы могут принимать необязательный аргумент, cancellationToken, который используется для отмены выполнения, например:

```cs
public class AwaitableExample : MonoBehaviour
{
   private CancellationTokenSource cancellationTokenSource = new();
   
   private async void Start()
   {
      while(!cancellationTokenSource.IsCancellationRequested)
      {
         Debug.Log($"Inside the while loop");
         
         try
         {
            await UnityEngine.Awaitable.EndOfFrameAsync(cancellationTokenSource.Token);
         }
         catch (Exception e)
         {
            Debug.Log(e);
            Debug.Log("cancellation requested");
         }
      }
      
      Debug.Log("operation ended");
   }
   

   private void Update()
   {
      Debug.Log("Inside Update");
      if(Keyboard.current.spaceKey.wasPressedThisFrame)
         cancellationTokenSource.Cancel();
   }
}

```

## Выполнение кода в фоновом потоке
Мы можем использовать `Awaitable`для выполнения кода, требующего высокой производительности, в фоновом потоке, используя методы `BackgroundThreadAsync`и `MainThreadAsync`. Мы должны позаботиться о том, чтобы этот код не использовал API Unity, поскольку методы Unity должны выполняться в основном потоке, но любые вычисления, требующие высокой производительности и не использующие API Unity, могут быть выполнены. Вот пример:

```cs
public class AwaitableExample : MonoBehaviour
{
   private async void Start()
   {
      var result = await ExecutesInTheBackground(1_000_000);
      Debug.Log($"Found the result in frame: {Time.frameCount}, the result is: {result}");
   }

   private async Awaitable<BigInteger> ExecutesInTheBackground(int n)
   {
      BigInteger a = 0;
      BigInteger b = 1;
      BigInteger c = 0;

      if (n == 0)
         return a;
      
      await UnityEngine.Awaitable.BackgroundThreadAsync();

      for (int i = 2; i <=n; i++)
      {
         c = a + b;
         a = b;
         b = c;
      }
      
      await UnityEngine.Awaitable.MainThreadAsync();

      return b;
   }

   private void Update() => Debug.Log("Inside the Update Method");
}

```

Этот метод находит 1000000-е число Фибоначчи, в то время как методы Update продолжают выполняться, а вычисления производятся в фоновом потоке, так что основной поток не мешает вычислениям.

## Использование FromAsyncOperation
Статический метод `FromAsyncOperation` возвращает `Awaitable`существующего `AsyncOperation` объекта.
```cs
public class AwaitableExample : MonoBehaviour
{
   [SerializeField] private ToSpawn objectToSpawn;
   private ToSpawn[] _spawnedObjects;
   private AsyncInstantiateOperation<ToSpawn> result;
   
   private async void Start()
   {
      var spawn = UnityEngine.Awaitable.FromAsyncOperation(AsyncInstantiation());
      await spawn;
      Debug.Log("At the end of the Start method");
   }

   private AsyncInstantiateOperation AsyncInstantiation()
   {
      Span<Vector3> positions = stackalloc Vector3[3];
      Span<Quaternion> rotations = stackalloc Quaternion[3];

      positions[0] = new Vector2(1, 1);
      positions[1] = new Vector2(2, 1);
      positions[2] = new Vector2(1, 3);
      
      rotations.Fill(Quaternion.identity);
      
      result = InstantiateAsync(objectToSpawn, 3, positions, rotations);
      result.completed += Message;
      return result;
   }

   private void Update() => Debug.Log("Inside the Update Method");
   
   private void Message(AsyncOperation _) => Debug.Log("All objects Instantiated! ");
}
```

## Awaitable - в пуле
В `Awaitables`Unity это объекты в пуле, так что они не аллоцируют много памяти. Это ссылочные типы, так что на них можно ссылаться из разных потоков, но поскольку они в пуле, нам нужно быть осторожными.

После использования они возвращаются в пул, и это означает, что мы не можем использовать их несколько раз. В пуле означает, что что-то вроде этого:
```cs
while(true)
{
    await Awaitable.NextFrameAsync();
}
```

не будет выделять память для каждого кадра, но пул также означает, что что-то вроде следующего невозможно и имеет неопределенное поведение:

```cs
public class AwaitableExample : MonoBehaviour
{
   private async void Start()
   {
      var myFoo = Foo1();

      int firstAwait = await myFoo;
      
      Debug.Log($"After first Await, firstAwait value {firstAwait}");

      int secondAwait = await myFoo;
      
      Debug.Log($"After second Await, secondAwait value: {secondAwait}");
   }

   private async Awaitable<int> Foo1()
   {
      Debug.Log("End of Foo1"); 
      await UnityEngine.Awaitable.EndOfFrameAsync();
      return 1;
   }
}
```

После того, как myFoo был использовать с await в первый раз и возвращен в пул, он не может быть использован во второй раз, поэтому мы получим null reference exception.

## AwaitableCompletionSource
Позволяет контроллировать завершение Awaitable
```cs
public class AwaitableExample : MonoBehaviour
{
   private AwaitableCompletionSource<int> bar = new();
   
   private async void Start()
   {
      Debug.Log("Beginning of Start");

      int result = await Keypress();

      Debug.Log($"Result at frame {Time.frameCount} with value: {result}");
   }

   private void Update()
   {
      if (Keyboard.current.aKey.wasPressedThisFrame)
      {
         bar?.SetResult(1);
         bar = null;
      }

      if (Keyboard.current.sKey.wasPressedThisFrame)
      {
         bar?.SetResult(2);
         bar = null;
      }
   }

   private async Awaitable<int> Keypress()
   {
      int result = await bar.Awaitable;
      return result;
   }
}
```

Awaitable здесь не завершится, пока мы не установим результат с помощью `AwaitableCompletionSource.SetResult`. После этого мы можем вернуть результат обычным образом.

Здесь важно присвоение null объекту `bar`после установки результата. Как и прежде, Awaitables Unity в пуле, попытка установить результат во второй раз приведет к ошибке.

## Создание WaitUntil с помощью Awaitable
```cs
private static async UnityEngine.Awaitable WaitUntil(Func<bool> condition, CancellationToken cancellationToken = default)
{
    while(!condition())
        await UnityEngine.Awaitable.EndOfFrameAsync(cancellationToken);
}
```