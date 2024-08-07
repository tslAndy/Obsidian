#other 
## Iterating over `enum`

```cs
public enum Day
{
    First = 1,
    Monday = 1,
    Tuesday = 2,
    Wednesday = 3,
    Thursday = 4,
    Friday = 5,
    Saturday = 6,
    Sunday = 7,
    Last = 7
}

public static class EnumExtensions
{
    public static Day Next(this Day day) => day == Day.Last ? Day.First : ++day;
}


// looping over Day
currentDay = currentDay.Next()

// or
var days = Enum.GetValues<Day>().Distinct();
foreach(var day in days) {}
```

## Operation State as `enum`

```cs
enum LoadFileStatus
{
    Success,
    FileNotFound,
    CorruptedData,
    OlderVersion
}

public LoadFileStatus LoadGame(SaveFile file, out SaveData data)
```

## Avoiding switch

```cs
// NOT GOOD
switch (currentDay)
{
    case Day.Monday:
        //do Monday things
        break;
    case Day.Tuesday:
        //do Tuesday things
        break;
    case Day.Wednesday:
        //do Wednesday things
        break;
    case Day.Thursday:
        //do Thursday things
        break;
    case Day.Friday:
        //do Friday things
        break;
    case Day.Saturday:
        //do Saturday things
        break;
    case Day.Sunday:
        //do Sunday things
        break;
    default:
        throw new ArgumentOutOfRangeException();
}
```

```cs
interface IDay
{
   void Morning();
   void Afternoon();
   void Evening();
   void Night();
}
```

```cs
public class Monday : IDay
{
   public void Morning()
   {
      Console.WriteLine("Doing Monday Morning things");
   }

   public void Afternoon()
   {
      Console.WriteLine("Doing Monday Afternoon things");
   }

   public void Evening()
   {
      Console.WriteLine("Doing Monday Evening things");
   }

   public void Night()
   {
      Console.WriteLine("Doing Monday Night things");
   }
}
```

```cs
Dictionary<Day, IDay> Schedule = new()
{
    {Day.Monday, new Monday()},
    {Day.Tuesday, new Tuesday()},
    {Day.Wednesday, new Wednesday()},
    {Day.Thursday, new Thursday()},
    {Day.Friday, new Friday()},
    {Day.Saturday, new Saturday()},
    {Day.Sunday, new Sunday()}
};
```

```cs
for(int i=0; i<10; i++)
{
    Schedule[currentDay].Morning();
    Schedule[currentDay].Afternoon();
    Schedule[currentDay].Evening();
    Schedule[currentDay].Night();
    
    currentDay = currentDay.Next();
}
```

```cs
// IF need to get new schedule each time
Dictionary<Day, Func<IDay>> Schedule = new()
{
    {Day.Monday, () => new Monday()},
    {Day.Tuesday, () => new Tuesday()},
    {Day.Wednesday, () => new Wednesday()},
    {Day.Thursday, () => new Thursday()},
    {Day.Friday, () => new Friday()},
    {Day.Saturday, () => new Saturday()},
    {Day.Sunday, () => new Sunday()}
};
```

## Flags

```cs
// TWO WAYS OF DEFINING
[Flags] public enum Directions
{
    None = 0,
    North = 1,
    West = 2,
    South = 4,
    East = 8
}

[Flags] public enum Directions
{
    None = 0,
    North = 1 << 0,
    West = 1 << 1,
    South = 1 << 2,
    East = 1 << 3
}
```

Flags позволяет корректно выводить значения:

```cs
AllowedDirections = Directions.East | Directions.West;
Debug.Log(AllowedDirections); // "West, East" instead of "10" 
```

Operations:

```cs
// combining
AllowedDirections = AllowedDirections | Directions.West;

// removing
AllowedDirections = AllowedDirections & ~Directions.West;

// checking
(AllowedDirections & Directions.East) == Directions.East

// or
AllowedDirections.HasFlag(Directions.East)

// equality
AllowedDirections.Equals(Directions.None)

// is defined
Console.WriteLine($"{(Directions)16} Exists: {Enum.IsDefined(typeof(Directions), (Directions)16)}");

// combined values
bool IsFlagDefined (Enum value) => !int.TryParse(value.ToString(), out _);
Console.WriteLine($"{(Directions)15} Exists: {IsFlagDefined((Directions)15)}");
```

## Тип enum
```cs
public enum Directions: byte // int, char, etc
{
    None = 0,
    North = 1,
    West = 2,
    South = 4,
    East = 8
}
```

## Некоторые оптимизации
- Вызовы `ToString()`, `GetNames()`, `IsDefined()` основаны на рефлексии, и эти значения не кэшируются.
- Парсинг значений основан на `String.Split()`, что не особо эффективно

При возможности можно кэшировать результаты методов, например получить список значений энама и хранить его в статическом поле. Другой вариант это использовать метод-помощник:

```cs
public static class GameStateExtensions
{
    public static string FastToString(this GameState gameState)
    {
        switch (gameState)
        {
            case GameState.Running: return "Running";
            case GameState.Paused: return "Paused";
            case GameState.GameOver: return "GameOver";
            case GameState.MainMenu: return "MainMenu";
            default: return "[Unknown]";
        }
    }
}
```