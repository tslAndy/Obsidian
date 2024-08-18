#refactor
## Add Parameter
**Методу не хватает данных для осуществления каких-то действий.**
Создайте новый параметр, чтобы передать эти данные.

![[Pasted image 20240818185251.png]]

## Remove Parameter
**Параметр не используется в теле метода.**
Удалите неиспользуемый параметр.

![[Pasted image 20240818185359.png]]

## Rename Method
**Название метода не раскрывает суть того, что он делает.**
Измените название метода.

![[Pasted image 20240818185429.png]]

## Separate Query from Modifier
**У вас есть метод, который возвращает какое-то значение, но при этом в процессе работы он изменяет что-то внутри объекта.**
Разделите метод на два разных метода. Один из них пускай возвращает значение, а второй модифицирует объект.

![[Pasted image 20240818185512.png]]

## Parameterize Method
**Несколько методов выполняют похожие действия, которые отличаются только какими-то внутренними значениями, числами или операциями.**
Объедините все эти методы в один с параметром, в который будет передаваться отличающееся значение.

![[Pasted image 20240818185551.png]]

## Introduce Parameter 
**В ваших методах встречается повторяющаяся группа параметров.**
Замените эти параметры объектом.

![[Pasted image 20240818185636.png]]

## Preserve Whole Object
**Вы получаете несколько значений из объекта, а затем передаёте их в метод как параметры.**
Вместо этого передавайте весь объект.
```cs
int low = daysTempRange.GetLow();
int high = daysTempRange.GetHigh();
bool withinPlan = plan.WithinRange(low, high);

//------------------------------------------------------------------------

bool withinPlan = plan.WithinRange(daysTempRange);
```

## Remove Setting Method
**Значение поля должно быть установлено только в момент создания и больше никогда не меняться.**
Удалите методы, устанавливающие значение этого поля.

![[Pasted image 20240818185745.png]]

## Replace Parameter with Explicit Methods
**Метод разбит на части, каждая из которых выполняется в зависимости от значения какого-то параметра.**
Извлеките отдельные части метода в собственные методы и вызывайте их вместо оригинального метода.

```cs
void SetValue(string name, int value) 
{
  if (name.Equals("height")) 
  {
    height = value;
    return;
  }
  if (name.Equals("width")) 
  {
    width = value;
    return;
  }
  Assert.Fail();
}

//------------------------------------------------------------------------

void SetHeight(int arg) 
{
  height = arg;
}
void SetWidth(int arg) 
{
  width = arg;
}
```

## Replace Parameter with Method Call
**Вызываем метод и передаем его результаты как параметры другого метода. При этом значение параметров могли бы быть получены и внутри вызываемого метода.**
Вместо передачи значения через параметры метода, попробуйте переместить код получения значения внутрь самого метода.

```cs
int basePrice = quantity * itemPrice;
double seasonDiscount = this.GetSeasonalDiscount();
double fees = this.GetFees();
double finalPrice = DiscountedPrice(basePrice, seasonDiscount, fees);

//------------------------------------------------------------------------

int basePrice = quantity * itemPrice;
double finalPrice = DiscountedPrice(basePrice);
```

## Hide Method
**Метод не используется другими классами либо используется только внутри своей иерархии классов.**
Сделайте метод приватным или защищённым.

![[Pasted image 20240818185946.png]]

## Replace Constructor with Factory Method
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

## Replace Error Code with Exception
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

## Replace Exception with Test
**Вы выбрасываете исключение там, где можно было бы обойтись простой проверкой условия.**
Замените выбрасывание исключения проверкой этого условия.
```cs
double GetValueForPeriod(int periodNumber) 
{
  try 
  {
    return values[periodNumber];
  } 
  catch (IndexOutOfRangeException e) 
  {
    return 0;
  }
}

//------------------------------------------------------------------------

double GetValueForPeriod(int periodNumber) 
{
  if (periodNumber >= values.Length) 
  {
    return 0;
  }
  return values[periodNumber];
}
```