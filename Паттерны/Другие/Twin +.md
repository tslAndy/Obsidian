#otherpatterns
Provides a way to handle multiple, related classes in a manner that allows them to work together without inheriting from a common base class. It provides a way to form two closely coupled subclasses that can act as a twin class having two ends.

- Use when you need to decouple classes that share common functionality but cannot inherit from a common base class due to various reasons such as the use of different frameworks or languages.
- Useful in performance-critical applications where inheritance might introduce unnecessary overhead.
- Applicable in systems requiring resilience through the ability to replace or update one of the twins without affecting the other.


```cs
using System.Threading;
using Unity.VisualScripting.Antlr3.Runtime;
using UnityEngine;

public abstract class GameItem
{
    public void Draw()
    {
        Debug.Log("draw");
        DoDraw();
    }

    public abstract void DoDraw();
    public abstract void Click();
}



public class BallItem : GameItem
{
    private bool isSuspended;

    private BallThread twin;

    public void SetTwin(BallThread thread) => twin = thread;

    public override void DoDraw()
    {
        Debug.Log("do draw");
    }

    public override void Click()
    {
        isSuspended = !isSuspended;
        if (isSuspended)
            twin.SuspendMe();
        else
            twin.ResumeMe();
    }

    public void Move() => Debug.Log("move");
}


public class BallThread
{
    private BallItem twin;
    public void SetTwin(BallItem item) => twin = item;

    private volatile bool isSuspended;
    private volatile bool isRunning = true;

    public void Run()
    {
        while (isRunning)
        {
            if (!isSuspended)
            {
                twin.Draw();
                twin.Move();
            }

            Thread.Sleep(250);
        }
    }

    public void SuspendMe()
    {
        isSuspended = true;
    }

    public void ResumeMe()
    {
        isSuspended = false;
    }

    public void StopMe()
    {
        isRunning = false;
        isSuspended = true;
    }
}

class Test
{
    public void RunTest()
    {
        var ballItem = new BallItem();
        var ballThread = new BallThread();

        ballItem.SetTwin(ballThread);
        ballThread.SetTwin(ballItem);

        ballItem.Click();
        ballItem.Click();
        ballThread.StopMe();
    }
}

```