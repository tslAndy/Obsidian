#dataorg
**В коде используется число, которое несёт какой-то определённый смысл.**
Замените это число константой с человеко-читаемым названием, объясняющим смысл этого числа.
```cs
double PotentialEnergy(double mass, double height) 
{
  return mass * height * 9.81;
}

//------------------------------------------------------------------------

const double GRAVITATIONAL_CONSTANT = 9.81;

double PotentialEnergy(double mass, double height) 
{
  return mass * height * GRAVITATIONAL_CONSTANT;
}
```
