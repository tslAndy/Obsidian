#behavioral
Улучшает цепочку ответственности используя центральный объект для управления распределением сообщений, уменьшая прямые зависимости между объектами. 

```cs

[CreateAssetMenu(fileName = "BaseStats", menuName = "Stats/BaseStats")]
public class BaseStats: ScriptableObject
{
	public int attack = 10;
	public int defense = 20;
}

```

```cs

public enum StatType { Attack, Defense }


public class Stats
{
	private readonly BaseStats baseStats;
	private readonly StatsMediator mediator;

	public StatsMediator Mediator => mediator;

	public int Attack
	{
		get
		{
			var q = new Query(StatType.Attack, baseStats.attack);
			mediator.PerformQuery(this, q);
			return q.Value;
		}
	}

	public int Defense
	{
		get
		{
			var q = new Query(StatType.Defense, baseStats.defense);
			mediator.PerformQuery(this, q);
			return q.Value;
		}
	}
}
```

```cs

public class StatsMediator
{
	private readonly LinkedList<StatModifier> modifiers = new();

	public event EventHandler<Query> Queries;
	public void PerformQuery(object sender, Query query) => Queries?.Invoke(sender, query);

	public void AddModifier(StatModifier modifier)
	{
		modifiers.AddLast(modifier);
		Queries += modifier.Handle;

		modifier.OnDispose += _ => {
			modifiers.Remove(modifier);
			Queries -= modifier.Handle;
		};
	}

	public void Update(float deltaTime)
	{
		var node = modifiers.First;
		while (node is not null)
		{
			var modifier = node.Value;
			modifier.Update(deltaTime);
			node = node.Next;
		}

		node = modifiers.First;
		while (node is not null)
		{
			var nextNode = node.Next;
			if (node.Value.MarkedForRemoval)
			{
				node.Value.Dispose();
			}

			node = nextNode;
		}
	}
	
}

```

```cs
public class Query
{
	public readonly StatType statType;
	public int val;

	public class Query(StatType statType, int val)
	{
		this.statType = statType;
		this.val = val;
	}
}
```

```cs
public abstract class StatModifier: IDisposable
{
	public bool MarkedForRemoval { get; private set; }

	pubilc event Action<StatModifier> OnDispose = delegate { };

	public abstract void Handle(object sender, Query query);

	private readonly CountdownTimer timer;

	protected StatsModifier(float duration)
	{
		if (duration <= 0) return;
		timer = new CountdownTimer(duration);
		timer.OnTimerStop += () => MarkedForRemoval = true;
		timer.Start();
	}

	public void Update(float deltaTime) => timer?.Tick(deltaTime);


	public void Dispose()
	{
		OnDispose.Invoke(this);
	}
}
```


```cs
public class BasicStatModifier: StatModifier
{
	private readonly StatType type;
	private readonly Func<int, int> operation;

	public BasicStatModifier(StatType type, float duration, Func<int, int> operation) : base(duration)
	{
		this.type = type;
		this.operation = operation;
	}

	public override void Handle(object sender, Query query)
	{
		if (query.statType == type)
		{
			query.Value = operation(query.Value);
		}
	}
}
```

```cs
public abstract class Pickup: MonoBehaviour, IVisitor
{
	protected abstract void ApplyPickupEffect(Entity entity);

	public void Visit<T>(T visitable) where T: Component, IVisitable
	{
		if (visitable is Entity entity)
		{
			ApplyPickupEffect(entity);
		}
	}

	public void OnTriggerEnter(Collider other)
	{
		other.GetComponent<IVisitable>()?.Accept(this);
		Destroy(gameObject);
	}
}
```

```cs
public class StatModifierPickup: Pickup
{
	public enum OperationType { Add, Multiply }

	[SerializeField] StatType statType = StatType.Attack;
	[SerializeField] OperationType operationType = OperationType.Add;
	[SerializeField] int value = 10;
	[SerializeField] float duration = 5f;

	protected override void ApplyPickupEffect(Entity entity)
	{
		StatModifier modifier = operationType switch
		{
			OperationType.Add => new BasicStatModifier(
			statType,
			duration,
			v => v + value),

			OperationType.Multiply => new BasicStatModifier(
			statType,
			duration,
			v => v * value),

			_ => throw new ArgumentOutOfRangeException()
		};

		entity.Stats.Mediator.AddModifier(modifier);
	}
}
```


```cs
public abstract class Entity: MonoBehaviour, IVisitable
{
	[SerializeField, Required] BaseStats baseStats;
	public Stats stats { get; private set; }

	void Awake()
	{
		stats = new Stats(new StatsMediator(), baseStats);
	}

	void Update()
	{
		stats.Mediator.Update(Time.deltaTime);
	}

	public void Acccept(IVisitor visitor) => visitor.Visit(this);

	public void Equip(IEquipable equipable)
	{
		stats.Mediator.AddModifier(equipable.GetStatModifier());	
	}

	public void Unequip(IEquipable equipable)
	{
		equipable.GetStatModifier().MarkedForRemoval = true;	
	}
}

```


```cs
public interface IEquipable
{
	public StatModifier GetStatModifier();
}

```