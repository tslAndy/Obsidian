#conditional
**У вас есть группа вложенных условных операторов, среди которых сложно выделить нормальный ход выполнения кода.**
Выделите все проверки специальных или граничных случаев выполнения в отдельные условия и поместите их перед основными проверками. В идеале, вы должны получить «плоский» список условных операторов, идущих один за другим.

```cs
public double GetPayAmount()
{
  double result;
  
  if (isDead)
  {
    result = DeadAmount();
  }
  else 
  {
    if (isSeparated)
    {
      result = SeparatedAmount();
    }
    else 
    {
      if (isRetired)
      {
        result = RetiredAmount();
      }
      else
      {
        result = NormalPayAmount();
      }
    }
  }
  
  return result;
}

//------------------------------------------------------------------------

public double GetPayAmount() 
{
  if (isDead)
  {
    return DeadAmount();
  }
  if (isSeparated)
  {
    return SeparatedAmount();
  }
  if (isRetired)
  {
    return RetiredAmount();
  }
  return NormalPayAmount();
}
```