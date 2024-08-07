#other 
```cs
public interface IFoo
{
    // some methods here
}

struct Foo : IFoo
{
    // our struct that implements IFoo
}

public void Bar(IFoo barParameter)
{
    // Do something with the barParameter
}
```

Проблема в том, что в случае со ссылочным типом все будет работать как надо, но в случае со структорой компилятор ожидает object, а значет при вызове функции будет боксинг, который будет потреблять как время так и память на хипе. Пишем следующим образом:

```cs
public void Bar<T>(T barParameter) where T : IFoo
{
    // Do something with the barParameter
}
```

## Переопределение методов объекта класса
Если мы будем использовать методы `ToString()` или `Equals()`, то боксинг все равно будет, так как эти методы от класса object. Нужно убедиться, что структура переопределяет эти методы.

```cs
public void Bar<T>(T barParameter) where T : IFoo, IEquatable<T>
{
    // Do something with the barParameter
    // Do something with the barParameter that uses the Equals method
}
```

# Использование ref
В случае, если структура будет больше 16 байтов, появится проблема с производительностью, так как структуры передаются по значению. 

```cs
public void Bar<T>(ref T barParameter) where T : IFoo, IEquatable<T>
{
    // Do something with the barParameter
    // Do something with the barParameter that uses the Equals method
}
```