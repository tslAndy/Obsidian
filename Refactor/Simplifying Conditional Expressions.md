#refactor
## Consolidate Conditional Expression
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


## Consolidate Duplicate Conditional Fragments
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

## Decompose Conditional
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

## Replace Conditional with Polymorphism
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

## Remove Control Flag
**У вас есть булева переменная, которая играет роль управляющего флага для нескольких булевых выражений.**
Используйте `break`, `continue` и `return` вместо этой переменной. 

## Replace Nested Conditional with Guard Clauses
**У вас есть группа вложенных условных операторов, среди которых сложно выделить нормальный ход выполнения кода.**
Выделите все проверки специальных или граничных случаев выполнения в отдельные условия и поместите их перед основными проверками. В идеале, вы должны получить «плоский» список условных операторов, идущих один за другим.

```cs
public double GetPayAmount()
{
  double result;
  
  if (isDead)
  {
    result = DeadAmount();
  }
  else 
  {
    if (isSeparated)
    {
      result = SeparatedAmount();
    }
    else 
    {
      if (isRetired)
      {
        result = RetiredAmount();
      }
      else
      {
        result = NormalPayAmount();
      }
    }
  }
  
  return result;
}

//------------------------------------------------------------------------

public double GetPayAmount() 
{
  if (isDead)
  {
    return DeadAmount();
  }
  if (isSeparated)
  {
    return SeparatedAmount();
  }
  if (isRetired)
  {
    return RetiredAmount();
  }
  return NormalPayAmount();
}
```

## Introduce Null Object
**Из-за того, что некоторые методы возвращают `null` вместо реальных объектов, у вас в коде присутствует множество проверок на `null`.**
Вместо null возвращайте `Null`-объект, который предоставляет поведение по умолчанию.

```cs
if (customer == null) 
{
  plan = BillingPlan.Basic();
}
else 
{
  plan = customer.GetPlan();
}

//------------------------------------------------------------------------

public sealed class NullCustomer: Customer 
{
  public override bool IsNull 
  {
    get { return true; }
  }
  
  public override Plan GetPlan() 
  {
    return new NullPlan();
  }
  // Some other NULL functionality.
}

// Replace null values with Null-object.
customer = order.customer ?? new NullCustomer();

// Use Null-object as if it's normal subclass.
plan = customer.GetPlan();
```

## Introduce Assertion
**Корректная работа участка кода предполагает наличие каких-то определённых условий или значений.**
Замените эти предположения конкретными проверками.
```cs
double GetExpenseLimit() 
{
  // Should have either expense limit or
  // a primary project.
  return (expenseLimit != NULL_EXPENSE) ?
    expenseLimit :
    primaryProject.GetMemberExpenseLimit();
}

//------------------------------------------------------------------------

double GetExpenseLimit() 
{
  Assert.IsTrue(expenseLimit != NULL_EXPENSE || primaryProject != null);

  return (expenseLimit != NULL_EXPENSE) ?
    expenseLimit:
    primaryProject.GetMemberExpenseLimit();
}
```