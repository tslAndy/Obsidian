#decoupling
## Описание
- Отделет получателя и отправителя
- Отделяет запрос и его обработку во времени

## Отличие между событием и запросом
- Событие описывает то, что уже произошло. Часто несколько получателей, из-за его более обширная видимость
- Запрос описывает действие, которое хотим выполнить в будущем. Чаще всего один получатель.


## Single Cast очередь 

Очередь является API класса.
- Очередь становится деталью реализации читателя. Все, что знает отправитель, это то, что он отправил сообщение.
- Очередь более инкапсулирована. При прочих равных условиях большая инкапсуляция обычно лучше.
- Не нужно беспокоиться о конкуренции между слушателями. При наличии нескольких слушателей нужно решить, получат ли они все каждый элемент или каждый элемент в очереди будет распределен по одному слушателю.


## Broadcast очередь

Так работает большинсто событийных систем. 
- Если ноль слушателей, все ноль из них видят событие. В большинстве систем событий, если в момент обработки события нет слушателей, событие отбрасывается.
- Может понадобиться фильтровать события. Очереди широковещательных сообщений часто видны большей части программы, и вы можете получить кучу слушателей. Умножьте множество событий на множество слушателей, и вы получите кучу обработчиков событий для вызова. Большинство систем широковещательных событий позволяют слушателю отсеивать набор событий, которые он получает. Например, они могут сказать, что хотят получать только события мыши или события в определенной области пользовательского интерфейса.

## Очередь работ

Также есть несколько слушателей. Разница в том, что каждый элемент в очереди попадает только к одному из них. Это распространенный шаблон для распределения заданий по пулу одновременно работающих потоков.
- Поскольку элемент попадает только к одному слушателю, очереди нужна логика, чтобы определить, какой из них лучше выбрать. Это может быть просто циклический или случайный выбор, или это может быть какая-то более сложная система приоритетов

## Упаковка запроса
Иногда может быть необходимо получить ссылку на отправителя, тогда стоит поместить ссылку в сам запрос


```cs
// AUDIO PLAYING EXAMPLE

struct PlayMessage
{
  SoundId id;
  int volume;
};

class Audio
{
	private static int numPending = 0;
	private static const MaxPending = 16;
	private static PlayMessage pending[MaxPending] = new();

	public static void Update()
	{
	    for (int i = 0; i < numPending; i++)
	    {
	      ResourceId resource = loadSound(pending_[i].id);
	      int channel = findOpenChannel();
	      if (channel == -1) return;
	      startSound(resource, channel, pending_[i].volume);
	    }
	}

	// will be called by client
	public static void PlaySound(SoundId id, int volume)
	{
		if (numPending == MaxPending)
			throw new Exception("Can not add more sounds");

		pending[numPending] = new PlayMessage(id, volume);
		numPending++;
	}
}
```

В примере ниже при переполнении массива можно выделить новый емкостью в два раза больше и скопировать элементы. Плюс использования очереди в том, что теперь обработка и воспроизведение звуков инкапсулировано, что дает возможность легко вынести код в отдельный поток. 

```cs
// CIRCLE BUFFER
class Audio
{
	private static int head = 0;
	private static int tail = 0;

	private static const MaxPending = 16;
	private static PlayMessage pending[MaxPending] = new();

	public static void Update()
	{
		// If there are no pending requests, do nothing.
		if (head_ == tail_) return;
		
		ResourceId resource = loadSound(pending_[head_].id);
		int channel = findOpenChannel();
		if (channel == -1) return;
		startSound(resource, channel, pending_[head_].volume);

		head++;
	}

	// will be called by client
	public static void PlaySound(SoundId id, int volume)
	{
		// don't add sound if same already exists
		for (int i = head; i != tail; i = (i + 1) % MaxPending)
		{
			if (pending[i].id != id) continue;
			pending[i].volume = Math.Max(volume, pending[i].volume);
			return;
		}


		if ((tail + 1) % MaxPending != head)
			throw new Exception("Can not add more sounds");

		pending[tail] = new PlayMessage(id, volume);

		// return to beginning 
		tail = (tail + 1) % MaxPending;
	}
}
```

