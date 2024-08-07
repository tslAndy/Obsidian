#decoupling
```cs
public abstract class EventChannel<T> : ScriptableObject
{
    private readonly HashSet<EventListener<T>> observers = new();

    public void Invoke(T value)
    {
        foreach (var observer in observers)
            observer.Raise(value);
    }

    public void Register(EventListener<T> observer) => observers.Add(observer);
    public void Deregister(EventListener<T> observer) => observers.Remove(observer);
}
```


```cs
[CreateAssetMenu(fileName = "Float Event Channel")]
public class FloatEventChannel : EventChannel<float> { }

[CreateAssetMenu(fileName = "Int Event Channel")]
public class IntEventChannel : EventChannel<int> { }


public readonly struct Empty { }

[CreateAssetMenu(fileName = "Empty Event Channel")]
public class EmptyEventChannel : EventChannel<Empty> { }
```


```cs
public abstract class EventListener<T> : MonoBehaviour
{
    [SerializeField] private EventChannel<T> eventChannel;
    [SerializeField] private UnityEvent<T> unityEvent;

    protected void Awake() => eventChannel.Register(this);
    protected void OnDestroy() => eventChannel.Deregister(this);

    public void Raise(T value) => unityEvent.Invoke(value);
}
```


```cs
public class EmptyEventListener: EventListener<Empty> { }
public class FloatEventListener: EventListener<float> { }
public class IntEventListener: EventListener<int> { }
```
