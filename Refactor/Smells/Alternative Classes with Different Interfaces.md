#smells
- [[Rename Method]]
- [[Move Method]]
- [[Extract Superclass]]
```python
class Snowman(Humanoid):
    def hug_snowman():
        ...

class Zombie(Humanoid):
    def hug_zombie():
        ...

#---------------------------------------------

class Snowman(Humanoid):
    def hug():
        ...

class Zombie(Humanoid):
    def hug():
        ...
```
