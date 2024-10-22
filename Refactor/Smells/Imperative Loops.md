#smells 
В отличие от конвейеров, циклы не обеспечивают декларативной читаемости того, что именно обрабатывается.

```python
const examples = ["foo", "bar", "baz"]
for (let idx = 0; idx < examples.length; idx++) {
  console.log(examples[idx])
}

#-----------------------------------------

const examples = ["foo", "bar", "baz"]
examples.forEach((example) => console.log(example))
```