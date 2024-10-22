#smells 
Проблема не с самим классом данных, а с любым случаем, когда внешняя функция работает только с внутренними параметрами объекта.

- [[Move Method]]
- [[Extract Method]]
- Freeze Variables

```python
@dataclass
class CommitManager:
    def update_author(commit: Commit, new_author: str):
        ...

    def update_message(commit: Commit, new_message: str):
        ...


@dataclass
class Commit:
    _author: str
    _message: str


commit_manager = CommitManager()
commit = Commit(author="Marceli Jerzyk", "Fix: Button Component styled width w/ rem (from px)")
commit_manager.update_author(commit, "Marcel Jerzyk")

#-----------------------------------------------------------------

@dataclass
class Commit:
    _author: str
    _message: str

    def set_author(self, new_author: str):
        ...

    def set_message(self, new_message: str):
        ...


commit = Commit(author="Marceli Jerzyk", "Fix: Button Component styled width w/ rem (from px)")
commit.set_author(commit, "Marcel Jerzyk")
```