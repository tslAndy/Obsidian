#composing
**Стоит использовать в том случае, когда тело метода очевиднее самого метода.**
Замените вызовы метода его содержимым и удалите сам метод.
```cs
class PizzaDelivery 
{
  // ...
  int GetRating() 
  {
    return MoreThanFiveLateDeliveries() ? 2 : 1;
  }
  bool MoreThanFiveLateDeliveries() 
  {
    return numberOfLateDeliveries > 5;
  }
}

//------------------------------------------------------------------------

class PizzaDelivery 
{
  // ...
  int GetRating() 
  {
    return numberOfLateDeliveries > 5 ? 2 : 1;
  }
}
```
