#composing
**У вас есть длинный метод, в котором локальные переменные так сильно переплетены, что это делает невозможным применение «извлечения метода».**
Преобразуйте метод в отдельный класс так, чтобы локальные переменные стали полями этого класса. После этого можно без труда разделить метод на части.

```cs
public class Order 
{
  // ...
  public double Price() 
  {
    double primaryBasePrice;
    double secondaryBasePrice;
    double tertiaryBasePrice;
    // Perform long computation.
  }
}

//------------------------------------------------------------------------

public class Order 
{
  // ...
  public double Price() 
  {
    return new PriceCalculator(this).Compute();
  }
}

public class PriceCalculator 
{
  private double primaryBasePrice;
  private double secondaryBasePrice;
  private double tertiaryBasePrice;
  
  public PriceCalculator(Order order) 
  {
    // Copy relevant information from the
    // order object.
  }
  
  public double Compute() 
  {
    // Perform long computation.
  }
}
```