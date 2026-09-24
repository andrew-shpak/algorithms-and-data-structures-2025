# Лекція 9 — Прості задачі на жадібний вибір і динамічне програмування

[Матеріал лекції](README.md)

Реалізації нижче написані на C#. Дані задавайте в коді. Для перших двох задач поясніть свій жадібний вибір, для решти — значення комірки `dp` і початкові значення.

**Усього: 12 завдань із повними реалізаціями та очікуваним виводом.**

> **Запуск реалізацій:** C# 14 / .NET 10. Встановіть .NET 10 SDK. Створіть консольний проєкт через `dotnet new console --framework net10.0`, замініть `Program.cs` одним повним блоком `csharp` і виконайте `dotnet run`. Кожен блок незалежний; класи й методи з інших завдань копіювати не потрібно. Для порожніх результатів у виводі використовуємо `[]`; логічні значення C# друкує як `True` / `False`.

## Завдання 1. Найбільше наборів наліпок за бюджет

Маємо список додатних цілих цін і невід'ємний бюджет. Кожен набір можна купити не більше одного разу. Знайдіть найбільшу **кількість** наборів, яку можна придбати, не перевищивши бюджет.

**Вимоги:** беріть найдешевші набори першими; сортувати копію цін готовим методом дозволено. Однакові ціни можуть належати різним наборам.

| Ціни; бюджет | Кількість |
|---|---|
| `[4, 2, 7, 1]; 7` | `3` |
| `[3, 3, 3]; 6` | `2` |
| `[5, 8]; 4` | `0` |
| `[]; 10` | `0` |

**Питання:** чому заміна дорожчого придбаного набору дешевшим не зменшує кількість покупок?

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

Console.WriteLine(MaxSets(new[] { 4, 2, 7, 1 }, 7));
Console.WriteLine(MaxSets(new[] { 3, 3, 3 }, 6));
Console.WriteLine(MaxSets(new[] { 5, 8 }, 4));
Console.WriteLine(MaxSets(Array.Empty<int>(), 10));

static int MaxSets(int[] prices, int budget)
{
    int[] sorted = (int[])prices.Clone();
    Array.Sort(sorted);
    int count = 0;
    foreach (int price in sorted)
    {
        if (price > budget)
        {
            break;
        }
        budget -= price;
        count++;
    }
    return count;
}
```

**Очікуваний вивід:**

```text
3
2
0
0
```

</details>

## Завдання 2. Короткі сеанси на одному приладі

Кожен сеанс задано парою `(start, end)`, де `start < end`. Оберіть найбільшу кількість сеансів без перетинів. Сеанс може починатися саме тоді, коли попередній закінчується.

**Вимоги:** упорядкуйте сеанси за часом завершення; за однакового завершення — за початком. Виберіть перший, а далі додавайте сеанс, якщо його початок не раніший за кінець останнього вибраного.

| Сеанси | Вибрані сеанси | Кількість |
|---|---|---|
| `[(1, 3), (2, 5), (3, 4), (4, 6)]` | `[(1, 3), (3, 4), (4, 6)]` | `3` |
| `[(0, 10), (1, 2), (2, 3)]` | `[(1, 2), (2, 3)]` | `2` |
| `[]` | `[]` | `0` |

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

(int Start, int End)[][] examples =
{
    new[] { (1, 3), (2, 5), (3, 4), (4, 6) },
    new[] { (0, 10), (1, 2), (2, 3) },
    Array.Empty<(int, int)>()
};
foreach (var sessions in examples)
{
    var selected = SelectSessions(sessions);
    Console.WriteLine($"[{string.Join(", ", selected)}]; Count={selected.Count}");
}

static List<(int Start, int End)> SelectSessions((int Start, int End)[] sessions)
{
    var sorted = ((int Start, int End)[])sessions.Clone();
    Array.Sort(sorted, (a, b) =>
    {
        int byEnd = a.End.CompareTo(b.End);
        return byEnd != 0 ? byEnd : a.Start.CompareTo(b.Start);
    });
    var result = new List<(int Start, int End)>();
    foreach (var session in sorted)
    {
        if (result.Count == 0 || session.Start >= result[^1].End)
        {
            result.Add(session);
        }
    }
    return result;
}
```

**Очікуваний вивід:**

```text
[(1, 3), (3, 4), (4, 6)]; Count=3
[(1, 2), (2, 3)]; Count=2
[]; Count=0
```

</details>

## Завдання 3. Сходи з кроками на одну або дві сходинки

Знайдіть кількість способів піднятися на `n` сходинок, роблячи крок на одну або дві (`0 ≤ n ≤ 30`). Різний порядок кроків дає різні способи. Для `n = 0` є один спосіб — не робити кроків.

**Вимоги:** побудуйте таблицю знизу вгору. `dp[i]` — кількість способів дістатися сходинки `i`. Не використовуйте рекурсивний перебір усіх послідовностей.

| n | Кількість способів |
|---|---|
| `0` | `1` |
| `1` | `1` |
| `3` | `3` |
| `5` | `8` |

**Перевірка для n = 3:** `[1, 1, 1]`, `[1, 2]`, `[2, 1]`.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

foreach (int n in new[] { 0, 1, 3, 5 })
{
    Console.WriteLine(CountWays(n));
}

static int CountWays(int n)
{
    int[] dp = new int[n + 1];
    dp[0] = 1;
    for (int i = 1; i <= n; i++)
    {
        dp[i] = dp[i - 1]; // Останній крок на одну сходинку.
        if (i >= 2)
        {
            dp[i] += dp[i - 2]; // Або останній крок на дві.
        }
    }
    return dp[n];
}
```

**Очікуваний вивід:**

```text
1
1
3
8
```

</details>

## Завдання 4. Мінімум монет незвичних номіналів

Є необмежена кількість монет номіналів `1`, `3`, `4`. Для суми `0 ≤ amount ≤ 100` знайдіть найменшу кількість монет.

**Вимоги:** `dp[0] = 0`; для кожної додатної суми переберіть усі номінали, які не перевищують цю суму. Відновлювати самі монети не обов'язково.

| Сума | Мінімум монет | Один із варіантів |
|---|---|---|
| `0` | `0` | Немає монет |
| `2` | `2` | `1 + 1` |
| `6` | `2` | `3 + 3` |
| `8` | `2` | `4 + 4` |

**Самоперевірка:** для суми `6` вибір найбільшої монети на кожному кроці дає `4 + 1 + 1`. Поясніть, чому тут жадібний підхід програє таблиці `dp`.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

foreach (int amount in new[] { 0, 2, 6, 8 })
{
    Console.WriteLine(MinCoins(amount));
}

static int MinCoins(int amount)
{
    int[] coins = { 1, 3, 4 };
    int[] dp = new int[amount + 1];
    for (int sum = 1; sum <= amount; sum++)
    {
        dp[sum] = sum; // Завжди можна взяти sum монет номіналом 1.
        foreach (int coin in coins)
        {
            if (coin <= sum)
            {
                dp[sum] = Math.Min(dp[sum], dp[sum - coin] + 1);
            }
        }
    }
    return dp[amount];
}
```

**Очікуваний вивід:**

```text
0
2
2
2
```

</details>

## Завдання 5. Найдешевше з’єднання мотузок

З’єднання двох мотузок коштує суму їх довжин і створює мотузку такої самої сумарної довжини. Щоразу з’єднуйте дві найкоротші через `PriorityQueue`. Знайдіть загальну вартість; для нуля або однієї мотузки вона `0`. Довжини додатні, до `1000`, мотузок до `20`.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

Console.WriteLine(JoinCost(new[] { 4, 3, 2, 6 }));
Console.WriteLine(JoinCost(new[] { 7 }));
Console.WriteLine(JoinCost(Array.Empty<int>()));

static int JoinCost(int[] lengths)
{
    var queue = new PriorityQueue<int, int>();
    foreach (int length in lengths) queue.Enqueue(length, length);
    int cost = 0;
    while (queue.Count > 1)
    {
        int joined = queue.Dequeue() + queue.Dequeue();
        cost += joined;
        queue.Enqueue(joined, joined);
    }
    return cost;
}
```

**Очікуваний вивід:**

```text
29
0
0
```

</details>

## Завдання 6. Накрити позначки відрізками довжини два

На прямій задані цілі координати позначок у межах `0..1000`. Накрийте їх найменшою кількістю замкнених відрізків довжини `2`. Починайте новий відрізок у найлівішій ще не накритій точці. Повторні координати не потребують окремого відрізка.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

Console.WriteLine(string.Join(", ", Cover(new[] { 5, 1, 8, 2, 4 })));
Console.WriteLine(string.Join(", ", Cover(new[] { 1, 1 })));
Console.WriteLine($"Count={Cover(Array.Empty<int>()).Count}");

static List<(int Start, int End)> Cover(int[] points)
{
    int[] sorted = (int[])points.Clone();
    Array.Sort(sorted);
    var segments = new List<(int Start, int End)>();
    foreach (int point in sorted)
        if (segments.Count == 0 || point > segments[^1].End)
            segments.Add((point, point + 2));
    return segments;
}
```

**Очікуваний вивід:**

```text
(1, 3), (4, 6), (8, 10)
(1, 3)
Count=0
```

</details>

## Завдання 7. Байдарки для двох

Кожна байдарка вміщує до двох учасників із сумарною вагою не більшою за `limit`. Усі ваги додатні й не перевищують `limit`, який не більший за `1000`. Знайдіть мінімум байдарок: після сортування пробуйте посадити найлегшого з найважчим.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

Console.WriteLine(Boats(new[] { 3, 2, 2, 1 }, 3));
Console.WriteLine(Boats(new[] { 1, 1, 2, 2 }, 3));
Console.WriteLine(Boats(Array.Empty<int>(), 3));

static int Boats(int[] weights, int limit)
{
    int[] sorted = (int[])weights.Clone();
    Array.Sort(sorted);
    int left = 0, right = sorted.Length - 1, count = 0;
    while (left <= right)
    {
        if (left < right && sorted[left] + sorted[right] <= limit) left++;
        right--;
        count++;
    }
    return count;
}
```

**Очікуваний вивід:**

```text
3
2
0
```

</details>

## Завдання 8. Призи без сусідніх клітинок

У ряду лежать невід’ємні призи. Можна вибрати будь-які клітинки, але не дві сусідні. Знайдіть найбільшу суму. Для кожного префікса порівняйте «пропустити поточний» і «взяти поточний плюс найкраще до попереднього».

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

Console.WriteLine(BestPrize(new[] { 2, 7, 9, 3, 1 }));
Console.WriteLine(BestPrize(new[] { 5 }));
Console.WriteLine(BestPrize(Array.Empty<int>()));

static long BestPrize(int[] prizes)
{
    long twoBack = 0, oneBack = 0;
    foreach (int prize in prizes)
    {
        long current = Math.Max(oneBack, twoBack + prize);
        twoBack = oneBack;
        oneBack = current;
    }
    return oneBack;
}
```

**Очікуваний вивід:**

```text
12
5
0
```

</details>

## Завдання 9. Шляхи лише праворуч і вниз

На прямокутній сітці до `10 × 10` деякі клітинки закриті. Порахуйте шляхи з верхньої лівої до нижньої правої клітинки з кроками тільки праворуч і вниз. Закрита стартова клітинка або порожня сітка дає `0`. Шукати найкоротший шлях не потрібно.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

bool[,] blocked = { { false, false, false }, { false, true, false }, { false, false, false } };
Console.WriteLine(CountPaths(blocked));
Console.WriteLine(CountPaths(new bool[,] { { false } }));
Console.WriteLine(CountPaths(new bool[,] { { true } }));
Console.WriteLine(CountPaths(new bool[0, 0]));

static long CountPaths(bool[,] blocked)
{
    int rows = blocked.GetLength(0), columns = blocked.GetLength(1);
    if (rows == 0 || columns == 0 || blocked[0, 0]) return 0;
    long[,] dp = new long[rows, columns];
    dp[0, 0] = 1;
    for (int r = 0; r < rows; r++)
        for (int c = 0; c < columns; c++)
        {
            if (blocked[r, c]) continue;
            if (r > 0) dp[r, c] += dp[r - 1, c];
            if (c > 0) dp[r, c] += dp[r, c - 1];
        }
    return dp[rows - 1, columns - 1];
}
```

**Очікуваний вивід:**

```text
2
1
0
0
```

</details>

## Завдання 10. Найдешевші сходи

На сходинці `i` сплачуємо `cost[i]`, після чого переходимо на одну або дві сходинки вище. Почати дозволено з індексу `0` або `1`; вершина має індекс `cost.Length` і безкоштовна. Знайдіть мінімальну вартість. Для нуля або однієї сходинки можна одразу почати на вершині, результат `0`.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

Console.WriteLine(MinCost(new[] { 10, 15, 20 }));
Console.WriteLine(MinCost(new[] { 1, 100, 1, 1, 1, 100, 1, 1, 100, 1 }));
Console.WriteLine(MinCost(new[] { 7 }));
Console.WriteLine(MinCost(Array.Empty<int>()));

static long MinCost(int[] cost)
{
    if (cost.Length < 2) return 0;
    long[] dp = new long[cost.Length + 1];
    for (int i = 2; i <= cost.Length; i++)
        dp[i] = Math.Min(dp[i - 1] + cost[i - 1], dp[i - 2] + cost[i - 2]);
    return dp[cost.Length];
}
```

**Очікуваний вивід:**

```text
15
6
0
0
```

</details>

## Завдання 11. Найдовша смуга однакових чисел

Знайдіть довжину найдовшого неперервного фрагмента з однакових чисел. `dp[i]` — довжина такої смуги, що закінчується саме на `i`. Якщо значення змінилося, починайте смугу довжини один; для порожнього масиву результат `0`.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

Console.WriteLine(LongestRun(new[] { 2, 2, 3, 3, 3, 1 }));
Console.WriteLine(LongestRun(new[] { 9 }));
Console.WriteLine(LongestRun(Array.Empty<int>()));

static int LongestRun(int[] values)
{
    int[] dp = new int[values.Length];
    int best = 0;
    for (int i = 0; i < values.Length; i++)
    {
        dp[i] = i > 0 && values[i] == values[i - 1] ? dp[i - 1] + 1 : 1;
        best = Math.Max(best, dp[i]);
    }
    return best;
}
```

**Очікуваний вивід:**

```text
3
1
0
```

</details>

## Завдання 12. Розділити напис на відомі слова

Перевірте, чи можна повністю розбити рядок на слова зі словника; кожне слово дозволено використовувати багато разів. Рядки містять малі латинські літери, довжина тексту до `40`, порожніх слів у словнику немає. `dp[i]` означає, що перші `i` символів уже можна розбити.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

var words = new HashSet<string> { "apple", "pie" };
foreach (string text in new[] { "applepie", "pieapplepie", "apples", "" })
    Console.WriteLine(CanSplit(text, words));

static bool CanSplit(string text, HashSet<string> words)
{
    bool[] dp = new bool[text.Length + 1];
    dp[0] = true;
    for (int end = 1; end <= text.Length; end++)
        for (int start = 0; start < end; start++)
            if (dp[start] && words.Contains(text[start..end]))
            {
                dp[end] = true;
                break;
            }
    return dp[text.Length];
}
```

**Очікуваний вивід:**

```text
True
True
False
True
```

</details>
