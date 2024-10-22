#betweenObjects
**Служебный класс не содержит метода, который вам нужен, при этом у вас нет возможности добавить метод в этот класс.**
Добавьте метод в клиентский класс и передавайте в него объект служебного класса в качестве аргумента.
```cs
class Report 
{
  // ...
  void SendReport() 
  {
    DateTime nextDay = previousEnd.AddDays(1);
    // ...
  }
}

//------------------------------------------------------------------------

class Report 
{
  // ...
  void SendReport() 
  {
    DateTime nextDay = NextDay(previousEnd);
    // ...
  }
  private static DateTime NextDay(DateTime date) 
  {
    return date.AddDays(1);
  }
}
```