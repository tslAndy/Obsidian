#composing
**Вы помещаете результат какого-то выражения в локальную переменную, чтобы использовать её далее в коде.** 
Выделите все выражение в отдельный метод и возвращайте результат из него. Замените использование вашей переменной вызовом метода. Новый метод может быть использован и в других методах.
```cs
double CalculateTotal() 
{
  double basePrice = quantity * itemPrice;
  
  if (basePrice > 1000) 
  {
    return basePrice * 0.95;
  }
  else 
  {
    return basePrice * 0.98;
  }
}

//------------------------------------------------------------------------

double CalculateTotal() 
{
  if (BasePrice() > 1000) 
  {
    return BasePrice() * 0.95;
  }
  else 
  {
    return BasePrice() * 0.98;
  }
}
double BasePrice() 
{
  return quantity * itemPrice;
}
```