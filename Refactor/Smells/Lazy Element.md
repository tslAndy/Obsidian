#smells 
Ленивый элемент — это элемент, который не делает достаточно. Если метод, переменная или класс не делает достаточно, чтобы окупить себя (из-за повышенной сложности проекта), его следует объединить в другую сущность.

- [[Inline Class]]
-  [[Inline Method]]
- [[Collapse Hierarchy]]

```python
class Strength:
    value: int

class Person:
    health: int
    intelligence: int
    strength: Strength
    
#---------------------------------------------

class Person:
    health: int
    intelligence: int
    strength: int
```