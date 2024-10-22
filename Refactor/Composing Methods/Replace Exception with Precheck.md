#composing
```cs
double getValueForPeriod (int periodNumber) {
  try {
    return values[periodNumber];
  } catch (ArrayIndexOutOfBoundsException e) {
    return 0;
  }
}

//------------------------------------------------------------------------

double getValueForPeriod (int periodNumber) {
  return (periodNumber >= values.length) ? 0 : values[periodNumber];
}
```
