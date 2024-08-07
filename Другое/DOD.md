#other 
Предположим что есть класс, и определенная функция часто меняет значения:

```cs
public class Enemy
{
   Vector3 m_position; // <-------------------
   string m_name;
   int m_type;
   int m_hp;
   float m_shieldRadius;
   int m_currentShieldStrength;
   int m_armorType;
   int m_maxShieldStrength;
   int m_armorStrength;
   Vector3 m_direction; // <-------------------
   float m_dodge;
   int m_primaryWeaponType;
   int m_secondaryWeaponType;
   int m_primaryWeaponDamage;
   int m_secondaryWeaponDamage;
   int m_primaryWepaonAmmo;
   int m_secondaryWeaponAmmo;
   float m_shieldRechargeTime;
   float m_velocity; // <-----------------------
   int m_armorType;
   //etc…
   
   public void Move()
   {
      m_position += m_direction * m_velocity;
   }
}
```

В этом случае лучше расположить переменные друг за другом чтобы оптимизировать промахи кэша. 

При новой итерации всеравно остается одие промах кэша, так как нужно снова получить позицию, и при этом в кэшэ находятся значения всех остальных переменных.
Можно вынести эти переменные в отдельный класс, чтобы за каждым классом `EnemyMove` следовал следующий. 

```cs
public class EnemyMove
{
   Vector3 m_position;
   Vector3 m_direction;
   float m_velocity;

   public void Move()
   {
      m_position += m_direction * m_velocity;
   }
}
```

При этом учитываем, что при размещении ссылочных типов в массиве хранятся только адреса элементов в куче, и нет никаких гарантий, что они будут расположены в памяти последовательно. 

```cs
// this class will hold all the data needed by enemies
public class EnemyData()
{
   public int NumEnemies; // how many enemies do we have?
   public Vector3[] Position; // position data for all enemies
   public Vector3[] Direction; // direction data for all enemies
   public float[] Velocity; // velocity data for all enemies
   // other enemy variables
   public float[] ShieldRadius;
   public float[] HP;
   public int[] PrimaryWeaponType;


	public static MoveAllEnemies(EnemyData enemyData)
	{
	   for(int i = 0; i < enemyData.NumEnemies; i++)
	      enemyData.Position[i] += enemyData.Direction[i] * enemyData.Velocity[i];
	}
}
```