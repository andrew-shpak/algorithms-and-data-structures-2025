# Лекції 8–9 — Прості задачі на бінарні дерева

[Матеріал лекції](Examples.md) · [Окремий набір на Union-Find](../Lecture-8/tasks.md)

Використовуйте C# і вузол із полями `Value`, `Left`, `Right`; відсутня дитина — `null`. Невеликі дерева створюйте вручну. Запис `v(left, right)` нижче описує дерево, `—` — відсутню дитину, а число без дужок — листок. Парсер цього запису реалізовувати не потрібно.

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
