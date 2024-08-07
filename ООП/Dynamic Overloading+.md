#oop
Объект `orc` имеет тип `IEnemy` во время компиляции, но во время выполнения имеет тип `Orc`. Предположим также, что у нас есть класс `Player`, который имеет открытый метод, `Hunt(IEnemy enemy)`и мы хотим его использовать, но игроку нужно выполнять различные типы охоты в зависимости от врага, а также использовать определенные методы класса `Orc`, которые не являются частью интерфейса `IEnemy`.

```cs
public interface IEnemy { }

public class Orc : IEnemy
{
    public string ThrowStone() => "The Orc throws a stone.";
}

public class Goblin : IEnemy
{
    public int RunningSpeed() => 20;
}

public class Dragon : IEnemy
{
    public void Angry() => Console.WriteLine("The dragon got angry.");
}
```

```cs
public class Player
{
    public int PlayerSpeed { get; set; }
    public int MaxSpeed { get; } = 100;
    
    // some player code ...
    
    public void Hunt(IEnemy enemy)
    {
        switch (enemy)
        {
            case Orc orc:
                Console.WriteLine("Hunting stealthily");
                Console.WriteLine(orc.ThrowStone());
                break;
            case Goblin goblin:
                int speed = goblin.RunningSpeed();
                Console.WriteLine($"Goblin is running with speed of: {speed}");
                PlayerSpeed = speed;
                Console.WriteLine($"Player is now hunting with speed: {PlayerSpeed}");
                break;
            case Dragon dragon:
                dragon.Angry();
                PlayerSpeed = MaxSpeed;
                Console.WriteLine($"Player tries to get away with speed: {PlayerSpeed}");
                break;
            default:
                Console.WriteLine("Error unknown enemy type");
                break;
        }
    }
    
    // other player code ...
}
```

Если требования меняются, например:
- Мы хотим изменить способ охоты игрока на определенный тип врагов.
- Мы хотим изменить методы внутри классов противника.
- Мы хотим добавить/удалить новые типы врагов.

Нужно зайти внутрь этого оператора `switch` и начать что-то менять. Это не только подвержено ошибкам из-за оператора `switch`, который в конечном итоге может достичь сотен строк кода, но и из-за класса `Player`. Класс `Player` наверняка будет иметь и другой код, и для каждого изменения, которое нам нужно внести в метод `Hunt`, мы рискуем сломать класс `Player`.

Класс не только представляет объект реального мира, но также может представлять поведение. Поэтому мы создаем класс `PlayerHunt` и добавляем внутрь этот метод `Hunt`, так что класс `Player` теперь будет выглядеть так:

```cs
public class Player
{
    public int PlayerSpeed { get; set; }
    public int MaxSpeed { get; } = 100;
    
    private readonly PlayerHunt _playerHunt = new();
    
    // some player code ...
    public void Hunt(IEnemy enemy) => _playerHunt.Hunt(this, enemy);
    // other player code ...
}
```

Обратите внимание, что мы изменили сигнатуру метода `Hunt`, потому что нам нужен способ получить доступ к методам и свойствам `Player`. Это решает нашу проблему с изменениями в классе `Player`, когда мы хотим внести изменения в различные методы охоты, но не решает нашу проблему с этим гигантским оператором `switch`. Теперь этот оператор `switch` находится внутри метода охоты класса `PlayerHunt`.

## Динамическая перегрузка

```cs
public void Hunt(Player player, IEnemy enemy) => 
    DynamicHunt(player, (dynamic) enemy);

```

```cs
public class PlayerHunt
{
    public void Hunt(Player player, IEnemy enemy) => DynamicHunt(player, (dynamic) enemy);

    private void DynamicHunt(Player player, Orc orc)
    {
        Console.WriteLine("Hunting stealthily");
        Console.WriteLine(orc.ThrowStone());
    }

    private void DynamicHunt(Player player, Goblin goblin)
    {
        int speed = goblin.RunningSpeed();
        Console.WriteLine($"Goblin is running with speed of: {speed}");
        player.PlayerSpeed = speed;
        Console.WriteLine($"Player is now hunting with speed: {player.PlayerSpeed}");
    }

    private void DynamicHunt(Player player, Dragon dragon)
    {
        player.PlayerSpeed = player.MaxSpeed;
        dragon.Angry();
        Console.WriteLine($"Player tries to get away with speed: {player.PlayerSpeed}");
    }

    private void DynamicHunt(Player player, IEnemy unknownEnemy)
    {
        Console.WriteLine("Error unknown enemy type");
    }
}
```

Скорее всего того же можно было бы достигнуть с помощью паттерна [[Посетитель -]]
