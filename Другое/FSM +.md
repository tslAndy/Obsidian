#other 
```cs
public interface IPredicate
{
    bool Evaluate();
}

public class FuncPredicate : IPredicate
{
    private readonly Func<bool> func;

    public FuncPredicate(Func<bool> func)
    {
        this.func = func;
    }

    public bool Evaluate() => func.Invoke();
}
```

```cs
public interface ITransition
{
    IState To { get; }
    IPredicate Condition { get; }
}


public class Transition : ITransition
{
    public IState To { get; }
    public IPredicate Condition { get; }

    public Transition(IState to, IPredicate condition)
    {
        To = to;
        Condition = condition;
    }
}
```

```cs
public interface IState
{
    void OnEnter();
    void Update();
    void FixedUpdate();
    void OnExit();
}


public abstract class BaseState : IState
{
    protected readonly PlayerController player;
    protected readonly Animator animator;

    protected static readonly int LocomotionHash = Animator.StringToHash("locomotion");
    protected static readonly int JumpHash = Animator.StringToHash("Jump");
    protected static readonly float crossFadeDuration = 0.25f;


    protected BaseState(PlayerController player, Animator animator)
    {
        this.player = player;
        this.animator = animator;
    }

    public virtual void FixedUpdate() { }

    public virtual void OnEnter() { }

    public virtual void OnExit() { }

    public virtual void Update() { }
}
```

```cs
public class JumpState : BaseState
{
    public JumpState(PlayerController player, Animator animator) : base(player, animator)
    {
    }

    public override void OnEnter()
    {
        animator.CrossFade(JumpHash, crossFadeDuration);
    }

    public override void FixedUpdate()
    {
        player.HandleJump();
        player.HandleMovement();
    }
}


public class LocomotionState : BaseState
{
    public LocomotionState(PlayerController player, Animator animator) : base(player, animator)
    {
    }

    public override void OnEnter()
    {
        animator.CrossFade(LocomotionHash, crossFadeDuration);
    }

    public override void FixedUpdate()
    {
        player.HandleMovement();

    }
}
```

```cs
public class StateMachine
{
    private StateNode current;
    private Dictionary<Type, StateNode> nodes = new();
    private HashSet<ITransition> anyTransitions = new();

    public void Update()
    {
        var transition = GetTransition();
        if (transition is not null)
            ChangeState(transition.To);
        current.State?.Update();
    }

    public void FixedUpdate()
    {
        current.State?.FixedUpdate();
    }

    public void SetState(IState state)
    {
        current = nodes[state.GetType()];
        current.State.OnEnter();
    }

    private void ChangeState(IState state)
    {
        if (state == current.State) return;

        var previousState = current.State;
        var nextState = nodes[state.GetType()].State;

        previousState?.OnExit();
        nextState?.OnEnter();
        current = nodes[state.GetType()];
    }

    private ITransition GetTransition()
    {
        foreach (var transition in anyTransitions)
            if (transition.Condition.Evaluate())
                return transition;

        foreach (var transition in current.Transitions)
            if (transition.Condition.Evaluate())
                return transition;

        return null;
    }

    public void AddTransition(IState from, IState to, IPredicate condition)
    {
        GetOrAddNode(from).AddTransition(GetOrAddNode(to).State, condition);
    }

    public void AddAnyTransition(IState to, IPredicate condition)
    {
        anyTransitions.Add(new Transition(GetOrAddNode(to).State, condition));
    }

    private StateNode GetOrAddNode(IState state)
    {
        var node = nodes.GetValueOrDefault(state.GetType());
        if (node == null)
        {
            node = new StateNode(state);
            nodes.Add(state.GetType(), node);
        }
        return node;
    }


    class StateNode
    {
        public IState State { get; }
        public HashSet<ITransition> Transitions { get; }

        public StateNode(IState state)
        {
            State = state;
            Transitions = new();
        }

        public void AddTransition(IState to, IPredicate condition)
        {
            Transitions.Add(new Transition(to, condition));
        }
    }
}
```

```cs
public class PlayerController
{
    private Animator animator;


    private StateMachine stateMachine;

    private void Awake()
    {
        animator = GetComponent<Animator>();

        stateMachine = new();

        // declare states
        var locomotionState = new LocomotionState(this, animator);
        var jumpState = new JumpState(this, animator);

        // define transitions
        At(locomotionState, jumpState, new FuncPredicate(() => jumpTimer.IsRunning));
        At(jumpState, locomotionState, new FuncPredicate(() => groundChecker.IsGrounded && !jumpTimer.IsRunning));

        // set initial state
        stateMachine.SetState(locomotionState);
    }

    private void At(IState from, IState to, IPredicate condition) => stateMachine.AddTransition(from, to, condition);
    private void Any(IState to, IPredicate condition) => stateMachine.AddAnyTransition(to, condition);

    private void Update()
    {
        stateMachine.Update();
    }

    private void FixedUpdate()
    {
        stateMachine.FixedUpdate();
    }

    public void HandleJump() { }
    public void HandleMovement() { }
}
```