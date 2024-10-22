#composing
**У вас есть локальная переменная, которая используется для хранения разных промежуточных значений внутри метода (за исключением переменных циклов).**
Используйте разные переменные для разных значений. Каждая переменная должна отвечать только за одну определённую вещь.
```cs
double temp = 2 * (height + width);
Console.WriteLine(temp);
temp = height * width;
Console.WriteLine(temp);

//------------------------------------------------------------------------

readonly double perimeter = 2 * (height + width);
Console.WriteLine(perimeter);
readonly double area = height * width;
Console.WriteLine(area);
```