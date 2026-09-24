# Лекції 10–11 — Прості завдання на графи

[Матеріал лекції](Examples.md)

Мова — C#. Вершини нумеруються від `0` до `n - 1`; ребра задавайте в коді. Якщо не сказано інакше, петель і повторних ребер немає. Завдання незалежні: тут тренуємо властивості та представлення графа, перевірку маршруту й залежності.

**Усього: 12 завдань із повними реалізаціями та очікуваним виводом.**

> **Запуск реалізацій:** C# 14 / .NET 10. Встановіть .NET 10 SDK. Створіть консольний проєкт через `dotnet new console --framework net10.0`, замініть `Program.cs` одним повним блоком `csharp` і виконайте `dotnet run`. Кожен блок незалежний; класи й методи з інших завдань копіювати не потрібно. Для порожніх результатів у виводі використовуємо `[]`; логічні значення C# друкує як `True` / `False`.

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

## Завдання 5. Звідки виходять і куди входять стрілки

Для орієнтованого графа порахуйте вхідний і вихідний степені. Виведіть джерела з нульовим вхідним степенем і стоки з нульовим вихідним, за зростанням. Ізольована вершина належить обом спискам.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

const int n = 4;
(int From, int To)[] edges = { (0, 1), (0, 2), (2, 1) };
int[] incoming = new int[n], outgoing = new int[n];
foreach (var (from, to) in edges) { outgoing[from]++; incoming[to]++; }
var sources = new List<int>();
var sinks = new List<int>();
for (int v = 0; v < n; v++)
{
    if (incoming[v] == 0) sources.Add(v);
    if (outgoing[v] == 0) sinks.Add(v);
}
Console.WriteLine($"In: {string.Join(", ", incoming)}");
Console.WriteLine($"Out: {string.Join(", ", outgoing)}");
Console.WriteLine($"Sources: {string.Join(", ", sources)}");
Console.WriteLine($"Sinks: {string.Join(", ", sinks)}");
```

**Очікуваний вивід:**

```text
In: 0, 2, 1, 0
Out: 2, 0, 1, 0
Sources: 0, 3
Sinks: 1, 3
```

</details>

## Завдання 6. Матриця у список стрілок

Перетворіть квадратну булеву матрицю суміжності орієнтованого графа на список ребер. Переглядайте рядки, а всередині них стовпці за зростанням. Зворотні ребра автоматично не додавайте.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

bool[,] matrix = { { false, true, false }, { false, false, true }, { false, false, false } };
Console.WriteLine(string.Join(", ", Edges(matrix)));
Console.WriteLine($"Empty={Edges(new bool[0, 0]).Count}");

static List<(int From, int To)> Edges(bool[,] matrix)
{
    var result = new List<(int From, int To)>();
    int n = matrix.GetLength(0);
    for (int from = 0; from < n; from++)
        for (int to = 0; to < n; to++)
            if (matrix[from, to]) result.Add((from, to));
    return result;
}
```

**Очікуваний вивід:**

```text
(0, 1), (1, 2)
Empty=0
```

</details>

## Завдання 7. Чи всі знайомі між собою

Дано симетричну матрицю простого неорієнтованого графа. Перевірте, чи кожна пара різних вершин з’єднана ребром. Діагональ ігноруйте. Графи з нуля або однієї вершини в цій вправі вважайте повними.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

Console.WriteLine(IsComplete(new bool[,] { { false, true, true }, { true, false, true }, { true, true, false } }));
Console.WriteLine(IsComplete(new bool[,] { { false, true, false }, { true, false, true }, { false, true, false } }));
Console.WriteLine(IsComplete(new bool[1, 1]));
Console.WriteLine(IsComplete(new bool[0, 0]));

static bool IsComplete(bool[,] graph)
{
    int n = graph.GetLength(0);
    for (int u = 0; u < n; u++)
        for (int v = u + 1; v < n; v++)
            if (!graph[u, v]) return false;
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

## Завдання 8. Трикутники знайомств

Порахуйте трійки різних вершин простого неорієнтованого графа, між якими є всі три ребра. Перебирайте лише `a < b < c`, щоб кожен трикутник врахувати один раз. Для маленького графа достатньо трьох циклів.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

bool[,] graph = new bool[4, 4];
foreach (var (u, v) in new[] { (0, 1), (1, 2), (2, 0), (2, 3) })
    graph[u, v] = graph[v, u] = true;
Console.WriteLine(Triangles(graph));
for (int u = 0; u < 4; u++)
    for (int v = 0; v < 4; v++) graph[u, v] = u != v;
Console.WriteLine(Triangles(graph));
Console.WriteLine(Triangles(new bool[0, 0]));

static int Triangles(bool[,] graph)
{
    int n = graph.GetLength(0), count = 0;
    for (int a = 0; a < n; a++)
        for (int b = a + 1; b < n; b++)
            for (int c = b + 1; c < n; c++)
                if (graph[a, b] && graph[a, c] && graph[b, c]) count++;
    return count;
}
```

**Очікуваний вивід:**

```text
1
4
0
```

</details>

## Завдання 9. Центр зірки

Визначте, чи простий неорієнтований граф є зіркою: один центр сполучений із кожною іншою вершиною, а між іншими ребер немає. Для `n < 3` у цій вправі поверніть `-1`; для зірки поверніть центр, інакше теж `-1`.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

Console.WriteLine(StarCenter(4, new[] { (0, 2), (1, 2), (3, 2) }));
Console.WriteLine(StarCenter(4, new[] { (0, 1), (1, 2), (2, 3) }));
Console.WriteLine(StarCenter(1, Array.Empty<(int, int)>()));

static int StarCenter(int n, (int U, int V)[] edges)
{
    if (n < 3 || edges.Length != n - 1) return -1;
    int[] degree = new int[n];
    foreach (var (u, v) in edges) { degree[u]++; degree[v]++; }
    int center = -1;
    for (int v = 0; v < n; v++)
    {
        if (degree[v] == n - 1) center = v;
        else if (degree[v] != 1) return -1;
    }
    return center;
}
```

**Очікуваний вивід:**

```text
2
-1
-1
```

</details>

## Завдання 10. Вартість заданого маршруту

Матриця `int?[,]` описує орієнтовані ребра з невід’ємною вартістю; `null` означає відсутність ребра, а `0` — безкоштовне ребро. Порахуйте вартість уже заданого маршруту або поверніть `-1`, якщо один із переходів неможливий. Порожній маршрут і одна вершина коштують `0`.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

int?[,] cost = new int?[3, 3];
cost[0, 1] = 4; cost[1, 2] = 0; cost[0, 2] = 10;
foreach (int[] route in new[] { new[] { 0, 1, 2 }, new[] { 0, 2 }, new[] { 1, 0 }, new[] { 2 }, Array.Empty<int>() })
    Console.WriteLine(RouteCost(cost, route));

static long RouteCost(int?[,] cost, int[] route)
{
    long total = 0;
    for (int i = 1; i < route.Length; i++)
    {
        int? edge = cost[route[i - 1], route[i]];
        if (edge is null) return -1;
        total += edge.Value;
    }
    return total;
}
```

**Очікуваний вивід:**

```text
4
10
-1
0
0
```

</details>

## Завдання 11. Перевірити готовий порядок залежностей

Перевірте запропонований порядок вершин: кожна вершина `0..n-1` має бути рівно один раз, а для ребра `u → v` вершина `u` повинна стояти раніше за `v`. Сам порядок будувати не потрібно.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

(int U, int V)[] edges = { (0, 2), (1, 2) };
foreach (int[] order in new[] { new[] { 0, 1, 2 }, new[] { 1, 0, 2 }, new[] { 2, 0, 1 }, new[] { 0, 0, 2 } })
    Console.WriteLine(ValidOrder(3, edges, order));
Console.WriteLine(ValidOrder(0, Array.Empty<(int, int)>(), Array.Empty<int>()));

static bool ValidOrder(int n, (int U, int V)[] edges, int[] order)
{
    if (order.Length != n) return false;
    int[] position = new int[n];
    Array.Fill(position, -1);
    for (int i = 0; i < n; i++)
    {
        int v = order[i];
        if (v < 0 || v >= n || position[v] != -1) return false;
        position[v] = i;
    }
    foreach (var (u, v) in edges)
        if (position[u] >= position[v]) return false;
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
```

</details>

## Завдання 12. Нова вершина посередині кожного ребра

У неорієнтованому графі замініть кожне ребро `(u, v)` двома: `(u, middle)` та `(middle, v)`. Для ребра з індексом `i` номер нової вершини — `n + i`. Старі ребра в результат не додавайте. Виведіть нову кількість вершин і список ребер.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

Subdivide(3, new[] { (0, 1), (1, 2) });
Subdivide(2, Array.Empty<(int, int)>());

static void Subdivide(int n, (int U, int V)[] edges)
{
    var result = new List<(int U, int V)>();
    for (int i = 0; i < edges.Length; i++)
    {
        int middle = n + i;
        result.Add((edges[i].U, middle));
        result.Add((middle, edges[i].V));
    }
    Console.WriteLine($"Vertices={n + edges.Length}; edges=[{string.Join(", ", result)}]");
}
```

**Очікуваний вивід:**

```text
Vertices=5; edges=[(0, 3), (3, 1), (1, 4), (4, 2)]
Vertices=2; edges=[]
```

</details>
