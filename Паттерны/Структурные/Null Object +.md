Классический Null Object:

```cs
public interface IEnemy
{
    public string GetName();
    public int GetHP();
            
}

public class Goblin : IEnemy
{
    public int GetHP() => 20;
    public string GetName() => "Bob the green";
}

public class Skeleton : IEnemy
{
    public int GetHP() => 10;
    public string GetName() => "Bones the mighty";
}
```

```cs
public sealed class NullEnemy : IEnemy
{
    public int GetHP() => 0;
    public string GetName() => "Anonymous";
}
```

```cs
public class EnemyFactory
{
    public static IEnemy GetEnemy(string type)
    {
        if (type == "Goblin")
            return new Goblin();
        if(type == "Skeleton")
            return new Skeleton();

        return new NullEnemy();
    }
}
```

Можно сделать Null Object синглтоном (или единственным статическим экземпляром):

```cs
public class NullEnemySingleton : IEnemy
{
    private static readonly IEnemy instance = new NullEnemySingleton();
    private NullEnemySingleton() { }

    public static IEnemy Null => instance;

    public int GetHP() => 0;

    public string GetName() => "Anonymous";
}

public class EnemyFactory
{
    public static IEnemy GetEnemy(string type)
    {
        if (type == "Goblin")
            return new Goblin();
        if(type == "Skeleton")
            return new Skeleton();

        return NullEnemySingleton.Null;
    }
}
```

## Interface Default Methods
```cs
public interface IEnemy
{
    public string GetName();
    public int GetHP();

    public static IEnemy Null => DefaultEnemy.Null;

    private class DefaultEnemy : IEnemy
    {
        private static readonly IEnemy instance = new DefaultEnemy();
        private DefaultEnemy() { }

        public static IEnemy Null => instance;
        public int GetHP() => 0;
        public string GetName() => "Anonymous";
    }
}
```

```cs

public class EnemyFactory
{
    public static IEnemy GetEnemy(string type)
    {
        if (type == "Goblin")
            return new Goblin();
        if (type == "Skeleton")
            return new Skeleton();

        return IEnemy.Null;
    }
}
```

```cs
if (enemies[1] == IEnemy.Null) 
```