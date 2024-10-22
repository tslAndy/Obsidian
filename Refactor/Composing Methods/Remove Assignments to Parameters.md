#composing
**Параметру метода присваивается какое-то значение.**
Вместо параметра воспользуйтесь новой локальной переменной.
```cs
int Discount(int inputVal, int quantity) 
{
  if (quantity > 50) 
  {
    inputVal -= 2;
  }
  // ...
}

//------------------------------------------------------------------------

int Discount(int inputVal, int quantity) 
{
  int result = inputVal;
  
  if (quantity > 50) 
  {
    result -= 2;
  }
  // ...
}
```