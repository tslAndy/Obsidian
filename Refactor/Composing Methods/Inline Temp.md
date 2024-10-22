#composing
**У вас есть временная переменная, которой присваивается результат простого выражения (и больше ничего).**
Замените обращения к переменной этим выражением.
```cs
bool HasDiscount(Order order)
{
  double basePrice = order.BasePrice();
  return basePrice > 1000;
}

//------------------------------------------------------------------------

bool HasDiscount(Order order)
{
  return order.BasePrice() > 1000;
}
```
