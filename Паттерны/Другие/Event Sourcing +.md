#otherpatterns 
Сохранение игрового состояния путем записи событий вместо сохранения состояния. Чем-то похож на [[Команда +]].

- Лучшая перепроигрываемость. Позволяет перематывать и воспроизводить игру с любой точки потока событий, обеспечивая такие функции, как отладка, повторы и альтернативные временные шкалы.
- Улучшенная масштабируемость. Горизонтальное масштабирование хранилища событий упрощается, поскольку события по своей сути неизменны и независимы.
- Упрощенная отладка. Позволяет упростить анализ изменений состояния игры и выявление потенциальных проблем.
- Слабое связывание. Отделяет игровую логику от конкретного способа обработки или отображения событий.

```cs
public class PlayerMovedEvent
{
    public Vector3 Position { get; private set; }

    public PlayerMovedEvent(Vector3 position)
    {
        Position = position;
    }
}

public class ItemPickedUpEvent
{
    public string ItemId { get; private set; }

    public ItemPickedUpEvent(string itemId)
    {
        ItemId = itemId;
    }
}


public void ReplayGame(IEnumerable<IEvent> events)
{
    foreach (var event in events)
    {
        if (event is PlayerMovedEvent moveEvent)
        {
            // Update player position based on moveEvent.Position
        }
        else if (event is ItemPickedUpEvent pickupEvent)
        {
            // Add item with pickupEvent.ItemId to player inventory
        }
        // ... handle other event types
    }
}
```

Сохранение событий:
- События могут храниться различными способами, включая базы данных, хранилище в памяти или даже распределенные системы.
- Выбор подходящего решения для хранения данных зависит от конкретных потребностей вашего проекта и требований к масштабируемости.