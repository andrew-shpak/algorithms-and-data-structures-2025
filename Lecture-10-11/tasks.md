# Лекції 10–11 — Прості завдання на графи

[Матеріал лекції](Examples.md)

Мова — C#. Вершини нумеруються від `0` до `n - 1`; ребра задавайте в коді. Якщо не сказано інакше, петель і повторних ребер немає. Завдання незалежні: тут тренуємо властивості та представлення графа, перевірку маршруту й залежності.

> **Запуск реалізацій:** C# 12+ / .NET 8+. Створіть консольний проєкт через `dotnet new console`, замініть `Program.cs` одним повним блоком `csharp` і виконайте `dotnet run`. Кожен блок незалежний; класи й методи з інших завдань копіювати не потрібно. Для порожніх результатів у виводі використовуємо `[]`; логічні значення C# друкує як `True` / `False`.

## Завдання 1. Скільки сусідів

Для неорієнтованого графа порахуйте степінь кожної вершини — кількість ребер, що до неї приєднані. Виведіть номери ізольованих вершин за зростанням.

**Вимоги:** одне ребро `(u, v)` збільшує два лічильники: для `u` та для `v`. Обхід BFS або DFS не потрібен.

| n; ребра | Степені вершин 0..n-1 | Ізольовані вершини |
|---|---|---|
| `5; [(0, 1), (0, 2), (2, 3)]` | `[2, 1, 2, 1, 0]` | `[4]` |
| `3; []` | `[0, 0, 0]` | `[0, 1, 2]` |
| `0; []` | `[]` | `[]` |

**Самоперевірка:** сума степенів дорівнює подвоєній кількості ребер.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

PrintDegrees(5, new[] { (0, 1), (0, 2), (2, 3) });
PrintDegrees(3, Array.Empty<(int, int)>());
PrintDegrees(0, Array.Empty<(int, int)>());

static void PrintDegrees(int n, (int U, int V)[] edges)
{
    int[] degree = new int[n];
    foreach (var (u, v) in edges)
    {
        degree[u]++;
        degree[v]++;
    }
    var isolated = new List<int>();
    for (int vertex = 0; vertex < n; vertex++)
    {
        if (degree[vertex] == 0)
        {
            isolated.Add(vertex);
        }
    }
    Console.WriteLine($"Степені: [{string.Join(", ", degree)}]; ізольовані: [{string.Join(", ", isolated)}]");
}
```

**Очікуваний вивід:**

```text
Степені: [2, 1, 2, 1, 0]; ізольовані: [4]
Степені: [0, 0, 0]; ізольовані: [0, 1, 2]
Степені: []; ізольовані: []
```

</details>

## Завдання 2. Чи можна пройти заданий маршрут

Дано неорієнтований граф та послідовність вершин. Поверніть `true`, якщо між **кожною парою сусідніх вершин маршруту** є ребро. Повторне відвідування вершин дозволене; найкоротший шлях шукати не потрібно.

**Вимоги:** використайте матрицю суміжності. Усі номери в маршруті коректні; порожній маршрут і маршрут з однієї вершини вважайте допустимими.

Для `n = 4` і ребер `[(0, 1), (1, 2), (2, 3)]`:

| Маршрут | Результат |
|---|---|
| `[0, 1, 2, 3]` | `true` |
| `[0, 1, 0]` | `true` |
| `[0, 2, 3]` | `false` |
| `[1, 1]` | `false` |
| `[3]` або `[]` | `true` |

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

bool[,] adjacency = new bool[4, 4];
foreach (var (u, v) in new[] { (0, 1), (1, 2), (2, 3) })
{
    adjacency[u, v] = true;
    adjacency[v, u] = true;
}
int[][] routes =
{
    new[] { 0, 1, 2, 3 }, new[] { 0, 1, 0 }, new[] { 0, 2, 3 },
    new[] { 1, 1 }, new[] { 3 }, Array.Empty<int>()
};
foreach (int[] route in routes)
{
    Console.WriteLine(IsValidWalk(adjacency, route));
}

static bool IsValidWalk(bool[,] adjacency, int[] route)
{
    for (int i = 1; i < route.Length; i++)
    {
        if (!adjacency[route[i - 1], route[i]])
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
True
False
False
True
True
```

</details>

## Завдання 3. Розвернути всі стрілки

Побудуйте список суміжності **зворотного орієнтованого графа**: кожне ребро `u → v` перетворюється на `v → u`.

**Вимоги:** створіть нові списки, не змінюючи початкових. Переглядайте вихідні вершини за зростанням і додавайте їх у відповідні списки зворотного графа. Зберігайте також вершини без ребер.

| n; ребра початкового графа | Зворотний список суміжності |
|---|---|
| `4; [(0, 1), (0, 2), (2, 1)]` | `0: []; 1: [0, 2]; 2: [0]; 3: []` |
| `2; [(0, 1), (1, 0)]` | `0: [1]; 1: [0]` |
| `2; []` | `0: []; 1: []` |

**Самостійно:** розверніть граф ще раз. Має відновитися той самий набір ребер, хоча порядок елементів списків може відрізнятися.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

PrintReversed(4, new[] { (0, 1), (0, 2), (2, 1) });
PrintReversed(2, new[] { (0, 1), (1, 0) });
PrintReversed(2, Array.Empty<(int, int)>());

static void PrintReversed(int n, (int U, int V)[] edges)
{
    var graph = new List<int>[n];
    var reversed = new List<int>[n];
    for (int i = 0; i < n; i++)
    {
        graph[i] = new List<int>();
        reversed[i] = new List<int>();
    }
    foreach (var (u, v) in edges)
    {
        graph[u].Add(v);
    }
    for (int u = 0; u < n; u++)
    {
        foreach (int v in graph[u])
        {
            reversed[v].Add(u);
        }
    }
    var descriptions = new List<string>();
    for (int i = 0; i < n; i++)
    {
        descriptions.Add($"{i}: [{string.Join(", ", reversed[i])}]");
    }
    Console.WriteLine(string.Join("; ", descriptions));
}
```

**Очікуваний вивід:**

```text
0: []; 1: [0, 2]; 2: [0]; 3: []
0: [1]; 1: [0]
0: []; 1: []
```

</details>

## Завдання 4. Порядок складання моделі — `Queue<int>`

Ребро `u → v` означає, що деталь `u` треба встановити раніше за деталь `v`. Побудуйте порядок монтажу алгоритмом Кана: порахуйте вхідні степені, поставте в чергу вершини з нульовим степенем і поступово приберіть їхні вихідні ребра.

**Вимоги:** початкові вершини додавайте в чергу за зростанням; сусідів кожної вершини також переглядайте за зростанням. Нову готову вершину додавайте в хвіст. Якщо оброблено менше ніж `n` вершин, поверніть `CYCLE` замість часткового порядку.

| n; залежності | Результат |
|---|---|
| `4; [(0, 2), (1, 2), (2, 3)]` | `[0, 1, 2, 3]` |
| `3; []` | `[0, 1, 2]` |
| `2; [(0, 1), (1, 0)]` | `CYCLE` |
| `0; []` | `[]` |

**Самоперевірка:** для кожного ребра `u → v` позиція `u` в результаті має бути меншою за позицію `v`.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

PrintOrder(4, new[] { (0, 2), (1, 2), (2, 3) });
PrintOrder(3, Array.Empty<(int, int)>());
PrintOrder(2, new[] { (0, 1), (1, 0) });
PrintOrder(0, Array.Empty<(int, int)>());

static void PrintOrder(int n, (int U, int V)[] edges)
{
    List<int>? order = AssemblyOrder(n, edges);
    Console.WriteLine(order is null ? "CYCLE" : $"[{string.Join(", ", order)}]");
}

static List<int>? AssemblyOrder(int n, (int U, int V)[] edges)
{
    var graph = new List<int>[n];
    int[] inDegree = new int[n];
    for (int i = 0; i < n; i++)
    {
        graph[i] = new List<int>();
    }
    foreach (var (u, v) in edges)
    {
        graph[u].Add(v);
        inDegree[v]++;
    }
    var ready = new Queue<int>();
    for (int i = 0; i < n; i++)
    {
        graph[i].Sort();
        if (inDegree[i] == 0)
        {
            ready.Enqueue(i);
        }
    }
    var order = new List<int>();
    while (ready.TryDequeue(out int u))
    {
        order.Add(u);
        foreach (int v in graph[u])
        {
            inDegree[v]--;
            if (inDegree[v] == 0)
            {
                ready.Enqueue(v);
            }
        }
    }
    return order.Count == n ? order : null;
}
```

**Очікуваний вивід:**

```text
[0, 1, 2, 3]
[0, 1, 2]
CYCLE
[]
```

</details>
