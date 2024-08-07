#oop
- Модули верхнего уровня не должны зависеть от модулей нижнего уровня. И те и другие должны зависеть от абстракций.
- Абстракции не должны зависеть от деталей. Детали должны зависеть от абстракций.

```cs
class Book
{
    public string Text { get; set; }
    public ConsolePrinter Printer { get; set; }
 
    public void Print()
    {
        Printer.Print(Text);
    }
}
 
class ConsolePrinter
{
    public void Print(string text)
    {
        Console.WriteLine(text);
    }
}
```

Класс `Book`, представляющий книгу, использует для печати класс `ConsolePrinter`. При подобном определении класс `Book` зависит от класса `ConsolePrinter`. Более того мы жестко определили, что печать книгу можно только на консоли с помощью класса `ConsolePrinter`. Другие же варианты, например, вывод на принтер, вывод в файл или с использованием каких-то элементов графического интерфейса - все это в данном случае исключено. Абстракция печати книги не отделена от деталей класса `ConsolePrinter`. Все это является нарушением принципа инверсии зависимостей.

Теперь попробуем привести наши классы в соответствие с принципом инверсии зависимостей, отделив абстракции от низкоуровневой реализации:

```cs
interface IPrinter
{
    void Print(string text);
}
 
class Book
{
    public string Text { get; set; }
    public IPrinter Printer { get; set; }
 
    public Book(IPrinter printer)
    {
        this.Printer = printer;
    }
 
    public void Print()
    {
        Printer.Print(Text);
    }
}
 
class ConsolePrinter : IPrinter
{
    public void Print(string text)
    {
        Console.WriteLine("Печать на консоли");
    }
}
 
class HtmlPrinter : IPrinter
{
    public void Print(string text)
    {
        Console.WriteLine("Печать в html");
    }
}
```

Теперь абстракция печати книги отделена от конкретных реализаций. В итоге и класс `Book` и класс `ConsolePrinter` зависят от абстракции `IPrinter`. Кроме того, теперь мы также можем создать дополнительные низкоуровневые реализации абстракции `IPrinter` и динамически применять их в программе:

```cs
Book book = new Book(new ConsolePrinter());
book.Print();
book.Printer = new HtmlPrinter();
book.Print();
```

## Другое
Используя DIP, мы можем отложить решения, которые нам нужно принять относительно деталей реализации. Скрытие деталей низкого уровня за абстракцией позволяет нам кодировать наши политики высокого уровня без необходимости знать что-либо об этих деталях.

## Фабрики для зависимостей
Не зависеть от конкретных реализаций вообще может звучать хорошо, но это нереалистично. В конце концов, в конечном итоге нам нужно создавать объекты из наших типов. Лучший способ справиться с созданием наших объектов — это минимизировать влияние изменений в конкретных реализациях на нашу кодовую базу.

Вот где фабрики полезны. Если нужны какие-то изменения, они концентрируются в определенном месте. В качестве простого примера, если у нас есть игра, в которой есть разные враги на разных территориях, вместо того, чтобы создавать этих врагов в коде, реализующем каждую территорию, мы можем сделать что-то вроде этого:

```cs
public class EnemyFactory : IEnemyFactory
{
   public IEnemy GetGraveyardEnemy() => new Skeleton();
   public IEnemy GetPlainsEnemy() => new Dragon();
   public IEnemy GetCastleEnemy() => new Goblin();
}
```

Любое изменение типа врага в каждой локации можно сделать в одном файле, в одной строке. Нам не нужно искать внутри всей нашей кодовой базы операторы, которые зависят от конкретных реализаций врагов.

## Отношение к принципу открытости-закрытости
Зависимость от абстракций и ее преимущества могут напомнить нам о принципе [[Open - Closed Principle +]]. DIP и OCP тесно связаны. OCP описывает то, чего мы хотим достичь, а DIP описывает механизм для этого достижения.

Следуя DIP, у нас есть способ вносить изменения в поведение нашей программы, не меняя наш код. Поскольку наш код зависит от абстракций, мы можем изменять поведение класса, расширяя наш код новыми реализациями.

В приведенном выше примере с врагами, если бы нам нужен был новый тип врага для нашего кладбища, скажем, зомби, мы бы создали класс зомби, вернули его из метода `GetGraveyardEnemy`, но никаких изменений в коде, отвечающем за поведение нашего кладбища, не потребовалось бы, поскольку этот код зависел бы от интерфейса `IEnemies`, а не от какой-либо конкретной реализации класса `Skeleton`.

## Пример принципа инверсии зависимости
Предположим, у нас есть враги, которые всякий раз, когда атакуют, в нашем пользовательском интерфейсе мигает красный свет. Мы могли бы сделать что-то вроде этого:

```cs
public class Enemy
{
   private readonly UILight _uiLight;

   public Enemy(UILight uiLight)
   {
      _uiLight = uiLight;
   }
   
   public void Attack(Character character)
   {
      // Code for Attacking
      _uiLight.Flash(Color.Red);
   }
}

public class UILight
{
   public void Flash(Color color)
   {
      // Code fore flashing the Light
   }
}
```

Здесь высокоуровневый модуль (Враг) зависит от низкоуровневого модуля (индикатор пользовательского интерфейса). DIP говорит нам, что оба должны зависеть от абстракций. Но какая это должна быть абстракция? Кто-то мог бы сделать это:

```cs
public class Enemy
{
   private readonly IUILight _uiLight;
   
   public Enemy(IUILight uiLight)
   {
      _uiLight = uiLight;
   }
   
   public void Attack(Character character)
   {
      // Code for Attacking
      _uiLight.Flash(Color.Red);
   }
}

public interface IUILight
{
   void Flash(Color color);
}

public class UILight : IUILight
{
   public void Flash(Color color)
   {
      // Code fore flashing the Light
   }
}
```

Хотя обычно это соответствует DIP, это не правильно. Как мы видели в [[Концептуальное значение интерфейсов]] интерфейс концептуально принадлежит классу, который его использует, а не тому, который его реализует.

Наш класс `Enemy` не имеет причин знать, что он общается с индикатором пользовательского интерфейса, и у него нет причин знать, что этот индикатор должен мигать красным. Единственное, что ему нужно знать, это то, что он общается с классом, который может дать уведомление о том, что враг атакует, что-то вроде этого:

```cs
public class Enemy
{
   private readonly IUIEnemyNotification _iuiEnemyNotification;
   
   public Enemy(IUIEnemyNotification iuiEnemyNotification)
   {
      _iuiEnemyNotification = iuiEnemyNotification;
   }
   
   public void Attack(Character character)
   {
      // Code for Attacking
      _iuiEnemyNotification.EnemyAttack();
   }
}

public interface IUIEnemyNotification
{
   void EnemyAttack();
}

public class UILight : IUIEnemyNotification
{
   public void EnemyAttack()
   {
      Flash(Color.Red);
   }
   
   private void Flash(Color color)
   {
      // Code fore flashing the Light
   }
}

```

Любое изменение нашего пользовательского интерфейса, который является низкоуровневым модулем, а это значит, что он с большей вероятностью изменится, теперь можно сделать, не затрагивая другие наши классы: конкретную реализацию или нашу абстракцию. Фактически, класс `Enemy`с интерфейсом `IUIEnemyNotification` может находиться в своей собственной сборке, а любой класс, связанный с пользовательским интерфейсом, который реализует его, `IUIEnemyNotification`— в отдельной.

Это упрощает реализацию любых изменений. Если, например, позже требования изменятся и вместо мигающего света мы решим, что всякий раз, когда враг атакует, увеличивается счетчик, показывающий количество произошедших атак противника, единственное, что нам нужно сделать, это создать новый класс счетчика, который реализует метод `IUIEnemyNotification`и вызывает внутри него `EnemyAttack`свой собственный `Counter.Increase`метод.

Если у нас есть фабрика для наших конкретных зависимостей, как показано ранее, никому не нужно трогать класс Enemy. Фактически, два разных человека могут работать над кодом, один над Enemyклассом, а другой над любым классом, отвечающим за уведомления пользовательского интерфейса для врага, и единственное ограничение между ними заключается в том, что тот, который работает в пользовательском интерфейсе, должен следовать «контракту», который определен абстракцией IUIEnemyNotification. Любые изменения в классе пользовательского интерфейса для уведомлений не повлияют на Enemyкласс. Вот почему DIP — это механизм, который позволяет нам следовать принципу открытости-закрытости .
## Статический полиморфизм
Динамический полиморфизм — не единственный способ соответствовать DIP. Используя generics, мы также можем использовать статический полиморфизм. Статический полиморфизм имеет меньше накладных расходов, поскольку он разрешается во время компиляции, но также имеет меньшую гибкость, поскольку он не может изменяться во время выполнения.

Статический полиморфизм нарушает нашу зависимость исходного кода, но поскольку он разрешается во время компиляции, универсальный класс должен быть перекомпилирован каждый раз, когда мы вносим изменения. Обычно, если только не требуются небольшие выигрыши, которые мы получаем в скорости выполнения от статического полиморфизма, динамический полиморфизм предпочтительнее.

## Games
```cs
public class Hero: MonoBehaviour, IEntity
{
    public List<AbilityData> abilities;

    // same ability system will be used in enemy and other entities.
    // problem is that hero class very coupled with AbilitySystem
    // and we can not change it to another implementation
    AbilitySystem abilitySystem;
}

public class AbilitySystem
{
    public void UseAbility(AbilityData ability)
    {
        // do smth
    }
}

public class AbilityData: ScriptableObject
{
    // some data   
}
```

Fixing:

```cs
public class Hero: MonoBehaviour, IEntity
{
    public List<AbilityData> abilities;

    // now we can use any ability system we want
    // and easily change them

	// we can make this field serializable 
    IAbilitySystem abilitySystem;

	void Start()
	{
		abilitySystem = new AbilitySystem();
	}
}

public interface IAbilitySystem
{
    public void UseAbility(AbilityData ability);
}

public class AbilitySystem : IAbilitySystem
{
    public void UseAbility(AbilityData ability)
    {
        // do smth
    }
}

public class AbilityData: ScriptableObject
{
    // some data   
}
```

```cs
// Now if we need to change type of ability system we can
// make it in one place
// no need to change it in all SerializedFields 
public class AbilitySystemFactory: Singleton<AbilitySystemFactory>
{
    public IAbilitySystem Create()
    {
        return new AbilitySystem();
    }
}

public class Hero: MonoBehaviour, IEntity
{
    public List<AbilityData> abilities;
    IAbilitySystem abilitySystem;

	void Start()
	{
		abilitySystem = AbilitySystemFactory.Instance.Create();
	}
}
```

[[Service Locator +]] - если кроме `AbilitySystem` есть `InventorySystem` и другие

```cs
public class AbilitySystemFactory: MonoBehaviour
{
	void Awake()
	{
		ServiceLocator.ForSceneOf(this).Register(this);	
	}

    public IAbilitySystem Create()
    {
        return new AbilitySystem();
    }
}
```

```cs
public class Hero: MonoBehaviour, IEntity
{
    public List<AbilityData> abilities;
    IAbilitySystem abilitySystem;

	void Start()
	{
		abilitySystem = ServiceLocator.ForSceneOf(this).
		Get<AbilitySystemFactory>().Create();
	}
}

// or

public class Hero: MonoBehaviour, IEntity
{
    public List<AbilityData> abilities;
    [Inject] AbilitySystemFactory abilitySystemFactory;

	void Start()
	{
		abilitySystem = abilitySystemFactory.Create();
	}
}

public class AbilitySystemFactory: MonoBehaviour
{
	[Provide] public AbilitySystemFactory ProvideAbilitySystemFactory() => this;

	public IAbilitySystem CreateAbilitySystem()
	{
		return new AbilitySystem();
	}
}
```

With builder:

```cs
public interface Command
{
	public string Name {get;}
	public void Execute();
}

public class AbilityCommand: Command
{
    public string Name {get;}
    public AbilityData Ability {get;}

    public AbilityCommand(AbilityData ability)
    {
        Name = ability.name;
        Ability = ability;
    }

    public void Execute()
    {
        Debug.Log($"Using ability {Name}");
    }
}
```

```cs
public interface IAbilitySystem
{
    List<Command> Commands {get;}
    void Add(Command command);
    void Remove(Command command);
    void Swap(Command commandA, Command commandB);
    void ExecuteCommandAt(int index);
    void Debug();
}

public class AbilitySystem : IAbilitySystem
{
    public List<Command> Commands => throw new NotImplementedException();

    private AbilitySystem() { }

    public void Add(Command command)
    {
        Commands.Add(command);
    }

    public void Debug()
    {
        foreach (var command in Commands)
        {
            Debug.Log(command.Name);
        }
    }

    public void ExecuteCommandAt(int index)
    {
        Commands[index].Execute();
    }

    public void Remove(Command command)
    {
        Commands.Remove(command);
    }

    public void Swap(Command commandA, Command commandB)
    {
        var indexA = Commands.IndexOf(commandA);
        var indexB = Commands.IndexOf(commandB);
        Commands[indexA] = commandB;
        Commands[indexB] = commandA;
    }

    public class Builder
    {
        IEntity entity;
        List<AbilityData> abilities;

        public Builder(IEntity entity)
        {
            this.entity = entity;
        }

        public Builder WithAbilities(List<AbilityData> abilities)
        {
            this.abilities = abilities;
            return this;
        }

        public IAbilitySystem Build()
        {
            var abilitySystem = new AbilitySystem();
            foreach (var ability in abilities)
            {
                abilitySystem.Add(new AbilityCommand(ability));
            }
            return abilitySystem;
        }
    }
}



public class AbilitySystemFactory: MonoBehaviour, IDependencyProvider
{
    [Provide]
    public AbilitySystemFactory ProvideAbilitySystemFactory()
    {
        return this;
    }

    public IAbilitySystem Create(IEntity entity, List<AbilityData> abilities)
    {
        return new AbilitySystem.Builder(entity).WithAbilities(abilities).Build();
    }

}
```

> [!warning]
> Досмотреть часть D in SOLID (git amend) части с инспектором

