#composing
**Вы хотите заменить существующий алгоритм другим?**
Замените тело метода, реализующего старый алгоритм, новым алгоритмом.
```cs
string FoundPerson(string[] people)
{
  for (int i = 0; i < people.Length; i++) 
  {
    if (people[i].Equals("Don"))
    {
      return "Don";
    }
    if (people[i].Equals("John"))
    {
      return "John";
    }
    if (people[i].Equals("Kent"))
    {
      return "Kent";
    }
  }
  return String.Empty;
}

//------------------------------------------------------------------------

string FoundPerson(string[] people)
{
  List<string> candidates = new List<string>() {"Don", "John", "Kent"};
  
  for (int i = 0; i < people.Length; i++) 
  {
    if (candidates.Contains(people[i])) 
    {
      return people[i];
    }
  }
  
  return String.Empty;
}
```
