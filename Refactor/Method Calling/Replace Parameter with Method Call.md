#methodcalls 
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
