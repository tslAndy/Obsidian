#smells 
Если метод внутри класса манипулирует большим количеством функций (будь то поля или методы) другого класса больше, чем из своего собственного. 
- [[Move Method]]
- [[Move Field]]
- [[Extract Method]]

```python
@dataclass(frozen=True)
class ShoppingItem:
    name: str
    price: float
    tax: float


class Order:
    ...
    def get_bill_total(self, items: list[ShoppingItem]) -> float:
        return sum([item.price * item.tax for item in items])

    def get_receipt_string(self, items: list[ShoppingItem]) -> list[str]:
        return [f"{item.name}: {item.price * item.tax}$" for item in items]

    def create_receipt(self, items: list[ShoppingItem]) -> float:
        bill = self.get_bill_total(items)
        receipt = self.get_receipt_string(items).join('\n')
        return f"{receipt}\nBill {bill}"

#--------------------------------------------------

@dataclass(frozen=True)
class ShoppingItem:
    name: str
    price: float
    tax: float

    @property
    def taxed_price(self) -> float:
        return self.price * self.tax

    def get_receipt_string(self) -> str:
        return f"{self.name}: {self.price * self.tax}$"

class Order:
    ...
    def get_bill_total(items: list[ShoppingItem]) -> float:
        return sum([item.taxed_price for item in items])

    def get_receipt_string(items: list[ShoppingItem]) -> list[str]:
        return [item.get_receipt_string() for item in items]

    def create_receipt(items: list[ShoppingItem]) -> float:
        bill = self.get_bill_total(items)
        receipt = self.get_receipt_string(items).join('\n')
        return f"{receipt}\nBill: {bill}$"
```