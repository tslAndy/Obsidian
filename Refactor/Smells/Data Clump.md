#smells 
- [[Extract Class]]
- Introduce Parameter Object

```python
def colorize(red: int, green: int, blue: int):
    ...

#--------------------

@dataclass(frozen=True)
class RGB:
    red: int
    green: int
    blue: int


def colorize(rgb: RGB):
    ...
```