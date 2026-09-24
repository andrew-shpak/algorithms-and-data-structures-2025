# Лекція 5 — Прості вправи на сортування

[Матеріал лекції](Examples.md)

Почніть із перевірки порядку й окремих кроків, потім переходьте до повних алгоритмів. Пишіть на C#, задавайте масиви в коді. У завданнях 1–6 та 8–10 реалізуйте кроки самостійно; готове сортування дозволено лише там, де це прямо вказано.

**Усього: 12 завдань із повними реалізаціями та очікуваним виводом.**

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

## Завдання 5. Сортування вибором короткого ряду

На кожній позиції знайдіть найменший елемент решти масиву та поміняйте його з поточним. Реалізуйте сортування вибором без готового сортування. Повторні числа та порожній масив допустимі.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

foreach (int[] values in new[] { new[] { 4, 1, 3, 1 }, new[] { 7 }, Array.Empty<int>() })
{
    for (int i = 0; i < values.Length; i++)
    {
        int best = i;
        for (int j = i + 1; j < values.Length; j++)
            if (values[j] < values[best]) best = j;
        (values[i], values[best]) = (values[best], values[i]);
    }
    Console.WriteLine($"[{string.Join(", ", values)}]");
}
```

**Очікуваний вивід:**

```text
[1, 1, 3, 4]
[7]
[]
```

</details>

## Завдання 6. Бульбашка з ранньою зупинкою

Виконуйте бульбашкові проходи, доки масив не впорядкований. Якщо за прохід не було обмінів, завершіть алгоритм. Поверніть кількість виконаних проходів; для нуля або одного елемента вона дорівнює нулю.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

foreach (int[] values in new[] { new[] { 3, 1, 2 }, new[] { 1, 2, 3 }, Array.Empty<int>() })
{
    int passes = BubbleSort(values);
    Console.WriteLine($"[{string.Join(", ", values)}]; passes={passes}");
}

static int BubbleSort(int[] values)
{
    int passes = 0;
    for (int end = values.Length - 1; end > 0; end--)
    {
        bool changed = false;
        for (int i = 0; i < end; i++)
            if (values[i] > values[i + 1])
            {
                (values[i], values[i + 1]) = (values[i + 1], values[i]);
                changed = true;
            }
        passes++;
        if (!changed) break;
    }
    return passes;
}
```

**Очікуваний вивід:**

```text
[1, 2, 3]; passes=2
[1, 2, 3]; passes=1
[]; passes=0
```

</details>

## Завдання 7. Коротші слова першими

Відсортуйте малі латинські слова за довжиною, а однакової довжини — порядком `StringComparer.Ordinal`. Тут дозволено `Array.Sort` із власним порівнянням. Порожній рядок повинен стати першим.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

string[] words = { "pear", "a", "fig", "bee", "" };
Array.Sort(words, (a, b) =>
{
    int byLength = a.Length.CompareTo(b.Length);
    return byLength != 0 ? byLength : StringComparer.Ordinal.Compare(a, b);
});
foreach (string word in words) Console.WriteLine($"[{word}]");
```

**Очікуваний вивід:**

```text
[]
[a]
[bee]
[fig]
[pear]
```

</details>

## Завдання 8. Квадрати впорядкованих чисел

Вхідний масив уже впорядкований за неспаданням, але може містити від’ємні числа. Побудуйте впорядкований масив квадратів без сортування: порівнюйте квадрати крайніх елементів і заповнюйте результат із кінця. Значення у межах `-1000..1000`.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

foreach (int[] values in new[] { new[] { -4, -1, 0, 3 }, new[] { -3, -2 }, Array.Empty<int>() })
    Console.WriteLine($"[{string.Join(", ", SortedSquares(values))}]");

static int[] SortedSquares(int[] values)
{
    int[] result = new int[values.Length];
    int left = 0, right = values.Length - 1;
    for (int i = result.Length - 1; i >= 0; i--)
    {
        int a = values[left] * values[left];
        int b = values[right] * values[right];
        if (a > b) { result[i] = a; left++; }
        else { result[i] = b; right--; }
    }
    return result;
}
```

**Очікуваний вивід:**

```text
[0, 1, 9, 16]
[4, 9]
[]
```

</details>

## Завдання 9. k-та цифра без сортування

Масив містить лише цифри `0..9`. Знайдіть значення на позиції `k` у відсортованому порядку, нумерація від одиниці. Повтори займають окремі позиції. Для некоректного `k` поверніть `null`. Використайте десять лічильників.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

int[] digits = { 8, 1, 4, 1, 9 };
foreach (int k in new[] { 1, 2, 3, 6 })
    Console.WriteLine(KthDigit(digits, k)?.ToString() ?? "NONE");

static int? KthDigit(int[] digits, int k)
{
    if (k < 1 || k > digits.Length) return null;
    int[] count = new int[10];
    foreach (int digit in digits) count[digit]++;
    for (int digit = 0; digit < count.Length; digit++)
    {
        k -= count[digit];
        if (k <= 0) return digit;
    }
    return null;
}
```

**Очікуваний вивід:**

```text
1
1
4
NONE
```

</details>

## Завдання 10. Скільки пар порушують порядок

Порахуйте інверсії: пари індексів `i < j`, для яких `values[i] > values[j]`. Рівні числа інверсією не є. Для короткого масиву використайте два цикли, не змінюючи вхід.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

foreach (int[] values in new[] { new[] { 3, 1, 2 }, new[] { 2, 2, 1 }, new[] { 1, 2, 3 }, Array.Empty<int>() })
{
    long inversions = 0;
    for (int i = 0; i < values.Length; i++)
        for (int j = i + 1; j < values.Length; j++)
            if (values[i] > values[j]) inversions++;
    Console.WriteLine(inversions);
}
```

**Очікуваний вивід:**

```text
2
2
0
0
```

</details>

## Завдання 11. Об’єднати перекриті відрізки

Для замкнених відрізків `(start, end)`, де `start ≤ end`, об’єднайте ті, що перетинаються або мають спільний кінець. Тут дозволено сортування за початком. Виведіть неперекриті відрізки зліва направо.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

Console.WriteLine(string.Join(", ", Merge(new[] { (5, 7), (1, 3), (3, 4), (8, 9) })));
Console.WriteLine($"Count={Merge(Array.Empty<(int, int)>()).Count}");

static List<(int Start, int End)> Merge((int Start, int End)[] intervals)
{
    var sorted = ((int Start, int End)[])intervals.Clone();
    Array.Sort(sorted, (a, b) => a.Start.CompareTo(b.Start));
    var result = new List<(int Start, int End)>();
    foreach (var interval in sorted)
    {
        if (result.Count == 0 || interval.Start > result[^1].End)
            result.Add(interval);
        else
            result[^1] = (result[^1].Start, Math.Max(result[^1].End, interval.End));
    }
    return result;
}
```

**Очікуваний вивід:**

```text
(1, 4), (5, 7), (8, 9)
Count=0
```

</details>

## Завдання 12. Перевірити стабільність готового результату

Картки мають числовий ключ і унікальну літерну назву. Дано початкові картки та їх перестановку. Перевірте, що результат впорядкований за ключем і рівні ключі зберегли початковий порядок. Гарантується, що перестановка містить ті самі картки.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

(int Key, string Id)[] original = { (2, "A"), (1, "B"), (2, "C"), (1, "D") };
Console.WriteLine(IsStable(original, new[] { original[1], original[3], original[0], original[2] }));
Console.WriteLine(IsStable(original, new[] { original[3], original[1], original[0], original[2] }));
Console.WriteLine(IsStable(Array.Empty<(int, string)>(), Array.Empty<(int, string)>()));

static bool IsStable((int Key, string Id)[] original, (int Key, string Id)[] sorted)
{
    var position = new Dictionary<string, int>();
    for (int i = 0; i < original.Length; i++) position[original[i].Id] = i;
    for (int i = 1; i < sorted.Length; i++)
    {
        var before = sorted[i - 1];
        var after = sorted[i];
        if (before.Key > after.Key) return false;
        if (before.Key == after.Key && position[before.Id] > position[after.Id]) return false;
    }
    return true;
}
```

**Очікуваний вивід:**

```text
True
False
True
```

</details>
