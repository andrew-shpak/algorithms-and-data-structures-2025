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

## Завдання 2: Найбільший квадрат з одиниць у матриці

### Мета

Дано матрицю `n × m`, заповнену нулями та одиницями. Знайти найбільший квадрат, що складається лише з одиниць, і повернути довжину його сторони та координати лівого верхнього кута. Якщо в матриці немає жодної одиниці — сторона дорівнює `0`.

- **Бали:** 2

### Приклад `Program.cs`

```csharp
int[,] matrix =
{
    { 1, 0, 1, 0, 0 },
    { 1, 0, 1, 1, 1 },
    { 1, 1, 1, 1, 1 },
    { 1, 0, 0, 1, 0 }
};

(int side, int row, int col) = LargestSquare(matrix);
Console.WriteLine($"Сторона = {side}, лівий верхній кут = [{row}, {col}]");

int[,] empty =
{
    { 0, 0 },
    { 0, 0 }
};
(side, row, col) = LargestSquare(empty);
Console.WriteLine($"Сторона = {side}, лівий верхній кут = [{row}, {col}]");

static (int Side, int Row, int Col) LargestSquare(int[,] matrix)
{
    // Ваш код тут
    throw new NotImplementedException();
}
```

### Приклад запуску

```text
Сторона = 2, лівий верхній кут = [1, 2]
Сторона = 0, лівий верхній кут = [-1, -1]
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
