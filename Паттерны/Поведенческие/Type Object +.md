#behavioral
```cs
public class Monster
{
    public class Breed
    {
        public readonly int Health;
        public readonly string Attack;

        public Breed(int health, string attack)
        {
            Health = health;
            Attack = attack;
        }

        public Monster SpawnMonster() => new Monster(this);
    }   


    private Breed _breed;
    private Monster(Breed breed)
    {
        _breed = breed;
    } 

    public int Health => _breed.Health;
    public string Attack => _breed.Attack;
}


var breed = new Monster.Breed(10, "ggg");
var monster = breed.SpawnMonster();
```

Создание объекта состоит из двух шагов: выделение и инициализация. Конструктор `Monster` позволяет нам выполнять всю необходимую инициализацию. В нашем примере это только сохранение породы, но полная игра будет загружать графику, инициализировать ИИ монстра и выполнять другую работу по настройке. Вместо того, чтобы просто вызывать `new`, `SpawnMonster()` может извлечь память из пула или пользовательской кучи перед передачей управления `Monster`для инициализации. Помещая эту логику внутрь `Breed`, в единственную функцию, которая имеет возможность создавать монстров, мы гарантируем, что все монстры пройдут через нужную нам схему управления памятью.


```cs

// COUNT ON CONSTRUCTION

public class Breed
{
	public readonly int Health;
	public readonly string Attack;
	private Breed _parent;

	public Breed(int health, string attack, Breed parent = null)
	{
		_parent = parent;

		if (_parent != null)
		{
			if (health == 0) Health = parent.Health;
			if (attack == "") Attack = parent.Attack;
		}
		else
		{
			Health = health;
			Attack = attack;
		}
	}

	public Monster SpawnMonster() => new Monster(this);
}
```


```cs

// COUNT EACH TIME

public class Breed
{
	private int _health;
	private string _attack;
	private Breed _parent;

	public Breed(int health, string attack, Breed parent = null)
	{
		_health = health;
		_attack = attack;
		_parent = parent;
	}

	public int GetHealth() => _health != 0 || _parent is null ? _health : _parent.GetHealth();
	public string GetAttack() => _attack != "" || _parent is null ? _attack : _parent.GetAttack();

	public Monster SpawnMonster() => new Monster(this);
}
```