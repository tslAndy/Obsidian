#methodcalls 
**Вы получаете несколько значений из объекта, а затем передаёте их в метод как параметры.**
Вместо этого передавайте весь объект.
```cs
int low = daysTempRange.GetLow();
int high = daysTempRange.GetHigh();
bool withinPlan = plan.WithinRange(low, high);

//------------------------------------------------------------------------

bool withinPlan = plan.WithinRange(daysTempRange);
```
