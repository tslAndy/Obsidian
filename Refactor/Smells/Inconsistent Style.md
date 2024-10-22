```python

# BAD
my_first_function(
    arg1=1,
    arg2=2,
    arg3=3
)
my_second_function(arg1=1,
                   arg2=2,
                   arg3=3)
my_third_function(
    arg1=1, arg2=2, arg3=3)


# ALSO BAD
class Character:
    DAMAGE_BONUS: float

    def rangeAttack(self, enemy: Character, damage: int, extra_damage: int):
        total_damage = damage + extra_damage*self.DAMAGE_BONUS
        ...

    def meleeAttack(self, enemy: Character, extra_damage: int, damage: int):
        total_damage = damage + extra_damage*self.DAMAGE_BONUS
        ...

witcher.rangeAttack(skeleton, 300, 200)
witcher.meleeAttack(skeleton, 300, 200)  # potentially overlooked error
```