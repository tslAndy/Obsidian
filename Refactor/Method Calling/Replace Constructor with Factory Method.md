#methodcalls 
**У вас есть сложный конструктор, делающий нечто большее, чем простая установка значений полей объекта.**
Создайте фабричный метод и замените им вызовы конструктора.
```cs
public class Employee 
{
  public Employee(int type) 
  {
    this.type = type;
  }
  // ...
}

//------------------------------------------------------------------------

public class Employee
{
  public static Employee Create(int type)
  {
    employee = new Employee(type);
    // Do some heavy lifting.
    return employee;
  }
  // ...
}
```
