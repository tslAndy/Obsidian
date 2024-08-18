#refactor
## Extract Method
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

## Inline Method
**Стоит использовать в том случае, когда тело метода очевиднее самого метода.**
Замените вызовы метода его содержимым и удалите сам метод.
```cs
class PizzaDelivery 
{
  // ...
  int GetRating() 
  {
    return MoreThanFiveLateDeliveries() ? 2 : 1;
  }
  bool MoreThanFiveLateDeliveries() 
  {
    return numberOfLateDeliveries > 5;
  }
}

//------------------------------------------------------------------------

class PizzaDelivery 
{
  // ...
  int GetRating() 
  {
    return numberOfLateDeliveries > 5 ? 2 : 1;
  }
}
```

## Extract Variable
**У вас есть сложное для понимания выражение.**
Поместите результат выражения или его части в отдельные переменные, поясняющие суть выражения.
```cs
void RenderBanner() 
{
  if ((platform.ToUpper().IndexOf("MAC") > -1) &&
       (browser.ToUpper().IndexOf("IE") > -1) &&
        wasInitialized() && resize > 0 )
  {
    // do something
  }
}

//------------------------------------------------------------------------

void RenderBanner() 
{
  readonly bool isMacOs = platform.ToUpper().IndexOf("MAC") > -1;
  readonly bool isIE = browser.ToUpper().IndexOf("IE") > -1;
  readonly bool wasResized = resize > 0;

  if (isMacOs && isIE && wasInitialized() && wasResized) 
  {
    // do something
  }
}
```

## Inline Temp
**У вас есть временная переменная, которой присваивается результат простого выражения (и больше ничего).**
Замените обращения к переменной этим выражением.
```cs
bool HasDiscount(Order order)
{
  double basePrice = order.BasePrice();
  return basePrice > 1000;
}

//------------------------------------------------------------------------

bool HasDiscount(Order order)
{
  return order.BasePrice() > 1000;
}
```

## Replace Temp with Query
**Вы помещаете результат какого-то выражения в локальную переменную, чтобы использовать её далее в коде.** 
Выделите все выражение в отдельный метод и возвращайте результат из него. Замените использование вашей переменной вызовом метода. Новый метод может быть использован и в других методах.
```cs
double CalculateTotal() 
{
  double basePrice = quantity * itemPrice;
  
  if (basePrice > 1000) 
  {
    return basePrice * 0.95;
  }
  else 
  {
    return basePrice * 0.98;
  }
}

//------------------------------------------------------------------------

double CalculateTotal() 
{
  if (BasePrice() > 1000) 
  {
    return BasePrice() * 0.95;
  }
  else 
  {
    return BasePrice() * 0.98;
  }
}
double BasePrice() 
{
  return quantity * itemPrice;
}
```

## Split Temporary Variable
**У вас есть локальная переменная, которая используется для хранения разных промежуточных значений внутри метода (за исключением переменных циклов).**
Используйте разные переменные для разных значений. Каждая переменная должна отвечать только за одну определённую вещь.
```cs
double temp = 2 * (height + width);
Console.WriteLine(temp);
temp = height * width;
Console.WriteLine(temp);

//------------------------------------------------------------------------

readonly double perimeter = 2 * (height + width);
Console.WriteLine(perimeter);
readonly double area = height * width;
Console.WriteLine(area);
```

## Remove Assignments to Parameters
**Параметру метода присваивается какое-то значение.**
Вместо параметра воспользуйтесь новой локальной переменной.
```cs
int Discount(int inputVal, int quantity) 
{
  if (quantity > 50) 
  {
    inputVal -= 2;
  }
  // ...
}

//------------------------------------------------------------------------

int Discount(int inputVal, int quantity) 
{
  int result = inputVal;
  
  if (quantity > 50) 
  {
    result -= 2;
  }
  // ...
}
```

## Replace Method with Method Object
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

## Substitute Algorithm
**Вы хотите заменить существующий алгоритм другим?**
Замените тело метода, реализующего старый алгоритм, новым алгоритмом.
```cs
string FoundPerson(string[] people)
{
  for (int i = 0; i < people.Length; i++) 
  {
    if (people[i].Equals("Don"))
    {
      return "Don";
    }
    if (people[i].Equals("John"))
    {
      return "John";
    }
    if (people[i].Equals("Kent"))
    {
      return "Kent";
    }
  }
  return String.Empty;
}

//------------------------------------------------------------------------

string FoundPerson(string[] people)
{
  List<string> candidates = new List<string>() {"Don", "John", "Kent"};
  
  for (int i = 0; i < people.Length; i++) 
  {
    if (candidates.Contains(people[i])) 
    {
      return people[i];
    }
  }
  
  return String.Empty;
}
```

## Replace Command With Function
```cs
class ChargeCalculator {
  constructor (customer, usage){
    this._customer = customer;
    this._usage = usage;
  }
  execute() {
    return this._customer.rate * this._usage;
  }
}

//------------------------------------------------------------------------

function charge(customer, usage) {
  return customer.rate * usage;
}
```

## Replace Function with Command
```cs
function score(candidate, medicalExam, scoringGuide) {
	let result = 0;
	let healthLevel = 0;
	// long body code
}

//------------------------------------------------------------------------

class Scorer {
  constructor(candidate, medicalExam, scoringGuide) {
    this._candidate = candidate;
    this._medicalExam = medicalExam;
    this._scoringGuide = scoringGuide;
  }

  execute() {
    this._result = 0;
    this._healthLevel = 0;
    // long body code
  }
}
```

## Replace Derived Variable with Query
```cs
get discountedTotal() {return this._discountedTotal;}
set discount(aNumber) {
const old = this._discount;
this._discount = aNumber;
this._discountedTotal += old - aNumber; 
}

//------------------------------------------------------------------------

get discountedTotal() {return this._baseTotal - this._discount;}
set discount(aNumber) {this._discount = aNumber;}
```

## Replace Exception with Precheck
```cs
double getValueForPeriod (int periodNumber) {
  try {
    return values[periodNumber];
  } catch (ArrayIndexOutOfBoundsException e) {
    return 0;
  }
}

//------------------------------------------------------------------------

double getValueForPeriod (int periodNumber) {
  return (periodNumber >= values.length) ? 0 : values[periodNumber];
}
```

## Replace Inline Code with Function Call
```cs
let appliesToMass = false;
for(const s of states) {
	if (s === "MA") appliesToMass = true;
}

//------------------------------------------------------------------------

appliesToMass = states.includes("MA");
```

## Replace Loop with Pipeline
```cs
const names = [];
for (const i of input) {
if (i.job === "programmer")
  names.push(i.name);
}

//------------------------------------------------------------------------

const names = input
.filter(i => i.job === "programmer")
.map(i => i.name)
;
```

## Replace Primitive with Object
```cs
orders.filter(o => "high" === o.priority
                || "rush" === o.priority);

//------------------------------------------------------------------------

orders.filter(o => o.priority.higherThan(new Priority("normal")))
```

## Replace Query with Parameter
```cs
targetTemperature(aPlan)

function targetTemperature(aPlan) {
  currentTemperature = thermostat.currentTemperature;
  // rest of function...

//------------------------------------------------------------------------

targetTemperature(aPlan, thermostat.currentTemperature)

function targetTemperature(aPlan, currentTemperature) {
  // rest of function...
```

## Replace Parameter with Query
```cs
availableVacation(anEmployee, anEmployee.grade);

function availableVacation(anEmployee, grade) {
  // calculate vacation...

//------------------------------------------------------------------------

availableVacation(anEmployee)

function availableVacation(anEmployee) {
  const grade = anEmployee.grade;
  // calculate vacation...
```

## Slide Statements
```cs
const pricingPlan = retrievePricingPlan();
const order = retreiveOrder();
let charge;
const chargePerUnit = pricingPlan.unit;

//------------------------------------------------------------------------

const pricingPlan = retrievePricingPlan();
const chargePerUnit = pricingPlan.unit;
const order = retreiveOrder();
let charge;
```

## Split Loop
```cs
let averageAge = 0;
let totalSalary = 0;
for (const p of people) {
	averageAge += p.age;
	totalSalary += p.salary;
}
averageAge = averageAge / people.length;

//------------------------------------------------------------------------

let totalSalary = 0;
for (const p of people) {
	totalSalary += p.salary;
}

let averageAge = 0;
for (const p of people) {
	averageAge += p.age;
}
averageAge = averageAge / people.length;
```

## Split Phase
```cs
const orderData = orderString.split(/\s+/);
const productPrice = priceList[orderData[0].split("-")[1]];
const orderPrice = parseInt(orderData[1]) * productPrice;

//------------------------------------------------------------------------

const orderRecord = parseOrder(order);
const orderPrice = price(orderRecord, priceList);

function parseOrder(aString) {
  const values =  aString.split(/\s+/);
  return ({
    productID: values[0].split("-")[1],
    quantity: parseInt(values[1]),
  });
}
function price(order, priceList) {
  return order.quantity * priceList[order.productID];
}
```

## Split Variable
```cs
let temp = 2 * (height + width);
console.log(temp);
temp = height * width;
console.log(temp);

//------------------------------------------------------------------------

const perimeter = 2 * (height + width);
console.log(perimeter);
const area = height * width;
console.log(area);
```