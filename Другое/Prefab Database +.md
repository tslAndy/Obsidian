#other 
```cs
using UnityEngine;

[CreateAssetMenu(
menuName = nameof(PrefabsDatabase), 
fileName = nameof(PrefabsDatabase))]
public sealed class PrefabsDatabase : ScriptableObject
{
   public SpriteRenderer IsometricDiamond;
   public ParticleSystem Particles;
}
```

```cs
using UnityEngine;

// Create once with the name CurrentPrefabDatabase and add it to the Resources folder
[CreateAssetMenu(
menuName = nameof(CurrentPrefabDatabase), 
fileName = nameof(CurrentPrefabDatabase))]
public sealed class CurrentPrefabDatabase : ScriptableObject
{
    public static CurrentPrefabDatabase Instance => 
        _instance ??= 
        Resources.Load<CurrentPrefabDatabase>(nameof(CurrentPrefabDatabase));

    public  PrefabsDatabase CurrentDataBase;
    private static CurrentPrefabDatabase _instance;
}
```

```cs

public static class PrefabsFacade
{
   public static PrefabsDatabase Prefabs => CurrentPrefabDatabase.Instance.CurrentDataBase;
}
```

Вместо этого:

```cs
using UnityEngine;

public class Capsule : MonoBehaviour
{
   [SerializeField] private SpriteRenderer isometricDiamond;
   [SerializeField] private ParticleSystem particles;
    
   private SpriteRenderer _go;
   private ParticleSystem _go2;

   private void Start()
    {
       _go = Instantiate(isometricDiamond, transform);
       _go2 = Instantiate(particles, transform);
    }
}
```

Используем это:

```cs
using UnityEngine;
using static PrefabsFacade;

public class Capsule : MonoBehaviour
{
   private SpriteRenderer _go;
   private ParticleSystem _go2;


   private void Start()
    {
       _go = Instantiate(Prefabs.IsometricDiamond, transform);
       _go2 = Instantiate(Prefabs.Particles, transform);    

       _go2.Play();
    }
}
```

Для более сложных сценариев лучшим решением будет [[Addressables]], но для быстрого прототипирования и небольших игр использование этой техники с папкой Resources может сэкономить нам много времени на поиск и замену ссылок на префабы по всему коду.