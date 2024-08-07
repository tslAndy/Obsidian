#other 
Проблема следующего кода в том, что `FixedUpdate` вызывается перед тем, как происходит обновление физики, т. е. `FixedUpdate` будет пытаться переместить объект с помощью `rb.MovePosition`, который запустится только после `FixedUpdate`

```cs
private void FixedUpdate()
{
    if(_isMoving)
        MoveToPositionWithWhile(_randomPoint);
}

private void MoveToPositionWithWhile(Vector2 position)
{
    // Won't work fixed update stops running until while loop is finished =>
    // Prevents physics simulation from running
    
    Vector2 currentPos = transform.position;

    while ((position - currentPos).sqrMagnitude >= EPSILON)
    {
        _rb.MovePosition(currentPos +
                         speed * Time.fixedDeltaTime *
                        (position - currentPos).normalized);
    }

    transform.position = position;
    _isMoving = false;
}
```

Для решения проблемы заменяем `while` на `if`:

```cs
private void MoveToPositionWithIf(Vector2 position)
{
    Vector2 currentPos = transform.position;
    
    if ((position - currentPos).sqrMagnitude >= EPSILON)
    {
        _rb.MovePosition(currentPos + 
        speed * Time.fixedDeltaTime * (position - currentPos).normalized);
    }
    else
    {
        transform.position = position;
        _isMoving = false;
    }
}
```

Рассмотрим другую ситуацию, при которой у нас есть массив с позициями врагов, и мы хотим найти расстояние между ближайшим врагом к игроку и самим игроком, а так же позицию врага:

```cs
private void Update()
{
    if (!Keyboard.current.spaceKey.wasPressedThisFrame)
	    return;

	while (_indexer < _distances.Length)
	{
		float distance = Vector2.Distance(_position, _distances[_indexer]);
		if (distance < _minDistance)
		{
			_minDistance = distance;
			_nearestPosition = _distances[_indexer];
		}
		_indexer++;
	}
}
```

Небольшое предисловие. 60 FPS может получаться кадрами разной длины, допустим, что большинство кадров было по 10 мс, а один 40 мс, это будет заметно.
Проблема в том, что кадр, на котором выполнится код поиска, может быть так раз длинным кадром.

Если точность не важна, мы можем распределить вычисление на несколько кадров.

## Использование условного перехода

```cs
private void Update()
{
    if (Keyboard.current.spaceKey.wasPressedThisFrame && !_calculating)
    {
        _indexer = 0;
        _calculating = true;
    }

    if (_calculating)
    {
        if (_indexer < _distances.Length)
        {
            _distance = Vector2.Distance(_position, _distances[_indexer]);
            if ( _distance < _minDistance)
            {
                _minDistance = _distance;
                _nearestPosition = _distances[_indexer];
            }
            _indexer++;
        }
        else
        {
            _doneCalculating = true;
            _calculating = false;
        }
    }
    
    if (_doneCalculating)
    {
        Debug.Log(_nearestPosition);
        _doneCalculating = false;
    }
}
```

## `Coroutine`

```cs
private void Update()
{
    if (Keyboard.current.spaceKey.wasPressedThisFrame && !_calculating)
        StartCoroutine(FindNearestWithWhileCoroutine());

    if (_doneCalculating)
    {
        Debug.Log(_nearestPosition);
        _doneCalculating = false;
    }
}

IEnumerator FindNearestWithWhileCoroutine()
{
    int i = 0;
    _calculating = true;
    
    while (i < _distances.Length)
    {
        _distance = Vector2.Distance(_position, _distances[i]);
        if ( _distance < _minDistance)
        {
            _minDistance = _distance;
            _nearestPosition = _distances[i];
        }
        i++;
        
        yield return null;
    }

    _doneCalculating = true;
}
```

## `Awaitable` on the Main Thread

```cs
private async void Update()
{
    if (Keyboard.current.spaceKey.wasPressedThisFrame && !_calculating)
        await FindNearestWithAwaitable();

    if (_doneCalculating)
    {
        Debug.Log(_nearestPosition);
        _doneCalculating = false;
    }
}

private async Awaitable FindNearestWithAwaitable()
{
    int i = 0;
    _calculating = true;

    while (i < _distances.Length)
    {
        _distance = Vector2.Distance(_position, _distances[i]);
        if ( _distance < _minDistance)
        {
            _minDistance = _distance;
            _nearestPosition = _distances[i];
        }
        i++;

        await Awaitable.EndOfFrameAsync();
    }

    _doneCalculating = true;
    _calculating = false;
}
```

## `Awaitable` on a Background Thread

```cs
private async void Update()
{
    if (Keyboard.current.spaceKey.wasPressedThisFrame && !_calculating)
        await FindNearestWithAwaitableThread();

    if (_doneCalculating)
    {
        Debug.Log(_nearestPosition);
        _doneCalculating = false;
    }
}

private async Awaitable FindNearestWithAwaitableThread()
{
    int i = 0;
    _calculating = true;

    await Awaitable.BackgroundThreadAsync();
    
    while (i < _distances.Length)
    {
        _distance = Vector2.Distance(_position, _distances[i]);
        if ( _distance < _minDistance)
        {
            _minDistance = _distance;
            _nearestPosition = _distances[i];
        }
        i++;
    }

    await Awaitable.MainThreadAsync();

    _doneCalculating = true;
    _calculating = false;
}


```