```cs
[Serializable]
public class SceneGroup
{
    public string GroupName = "New Scene Group";
    public List<SceneData> Scenes;

    public string FindSceneNameByType(SceneType sceneType)
    {
        return Scenes.FirstOrDefault(scene => scene.SceneType == sceneType)?.Reference.Name;
    }
}

[Serializable]
public class SceneData
{
    public SceneReference Reference;
    public string Name => Reference.Name;
    public SceneType SceneType;
}

public enum SceneType { ActiveScene, MainMenu, UserInterface, HUD, Cinematic, Environment, Tooling }


public class SceneGroupManager
{
    public event Action<string> OnSceneLoaded = delegate { };
    public event Action<string> OnSceneUnloaded = delegate { };
    public event Action OnSceneGroupLoaded = delegate { };

    readonly AsyncOperationHandleGroup handleGroup = new(10);

    SceneGroup ActiveSceneGroup;

    public async Task LoadScenes(SceneGroup group, IProgress<float> progress, bool reloadDupScenes = false)
    {
        ActiveSceneGroup = group;
        var loadedScenes = new List<string>();

        await UnloadScenes();

        int sceneCount = SceneManager.sceneCount;
        for (var i = 0; i < sceneCount; i++)
        {
            loadedScenes.Add(SceneManager.GetSceneAt(i).name);
        }

        var totalScenesToLoad = ActiveSceneGroup.Scenes.Count;
        var operationGroup = new AsyncOperationGroup(totalScenesToLoad);
        for (var i = 0; i < totalScenesToLoad; i++)
        {
            var sceneData = group.Scenes[i];
            if (reloadDupScenes == false && loadedScenes.Contains(sceneData.Name)) continue;

            if (sceneData.Reference.State == SceneReferenceState.Regular)
            {
                var operation = SceneManager.LoadSceneAsync(sceneData.Reference.Path, LoadSceneMode.Additive);
                operationGroup.Operations.Add(operation);
            }

            else if (sceneData.Reference.State == SceneReferenceState.Addressable)
            {
                var sceneHandle = Addressables.LoadSceneAsync(sceneData.Reference.Path, LoadSceneMode.Additive);
                handleGroup.Handles.Add(sceneHandle);
            }


            OnSceneLoaded.Invoke(sceneData.Name);
        }

        while (!operationGroup.IsDone || !handleGroup.IsDone)
        {
            progress?.Report((operationGroup.Progress + handleGroup.Progress) / 2);
            await Task.Delay(100);
        }

        Scene activeScene = SceneManager.GetSceneByName(ActiveSceneGroup.FindSceneNameByType(SceneType.ActiveScene));
        if (activeScene.IsValid())
        {
            SceneManager.SetActiveScene(activeScene);
        }

        OnSceneGroupLoaded.Invoke();
    }

    public async Task UnloadScenes()
    {
        var scenes = new List<string>();
        var activeScene = SceneManager.GetActiveScene().name;
        int sceneCount = SceneManager.sceneCount;

        for (var i = sceneCount - 1; i > 0; i--)
        {
            var sceneAt = SceneManager.GetSceneAt(i);
            if (!sceneAt.isLoaded) continue;

            string sceneName = sceneAt.name;
            if (sceneName.Equals(activeScene) || sceneName == "Bootstrapper") continue;

            if (handleGroup.Handles.Any(
                h => h.IsValid() && h.Result.Scene.name == sceneName
            )) continue;

            scenes.Add(sceneName);
        }

        var operationGroup = new AsyncOperationGroup(scenes.Count);
        foreach (var scene in scenes)
        {
            var operation = SceneManager.UnloadSceneAsync(scene);
            if (operation == null) continue;
            operationGroup.Operations.Add(operation);
            OnSceneUnloaded.Invoke(scene);
        }

        foreach (var handle in handleGroup.Handles)
        {
            if (handle.IsValid())
            {
                Addressables.UnloadSceneAsync(handle);
            }
        }
        handleGroup.Handles.Clear();

        while (!operationGroup.IsDone)
        {
            await Task.Delay(100);
        }

        // optional
        await Resources.UnloadUnusedAssets();
    }
}

public readonly struct AsyncOperationGroup
{
    public readonly List<AsyncOperation> Operations;

    public float Progress => Operations.Count == 0 ? 0 : Operations.Average(o => o.progress);
    public bool IsDone => Operations.All(o => o.isDone);

    public AsyncOperationGroup(int initialCapacity)
    {
        Operations = new List<AsyncOperation>(initialCapacity);
    }
}

public readonly struct AsyncOperationHandleGroup
{
    public readonly List<AsyncOperationHandle<SceneInstance>> Handles;
    public float Progress => Handles.Count == 0 ? 0 : Handles.Average(h => h.PercentComplete);
    public bool IsDone => Handles.Count == 0 || Handles.All(o => o.isDone);

    public AsyncOperationHandleGroup(int initialCapacity)
    {
        Handles = new List<AsyncOperationHandle<SceneInstance>>(initialCapacity);
    }
}

public class LoadingProgress : IProgress<float>
{
    public event Action<float> Progressed;

    const float ratio = 1f;

    public void Report(float value)
    {
        Progressed?.Invoke(value / ratio);
    }
}


public class SceneLoader
{
    [SerializeField] Image loadingBar;
    [SerializeField] float fillSpeed = 0.5f;
    [SerializeField] Canvas loadingCanvas;
    [SerializeField] Camera loadingCamera;
    [SerializeField] SceneGroup[] sceneGroups;

    float targetProgress;
    bool isLoading;

    public readonly SceneGroupManager manager = new();

    void Awake()
    {
        manager.OnSceneLoaded += sceneName => Debug.Log("Loaded: " + sceneName);
        manager.OnSceneUnloaded += sceneName => Debug.Log("Unloaded: " + sceneName);
        manager.OnSceneGroupLoaded += () => Debug.Log("Scene group loaded");
    }

    async void Start()
    {
        await LoadSceneGroup(0);
    }

    void Update()
    {
        if (!isLoading) return;
        float currentFilAmount = loadingBar.fillAmount;
        float progressDifference = Mathf.Abs(currentFilAmount - targetProgress);

        float dynamicFillSpeed = progressDifference * fillSpeed;
        loadingBar.fillAmount = Mathf.Lerp(currentFilAmount, targetProgress, Time.deltaTime * dynamicFillSpeed);

    }

    public async Task LoadSceneGroup(int index)
    {
        loadingBar.fillAmount = 0f;
        targetProgress = 1f;

        if (index < 0 || index >= sceneGroups.Length)
        {
            Debug.LogError("Invalid scene group index: " + index);
            return;
        }

        LoadingProgress progress = new();
        progress.Progressed += target => targetProgress = Mathf.Max(target, targetProgress);

        EnableLoadingCanvas();
        await manager.LoadScenes(sceneGroups[index], progress);
        EnableLoadingCanvas(false);
    }

    void EnableLoadingCanvas(bool enable = true)
    {
        isLoading = enable;
        loadingCanvas.gameObject.SetActive(enable);
        loadingCamera.gameObject.SetActive(enable);
    }
}

public class Bootstrapper : PerisistentSingleton<Bootstrapper>
{
    [RuntimeInitializeOnLoadMethod(RuntimeInitializeLoadType.BeforeSceneLoad)]
    static async void Init()
    {
        Debug.Log("Bootstrapper...");
        await SceneManager.LoadSceneAsync("Bootstrapper", LoadSceneMode.Single);
    }
}
```