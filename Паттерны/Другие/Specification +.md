#otherpatterns
```cs
public interface ISpecification<in T>
{
  bool IsSatisfied(T obj);
}
```

```cs
public class In21CenturySpecification : ISpecification<DateTime>
{
  private readonly DateTime _start = new DateTime(2001, 01, 01);
  private readonly DateTime _end = new DateTime(2101, 01, 01);

  public bool IsSatisfied(DateTime obj)
  {
    bool result = obj >= _start && obj < _end;
    return result;
  }
}
```

```cs
var specification = new In21CenturySpecification();
Movie movie = _movieService.GetRandomMovie();
bool isIn21Century = specification.IsSatisfied(movie.ReleaseDate);
```
