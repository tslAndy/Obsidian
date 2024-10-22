#smells 
```python
class Minion:
    _location: Location

    def action(self):
        ...
        if self._location.field.is_frontline():
            ...

class Location:
    field: Field

class Field:
    def is_frontline(self)
        ...

#---------------------------------------------

class Minion:
    _location: Location

    def action(self):
        ...
        if self.is_frontline():
            ...

    def is_frontline(self)
        return self._location.is_frontline()


class Location:
    _field: Field

    def is_frontline(self)
        return self._field.is_frontline()


class Field:
    def is_frontline(self)
        ...
```