#otherpatterns 
Вводит уровень абстракции между игровой логикой и источником данных. Определяет интерфейс `IRepository<T>` который определяет общий интерфейс для управления определенным типом данных `T`. 

Плюсы:
- Улучшенная организация кода. Отделяет логику доступа к данным от игровой логики
- Улучшенная гибкость. Переключение источника данных просто, так как игра взаимодейтсвует с абстрактным интерфейсом, а не определенной реализацией
- Улучшенная тестируемость. Позволяет имитировать репозиторий для модульного тестирования игровой логики без решения реальных проблем доступа к данным.
- Уменьшенная связанность. Отделяет игровую логику от специфик источника данных. 

```cs
public interface IRepository<T> where T : class
{
    T GetById(int id);
    void Save(T entity);
    void Delete(T entity);
    // ... other common operations
}
```

```cs
public class PlayerManager
{
    private readonly IPlayerRepository _playerRepository;

    public PlayerManager(IPlayerRepository playerRepository)
    {
        _playerRepository = playerRepository;
    }

    public void LoadPlayerData(int playerId)
    {
        Player player = _playerRepository.GetById(playerId);
        // Use the loaded player data
    }

    public void SavePlayerData(Player player)
    {
        _playerRepository.Save(player);
    }
}
```

## Injecting Repositories in Unity
```cs
[Inject]
private IPlayerRepository _playerRepository;
```

Можно использовать с этим паттерном [[Unit Of Work +]]