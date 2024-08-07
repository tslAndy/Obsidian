#oop
Иногда мы можем оказаться в ситуации, когда нам нужно знать тип дочернего класса, который вызвал метод. Например, у нас может быть метод копирования в родителе, который возвращает копию нашего объекта, которая наследуется дочерними классами. Мы хотим, чтобы этот метод возвращал копию, которая является типом дочернего объекта, а не типом родителя.

Рассмотрим пример без CRT:

```cs
public abstract class Unit
{
   public abstract int AttackDamage { get; set; }
   public abstract void Attack();
   public abstract Unit MirrorImage();
}
```

```cs
public class SmallUnit : Unit
{
   public sealed override int AttackDamage { get; set; }
   public sealed override void Attack() => Console.WriteLine($"I am a small unit that attacked for {AttackDamage}");
   public sealed override Unit MirrorImage() => new SmallUnit { AttackDamage = AttackDamage };
}

public class BigUnit : Unit
{
   public sealed override int AttackDamage { get; set; }
   public int SpellDamage { get; set; }
   public sealed override void Attack() => Console.WriteLine($"I am a big unit that attacked for {AttackDamage}");
   public sealed override Unit MirrorImage() =>
      new BigUnit
      {
         AttackDamage = AttackDamage,
         SpellDamage = SpellDamage
      };
   public void CastSpell() => Console.WriteLine($"I am a big unit that cast a spell for {SpellDamage}");
}

```

```cs
Unit unit= new BigUnit
{
   AttackDamage = 5,
   SpellDamage = 6
};
Unit unit2= new SmallUnit 
{ AttackDamage = 5 };
```

```cs
var mirror = unit.MirrorImage();
```

Единственный способ вызвать метод `CastSpell` это:

```cs
(mirror as BigUnit)?.CastSpell();
```

Теперь пример с CRT:

```cs
public abstract class UnitCrtp<T> where T : UnitCrtp<T>
{
   public abstract int AttackDamage { get; set; }
   
   public abstract void Attack();
   
   public T MirrorImage() => Copy();
   protected abstract T Copy();
}
```

```cs
public class SmallUnit2 : UnitCrtp<SmallUnit2>
{
   public sealed override int AttackDamage { get; set; }
   public sealed override void Attack() => Console.WriteLine($"I am a small unit that attacked for {AttackDamage}");
   protected sealed override SmallUnit2 Copy() => new() { AttackDamage = AttackDamage };
}

public class BigUnit2 : UnitCrtp<BigUnit2> 
{
   public sealed override int AttackDamage { get; set; }
   public int SpellDamage { get; set; }

   public sealed override void Attack() => Console.WriteLine($"I am a big unit that that attacked for {AttackDamage}");
   protected sealed override BigUnit2 Copy() =>
      new()
      {
         AttackDamage = AttackDamage,
         SpellDamage = SpellDamage
      };
   public void CastSpell() => Console.WriteLine($"I am a big unit that cast a spell for {SpellDamage}");
}
```

```cs
UnitCrtp<BigUnit2> bigUnitCrtp= new BigUnit2
{
   AttackDamage = 5,
   SpellDamage = 6
};
UnitCrtp<SmallUnit2> smallUnitCrtp= new SmallUnit2 { AttackDamage = 5 };

var mirrorCrtp = bigUnitCrtp.MirrorImage();
mirrorCrtp.CastSpell();
```

При этом важно не перепутать парамметр возвращаемого объекта, он должен быть таким же, как сам класс, а не как в примере ниже:

```cs
public class SmallUnit3 : UnitCrtp<BigUnit2>
{
   public sealed override int AttackDamage { get; set; }
   public sealed override void Attack() => Console.WriteLine($"I am a small unit that attacked for {AttackDamage}");
   protected override BigUnit2 Copy() =>  new() { AttackDamage = AttackDamage };
}
```