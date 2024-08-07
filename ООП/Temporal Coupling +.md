#oop 
Это неявная зависимость, при которое два или более утверждения должны быть вызваны определенном порядке.

В примере ниже может быть сложно найти ошибку, если метод Initialize не был вызван, так как поля класса содержат значения по умолчанию. При этом нужно, чтобы этот метод был вызван перед всеми остальными. 

```cs
public class Enemy
{
   private int _health;
   private int _damage;
   
   public void Initialize(int health, int damage)
   {
      _health = health;
      _damage = damage;
   }

   public void Attack()
   {
      Console.WriteLine($"Attacking and making {_damage} damage");
   }
}
```

Или как в этом примере, перед тем, как поместить что-то в сундук, его надо открыть,
а после каких-то операций закрыть.

```cs
public class Chest
{
   private readonly List<Item> _chestItems;
   private bool _isOpen;

   public Chest()
   {
      _chestItems = new List<Item>();
   }
   
   public void Open()
   {
      Console.WriteLine("Chest has opened");
      _isOpen = true;
   }

   public void AddItem(Item item)
   {
      if(_isOpen)
         _chestItems.Add(item);
   }

   public void Remove(Item item)
   {
      if(_isOpen)
         _chestItems.Remove(item);
   }

   public void RemoveAll()
   {
      if(_isOpen)
       _chestItems.Clear();
   }
   
   public void Close()
   {
      Console.WriteLine("Chest has closed");
      _isOpen = false;
   }
}
```

## Constructor Injection
Работает в случае, если имеем доступ к конструктору. Не сработает с `MonoBehaviour`

```cs
public class Enemy
{
   private readonly int _health;
   private readonly int _damage;
   
   public  Enemy(int health, int damage)
   {
      _health = health;
      _damage = damage;
   }

   public void Attack()
   {
      Console.WriteLine($"Attacking and making {_damage} damage");
   }
}
```

## Abstract Factory

```cs
public interface IEnemyFactory
{
   public IEnemy Create(int health, int damage);
}

public interface IEnemy
{
   public void Initialize(int health, int damage);
   public void Attack();
}
```

```cs
public class EnemyFactory : IEnemyFactory
{
   public IEnemy Create(int health, int damage)
   {
      IEnemy enemy = new Enemy();
      enemy.Initialize(health, damage);
      return enemy;
   }
}

internal class Enemy : IEnemy
{
   private int _health;
   private int _damage;
   
   public void Initialize(int health, int damage)
   {
      _health = health;
      _damage = damage;
   }

   public void Attack()
   {
      Console.WriteLine($"Attacking and making {_damage} damage");
   }
}
```

```cs
IEnemyFactory enemyFactory = new EnemyFactory();
IEnemy enemy = enemyFactory.Create(10, 5);
enemy.Attack();
```

>[!info] 
>Класс `Enemy` с модификатором `internal`, поэтому мы можем поместить фабрику и врага в отдельную библиотеку, и мы не сможем напрямую создавать экземпляры врага вне этой библиотеки

## Template Method

```cs
public abstract class Chest
{
   private readonly List<Item> _chestItems;

   protected Chest()
   {
      _chestItems = new List<Item>();
   }
   
   public void Execute()
   {
      Open();
      Operations();
      Close();
   }

   protected void AddItem(Item item)
   {
      _chestItems.Add(item);
   }

   protected void Remove(Item item)
   {
      _chestItems.Remove(item);
   }

   protected void RemoveAll()
   {
      _chestItems.Clear();
   }

   protected abstract void Operations();
   
   private void Open()
   {
      Console.WriteLine("Chest has opened");
   }
   
   private void Close()
   {
      Console.WriteLine("Chest has closed");
   }
}
```

```cs
public class AddTwoItemsToChest : Chest
{
   private readonly Item _item1;
   private readonly Item _item2;

   public AddTwoItemsToChest(Item item1, Item item2) : base()
   {
      _item1 = item1;
      _item2 = item2;
   }

   protected override void Operations()
   {
      AddItem(_item1);
      AddItem(_item2);
   }
}
```

```cs
Item item1 = new Item();
Item item2 = new Item();
AddTwoItemsToChest addTwoItemsToChest = new(item1, item2);

addTwoItemsToChest.Execute();
```


## Composition

В данном примере проблемой является то, что кто угодно может создать новый экземпляр класса `Chest` и вызывать его методы, без вызова `Open` и `Close` в начале и конце соответственно. Его можно сделать `internal` либо эмулировать `friend` из C++.

```cs
public class AddTwoItemsToChest
{
   private readonly Chest _chest;
   public AddTwoItemsToChest(Chest chest)
   {
      _chest = chest;
   }

   public void Execute(Item item1, Item item2)
   {
      _chest.Open();
      _chest.AddItem(item1);
      _chest.AddItem(item2);
      _chest.Close();
   }
}
```

```cs
Item item1 = new Item();
Item item2 = new Item();
Chest chest = new Chest();
AddTwoItemsToChest addTwoItemsToChest = new(chest);

addTwoItemsToChest.Execute(item1, item2);
```

## Избавиться от внутреннего состояния
Если вынести параметр damage из врага и сделать его параметром метода Attack, то у нас вообще не будет необходимости в инициализации, однако при этом класс не сможет иметь собственный урон. Такой подход работает хорошо в случае с DOD, когда данные и поведение отделены друг от друга.


## Параметр-делегат
```cs
public class Chest
{
   public List<Item> ChestItems { get; }

   public Chest()
   {
      ChestItems = new List<Item>();
   }

   public void Operation(Action operation)
   {
      Open();
      operation?.Invoke();
      Close();
   }
   
   private void Open()
   {
      Console.WriteLine("Chest has opened");
   }
   
   private void Close()
   {
      Console.WriteLine("Chest has closed");
   }
}
```

```cs
Item item1 = new Item();
Item item2 = new Item();
Chest chest = new Chest();

chest.Operation(() =>
{
   Console.WriteLine("Example operation");
   chest.ChestItems.Add(item1);
   chest.ChestItems.Add(item2);
});
```