#smells
В программировании проверки результата метода загромождаеют код, поскольку после вызова метода требуется дополнительный код для проверки того, является ли какой-либо код состояния допустимым, отмечен ли логический флаг или возвращаемое значение не является `None`. И все это находится за пределами области действия метода. Вместо этого, следуя принципу Fail Fast Principle, код должен выдать ошибку.
- [[Move Method]]
```python
def create_foo() -> dict:
    response = requests.get('https://api.github.com/events')
    if response.status_code != requests.codes.ok:
        return {'status_code': response.status_code, 'foo': None}
    return {
        'status_code': response.status_code,
        'foo': Foo(**response.json())
    }

foo_response: dict = create_foo()
if foo['status_code'] != requests.codes.ok:
    foo: Foo = foo_response['foo']

#---------------------------------------------------------

def create_foo() -> Foo:
    response = requests.get('https://api.github.com/events')
    if response.status_code != requests.codes.ok:
        raise Exception('Something went wrong')
    return Foo(**response.json())

foo: Foo = create_foo()
```