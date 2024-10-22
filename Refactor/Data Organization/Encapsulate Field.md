#dataorg
**У вас есть публичное поле.**
Сделайте поле приватным и создайте для него методы доступа.
```cs
class Person 
{
  public string name;
}

//------------------------------------------------------------------------

class Person 
{
  private string name;

  public string Name
  {
    get { return name; }
    set { name = value; }
  }
}
```
