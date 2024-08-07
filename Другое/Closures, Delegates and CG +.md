#other
## Closures
Рассмотрим следующий код:
```cs
bool Contains(int val, List<int> list)
{
	return list.Find(i => i == val) >= 0;
}
```

`List.Find` принимает делегат типа `Predicate<T>` который определен как `bool Predicate(T)`. Мы передаем лямбду `i => i == val` которая соответсвует сигнатуре `T/int` и возвращает bool. Каким-то образом функция-лямбда имеет доступ к локальной переменной val верхней функции Contains.

Для этого создается новый класс с полем для каждой локальной переменной, к которой замыканию нужен доступ, и помещает лямбду в класс как функцию экземпляра.

Замыкание имеет следующие эффекты:
- Каждая лямбда в замыкании создает класс
- Использование лямбды создает новый экземпляр класса
- Использование лямбды копирует все локальные переменные который использует лямбда в экземпляр класса
- Использование лямбды создает новый экземпляр делегата для оборачивания экземплярной функции-лямбды

Так как каждая лямбда должна содержать свое собственное множество переменных, мы не можем создать единственный класс. Вместо этого мы создадим интерфейс аналогичный определению делегата. 

```cs
delegate bool Predicate<T>(T obj);

interface IPredicate<T>
{
	bool Invoke(T obj);
}
```

Нужно создать собственный метод расширения для списка:
```cs
using System.Collections.Generic;
 
public static partial class ListExtensions
{
	public static int Find<T>(this List<T> list, IPredicate<T> pred)
	{
		for (int i = 0, count = list.Count; i < count; ++i)
		{
			if (pred(list[i]))
			{
				return i;
			}
		}
		return -1;
	}
}

class ContainsPredicate : IPredicate<int>
{
	public int Val;
 
	public bool Invoke(int obj)
	{
		return obj == Val;
	}
}
 
bool Contains(int val, List<int> list)
{
	ContainsPredicate pred = new ContainsPredicate { Val = val };
	return list.Find(pred) >= 0;
}

class TestScript
{
	private static readonly ContainsPredicate containsPredicate = new ContainsPredicate();
 
	bool Contains(int val, List<int> list)
	{
		containsPredicate.Val = val;
		return list.Find(containsPredicate) >= 0;
	}
}

// With pool:
class TestScript
{
	private static readonly Stack<ContainsPredicate> containsPredicatePool
		= new Stack<ContainsPredicate>();
 
	bool Contains(int val, List<int> list)
	{
		// Need a predicate. Get from the pool if there are any. Otherwise create one.
		ContainsPredicate pred = containsPredicatePool.Count > 0
			? containsPredicatePool.Pop()
			: new ContainsPredicate();
 
		// Use the predicate
		pred.Val = val;
		int index = list.Find(pred) >= 0;
 
		// Done with the predicate. Put it into the pool.
		containsPredicatePool.Push(pred);
 
		return index;
	}
}
```

## Delegates
Рассмотрим код:
```cs
void TakeDelegate(Action del)
{
}

void MyFunction()
{
}
 
TakeDelegate(MyFunction);
```

Здесь трудно заметить аллокацию, так как она неявно встроена в язык, и по факту происходит следующее:

```cs
TakeDelegate(new Action(MyFunction));
```

При этом если использовать static Action для кэширования, то новых аллокаций не будет.

Проверить этот код: 

```cs
static Action someDelegate;
someDelegate = MyFunction;

TakeDelegate(someDelegate)
```

