#other 
Instead of:
```cs
void Update()
{
    if (_direction == Direction.Left)
    {
        // Left code
        Debug.Log("Going left");

        if (Input.GetKeyDown(KeyCode.S))
        {
            // lot of if key S got pressed code
            _direction = Direction.Right;
        }
    }
    else if (_direction == Direction.Right)
    {
        // Right code
        Debug.Log("Going right");

        if (Input.GetKeyDown(KeyCode.A))
        {
            // lot of if key A got pressed code
            _direction = Direction.Left;
        }
    }
}
```

Use:
```cs
void Update()
{
   if (Input.GetKeyDown(KeyCode.S))
   {
      // lot of if key S got pressed code
      _direction = Direction.Right;
   }
   else if (Input.GetKeyDown(KeyCode.A))
   {
      // lot of if key A got pressed code
      _direction = Direction.Left;
   }
   
   if (_direction == Direction.Left)
   {
      // Left code
      Debug.Log("Going left");
   }
   else if (_direction == Direction.Right)
   {
      // Right code
      Debug.Log("Going right");
   }
}
```

Or:

```cs
void Update()
{
   _direction = GetDirection();
   ChangeDirection(_direction);
}

Direction GetDirection()
{
   if (Input.GetKeyDown(KeyCode.S))
   {
      // lot of if key S got pressed code
      return Direction.Right;
   }
   if (Input.GetKeyDown(KeyCode.A))
   {
      // lot of if key A got pressed code
      return  Direction.Left;
   }
   return _direction;
}

void ChangeDirection(Direction direction)
{
   if (_direction == Direction.Left)
   {
      // Left code
      Debug.Log("Going left");
   }
   else if (_direction == Direction.Right)
   {
      // Right code
      Debug.Log("Going right");
   }
}
```