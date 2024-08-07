#oop 
Сущноси программы должны быть открыты для расширения, но закрыты для изменения. Т. е. система должна быть построена таким образом, что все ее изменения должны быть реализованы добавлением нового кода, а не изменения существующего. 

Можно реализовать с помощью [[Стратегия +]]:
```cs
class Cook
{
    public string Name { get; set; }
 
    public Cook(string name)
    {
        this.Name = name;
    }
 
    public void MakeDinner(IMeal meal)
    {
        meal.Make();
    }
}
 
interface IMeal
{
    void Make();
}
class PotatoMeal : IMeal
{
    public void Make()
    {
        Console.WriteLine("Чистим картошку");
        Console.WriteLine("Ставим почищенную картошку на огонь");
        Console.WriteLine("Сливаем остатки воды, разминаем варенный картофель в пюре");
        Console.WriteLine("Посыпаем пюре специями и зеленью");
        Console.WriteLine("Картофельное пюре готово");
    }
}
class SaladMeal : IMeal
{
    public void Make()
    {
        Console.WriteLine("Нарезаем помидоры и огурцы");
        Console.WriteLine("Посыпаем зеленью, солью и специями");
        Console.WriteLine("Поливаем подсолнечным маслом");
        Console.WriteLine("Салат готов");
    }
}
```

Либо с помощью [[Шаблонный метод +]]:
```cs

abstract class MealBase
{
    public void Make()
    {
        Prepare();
        Cook();
        FinalSteps();
    }
    protected abstract void Prepare();
    protected abstract void Cook();
    protected abstract void FinalSteps();
}
 
class PotatoMeal : MealBase
{
    protected override void Cook()
    {
        Console.WriteLine("Ставим почищенную картошку на огонь");
        Console.WriteLine("Варим около 30 минут");
        Console.WriteLine("Сливаем остатки воды, разминаем варенный картофель в пюре");
    }
 
    protected override void FinalSteps()
    {
        Console.WriteLine("Посыпаем пюре специями и зеленью");
        Console.WriteLine("Картофельное пюре готово");
    }
 
    protected override void Prepare()
    {
        Console.WriteLine("Чистим и моем картошку");
    }
}
 
class SaladMeal : MealBase
{
    protected override void Cook()
    {
        Console.WriteLine("Нарезаем помидоры и огурцы");
        Console.WriteLine("Посыпаем зеленью, солью и специями");
    }
 
    protected override void FinalSteps()
    {
        Console.WriteLine("Поливаем подсолнечным маслом");
        Console.WriteLine("Салат готов");
    }
 
    protected override void Prepare()
    {
        Console.WriteLine("Моем помидоры и огурцы");
    }
}
```

## Другой пример
Предположим, что нам нужно создать персонажа, который может атаковать тремя различными типами атак: обычной, огненной и земной. Мы можем создать наш класс следующим образом:

```cs
public enum AttackType
{
   Normal,
   Fire,
   Earth
}

public class Character
{
   public void Attack(AttackType attackType)
   {
      if (attackType == AttackType.Earth)
      {
         Console.WriteLine("Earth attack");
      }
      else if (attackType == AttackType.Fire)
      {
         Console.WriteLine("Fire Attack");
      }
      else if (attackType == AttackType.Normal)
      {
         Console.WriteLine("Normal Attack");
      }
   }
}
```

Этот класс не следует OCP. Причина в том, что когда нам нужно добавить новый тип атаки, нам нужно пойти и изменить код класса. Нам нужно добавить еще один оператор if.

Это может показаться не проблемой, поскольку этот класс небольшой и простой для нужд этого примера, но в обычной программе каждый оператор if будет содержать все, что необходимо для выполнения определенного типа атаки. Будет много операторов, которые обрабатывают специфику каждой атаки, возможно, некоторые вложенные операторы if и, возможно, некоторые глобальные переменные, которые используются между всеми этими if.

В этом примере, если мы хотим следовать OCP, мы должны иметь возможность добавлять новые типы атак, не меняя код и не перекомпилируя класс. Вот класс, который имеет то же поведение, но следует OCP:

```cs
public class Character2
{
   public void Attack(IAttackType attackType)
   {
      attackType.Perform();
   }
}

public interface IAttackType
{
   void Perform();
}

public class EarthAttack : IAttackType
{
   public void Perform()
   {
      Console.WriteLine("Earth attack");
   }
}

public class FireAttack : IAttackType
{
   public void Perform()
   {
      Console.WriteLine("Fire attack");
   }
}

public class NormalAttack : IAttackType
{
   public void Perform()
   {
      Console.WriteLine("Normal attack");
   }
}
```

Пример сверху реализован с помощью динамического полиморфизма, со статическим его можно реализовать следующим образом:
```cs
public class Character3
{
   public void Attack<T>(T attackType) where T: IAttackType
   {
      attackType.Perform();
   }
}
```

## Более сложный пример
Допустим, у нас есть персонаж, который при вводе данных перемещается. В начале требования таковы: при нажатии клавиши `a` персонаж перемещается влево, а при нажатии клавиши `d` персонаж перемещается вправо. Наш код будет выглядеть примерно так:

```cs
public class Character
{
   public void Move()
   {
      var key = Console.ReadKey();

      if (key.KeyChar == 'a')
      {
         Console.WriteLine("Moving left");
      }
      else if (key.KeyChar == 'd')
      {
         Console.WriteLine("Moving right");
      }
   }
}
```

Этот код не следует OCP. Очевидно, что этот код открыт для изменений.
Здесь мы можем попытаться предусмотреть все возможные изменения. Входные данные могут измениться, например, нам может потребоваться изменить ключи, чтобы `a`и `d`были чем-то другим, или нам может потребоваться добавить новые ключи, но также может измениться и выход. Например, нам может потребоваться добавить функционал перемещения вверх.

Если мы попытаемся все это абстрагировать, наш код станет неоправданно сложным:

```cs
public interface IMovement
{
   void Perform();
}

public class MoveRight : IMovement
{
   public void Perform() => Console.WriteLine("Moving Right");
}

public class MoveLeft : IMovement
{
   public void Perform() => Console.WriteLine("Moving Left");
}
```

```cs
public interface IInput
{
   IMovement GetMovement();
}

public class RightMovementInput : IInput
{
   public IMovement GetMovement() => new MoveRight();
}

public class LeftMovementInput : IInput
{
   public IMovement GetMovement() => new MoveLeft();
}
```

```cs
public interface IInputFactory
{
   public IInput CreateInput(char key);
}

public class InputFactory : IInputFactory
{
   public IInput CreateInput(char key)
   {
      if (key == 'a')
         return new LeftMovementInput();
      if (key == 'd')
         return new RightMovementInput();

      throw new ArgumentOutOfRangeException(nameof(key)," Not a valid key!");
   }
}
```

```cs
public class Character2
{
   public void Move(IMovement movement)
   {
      movement.Perform();
   }
}
```

```cs
Character2 player = new();
IInputFactory inputFactory = new InputFactory();

var key = Console.ReadKey();

var inputAction = inputFactory.CreateInput(key.KeyChar);
player.Move(inputAction.GetMovement());
```

## Почему принцип OCP сложнее всего соблюдать
Есть две проблемы, с которыми нам следует быть осторожными при использовании OCP.

Первая проблема заключается в том, что в конечном итоге, независимо от того, насколько мы абстрагируемся, в нашей программе будет класс, который будет открыт для изменений. Мы можем переместить проблему так далеко, как захотим, даже дойдя до метода `main`, используя строителей и фабрики, но в конечном итоге будут некоторые классы, которые не подчиняются OCP. Это не значит, что мы не должны пытаться следовать OCP, но ни одна программа не будет иметь все свои классы, соответствующие OCP. В приведенном выше примере наш `InputFactory`класс открыт для изменений, если когда-либо будет добавлен новый ключ для ввода.

Даже если наш `InputFactory` класс открыт для модификации, код, который он содержит, изолирован от остальной части нашей программы, поэтому любые изменения будет легче вносить, и у нас будет меньше шансов создать ошибки. Однако есть более важная проблема.

Чем больше абстракций мы создаем, чтобы можно было легко расширять наши классы, не изменяя их, тем сложнее становятся наши системы из-за зависимостей, которые мы создаем. Если мы попытаемся кодировать так, чтобы наш код было легко изменять во всех видах изменений, которые могут потребоваться в будущем, мы будем переусложнять проектирование.