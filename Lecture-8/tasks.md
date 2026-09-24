# Лекція 8 — Прості завдання на Union-Find

[Матеріал лекції](README.md)

Ця папка присвячена DSU; [завдання на бінарні дерева](../Lecture-8-9/tasks.md) містяться окремо. Реалізації нижче написані на C#. Об'єкти нумеруються від `0` до `n - 1`. У задачах 1 і 4 масив `parent` уже заданий і утворює коректний ліс: корінь посилається сам на себе.

> **Запуск реалізацій:** C# 14 / .NET 10. Встановіть .NET 10 SDK. Створіть консольний проєкт через `dotnet new console --framework net10.0`, замініть `Program.cs` одним повним блоком `csharp` і виконайте `dotnet run`. Кожен блок незалежний; класи й методи з інших завдань копіювати не потрібно. Для порожніх результатів у виводі використовуємо `[]`; логічні значення C# друкує як `True` / `False`.

## Завдання 1. Дійти до представника

Напишіть `Find(x)`: переходьте за `parent[x]`, поки не знайдете корінь. У цій вправі масив не змінюйте.

**Вхід:** `parent = [0, 0, 1, 3, 3]`.

| x | Шлях | Результат Find |
|---|---|---|
| `2` | `2 → 1 → 0` | `0` |
| `4` | `4 → 3` | `3` |
| `0` | `0` | `0` |

**Граничний випадок:** для `parent = [0]` виклик `Find(0)` повертає `0` без переходів.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

int[] parent = { 0, 0, 1, 3, 3 };
foreach (int x in new[] { 2, 4, 0 })
{
    Console.WriteLine(Find(parent, x));
}
Console.WriteLine($"parent=[{string.Join(", ", parent)}]");
Console.WriteLine(Find(new[] { 0 }, 0));

static int Find(int[] parent, int x)
{
    while (parent[x] != x)
    {
        x = parent[x];
    }
    return x;
}
```

**Очікуваний вивід:**

```text
0
3
0
parent=[0, 0, 1, 3, 3]
0
```

</details>

## Завдання 2. Чи з'єднані роз'єми

Спочатку п'ять роз'ємів `0..4` не з'єднані. `Connect(a, b)` об'єднує їхні множини, а `Connected(a, b)` перевіряє рівність представників. Уже наявне з'єднання не змінює множини.

**Вимоги:** використайте `Find` і `Union`; не будуйте обхід графа. Конкретний номер представника може залежати від вашої реалізації, тому перевіряйте саме зв'язність.

| Операція | Результат запиту |
|---|---|
| `Connected(0, 2)` | `false` |
| `Connect(0, 1)` | — |
| `Connect(1, 2)` | — |
| `Connected(0, 2)` | `true` |
| `Connected(0, 4)` | `false` |
| `Connect(0, 2)` | — |
| `Connected(3, 3)` | `true` |

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

var dsu = new DisjointSets(5);
Console.WriteLine(dsu.Connected(0, 2));
dsu.Connect(0, 1);
dsu.Connect(1, 2);
Console.WriteLine(dsu.Connected(0, 2));
Console.WriteLine(dsu.Connected(0, 4));
dsu.Connect(0, 2);
Console.WriteLine(dsu.Connected(3, 3));

public sealed class DisjointSets
{
    private readonly int[] _parent;

    public DisjointSets(int n)
    {
        _parent = new int[n];
        for (int i = 0; i < n; i++)
        {
            _parent[i] = i;
        }
    }

    private int Find(int x)
    {
        while (_parent[x] != x)
        {
            x = _parent[x];
        }
        return x;
    }

    public bool Connected(int a, int b) => Find(a) == Find(b);

    public void Connect(int a, int b)
    {
        int rootA = Find(a);
        int rootB = Find(b);
        if (rootA != rootB)
        {
            _parent[rootB] = rootA;
        }
    }
}
```

**Очікуваний вивід:**

```text
False
True
False
True
```

</details>

## Завдання 3. Розмір зібраної групи

Додайте до DSU метод `Size(x)`, що повертає кількість об'єктів у множині `x`. Спочатку кожна множина має розмір `1`; при об'єднанні різних коренів розміри додаються.

**Вимоги:** зберігайте розмір у корені. Повторне об'єднання тієї самої множини не має подвоювати розмір.

Для `n = 6` виконайте `Union(0, 1)`, `Union(2, 3)`, `Union(1, 2)`, `Union(0, 3)`.

| Запит після всіх об'єднань | Результат |
|---|---|
| `Size(0)` | `4` |
| `Size(3)` | `4` |
| `Size(4)` | `1` |
| `Size(5)` | `1` |

**Граничний випадок:** для `n = 1` операція `Union(0, 0)` залишає `Size(0) = 1`.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

var dsu = new DisjointSets(6);
dsu.Union(0, 1);
dsu.Union(2, 3);
dsu.Union(1, 2);
dsu.Union(0, 3);
foreach (int x in new[] { 0, 3, 4, 5 })
{
    Console.WriteLine(dsu.Size(x));
}
var single = new DisjointSets(1);
single.Union(0, 0);
Console.WriteLine(single.Size(0));

public sealed class DisjointSets
{
    private readonly int[] _parent;
    private readonly int[] _size;

    public DisjointSets(int n)
    {
        _parent = new int[n];
        _size = new int[n];
        for (int i = 0; i < n; i++)
        {
            _parent[i] = i;
            _size[i] = 1;
        }
    }

    private int Find(int x)
    {
        while (_parent[x] != x)
        {
            x = _parent[x];
        }
        return x;
    }

    public int Size(int x) => _size[Find(x)];

    public void Union(int a, int b)
    {
        a = Find(a);
        b = Find(b);
        if (a == b)
        {
            return; // Розмір тієї самої множини не додаємо двічі.
        }
        if (_size[a] < _size[b])
        {
            (a, b) = (b, a);
        }
        _parent[b] = a;
        _size[a] += _size[b];
    }
}
```

**Очікуваний вивід:**

```text
4
4
1
1
1
```

</details>

## Завдання 4. Скоротити шлях до кореня

Дано `parent = [0, 0, 1, 2, 4]`. Реалізуйте `Find` із **повним стисненням шляху**: після знаходження кореня кожен відвіданий вузол має вказувати безпосередньо на нього.

| Послідовні виклики | Повернене значення | Масив parent після виклику |
|---|---|---|
| `Find(3)` | `0` | `[0, 0, 0, 0, 4]` |
| `Find(3)` ще раз | `0` | `[0, 0, 0, 0, 4]` |
| `Find(4)` | `4` | `[0, 0, 0, 0, 4]` |

**Самоперевірка:** до першого виклику шлях від `3` має три ребра, після нього — одне. Об'єкт `4` має залишитися в окремій множині.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

int[] parent = { 0, 0, 1, 2, 4 };
foreach (int x in new[] { 3, 3, 4 })
{
    int root = Find(parent, x);
    Console.WriteLine($"{root}; parent=[{string.Join(", ", parent)}]");
}

static int Find(int[] parent, int x)
{
    if (parent[x] != x)
    {
        parent[x] = Find(parent, parent[x]);
    }
    return parent[x];
}
```

**Очікуваний вивід:**

```text
0; parent=[0, 0, 0, 0, 4]
0; parent=[0, 0, 0, 0, 4]
4; parent=[0, 0, 0, 0, 4]
```

</details>
