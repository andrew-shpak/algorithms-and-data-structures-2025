# Лекція 5 — Прості вправи перед повним сортуванням

[Матеріал лекції](Examples.md)

Кожна вправа тренує одну властивість або один крок алгоритму. Пишіть на C#, задавайте масиви в коді. `Array.Sort`, `List.Sort` і LINQ-сортування в цих вправах не використовуйте.

> **Запуск реалізацій:** C# 14 / .NET 10. Встановіть .NET 10 SDK. Створіть консольний проєкт через `dotnet new console --framework net10.0`, замініть `Program.cs` одним повним блоком `csharp` і виконайте `dotnet run`. Кожен блок незалежний; класи й методи з інших завдань копіювати не потрібно. Для порожніх результатів у виводі використовуємо `[]`; логічні значення C# друкує як `True` / `False`.

## Завдання 1. Чи вже є порядок

Метод `IsSorted(int[] values)` повертає `true`, якщо числа розташовані за неспаданням: кожне наступне не менше за попереднє. Рівні сусідні значення дозволені.

**Вимоги:** один прохід без зміни масиву. Порожній масив і масив з одного елемента вважайте впорядкованими.

| Масив | Результат |
|---|---|
| `[1, 2, 2, 5]` | `true` |
| `[1, 4, 3]` | `false` |
| `[7]` | `true` |
| `[]` | `true` |

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

int[][] examples = { new[] { 1, 2, 2, 5 }, new[] { 1, 4, 3 }, new[] { 7 }, Array.Empty<int>() };
foreach (int[] values in examples)
{
    Console.WriteLine(IsSorted(values));
}

static bool IsSorted(int[] values)
{
    for (int i = 1; i < values.Length; i++)
    {
        if (values[i] < values[i - 1])
        {
            return false;
        }
    }
    return true;
}
```

**Очікуваний вивід:**

```text
True
False
True
True
```

</details>

## Завдання 2. Один прохід бульбашки

Пройдіть масив зліва направо: порівнюйте елементи за індексами `i` та `i + 1`, міняйте їх місцями, якщо лівий більший. Виконайте **рівно один прохід**, виведіть масив і кількість обмінів.

| Початковий масив | Після проходу | Обміни |
|---|---|---|
| `[5, 1, 4, 2]` | `[1, 4, 2, 5]` | `3` |
| `[1, 2, 3]` | `[1, 2, 3]` | `0` |
| `[2, 2, 1]` | `[2, 1, 2]` | `1` |
| `[]` | `[]` | `0` |

**Питання:** чому найбільший елемент уже в кінці, хоча весь масив ще може бути невпорядкованим?

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

int[][] examples = { new[] { 5, 1, 4, 2 }, new[] { 1, 2, 3 }, new[] { 2, 2, 1 }, Array.Empty<int>() };
foreach (int[] values in examples)
{
    int swaps = 0;
    for (int i = 0; i + 1 < values.Length; i++)
    {
        if (values[i] > values[i + 1])
        {
            (values[i], values[i + 1]) = (values[i + 1], values[i]);
            swaps++;
        }
    }
    Console.WriteLine($"[{string.Join(", ", values)}]; swaps={swaps}");
}
```

**Очікуваний вивід:**

```text
[1, 4, 2, 5]; swaps=3
[1, 2, 3]; swaps=0
[2, 1, 2]; swaps=1
[]; swaps=0
```

</details>

## Завдання 3. Чотири розміри ґудзиків

Кожен ґудзик має розмір `0`, `1`, `2` або `3`. Побудуйте відсортований за зростанням масив за допомогою масиву лічильників із чотирьох елементів.

**Вимоги:** спочатку порахуйте ґудзики кожного розміру, потім заповніть результат. Не порівнюйте ґудзики попарно. Усі вхідні розміри належать діапазону `0..3`.

| Вхід | Результат |
|---|---|
| `[3, 0, 2, 3, 1, 0]` | `[0, 0, 1, 2, 3, 3]` |
| `[2, 2]` | `[2, 2]` |
| `[]` | `[]` |

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

int[][] examples = { new[] { 3, 0, 2, 3, 1, 0 }, new[] { 2, 2 }, Array.Empty<int>() };
foreach (int[] sizes in examples)
{
    Console.WriteLine($"[{string.Join(", ", SortButtons(sizes))}]");
}

static int[] SortButtons(int[] sizes)
{
    int[] counts = new int[4];
    foreach (int size in sizes)
    {
        counts[size]++;
    }
    int[] result = new int[sizes.Length];
    int index = 0;
    for (int size = 0; size < counts.Length; size++)
    {
        for (int copy = 0; copy < counts[size]; copy++)
        {
            result[index++] = size;
        }
    }
    return result;
}
```

**Очікуваний вивід:**

```text
[0, 0, 1, 2, 3, 3]
[2, 2]
[]
```

</details>

## Завдання 4. Стабільно розкласти на дві групи

Створіть новий список: спочатку всі парні числа, потім усі непарні. **Усередині кожної групи збережіть початковий порядок**; числове сортування не потрібне.

**Вимоги:** зробіть два проходи по вхідному масиву. Перевіряйте парність через `x % 2 == 0`; це працює також для від'ємних чисел.

| Вхід | Результат |
|---|---|
| `[5, 2, 7, 4, 3, 6]` | `[2, 4, 6, 5, 7, 3]` |
| `[-3, -2, 0, 1]` | `[-2, 0, -3, 1]` |
| `[9, 1, 7]` | `[9, 1, 7]` |
| `[]` | `[]` |

**Самоперевірка:** поясніть на першому прикладі, чим збереження порядку всередині груп відрізняється від звичайного сортування всіх чисел.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

int[][] examples =
{
    new[] { 5, 2, 7, 4, 3, 6 }, new[] { -3, -2, 0, 1 },
    new[] { 9, 1, 7 }, Array.Empty<int>()
};
foreach (int[] values in examples)
{
    var result = new List<int>();
    foreach (int value in values)
    {
        if (value % 2 == 0)
        {
            result.Add(value);
        }
    }
    foreach (int value in values)
    {
        if (value % 2 != 0)
        {
            result.Add(value);
        }
    }
    Console.WriteLine($"[{string.Join(", ", result)}]");
}
```

**Очікуваний вивід:**

```text
[2, 4, 6, 5, 7, 3]
[-2, 0, -3, 1]
[9, 1, 7]
[]
```

</details>
