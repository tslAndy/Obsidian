#oop

Для того, чтобы выбрать, что лучше использовать, нужно понять, какое [[Отношения между объектами +]]

- Нужно использовать наследование, если между классами отношение "is a", и подкласс - более специфическая версия базового класса.
- Композиция при отношении "has a", и можно комбинировать поведение и свойства.

При необходимости можно залить код в ChatGPT и попросить сгенерировать uml, потом зайти на plantuml.com и посмотреть что получилось

```cs
public enum ResourceType
{
    Empty,
    Stone,
    Oak,
    Birch,
    Pine,
    Iron,
    Coal,

    Gold,
    Diamond
}

public abstract class BaseNode
{
    public abstract ResourceType Gather();
}

public class StoneNode : BaseNode
{
    public override ResourceType Gather() => ResourceType.Stone;
}

public class OakNode : BaseNode
{
    public override ResourceType Gather() => ResourceType.Oak;
}

/*
Предположим, что мы захотели специальгый ресурс, который может содержать два типа. 

При подходе с наследование ситуация быстро выйдет из под контроля
*/


public abstract class SpecialNode: BaseNode
{
    public override ResourceType Gather() => ResourceType.Empty;
    public abstract ResourceType SpecialGather();
}

public class IronNode : SpecialNode
{

    public override ResourceType Gather() => ResourceType.Stone;
    public override ResourceType SpecialGather() => ResourceType.Iron;
}

```

Общего между классами - они все узлы, и различаются они типом, т. е. каждый узел содержит тип:
```cs
public abstract class BaseNode
{
    public ResourceType resourceType;
    public abstract ResourceType Gather();
}


public abstract class SpecialNode: BaseNode
{
    public ResourceType specialResourceType;
    public abstract ResourceType GatherSpecial();
}
```

Здесь создание класса `SpecialNode` могло показаться хорошей идеей, но это не так, потому что клиенту придется использовать кастинг чтобы понять, с каким типом он работает. 

Можно скомбинировать классы:
```cs
public abstract class BaseNode
{
    public ResourceType resourceType;
    public ResourceType specialResourceType;
 
    public abstract ResourceType Gather();
    public abstract ResourceType GatherSpecial();
}
```

Если еще подумать, то класс Node содержит не тип ресурса, а сам ресурс:

```cs

// not abstract anymore
public class BaseNode
{
    [SerializeField] private ResourceConfig resourceConfig;
    [SerializeField] private ResourceConfig specialResourceConfig;

    private Resource baseResource, specialResource;
    
    protected void Awake()
    {
        baseResource = resourceConfig.Generate();
        specialResource = specialResourceConfig.Generate();
    }

	// tuple is better than two separate methods
	// given names to help understand what is what
	// and we can reference to them as . notation
	/*
		result = Gather();
		result.baseRes;
		result.specialRes;
	*/
    public (Resource baseRes, Resource specialRes) Gether() => (baseResource, specialResource);

}

// create menu item etc.
public class ResourceConfig: ScriptableObject
{
    public ResourceType resourceType;
    public int amount;

    public Resource Generate() => new Resource()
    {
        resourceType = resourceType,
        amount = amount
    };
}

public class Resource
{
    public ResourceType resourceType;
    public int amount;
}
```

