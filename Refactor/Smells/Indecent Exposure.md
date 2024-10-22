#smells 
Ненужное раскрытие внутренних деталей — это запах кода _Indecent Exposure_ . Методы и переменные класса, работающие только с другими методами того же класса, должны быть закрытыми.

```python
@dataclass
class Counter:
    count: int = field(init=False)

    def bump(self) -> None:
        self.count += 1

counter = Counter()
counter.bump()
print(f"Count: {counter.count}")

#---------------------------------------------

@dataclass
class Counter:
    __count: int = field(init=False)

    @property
    def count() -> int:
        return self.__count

    def bump(self) -> None:
        self.__count += 1

    def reset(self) -> None:
        self.__count = 0

counter = Counter()
counter.bump()
print(f"Count: {counter.count}")
```