#other 
Предположим, что проходимся по компонентам для их обновления, и один из компонентов содержит данные, которые редко нужны (например только при смерти врага). Нам не нужны эти данные в основном цикле, так как они будут занимать место в кэше. 

```cs
class AIComponent
{
    public void Update() { }

    // all these we can move to separate class
    private LootType lootType;
    private int minDrops;
    private int maxDrops;
    private float chanceOfDrop;
}
```

```cs
class AIComponent
{
    public void Update() { }
    
    private LootDrop lootDrop;
}

class LootDrop
{
    private LootType lootType;
    private int minDrops;
    private int maxDrops;
    private float chanceOfDrop;    
}
```