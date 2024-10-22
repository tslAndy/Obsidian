#conditional
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