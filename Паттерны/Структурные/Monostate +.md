#structural
Monostate allows multiple instances of a class to share the same state by channeling their state management through a common shared structure. This ensures consistent state across all instances while maintaining the facade of independent objects.

```cs
public class LoadBalancer {
    private static List<Server> servers = new();
    private static int nextServerIndex = 0;

    public LoadBalancer() {
        // Initialize servers
        servers.add(new Server("192.168.0.1", 8080, 1));
        servers.add(new Server("192.168.0.2", 8080, 2));
        servers.add(new Server("192.168.0.3", 8080, 3));
    }

    public void serverRequest(Request request) {
        Server server = servers.get(nextServerIndex);
        server.serve(request);
        nextServerIndex = (nextServerIndex + 1) % servers.size();
    }
}

```


Другой пример:

```cs
public class Monostate
{
   private static int _level;
   private static float _damage;

   public int Level
   {
      get => _level;
      set => _level = value;
   }
   public float Damage
   {
      get => _damage;
      set => _damage = value;
   }
}
```

С интерфейсом по умолчанию:

```cs
public interface IMonostate
{
   protected static int _level;
   protected static float _damage;

   public int Level
   {
      get => _level;
      set => _level = value;
   }
   public float Damage
   {
      get => _damage;
      set => _damage = value;
   }
}
```