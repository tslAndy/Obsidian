#conditional
**Одинаковый фрагмент кода находится во всех ветках условного оператора.**
Вынесите его за рамки оператора.
```cs
if (IsSpecialDeal()) 
{
  total = price * 0.95;
  Send();
}
else 
{
  total = price * 0.98;
  Send();
}

//------------------------------------------------------------------------

if (IsSpecialDeal())
{
  total = price * 0.95;
}
else
{
  total = price * 0.98;
}
Send();
```