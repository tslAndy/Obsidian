#smells
```python
if (timer.has_expired() and not timer.is_recurrent()):
#---------------------------------------------
if (should_be_deleted(timer)):
```

```python
def cook(ready: bool, bag: list):
    if (ready):
        if (['raspberry', 'apple', 'tomato'] in bag and ['carrot', 'spinach', 'garlic'] not in bag):
            ...

#---------------------------------------------

# "ready" extracted out of the function scope
def cook(bag: list):
    def hasFruit(container: list) -> bool:
        return ['raspberry', 'apple', 'tomato'] in container
    def hasVeggie(container: list) -> bool:
        return ['carrot', 'spinach', 'garlic'] in container

    if not hasFruit(bag):
        return
    if hasVeggie(bag):
        return
    ...
```