#smells 
Классы должны знать друг о друге как можно меньше. Сначала два класса могли переплетаться, но со временем они соединились. Разработчики не могут повторно использовать взаимосвязанные классы изолированно.

- [[Move Method]]
- [[Move Field]]
- Incapsulate Field
- [[Replace Inheritance with Delegation]]
- [[Change Bidirectional Association to Unidirectional]]

```python
# BAD
@dataclass
class Commit:
    name: str

    def push(self, repo: Repo):
        repo.push(self.name)

    def commit(self, url: str):
        ...

# BAD
@dataclass
class Repo:
    url: str

    def push(self, name: str):
        ...

    def commit(self, commit: Commit):
        commit.commit(self.url)
```