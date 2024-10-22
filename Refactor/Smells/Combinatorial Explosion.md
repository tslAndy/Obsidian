#smells
```python
class Minion:
    name: str
    state: 'ready'

    def action(self):
        if self.state == 'ready':
            self.animate('standing')
        elif self.state == 'fighting':
            self.animate('fighting')
        elif self.state == 'resting':
            self.animate('resting')

    def next_state(self):
        if self.state == 'ready':
            return 'fighting'
        elif self.state == 'fighting':
            return 'resting'
        elif self.state == 'resting':
            return 'ready'

    def animate(self, animation: str):
        print(f"{self.name} is {animation}!")

#---------------------------------------------

class State(ABC):
    @abstractmethod
    def next() -> State:
        """ Return next State """

    @abstractmethod
    def animate() -> str:
        """ Returns a text-based animation """

class Ready(State):
    def next():
        return States.FIGHT

    def animate():
        return 'standing'

class Fight(State):
    def next():
        return States.REST

    def animate():
        return 'fighting'

class Rest(State):
    def next():
        return States.READY

    def animate():
        return 'resting'

class States(Enum):
    READY: State = Ready
    FIGHT: State = Fight
    REST: State = Rest

class Minion:
    name: str
    state: State = States.Ready

    def action(self):
        self.state.animate()

    def next_state(self):
        self.state = self.state.next()

    def animate(self, animation: str):
        print(f"{self.name} is {self.state.animate()}!")
```
