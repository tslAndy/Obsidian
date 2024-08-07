## Injections
- Constructor Injection
- Field Injection
- Property Injection
- Method Injection

### Constructor Injection
Использует конструктор класса чтобы выяснить, какие объекты нужно инъецировать. Это предпочтительный способ, так как при нем ясно видны все зависимости и можно ясно понять, когда зависимости будут инъецированы. Так же это упрощает замену DI фреймворка на другой.

```cs
public class Weapon
{
	private readonly BulleFactory _bulletFactory;
	private readonly Settings _settings;

	public Weapon(BulleFactory bulletFactory, Settings settings)
	{
		_bulletFactory = bulletFactory;
		_settings = settings;
	}
}
```

### Field Injection
Инъекция происходит сразу после вызова конструктора.
```cs
public class Weapon
{
	[Inject] private BulleFactory _bulletFactory;
	[Inject] private Settings _settings;
}
```

### Property Injection
```cs
public class Weapon
{
	[Inject] private BulleFactory _bulletFactory {get; set;}
	[Inject] private Settings _settings {get; private set;}
}
```

### Method Injection
Это важно в MonoBehaviour, т.к. они не позволяют работать с конструкторами. Благодаря использованию инъекции метода мы можем устанавливать все зависимости нашего класса в одном месте.
```cs
public class Weapon
{
	private BulleFactory _bulletFactory;
	private Settings _settings;

	[Inject]
	public void Construct(BulleFactory bulletFactory, Settings settings)
	{
		_bulletFactory = bulletFactory;
		_settings = settings;
	}
}
```

## Bindings
Здесь Zenject должен знать, какую реализацию предоставить конструктору, зависящего от интерфейса IWeapon. Это называется Dependency Mapping (binding of a type to an instance).
```cs
public class Player
{
	private IWeapon _weapon;

	public Player(IWeapon weapon)
	{
		_weapon = weapon;
	}
}
```

```cs
public class Pistol: IWeapon
{
	private BulletFactory _bulletFactory;

	public Pistol(BulletFactory bulletFactory)
	{
		_bulletFactory = bulletFactory;
	}
}
```

```cs
Container.Bind<Player>().AsSingle();
Container.Bind<IWeapon>().To<Pistol>();
Container.Bind<BulletFactory>().AsSingle();
```

### Binding Statement Anatomy
```cs
Container
.Bind<ContractType>()
.To<ResultType>()
.WithId(Identifier)
.FromConstructionMethod()
.AsScope()
.WithArguments(Arguments)
.When(condition)
.CopyIntoSubContainers()
.NonLazy();
```

### Identifiers
```cs
public class Player
{
	[Inject(Id = "PlayerWeapon")]
	private IWeapon _weapon;
}

Container.Bind<IWeapon>().To<Pistol>().WithId("PlayerWeapon");
```

### Construction Method
```cs
Container
.Bind<IWeapon>()
.To<Pistol>()
.FromNew();

Container
.Bind<IWeapon>()
.To<Pistol>()
.FromInstance(new Pistol());
```

3:43