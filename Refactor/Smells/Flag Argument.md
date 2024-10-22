#smells 
- Remove flag argumment
- [[Extract Method]]
```python
class Concert:
    def book(self, customer: Customer, is_premium: bool):
        if is_premium:
            ...
        else:
            ...

book(marcel, false) # ? false what

#---------------------------------------------

class Concert:
    def book_premium(self, customer: Customer):
        ...

    def book_regular(self, customer: Customer):
        ...

book_regular(marcel)
```