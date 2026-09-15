# Lab 4

## Проєкт «Сума натуральних чисел»

У цьому завданні реалізовано рекурсивний алгоритм, що обчислює суму всіх
натуральних чисел від 1 до `N`. Використання узагальненого (generic) методу `SumN<T>`
з обмеженням `INumber<T>` дозволяє працювати як з типом `int`, так і з довшими
цілими значеннями (`long`) без дублювання логіки.

### Приклад `Program.cs`

```csharp
using System.Numerics;

static T SumN<T>(T n) where T : INumber<T>
{
    if (n == T.Zero) return T.Zero;  // Базовий випадок
    return n + SumN(n - T.One);
}

// Для цілих чисел
Console.Write("Введіть ціле число N: ");
int intN = int.Parse(Console.ReadLine()!);
Console.WriteLine($"Сума (int): {SumN(intN)}");

// Для довгих цілих
Console.Write("\nВведіть велике число N: ");
long longN = long.Parse(Console.ReadLine()!);
Console.WriteLine($"Сума (long): {SumN(longN)}");
```

### Приклад запуску

```text
$ dotnet run
Введіть ціле число N: 5
Сума (int): 15

Введіть велике число N: 1000
Сума (long): 500500
```

> **Порада:** рекурсивні виклики зменшують значення `n`, доки не буде
> досягнуто базового випадку `n == 0`. Переконайтеся, що вхідні значення не
> призводять до переповнення типу.

### Примітка про лямбда-вирази

Лямбда-вирази (`(a, b) => ...`) у C# дають змогу оголошувати невеликі
функції безпосередньо в місці виклику. Це зручно, коли компаратор або інша
допоміжна логіка потрібні один раз, але не варто засмічувати код окремими
методами. У деяких завданнях цієї лабораторної ми свідомо
використовуємо звичайні методи-порівнювачі замість лямбд, аби показати, що
передавання поведінки через делегати (`Func<...>`, `Action<...>`) працює однаково й без
додаткового синтаксису. Якщо ви впевнено почуваєтесь із лямбдами, їх можна
підставити в ті самі місця:

 
Ось ще кілька простих прикладів лямбда-виразів:

```csharp
// 1. Миттєвий розрахунок
Func<int, int, int> add = (a, b) => a + b;
Console.WriteLine(add(4, 6));  // 10

// 2. Обхід колекції
List<int> nums = [1, 2, 3];
nums.ForEach(value => Console.Write($"{value} "));

// 3. Захоплення змінних
int total = 0;
Action<int> addToTotal = value => total += value;
addToTotal(5);
addToTotal(7);
Console.WriteLine(total);  // 12

// 4. Порівняння рядків без урахування регістру
Func<string, string, bool> caseInsensitive =
    (a, b) => string.Compare(a, b, StringComparison.OrdinalIgnoreCase) < 0;
MergeSort(words, 0, words.Count - 1, caseInsensitive);
```

### Лямбда всередині методів

Лямбда можна оголошувати безпосередньо
в тілі методу.
Лямбда автоматично захоплює локальні змінні
та `this`, тому в її тілі доступні
поля об'єкта. Приклад нижче показує
локальні обчислення без виносу
допоміжних методів у простір класу:

```csharp
public class GradesBook
{
    private readonly record struct Grade(double Value, double Weight);

    private readonly List<Grade> _grades = [];

    public double Average()
    {
        Func<double> accumulateWeighted = () =>
        {
            double total = 0.0;
            foreach (var grade in _grades)
            {
                total += grade.Value * grade.Weight;
            }
            return total;
        };

        Func<List<Grade>, double> totalWeight = static grades =>
        {
            double sum = 0.0;
            foreach (var grade in grades)
            {
                sum += grade.Weight;
            }
            return sum;
        };

        double weight = totalWeight(_grades);
        return weight == 0.0 ? 0.0 : accumulateWeighted() / weight;
    }
}
```

Лямбда `accumulateWeighted` захоплює `this`,
тому має доступ до полів класу.
`totalWeight` позначена `static` і не має доступу до стану об'єкта,
тож передаємо список оцінок аргументом.
Такий підхід робить код компактним
і тримає допоміжну логіку поруч із місцем використання.
