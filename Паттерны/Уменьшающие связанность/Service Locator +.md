#decoupling
Обращение к конкретым классам систем плохая идея, так как мы взаимодействуем с конкретной реализацией системы, и при ее изменении придется менять все места, в которых к ней обращаются. Лучше иметь одно место, в котором контроллируется, как систему находят. Т. е. мы отделяем код, которому нужна служба от кода, в котором эта служба находится.

Класс службы определяет абстрактный интерфейс для набора операций, конкретный провайдер службы их реализует. Локатор обеспечивает доступ, скрывая конкретный тип службы и процесс ее поиска. 

Вместо того, чтобы использовать глобальный механизм для предоставления некоторого кода доступа к объекту, который ему нужен, сначала рассмотрите возможность передачи ему объекта . Это очень просто, и это делает связь совершенно очевидной.

```cs

abstract class Audio
{
	public abstract void PlaySound(int soundId);
	public abstract void StopSound(int soundId);
	public abstract void StopAllSounds(int soundId);
}

class ConsoleAudio: Audio
{
	public override void PlaySound(int soundId)
	{
	    // Play sound using console audio api...
	}
	
	public override void StopSound(int soundId)
	{
	    // Stop sound using console audio api...
	}
	
	public override void StopAllSounds()
	{
	    // Stop all sounds using console audio api...
	}
}

class NullAudio: Audio
{
	public override void PlaySound(int soundId) { }
	
	public override void StopSound(int soundId) { }
	
	public override void StopAllSounds() { }
}

static class Locator
{
	private static NullAudio nullAudio = new();
	public static Audio AudioService {get; private set;};

	// now can easily disable service by Provide(null)
	public static void Provide(Audio audioService)
	{
		AudioService = audioService is null ? nullAudio : audioService;
	}
}


// on game start
Audio audioService = new ConsoleAudio();
Locator.Provide(audioService);
```

## Decorated Services
```cs
class LoggedAudio : public Audio
{
	private Audio _wrapped;
	public LoggedAudio(Audio wrapped)
	{
		_wrapped = wrapped;
	}

	public virtual void PlaySound(int soundId) 
	{
		Debug.Log("Play");
		_wrapped.PlaySound(soundId); 
	}
	
	public virtual void StopSound(int soundId) 
	{
		Debug.Log("Stop");
		_wrapped.StopSound(soundId); 
	}
	
	public virtual void StopAllSounds() 
	{
		Debug.Log("Stopping all");
		_wrapped.StopAllSounds();
	}
}
```

Типы регистрации
- Внешняя, т. е. какой-то код конфигурирует службу и передает ее в локатор
- Во время компиляции `#if DEBUG`
- В рантайме при первом обращении

Для инверсии контроля есть две опции, это инъекция зависимостей и локатор.  

```cs

public interface ILocalization
{
	public string GetLocalizedWord(string key);
}


public class MockLocalization: ILocalization
{
	private readonly List<string> words = new() {
	"hund", "katt", "fisk", "bil", "hus"
	};

	private Random random = new();

	private string GetLocalizedWord(string key) => words[random.Next(words.count)];

}



public interfce ISerializer
{
	public void Serialize();
}

public class MockSerializer: ISerializer
{
	public void Serialize() 
	{
		Debug.Log("MockSerializer.Serialize");
	}
}



public interface IAudioService
{
	public void Play();
}

public class MockAudioService: IAudioService
{
	public void Play() 
	{
		Debug.Log("MockAudioService.Play");
	}
}


public interface IGameService
{
	public void StartGame();
}

public class MockGameService: IGameService
{
	public void StartGame() 
	{
		Debug.Log("MockGameService.StartGame");
	}
}

```

```cs
public class ServiceManager
{
	private readonly Dictionary<Type, object> services = new();
	public IEnumerable<object> RegisteredServices => services.Values;

	public ServiceManager Register<T>(T service)
	{
		Type type = typeof(T);

		if (!services.TryAdd(type, service))
			Debug.Log($"Service of type {type.FullName} is alreade added");
	
		return this;
	}

	public ServiceManager Register(Type type, object service)
	{
		if (!type.IsInstanceOf(service))
			throw new Exception("Type of service doesn't match type of service");
		
		if (!services.TryAdd(type, service))
			Debug.Log($"Service of type {type.FullName} is alreade added");

		return this;
	}

	public T Get<T> where T: class
	{
		Type type = typeof(T);
		if (services.TryGetValue(type, out object service))
			return service as T;
		throw new ArgumentException("Service is not registred");
	}

	public bool TryGet<T>(out T service) where T: class
	{
		Type type = typeof(T);
		if (services.TryGetValue(type, out object serviceObj))
		{
			service = serviceObj as T;
			return true;
		}

		service = null;
		return false;
	}
}

```

```cs
[DisallowMultiComponent]
[RequireComponent(typeof(ServiceLocator))]
public abstract class Bootstrapper: MonoBehaviour
{
	private ServiceLocator container;
	internal ServiceLocator Container => container.OrNull() ?? (container = GetComponent<ServiceLocator>());

	private bool hasBeenBootstrapped;

	void Awake() => BootstrapOnDemand();

	public void BootstrapOnDemand()
	{
		if (hasBeenBootstrapped) return;
		hasBeenBootstrapped = true;
		Bootstrap();
	}

	protected abstract void Bootstrap();	
}

[AddComponentMenu("ServiceLocator/ServiceLocator Global")]
public class ServiceLocatorGlobalBootstrapper: Bootstrapper
{
	[SerializeField] private bool dontDestroyOnLoad = true;

	protected override void Bootstrap()
	{
		Container.ConfigureAsGlobal(dontDestroyOnLoad)
	}
}

[AddComponentMenu("ServiceLocator/ServiceLocator Scene")]
public class ServiceLocatorSceneBootstrapper: Bootstrapper
{
	protected override void Bootstrap()
	{
		Container.ConfigureForScene()
	}
}

```

```cs

public class ServiceLocator: MonoBehaviour
{
	private static ServiceLocator globalService;
	private static Dictionary<Scene, ServiceLocator> sceneConteiners;

	const string globalServiceLocatorName = "ServiceLocator [Global]";
	const string sceneServiceLocatorName = "ServiceLocator [Scene]";

	private readonly ServiceManager services = new();

	private static List<GameObject> tmpSceneGameObjects;

	internal void ConfigureAsGlobal(bool dontDestroyOnLoad)
	{
		if (global == this)
		{
			Debug.LogWarning("Already configured as global");
		}
		else if (global is not null)
		{
			Debug.LogError("Another is already configured as global");
		}
		else
		{
			global = this;
			if (dontDestroyOnLoad) DontDestroyOnLoad(gameObject);
		}
	}

	internal void ConfigureForScene()
	{
		Scene scene = gameObject.scene;

		if (sceneContainers.ContainsKey(scene))
		{
			Debug.LogError("Another is already configured for this scene");
			return;
		}
		sceneContainers.Add(scene, this);
	}


	public static ServiceLocator Global
	{
		get
		{
			if (globalService is not null) return globalService;
			// bootstrap or init the new instance 
			if (FindFirstObjectOfType<ServiceLocatorGlobalBootstrapper>() is {} found)
			{
				found.BootstrapOnDemand();
				return globalService;
			}


			var container = new GameObject(globalServiceLocatorName, typeof(ServiceLocator));
			container.AddComponent<ServiceLocatorGlobalBootstrapper>().BootstrapOnDemand();

			return globalService;
		}
	}

	public static ServiceLocator For(MonoBehaviour mb)
	{
		return mb.GetComponent<ServiceLocator>().OrNull() ?? 
		ForSceneOf(mb) : Global;
	}


	public static ServiceLocator ForSceneOf(MonoBehaviour mb)
	{
		Scene scene = mb.gameObject.scene;
		if (sceneConteiners.TryGetValue(scene, out ServiceLocator container)
		&& container != mb)
			return container;

		tmpSceneGameObjects.Clear();
		scene.GetRootGameObjects(tmpSceneGameObjects);

		foreach (GameObject go in 
		tmpSceneGameObjects.Where(go => go.GetComponent<ServiceLocatorSceneBootstrapper>() is not null
		))
		{
			if (go.TryGetComponent(out ServiceLocatorSceneBootstrapper bootstrapper && bootstrapper != mb))
			{
				bootstrapper.BootstrapOnDemand();
				return bootstrapper.Container;			
			}
		}

		return Global;
	}

	public ServiceLocator Register<T>(T service)
	{
		services.Register(service);
		return this;
	}

	public ServiceLocator Register(Type type, object service)
	{
		services.Register(type, service);
		return this;
	}

	public ServiceLocator Get<T>(out T service) where T : class
	{
		if (TryGetService(out service)) return this;

		if (TryGetNextInHierarchy(out ServiceLocator container))
		{
			container.Get(out service);
			return this;
		}

		throw new ArgumentException("Service not registred");
	}

	private bool TryGetService<T>(out T service) where T : class
	{
		return services.TryGet(out service);
	}

	private bool TryGetNextInHierarchy(out ServiceLocator container)
	{
		if (this == globalService)
		{
			container = null;
			return false;
		}

		container = transform.parent.OrNull()?.GetComponentInParent<ServieLocator>().OrNull() ?? ForSceneOf(this);
	}

	void OnDestroy()
	{
		if (this == globalService)
		{
			globalService = null;
		}
		else if (sceneContainers.ContainsValue(this))
		{
			sceneContainers.Remove(gameObject.scene);
		}
	}	
	[RuntimeInitializeOnLoadMethod(RuntimeInitializeLoadType.SubsystemRegistration)]
	static void ResetStatics()
	{
		globalService = null;
		sceneContainers.Clear();
		tmpSceneGameObjects.Clear();
	}

	#if UNITY_EDITOR
	[MenuItem("GameObject/ServiceLocator/Add Global")]
	static void AddGlobal()
	{
		var go = new GameObject(
		globalServiceLocatorName,
		typeof(ServiceLocatorGlobalBootstrapper));
	}


	[MenuItem("GameObject/ServiceLocator/Add Scene")]
	static void AddGlobal()
	{
		var go = new GameObject(
		sceneServiceLocatorName,
		typeof(ServiceLocatorSceneBootstrapper));
	}
	#endif
}

```


```cs
class Hero: MonoBehaviour
{

	ILocalization localization;
	ISerializer serializer;
	IAudioService audioService;
	IGameService gameService;

	void Awake()
	{
		ServiceLocator.Global.Register<ILocalization>(localization = new MockLocalization());

		ServiceLocator.ForSceneOf(this).Register<IGameService>(gameService = new MockGameService());

		ServiceLocator.For(this).Register<ISerializer>(serializer = new MockSerializer());
	}

	void Start()
	{
		ServiceLocator.For(this)
		.Get(out serializer)
		.Get(out localization)
		.Get(out gameService)
		.Get(out audioService);
	}

}

```

```cs
// ANOTHER VARIANT
class Hero: MonoBehaviour
{
	[Title("Registered Services")]
	[SerializeField]
	List<Object> services;

	void Awake()
	{
		ServiceLocator sl = ServiceLocator.For(this);
		foreach (Object service in services)
		{
			sl.Register(service.GetType(), service);
		}
	}

	void Start()
	{

	}
}

```