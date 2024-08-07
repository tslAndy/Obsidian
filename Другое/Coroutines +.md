#other 
## Важно знать
API Unity не является потокобезопасным, и вам следует вызывать функции, специфичные для Unity, из основного потока. Здесь сопрограммы имеют преимущество перед потоками, поскольку они работают в основном потоке, хотя и не так, как обычный код.

## Loop control

```cs
private IEnumerator DoTask()
{
    float startXPos = 3.0f;
    float endXPos = 10.0f;

    Vector3 position;

    for (float t = 0.0f; t < 5.0f; t += Time.deltaTime)
    {
        position = transform.position;
        position.x = Mathf.Lerp(startXPos, endXPos, t / 5.0f);
        transform.position = position;

        yield return null;
    }

    position = transform.position;
    position.x = endXPos;
    transform.position = position;
}
```

## Отличие `Invoke` От `StartCoroutine`
Во-первых, с помощью `Invoke` нельзя вызывать методы с параметром.
Во-вторых, `WaitForSeconds` масштабируется согласно `Time.timeScale`, `Invoke`, в свою очередь, нет. При этом если нужен корутин с реальным временем, то можно использовать `WaitForSecondsRealtime`.  Если период времени не меняется, то можно сохранить `WaitForSeconds` как приватное поле.

```cs
private void Start()
{
    Invoke("MyFunc", 5.0f);
}

// OR

private void Start()
{
    StartCoroutine(MyFunc2(1, 5.0f));
}

private IEnumerator MyFunc2(int param1, float waitTime)
{
    yield return new WaitForSeconds(waitTime);
}
```

## Цепочки корутинов

```cs
private WaitForSeconds waitFiveSeconds = new WaitForSeconds(5.0f);

private void Start()
{
    StartCoroutine(MyFunc1());
}

private IEnumerator MyFunc1()
{
    Debug.Log("MyFunc1: started.");
    yield return StartCoroutine(MyFunc2(1, 5.0f));
    Debug.Log("MyFunc1: finished MyFunc2.");
    yield return waitFiveSeconds;
    Debug.Log("MyFunc1: waited 5 seconds.");
}

private IEnumerator MyFunc2(int param1, float waitTime)
{
    Debug.Log("MyFunc2: started.");
    yield return waitFiveSeconds;
    Debug.Log("MyFunc2: waited 5 seconds.");
}
```


## `IEnumerator` в `Start`, `OnCollision`, `OnEnter` и т д
Многие методы, не связанные с жизненным циклом напрямую могут иметь возвращаемым типом `IEnumerator`.

```cs
private WaitForSeconds waitFiveSeconds = new WaitForSeconds(5.0f);

private IEnumerator Start()
{
    yield return waitFiveSeconds;
    // DO SMTH
}
```

## Тяжелые операции в отдельном корутине, а не основном цикле
```cs
private void Start()
{
    StartCoroutine(UpdateEachSecond());
}

private IEnumerator UpdateEachSecond()
{
    var wait = new WaitForSecondsRealtime(1.0f);

    while(true)
    {
        // Do expensive operations here.

        yield return wait;
    }
}
```

При этом важно помнить, что на отключенных объектах корутин все еще будет работать, и его нужно останавливать:

```cs
private Coroutine updateEachSecond;

private void OnEnable()
{
    updateEachSecond = StartCoroutine(UpdateEachSecond());
}

private void OnDisable()
{
    StopCoroutine(updateEachSecond);
    updateEachSecond = null;
}

private IEnumerator UpdateEachSecond()
{
    var wait = new WaitForSecondsRealtime(1.0f);

    while(true)
    {
        // Do expensive operations here.

        yield return wait;
    }
}
```

## Custom `YieldInstructions`

```cs
public class WaitForRandomChance : CustomYieldInstruction
{
    private float threshold = 0.0f;

    public override bool keepWaiting
    {
        get
        {
            return Random.value <= threshold;
        }
    }

    public WaitForRandomChance(float threshold)
    {
        this.threshold = threshold;
    }
}


private IEnumerator MyFunc()
{
    yield return new WaitForRandomChance(0.25f);
}

```

Или

```cs
public class WaitForButtonDown : CustomYieldInstruction
{
    private string buttonName;

    public override bool keepWaiting
    {
        get
        {
            return !Input.GetButtonDown(buttonName);
        }
    }

    public WaitForButtonDown(string buttonName)
    {
        this.buttonName = buttonName;
    }
}
```

## `WaitWhile` и `WaitUntil`
Принимают bool делегат:
```cs
private IEnumerator MyFunc(string buttonName)
{
    yield return new WaitUntil(() => Input.GetButtonDown(buttonName));
}
```

## Кэширование `WaitForSeconds`

```cs
private IEnumerator EnemySpawner(float time)
{
    WaitForSeconds delay = new WaitForSeconds(time);
    
    // some code at the beginning
    while (aCondition)
    {
        // spawn enemy code
        yield return delay;
        // some other code
    }
}
```