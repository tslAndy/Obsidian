#smells 
- Remove Setting Method
- Choose Proper Access Control
- [[Separate Query from Modifier]]
- [[Change Reference to Value]]
- [[Replace Derived Variable with Query]]
- [[Extract Method]]
- Encapsulate Variable
- Combine Methods into Class

```python

@dataclass
class Foo:
    name: str
    value: float
    premium: bool

# foo object instance will be passed around and modified

#---------------------------------------------

@dataclass(frozen=True)
class Foo:
    name: str
    value: float
    premium: bool

# foo object instance will be passed around and, if that is necessary,
# recreated, thus making the state change explicit for everyone and handled
```