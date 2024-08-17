#other 
Предположим, чно у нас есть массив объектов, которые нужно обрабатывать согласно какому-то свойству, например, если объект активен или нет. 

Чтобы не проверять на каждом элементе, активен он или нет, можно использовать массив, в котором активные элементы будут в начале, а неактивные в конце.

```cs
public class ActiveArray<T>
{
    private T[] array = new T[1000];
    private int inactiveIndex = 0;

    public void Add(T elem)
    {
        array[inactiveIndex++] = elem;
    }

    public void Remove(int index)
    {
        array[--inactiveIndex] = default;
    }

	public int Length => inactiveIndex - 1;
	public T this[int index] => arrary[index];
}
```

Так же по возможности стоит избегать проверок типа :
```cs
if (m < 0) m = 0
else if (m > 10) m = 10

```

и конвертировать в:
```cs
Mathf.Clamp(m, 0, 10);
```