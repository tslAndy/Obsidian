#otherpatterns 
Каррирование разлагает функцию, которая принимает несколько аргументов, на последовательность функций, каждая из которых принимает один аргумент. Этот метод является неотъемлемой частью функционального программирования, позволяя создавать функции более высокого порядка посредством частичного применения ее аргументов.

```cs
Func<Int32, Int32> SubtractOne = x => x - 1;
Int32 result = SubtractOne(8);


Func<Int32, Func<Int32, Int32>> SubtractOne = x => y => x - y;
Int32 result = SubtractOne(8)(8);



public static Func<TInput1, TOutput> Curry<TInput1, TOutput>(this Func<TInput1, TOutput> f)
{
    return x => f(x);
}
Func<Int32, Int32, Int32, Int32, Int32> Add = (w, x, y, z) => w + x + y + z;
Console.WriteLine(Add(1, 2, 3, 4));             // Normal call
Console.WriteLine(Add.Curry()(1)(2)(3)(4));     // Curried Call
```