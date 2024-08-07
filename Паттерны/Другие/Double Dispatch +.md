#otherpatterns
The Double Dispatch pattern is used to achieve dynamic polymorphism based on the types of two objects involved in a method call. It allows method behavior to be different based on the combination of the runtime types of both the object on which the method is called and the object being passed as a parameter.

```cs
public abstract class GameObject {
  // Other properties and methods...

  public abstract void Collision(GameObject gameObject);
}

public class FlamingAsteroid: GameObject {
  // Other properties and methods...

  public override void Collision(GameObject gameObject) {
    gameObject.CollisionWithFlamingAsteroid(this);
  }
}

public class SpaceStationMir : GameObject {
  // Other properties and methods...

  public override void Collision(GameObject gameObject) {
    gameObject.CollisionWithSpaceStationMir(this);
  }
}

```
