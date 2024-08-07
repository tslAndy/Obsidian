#other 
Допустим, мы хотим инстанциировать врагов в реальном времени, и в случае каких-то действий игрока с ними что-то происходит, например, они меняют цвет по нажатию кнопки.

```cs
public class Enemy : MonoBehaviour
{ 
   [SerializeField] private SpriteRenderer rd;
 
   public void ChangeColor(Color color)
   {
      rd.color = color;
   }
}
```

```cs
/*

These methods are slow: 
FindObjectOfType, FindObjectsOfType,
FindFirstObjectByType, FindAnyObjectByType

*/
public class Player : MonoBehaviour
{
	// will change only first enemy 
   private void OnMouseDown()
   {
      var enemy = (Enemy)FindObjectOfType(typeof(Enemy));
      ChangeEnemiesColor(enemy);
   }

    private void ChangeEnemyColor(Enemy enemy)
    {
       enemy.ChangeColor(Color.blue);
    }
}
```

```cs
// will change all enemies
private void OnMouseDown()
{
    var enemies = FindObjectsOfType<Enemy>();
    foreach (var enemy in enemies)
    {
        ChangeEnemyColor(enemy);
    }
}
```

## Static events
```cs
public class Player : MonoBehaviour
{
   public static event Action<Color> PlayerClicked;
   private void OnMouseDown()
   {
      PlayerClicked?.Invoke(Color.blue);
   }
}
```

Проблема будет в случае, если мы захотим добавить больше одного игрока. Плюс из-за того, что событие статично, нужно помнить написать код для инициализации сцены если использовать в Unity особенность Domain Reload.

```cs
public class Enemy : MonoBehaviour
{
   private void OnEnable()
   {
      Player.PlayerClicked += ChangeColor;
   }

   private void OnDisable()
   {
      Player.PlayerClicked -= ChangeColor;
   }

   [SerializeField] private SpriteRenderer rd;
 
   private void ChangeColor(Color color)
   {
      rd.color = color;
   }
}
```

## Static List
```cs
public class Enemy : MonoBehaviour
{
   [SerializeField] private SpriteRenderer rd;

   public static List<Enemy> EnemyList;
   
   private void Awake() => EnemyList.Add(this);

   private void OnDestroy() => EnemyList.Remove(this);

   public void ChangeColor(Color color) => rd.color = color;
}
```

```cs
public class Player : MonoBehaviour
{
   private void OnMouseDown()
   {
      foreach (var enemy in Enemy.EnemyList)
      {
         enemy.ChangeColor(Color.blue);
      }
   }
}
```

Здесь проблема в зависимости. Высокий уровень (игрок) зависит от низкого (враг). Любое изменение класса врага может повлиять на класс игрока. Плюс те же проблемы, что и у статических событий - это глобальные переменные, и нужно написать логику инициализации при Domain Reloading.

```cs
public class Player : MonoBehaviour
{
   private void OnMouseDown()
   {
      foreach (var enemy in Enemy.EnemyList)
      {
         enemy.ChangeColor(Color.blue);
      }
   }
}
```

## Mediator
Можно реализовать как [[Посредник -]]
- Может быть классом, содержащим статический список. Есть те же проблемы статических переменных, но, по как минимум классы игрока и врага не знают друг о друге, и изменения в одном не повлияет на другого.
- Может быть [[Синглтон]]. Список не должен быть статическим, но влечет за собой все проблемы синглтонов.
- Может быть [[Monostate +]]. Похоже на синглтон

Посредник не обязательно должен содержать список, он может иметь событие, к которому подписывается враг, а игрок вызывает через публичный метод. При этом вызов этого публичного метода можно контроллировать в классе игрока, например, выполняя разные проверки. 

## `Scriptable Object`
Идея в том, что у нас будет `Scriptable Object`, который будет содержать события игрока, и любой префаб, которому нужен будет доступ к этим событиям, будет содержать поле с этим объектом.

Их плюс в отличии от синглтонов и остального, что в отличии от статических полей, они удаляются когда они не требуются, а не остаются там всегда. 

```cs
[CreateAssetMenu(fileName = "PlayerEvents", menuName = "PlayerEvents", order = 0)]
public class PlayerEvents : ScriptableObject
{
   public event Action<Color> PlayerClicked;

   public void OnPlayerClicked(Color color) => PlayerClicked?.Invoke(color);
}
```

```cs
public class Player : MonoBehaviour
{
   [SerializeField] private PlayerEvents playerEvents;
   
   private void OnMouseDown() => playerEvents.OnPlayerClicked(Color.blue);
}
```

```cs
public class Enemy : MonoBehaviour
{
   [SerializeField] private SpriteRenderer rd;
   [SerializeField] private PlayerEvents playerEvents;

   private void Awake() => playerEvents.PlayerClicked += ChangeColor;
   private void OnDestroy() => playerEvents.PlayerClicked -= ChangeColor;

   private void ChangeColor(Color color) => rd.color = color;
}
```