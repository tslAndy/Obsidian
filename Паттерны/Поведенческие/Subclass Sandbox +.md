#behavioral
Базовый класс предоставляет операции, которые будут подходить для большинства подклассов. По сути является инверсией Шаблонного Метода. 

```cs

class Superpower
{
	public virtual void Activate() { }

	public void Move() { 
		// ...
	}
	
	public void PlaySound() { 
		// ... 
	}
	
	public void SpawnParticles() { 
		// ... 
	}
}

```

При необходимости можно сгруппировать операции в подклассы

```cs
class SoundPlayer
{
	public void PlaySound() { }
	public void StopSound() { }
	public void SetVolume() { }
}

class Superpower
{
	private SoundPlayer _soundPlayer = new();

	public SoundPlayer soundPlayer => _soundPlayer;
}

```