#conditional
**У вас есть несколько условных операторов, ведущих к одинаковому результату или действию.**
Объедините все условия в одном условном операторе.
```cs
double DisabilityAmount() 
{
  if (seniority < 2) 
  {
    return 0;
  }
  if (monthsDisabled > 12) 
  {
    return 0;
  }
  if (isPartTime) 
  {
    return 0;
  }
  // Compute the disability amount.
  // ...
}

//------------------------------------------------------------------------

double DisabilityAmount()
{
  if (IsNotEligibleForDisability())
  {
    return 0;
  }
  // Compute the disability amount.
  // ...
}

```