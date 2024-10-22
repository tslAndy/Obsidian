#smells 
Скрытая зависимость — это ситуация, когда внутри класса методы молча разрешают зависимости, скрывая это поведение от вызывающего. _Скрытые зависимости_ могут вызвать исключение времени выполнения, когда вызывающий не настроил соответствующую среду заранее.

```python
class Customer:
    pass

customer = Customer()

class Cart:
    def __init__(self):
        self.customer = customer  # gets customer from global scope

cart = Cart()

#--------------------------------------

class Customer:
    pass

customer = Customer()

class Cart:
    def __init__(self, customer):
        self.customer = customer  # gets customer explicitly

cart = Cart(customer)
```