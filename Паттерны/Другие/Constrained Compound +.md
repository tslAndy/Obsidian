#otherpatterns
> Часто нужно определить некоторое количество поведений только в терминах маленького набора атомарных операций которые могут быть свободно скомбинированны.

Рассмотрим UI-фреймворк, который отрисовывает виджеты. Каждый виджет должен быть отрисован разным способом, но при этом используются только несколько примитивных  операций: текст, линии, фигуры. Мы хотим, чтобы реализации виджетов имели доступ только к этим базовым операциям, и не имели доступ ко всей кодовой базе.

Как разработчик API, есть две вещи, которые вы хотите предоставить клиентам: набор базовых операций и песочница для их комбинации. 

> Дайте клиенту возможность определить функцию, которая будет иметь доступ к объкту, содержащему операции, необходимые для реализации функции. 


## Операции как protected-методы
```cs
public abstract class Widget
{
    // The sandbox.
    public abstract void Draw();

    protected void DrawString(string text, int x, int y) { /* */ }
    protected void DrawLine(int x1, int y1, int x2, int y2) { /* */ }
    protected void DrawRect(int x, int y, int w, int h) { /* */ }
}

public class MyWidget : Widget
{
    public override void Draw()
    {
        DrawString("MyWidget", 5, 3);
        DrawRect(0, 0, 200, 20);

        for (int x = 0; x < 200; x += 5)
        {
            DrawLine(x, 15, x, 20);
        }
    }
}
```

Если использовать этот подход, мы быстро столкнемся с ситуацией, когда базовый класс не будет иметь достаточно контекстной информации для предоставления атомарных операций. Эта информация должна быть передана, но вы не хотите, чтобы песочница имела к ней доступ. 

```cs
public abstract class Widget
{
    // The external public API.
    public void Draw(DrawContext context)
    {
        // Store what we need to implement the atomic operations.
        mContext = context;

        // But don't give it to the sandbox.
        DrawInternal();

        // Done with it.
        mContext = null;
    }

    // The sandbox.
    protected abstract void DrawInternal();

    protected void DrawString(string text, int x, int y) { /* */ }
    protected void DrawLine(int x1, int y1, int x2, int y2) { /* */ }
    protected void DrawRect(int x, int y, int w, int h) { /* */ }

    private DrawContext mContext;
}
```

## Атомарные операции на параметре
```cs
public class Widget
{
    public Widget(Action<DrawOperations> drawFunc)
    {
        mDrawFunc = drawFunc;
    }

    public void Draw(DrawContext context)
    {
        mDrawFunc(new DrawOperations(context));
    }

    private class DrawOperations
    {
        public DrawOperations(DrawContext context)
        {
            mContext = context;
        }

        public void DrawString(string text, int x, int y) { /* */ }
        public void DrawLine(int x1, int y1, int x2, int y2) { /* */ }
        public void DrawRect(int x, int y, int w, int h) { /* */ }

        private DrawContext mContext;

    }

    private Action<DrawOperations> mDrawFunc;
}

public Widget MakeMyWidget()
{
    Widget widget = new Widget(
        operations =>
        {
            operations.DrawString("MyWidget", 5, 3);
            operations.DrawRect(0, 0, 200, 20);

            for (int x = 0; x < 200; x += 5)
            {
                operations.DrawLine(x, 15, x, 20);
            }
        });
    }

    return widget;
}
```