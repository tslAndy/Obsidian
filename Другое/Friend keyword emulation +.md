#other
## Введение
Допустим, у нас есть классы `Orc`, `Goblin` и `Enemy`. Мы хотим, только чтобы классы `Orc` и `Goblin` могли вызывать методы `Move` и `Attack` класса `Enemy`.
```cs
public class Enemy
{
      public void Move(int speed) => Console.WriteLine($"Moving with speed: {speed}");
      public void Attack(int damage) => Console.WriteLine($"Doing {damage} damage.");
}

public class Goblin 
{
   private readonly Enemy _enemy;
   private readonly int _speed = 10;
   private readonly int _damage = 100;

   public Goblin() => _enemy = new Enemy();

   public void Move()
   {
      Console.WriteLine("A goblin is Moving...");
      _enemy.Move(_speed);
      Console.WriteLine("The goblin has finished attacking.");
   }

   public void Attack()
   {
      Console.WriteLine("A goblin is Attacking...");
      _enemy.Attack(_damage);
      Console.WriteLine("The goblin has finished moving");
   }
}

public class Orc 
{
   private readonly Enemy _enemy;
   private readonly int _speed = 5;
   private readonly int _damage = 200;

   public Orc() => _enemy = new Enemy();

   public void Move()
   {
      Console.WriteLine("An Orc is Moving...");
      _enemy.Move(_speed);
      Console.WriteLine("The Orc has finished attacking.");
   }

   public void Attack()
   {
      Console.WriteLine("An Orc is Attacking...");
      _enemy.Attack(_damage);
      Console.WriteLine("The Orc has finished moving");
   }
}
```

## Internal
Internal модификатор доступа позволяет получать доступ к членам и типам только из классов, существующих в той же сборке.

```cs
public class Enemy
{
      internal void Move(int speed) => Console.WriteLine($"Moving with speed: {speed}");
      internal void Attack(int damage) => Console.WriteLine($"Doing {damage} damage.");
}
```

`Move` и `Attack` теперь доступны из всех классов в сборке, это значит, что сборка должна иметь только 3 класса. 

## Вложенные классы
```cs
public class Enemy
{
      private void Move(int speed) => Console.WriteLine($"Moving with speed: {speed}");
      private void Attack(int damage) => Console.WriteLine($"Doing {damage} damage.");
      
      public class Goblin 
      {
         private readonly Enemy _enemy;
         private readonly int _speed = 10;
         private readonly int _damage = 100;

         public Goblin() => _enemy = new Enemy();

         public void Move()
         {
            Console.WriteLine("A goblin is Moving...");
            _enemy.Move(_speed);
            Console.WriteLine("The goblin has finished attacking.");
         }

         public void Attack()
         {
            Console.WriteLine("A goblin is Attacking...");
            _enemy.Attack(_damage);
            Console.WriteLine("The goblin has finished moving");
         }
      }
      
      public class Orc 
      {
         private readonly Enemy _enemy;
         private readonly int _speed = 5;
         private readonly int _damage = 200;

         public Orc() => _enemy = new Enemy();

         public void Move()
         {
            Console.WriteLine("An Orc is Moving...");
            _enemy.Move(_speed);
            Console.WriteLine("The Orc has finished attacking.");
         }

         public void Attack()
         {
            Console.WriteLine("An Orc is Attacking...");
            _enemy.Attack(_damage);
            Console.WriteLine("The Orc has finished moving");
         }
      }
}
```

```cs
Enemy.Goblin goblin = new();
Enemy.Orc orc = new();
```

Проблема этого решения в том, что теперь могут быть только отношения "один к одному" или "один ко многим" (основной класс и вложенный / вложенные). Если бы захотели, чтобы классы Orc и Goblin имели дружественный доступ к классу Character, то пришлось бы использовать метод с internal.

## Вложенный класс в родительский базовый класс
Если у нас много классов «друзей», это уменьшит размер нашего класса, но увеличит стоимость обслуживания, поскольку теперь мы используем наследование поверх композиции. 

Теперь методы класса `Enemy` публичны, но это не имеет значения, поскольку `Enemy` имеет модификатор `protected`. Мы больше не можем создавать объекты класса, `Enemy`если только не наследуем от `BaseEnemy`, поэтому публичная доступность не является проблемой.

Теперь мы можем иметь более одного защищенного класса в базовом классе. Это означает, что наши классы `Orc` и `Goblin` могут дружить с более чем одним классом.

```cs
public abstract class BaseEnemy
{
   protected class Enemy
   {
      public void Move(int speed) => Console.WriteLine($"Moving with speed: {speed}");
      public void Attack(int damage) => Console.WriteLine($"Doing {damage} damage.");
   }  
}

public class Goblin : BaseEnemy
{
   private readonly Enemy _enemy;
   private readonly int _speed = 10;
   private readonly int _damage = 100;

   public Goblin() => _enemy = new Enemy();

   public void Move()
   {
      Console.WriteLine("A goblin is Moving...");
      _enemy.Move(_speed);
      Console.WriteLine("The goblin has finished attacking.");
   }

   public void Attack()
   {
      Console.WriteLine("A goblin is Attacking...");
      _enemy.Attack(_damage);
      Console.WriteLine("The goblin has finished moving");
   }
}
      
public class Orc : BaseEnemy
{
   private readonly Enemy _enemy;
   private readonly int _speed = 5;
   private readonly int _damage = 200;

   public Orc() => _enemy = new Enemy();

   public void Move()
   {
      Console.WriteLine("An Orc is Moving...");
      _enemy.Move(_speed);
      Console.WriteLine("The Orc has finished attacking.");
   }

   public void Attack()
   {
      Console.WriteLine("An Orc is Attacking...");
      _enemy.Attack(_damage);
      Console.WriteLine("The Orc has finished moving");
   }
}
```

```cs
Goblin goblin = new();
Orc orc = new();
```

К сожалению, с этим способом наш класс `Enemy` больше похож на вспомогательный класс. Другие классы не могут использовать его сейчас, поэтому если мы хотим класс, который может использоваться другими классами для его публичных поведений и иметь некоторые дружественные классы, которые могут использовать его приватные поведения, этот способ не подойдет.

## Вложенный класс с реализацией интерфейса по умолчанию
Разновидностью вышеприведенного варианта является использование реализации интерфейса по умолчанию. Теперь у нас может быть много интерфейсов с разными классами. Это упрощает поддержку, если мы хотим отношений «многие к одному» или «многие ко многим», но у нас все еще есть проблема, что эти классы могут использоваться только их друзьями и полностью недоступны из любого класса, который не реализует интерфейс, поэтому нет публичного доступа для некоторых классов и закрытого доступа для других.

```cs
public interface IEnemy
{
   protected class Enemy
   {
      public void Move(int speed) => Console.WriteLine($"Moving with speed: {speed}");
      public void Attack(int damage) => Console.WriteLine($"Doing {damage} damage.");
   }
}
```

```cs
public class Goblin : IEnemy
{
   private readonly IEnemy.Enemy _enemy;
   private readonly int _speed = 10;
   private readonly int _damage = 100;

   public Goblin() => _enemy = new IEnemy.Enemy();

   public void Move()
   {
      Console.WriteLine("A goblin is Moving...");
      _enemy.Move(_speed);
      Console.WriteLine("The goblin has finished attacking.");
   }

   public void Attack()
   {
      Console.WriteLine("A goblin is Attacking...");
      _enemy.Attack(_damage);
      Console.WriteLine("The goblin has finished moving");
   }
}
      
public class Orc : IEnemy
{
   private readonly IEnemy.Enemy _enemy;
   private readonly int _speed = 5;
   private readonly int _damage = 200;

   public Orc() => _enemy = new IEnemy.Enemy();

   public void Move()
   {
      Console.WriteLine("An Orc is Moving...");
      _enemy.Move(_speed);
      Console.WriteLine("The Orc has finished attacking.");
   }

   public void Attack()
   {
      Console.WriteLine("An Orc is Attacking...");
      _enemy.Attack(_damage);
      Console.WriteLine("The Orc has finished moving");
   }
}
```