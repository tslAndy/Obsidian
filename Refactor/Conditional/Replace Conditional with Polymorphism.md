#conditional
**У вас есть условный оператор, который, в зависимости от типа или свойств объекта, выполняет различные действия.**
Создайте подклассы, которым соответствуют ветки условного оператора. В них создайте общий метод и переместите в него код из соответствующей ветки условного оператора. Впоследствии замените условный оператор на вызов этого метода. Таким образом, нужная реализация будет выбираться через полиморфизм в зависимости от класса объекта.
```cs
public class Bird 
{
  // ...
  public double GetSpeed() 
  {
    switch (type) 
    {
      case EUROPEAN:
        return GetBaseSpeed();
      case AFRICAN:
        return GetBaseSpeed() - GetLoadFactor() * numberOfCoconuts;
      case NORWEGIAN_BLUE:
        return isNailed ? 0 : GetBaseSpeed(voltage);
      default:
        throw new Exception("Should be unreachable");
    }
  }
}

//------------------------------------------------------------------------

public abstract class Bird 
{
  // ...
  public abstract double GetSpeed();
}

class European: Bird 
{
  public override double GetSpeed() 
  {
    return GetBaseSpeed();
  }
}
class African: Bird 
{
  public override double GetSpeed() 
  {
    return GetBaseSpeed() - GetLoadFactor() * numberOfCoconuts;
  }
}
class NorwegianBlue: Bird
{
  public override double GetSpeed() 
  {
    return isNailed ? 0 : GetBaseSpeed(voltage);
  }
}

// Somewhere in client code
speed = bird.GetSpeed();
```