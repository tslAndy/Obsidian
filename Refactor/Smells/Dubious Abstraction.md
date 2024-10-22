#smells 
Чем меньше и слаженнее интерфейсы классов, тем лучше, потому что они имеют тенденцию деградировать со временем. Они должны обеспечивать постоянный уровень абстракции. Интерфейсы функций должны быть на один уровень абстракции ниже операции, определенной в их названии. Что-то кажется сомнительным: действительно ли инструмент запрашивает это сообщение? Или это делает устройство подключения? Может быть, мне следует извлечь его?

```python
from abc import ABC  # Abstract Base Class

class Instrument(ABC):
    adapter: ConnectionAdapter

    def reset(self):
        self.write("*RST")

    def write(self, command):
        ...
```

```python
from abc import ABC

class Instrument(ABC):
    adapter: ConnectionAdapter

    def reset(self):
        self.adapter.write("*RST")
```