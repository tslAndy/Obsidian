#smells 
```python
class Human:
    def talk():
        ...

class Elf:
    def chat():
        ...
        
#---------------------------------------------

class Character(ABC):
    @abstractmethod
    def talk():
        """ Converse """

class Human(Character):
    def talk():
        ...

class Elf(Character):
    def talk():
        ...
```