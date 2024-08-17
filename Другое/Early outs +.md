#other 
Есть два типа - early outs и single point of return.

Single Point Of Return:
```cs
void TryAttack()
{
  FacingData facingData = PrepareFacingData();
  if (TestFacing(facingData))
  {
    WeaponData weaponData = PrepareWeaponData();
    if (TestWeaponReady(weaponData))
    {
      PathData pathData = PareparePathData();
      if (TestPathClear(pathData))
      {
        Attack();
      }
    }
  }
  // return here
}
```

Early Outs:
```cs
void TryAttack()
{
  FacingData facingData = PrepareFacingData();
  if (!TestFacing(facingData))
    return;
 
  WeaponData weaponData = PrepareWeaponData();
  if (!TestWeaponReady(weaponData))
    return;
 
  PathData pathData = PareparePathData();
  if (!TestPathClear(pathData))
    return;
 
  Attack();
}


for (Characer &c : characters)
{
  FacingData facingData = ParepareFacingData(c);
  if (!TestFacing(facingData))
    continue;
 
  WeaponData weaponData = PrepareWeaponData(c);
  if (!TestWeaponReady(weaponData))
    continue;
 
  PathData pathData = PareparePathData(c);
  if (!TestPathClear(pathData))
    continue;
 
  charactersWhoCanAttack.Add(c);
}
```

Допустим, мы хотим отрефакторить следующую функцию:
```cs
void TryAttack()
{
  FacingData facingData = PrepareFacingData();
  if (TestFacing(facingData))
  {
    WeaponData weaponData = PrepareWeaponData();
    if (TestWeaponReady(weaponData))
    {
      PathData pathData = PareparePathData();
      if (TestPathClear(pathData))
      {
        Attack();
      }
    }
  }
 
  // always execute
  PostAttackTry();
}
```

Если использовать early out'ы, то получим следующий код:
```cs
void TryAttack()
{
  FacingData facingData = PrepareFacingData();
  if (!TestFacing(facingData))
  {
    PostAttackTry();
    return;
  }
 
  WeaponData weaponData = PrepareWeaponData();
  if (!TestWeaponReady(weaponData))
  {
    PostAttackTry();
    return;
  }
 
  PathData pathData = PareparePathData();
  if (!TestPathClear(pathData))
  {
    PostAttackTry();
    return;
  }
 
  Attack();
 
  PostAttackTry();
}
```

Что выглядит плохо. Лучше вынести  проверки в отдельную функцию:

```cs
bool CanAttack()
{
  FacingData facingData = PrepareFacingData();
  if (!TestFacing(facingData))
    return false;
 
  WeaponData weaponData = PrepareWeaponData();
  if (!TestWeaponReady(weaponData))
    return false;
 
  PathData pathData = PareparePathData();
  if (!TestPathClear(pathData))
    return false;
 
  return true;
}
 
void TryAttack()
{
  if (CanAttack())
    Attack();
 
  PostAttackTry();
}
```

Допустим, что мы хотим отладить код. Здесь проблема в том, что нам придется переходить к вызовам `TestFacing(facingData)`, `TestWeaponReady(weaponData)`, `TestPathClear(pathData)`, что будет занимать дополнительное время. 

```cs
void TryAttack()
{
  FacingData facingData = PrepareFacingData();
  if (!TestFacing(facingData).IsFacingValid())
    return;
 
  WeaponData weaponData = PrepareWeaponData();
  if (!TestWeaponReady(weaponData).IsWeaponReady())
    return;
 
  PathData pathData = PareparePathData();
  if (!TestPathClear(pathData).IsPathClear())
    return;
 
  Attack();
}
```

Перепишем следующим образом:
```cs
void TryAttack()
{
  FacingData facingData = PrepareFacingData();
  FacingResults facingResults = TestFacing(facingData);
  if (!facingResults.IsFacingValid())
    return;
 
  WeaponData weaponData = PrepareWeaponData();
  WeaponResults weaponResults = TestWeaponReady(weaponData);
  if (!weaponResults.IsWeaponReady())
    return;
 
  PathData pathData = PareparePathData();
  PathResults pathResults = TestPathClear(pathData);
  if (!pathResults.IsPathClear())
    return;
 
  Attack();
}
```