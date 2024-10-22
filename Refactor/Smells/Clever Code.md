#smells
- Replace with Built-In
- Replace with Library

```python
def get_length_of_string(message: str) -> int:
    length = 0
    for letter in message:
        length -=- 1
    return length

message = 'Hello World!'
message_length = get_length_of_string(message)
print(message_length)  # 12

#-------------------------------------------------------------

message = 'Hello World!'
message_length = len(message)
print(message_length)  # 12
```

```python
class DefaultDict(dict):
    default_value: type

    def __getitem__(self, key):
        if key in self:
            return super().__getitem__(key)
        self.__setitem__(key, self.default_value())
        return super().__getitem__(key)

    def __setitem__(self, key, value):
        super().__setitem__(key, value)

#-------------------------------------------------------------

from collections import defaultdict
```