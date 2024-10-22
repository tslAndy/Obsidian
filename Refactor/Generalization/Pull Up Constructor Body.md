#generalization 
**Подклассы имеют конструкторы с преимущественно одинаковым кодом.**
Создайте конструктор в суперклассе и вынесите в него общий для подклассов код. Вызывайте конструктор суперкласса в конструкторах подкласса.
```cs
public class Manager: Employee 
{
  public Manager(string name, string id, int grade) 
  {
    this.name = name;
    this.id = id;
    this.grade = grade;
  }
  // ...
}

//------------------------------------------------------------------------

public class Manager: Employee 
{
  public Manager(string name, string id, int grade): base(name, id)
  {
    this.grade = grade;
  }
  // ...
}
```
