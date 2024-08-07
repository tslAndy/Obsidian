#otherpatterns
```cs

using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Threading.Tasks;
using UnityEngine;



[Serializable]
public class Base
{
    public string id;
}

[Serializable]
public class Item : Base
{
    public string name;
    public string description;
    public int price;
}


[Serializable]
public class Shop : Base
{
    public string name;
    public int gold;
    public List<Item> products;
}


[Serializable]
public class Player : Base
{
    public int gold;
    public List<Item> inventory;
}

// transRepository pattern used to decouple game code from the data access layers
// CRUD operations

[Serializable]
public class GameData
{
    public List<Player> players;
    public List<Shop> shops;
}

[Serializable]
public abstract class DataContext
{
    public GameData data = new();

    public abstract Task Load();
    public abstract Task Save();

    public List<T> Set<T>()
    {
        if (typeof(T) == typeof(Player))
            return data.players as List<T>;

        if (typeof(T) == typeof(Shop))
            return data.shops as List<T>;

        return null;
    }
}

[Serializable]
public class JsonDataContext : DataContext
{
    public string fileName;
    private string FilePath => Path.Combine(Application.dataPath, fileName);

    public override async Task Load()
    {
        if (!File.Exists(FilePath)) return;
        using var reader = new StreamReader(FilePath);
        var json = await reader.ReadToEndAsync();
        JsonUtility.FromJsonOverwrite(json, data);
    }

    public override async Task Save()
    {
        var json = JsonUtility.ToJson(data);
        using var writer = new StreamWriter(FilePath);
        await writer.WriteAsync(json);
    }
}


public abstract class Repository<T> : MonoBehaviour where T : Base
{
    [HideInInspector] public DataContext context;

    private List<T> Entities => context.Set<T>();

    public T GetById(string id) => Entities.FirstOrDefault(e => e.id == id);

    public void Add(T entity) { }
    public void Modify(T entity) { }
    public void Delete(T entity) { }

    public async Task Save()
    {
        await context.Save();
    }
}

[Serializable]
public class Shops: Repository<Shop>
{
    
}

[Serializable]
public class Players: Repository<Player>
{
    
}


// unit of work contains a list of objects 
// affected by a business transaction
// and coordinates the writing out of changes 
// and the resolution of concurrency problems



public class UnitOfWork: MonoBehaviour
{
    [SerializeField] private DataContext dataContext;
    [SerializeField] private Players players;
    [SerializeField] private Shops shops;

    public Players Players => players;
    public Shops Shops => shops;

    public async void Save() => await dataContext.Save();
}

// instead of 
player.gold -= item.price;
players.Modify(player);
players.Save();

// we use
unitOfWork.Players.Modify(player);
unitOfWork.players.Save();

```