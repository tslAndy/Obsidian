Часто можно столкнуться с ситуацией, например, когда пуля попадая куда-то должна воспроизвести звук, но из-за того, что пуля уничтожается раньше, звук пропадает.
Использование AudioSource.PlayClipAtPoint так же не решает проблему, потому что он создает новый объект при каждом вызове.

Так же важной является настройка проекта, в которой определяем, сколько звуков можно одновременно услышать в игре. Допустим, что у нас тысячи звуков вызовов, как при этом сделать так, чтобы звуки UI и музыка были слышны?

![[Screenshot_20240816_150449.png]]

```cs
[Serializable]
public class SoundData
{
    public AudioClip clip;
    public AudioMixerGroup mixerGroup;
    public bool loop;
    public bool playOnAwake;
    public bool frequentSound;
}

```

```cs
public class SoundEmitter : MonoBehaviour
{
    public SoundData Data { get; private set; }
    AudioSource audioSource;
    Coroutine playingCoroutine;

    void Awake()
    {
        audioSource = gameObject.GetOrAdd<AudioSource>();
    }

    public void Initialize(SoundData data)
    {
        Data = data;
        audioSource.clip = data.clip;
        audioSource.outputAudioMixerGroup = data.mixerGroup;
        audioSource.loop = data.loop;
        audioSource.playOnAwake = data.playOnAwake;
    }

    public void Play()
    {
        if (playingCoroutine is not null)
        {
            StopCoroutine(playingCoroutine);
        }

        audioSource.Play();
        playingCoroutine = StartCoroutine(WaitForSoundToEnd());
    }

    IEnumerator WaitForSoundToEnd()
    {
        yield return new WaitWhile(() => audioSource.isPlaying);
        SoundManager.Instance.ReturnToPool(this);
    }

    public void Stop()
    {
        if (playingCoroutine is not null)
        {
            StopCoroutine(playingCoroutine);
            playingCoroutine = null;
        }

        audioSource.Stop();
        SoundManager.Instance.ReturnToPool(this);
    }

    public void WithRandomPitch(float min = -0.05f, float max = 0.05f)
    {
        audioSource.pitch += Random.Range(min, max);
    }
}

```

```cs
public class SoundManager : PersistentSingleton<MonoBehaviour>
{
    IObjectPool<SoundEmitter> soundEmitterPool;
    readonly List<SoundEmitter> activeSoundEmitters = new();
    public readonly Queue<SoundEmitter> FrequentSoundEmitters = new();

    [SerializeField] SoundEmitter soundEmitterPrefab;
    [SerializeField] bool collectionCheck = true;
    [SerializeField] int defaultCapacity = 10;
    [SerializeField] int maxPoolSize = 100;
    [SerializeField] int maxSoundInstances = 30;

    private void Start()
    {
        InitializePool();
    }

    public SoundBuilder CreateSound() => new SoundBuilder(this);

    public SoundEmitter Get() => soundEmitterPool.Get();

    public bool CanPlaySound(SoundData data)
    {
        if (!data.frequentSound) return true;
        if (FrequentSoundEmitters.Count >= maxSoundInstances && FrequentSoundEmitters.TryDequeue(out var soundEmitter))
        {
            try
            {
                soundEmitter.Stop();
                return true;
            }
            catch
            {
                Debug.Log("SoundEmitter is already released");
            }
            return false;
        }
        return true;
    }

    public void ReturnToPool(SoundEmitter soundEmitter)
    {
        soundEmitterPool.Release(soundEmitter);
    }

    private void SoundEmitter CreateSoundEmitter()
    {
        var soundEmitter = Instantiate(soundEmitterPrefab);
        soundEmitter.gameObject.SetActive(false);
        return soundEmitter;
    }

    private void OnTakeFromPool(SoundEmitter soundEmitter)
    {
        soundEmitter.gameObject.SetActive(true);
        activeSoundEmitters.Add(soundEmitter);
    }

    private void OnReturnedToPool(SoundEmitter soundEmitter)
    {
        soundEmitter.gameObject.SetActive(false);
        activeSoundEmitters.Remove(soundEmitter);
    }

    private void OnDestroyPoolObject(SoundEmitter soundEmitter)
    {
        soundEmitter.gameObject.SetActive(false);
        activeSoundEmitters.Remove(soundEmitter);
    }

    private void InitializePool()
    {
        soundEmitterPrefab = new ObjectPool<SoundEmitter>(
            CreateSoundEmitter,
            OnTakeFromPool,
            OnReturnedToPool,
            OnDestroyPoolObject,
            collectionCheck,
            defaultCapacity,
            maxPoolSize
        );
    }
}
```

```cs
// Usage, f. e. on collision
SoundManager.Instance.CreateSound()
    .WithSoundData(soundData)
    .WithRandomPitch()
    .WithPosition(fx.transform.position)
    .Play();
```