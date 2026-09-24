# Лекції 8–9 — Прості задачі на бінарні дерева

[Матеріал лекції](Examples.md) · [Окремий набір на Union-Find](../Lecture-8/tasks.md)

Використовуйте C# і вузол із полями `Value`, `Left`, `Right`; відсутня дитина — `null`. Невеликі дерева створюйте вручну. Запис `v(left, right)` нижче описує дерево, `—` — відсутню дитину, а число без дужок — листок. Парсер цього запису реалізовувати не потрібно.

**Усього: 12 завдань із повними реалізаціями та очікуваним виводом.**

> **Запуск реалізацій:** C# 14 / .NET 10. Встановіть .NET 10 SDK. Створіть консольний проєкт через `dotnet new console --framework net10.0`, замініть `Program.cs` одним повним блоком `csharp` і виконайте `dotnet run`. Кожен блок незалежний; класи й методи з інших завдань копіювати не потрібно. Для порожніх результатів у виводі використовуємо `[]`; логічні значення C# друкує як `True` / `False`.

## Завдання 1. Дзеркальне дерево

Метод `Mirror(root)` міняє місцями ліве й праве піддерева **кожного** вузла. Можна змінювати початкове дерево.

**Вимоги:** обробіть `null` як базовий випадок. Перевіряйте значення та посилання дітей, а не лише порядок друку.

| Дерево до | Дерево після |
|---|---|
| `1(2(4, —), 3)` | `1(3, 2(—, 4))` |
| `7` | `7` |
| `—` | `—` |

**Самостійно:** застосуйте `Mirror` двічі; має відновитися початкова структура.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

Node?[] examples = { new Node(1, new Node(2, new Node(4)), new Node(3)), new Node(7), null };
foreach (Node? root in examples)
{
    Mirror(root);
    Console.WriteLine(Describe(root));
    Mirror(root);
    Console.WriteLine($"Двічі: {Describe(root)}");
}

static void Mirror(Node? root)
{
    if (root is null)
    {
        return;
    }
    (root.Left, root.Right) = (root.Right, root.Left);
    Mirror(root.Left);
    Mirror(root.Right);
}

static string Describe(Node? root)
{
    if (root is null)
    {
        return "—";
    }
    if (root.Left is null && root.Right is null)
    {
        return root.Value.ToString();
    }
    return $"{root.Value}({Describe(root.Left)}, {Describe(root.Right)})";
}

public sealed class Node(int value, Node? left = null, Node? right = null)
{
    public int Value { get; } = value;
    public Node? Left { get; set; } = left;
    public Node? Right { get; set; } = right;
}
```

**Очікуваний вивід:**

```text
1(3, 2(—, 4))
Двічі: 1(2(4, —), 3)
7
Двічі: 7
—
Двічі: —
```

</details>

## Завдання 2. Чи однакові два дерева

Метод `SameTree(a, b)` повертає `true`, лише якщо дерева мають однакові значення **й однакову структуру**. Положення лівої та правої дитини має значення.

**Вимоги:** спочатку розгляньте випадки двох `null` та лише одного `null`, потім порівнюйте корені й відповідні піддерева.

| Перше дерево | Друге дерево | Результат |
|---|---|---|
| `1(2, 3)` | `1(2, 3)` | `true` |
| `1(2, —)` | `1(—, 2)` | `false` |
| `1(2, 3)` | `1(2, 4)` | `false` |
| `—` | `—` | `true` |
| `7` | `—` | `false` |

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

Console.WriteLine(SameTree(new Node(1, new Node(2), new Node(3)), new Node(1, new Node(2), new Node(3))));
Console.WriteLine(SameTree(new Node(1, new Node(2)), new Node(1, null, new Node(2))));
Console.WriteLine(SameTree(new Node(1, new Node(2), new Node(3)), new Node(1, new Node(2), new Node(4))));
Console.WriteLine(SameTree(null, null));
Console.WriteLine(SameTree(new Node(7), null));

static bool SameTree(Node? a, Node? b)
{
    if (a is null || b is null)
    {
        return a is null && b is null;
    }
    return a.Value == b.Value
        && SameTree(a.Left, b.Left)
        && SameTree(a.Right, b.Right);
}

public sealed class Node(int value, Node? left = null, Node? right = null)
{
    public int Value { get; } = value;
    public Node? Left { get; set; } = left;
    public Node? Right { get; set; } = right;
}
```

**Очікуваний вивід:**

```text
True
False
False
True
False
```

</details>

## Завдання 3. Сума ключів у діапазоні BST

Дано коректне BST з різними цілими ключами. Знайдіть суму ключів у включному діапазоні `[low, high]`, де `low ≤ high`. Для прикладів використайте дерево `8(3(1, 6), 10(—, 14))`.

**Вимоги:** використайте порядок BST: якщо ключ менший за `low`, його ліве піддерево можна пропустити; якщо більший за `high` — праве. Суму зберігайте в `long`.

| Діапазон | Ключі, що входять | Сума |
|---|---|---|
| `[4, 10]` | `6, 8, 10` | `24` |
| `[1, 3]` | `1, 3` | `4` |
| `[8, 8]` | `8` | `8` |
| `[11, 13]` | Немає | `0` |

Для порожнього дерева результат завжди `0`.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

var root = new Node(8, new Node(3, new Node(1), new Node(6)), new Node(10, null, new Node(14)));
foreach (var (low, high) in new[] { (4, 10), (1, 3), (8, 8), (11, 13) })
{
    Console.WriteLine(RangeSum(root, low, high));
}
Console.WriteLine(RangeSum(null, 1, 10));

static long RangeSum(Node? root, int low, int high)
{
    if (root is null)
    {
        return 0;
    }
    if (root.Value < low)
    {
        return RangeSum(root.Right, low, high);
    }
    if (root.Value > high)
    {
        return RangeSum(root.Left, low, high);
    }
    return (long)root.Value + RangeSum(root.Left, low, high) + RangeSum(root.Right, low, high);
}

public sealed class Node(int value, Node? left = null, Node? right = null)
{
    public int Value { get; } = value;
    public Node? Left { get; set; } = left;
    public Node? Right { get; set; } = right;
}
```

**Очікуваний вивід:**

```text
24
4
8
0
0
```

</details>

## Завдання 4. Сума вздовж шляху до листка

Метод `HasPathSum(root, target)` перевіряє, чи існує шлях **від кореня до листка**, сума значень якого дорівнює `target`. Зупинка у внутрішньому вузлі не зараховується.

**Вимоги:** на кожному кроці віднімайте значення вузла від залишку суми. Для прикладів використайте дерево `5(4(1, —), 8(—, 2))`.

| target | Результат | Пояснення |
|---|---|---|
| `10` | `true` | `5 → 4 → 1` |
| `15` | `true` | `5 → 8 → 2` |
| `9` | `false` | `5 → 4` не закінчується листком |
| `0` | `false` | Такого шляху немає |

Для порожнього дерева відповідь `false`, навіть якщо `target = 0`. Для єдиного вузла `7` і `target = 7` відповідь `true`.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

var root = new Node(5, new Node(4, new Node(1)), new Node(8, null, new Node(2)));
foreach (int target in new[] { 10, 15, 9, 0 })
{
    Console.WriteLine(HasPathSum(root, target));
}
Console.WriteLine(HasPathSum(null, 0));
Console.WriteLine(HasPathSum(new Node(7), 7));

static bool HasPathSum(Node? root, long target)
{
    if (root is null)
    {
        return false;
    }
    long rest = target - root.Value;
    if (root.Left is null && root.Right is null)
    {
        return rest == 0;
    }
    return HasPathSum(root.Left, rest) || HasPathSum(root.Right, rest);
}

public sealed class Node(int value, Node? left = null, Node? right = null)
{
    public int Value { get; } = value;
    public Node? Left { get; set; } = left;
    public Node? Right { get; set; } = right;
}
```

**Очікуваний вивід:**

```text
True
True
False
False
False
True
```

</details>

## Завдання 5. Сума лише лівих листків

Додайте значення листків, які є саме лівими дітьми своїх батьків. Корінь без дітей не вважається лівим листком. Для порожнього дерева сума `0`; використайте `long`.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

var root = new Node(5, new Node(2), new Node(8, new Node(6), new Node(9)));
Console.WriteLine(LeftLeafSum(root));
Console.WriteLine(LeftLeafSum(new Node(7)));
Console.WriteLine(LeftLeafSum(null));

static long LeftLeafSum(Node? root)
{
    if (root is null) return 0;
    long sum = 0;
    if (root.Left is not null && root.Left.Left is null && root.Left.Right is null)
        sum += root.Left.Value;
    else sum += LeftLeafSum(root.Left);
    return sum + LeftLeafSum(root.Right);
}

public sealed class Node(int value, Node? left = null, Node? right = null)
{
    public int Value { get; } = value;
    public Node? Left { get; set; } = left;
    public Node? Right { get; set; } = right;
}
```

**Очікуваний вивід:**

```text
8
0
0
```

</details>

## Завдання 6. Симетрія відносно кореня

Перевірте, чи ліве та праве піддерева є дзеркальними за структурою і значеннями. Порівнюйте зовнішніх і внутрішніх дітей попарно. Порожнє дерево вважайте симетричним.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

var symmetric = new Node(1, new Node(2, new Node(3), new Node(4)), new Node(2, new Node(4), new Node(3)));
var asymmetric = new Node(1, new Node(2, new Node(3)), new Node(2, new Node(3)));
Console.WriteLine(IsSymmetric(symmetric));
Console.WriteLine(IsSymmetric(asymmetric));
Console.WriteLine(IsSymmetric(null));

static bool IsSymmetric(Node? root) => root is null || Mirrored(root.Left, root.Right);
static bool Mirrored(Node? a, Node? b)
{
    if (a is null || b is null) return a is null && b is null;
    return a.Value == b.Value && Mirrored(a.Left, b.Right) && Mirrored(a.Right, b.Left);
}

public sealed class Node(int value, Node? left = null, Node? right = null)
{
    public int Value { get; } = value;
    public Node? Left { get; set; } = left;
    public Node? Right { get; set; } = right;
}
```

**Очікуваний вивід:**

```text
True
False
True
```

</details>

## Завдання 7. Прибрати нульові листки

Видаліть усі листки зі значенням `0`. Якщо їхній нульовий батько після цього теж стає листком, видаліть і його. Обробляйте дітей перед батьком; поверніть новий корінь, який може бути `null`.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

var root = new Node(1, new Node(0, new Node(0)), new Node(2));
Node? result = Prune(root);
Console.WriteLine($"Root={result?.Value}, Left={result?.Left?.Value.ToString() ?? "NONE"}, Right={result?.Right?.Value}");
Console.WriteLine(Prune(new Node(0)) is null);
Console.WriteLine(Prune(null) is null);

static Node? Prune(Node? root)
{
    if (root is null) return null;
    root.Left = Prune(root.Left);
    root.Right = Prune(root.Right);
    return root.Value == 0 && root.Left is null && root.Right is null ? null : root;
}

public sealed class Node(int value, Node? left = null, Node? right = null)
{
    public int Value { get; } = value;
    public Node? Left { get; set; } = left;
    public Node? Right { get; set; } = right;
}
```

**Очікуваний вивід:**

```text
Root=1, Left=NONE, Right=2
True
True
```

</details>

## Завдання 8. Обчислити дерево виразу

Листок дерева містить ціле число, внутрішній вузол — оператор `+` або `*` і рівно двох дітей. Рекурсивно обчисліть результат. Дерева коректні, переповнення немає; парсер текстових виразів не потрібний.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

var expression = new Expr("*", new Expr("+", new Expr("2"), new Expr("3")), new Expr("4"));
Console.WriteLine(Evaluate(expression));
Console.WriteLine(Evaluate(new Expr("7")));

static int Evaluate(Expr node)
{
    if (node.Left is null && node.Right is null) return int.Parse(node.Token);
    int left = Evaluate(node.Left!);
    int right = Evaluate(node.Right!);
    return node.Token switch
    {
        "+" => left + right,
        "*" => left * right,
        _ => throw new ArgumentException("Невідомий оператор")
    };
}

public sealed record Expr(string Token, Expr? Left = null, Expr? Right = null);
```

**Очікуваний вивід:**

```text
20
7
```

</details>

## Завдання 9. Усі шляхи до листків

Поверніть шляхи від кореня до кожного листка у вигляді рядків із роздільником `->`. Обробляйте ліве піддерево перед правим. Для порожнього дерева список шляхів порожній.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

var root = new Node(1, new Node(2, null, new Node(5)), new Node(3));
foreach (string path in Paths(root, "")) Console.WriteLine(path);
Console.WriteLine($"Empty={Paths(null, "").Count}");

static List<string> Paths(Node? root, string prefix)
{
    var result = new List<string>();
    if (root is null) return result;
    string path = prefix.Length == 0 ? root.Value.ToString() : prefix + "->" + root.Value;
    if (root.Left is null && root.Right is null) result.Add(path);
    else
    {
        result.AddRange(Paths(root.Left, path));
        result.AddRange(Paths(root.Right, path));
    }
    return result;
}

public sealed class Node(int value, Node? left = null, Node? right = null)
{
    public int Value { get; } = value;
    public Node? Left { get; set; } = left;
    public Node? Right { get; set; } = right;
}
```

**Очікуваний вивід:**

```text
1->2->5
1->3
Empty=0
```

</details>

## Завдання 10. Додати два дерева

Побудуйте нове дерево: значення в однакових позиціях додаються, а відсутній вузол дає внесок `0`. Якщо в обох дерев у позиції порожньо, нового вузла немає. Створюйте нові вузли; вхідні дерева не змінюйте. Значення невеликі.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

var first = new Node(1, new Node(2));
var second = new Node(4, null, new Node(7));
Node? result = AddTrees(first, second);
Console.WriteLine($"{result?.Value}: {result?.Left?.Value}, {result?.Right?.Value}");
Console.WriteLine($"Original={first.Value}, NewNode={!ReferenceEquals(first, result)}");
Console.WriteLine(AddTrees(null, null) is null);

static Node? AddTrees(Node? a, Node? b)
{
    if (a is null && b is null) return null;
    return new Node((a?.Value ?? 0) + (b?.Value ?? 0),
        AddTrees(a?.Left, b?.Left), AddTrees(a?.Right, b?.Right));
}

public sealed class Node(int value, Node? left = null, Node? right = null)
{
    public int Value { get; } = value;
    public Node? Left { get; set; } = left;
    public Node? Right { get; set; } = right;
}
```

**Очікуваний вивід:**

```text
5: 2, 7
Original=1, NewNode=True
True
```

</details>

## Завдання 11. Спільний предок двох ключів BST

У BST з різними ключами знайдіть найнижчого спільного предка двох заданих ключів. Обидва ключі гарантовано є в дереві; вузол може бути предком самого себе. Якщо обидва ключі менші за поточний — ідіть ліворуч, якщо більші — праворуч.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

var root = new Node(6, new Node(2, new Node(1), new Node(4)), new Node(8, new Node(7), new Node(9)));
Console.WriteLine(CommonAncestor(root, 1, 4));
Console.WriteLine(CommonAncestor(root, 1, 9));
Console.WriteLine(CommonAncestor(root, 8, 9));

static int CommonAncestor(Node root, int a, int b)
{
    Node? current = root;
    while (current is not null)
    {
        if (a < current.Value && b < current.Value) current = current.Left;
        else if (a > current.Value && b > current.Value) current = current.Right;
        else return current.Value;
    }
    throw new ArgumentException("Ключів немає в дереві");
}

public sealed class Node(int value, Node? left = null, Node? right = null)
{
    public int Value { get; } = value;
    public Node? Left { get; set; } = left;
    public Node? Right { get; set; } = right;
}
```

**Очікуваний вивід:**

```text
2
6
8
```

</details>

## Завдання 12. Порядок батько-дитина у мін-купі

Перевірте лише властивість порядку: кожен батько не більший за кожну наявну дитину. Форму повного дерева в цій вправі не перевіряйте. Рівні значення дозволені; порожнє дерево задовольняє умову.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

Console.WriteLine(HasHeapOrder(new Node(1, new Node(3), new Node(2))));
Console.WriteLine(HasHeapOrder(new Node(2, new Node(1), new Node(3))));
Console.WriteLine(HasHeapOrder(new Node(2, new Node(2))));
Console.WriteLine(HasHeapOrder(null));

static bool HasHeapOrder(Node? root)
{
    if (root is null) return true;
    if (root.Left is not null && root.Left.Value < root.Value) return false;
    if (root.Right is not null && root.Right.Value < root.Value) return false;
    return HasHeapOrder(root.Left) && HasHeapOrder(root.Right);
}

public sealed class Node(int value, Node? left = null, Node? right = null)
{
    public int Value { get; } = value;
    public Node? Left { get; set; } = left;
    public Node? Right { get; set; } = right;
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
