#otherpatterns 
Использование сеттеров может быть опасным, лучше ограничить операции записи и оставить их только для мест которые действительно имеют для этого причины. В качестве дополнительной меры можно сделать инъецируемый интерфейс только для чтения, и этот интерфейс будет содержать ссылку на объект, который может записывать. Объект, который может записывать, доступен только подклассам транзакции. 

```cs
public interface IMainPlayer
{
    int Level { get; }
    IResource Gold { get; }
}
 
public class MainPlayer : IMainPlayer
{
    public int Level { get { return _level; } }
    public IResource Gold { get { return _gold; } }
 
    public void ExecuteTransaction(MainPlayerTransaction transaction)
    {
        _injector.Inject(transaction);
        transaction.Execute(this);
        MarkDirty();
    }

    public void SetLevel(int newLevel) { _level = newLevel; }
}
 
public class UpdateAfterRoundTransaction : MainPlayerTransaction
{
    public UpdateAfterRoundTransaction(GameState gameState, string reason)
    {
        _gameState = gameState;
        _reason = reason;
    }
 
    public override void Execute(MainPlayer mainPlayer)
    {
        Debug.Log("Updating after round for reason: " + _reason);
        mainPlayer.SetLevel(_gameState.Level);
        mainPlayer.Gold.Set(_gameState.Gold);
    }
}
 
public class FinishRoundCommand : BaseCommand
{
    public bool Successful;
 
    [Inject] private IMainPlayer _mainPlayer;
    [Inject] private IBackend _backend;
 
    public IEnumerator Run(IngameStatistics statistics)
    {
        Successful = false;
 
        var eorCall = new FinishRoundCall(statistics);
        yield return _backend.Request(eorCall);
 
        var gameState = eorCall.ParseResponse();
        _mainPlayer.ExecuteTransaction(new UpdateAfterRoundTransaction(gameState, "Normal end of round response"));
        Successful = gameState.Successful;
    }
}

```