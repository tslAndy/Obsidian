#conditional
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