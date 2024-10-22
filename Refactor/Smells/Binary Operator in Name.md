#smells
Имена методов или функций, которые имеют бинарные побитовые операторы, такие как `AND`и `OR` являются очевидными кандидатами на роль явных нарушителей [[Single Responsibility Principle +]]. 
- [[Extract Method]]

```python
def render_and_save():
    # render logic
    ...
    # save logic
    ...
    
#--------------------------------

def render():
    # render logic
    ...

def save():
    # save logic
    ...
```
