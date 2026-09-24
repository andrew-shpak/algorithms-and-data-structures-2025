# Лекція 9 — Прості задачі на жадібний вибір і динамічне програмування

[Матеріал лекції](README.md)

Реалізації нижче написані на C#. Дані задавайте в коді. Для перших двох задач поясніть свій жадібний вибір, для решти — значення комірки `dp` і початкові значення.

> **Запуск реалізацій:** C# 12+ / .NET 8+. Створіть консольний проєкт через `dotnet new console`, замініть `Program.cs` одним повним блоком `csharp` і виконайте `dotnet run`. Кожен блок незалежний; класи й методи з інших завдань копіювати не потрібно. Для порожніх результатів у виводі використовуємо `[]`; логічні значення C# друкує як `True` / `False`.

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
