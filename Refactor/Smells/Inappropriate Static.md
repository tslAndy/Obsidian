#smells 
В случае сомнений следует предпочесть нестатические методы статическим. Лучший способ проверить, должен ли метод быть статическим, — подумать, должен ли метод вести себя полиморфно. Использование статических функций и переменных затрудняет тестирование кода; требует создания макетов.

```python
class FooUtils:
    @staticmethod
    def wrap_tag_premium(foo: Foo):
        return f"[PREMIUM]: {foo.action()}"

    @staticmethod
    def wrap_tag_special(foo: Foo):
        return f"[SPECIAL]: {foo.action()}"

#------------------------------------------

@dataclass
class FooTagWrapper:
    foo: Foo

    def wrap_tag_premium(self):
        return f"[PREMIUM]: {self.foo}"

    def wrap_tag_special(self):
        return f"[SPECIAL]: {self.foo}"
```