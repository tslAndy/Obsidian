#smells 
- [[Replace Parameter with Query]]
- [[Preserve Whole Object]]
- Introduce Parameter Object
- Remove [[Flag Argument]]
- Combine methods into class


```python
def foo(author: str, commit_id: str, files: List[str], sha_id: str, time: str):
    ...

author, commit_id, files, sha_id, time = get_last_commit()
foo(author, commit_id, files, sha_id, time)

#---------------------------------------------

@dataclass(frozen=True)
class Commit:
    author: str
    commit_id: str
    files: List[str]
    sha_id: str
    time: str

    def foo(self):
        ...

commit = Commit(**get_last_commit())
commit.foo()
```