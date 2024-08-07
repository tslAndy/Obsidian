#other 

```cs
public interface IMyUpdate
{
   void MyUpdate();
}
```

```cs
public class UpdateManager : MonoBehaviour
{
   private readonly HashSet<IMyUpdate> myMod3Updates = new();

   public void AddMod3Updatable(IMyUpdate myUpdate) => myMod3Updates.Add(myUpdate);
   public void RemoveMod3Updatable(IMyUpdate myUpdate) => myMod3Updates.Remove(myUpdate);

   private void RunMod3Updates()
   {
      if (Time.frameCount % 3 == 0)
      {
         using var e = myMod3Updates.GetEnumerator();
         while (e.MoveNext())
            e.Current?.MyUpdate();
      }
   }

   private void Update()
   {
      RunMod3Updates();
   }
}
```

```cs
public class MyMod3Update : MonoBehaviour, IMyUpdate
{
   [SerializeField] private UpdateManager updateManager;

   private void OnEnable() => updateManager.AddMod3Updatable(this);

   void IMyUpdate.MyUpdate()
   {
      // our update code here
   }

   private void OnDisable() => updateManager.RemoveMod3Updatable(this);
}
public class UpdateManager : MonoBehaviour
{
   private readonly HashSet<IMyUpdate> myUpdates = new();
   private readonly HashSet<IMyUpdate> myMod3Updates = new();

   public void AddMod4Updatable(IMyUpdate myUpdate) => myUpdates.Add(myUpdate);
   public void RemoveMod4Updatable(IMyUpdate myUpdate) => myUpdates.Remove(myUpdate);

   public void AddMod3Updatable(IMyUpdate myUpdate) => myMod3Updates.Add(myUpdate);
   public void RemoveMod3Updatable(IMyUpdate myUpdate) => myMod3Updates.Remove(myUpdate);

   private void RunMod4Updates()
   {
      if (Time.frameCount % 4 == 0)
      {
         using var e = myUpdates.GetEnumerator();
         while (e.MoveNext())
            e.Current?.MyUpdate();
      }
   }

   private void RunMod3Updates()
   {
      if (Time.frameCount % 3 == 0)
      {
         using var e = myMod3Updates.GetEnumerator();
         while (e.MoveNext())
            e.Current?.MyUpdate();
      }
   }

   private void Update()
   {
      RunMod4Updates();
      RunMod3Updates();
   }
}
```

При необходимости нескольких условий:

```cs
public class UpdateManager : MonoBehaviour
{
   private readonly HashSet<IMyUpdate> myUpdates = new();
   private readonly HashSet<IMyUpdate> myMod3Updates = new();

   public void AddMod4Updatable(IMyUpdate myUpdate) => myUpdates.Add(myUpdate);
   public void RemoveMod4Updatable(IMyUpdate myUpdate) => myUpdates.Remove(myUpdate);

   public void AddMod3Updatable(IMyUpdate myUpdate) => myMod3Updates.Add(myUpdate);
   public void RemoveMod3Updatable(IMyUpdate myUpdate) => myMod3Updates.Remove(myUpdate);

   private void RunMod4Updates()
   {
      if (Time.frameCount % 4 == 0)
      {
         using var e = myUpdates.GetEnumerator();
         while (e.MoveNext())
            e.Current?.MyUpdate();
      }
   }

   private void RunMod3Updates()
   {
      if (Time.frameCount % 3 == 0)
      {
         using var e = myMod3Updates.GetEnumerator();
         while (e.MoveNext())
            e.Current?.MyUpdate();
      }
   }

   private void Update()
   {
      RunMod4Updates();
      RunMod3Updates();
   }
}
```

При необходимости скрипт может регистрироваться или отменять регистрацию при каких-то условиях:

```cs
public class WithMyUpdate : MonoBehaviour, IMyUpdate
{
    [SerializeField] private UpdateManager updateManager;

    void IMyUpdate.MyUpdate()
    {
        // our update code here
    }
    
    private void OnDisable() => updateManager.RemoveMod4Updatable(this);
    
    public void StartRunningUpdate() => updateManager.AddMod4Updatable(this);
    public void StopRunningUpdate() => updateManager.RemoveMod4Updatable(this);
}

```
