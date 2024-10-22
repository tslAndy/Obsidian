#conditional
**У вас есть сложный условный оператор (`if-then`/`else` или `switch`).**
Выделите в отдельные методы все сложные части оператора: условие, `then` и `else`.
```cs
if (date < SUMMER_START || date > SUMMER_END) 
{
  charge = quantity * winterRate + winterServiceCharge;
}
else 
{
  charge = quantity * summerRate;
}

//------------------------------------------------------------------------

if (isSummer(date))
{
  charge = SummerCharge(quantity);
}
else 
{
  charge = WinterCharge(quantity);
}
```
