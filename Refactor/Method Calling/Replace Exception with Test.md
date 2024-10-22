#methodcalls 
**Вы выбрасываете исключение там, где можно было бы обойтись простой проверкой условия.**
Замените выбрасывание исключения проверкой этого условия.
```cs
double GetValueForPeriod(int periodNumber) 
{
  try 
  {
    return values[periodNumber];
  } 
  catch (IndexOutOfRangeException e) 
  {
    return 0;
  }
}

//------------------------------------------------------------------------

double GetValueForPeriod(int periodNumber) 
{
  if (periodNumber >= values.Length) 
  {
    return 0;
  }
  return values[periodNumber];
}
```