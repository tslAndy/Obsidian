#methodcalls 
**Метод разбит на части, каждая из которых выполняется в зависимости от значения какого-то параметра.**
Извлеките отдельные части метода в собственные методы и вызывайте их вместо оригинального метода.

```cs
void SetValue(string name, int value) 
{
  if (name.Equals("height")) 
  {
    height = value;
    return;
  }
  if (name.Equals("width")) 
  {
    width = value;
    return;
  }
  Assert.Fail();
}

//------------------------------------------------------------------------

void SetHeight(int arg) 
{
  height = arg;
}
void SetWidth(int arg) 
{
  width = arg;
}
```
