#methodcalls 
**Метод возвращает определенное значение, которое будет сигнализировать об ошибке.**
Вместо этого следует выбрасывать исключение.
```cs
int Withdraw(int amount) 
{
  if (amount > _balance) 
  {
    return -1;
  }
  else 
  {
    balance -= amount;
    return 0;
  }
}

//------------------------------------------------------------------------

///<exception cref="BalanceException">Thrown when amount > _balance</exception>
void Withdraw(int amount)
{
  if (amount > _balance) 
  {
    throw new BalanceException();
  }
  balance -= amount;
}
```
