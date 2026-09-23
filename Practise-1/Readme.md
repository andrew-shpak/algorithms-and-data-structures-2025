# Practise-1 — Складні задачі на масиви (C#)

## [IDE](https://onecompiler.com/csharp)

> Локально: створіть проєкт командою `dotnet new console`, вставте код у `Program.cs` і запустіть `dotnet run`.

**Усього: 5 балів за три задачі.** Заборонено використовувати `Array.Sort`, `Array.Reverse`, LINQ та готові колекції (`List`, `Queue`, `Stack`, `HashSet`, `Dictionary`) — усі алгоритми пишемо вручну на масивах.

---

## Завдання 1: Обертання квадратної матриці на 90° на місці

### Мета

Повернути квадратну матрицю `n × n` на 90° за годинниковою стрілкою, не створюючи другу матрицю. Дозволено лише одну тимчасову змінну для обміну елементів.

- **Бали:** 2

### Приклад `Program.cs`

```csharp
int[,] matrix =
{
    { 1, 2, 3, 4 },
    { 5, 6, 7, 8 },
    { 9, 10, 11, 12 },
    { 13, 14, 15, 16 }
};

Console.WriteLine("До:");
Print(matrix);

RotateClockwise(matrix);

Console.WriteLine("Після:");
Print(matrix);

static void RotateClockwise(int[,] matrix)
{
    // Ваш код тут
    throw new NotImplementedException();
}

static void Print(int[,] matrix)
{
    for (int i = 0; i < matrix.GetLength(0); i++)
    {
        for (int j = 0; j < matrix.GetLength(1); j++)
            Console.Write($"{matrix[i, j],4}");
        Console.WriteLine();
    }
}
```

### Приклад запуску

```text
До:
   1   2   3   4
   5   6   7   8
   9  10  11  12
  13  14  15  16
Після:
  13   9   5   1
  14  10   6   2
  15  11   7   3
  16  12   8   4
```

---

## Завдання 2: Медіана двох відсортованих масивів

### Мета

Дано два відсортовані за зростанням масиви `a` та `b` (будь-який з них може бути порожнім, але не обидва одночасно). Знайти медіану об'єднаного відсортованого масиву, **не будуючи** цей масив. Для парної загальної кількості елементів медіана — середнє арифметичне двох центральних.

- **Бали:** 2

### Приклад `Program.cs`

```csharp
Console.WriteLine($"Медіана = {Median(new[] { 1, 3 }, new[] { 2 })}");
Console.WriteLine($"Медіана = {Median(new[] { 1, 2 }, new[] { 3, 4 })}");
Console.WriteLine($"Медіана = {Median(new[] { 1, 5, 9, 13 }, new[] { 2, 3, 4, 6, 7, 8, 10, 11 })}");
Console.WriteLine($"Медіана = {Median(new int[0], new[] { 4, 8 })}");

static double Median(int[] a, int[] b)
{
    // Ваш код тут
    throw new NotImplementedException();
}
```

### Приклад запуску

```text
Медіана = 2
Медіана = 2.5
Медіана = 6.5
Медіана = 6
```

---

## Завдання 3: Перше відсутнє натуральне число

### Мета

Дано масив цілих чисел (можуть бути від'ємні, нулі та дублікати). Знайти найменше натуральне число (від `1`), якого немає в масиві.

- **Бали:** 1

### Приклад `Program.cs`

```csharp
Console.WriteLine($"Відсутнє: {FirstMissingPositive(new[] { 3, 4, -1, 1 })}");
Console.WriteLine($"Відсутнє: {FirstMissingPositive(new[] { 1, 2, 0 })}");
Console.WriteLine($"Відсутнє: {FirstMissingPositive(new[] { 7, 8, 9, 11, 12 })}");
Console.WriteLine($"Відсутнє: {FirstMissingPositive(new[] { 1, 1, 2, 2, 3 })}");

static int FirstMissingPositive(int[] numbers)
{
    // Ваш код тут
    throw new NotImplementedException();
}
```

### Приклад запуску

```text
Відсутнє: 2
Відсутнє: 3
Відсутнє: 1
Відсутнє: 4
```
