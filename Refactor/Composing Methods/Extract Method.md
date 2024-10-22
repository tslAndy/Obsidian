#composing
**У вас есть фрагмент кода, который можно сгруппировать.**
Выделите участок кода в новый метод (или функцию) и вызовите этот метод вместо старого кода.
```cs
void PrintOwing() 
{
  this.PrintBanner();

  // Print details.
  Console.WriteLine("name: " + this.name);
  Console.WriteLine("amount: " + this.GetOutstanding());
}

//------------------------------------------------------------------------

void PrintOwing()
{
  this.PrintBanner();
  this.PrintDetails();
}

void PrintDetails()
{
  Console.WriteLine("name: " + this.name);
  Console.WriteLine("amount: " + this.GetOutstanding());
}
```
