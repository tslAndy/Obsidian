#oop 

Важно использовать методы, определенные в классе, для операций над его данными, вместо прямой проверки значений данных. Другими словами, мы должны максимизировать использование поведения модуля и минимизировать использование его внутреннего состояния для решения об дальнейшем выполнении кода. 

Этот принцип может быть нарушен двумя способами
- Мы запрашиваем внутреннее состояние объекта, и, в зависимости от этого состояния, вызываем методы *этого* объекта
- Мы запрашиваем внутреннее состояние объекта, и, в зависимости от этого состояния, вызываем методы *другого* объекта

## Запрос внутреннего состояния объекта
Рассмотрим следующий класс:
```cs
public class Person(string name, int age)
{
   private string Name { get; } = name;
   public int Age
   {
      get => age;
      set => age = value;
   }

   public void DrinkWine(int numberOfGlasses)
   {
      Console.WriteLine($" {Name} drinks {numberOfGlasses} glasses of wine at age {Age}");
   }
}
```


```cs
Person mike = new("Mike", 15);

for (int age = mike.Age; age < 22; age++)
{
   mike.Age++;
   Console.WriteLine($"Happy birthday Mike! You are {mike.Age} years old!");
   
   if (mike.Age >= 18)
      mike.DrinkWine(2);
}
```

Здесь мы нарушаем принцип, так как запрашиваем внутренее состояние объекта, и в зависимости от него вызываем его метод `DrinkWine`.

Так же мы раскрыли свойство `Age`, что плохо по двум причинам:

У нас только один условный переход для проверки возраста, но в будущем может понадобиться отталкиваться от разного возраста в зависимости от места. Поэтому придется искать все эти проверки по кодовой базе.

Может быть предложено решение, нарушающее принцип, но решающую проблему с изменением возраста:

```cs
// in Person class
public static int DrinkingAge => 18;


// Usage
if (mike.Age >= Person.DrinkingAge)
      mike.DrinkWine(2);
```

Проблема здесь в том, что у нас и так есть два нарушения (запрашиваем Age и DrinkingAge), но и имеем случай [[Temporal Coupling +]] . Каждый раз, когда кто-то хочет использовать метод DrinkWine, он должен помнить выполнить проверки перед этим. 

Решаем проблему следующим образом:
```cs
public class Person(string name, int age)
{
   private static int DrinkingAge => 18;
   private string Name { get; } = name;
   public int Age
   {
      get => age;
      set => age = value;
   }

   public void DrinkWine(int numberOfGlasses)
   {
      if(Age >= DrinkingAge)
         Console.WriteLine($" {Name} drinks {numberOfGlasses} glasses of wine at age {Age}");
   }
}
```

## Замена условного перехода

```cs
public class Person(string name, int age)
{
   private string Name { get; } = name;
   public int Age
   {
      get => age;
      set => age = value;
   }

   public void DrinkWine(int numberOfGlasses)
   {
         Console.WriteLine($" {Name} drinks {numberOfGlasses} glasses of wine at age {Age}");
   }

   public void DrinkOrangeJuice(int numberOfGlasses)
   {
      Console.WriteLine($" {Name} drinks {numberOfGlasses} glasses of orange juice at age {Age}");
   }
}


// Client
for (int age = mike.Age; age < 22; age++)
{
   mike.Age++;
   Console.WriteLine($"Happy birthday Mike! You are {mike.Age} years old!");

	// NOT GOOD
   if(mike.Age >= 18)
      mike.DrinkWine(2);
   else
      mike.DrinkOrangeJuice(2);
}
```

Инкапсуляция двух методов (приватные) и создание публичного который решает, какой из них вызвать:

```cs

public class Person(string name, int age)
{
   private static int DrinkingAge => 18;
   private string Name { get; } = name;

   public int Age
   {
      get => age;
      set => age = value;
   }

   public void DrinkBeverage(int numberOfGlasses)
   {
      if(Age >= DrinkingAge)
         DrinkWine(numberOfGlasses);
      else
         DrinkOrangeJuice(numberOfGlasses);
   }
   
   private void DrinkWine(int numberOfGlasses)
   {
         Console.WriteLine($" {Name} drinks {numberOfGlasses} glasses of wine at age {Age}");
   }

   private void DrinkOrangeJuice(int numberOfGlasses)
   {
      Console.WriteLine($" {Name} drinks {numberOfGlasses} glasses of orange juice at age {Age}");
   }
}
```

Если мы не можем инкапсулировать методы, например, потому что они должны быть доступны клиентам, добавляем проверки в эти методы:
```cs
public void DrinkBeverage(int numberOfGlasses)
{
    if(Age >= DrinkingAge)
        DrinkWine(numberOfGlasses);
    else
        DrinkOrangeJuice(numberOfGlasses);
}
   
public void DrinkWine(int numberOfGlasses)
{
    if(Age >= DrinkingAge)
        Console.WriteLine($" {Name} drinks {numberOfGlasses} glasses of wine at age {Age}");
}

public void DrinkOrangeJuice(int numberOfGlasses)
{
    Console.WriteLine($" {Name} drinks {numberOfGlasses} glasses of orange juice at age {Age}");
}

```

Возможна проблема, например, нужно вызывать два метода основываясь на других данных помимо возраста, нужно создать метод, который удаляет дублированную проверку и делает ее публичной, так что она может быть вызвана пользователями класса:

```cs
public class Person(string name, int age)
{
   private static int DrinkingAge => 18;
   private string Name { get; } = name;
   
   public int Age
   {
      get => age;
      set => age = value;
   }

   public void DrinkBeverage(int numberOfGlasses)
   {
      if(IsInLegalDrinkingAge())
         DrinkWine(numberOfGlasses);
      else
         DrinkOrangeJuice(numberOfGlasses);
   }
   
   public void DrinkWine(int numberOfGlasses)
   {
      if(IsInLegalDrinkingAge())
         Console.WriteLine($" {Name} drinks {numberOfGlasses} glasses of wine at age {Age}");
   }

   public void DrinkOrangeJuice(int numberOfGlasses)
   {
      Console.WriteLine($" {Name} drinks {numberOfGlasses} glasses of orange juice at age {Age}");
   }

   public bool IsInLegalDrinkingAge() => Age >= DrinkingAge;
}
```

При необходимости, метод `DrinkWine` можно сделать `TryDrinkWine`, который будет возвращать булево значение. 

## Получение информации о другом объекте
Принцип так же применим когда мы запрашиваем состояние одного объекта, чтобы вызвать методы в другом. Это создает больше зависимостей между типами. 

Допустим, у нас есть три класса, `Player`, `PlayerCollection` и `Enemy`. Класс врага содержит метод Attack, который принимает Player в качестве аргумента. 

Предположим, что мы хотим, чтобы враг атаковал игрока, ближайшего к какой-то точке.

Один из способов это создать класс с методом, который вычисляет ближайшего к точке игрока принимая `PlayerCollection` как аргумент и передавая Player назад в метод Attack.

Другой подход это метод в классе `PlayerCollection` который возвращает ближайшего к точке игрока, и передает в Attack. В этом случае ответственность за вычисление ложится на класс `PlayerCollection`. 

Наконец, мы можем пропустить вопрос о том, кто является ближайшим игроком к точке, и создать новый метод в классе Enemy, который принимает коллекцию игроков и точку на нашем уровне в качестве аргументов. Этот метод вычисляет ближайшего игрока к этой точке и внутренне вызывает метод Attack с этим игроком в качестве параметра. В этом случае мы следуем принципу. 

Какой метод выбрать, зависит от контекста, не стоит всегда следовать приницпу. Некоторые факторы, которые влияют на решение:
- Если нужно вычисять дистанции не только для игроков, но и для других элементов, то лучше создать отдельный класс
- Другие классы должны знать информацию о расстоянии игроков? Если да, то метод в классе `PlayerCollection` 
- Если функционал используется только в классе `Enemy`, то лучше чтобы это было его ответственностью