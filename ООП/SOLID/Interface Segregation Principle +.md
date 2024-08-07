#oop
Относится к тем случаям, когда классы имеют "жирный интерфейс", то есть слишком раздутый интерфейс, не все методы и свойства которого используются и могут быть востребованы. Клиенты не должны вынужденно зависеть от методов, которыми не пользуются.

Техники для выявления нарушения этого принципа:
- Слишком большие интерфейсы
- Компоненты в интерфейсах слабо согласованы (перекликается с [[Single Responsibility Principle +]])
- Методы без реализации (перекликается с [[Liskov Substitution Principle +]])

В этом случае интерфейс класса разделяется на отдельные части, которые составляют раздельные интерфейсы. Затем эти интерфейсы независимо друг от друга могут применяться и изменяться. В итоге применение принципа разделения интерфейсов делает систему слабосвязанной, и тем самым ее легче модифицировать и обновлять.

Рассмотрим на примере. Допустим у нас есть интерфейс отправки сообщения:
```cs
interface IMessage
{
    void Send();
    string Text { get; set;}
    string Subject { get; set;}
    string ToAddress { get; set; }
    string FromAddress { get; set; }
}
```

Класс `EmailMessage` выглядит целостно:

```cs
class EmailMessage : IMessage
{
    public string Subject { get; set; } = "";
    public string Text { get; set; } = "";
    public string FromAddress { get; set; } = "";
    public string ToAddress { get; set; } = "";
 
    public void Send() => Console.WriteLine($"Отправляем Email-сообщение: {Text}");
}
```

Здесь мы уже сталкиваемся с небольшой проблемой: свойство `Subject`, которое определяет тему сообщения, при отправке смс не указывается, поэтому оно в данном классе не нужно. Таким образом, в классе `SmsMessage` появляется избыточная функциональность, от которой класс `SmsMessage` начинает зависеть.

```cs
class SmsMessage : IMessage
{
    public string Text { get; set; } = "";
    public string FromAddress { get; set; } = "";
    public string ToAddress { get; set; } = "";
 
    public string Subject
    {
        get
        {
            throw new NotImplementedException();
        }
 
        set
        {
            throw new NotImplementedException();
        }
    }
 
    public void Send() => Console.WriteLine($"Отправляем Sms-сообщение: {Text}");
}
```

Допустим, нам надо создать класс для отправки голосовых сообщений. Класс голосовой почты также имеет отправителя и получателя, только само сообщение передается в виде звука, что на уровне C# можно выразить в виде массива байтов. И в этом случае было бы неплохо, если бы интерфейс `IMessage` включал бы в себя дополнительные свойства и методы для этого, например:

```cs
interface IMessage
{
    void Send();
    string Text { get; set;}
    string ToAddress { get; set; }
    string Subject { get; set; }
    string FromAddress { get; set; }
 
    byte[] Voice { get; set; }
}
```

Тогда класс голосовой почты `VoiceMessage` мог бы выглядеть следующим образом:

```cs
class VoiceMessage : IMessage
{
    public string ToAddress { get; set; } = "";
    public string FromAddress { get; set; } = "";
    public byte[] Voice { get; set; } = new byte[] {};
 
    public string Text
    {
        get
        {
            throw new NotImplementedException();
        }
 
        set
        {
            throw new NotImplementedException();
        }
    }
 
    public string Subject
    {
        get
        {
            throw new NotImplementedException();
        }
 
        set
        {
            throw new NotImplementedException();
        }
    }
 
    public void Send() => Console.WriteLine("Передача голосовой почты");
}
```

И здесь опять же мы сталкиваемся с ненужными свойствами. Плюс нам надо добавить новое свойство в предыдущие классы `SmsMessage` и `EmailMessage`, причем этим классам свойство Voice в принципе то не нужно. В итоге здесь мы сталкиваемся с явным нарушением принципа разделения интерфейсов.

Для решения возникшей проблемы нам надо выделить из классов группы связанных методов и свойств и определить для каждой группы свой интерфейс:

```cs
interface IMessage
{
    void Send();
    string ToAddress { get; set; }
    string FromAddress { get; set; }
}
interface IVoiceMessage : IMessage
{
    byte[] Voice { get; set; }
}
interface ITextMessage : IMessage
{
    string Text { get; set; }
}
 
interface IEmailMessage : ITextMessage
{
    string Subject { get; set; }
}
 
class VoiceMessage : IVoiceMessage
{
    public string ToAddress { get; set; } = "";
    public string FromAddress { get; set; } = "";
 
    public byte[] Voice { get; set; } = Array.Empty<byte>(); 
    public void Send() => Console.WriteLine("Передача голосовой почты");
}
class EmailMessage : IEmailMessage
{
    public string Text { get; set; } = "";
    public string Subject { get; set; } = "";
    public string FromAddress { get; set; } = "";
    public string ToAddress { get; set; } = "";
 
    public void Send() => Console.WriteLine("Отправляем по Email сообщение: {Text}");
}
 
class SmsMessage : ITextMessage
{
    public string Text { get; set; } = "";
    public string FromAddress { get; set; } = "";
    public string ToAddress { get; set; } = "";
    public void Send() => Console.WriteLine("Отправляем по Sms сообщение: {Text}");
}
```

## Пустые методы
Одним из типичных нарушений данного принципа являются нереализованные методы. Подобные методы обычно говорят о том, что в проектировании системы есть недостатки и упущения. Кроме того, нереализованные методы нередко также нарушают принцип Лисков.

```cs
interface IPhone
{
    void Call();
    void TakePhoto();
    void MakeVideo();
    void BrowseInternet();
}
class Phone : IPhone
{
    public void Call() => Console.WriteLine("Звоним");
     
    public void TakePhoto() => Console.WriteLine("Фотографируем");
     
    public void MakeVideo() => Console.WriteLine("Снимаем видео");
     
    public void BrowseInternet() => Console.WriteLine("Серфим в интернете");
}
```

Пусть у нас есть также класс клиента, который использует объект Phone для фотографирования:

```cs
class Photograph
{
    public void TakePhoto(Phone phone)
    {
        phone.TakePhoto();
    }
}


Photograph photograph = new Photograph();
Phone myPhone = new Phone();
photograph.TakePhoto(myPhone);
```

Объект `Photograph`, который представляет фотографа, теперь может фотографировать, используя объект `Phone`. Однако более для фотосъемки можно использовать также и обычную фотокамеру, которая не обладает множеством возможностей телефона. И мы хотели бы, чтобы фотограф мог бы также использовать и фотокамеру для фотосъемки. В этом случае мы могли взять общий интерфейс `IPhone` и реализовать его метод `TakePhoto` в классе фотокамеры:

```cs
class Camera : IPhone
{
    public void Call() { }
    public void TakePhoto()
    {
        Console.WriteLine("Фотографируем");
    }
    public void MakeVideo() { }
    public void BrowseInternet() { }
}
```

Однако здесь мы опять сталкиваемся с тем, что клиент - класс `Camera` зависит от методов, которые он не использует - то есть методов `Call`, `MakeVideo`, `BrowseInternet`

Для решения возникшей задачи мы можем воспользоваться принципом разделения интерфейсов:

```cs
interface ICall
{
    void Call();
}
interface IPhoto
{
    void TakePhoto();
}
interface IVideo
{
    void MakeVideo();
}
interface IWeb
{
    void BrowseInternet();
}
 
class Camera : IPhoto
{
    public void TakePhoto()
    {
        Console.WriteLine("Фотографируем с помощью фотокамеры");
    }
}
 
class Phone : ICall, IPhoto, IVideo, IWeb
{
    public void Call()
    {
        Console.WriteLine("Звоним");
    }
    public void TakePhoto()
    {
        Console.WriteLine("Фотографируем с помощью смартфона");
    }
    public void MakeVideo()
    {
        Console.WriteLine("Снимаем видео");
    }
    public void BrowseInternet()
    {
        Console.WriteLine("Серфим в интернете");
    }
}


class Photograph
{
    public void TakePhoto(IPhoto photoMaker)
    {
        photoMaker.TakePhoto();
    }
}
```

## Другое
Принцип разделения интерфейсов решает проблему типов, которые могут делать разные вещи. Некоторые классы, хотя и представляют собой единую сущность, которая что-то представляет, могут делать вещи, которые концептуально можно сгруппировать в разные интерфейсы.

Например, класс «Персонаж» может иметь методы для перемещения персонажа, методы, отвечающие за то, чтобы персонаж атаковал, и методы, отвечающие за обработку урона, который персонаж получает при атаке.

Мы можем поддаться искушению создать такой класс, чтобы создать интерфейс `ICharacter`, который имеет все публичные методы, которые `Character`класс должен будет реализовать. Но это неправильно. Как было сказано в [[Концептуальное значение интерфейсов]] интерфейс концептуально принадлежит классу или системе, которые его используют, а не классу, который его реализует.

## Знание только того, что вам нужно
Предположим, что у нас есть система, которая отвечает за движение, система, которая отвечает за атаки, и некоторые системы, которые наносят урон. Например, в игре персонажи, будь то персонаж игрока или враги, могут быть не единственными сущностями, которые могут получать урон. Дверь может получать урон от ударов, а дерево — от огня.

Все, что наносит урон, не должно знать, что представляет собой сущность, которая получает урон. Ему нужно знать только то, что может сделать сущность. То же самое относится к системам, которые отвечают за создание движения и атак.

Даже если в нашей игре только персонажи могут получать урон, эта потребность все равно остается. Система, которая наносит урон, не должна зависеть от методов, которые никогда не будут использоваться, например, методов, которые отвечают за движение.

## Пример, который использует ISP
Предположим, что интерфейс `ICharacter` выглядит так:
```cs
public interface ICharacter
{
   void MoveUp();
   void MoveDown();
   void MoveRight();
   void MoveLeft();

   void ApplyDamage();
   void OnDeath();

   void Attack();
   void GetTarget();
}
```

Это нарушение ISP. У нас есть два разных способа справиться с этим, чтобы наш код соответствовал ISP. В обоих случаях первое, что нам нужно сделать, это разделить интерфейс `ICharacter` на три интерфейса:

```cs
public interface IMovable
{
   void MoveUp();
   void MoveDown();
   void MoveRight();
   void MoveLeft();
}

public interface IAttack
{
   void Attack();
   void GetTarget();
}

public interface IDamageable
{
   void ApplyDamage();
   void OnDeath();
}
```

Менее распространенный способ, который также добавляет сложности нашему коду, — это композиция. Наш класс `Character` может иметь поле `IMovable`, которое создает объект, реализующий интерфейс `IMovable`. Этот объект будет иметь ссылку на класс `Character` и будет действовать как [[Адаптер +]]. Каждый из четырех методов перемещения интерфейса будет вызывать соответствующие методы `Character`, которые будут перемещать нашего персонажа. Этот способ не только более сложен, но и менее производительен, так как нам придется каждый раз создавать новый объект типа `IMovable`. Это решение полезно, если создаваемый нами объект должен действовать как адаптер, потому что разные классы будут иметь разные методы, которые по-разному отвечают за движение в нашей программе. Например, персонаж игрока сразу же движется вправо, но NPC должен продолжать двигаться до первого перекрестка, где он повернет направо.

Самый распространенный способ, который также является самым простым, заключается в том, что наш класс `Character` будет реализовывать все эти три интерфейса напрямую вместо интерфейса `ICharacter`:
```cs
public class Character : IMovable, IAttack, IDamageable
{
	// ...
}
```

Т. е. мы перешли от этого:

![[Interface_segregation.webp]]

К этому:

![[Interface_segregation2.webp]]

## Общие методы и общие объекты
Иногда мы можем оказаться в ситуации, когда метод нужен в двух или более интерфейсах. Это нормально, мы можем включить этот метод во все интерфейсы, которым он нужен. Если по какой-то причине мы обнаруживаем, что многие методы являются общими для всех интерфейсов, то нам нужно сделать шаг назад и подумать, должны ли эти методы быть в их собственном интерфейсе. Концептуально эти методы могут представлять что-то еще, что не описывается интерфейсами, которые мы создали.

Нередко бывает так, что системе может потребоваться два или более наших интерфейсов для функционирования. Например, наша атакующая система может быть реализована таким образом, что ей необходимо переместить атакующих в нужную позицию с помощью нашей системы движения, прежде чем она сможет начать атаку. Для этого есть два решения.

Первый — передать один и тот же объект дважды. Например, если у нас есть метод, `InitiateAttack` который вызывается с двумя аргументами, `IMovable` и `IAttack`, мы можем вызвать его так: `InitiateAttack(enemyCharacter, enemyCharacter)`. Это может показаться странным, но на самом деле мы не передаем один и тот же тип дважды. Для метода мы передаем два разных типа, an `IMovable`и an `IAttack`, даже если они представлены одним и тем же объектом.

Хотя первое решение приемлемо, необходимость передавать один и тот же объект два или три раза является признаком того, что в архитектуре кода что-то упущено, поэтому есть второе решение:

## Добавление слоя между интерфейсами и классами, которые их используют
В приведенном выше примере, когда вражеский персонаж должен вести себя и как `IMovable` и как `IAttack`для нашей атакующей системы, что это означает?

На самом деле это вражеский атакующий. В приведенном выше примере ISP первым решением было создание адаптера и его использование с композицией. Вражеский атакующий подходит под критерии такого адаптера.

```cs
public class Character : IDamageable
{
   public IMovable Movable { get; }
   public IAttack Attacker { get; }

   public Character(IMovable movable, IAttack attacker)
   {
      Movable = movable;
      Attacker = attacker;
   }

   public void ApplyDamage() { }
   public void OnDeath() { }
}
```

Здесь мы соблюдаем разделение интерфейсов через делегирование. Затем создаем интерфейс `IEnemyAttacker`и класс `EnemyAttacker`:
```cs
public interface IEnemyAttacker
{
   void Attack();
   // plus other needed methods here
}

public class EnemyAttacker : IEnemyAttacker
{
   private IMovable _movable;
   private IAttack _attack;

   public EnemyAttacker(IMovable movable, IAttack attack)
   {
      _movable = movable;
      _attack = attack;
   }

   public void Attack() { }
   // plus other needed methods here
}
```

Здесь важно то, что класс `EnemyAttacker` действует как адаптер. Какие бы методы ни были реализованы в этом классе, они будут использовать методы `IMovable` и `IAttack`для создания поведения.

Теперь, если мы создали персонажа, мы можем создать вражеского атакующего всякий раз, когда этот объект нужен для нашей системы атаки. Например:

```cs
IMovable movable = new Movable();
IAttack attack = new Attacker();

Character character = new Character(movable, attack);
AttackSystem attackSystem = new AttackSystem();

IEnemyAttacker enemyAttacker = new EnemyAttacker(character.Movable, character.Attacker);
attackSystem.InitiateAttack(enemyAttacker);
```

Это, очевидно, сложнее, но показывает необходимость создания представления чего-либо, когда нам нужно передать один и тот же объект в систему два или более раз как другой тип.

Когда мы оказываемся в ситуациях, когда один и тот же объект передается как разные типы, нам следует остановиться и подумать, действительно ли комбинация этих типов представляет собой что-то, чего мы не реализовали в нашем коде.
