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

```cs
Container
.Bind<IWeapon>()
.To<Pistol>()
.FromFactory<PistolFactory>();


public class PistolFactory: IFactory<Pistol>
{
	[Inject] private Difficulty _difficulty;

	public Pistol Create()
	{
		if (_difficulty == Difficulty.Easy)
			return new StrongPistol();
		return new WeakPistol();
	}
}
```

```cs
Container.Bind<Foo>().FromMethod(SomeMethod);

Foo SomeMethod(InjectContext context)
{
    ...
    return new Foo();
}
```

```cs
Container.Bind<Foo>().FromMethodMultiple(GetFoos);

IEnumerable<Foo> GetFoos(InjectContext context)
{
    ...
    return new Foo[]
    {
        new Foo(),
        new Foo(),
        new Foo(),
    }
}
```

```cs
/*
Это более общая и более мощная альтернатива FromFactory, поскольку вы можете использовать любой метод построения, который захотите для своей пользовательской фабрики (в отличие от FromFactory, который предполагает FromNew().AsCached())
*/

class FooFactory : ScriptableObject, IFactory<Foo>
{
    public Foo Create()
    {
        // ...
        return new Foo();
    }
}

Container.Bind<Foo>().
	FromIFactory(x => x.
				 To<FooFactory().
				 FromScriptableObjectResource("FooFactory")
				 ).
	AsSingle();
```

### Scope
Как часто нужно переспользовать объект между несколькими инъекциями
```cs
Container
.Bind<IWeapon>()
.To<Pistol>()
.AsTransient(); // new instance per injection

Container
.Bind<IWeapon>()
.To<Pistol>()
.AsSingle(); // reuse for each result type
```

```cs
// будет создано два разных экземпляра
Container.Bind<IAudioService>().To<AudioService>().AsCached();
Container.Bind<AudioService>().AsCached();

// будет создан только экземпляр
Container.Bind<IAudioService>().To<AudioService>().AsSingle();
Container.Bind<AudioService>().AsSingle();


public class Player
{
	[Inject] IAudioService _audioService;
}

public class Weapon
{
	[Inject] AudioService _audioService;
}
```

### Arguments
```cs
Container
.Bind<IWeapon>()
.To<Pistol>()
.WithArguments(120);

public class Pistol
{
	public int Ammo { get; private set;}

	public Pistol(int ammo)
	{
		Ammo = ammo;
	}
}
```

### Conditions
```cs
Container
.Bind<IWeapon>()
.To<Pistol>()
.When(c => c.ObjectType == typeof(Player));

// or
Container
.Bind<IWeapon>()
.To<Pistol>()
.WhenInjectedInto<Player>;

public class Player
{
	[Inject] private IWeapon _weapon;

	// will recieve injection
	public Player(IWeapon weapon)
	{
		_weapon = weapon;
	}
}

public class Enemy
{
	[Inject] private IWeapon _weapon;

	// will not recieve injection
	public Player(IWeapon weapon)
	{
		_weapon = weapon;
	}
}
```

### Sub Containers
```cs
Container
.Bind<IWeapon>()
.To<Pistol>()
.CopyIntoAllSubContainers;
```

### Non Lazy
```cs
Container
.Bind<IWeapon>()
.To<Pistol>()
.NonLazy();
```

## Installers
Type of installers:
```cs
Installer<>
MonoInstaller
ScriptableObjectInstaller
```

```cs
public class GameInstaller: Installer<GameInstaller>
{
	public override void InstallBindings()
	{
		Container.Bind<IPlayer>().To<Player>();
	}
}

public class AppInstaller: MonoInstaller
{
	public override void InstallBindings()
	{
		GameInstaller.Install(Container);
	}
}

public class AppInstaller: MonoInstaller
{
	// visible in inspector
	public string PlayerName;
	
	public override void InstallBindings()
	{
		GameInstaller.Install(Container, PlayerName);
	}
}
```

## Reusing Mono Installers
- In-Scene Prefabs (attach `MonoInstaller` to separate `GameObject` and make prefab of it)
- Prefab Installers (Есть скрипт `SceneContext`, заносим префаб в список Prefab Installers)
- Resources (создаем префаб, как и раньше, и помещаем его в папку Resources)

```cs
public class GameInstaller: Installer<GameInstaller>
{
	// if necessary we can use arguments
	// [Inject] private string PlayerName;

	public override void InstallBindings()
	{
		Container.Bind<IPlayer>().To<Player>();
	}
}

public class AppInstaller: MonoInstaller
{
	public override void InstallBindings()
	{
		GameInstaller.InstallFromResource("Game Installer", Container);

		// version for arguments
		// GameInstaller.InstallFromResource("Game Installer", Container, "Player1");
	}
}
```

## Context
- Scene Context
- `GameObject` Context
- Decorator Context
- Project Context 

### Scene Context
- Entry point of each scene
- Executed first
- Initializes scene dependencies

### GameObject Context
- Groups dependencies
- Multiple instances per scene
- Facade pattern

### Decorator Context
- Nested dependencies
- Dynamically adds functionality
- Share SceneContext's Container

Project Context
- Global dependencies
- Initialized once
- Dependencies reused in every scene

## Zenject Order of Operations
- Unity Awake Phase
	- ProjectContext initialized (once)
		- Injectable MonoBehaviours passed into Container
		- InstallBindings called on each Installer
		- Non-lazy objects constructed


## Non MonoBehaviour classes

Zenject interfaces:
- IInitializable
	- Must implement Initialize method
	- Initialize is called during Unity's Start phase
- ITickable, ILateTickable, IFixedTickable
	- Must implement Tick method
	- Tick is called on Update, LateUpdate, FixedUpdate
- IDisposable
	- Must implement Dispose method
	- Dispose is called during OnDestroy


We need to refactor this class:
```cs
public class TwitterService
{
    private string _apiKey;

    public TwitterService(string apiKey)
    {
        _apiKey = apiKey;
    }

    private void Start()
    {
        Authenticate();
    }

    private void OnDestroy()
    {
        Deauthenticate();
    }

    private void Update()
    {
        if (IsOverRateLimit())
        {

        }
    }

    public void Authenticate() {}
    public void Deauthenticate() {}
    public bool IsOverRateLimit() { return false; }
}
```

## Signals

```cs
public class PlayerHealedSignal: Signal<PlayerHealedSignal, int>
{

}


public class Player
{
    private PlayerHealedSignal _onPlayerHealed;

    public Player(PlayerHealedSignal onPlayerHealed)
    {
        _onPlayerHealed = onPlayerHealed;
    } 

    public void Heal(int amount)
    {
        _onPlayerHealed.Fire(healed);
    }
}

public class GameInstaller: MonoInstaller<GameInstaller>
{
    public override void InstallBindings()
    {
        Container.DeclareSignal<PlayerHealedSignal>();
        Container.BindSignal<int, PlayerHealedSignal>()
        .To(healed => Debug.Log("onPlayerHeals"));
    }   
}
```

## Sub Containers
Project Context -> Scene Context -> Game Object Context

Sub-container
- Child of another container
- Binding are only visible to itself and its children
- Used to abstract related dependency

### Simple sub-container definition
```cs
public class GameInstaller: MonoInstaller<GameInstaller> 
{
    public override void InstallBindings()
    {
        Container.Bind<Greeter>().FromSubContainerResolve()
            .ByMethod(InstallGreeter).AsSingle();
    }   

    private void InstallGreeter(DiContainer subContainer)
    {
        subContainer.Bind<Greeter>().AsSingle();
        subContainer.BindInstance("Hello world");
    }
}
```