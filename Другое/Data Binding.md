```cs
public interface ISerializer
{
    string Serialize<T>(T obj);
    T Deserialize<T>(string json);
}

public class JsonSeriazlier : ISerializer
{
    public T Deserialize<T>(string json)
    {
        return JsonUtility.FromJson<T>(json);

    }

    public string Serialize<T>(T obj)
    {
        return JsonUtility.ToJson(obj, true);
    }
}
```

```cs
public interface IDataService
{
    void Save(GameData data, bool overwrite = true);
    GameData Load(string name);
    void Delete(string name);
    void DeleteAll();
    IEnumerable<string> ListSaves();
}

public class FileDataService : IDataService
{
    ISerializer serializer;
    string dataPath;
    string fileExtension;

    public FileDataService(ISerializer serializer)
    {
        this.serializer = serializer;
        this.dataPath = Application.persistentDataPath;
        this.fileExtension = "json";
    }

    string GetPathToFile(string filename)
    {
        return Path.Combine(dataPath, string.Concat(filename, '.', fileExtension));
    }


    public void Delete(string name)
    {
        string fileLocation = GetPathToFile(name);
        if (File.Exists(fileLocation))
        {
            File.Delete(fileLocation);
        }
    }

    public void DeleteAll()
    {
        foreach (string filePath in Directory.GetFiles(dataPath))
        {
            File.Delete(filePath);
        }
    }

    public IEnumerable<string> ListSaves()
    {
        foreach (string path in Directory.EnumerateFiles(dataPath))
        {
            if (Path.GetExtension(path) == fileExtension)
            {
                yield return Path.GetFileNameWithoutExtension(path);
            }
        }
    }

    public GameData Load(string name)
    {
        string fileLocation = GetPathToFile(name);
        if (!File.Exists(fileLocation))
        {
            throw new ArgumentException($"No persisted GameData with name '{name}'");
        }
        return serializer.Deserialize<GameData>(File.ReadAllText(fileLocation));
    }

    public void Save(GameData data, bool overwrite = true)
    {
        string fileLocation = GetPathToFile(data.Name);
        if (!overwrite && File.Exists(fileLocation))
        {
            throw new IOException($"The file '{data.Name}.{fileExtension} already exists and can not be overwritten' ");
        }

        File.WriteAllText(fileLocation, serializer.Serialize(data));
    }
}
```

```cs
[Serializable]
public class GameData
{
    public string Name;
    public string CurrentLevelGame;
    public PlayerData playerData;
    public InventoryData inventoryData;
}

```

```cs
public class SaveLoadSystem : PersistentSingleton<SaveLoadSystem>
{
    [SerializeField]
    public GameData gameData:

    IDataService dataService;

    protected override void Awake()
    {
        base.Awake();
        dataService = new FileDataService(new JsonSeriazlier());
    }

    void OnEnable() => SceneManager.sceneLoaded += OnSceneLoaded;
    void OnDisable() => SceneManager.sceneLoaded -= OnSceneLoaded;

    void OnSceneLoaded(Scene scene, LoadSceneMode mode)
    {
        if (scene.name == "Menu") return;
        Bind<Hero, PlayerData>(gameData.playerData);
        Bind<Inventory, InventoryData>(gameData.inventoryData);
    }

    void Bind<T, TData>(TData data) where T : MonoBehaviour, IBind<TData> where TData : ISaveable, new()
    {
        var entity = FindObjectsByType<T>(FindObjectsSortMode.None).FirstOrDefault();
        if (entity != null)
        {
            if (data == null)
            {
                data = new TData { Id = entity.Id };
            }
            entity.Bind(data);
        }
    }

    void Bind<T, TData>(List<TData> datas) where T : MonoBehaviour, IBind<TData> where TData : ISaveable, new()
    {
        var entities = FindObjectsByType<T>(FindObjectsSortMode.None);
        foreach (var entity in entities)
        {
            var data = datas.FirstOrDefault(d => d.Id == entity.Id);
            if (data == null)
            {
                data = new TData { Id = entity.Id };
                datas.Add(data);
            }
            entity.Bind(data);

        }
    }


    public void NewGame()
    {
        gameData = new GameData
        {
            Name = "New Game",
            CurrentLevelGame = "Demo"
        };

        SceneManeger.LoadScene(gameData.CurrentLevelGame);
    }

    public void SaveGame()
    {
        dataService.Save(gameData);
    }

    public void LoadGame(string gameName)
    {
        gameData = dataService.Load(gameName);

        if (String.IsNullOrWhiteSpace(gameData.CurrentLevelGame))
        {
            gameData.CurrentLevelGame = "Demo";
        }

        SceneManeger.LoadScene(gameData.CurrentLevelGame);
    }


    public void DeletGame(string gameName)
    {
        dataService.Delete(gameName);
    }
}
```

```cs
[CustomEditor(typeof(SaveLoadSystem))]
public class SaveManagerEditor : UnityEditor.Editor
{
    public override void OnInspectorGUI()
    {
        SaveLoadSystem saveLoadSystem = (SaveLoadSystem)target;
        string gameName = saveLoadSystem.gameData.Name;

        DrawDefaultInspector();

        if (GUILayout.Button("Save Game"))
        {
            saveLoadSystem.SaveGame();
        }

        if (GUILayout.Button("Load Game"))
        {
            saveLoadSystem.LoadGame(gameName);
        }

        if (GUILayout.Button("Delete Game"))
        {
            saveLoadSystem.DeletGame(gameName);
        }
    }
}
```

```cs
public interface ISaveable
{
    SerializableGuid Id { get; init; }
}

public interface IBind<TData> where TData : ISaveable
{
    SerializableGuid Id { get; set; }
    void Bind(TData data);
}

// ----------------
public class Hero : MonoBehaviour, IBind<PlayerData>
{
    [field: SerializeField] public SerializableGuid Id { get; set; } = SerializableGuid.NewGuid();
    [SerializeField] PlayerData data;

    public void Bind(PlayerData data)
    {
        this.data = data;
        this.data.Id = Id;
        transform.position = data.position;
        transform.rotation = data.rotation;
    }

    void Update()
    {
        data.position = transform.position;
        data.rotation = transform.rotation;
    }
}


[Serializable]
public class PlayerData : ISaveable
{
    [field: SerializeField] public SerializableGuid Id { get; set; }
    public Vector3 position;
    public Quaternion rotation;
}
// ----------------


[Serializable]
public class InventoryData : ISaveable
{
    [field: SerializeField] public SerializableGuid Id { get; set; }
    public Item[] Items;
    public int Capacity;
    public int Coins;
}

public class Inventory : MonoBehaviour, IBind<InventoryData>
{
    // other fields
    [field: SerializeField] public SerializableGuid Id { get; set; } = SerializableGuid.NewGuid();

    public void Bind(InventoryData data)
    {
        // save
    }
}
```