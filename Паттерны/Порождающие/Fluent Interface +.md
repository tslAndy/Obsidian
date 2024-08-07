#creational
Помогает сконфигурировать объект без использования большого количества параметров в конструкторе. Допускает свободный порядок вызова конфигурационных методов

Может менять как изначальный объект, так и каждый раз создавать новый.

Применение в играх - конфигурирование сущности

Здесь есть два варианта
- Храним дублирующие переменные в классе строителя и выставляем их в классе врага самом конце. При этом хорошо задать значения по умолчанию.
- Сразу конфигурируем конечный объект. 

Если хотим полностью запретить создание объекта вне строителя, делаем конструктор врага приватным.

Можно инициализировать строителя с префабом вместо создания нового объекта.

```cs

class Enemy
{
	public int Health {get; private set;}
	public int Speed {get; private set;}
	public int Damage {get; private set;}
	public string Name {get; private set;}

	class EnemyBuilder
	{
		private Enemy _instance = new();
		
		public EnemyBuilder WithHealth(int health)
		{
			_instance.Health = health;
			return this;
		}
		
		public EnemyBuilder WithName(string name)
		{
			_instance.Name = name;
			return this;
		}

		// ... other methods to configure

		public Enemy Build() => _instance;

	}
}

```

