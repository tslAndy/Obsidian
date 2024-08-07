#other 
## Самый частый случай
Для добавления функционала к типам, к реализации которых у нас нет доступа либо от них нельзя наследоваться, например векторы, transform, числа, цвета и т. д.

```cs
public static class MyExtensionMethods
{
    public static float AddPi(this int number)
    {
        return number + 3.14f;
    }

    public static float MultiplyByTen(this float number)
    {
        return number * 10;
    }

    public static bool CheckBetween(this float number1, int number2, int number3)
    {
        return number1 > number2 && number1 < number3;
    }
}


bool result = foo.AddPi().MultiplyByTen().CheckBetween(60, 80)
```

```cs
// compare previous to classic approach
public static class MyMethods
{
    public static float AddPi(int number)
    {
        return number + 3.14f;
    }

    public static float MultiplyByTen(float number)
    {
        return number * 10;
    }

    public static bool CheckBetween(float number1, int number2, int number3)
    {
        return number1 > number2 && number1 < number3;
    }
}

bool result = MyMethods.CheckBetween(MyMethods.MultiplyByTen(MyMethods.AddPi(foo)),60,80)
```

## `Enum` extension

```cs
public enum BusStop
{
    firstStop,
    secondStop,
    thirdStop,
    fourthStop
}

var result =  currentStop.GetNext().GetTimeOfArrival();
```

## Architecture

```cs
public interface IEnemy
{
    void Attack();
}

public class Skeleton : IEnemy
{
    public void Attack()
    {
        Console.WriteLine("The skeleton Attacks");
    }
}

public class Goblin : IEnemy
{
    public void Attack()
    {
        Console.WriteLine("The Goblin Attacks");
    }

	// Only goblin has special attack
    public void GoblinSpecialAttack()
    {
        Console.WriteLine("The goblin performs a special attack");
    }
}
```

```cs
List<IEnemy> enemiesList = new List<IEnemy>()
{
    new Skeleton(),
    new Skeleton(),
    new Goblin()
};
```

```cs
// NOT GOOD
foreach (IEnemy enemy in enemiesList)
{
    if (enemy is Goblin) 
        ((Goblin)enemy).GoblinSpecialAttack();
    else 
        enemy.Attack();
}
```

```cs
public static class IEnemyExtensions
{
    public static void SpecialAttack(this IEnemy enemy)
    {
        if (enemy is Goblin)
            ((Goblin)enemy).GoblinSpecialAttack();
        else
            enemy.Attack();
    }
}

foreach (IEnemy enemy in enemiesList)
{
    enemy.SpecialAttack();
}
```

## Adding behavior to nulls

```cs
List<IEnemy> enemiesList = new List<IEnemy>()
{
    new Skeleton(),
    new Skeleton(),
    null,
    new Goblin()
};
```

```cs
public static class IEnemyExtensions
{
    public static void SpecialAttack(this IEnemy enemy)
    {
        if (enemy is null)
        {
            Console.WriteLine("Do nothing or default behaviour");
            return;
        }

        if (enemy is Goblin)
            ((Goblin)enemy).GoblinSpecialAttack();
        else
            enemy.Attack();
    }
}
```

## Differentiate between development and production code

```cs
// NOT GOOD

public class Goblin
{
    public void Attack()
    {
#if DEBUG
        Console.WriteLine("The Goblin Attacks in debug build");
#else
        Console.WriteLine("The Goblin Attacks in release build");
#endif
    }

    public void GoblinSpecialAttack()
    {
#if DEBUG
        Console.WriteLine("The goblin performs a special attack in debug build");
#else
        Console.WriteLine("The goblin performs a special attack in release build");
#endif
    }

    public void Move()
    {
#if DEBUG
        Console.WriteLine("The goblin moves in debug build");
#else
        Console.WriteLine("The goblin moves in release build");
#endif
    }
}
```

```cs
namespace GoblinDebugExtensions
{
    public static class GoblinExtensions
    {
        public static void Attack(this Goblin goblin)
        {
            Console.WriteLine("The Goblin Attacks in debug build");
        }

        public static void GoblinSpecialAttack(this Goblin goblin)
        {
            Console.WriteLine("The goblin performs a special attack in debug build");
        }

        public static void Move(this Goblin goblin)
        {
            Console.WriteLine("The goblin moves in debug build");
        }
    }
}

namespace GoblinReleaseExtensions
{
    public static class GoblinExtensions
    {
        public static void Attack(this Goblin goblin)
        {
            Console.WriteLine("The Goblin Attacks in release build");
        }

        public static void GoblinSpecialAttack(this Goblin goblin)
        {
            Console.WriteLine("The goblin performs a special attack in release build");
        }

        public static void Move(this Goblin goblin)
        {
            Console.WriteLine("The goblin moves in release build");
        }
    }
}
```

```cs
#if DEBUG
using GoblinDebugExtensions;
#else
using GoblinReleaseExtensions;
#endif
```