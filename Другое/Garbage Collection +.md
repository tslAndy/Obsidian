#other 
В Unity используется по умолчанию используется инкрементарный сборщик мусора, который разбивает очистку между несколькими кадрами. При его отключении используется сборщик типа `stop-the-world`, который останавливает выполнение программы до завершения очистки. Слишком частые изменения объектов (в каждом кадре) приводят к переключению GC в неинкрементальный режим, вместо того, чтобы распределять запуски GC по нескольким кадрам, мы получаем одну большую остановку приложения до завершения сборки мусора. Так же из-за отсутсвия уплотнения повышена вероятность фрагментации.

## Строки
Например, следующий код будет каждый кадр выделять память в куче для объекта `enemyString`. При этом для `"There are"`, `enemiesLeft.ToString()` и `"enemies"` так же будут создаваться объекты в куче. Т. е. будет 4 выделения памяти каждый кадр

```cs

private void Update()
{
    string enemyString = 
    "There are " + enemiesLeft.ToString() + " enemies";
    
    enemiesText.text = enemyString;
}

```

При этом компилятор оптимизирует конкатенцию, иначе бы код выше генерировал пять строк, т. е. сначал для `"There are " + enemiesLeft.ToString()`и потом ` + "enemies"`. Однако компилятор может скомбинировать до четырех многократно конкатенированных строк в один вызов метода `String.Concat()`, который выполняет конкатенацию на месте.

>“You can also use your language’s string concatenation operator … compilers translate the concatenation operator into a call to one of the overloads of `String.Concat`”

Если нужно скомбинировать больше четырех строк, компилятору придется несколько раз вызвать `String.Concat`. Плюс, это не сработает для циклов которые инкрементарно строят строку из меньших строк. 

```cs
public string GetPositionString(float[] positions)
{
    string positionString = "Positions are: " + positions[0];

    for(int i = 1; i < positions.Length; ++i)
    {
        positionString += ", " + positions[i].ToString();
    }

    return positionString;
}
```

Такой код выделяет много памяти, так как каждая итерация создает объекты для каждой части строки и для результата.

Один из способов решения проблемы - обновлять строки только при изменении значения:
```cs
public void ChangeEnemyCount(int change)
{
    enemiesLeft += change;

    string enemyString = "There are " + enemiesLeft.ToString() + " enemies left.";
    enemiesText.text = enemyString;
}
```

В случае, если строка увеличивается инкрементарно, то стоит использовать String Builder, он лучше всего работает когда больше 10 строк:

```cs
public string GetPositionString(float[] positions)
{
    var sb = new System.Text.StringBuilder();

    for(int i = 1; i < positions.Length; ++i)
    {
        sb.Append(", " + positions[i].ToString());
    }

    return sb.ToString();
}
```

Строки иммутабельны, т. е. модифицированная строка по факту выделяет новое место в куче. С другой стороны, `StringBuilder` резервирует буфер в памяти и изменяет его содержимое на месте. Резервирование большого объема памяти связано с накладными расходами, но без `StringBuilder` на каждой итерации в куче выделялась бы новая строка размером со всю построенную на данном этапе строку - общее использование памяти со `StringBuilder` значительно снижается для больших циклов.

При необходимости в есть метод `AppendFormat()`, позволяющий заносить форматированный текст. Так же необязательно аллоцировать `StringBuilder` на каждом кадре, можно его очищать и использовать заново.

Еще один вариант, это разбить текст на две части, т. е. на ту, которая меняется (число) и ту, которая не меняется.

Следующий пример будет генерировать много мусора:

```cs
int day = 27;
int month = 04;
int year = 2024;
Debug.Log(string.Format("Date: {0}/{1}/{2}", day, month, year));

// так как сигнатура метода Format это
string Format(string format, object arg0, object arg1, object arg2);
// Т. е. будет выделять память для объектов 
```

## Инстанциирование и удаление объектов
При постоянном создании и удалении объектов может возникнуть фрагментация памяти, связанная с тем, что в памяти могут образоваться пробелы, размера которых будут недостаточными для нового объекта. Сборщик мусора в Unity не сжимает память. Один из вариантов решения - использовать пул объектов. Например, инстанциируем некоторое количество при создании сцены, а удаляем все только когда сцена меняется. 

## Поиск компонентов и объектов
При использовании `GetComponent` создается список компонентов объекта, после чего этот список нужно удалять. Лучше получить компонент во время `Awake` и сохранить его в приватном поле. 

## Методы, вызвращающие массивы
Если метод при каждом вызове будет создавать новый массив, это скажется на сборке мусора:

```cs

float[] RandomList(int numElements) {
	var result = new float[numElements];
	
	for (int i = 0; i < numElements; i++) {
		result[i] = Random.value;
	}
	
	return result;
}
```

В примере ниже используем уже созданный массив, что позволяет избежать лишних аллокаций

```cs
void RandomList(float[] arrayToFill) {
	for (int i = 0; i < arrayToFill.Length; i++) {
		arrayToFill[i] = Random.value;
	}
}
```

## Переиспользование коллекций
Коллекции из System.Collection содержат метод Clear, который убирает содержимое, но не освобождает аллоцированную память. 

```cs
List<float> m_NearestNeighbors = new List<float>();

void Update() {

    m_NearestNeighbors.Clear();

    findDistancesToNearestNeighbors(NearestNeighbors);

    m_NearestNeighbors.Sort();

    // … use the sorted list somehow …
}
```

## Замыкания и анонимные методы
Желательно избегать замыканий. Минимизировать использование анонимных методов и ссылок на методы в критически важных для производительности метах. 

Ссылки на метод - референсные тип, аллоцируются на хипе, т. е. если передавать ссылку на метод как аргумент, можно создать много временных аллокаций. Это происходит независимо от того, какой метод передается, анонимный или уже определенный.

Если конвертировать анонимный метод в замыкание, количество памяти нужное для передачи замыкания сильно увеличивается. 

```cs
// Good C# script example: using an anonymous method 
// to sort a list. 
// This sorting method doesn’t create garbage
List<float> listOfNumbers = getListOfRandomNumbers();

listOfNumbers.Sort( (x, y) =>

(int)x.CompareTo((int)(y/2)) 

);
```

Анонимному методу нужен доступ к переменной, находящейся вне его области, и метод становится замыканием. Переменная `desiredDivisor` должна быть передана в замыкание. Чтобы гарантировать передачу в замыкание правильных значений, C# генерирует анонимный класс, который может сохранять внешние переменные, необходимые замыканию. Копия этого класса создается, когда замыкание передается методу `Sort`, и копия инициализируется значением целого числа `desiredDivisor`.

```cs
// Bad C# script example: the anonymous method 
// has become a closure,
// and now allocates memory to store 
// the value of desiredDivisor
// every time it is called.

List<float> listOfNumbers = getListOfRandomNumbers();

int desiredDivisor = getDesiredDivisor();
listOfNumbers.Sort( (x, y) =>

(int)x.CompareTo((int)(y/desiredDivisor))

);
```

Более подробно тут: [[Closures, Delegates and CG +]]
## Боксинг
Происходит, когда тип значений конвертируется в ссылочный тип, например при передаче типов значений методам принимающим ссылочный тип. Аллокатор в движке использует разные пулы для больших и маленьких аллокаций, но при этом в нем нет поколений, поэтому он не может эффективно вычищать небольшие, частые временные выделения памяти как при боксинге. 

## API Unity с массивами значений
 Все API Unity, которые возвращают массивы, создают новую копию массива каждый раз, когда к ним обращаются. Если ваш код обращается к API Unity со значениями массива чаще, чем необходимо, это, скорее всего, окажет пагубное влияние на производительность.
 
В качестве примера, следующий код без необходимости создает четыре копии массива вершин за одну итерацию цикла. Выделения происходят каждый раз, когда к `.vertices` осуществляется доступ, т. е. возвращается копия всех вертексов:
```cs
// BAD EXAMPLE
void Update() {
    for(int i = 0; i < mesh.vertices.Length; i++) {
        float x, y, z;

        x = mesh.vertices[i].x;
        y = mesh.vertices[i].y;
        z = mesh.vertices[i].z;

        // ...

        DoSomething(x, y, z);   
    }
}
```

Единое выделение массива, независимо от количества итераций цикла:

```cs
// A LITTLE BIT BETTER EXAMPLE

void Update() {
    var vertices = mesh.vertices;

    for(int i = 0; i < vertices.Length; i++) {

        float x, y, z;

        x = vertices[i].x;
        y = vertices[i].y;
        z = vertices[i].z;

        // ...

        DoSomething(x, y, z);   
    }
}
```

```cs
// GOOD EXAMPLE
List<Vector3> m_vertices = new List<Vector3>();

void Update() {
    mesh.GetVertices(m_vertices);

    for(int i = 0; i < m_vertices.Length; i++) {

        float x, y, z;

        x = m_vertices[i].x;
        y = m_vertices[i].y;
        z = m_vertices[i].z;

        // ...

        DoSomething(x, y, z);   
    }
}
```

## Оценочный Capacity
Если заранее известно, сколько будет занимать коллекция, лучше выставить это значение. У списка по умолчанию 16 элементов, если последовательное добавлять по одному элементу до 1000, то будет выполнено 16 + 32 + 64 + 128 + 256 + 512 лишних копирований и 7 временных массивов, который сборщик мусора должен будет удалить. 