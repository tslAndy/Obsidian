#composing
**У вас есть сложное для понимания выражение.**
Поместите результат выражения или его части в отдельные переменные, поясняющие суть выражения.
```cs
void RenderBanner() 
{
  if ((platform.ToUpper().IndexOf("MAC") > -1) &&
       (browser.ToUpper().IndexOf("IE") > -1) &&
        wasInitialized() && resize > 0 )
  {
    // do something
  }
}

//------------------------------------------------------------------------

void RenderBanner() 
{
  readonly bool isMacOs = platform.ToUpper().IndexOf("MAC") > -1;
  readonly bool isIE = browser.ToUpper().IndexOf("IE") > -1;
  readonly bool wasResized = resize > 0;

  if (isMacOs && isIE && wasInitialized() && wasResized) 
  {
    // do something
  }
}
```