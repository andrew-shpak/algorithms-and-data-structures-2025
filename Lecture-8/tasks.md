# Лекція 8 — Прості завдання на Union-Find

[Матеріал лекції](README.md)

Ця папка присвячена DSU; [завдання на бінарні дерева](../Lecture-8-9/tasks.md) містяться окремо. Реалізації нижче написані на C#. Об'єкти нумеруються від `0` до `n - 1`. У задачах 1 і 4 масив `parent` уже заданий і утворює коректний ліс: корінь посилається сам на себе.

**Усього: 12 завдань із повними реалізаціями та очікуваним виводом.**

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

## Завдання 5. Скільки окремих груп лишилося

Почніть із `n` одноелементних множин. Після кожного об’єднання виведіть кількість компонент. Зменшуйте її лише тоді, коли об’єднали різні корені. Для `n = 0` компонент немає.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

var dsu = new Dsu(5);
Console.WriteLine(dsu.Count);
foreach (var (a, b) in new[] { (0, 1), (2, 3), (1, 3), (0, 2) })
{
    dsu.Union(a, b);
    Console.WriteLine(dsu.Count);
}
Console.WriteLine(new Dsu(0).Count);

public sealed class Dsu
{
    private readonly int[] _parent;
    private readonly int[] _size;
    public int Count { get; private set; }

    public Dsu(int n)
    {
        _parent = new int[n];
        _size = new int[n];
        Count = n;
        for (int i = 0; i < n; i++) { _parent[i] = i; _size[i] = 1; }
    }

    public int Find(int x)
    {
        if (_parent[x] != x) _parent[x] = Find(_parent[x]);
        return _parent[x];
    }

    public int Size(int x) => _size[Find(x)];

    public bool Union(int a, int b)
    {
        a = Find(a);
        b = Find(b);
        if (a == b) return false;
        if (_size[a] < _size[b]) (a, b) = (b, a);
        _parent[b] = a;
        _size[a] += _size[b];
        Count--;
        return true;
    }
}
```

**Очікуваний вивід:**

```text
5
4
3
2
2
0
```

</details>

## Завдання 6. Найбільша команда після кожного злиття

Після кожного об’єднання підтримуйте розмір найбільшої множини. Початково для `n > 0` він дорівнює `1`. Після успішного або повторного об’єднання достатньо перевірити розмір множини одного з аргументів.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

var dsu = new Dsu(6);
int largest = 1;
foreach (var (a, b) in new[] { (0, 1), (2, 3), (1, 2), (0, 3), (4, 5) })
{
    dsu.Union(a, b);
    largest = Math.Max(largest, dsu.Size(a));
    Console.WriteLine(largest);
}

public sealed class Dsu
{
    private readonly int[] _parent;
    private readonly int[] _size;
    public int Count { get; private set; }

    public Dsu(int n)
    {
        _parent = new int[n];
        _size = new int[n];
        Count = n;
        for (int i = 0; i < n; i++) { _parent[i] = i; _size[i] = 1; }
    }

    public int Find(int x)
    {
        if (_parent[x] != x) _parent[x] = Find(_parent[x]);
        return _parent[x];
    }

    public int Size(int x) => _size[Find(x)];

    public bool Union(int a, int b)
    {
        a = Find(a);
        b = Find(b);
        if (a == b) return false;
        if (_size[a] < _size[b]) (a, b) = (b, a);
        _parent[b] = a;
        _size[a] += _size[b];
        Count--;
        return true;
    }
}
```

**Очікуваний вивід:**

```text
2
2
4
4
4
```

</details>

## Завдання 7. Перше зайве з’єднання

Кабелі між об’єктами додаються послідовно. Знайдіть індекс першого кабелю, кінці якого вже належать одній компоненті, або `-1`. Індекси від нуля. Скористайтеся результатом `Union`, без пошуку шляхів у графі.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

Console.WriteLine(FirstRedundant(4, new[] { (0, 1), (1, 2), (0, 2), (2, 3) }));
Console.WriteLine(FirstRedundant(3, new[] { (0, 1), (1, 2) }));
Console.WriteLine(FirstRedundant(0, Array.Empty<(int, int)>()));

static int FirstRedundant(int n, (int A, int B)[] cables)
{
    var dsu = new Dsu(n);
    for (int i = 0; i < cables.Length; i++)
        if (!dsu.Union(cables[i].A, cables[i].B)) return i;
    return -1;
}

public sealed class Dsu
{
    private readonly int[] _parent;
    private readonly int[] _size;
    public int Count { get; private set; }

    public Dsu(int n)
    {
        _parent = new int[n];
        _size = new int[n];
        Count = n;
        for (int i = 0; i < n; i++) { _parent[i] = i; _size[i] = 1; }
    }

    public int Find(int x)
    {
        if (_parent[x] != x) _parent[x] = Find(_parent[x]);
        return _parent[x];
    }

    public int Size(int x) => _size[Find(x)];

    public bool Union(int a, int b)
    {
        a = Find(a);
        b = Find(b);
        if (a == b) return false;
        if (_size[a] < _size[b]) (a, b) = (b, a);
        _parent[b] = a;
        _size[a] += _size[b];
        Count--;
        return true;
    }
}
```

**Очікуваний вивід:**

```text
2
-1
-1
```

</details>

## Завдання 8. Пари об’єктів усередині компонент

Після об’єднань порахуйте неупорядковані пари різних об’єктів, які належать одній компоненті. Компонента розміру `s` дає `s * (s - 1) / 2` пар. Кожен корінь врахуйте один раз; результат зберігайте в `long`.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

var dsu = new Dsu(6);
dsu.Union(0, 1);
dsu.Union(1, 2);
dsu.Union(3, 4);
Console.WriteLine(CountPairs(dsu, 6));
Console.WriteLine(CountPairs(new Dsu(3), 3));
Console.WriteLine(CountPairs(new Dsu(0), 0));

static long CountPairs(Dsu dsu, int n)
{
    long result = 0;
    for (int i = 0; i < n; i++)
        if (dsu.Find(i) == i)
        {
            long size = dsu.Size(i);
            result += size * (size - 1) / 2;
        }
    return result;
}

public sealed class Dsu
{
    private readonly int[] _parent;
    private readonly int[] _size;
    public int Count { get; private set; }

    public Dsu(int n)
    {
        _parent = new int[n];
        _size = new int[n];
        Count = n;
        for (int i = 0; i < n; i++) { _parent[i] = i; _size[i] = 1; }
    }

    public int Find(int x)
    {
        if (_parent[x] != x) _parent[x] = Find(_parent[x]);
        return _parent[x];
    }

    public int Size(int x) => _size[Find(x)];

    public bool Union(int a, int b)
    {
        a = Find(a);
        b = Find(b);
        if (a == b) return false;
        if (_size[a] < _size[b]) (a, b) = (b, a);
        _parent[b] = a;
        _size[a] += _size[b];
        Count--;
        return true;
    }
}
```

**Очікуваний вивід:**

```text
4
0
0
```

</details>

## Завдання 9. Найменший номер як підпис групи

Внутрішній корінь DSU може мати довільний номер. Для кожного об’єкта виведіть найменший номер у його компоненті як зрозумілий підпис. Спочатку обчисліть мінімум для кожного кореня; дерево DSU через це перебудовувати не потрібно.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

const int n = 5;
var dsu = new Dsu(n);
dsu.Union(3, 1);
dsu.Union(1, 4);
dsu.Union(2, 0);
int[] smallest = new int[n];
Array.Fill(smallest, n);
for (int i = 0; i < n; i++)
{
    int root = dsu.Find(i);
    smallest[root] = Math.Min(smallest[root], i);
}
var labels = new List<int>();
for (int i = 0; i < n; i++) labels.Add(smallest[dsu.Find(i)]);
Console.WriteLine(string.Join(", ", labels));

public sealed class Dsu
{
    private readonly int[] _parent;
    private readonly int[] _size;
    public int Count { get; private set; }

    public Dsu(int n)
    {
        _parent = new int[n];
        _size = new int[n];
        Count = n;
        for (int i = 0; i < n; i++) { _parent[i] = i; _size[i] = 1; }
    }

    public int Find(int x)
    {
        if (_parent[x] != x) _parent[x] = Find(_parent[x]);
        return _parent[x];
    }

    public int Size(int x) => _size[Find(x)];

    public bool Union(int a, int b)
    {
        a = Find(a);
        b = Find(b);
        if (a == b) return false;
        if (_size[a] < _size[b]) (a, b) = (b, a);
        _parent[b] = a;
        _size[a] += _size[b];
        Count--;
        return true;
    }
}
```

**Очікуваний вивід:**

```text
0, 1, 0, 1, 1
```

</details>

## Завдання 10. Учасники однієї групи

Поверніть усі номери, які належать до компоненти заданого об’єкта, за зростанням. Для невеликої DSU пройдіть номери `0..n-1` і порівняйте їх представників. Сам об’єкт також входить у результат.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

var dsu = new Dsu(5);
dsu.Union(0, 2);
dsu.Union(2, 4);
Console.WriteLine(string.Join(", ", Members(dsu, 5, 2)));
Console.WriteLine(string.Join(", ", Members(dsu, 5, 1)));

static List<int> Members(Dsu dsu, int n, int member)
{
    int root = dsu.Find(member);
    var result = new List<int>();
    for (int i = 0; i < n; i++)
        if (dsu.Find(i) == root) result.Add(i);
    return result;
}

public sealed class Dsu
{
    private readonly int[] _parent;
    private readonly int[] _size;
    public int Count { get; private set; }

    public Dsu(int n)
    {
        _parent = new int[n];
        _size = new int[n];
        Count = n;
        for (int i = 0; i < n; i++) { _parent[i] = i; _size[i] = 1; }
    }

    public int Find(int x)
    {
        if (_parent[x] != x) _parent[x] = Find(_parent[x]);
        return _parent[x];
    }

    public int Size(int x) => _size[Find(x)];

    public bool Union(int a, int b)
    {
        a = Find(a);
        b = Find(b);
        if (a == b) return false;
        if (_size[a] < _size[b]) (a, b) = (b, a);
        _parent[b] = a;
        _size[a] += _size[b];
        Count--;
        return true;
    }
}
```

**Очікуваний вивід:**

```text
0, 2, 4
1
```

</details>

## Завдання 11. Скорочення шляху через дідуся

Реалізуйте ітеративний `Find` із path halving: спрямовуйте поточний вузол на його дідуся, потім переходьте до нового батька. Масив `parent` задає коректний ліс. Порівняйте масив після першого й повторного викликів: повне стиснення за один виклик тут не вимагається.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

int[] parent = { 0, 0, 1, 2, 3 };
Console.WriteLine(FindHalving(parent, 4));
Console.WriteLine(string.Join(", ", parent));
Console.WriteLine(FindHalving(parent, 4));
Console.WriteLine(string.Join(", ", parent));

static int FindHalving(int[] parent, int x)
{
    while (parent[x] != x)
    {
        parent[x] = parent[parent[x]];
        x = parent[x];
    }
    return x;
}
```

**Очікуваний вивід:**

```text
0
0, 0, 0, 2, 2
0
0, 0, 0, 2, 0
```

</details>

## Завдання 12. Сусідні однакові плитки

Плитки стоять у ряд. Об’єднайте індекси сусідніх плиток однакового кольору; однакові кольори через проміжок не з’єднуються. Виведіть кількість суцільних кольорових ділянок через `Dsu.Count`.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

foreach (string[] colors in new[] { new[] { "red", "red", "blue", "red" }, new[] { "x", "x", "x" }, Array.Empty<string>() })
{
    var dsu = new Dsu(colors.Length);
    for (int i = 1; i < colors.Length; i++)
        if (colors[i] == colors[i - 1]) dsu.Union(i - 1, i);
    Console.WriteLine(dsu.Count);
}

public sealed class Dsu
{
    private readonly int[] _parent;
    private readonly int[] _size;
    public int Count { get; private set; }

    public Dsu(int n)
    {
        _parent = new int[n];
        _size = new int[n];
        Count = n;
        for (int i = 0; i < n; i++) { _parent[i] = i; _size[i] = 1; }
    }

    public int Find(int x)
    {
        if (_parent[x] != x) _parent[x] = Find(_parent[x]);
        return _parent[x];
    }

    public int Size(int x) => _size[Find(x)];

    public bool Union(int a, int b)
    {
        a = Find(a);
        b = Find(b);
        if (a == b) return false;
        if (_size[a] < _size[b]) (a, b) = (b, a);
        _parent[b] = a;
        _size[a] += _size[b];
        Count--;
        return true;
    }
}
```

**Очікуваний вивід:**

```text
3
1
0
```

</details>
