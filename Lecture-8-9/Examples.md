# Лекція 8-9 — Бінарні дерева, BST та AVL-дерева

> **Тривалість:** ≈ 196 хвилин (дві пари) з трьома перервами.
> **Мова прикладів:** C# (.NET, `Nullable` увімкнено, жодних попереджень компілятора).
> **Як запускати приклади:** кожен блок, що починається з коментаря `// Файл: ИмяФайлу.cs`, — це самодостатня програма. Збережіть його у файл і виконайте `dotnet run ИмяФайлу.cs` (file-based apps, .NET 10+). Результат наведено у блоці «Приклад запуску».

---

## Зміст

| № | Розділ | ≈ хв |
|---|--------|------|
| 1 | [Дерева: термінологія](#1-дерева-термінологія) | 10 |
| 2 | [Типи бінарних дерев](#2-типи-бінарних-дерев) | 8 |
| 3 | [Представлення дерев у пам'яті](#3-представлення-дерев-у-памяті) | 10 |
| 4 | [Рекурсивні обходи](#4-рекурсивні-обходи) | 10 |
| 5 | [Ітеративні обходи та Morris](#5-ітеративні-обходи-та-morris) | 15 |
| | ☕ **Перерва 1** (≈ 53 хв від початку) | — |
| 6 | [Обхід у ширину: рівні, зигзаг, вертикальний вигляд](#6-обхід-у-ширину-рівні-зигзаг-вертикальний-вигляд) | 12 |
| 7 | [Метрики дерева та рекурсивні патерни](#7-метрики-дерева-та-рекурсивні-патерни) | 20 |
| 8 | [Побудова, серіалізація та друк дерева](#8-побудова-серіалізація-та-друк-дерева) | 13 |
| | ☕ **Перерва 2** (≈ 98 хв) | — |
| 9 | [Бінарне дерево пошуку (BST): базові операції](#9-бінарне-дерево-пошуку-bst-базові-операції) | 20 |
| 10 | [BST: розширені операції](#10-bst-розширені-операції) | 18 |
| 11 | [Вироджений BST: чому потрібне балансування](#11-вироджений-bst-чому-потрібне-балансування) | 5 |
| | ☕ **Перерва 3** (≈ 141 хв) | — |
| 12 | [AVL-дерева](#12-avl-дерева) | 28 |
| 13 | [Червоно-чорні дерева, B-дерева, префіксні дерева](#13-червоно-чорні-дерева-b-дерева-префіксні-дерева) | 12 |
| 14 | [Дерева в .NET: SortedSet і SortedDictionary](#14-дерева-в-net-sortedset-і-sorteddictionary) | 8 |
| 15 | [Підсумки, питання для самоперевірки, практичні задачі](#15-підсумки-питання-для-самоперевірки-практичні-задачі) | 7 |
| | **Разом** | **≈ 196** |

---

## 1. Дерева: термінологія

*≈ 10 хв*

### 1.1 Що таке дерево

**Дерево** — це ієрархічна структура даних, що складається з **вузлів** (nodes), з'єднаних **ребрами** (edges), причому:

- існує рівно один виділений вузол — **корінь** (root);
- кожен вузол, крім кореня, має рівно одного **батька** (parent);
- від кореня до будь-якого вузла існує **єдиний** шлях (немає циклів).

Дерево з `n` вузлами завжди має рівно `n − 1` ребро.

**Бінарне дерево** — дерево, у якому кожен вузол має **не більше двох** дітей: **лівого** (left) і **правого** (right). Порядок має значення: «лише лівий син» і «лише правий син» — це різні дерева.

Де зустрічаються дерева:

- файлова система (каталоги й файли);
- DOM у браузері, XML/JSON-документи;
- синтаксичні дерева компілятора (Roslyn для C#);
- індекси баз даних (B+-дерева);
- `SortedSet<T>`, `SortedDictionary<TKey,TValue>` у .NET (червоно-чорні дерева);
- купи (heap) для черг з пріоритетом, дерева Гаффмана, дерева рішень у ML.

### 1.2 Базові терміни

Розглянемо дерево, яке будемо використовувати в багатьох прикладах:

```
                 1            ← рівень 0 (корінь)
               /   \
              2     3         ← рівень 1
             / \     \
            4   5     6       ← рівень 2
               /
              7               ← рівень 3
```

| Термін | Англ. | Визначення | Приклад на рисунку |
|--------|-------|-----------|--------------------|
| Корінь | root | Вузол без батька | `1` |
| Лист | leaf | Вузол без дітей | `4`, `7`, `6` |
| Внутрішній вузол | internal node | Вузол, що має хоча б одну дитину | `1`, `2`, `3`, `5` |
| Батько / дитина | parent / child | `u` — батько `v`, якщо є ребро `u → v` | `2` — батько `5` |
| Брати | siblings | Діти одного батька | `4` і `5` |
| Предок / нащадок | ancestor / descendant | Вузол на шляху від кореня / нижче за шляхом | `1`, `2`, `5` — предки `7` |
| Піддерево | subtree | Вузол разом з усіма нащадками | піддерево `2` = {2, 4, 5, 7} |
| Степінь вузла | degree | Кількість дітей вузла | deg(2)=2, deg(3)=1, deg(4)=0 |
| Глибина вузла | depth | Кількість **ребер** від кореня до вузла | depth(7)=3 |
| Рівень | level | Множина вузлів однакової глибини | рівень 2 = {4, 5, 6} |
| Висота вузла | height | Кількість **ребер** на найдовшому шляху від вузла до листа | height(2)=2, height(4)=0 |
| Висота дерева | tree height | Висота кореня | 3 |

> ⚠️ **Про домовленості.** У літературі висоту рахують або в **ребрах** (лист має висоту 0, порожнє дерево — −1), або у **вузлах** (лист — 1, порожнє — 0). У цій лекції:
> - у розділах 1–11 висота рахується **в ребрах** (порожнє дерево = −1), якщо не сказано інше;
> - в AVL-дереві (розділ 12) зручніше зберігати висоту **у вузлах** (лист = 1, `null` = 0). Різниця висот від цього не змінюється.

Глибина — «скільки кроків донизу від кореня», висота — «скільки кроків до найдальшого листа». Для кореня depth = 0, для листа height = 0.

### 1.3 Узагальнений вузол `TreeNode<T>`

У C# вузол — це клас із **nullable-посиланнями** на дітей (`TreeNode<T>?`). Звільняти пам'ять не потрібно: вузол без посилань прибере збирач сміття (GC).

```csharp
// Файл: TreeTerminology.cs
// Демонстрація базових термінів на дереві з рисунка 1.2.

var root = new TreeNode<int>(1,
    new TreeNode<int>(2,
        new TreeNode<int>(4),
        new TreeNode<int>(5, new TreeNode<int>(7), null)),
    new TreeNode<int>(3,
        null,
        new TreeNode<int>(6)));

Console.WriteLine($"Корінь: {root.Value}");
Console.WriteLine($"Висота дерева (у ребрах): {TreeInfo.Height(root)}");
Console.WriteLine();
Console.WriteLine("Вузол | глибина | висота | степінь | лист?");

// Обходимо всі вузли й друкуємо характеристики кожного.
TreeInfo.Describe(root, depth: 0);

/// <summary>Вузол бінарного дерева з довільним типом значення.</summary>
public sealed class TreeNode<T>(T value, TreeNode<T>? left = null, TreeNode<T>? right = null)
{
    public T Value { get; set; } = value;

    // null означає «дитини немає».
    public TreeNode<T>? Left { get; set; } = left;
    public TreeNode<T>? Right { get; set; } = right;

    // Степінь вузла = кількість ненульових дітей.
    public int Degree => (Left is null ? 0 : 1) + (Right is null ? 0 : 1);

    public bool IsLeaf => Left is null && Right is null;
}

public static class TreeInfo
{
    // Висота в ребрах: порожнє дерево має висоту -1, лист — 0.
    public static int Height<T>(TreeNode<T>? node) =>
        node is null ? -1 : 1 + Math.Max(Height(node.Left), Height(node.Right));

    // Глибину передаємо «згори вниз» як параметр рекурсії.
    public static void Describe<T>(TreeNode<T>? node, int depth)
    {
        if (node is null)
        {
            return;
        }

        Console.WriteLine(
            $"{node.Value,5} | {depth,7} | {Height(node),6} | {node.Degree,7} | {(node.IsLeaf ? "так" : "ні")}");

        Describe(node.Left, depth + 1);
        Describe(node.Right, depth + 1);
    }
}
```

**Приклад запуску:**

```text
Корінь: 1
Висота дерева (у ребрах): 3

Вузол | глибина | висота | степінь | лист?
    1 |       0 |      3 |       2 | ні
    2 |       1 |      2 |       2 | ні
    4 |       2 |      0 |       0 | так
    5 |       2 |      1 |       1 | ні
    7 |       3 |      0 |       0 | так
    3 |       1 |      1 |       1 | ні
    6 |       2 |      0 |       0 | так
```

> 💡 Зверніть увагу на два напрямки передачі інформації в рекурсії:
> - **глибина** передається **згори вниз** (параметр `depth + 1`);
> - **висота** обчислюється **знизу вгору** (результат з дітей → батьку).
>
> Майже всі задачі на деревах — це комбінація цих двох патернів.

### Типові помилки

1. **Плутати глибину й висоту.** Глибина рахується від кореня, висота — до листа.
2. **Змішувати домовленості** «в ребрах» і «у вузлах» в одній програмі — отримуєте «зсув на 1» у формулах.
3. **Забувати про `null`**: з увімкненим `Nullable` компілятор попередить про `node.Left.Value` без перевірки — не глушіть попередження оператором `!` без причини.

---

## 2. Типи бінарних дерев

*≈ 8 хв*

### 2.1 Класифікація з рисунками

**Повне (full, strictly binary)** — кожен вузол має **0 або 2** дітей.

```
        A
       / \
      B   C
         / \
        D   E
```

**Завершене (complete)** — усі рівні заповнені повністю, крім, можливо, останнього, який заповнюється **зліва направо** без «дірок». Саме така форма в бінарної купи.

```
          A
        /   \
       B     C
      / \   /
     D   E F
```

**Досконале (perfect)** — усі внутрішні вузли мають 2 дітей і **всі листи на одній глибині**. Для висоти `h` (у ребрах) має рівно `2^(h+1) − 1` вузлів.

```
          A
        /   \
       B     C
      / \   / \
     D   E F   G
```

**Вироджене (degenerate, pathological)** — кожен вузол має не більше однієї дитини; фактично це зв'язний список, висота `n − 1`.

```
    A
     \
      B
       \
        C
         \
          D
```

**Збалансоване (height-balanced)** — для **кожного** вузла висоти лівого й правого піддерев відрізняються не більше ніж на 1. Висота такого дерева — `O(log n)`.

```
          A              (збалансоване: у кожному вузлі |hL − hR| ≤ 1)
        /   \
       B     C
      /     / \
     D     E   F
```

Співвідношення між класами:

```
  perfect  ⊂  complete  ⊂  balanced
  perfect  ⊂  full
  degenerate (n ≥ 3) — не збалансоване
```

| Тип | Властивість | Типове застосування |
|-----|-------------|---------------------|
| Full | 0 або 2 дітей | дерева виразів, дерево Гаффмана |
| Complete | рівні заповнені зліва направо | бінарна купа, зберігання в масиві |
| Perfect | повністю заповнені рівні | теоретичний аналіз, сегментні дерева |
| Degenerate | ланцюжок | найгірший випадок BST |
| Balanced | \|hL − hR\| ≤ 1 всюди | AVL, гарантія `O(log n)` |

### 2.2 Перевірка типу дерева в коді

```csharp
// Файл: TreeKinds.cs
// Перевіряємо, до яких класів належать кілька дерев.

// Повне, але не завершене: у B немає дітей, а в C є.
var full = N("A", N("B"), N("C", N("D"), N("E")));

// Завершене, але не повне: C має лише лівого сина.
var complete = N("A", N("B", N("D"), N("E")), N("C", N("F"), null));

// Досконале.
var perfect = N("A", N("B", N("D"), N("E")), N("C", N("F"), N("G")));

// Вироджене: ланцюжок праворуч.
var degenerate = N("A", null, N("B", null, N("C", null, N("D"))));

foreach (var (name, tree) in new[] { ("full", full), ("complete", complete), ("perfect", perfect), ("degenerate", degenerate) })
{
    Console.WriteLine(
        $"{name,-10}: full={Kinds.IsFull(tree),-5} complete={Kinds.IsComplete(tree),-5} " +
        $"perfect={Kinds.IsPerfect(tree),-5} degenerate={Kinds.IsDegenerate(tree),-5} balanced={Kinds.IsBalanced(tree)}");
}

static TreeNode<string> N(string value, TreeNode<string>? left = null, TreeNode<string>? right = null) =>
    new(value, left, right);

public sealed class TreeNode<T>(T value, TreeNode<T>? left = null, TreeNode<T>? right = null)
{
    public T Value { get; set; } = value;
    public TreeNode<T>? Left { get; set; } = left;
    public TreeNode<T>? Right { get; set; } = right;
}

public static class Kinds
{
    // Повне: у кожного вузла 0 або 2 дітей.
    public static bool IsFull<T>(TreeNode<T>? node)
    {
        if (node is null)
        {
            return true;
        }

        // Рівно одна дитина — порушення.
        if ((node.Left is null) != (node.Right is null))
        {
            return false;
        }

        return IsFull(node.Left) && IsFull(node.Right);
    }

    // Завершене: обхід у ширину; після першого "null" не повинно бути жодного вузла.
    public static bool IsComplete<T>(TreeNode<T>? root)
    {
        var queue = new Queue<TreeNode<T>?>();
        queue.Enqueue(root);
        var seenNull = false;

        while (queue.Count > 0)
        {
            var node = queue.Dequeue();
            if (node is null)
            {
                seenNull = true;
                continue;
            }

            // Вузол після "дірки" — дерево не завершене.
            if (seenNull)
            {
                return false;
            }

            queue.Enqueue(node.Left);
            queue.Enqueue(node.Right);
        }

        return true;
    }

    // Досконале: кількість вузлів дорівнює 2^(h+1) - 1.
    public static bool IsPerfect<T>(TreeNode<T>? root)
    {
        var height = Height(root);
        return Size(root) == (1 << (height + 1)) - 1;
    }

    // Вироджене: жоден вузол не має двох дітей.
    public static bool IsDegenerate<T>(TreeNode<T>? node)
    {
        while (node is not null)
        {
            if (node.Left is not null && node.Right is not null)
            {
                return false;
            }

            node = node.Left ?? node.Right;
        }

        return true;
    }

    // Збалансоване (наївна версія O(n^2); ефективну O(n) побачимо в розділі 7).
    public static bool IsBalanced<T>(TreeNode<T>? node) =>
        node is null ||
        (Math.Abs(Height(node.Left) - Height(node.Right)) <= 1 && IsBalanced(node.Left) && IsBalanced(node.Right));

    public static int Height<T>(TreeNode<T>? node) =>
        node is null ? -1 : 1 + Math.Max(Height(node.Left), Height(node.Right));

    public static int Size<T>(TreeNode<T>? node) =>
        node is null ? 0 : 1 + Size(node.Left) + Size(node.Right);
}
```

**Приклад запуску:**

```text
full      : full=True  complete=False perfect=False degenerate=False balanced=True
complete  : full=False complete=True  perfect=False degenerate=False balanced=True
perfect   : full=True  complete=True  perfect=True  degenerate=False balanced=True
degenerate: full=False complete=False perfect=False degenerate=True  balanced=False
```

### 2.3 Корисні формули

| Факт | Формула |
|------|---------|
| Максимум вузлів на рівні `k` | `2^k` |
| Максимум вузлів у дереві висоти `h` | `2^(h+1) − 1` |
| Мінімальна висота дерева з `n` вузлами | `⌊log₂ n⌋` |
| Максимальна висота дерева з `n` вузлами | `n − 1` (вироджене) |
| У повному бінарному дереві | листів = внутрішніх вузлів + 1 |
| Кількість різних форм бінарних дерев з `n` вузлами | число Каталана `C(n) = (2n)! / ((n+1)! n!)` |

### Міні-вправа 2.1

Намалюйте всі різні форми бінарних дерев із 3 вузлами. Скільки їх?

<details>
<summary>Розв'язок</summary>

Їх `C(3) = 5`:

```
      o        o          o          o        o
     /        /          / \          \        \
    o        o          o   o          o        o
   /          \                       /          \
  o            o                     o            o
```

Лише одна з п'яти форм (середня) збалансована й досконала; решта чотири — вироджені ланцюжки.

</details>

---

## 3. Представлення дерев у пам'яті

*≈ 10 хв*

### 3.1 Вузли з посиланнями (node references)

Найпоширеніше представлення: кожен вузол — об'єкт у купі з посиланнями `Left` і `Right`.

```
  root ──► ┌───────┬───┬───────┐
           │ Left  │ 1 │ Right │
           └───┬───┴───┴───┬───┘
               ▼           ▼
        ┌──────┬───┬─────┐ ┌──────┬───┬──────┐
        │ null │ 2 │ null│ │ null │ 3 │ null │
        └──────┴───┴─────┘ └──────┴───┴──────┘
```

- ✅ Довільна форма дерева, легкі вставка/видалення.
- ❌ Накладні витрати: заголовок об'єкта + 2 посилання (≈ 32+ байти на вузол у 64-біт .NET), погана локальність кешу.

### 3.2 Масив (неявне дерево, heap indexing)

Для **завершеного** дерева вузли можна зберігати в масиві у порядку обходу в ширину. Для індексу `i` (нумерація з 0):

```
  left(i)   = 2i + 1
  right(i)  = 2i + 2
  parent(i) = (i - 1) / 2      (цілочисельне ділення)
```

```
Дерево:              Масив:
          10          індекс:  0   1   2   3   4   5
        /    \                ┌───┬───┬───┬───┬───┬───┐
      20      30      значення│10 │20 │30 │40 │50 │60 │
     /  \    /                └───┴───┴───┴───┴───┴───┘
   40    50 60                  │   │   │   ▲   ▲   ▲
                                │   └───┼───┴───┘   │   діти 1: 3, 4
                                └───────┴───────────┘   діти 2: 5, (6)
```

- ✅ Немає посилань, чудова локальність, так реалізовано `PriorityQueue<TElement,TPriority>` у .NET.
- ❌ Для незавершених дерев — «дірки» й марна пам'ять (вироджене дерево з `n` вузлами потребує масиву розміру `2^n − 1`).

### 3.3 Посилання на батька (parent pointers)

Іноді у вузол додають поле `Parent`. Це дозволяє підніматися вгору без стеку: знаходити наступника в BST, шлях до кореня, LCA за `O(h)` без рекурсії. Ціна — ще одне посилання і необхідність оновлювати його при кожній зміні структури (часте джерело багів!).

```
          1
        ↗ ↓ ↘ ↑
       2       3        ↓ — Left/Right,  ↑ — Parent
```

### 3.4 Код: масив ↔ вузли, батьківські посилання

```csharp
// Файл: TreeRepresentations.cs
// 1) Навігація в масивному представленні.
// 2) Перетворення масиву у вузли.
// 3) Вузли з посиланням на батька: шлях до кореня.

int?[] heap = [10, 20, 30, 40, 50, 60];

Console.WriteLine("== Масив (heap indexing) ==");
for (var i = 0; i < heap.Length; i++)
{
    var left = 2 * i + 1;
    var right = 2 * i + 2;
    var parent = i == 0 ? "—" : heap[(i - 1) / 2].ToString();

    // Якщо індекс за межами масиву — дитини немає.
    var leftText = left < heap.Length ? heap[left].ToString() : "—";
    var rightText = right < heap.Length ? heap[right].ToString() : "—";
    Console.WriteLine($"i={i}: value={heap[i]}, parent={parent}, left={leftText}, right={rightText}");
}

Console.WriteLine();
Console.WriteLine("== Масив -> вузли (з null як «діркою») ==");
int?[] sparse = [1, 2, 3, null, 5, null, 6];
var root = ArrayTree.Build(sparse, 0, parent: null);
Console.Write("Прямий обхід: ");
ArrayTree.PrintPreorder(root);
Console.WriteLine();

Console.WriteLine();
Console.WriteLine("== Шлях від вузла до кореня через Parent ==");
var six = root!.Right!.Right!;
for (var node = six; node is not null; node = node.Parent)
{
    Console.Write(node.Parent is null ? $"{node.Value}\n" : $"{node.Value} -> ");
}

/// <summary>Вузол із додатковим посиланням на батька.</summary>
public sealed class ParentNode(int value, ParentNode? parent)
{
    public int Value { get; } = value;
    public ParentNode? Left { get; set; }
    public ParentNode? Right { get; set; }
    public ParentNode? Parent { get; } = parent;
}

public static class ArrayTree
{
    // Рекурсивно будуємо вузол для індексу index; діти — 2i+1 та 2i+2.
    public static ParentNode? Build(int?[] values, int index, ParentNode? parent)
    {
        if (index >= values.Length || values[index] is not int value)
        {
            return null;
        }

        var node = new ParentNode(value, parent);
        node.Left = Build(values, 2 * index + 1, node);
        node.Right = Build(values, 2 * index + 2, node);
        return node;
    }

    public static void PrintPreorder(ParentNode? node)
    {
        if (node is null)
        {
            return;
        }

        Console.Write($"{node.Value} ");
        PrintPreorder(node.Left);
        PrintPreorder(node.Right);
    }
}
```

**Приклад запуску:**

```text
== Масив (heap indexing) ==
i=0: value=10, parent=—, left=20, right=30
i=1: value=20, parent=10, left=40, right=50
i=2: value=30, parent=10, left=60, right=—
i=3: value=40, parent=20, left=—, right=—
i=4: value=50, parent=20, left=—, right=—
i=5: value=60, parent=30, left=—, right=—

== Масив -> вузли (з null як «діркою») ==
Прямий обхід: 1 2 5 3 6 

== Шлях від вузла до кореня через Parent ==
6 -> 3 -> 1
```

> ⚠️ Зауважте: формат `[1, 2, 3, null, 5, null, 6]` тут — це **позиції в масиві купи** (для кожного `null` місце його «дітей» теж зарезервоване). У розділі 8 ми використаємо компактніший формат LeetCode, де діти `null` не записуються.

### Порівняння представлень

| Критерій | Посилання | Масив | + Parent |
|----------|-----------|-------|----------|
| Пам'ять на вузол | 2 посилання | 0 посилань | 3 посилання |
| Довільна форма | ✅ | ❌ (дірки) | ✅ |
| Перехід до батька | ❌ (потрібен стек) | ✅ `(i−1)/2` | ✅ |
| Локальність кешу | погана | відмінна | погана |
| Типове використання | BST, AVL, RB | купа, сегментне дерево | `SortedSet` ітератори, RB-дерева в підручниках |

---

## 4. Рекурсивні обходи

*≈ 10 хв*

### 4.1 Три порядки DFS

**Обхід у глибину (DFS)** відрізняється лише моментом, коли ми «обробляємо» поточний вузол:

| Обхід | Порядок | Мнемоніка | Застосування |
|-------|---------|-----------|--------------|
| Прямий (preorder) | **Вузол**, Ліве, Праве | N-L-R | копіювання/серіалізація дерева, префіксний запис виразу |
| Центрований (inorder) | Ліве, **Вузол**, Праве | L-N-R | BST → відсортована послідовність |
| Зворотний (postorder) | Ліве, Праве, **Вузол** | L-R-N | видалення дерева, обчислення розмірів/висот, постфіксний запис |

Для дерева з рисунка 1.2:

```
                 1
               /   \
              2     3
             / \     \
            4   5     6
               /
              7

Preorder : 1 2 4 5 7 3 6
Inorder  : 4 2 7 5 1 3 6
Postorder: 4 7 5 2 6 3 1
```

**Трюк «обведення контуру»:** обведіть дерево олівцем проти годинникової стрілки, починаючи ліворуч від кореня. Preorder — записуємо вузол, коли проходимо **ліворуч** від нього; inorder — **під** ним; postorder — **праворуч**.

### 4.2 Код

```csharp
// Файл: RecursiveTraversals.cs
// Три рекурсивні обходи + обхід через делегат (узагальнений «відвідувач»).

var root = new TreeNode<int>(1,
    new TreeNode<int>(2, new TreeNode<int>(4), new TreeNode<int>(5, new TreeNode<int>(7))),
    new TreeNode<int>(3, null, new TreeNode<int>(6)));

Console.WriteLine("Preorder : " + string.Join(" ", Traversals.Preorder(root)));
Console.WriteLine("Inorder  : " + string.Join(" ", Traversals.Inorder(root)));
Console.WriteLine("Postorder: " + string.Join(" ", Traversals.Postorder(root)));

// Дерево виразу (3 + 4) * (5 - 2): різні обходи дають різні нотації.
var expression = new TreeNode<string>("*",
    new TreeNode<string>("+", new TreeNode<string>("3"), new TreeNode<string>("4")),
    new TreeNode<string>("-", new TreeNode<string>("5"), new TreeNode<string>("2")));

Console.WriteLine();
Console.WriteLine("Префіксна (preorder)  : " + string.Join(" ", Traversals.Preorder(expression)));
Console.WriteLine("Інфіксна (inorder)    : " + string.Join(" ", Traversals.Inorder(expression)));
Console.WriteLine("Постфіксна (postorder): " + string.Join(" ", Traversals.Postorder(expression)));
Console.WriteLine("Значення виразу       : " + Traversals.Evaluate(expression));

public sealed class TreeNode<T>(T value, TreeNode<T>? left = null, TreeNode<T>? right = null)
{
    public T Value { get; set; } = value;
    public TreeNode<T>? Left { get; set; } = left;
    public TreeNode<T>? Right { get; set; } = right;
}

public static class Traversals
{
    public static List<T> Preorder<T>(TreeNode<T>? root)
    {
        var result = new List<T>();
        Visit(root);
        return result;

        // Локальна функція бачить змінну result — не треба передавати список параметром.
        void Visit(TreeNode<T>? node)
        {
            if (node is null)
            {
                return;             // базовий випадок
            }

            result.Add(node.Value); // N
            Visit(node.Left);       // L
            Visit(node.Right);      // R
        }
    }

    public static List<T> Inorder<T>(TreeNode<T>? root)
    {
        var result = new List<T>();
        Visit(root);
        return result;

        void Visit(TreeNode<T>? node)
        {
            if (node is null)
            {
                return;
            }

            Visit(node.Left);       // L
            result.Add(node.Value); // N
            Visit(node.Right);      // R
        }
    }

    public static List<T> Postorder<T>(TreeNode<T>? root)
    {
        var result = new List<T>();
        Visit(root);
        return result;

        void Visit(TreeNode<T>? node)
        {
            if (node is null)
            {
                return;
            }

            Visit(node.Left);       // L
            Visit(node.Right);      // R
            result.Add(node.Value); // N
        }
    }

    // Обчислення виразу — класичний postorder: спочатку діти, потім операція.
    public static int Evaluate(TreeNode<string> node)
    {
        if (node.Left is null || node.Right is null)
        {
            return int.Parse(node.Value);
        }

        var left = Evaluate(node.Left);
        var right = Evaluate(node.Right);
        return node.Value switch
        {
            "+" => left + right,
            "-" => left - right,
            "*" => left * right,
            "/" => left / right,
            _ => throw new InvalidOperationException($"Невідомий оператор {node.Value}"),
        };
    }
}
```

**Приклад запуску:**

```text
Preorder : 1 2 4 5 7 3 6
Inorder  : 4 2 7 5 1 3 6
Postorder: 4 7 5 2 6 3 1

Префіксна (preorder)  : * + 3 4 - 5 2
Інфіксна (inorder)    : 3 + 4 * 5 - 2
Постфіксна (postorder): 3 4 + 5 2 - *
Значення виразу       : 21
```

### 4.3 Складність

- Час: **O(n)** — кожен вузол відвідується рівно один раз.
- Пам'ять: **O(h)** на стек викликів, де `h` — висота. Для збалансованого дерева це `O(log n)`, для виродженого — `O(n)`.

> ⚠️ Стек потоку в .NET за замовчуванням ≈ 1 МБ. Рекурсивний обхід виродженого дерева з ~100 000 вузлів може завершитися `StackOverflowException`, який **неможливо перехопити** `try/catch` — процес просто падає. Тому важливо знати ітеративні версії.

### Типові помилки

- Забути базовий випадок `if (node is null) return;` → `NullReferenceException`.
- Створювати новий `List<T>` у кожному рекурсивному виклику й конкатенувати: `Inorder(l).Concat(...)` — це `O(n·h)` пам'яті й часу.
- Використовувати `yield return` у рекурсії наївно: кожен рівень створює свій ітератор, і загальна вартість стає `O(n·h)`.

### Міні-вправа 4.1

Дано preorder `A B D E C F` та inorder `D B E A F C`. Відновіть дерево й запишіть postorder.

<details>
<summary>Розв'язок</summary>

Перший елемент preorder — корінь `A`. В inorder ліворуч від `A` — ліве піддерево `{D, B, E}`, праворуч — `{F, C}`. Рекурсивно: корінь лівого — `B` (наступний у preorder), діти `D` і `E`; корінь правого — `C`, лівий син `F`.

```
        A
       / \
      B   C
     / \  /
    D  E F
```

Postorder: `D E B F C A`. Алгоритм у коді — у розділі 8.

</details>

---

## 5. Ітеративні обходи та Morris

*≈ 15 хв*

### 5.1 Ідея: явний стек замість стеку викликів

Рекурсія неявно зберігає «куди повернутися» у стеку викликів. Ми можемо зберігати це самостійно в `Stack<TreeNode<T>>` — він живе в купі, тож обмежений лише пам'яттю.

**Preorder:** покласти корінь; поки стек не порожній — зняти вузол, обробити, покласти **спочатку правого, потім лівого** (щоб лівий опинився зверху).

**Inorder:** «спускайся ліворуч, складаючи вузли в стек; коли далі нікуди — зніми вузол, обробь, перейди праворуч».

```
Трасування inorder для дерева 1(2(4,5(7)),3(,6)):

крок  дія                      стек (верх праворуч)   вивід
 1    спуск ліворуч від 1      [1, 2, 4]
 2    pop 4, праворуч null     [1, 2]                  4
 3    pop 2, перейти до 5      [1]                     4 2
 4    спуск ліворуч від 5      [1, 5, 7]
 5    pop 7                    [1, 5]                  4 2 7
 6    pop 5                    [1]                     4 2 7 5
 7    pop 1, перейти до 3      [3]                     4 2 7 5 1
 8    pop 3, перейти до 6      [6]                     4 2 7 5 1 3
 9    pop 6                    []                      4 2 7 5 1 3 6
```

**Postorder:** два варіанти:
1. **Два стеки** (або стек + розворот): робимо «дзеркальний preorder» N-R-L і розвертаємо результат → L-R-N.
2. **Один стек + `lastVisited`**: знімаємо вузол лише тоді, коли його правий син відсутній або вже оброблений.

### 5.2 Morris inorder — O(1) додаткової пам'яті

Ідея Морріса: тимчасово **прошивати** дерево. Для поточного вузла `cur` знаходимо його inorder-попередника `pre` (найправіший вузол лівого піддерева) і ставимо `pre.Right = cur` — «нитку» для повернення. Коли повернемося по нитці вдруге — прибираємо її.

```
Перед:          Після прошивання (для cur = 1):
     1                 1
    / \               / \
   2   3             2   3
    \                 \
     5                 5
                        ╲
                         ↺ (5.Right → 1, тимчасово)
```

- Час `O(n)` (кожне ребро проходиться не більше 3 разів), пам'ять `O(1)`.
- Дерево **тимчасово модифікується** → не можна використовувати паралельно з іншими читачами.

### 5.3 Код

```csharp
// Файл: IterativeTraversals.cs
// Ітеративні preorder, inorder, postorder (2 способи) та Morris inorder.

var root = new TreeNode<int>(1,
    new TreeNode<int>(2, new TreeNode<int>(4), new TreeNode<int>(5, new TreeNode<int>(7))),
    new TreeNode<int>(3, null, new TreeNode<int>(6)));

Console.WriteLine("Preorder (стек)          : " + string.Join(" ", Iterative.Preorder(root)));
Console.WriteLine("Inorder (стек)           : " + string.Join(" ", Iterative.Inorder(root)));
Console.WriteLine("Postorder (розворот)     : " + string.Join(" ", Iterative.PostorderReversed(root)));
Console.WriteLine("Postorder (1 стек)       : " + string.Join(" ", Iterative.PostorderOneStack(root)));
Console.WriteLine("Inorder (Morris, O(1))   : " + string.Join(" ", Iterative.MorrisInorder(root)));
Console.WriteLine("Після Morris дерево ціле : " + string.Join(" ", Iterative.Inorder(root)));

// Глибоке вироджене дерево: рекурсія тут впала б із StackOverflow.
TreeNode<int>? chain = null;
for (var i = 1_000_000; i >= 1; i--)
{
    chain = new TreeNode<int>(i, chain, null); // ланцюжок ліворуч
}

var deep = Iterative.Inorder(chain);
Console.WriteLine($"Ланцюжок з {deep.Count} вузлів: перший={deep[0]}, останній={deep[^1]}");

public sealed class TreeNode<T>(T value, TreeNode<T>? left = null, TreeNode<T>? right = null)
{
    public T Value { get; set; } = value;
    public TreeNode<T>? Left { get; set; } = left;
    public TreeNode<T>? Right { get; set; } = right;
}

public static class Iterative
{
    public static List<T> Preorder<T>(TreeNode<T>? root)
    {
        var result = new List<T>();
        if (root is null)
        {
            return result;
        }

        var stack = new Stack<TreeNode<T>>();
        stack.Push(root);

        while (stack.Count > 0)
        {
            var node = stack.Pop();
            result.Add(node.Value);

            // Правого кладемо першим, щоб лівий був зверху і оброблявся раніше.
            if (node.Right is not null)
            {
                stack.Push(node.Right);
            }

            if (node.Left is not null)
            {
                stack.Push(node.Left);
            }
        }

        return result;
    }

    public static List<T> Inorder<T>(TreeNode<T>? root)
    {
        var result = new List<T>();
        var stack = new Stack<TreeNode<T>>();
        var current = root;

        while (current is not null || stack.Count > 0)
        {
            // 1. Спускаємося якомога лівіше, запам'ятовуючи шлях.
            while (current is not null)
            {
                stack.Push(current);
                current = current.Left;
            }

            // 2. Найлівіший необроблений вузол — обробляємо.
            current = stack.Pop();
            result.Add(current.Value);

            // 3. Переходимо в праве піддерево.
            current = current.Right;
        }

        return result;
    }

    // N-R-L з подальшим розворотом дає L-R-N.
    public static List<T> PostorderReversed<T>(TreeNode<T>? root)
    {
        var result = new List<T>();
        if (root is null)
        {
            return result;
        }

        var stack = new Stack<TreeNode<T>>();
        stack.Push(root);

        while (stack.Count > 0)
        {
            var node = stack.Pop();
            result.Add(node.Value);

            // Тепер навпаки: лівого першим, щоб правий оброблявся раніше.
            if (node.Left is not null)
            {
                stack.Push(node.Left);
            }

            if (node.Right is not null)
            {
                stack.Push(node.Right);
            }
        }

        result.Reverse();
        return result;
    }

    public static List<T> PostorderOneStack<T>(TreeNode<T>? root)
    {
        var result = new List<T>();
        var stack = new Stack<TreeNode<T>>();
        TreeNode<T>? lastVisited = null;
        var current = root;

        while (current is not null || stack.Count > 0)
        {
            while (current is not null)
            {
                stack.Push(current);
                current = current.Left;
            }

            var peek = stack.Peek();

            // Якщо праве піддерево є і ще не оброблене — йдемо туди.
            if (peek.Right is not null && lastVisited != peek.Right)
            {
                current = peek.Right;
            }
            else
            {
                // Обидва піддерева готові — обробляємо сам вузол.
                result.Add(peek.Value);
                lastVisited = stack.Pop();
            }
        }

        return result;
    }

    public static List<T> MorrisInorder<T>(TreeNode<T>? root)
    {
        var result = new List<T>();
        var current = root;

        while (current is not null)
        {
            if (current.Left is null)
            {
                // Лівого піддерева немає — обробляємо й ідемо праворуч (можливо, по нитці).
                result.Add(current.Value);
                current = current.Right;
                continue;
            }

            // Шукаємо inorder-попередника: найправіший вузол лівого піддерева.
            var predecessor = current.Left;
            while (predecessor.Right is not null && predecessor.Right != current)
            {
                predecessor = predecessor.Right;
            }

            if (predecessor.Right is null)
            {
                // Перший візит: прокладаємо нитку й спускаємося ліворуч.
                predecessor.Right = current;
                current = current.Left;
            }
            else
            {
                // Другий візит (повернулися по нитці): прибираємо нитку, обробляємо вузол.
                predecessor.Right = null;
                result.Add(current.Value);
                current = current.Right;
            }
        }

        return result;
    }
}
```

**Приклад запуску:**

```text
Preorder (стек)          : 1 2 4 5 7 3 6
Inorder (стек)           : 4 2 7 5 1 3 6
Postorder (розворот)     : 4 7 5 2 6 3 1
Postorder (1 стек)       : 4 7 5 2 6 3 1
Inorder (Morris, O(1))   : 4 2 7 5 1 3 6
Після Morris дерево ціле : 4 2 7 5 1 3 6
Ланцюжок з 1000000 вузлів: перший=1000000, останній=1
```

### Порівняння

| Метод | Час | Додаткова пам'ять | Змінює дерево | Ризик StackOverflow |
|-------|-----|-------------------|---------------|---------------------|
| Рекурсія | O(n) | O(h) стек викликів | ні | так |
| Явний `Stack<T>` | O(n) | O(h) у купі | ні | ні |
| Morris | O(n) | O(1) | тимчасово | ні |

### Типові помилки

- У preorder покласти в стек **спочатку лівого** → отримаєте N-R-L.
- В ітеративному inorder умова циклу лише `stack.Count > 0` → цикл не стартує, бо на початку стек порожній. Потрібно `current is not null || stack.Count > 0`.
- У Morris забути прибрати нитку (`predecessor.Right = null`) → дерево зіпсоване, наступний обхід зациклиться.
- У postorder з одним стеком порівнювати `lastVisited` зі значенням (`Value`), а не з посиланням — при дублікатах значень алгоритм зламається.

### Міні-вправа 5.1

Напишіть ітеративну функцію, яка повертає **k-й за порядком inorder** вузол (k з 1), зупиняючись одразу, щойно його знайдено.

<details>
<summary>Розв'язок</summary>

```csharp
// Файл: KthInorder.cs
var root = new Node(1,
    new Node(2, new Node(4), new Node(5, new Node(7))),
    new Node(3, null, new Node(6)));

for (var k = 1; k <= 8; k++)
{
    var found = KthInorder(root, k);
    Console.Write(found is null ? $"k={k}: немає\n" : $"k={k}: {found.Value}; ");
}

static Node? KthInorder(Node? root, int k)
{
    var stack = new Stack<Node>();
    var current = root;

    while (current is not null || stack.Count > 0)
    {
        while (current is not null)
        {
            stack.Push(current);
            current = current.Left;
        }

        current = stack.Pop();

        // Зменшуємо лічильник; щойно він став 0 — це шуканий вузол.
        if (--k == 0)
        {
            return current;
        }

        current = current.Right;
    }

    return null; // k більше за кількість вузлів
}

public sealed record Node(int Value, Node? Left = null, Node? Right = null);
```

```text
k=1: 4; k=2: 2; k=3: 7; k=4: 5; k=5: 1; k=6: 3; k=7: 6; k=8: немає
```

</details>

---

> ## ☕ Перерва 1 (≈ 10 хв)
>
> Пройдено ≈ 53 хв: термінологія, типи, представлення й усі обходи в глибину.

---

## 6. Обхід у ширину: рівні, зигзаг, вертикальний вигляд

*≈ 12 хв*

### 6.1 Level-order (BFS) з `Queue<T>`

**Обхід у ширину** відвідує вузли рівень за рівнем, зліва направо. Замість стеку використовуємо **чергу** (FIFO): вузли, знайдені раніше, обробляються раніше.

```
                 1                черга (голова ліворуч)       вивід
               /   \              [1]
              2     3             [2, 3]                       1
             / \     \            [3, 4, 5]                    1 2
            4   5     6           [4, 5, 6]                    1 2 3
               /                  [5, 6]                       1 2 3 4
              7                   [6, 7]                       1 2 3 4 5
                                  [7]                          1 2 3 4 5 6
                                  []                           1 2 3 4 5 6 7
```

**Групування по рівнях.** Ключовий прийом: на початку кожної ітерації зовнішнього циклу `queue.Count` — це рівно кількість вузлів поточного рівня. Знімаємо саме стільки — і отримуємо один рівень.

### 6.2 Зигзаг і вертикальний вигляд

- **Зигзаг (spiral):** рівні по черзі зліва направо та справа наліво. Найпростіше — звичайний BFS по рівнях, і розвертати кожен непарний рівень.
- **Вид справа (right side view):** останній вузол кожного рівня.
- **Вертикальний вигляд (vertical order):** призначаємо корню стовпчик 0, лівому сину `col − 1`, правому `col + 1`. Групуємо вузли за стовпчиком (BFS гарантує порядок «згори вниз»).

```
    стовпчик:  -2  -1   0   1   2
                        1
                    2       3
                4       5       6
                    7
Вертикально: [-2: 4] [-1: 2 7] [0: 1 5] [1: 3] [2: 6]
```

### 6.3 Код

```csharp
// Файл: LevelOrder.cs
// BFS: плоский обхід, рівні, зигзаг, вид справа, вертикальний порядок.

var root = new TreeNode<int>(1,
    new TreeNode<int>(2, new TreeNode<int>(4), new TreeNode<int>(5, new TreeNode<int>(7))),
    new TreeNode<int>(3, null, new TreeNode<int>(6)));

Console.WriteLine("Level-order : " + string.Join(" ", Bfs.LevelOrder(root)));

Console.WriteLine("По рівнях   :");
var levels = Bfs.Levels(root);
for (var i = 0; i < levels.Count; i++)
{
    Console.WriteLine($"  рівень {i}: [{string.Join(", ", levels[i])}]");
}

Console.WriteLine("Зигзаг      : " + string.Join(" | ", Bfs.Zigzag(root).Select(l => string.Join(" ", l))));
Console.WriteLine("Вид справа  : " + string.Join(" ", Bfs.RightSideView(root)));
Console.WriteLine("Вертикально : " + string.Join(" ",
    Bfs.VerticalOrder(root).Select(p => $"[{p.Key}: {string.Join(" ", p.Value)}]")));

public sealed class TreeNode<T>(T value, TreeNode<T>? left = null, TreeNode<T>? right = null)
{
    public T Value { get; set; } = value;
    public TreeNode<T>? Left { get; set; } = left;
    public TreeNode<T>? Right { get; set; } = right;
}

public static class Bfs
{
    public static List<T> LevelOrder<T>(TreeNode<T>? root)
    {
        var result = new List<T>();
        if (root is null)
        {
            return result;
        }

        var queue = new Queue<TreeNode<T>>();
        queue.Enqueue(root);

        while (queue.Count > 0)
        {
            var node = queue.Dequeue();
            result.Add(node.Value);

            if (node.Left is not null)
            {
                queue.Enqueue(node.Left);
            }

            if (node.Right is not null)
            {
                queue.Enqueue(node.Right);
            }
        }

        return result;
    }

    public static List<List<T>> Levels<T>(TreeNode<T>? root)
    {
        var result = new List<List<T>>();
        if (root is null)
        {
            return result;
        }

        var queue = new Queue<TreeNode<T>>();
        queue.Enqueue(root);

        while (queue.Count > 0)
        {
            // Фіксуємо розмір ДО циклу: діти, додані під час ітерації, належать наступному рівню.
            var levelSize = queue.Count;
            var level = new List<T>(levelSize);

            for (var i = 0; i < levelSize; i++)
            {
                var node = queue.Dequeue();
                level.Add(node.Value);

                if (node.Left is not null)
                {
                    queue.Enqueue(node.Left);
                }

                if (node.Right is not null)
                {
                    queue.Enqueue(node.Right);
                }
            }

            result.Add(level);
        }

        return result;
    }

    public static List<List<T>> Zigzag<T>(TreeNode<T>? root)
    {
        var levels = Levels(root);
        for (var i = 1; i < levels.Count; i += 2)
        {
            levels[i].Reverse(); // непарні рівні — справа наліво
        }

        return levels;
    }

    // Останній елемент кожного рівня видно, якщо дивитися справа.
    public static List<T> RightSideView<T>(TreeNode<T>? root) =>
        Levels(root).Select(level => level[^1]).ToList();

    public static SortedDictionary<int, List<T>> VerticalOrder<T>(TreeNode<T>? root)
    {
        // SortedDictionary тримає стовпчики впорядкованими за ключем.
        var columns = new SortedDictionary<int, List<T>>();
        if (root is null)
        {
            return columns;
        }

        var queue = new Queue<(TreeNode<T> Node, int Column)>();
        queue.Enqueue((root, 0));

        while (queue.Count > 0)
        {
            var (node, column) = queue.Dequeue();

            if (!columns.TryGetValue(column, out var list))
            {
                list = [];
                columns[column] = list;
            }

            list.Add(node.Value);

            if (node.Left is not null)
            {
                queue.Enqueue((node.Left, column - 1));
            }

            if (node.Right is not null)
            {
                queue.Enqueue((node.Right, column + 1));
            }
        }

        return columns;
    }
}
```

**Приклад запуску:**

```text
Level-order : 1 2 3 4 5 6 7
По рівнях   :
  рівень 0: [1]
  рівень 1: [2, 3]
  рівень 2: [4, 5, 6]
  рівень 3: [7]
Зигзаг      : 1 | 3 2 | 4 5 6 | 7
Вид справа  : 1 3 6 7
Вертикально : [-2: 4] [-1: 2 7] [0: 1 5] [1: 3] [2: 6]
```

### Складність і порівняння DFS/BFS

| | DFS (стек) | BFS (черга) |
|--|-----------|-------------|
| Час | O(n) | O(n) |
| Пам'ять | O(h) | O(w), `w` — максимальна ширина рівня |
| Вироджене дерево | O(n) | O(1) |
| Досконале дерево | O(log n) | O(n/2) = O(n) |
| Добре для | шляхи, піддерева, серіалізація | найкоротша відстань від кореня, рівні |

### Типові помилки

- Використовувати `queue.Count` як межу `for` **напряму**: `for (i = 0; i < queue.Count; i++)` — розмір змінюється під час циклу, рівні «злипаються».
- Застосувати `Stack<T>` замість `Queue<T>` — вийде вже не BFS.
- У вертикальному вигляді використати DFS без збереження глибини — порядок вузлів у стовпчику буде не «згори вниз».

### Міні-вправа 6.1

Знайдіть **максимальну ширину** дерева (найбільшу кількість вузлів на одному рівні) та номер цього рівня.

<details>
<summary>Розв'язок</summary>

```csharp
// Файл: MaxWidth.cs
var root = new Node(1,
    new Node(2, new Node(4), new Node(5, new Node(7))),
    new Node(3, null, new Node(6)));

var (width, level) = MaxWidth(root);
Console.WriteLine($"Максимальна ширина {width} на рівні {level}");

static (int Width, int Level) MaxWidth(Node? root)
{
    if (root is null)
    {
        return (0, -1);
    }

    var queue = new Queue<Node>();
    queue.Enqueue(root);
    var best = (Width: 0, Level: -1);

    for (var level = 0; queue.Count > 0; level++)
    {
        var size = queue.Count;
        if (size > best.Width)
        {
            best = (size, level);
        }

        for (var i = 0; i < size; i++)
        {
            var node = queue.Dequeue();
            if (node.Left is not null) queue.Enqueue(node.Left);
            if (node.Right is not null) queue.Enqueue(node.Right);
        }
    }

    return best;
}

public sealed record Node(int Value, Node? Left = null, Node? Right = null);
```

```text
Максимальна ширина 3 на рівні 2
```

</details>

---

## 7. Метрики дерева та рекурсивні патерни

*≈ 20 хв*

### 7.1 Шаблон «розв'яжи для дітей — скомбінуй»

Більшість задач на бінарних деревах розв'язуються за схемою:

```
Solve(node):
    if node == null: return <базове значення>
    left  = Solve(node.Left)
    right = Solve(node.Right)
    return Combine(node, left, right)
```

| Задача | Базове значення | Combine |
|--------|-----------------|---------|
| Кількість вузлів | 0 | `1 + left + right` |
| Висота (у ребрах) | −1 | `1 + max(left, right)` |
| Кількість листів | 0 | лист ? 1 : `left + right` |
| Сума значень | 0 | `node.Value + left + right` |
| Однакові дерева | обидва null → true | значення рівні && left && right |

### 7.2 «Повертаю одне — оновлюю інше» (діаметр, max path sum)

Деякі задачі вимагають **двох різних величин**:
- що функція **повертає батьку** (наприклад, найдовший шлях **униз** від вузла — лише в одну гілку);
- що вона **оновлює глобально** (найкращий шлях, що **проходить через** вузол — обидві гілки).

**Діаметр** — кількість ребер на найдовшому шляху між будь-якими двома вузлами (шлях не обов'язково проходить через корінь!).

```
Діаметр = 6 ребер, не через корінь:
            1
           /
          2
        /   \
       3     4
      /       \
     5         6
    /           \
   7             8        шлях 7-5-3-2-4-6-8
```

Для кожного вузла: `через_вузол = height(left) + height(right) + 2` (у ребрах, з height(null) = −1). Повертаємо батьку `1 + max(...)`.

**Максимальна сума шляху** — те саме, але ігноруємо гілки з від'ємним внеском: `gain = max(0, gain(child))`.

**Перевірка збалансованості за O(n).** Наївна версія (розділ 2) викликає `Height` для кожного вузла → O(n²) для виродженого дерева. Ефективна: одна функція повертає висоту **або** спеціальне значення «−2 = не збалансоване», і ми одразу «пробулькуємо» його вгору.

### 7.3 LCA — найнижчий спільний предок

**LCA(p, q)** — найглибший вузол, для якого і `p`, і `q` є нащадками (вузол вважається нащадком самого себе).

```
                 1
               /   \
              2     3
             / \     \
            4   5     6
               /
              7
LCA(4, 7) = 2      LCA(7, 6) = 1      LCA(5, 7) = 5
```

Рекурсивна ідея для **довільного** бінарного дерева:
- якщо `node` — це `p` або `q`, повертаємо `node`;
- шукаємо в лівому й правому піддеревах;
- якщо обидва результати ненульові — `p` і `q` в різних гілках, `node` і є LCA;
- інакше повертаємо ненульовий результат.

### 7.4 Код: розмір, висота, листи, діаметр, баланс, max path sum

```csharp
// Файл: TreeMetrics.cs
// Класичні рекурсивні метрики бінарного дерева.

var sample = new TreeNode<int>(1,
    new TreeNode<int>(2, new TreeNode<int>(4), new TreeNode<int>(5, new TreeNode<int>(7))),
    new TreeNode<int>(3, null, new TreeNode<int>(6)));

Console.WriteLine("== Дерево 1(2(4,5(7)),3(,6)) ==");
Console.WriteLine($"Розмір       : {Metrics.Size(sample)}");
Console.WriteLine($"Висота       : {Metrics.Height(sample)}");
Console.WriteLine($"Листи        : {Metrics.Leaves(sample)}");
Console.WriteLine($"Сума         : {Metrics.Sum(sample)}");
Console.WriteLine($"Діаметр      : {Metrics.Diameter(sample)}");
Console.WriteLine($"Збалансоване : {Metrics.IsBalanced(sample)}");

// Діаметр, що НЕ проходить через корінь.
var hook = new TreeNode<int>(1,
    new TreeNode<int>(2,
        new TreeNode<int>(3, new TreeNode<int>(5, new TreeNode<int>(7))),
        new TreeNode<int>(4, null, new TreeNode<int>(6, null, new TreeNode<int>(8)))));

Console.WriteLine();
Console.WriteLine("== «Гачок» з рисунка 7.2 ==");
Console.WriteLine($"Висота       : {Metrics.Height(hook)}");
Console.WriteLine($"Діаметр      : {Metrics.Diameter(hook)}");
Console.WriteLine($"Збалансоване : {Metrics.IsBalanced(hook)}");

//        -10
//        /  \
//       9    20
//           /  \
//          15   7        найкращий шлях 15 -> 20 -> 7 = 42
var negative = new TreeNode<int>(-10,
    new TreeNode<int>(9),
    new TreeNode<int>(20, new TreeNode<int>(15), new TreeNode<int>(7)));

Console.WriteLine();
Console.WriteLine($"Max path sum для -10(9,20(15,7)) : {Metrics.MaxPathSum(negative)}");
Console.WriteLine($"Max path sum для одного вузла -3  : {Metrics.MaxPathSum(new TreeNode<int>(-3))}");

public sealed class TreeNode<T>(T value, TreeNode<T>? left = null, TreeNode<T>? right = null)
{
    public T Value { get; set; } = value;
    public TreeNode<T>? Left { get; set; } = left;
    public TreeNode<T>? Right { get; set; } = right;
}

public static class Metrics
{
    public static int Size<T>(TreeNode<T>? node) =>
        node is null ? 0 : 1 + Size(node.Left) + Size(node.Right);

    public static int Height<T>(TreeNode<T>? node) =>
        node is null ? -1 : 1 + Math.Max(Height(node.Left), Height(node.Right));

    public static int Leaves<T>(TreeNode<T>? node) => node switch
    {
        null => 0,
        { Left: null, Right: null } => 1, // property pattern: вузол без дітей
        _ => Leaves(node.Left) + Leaves(node.Right),
    };

    public static int Sum(TreeNode<int>? node) =>
        node is null ? 0 : node.Value + Sum(node.Left) + Sum(node.Right);

    public static int Diameter<T>(TreeNode<T>? root)
    {
        var best = 0;
        HeightAndUpdate(root);
        return best;

        // Повертає висоту в ребрах, а побічно оновлює найкращий діаметр.
        int HeightAndUpdate(TreeNode<T>? node)
        {
            if (node is null)
            {
                return -1;
            }

            var left = HeightAndUpdate(node.Left);
            var right = HeightAndUpdate(node.Right);

            // Шлях через node: (left + 1) ребер ліворуч + (right + 1) ребер праворуч.
            best = Math.Max(best, left + right + 2);
            return 1 + Math.Max(left, right);
        }
    }

    private const int Unbalanced = -2;

    // O(n): одна функція і перевіряє баланс, і рахує висоту.
    public static bool IsBalanced<T>(TreeNode<T>? root) => CheckedHeight(root) != Unbalanced;

    private static int CheckedHeight<T>(TreeNode<T>? node)
    {
        if (node is null)
        {
            return -1;
        }

        var left = CheckedHeight(node.Left);
        if (left == Unbalanced)
        {
            return Unbalanced; // рання зупинка: далі рахувати нема сенсу
        }

        var right = CheckedHeight(node.Right);
        if (right == Unbalanced || Math.Abs(left - right) > 1)
        {
            return Unbalanced;
        }

        return 1 + Math.Max(left, right);
    }

    public static int MaxPathSum(TreeNode<int> root)
    {
        var best = int.MinValue; // шлях має містити хоча б один вузол
        Gain(root);
        return best;

        // Повертає найкращу суму шляху, що починається в node і йде ВНИЗ в одну гілку.
        int Gain(TreeNode<int>? node)
        {
            if (node is null)
            {
                return 0;
            }

            // Від'ємну гілку краще не брати зовсім.
            var left = Math.Max(0, Gain(node.Left));
            var right = Math.Max(0, Gain(node.Right));

            // Шлях, що «перегинається» через node, використовує обидві гілки.
            best = Math.Max(best, node.Value + left + right);

            // Батьку можна передати лише одну гілку.
            return node.Value + Math.Max(left, right);
        }
    }
}
```

**Приклад запуску:**

```text
== Дерево 1(2(4,5(7)),3(,6)) ==
Розмір       : 7
Висота       : 3
Листи        : 3
Сума         : 28
Діаметр      : 5
Збалансоване : True

== «Гачок» з рисунка 7.2 ==
Висота       : 4
Діаметр      : 6
Збалансоване : False

Max path sum для -10(9,20(15,7)) : 42
Max path sum для одного вузла -3  : -3
```

### 7.5 Код: LCA, дзеркало, симетрія, однакові дерева, шляхи з сумою

```csharp
// Файл: TreePatterns.cs
// LCA, mirror, symmetric, same tree, root-to-leaf path sum з друком шляхів.

var root = new TreeNode<int>(1,
    new TreeNode<int>(2, new TreeNode<int>(4), new TreeNode<int>(5, new TreeNode<int>(7))),
    new TreeNode<int>(3, null, new TreeNode<int>(6)));

var n4 = root.Left!.Left!;
var n5 = root.Left!.Right!;
var n7 = n5.Left!;
var n6 = root.Right!.Right!;

Console.WriteLine($"LCA(4, 7) = {Patterns.Lca(root, n4, n7)!.Value}");
Console.WriteLine($"LCA(7, 6) = {Patterns.Lca(root, n7, n6)!.Value}");
Console.WriteLine($"LCA(5, 7) = {Patterns.Lca(root, n5, n7)!.Value}");

//        1                 1
//      /   \             /   \
//     2     2           2     2
//    / \   / \           \     \
//   3   4 4   3           3     3
var symmetric = new TreeNode<int>(1,
    new TreeNode<int>(2, new TreeNode<int>(3), new TreeNode<int>(4)),
    new TreeNode<int>(2, new TreeNode<int>(4), new TreeNode<int>(3)));
var notSymmetric = new TreeNode<int>(1,
    new TreeNode<int>(2, null, new TreeNode<int>(3)),
    new TreeNode<int>(2, null, new TreeNode<int>(3)));

Console.WriteLine();
Console.WriteLine($"Симетричне 1(2(3,4),2(4,3))   : {Patterns.IsSymmetric(symmetric)}");
Console.WriteLine($"Симетричне 1(2(,3),2(,3))     : {Patterns.IsSymmetric(notSymmetric)}");

var copy = Patterns.Clone(root);
Console.WriteLine($"Копія однакова з оригіналом   : {Patterns.IsSame(root, copy)}");
Patterns.Mirror(copy);
Console.WriteLine($"Після Mirror однакова         : {Patterns.IsSame(root, copy)}");
Console.WriteLine($"Preorder дзеркала             : {string.Join(" ", Patterns.Preorder(copy))}");
Patterns.Mirror(copy);
Console.WriteLine($"Дзеркало двічі == оригінал    : {Patterns.IsSame(root, copy)}");

//            5
//          /   \
//         4     8
//        /     / \
//       11    13  4
//      /  \      / \
//     7    2    5   1
var pathTree = new TreeNode<int>(5,
    new TreeNode<int>(4, new TreeNode<int>(11, new TreeNode<int>(7), new TreeNode<int>(2))),
    new TreeNode<int>(8, new TreeNode<int>(13), new TreeNode<int>(4, new TreeNode<int>(5), new TreeNode<int>(1))));

Console.WriteLine();
Console.WriteLine($"Чи є шлях корінь->лист із сумою 22: {Patterns.HasPathSum(pathTree, 22)}");
Console.WriteLine("Усі такі шляхи:");
foreach (var path in Patterns.PathsWithSum(pathTree, 22))
{
    Console.WriteLine("  " + string.Join(" -> ", path) + " = 22");
}

Console.WriteLine("Усі шляхи корінь->лист:");
foreach (var path in Patterns.PathsWithSum(pathTree, target: null))
{
    Console.WriteLine($"  {string.Join(" -> ", path)} (сума {path.Sum()})");
}

public sealed class TreeNode<T>(T value, TreeNode<T>? left = null, TreeNode<T>? right = null)
{
    public T Value { get; set; } = value;
    public TreeNode<T>? Left { get; set; } = left;
    public TreeNode<T>? Right { get; set; } = right;
}

public static class Patterns
{
    // Порівнюємо ПОСИЛАННЯ на вузли, а не значення (значення можуть повторюватися).
    public static TreeNode<T>? Lca<T>(TreeNode<T>? node, TreeNode<T> p, TreeNode<T> q)
    {
        if (node is null || node == p || node == q)
        {
            return node;
        }

        var left = Lca(node.Left, p, q);
        var right = Lca(node.Right, p, q);

        // p і q знайдені в різних піддеревах — поточний вузол є точкою розгалуження.
        if (left is not null && right is not null)
        {
            return node;
        }

        return left ?? right;
    }

    public static bool IsSame(TreeNode<int>? a, TreeNode<int>? b)
    {
        if (a is null || b is null)
        {
            return a is null && b is null; // true лише якщо обидва порожні
        }

        return a.Value == b.Value && IsSame(a.Left, b.Left) && IsSame(a.Right, b.Right);
    }

    public static bool IsSymmetric(TreeNode<int>? root) =>
        root is null || IsMirrorPair(root.Left, root.Right);

    // Дві гілки дзеркальні, якщо зовнішні і внутрішні пари теж дзеркальні.
    private static bool IsMirrorPair(TreeNode<int>? a, TreeNode<int>? b)
    {
        if (a is null || b is null)
        {
            return a is null && b is null;
        }

        return a.Value == b.Value
            && IsMirrorPair(a.Left, b.Right)   // зовнішня пара
            && IsMirrorPair(a.Right, b.Left);  // внутрішня пара
    }

    public static void Mirror<T>(TreeNode<T>? node)
    {
        if (node is null)
        {
            return;
        }

        // Обмін кортежем — без тимчасової змінної.
        (node.Left, node.Right) = (node.Right, node.Left);
        Mirror(node.Left);
        Mirror(node.Right);
    }

    public static TreeNode<T>? Clone<T>(TreeNode<T>? node) =>
        node is null ? null : new TreeNode<T>(node.Value, Clone(node.Left), Clone(node.Right));

    public static IEnumerable<T> Preorder<T>(TreeNode<T>? node)
    {
        var stack = new Stack<TreeNode<T>>();
        if (node is not null)
        {
            stack.Push(node);
        }

        while (stack.Count > 0)
        {
            var current = stack.Pop();
            yield return current.Value;
            if (current.Right is not null) stack.Push(current.Right);
            if (current.Left is not null) stack.Push(current.Left);
        }
    }

    // Сума передається згори вниз як «залишок».
    public static bool HasPathSum(TreeNode<int>? node, int remaining)
    {
        if (node is null)
        {
            return false;
        }

        remaining -= node.Value;
        if (node.Left is null && node.Right is null)
        {
            return remaining == 0; // перевіряємо лише у ЛИСТІ
        }

        return HasPathSum(node.Left, remaining) || HasPathSum(node.Right, remaining);
    }

    // target == null — повернути всі шляхи корінь->лист.
    public static List<List<int>> PathsWithSum(TreeNode<int> root, int? target)
    {
        var result = new List<List<int>>();
        var path = new List<int>();
        Walk(root, 0);
        return result;

        void Walk(TreeNode<int>? node, int sum)
        {
            if (node is null)
            {
                return;
            }

            path.Add(node.Value);   // «зробити крок»
            sum += node.Value;

            if (node.Left is null && node.Right is null)
            {
                if (target is null || sum == target)
                {
                    result.Add([.. path]); // КОПІЯ, бо path ще змінюватиметься
                }
            }
            else
            {
                Walk(node.Left, sum);
                Walk(node.Right, sum);
            }

            path.RemoveAt(path.Count - 1); // backtracking: «повернутися назад»
        }
    }
}
```

**Приклад запуску:**

```text
LCA(4, 7) = 2
LCA(7, 6) = 1
LCA(5, 7) = 5

Симетричне 1(2(3,4),2(4,3))   : True
Симетричне 1(2(,3),2(,3))     : False
Копія однакова з оригіналом   : True
Після Mirror однакова         : False
Preorder дзеркала             : 1 3 6 2 5 7 4
Дзеркало двічі == оригінал    : True

Чи є шлях корінь->лист із сумою 22: True
Усі такі шляхи:
  5 -> 4 -> 11 -> 2 = 22
  5 -> 8 -> 4 -> 5 = 22
Усі шляхи корінь->лист:
  5 -> 4 -> 11 -> 7 (сума 27)
  5 -> 4 -> 11 -> 2 (сума 22)
  5 -> 8 -> 13 (сума 26)
  5 -> 8 -> 4 -> 5 (сума 22)
  5 -> 8 -> 4 -> 1 (сума 18)
```

### Типові помилки

- **Діаметр через корінь:** `Height(root.Left) + Height(root.Right) + 2` — неправильно для «гачка», де найдовший шлях лежить цілком в одному піддереві.
- **Max path sum з `best = 0`:** для дерева з одних від'ємних чисел відповідь має бути найбільше від'ємне число, а не 0.
- **Path sum перевіряють у `null`**, а не в листі: для дерева `1(2, null)` і суми `1` умова `remaining == 0` спрацює на порожньому правому сині, хоча `1` — не лист.
- **Забути backtracking** (`path.RemoveAt`) або додати в результат сам `path`, а не копію — усі шляхи в результаті стануть однаковими/порожніми.
- **LCA за значенням** замість посилання — ламається при дублікатах.

### Міні-вправа 7.1

Порахуйте кількість **«добрих» вузлів**: вузол добрий, якщо на шляху від кореня до нього немає значення, більшого за його власне.

<details>
<summary>Розв'язок</summary>

Передаємо максимум на шляху **згори вниз**:

```csharp
// Файл: GoodNodes.cs
//        3
//       / \
//      1   4
//     /   / \
//    3   1   5          добрі: 3 (корінь), 3 (ліворуч унизу), 4, 5
var root = new Node(3, new Node(1, new Node(3)), new Node(4, new Node(1), new Node(5)));
Console.WriteLine($"Добрих вузлів: {CountGood(root, int.MinValue)}");

static int CountGood(Node? node, int maxSoFar)
{
    if (node is null)
    {
        return 0;
    }

    var good = node.Value >= maxSoFar ? 1 : 0;
    var newMax = Math.Max(maxSoFar, node.Value);
    return good + CountGood(node.Left, newMax) + CountGood(node.Right, newMax);
}

public sealed record Node(int Value, Node? Left = null, Node? Right = null);
```

```text
Добрих вузлів: 4
```

</details>

### Міні-вправа 7.2

Знайдіть LCA двох **значень** у **BST** за `O(h)` без обходу всього дерева.

<details>
<summary>Розв'язок</summary>

У BST якщо обидва значення менші за вузол — LCA ліворуч, якщо обидва більші — праворуч, інакше поточний вузол є точкою розгалуження.

```csharp
// Файл: BstLca.cs
//          6
//        /   \
//       2     8
//      / \   / \
//     0   4 7   9
//        / \
//       3   5
var root = new Node(6,
    new Node(2, new Node(0), new Node(4, new Node(3), new Node(5))),
    new Node(8, new Node(7), new Node(9)));

Console.WriteLine($"LCA(2, 8) = {Lca(root, 2, 8)}");
Console.WriteLine($"LCA(2, 4) = {Lca(root, 2, 4)}");
Console.WriteLine($"LCA(3, 5) = {Lca(root, 3, 5)}");

static int Lca(Node root, int p, int q)
{
    var node = root;
    while (true)
    {
        if (p < node.Value && q < node.Value && node.Left is not null)
        {
            node = node.Left;
        }
        else if (p > node.Value && q > node.Value && node.Right is not null)
        {
            node = node.Right;
        }
        else
        {
            return node.Value; // значення «розходяться» тут
        }
    }
}

public sealed record Node(int Value, Node? Left = null, Node? Right = null);
```

```text
LCA(2, 8) = 6
LCA(2, 4) = 2
LCA(3, 5) = 4
```

</details>

---

## 8. Побудова, серіалізація та друк дерева

*≈ 13 хв*

### 8.1 Побудова з preorder + inorder

- `preorder[0]` — корінь.
- Позиція кореня в inorder ділить inorder на ліве й праве піддерева; розмір лівого = `k`.
- Наступні `k` елементів preorder — ліве піддерево, решта — праве.

```
preorder = [3, 9, 20, 15, 7]        inorder = [9, 3, 15, 20, 7]
            ▲  └┘  └────────┘                  └┘ ▲  └────────┘
          корінь ліве  праве                 ліве корінь праве

             3
            / \
           9   20
              /  \
             15   7
```

Щоб не шукати корінь в inorder лінійно (що дає `O(n²)`), заздалегідь будуємо `Dictionary<значення, індекс>` → `O(n)`. Умова: значення **унікальні**.

> ❓ Чи можна відновити дерево з preorder + postorder? **Не завжди однозначно**: `1(2, null)` і `1(null, 2)` мають однакові preorder `1 2` і postorder `2 1`. Inorder обов'язковий (або дерево має бути повним).

### 8.2 Серіалізація в рядок

Формат рівнями з `#` для відсутніх дітей (як на LeetCode), хвостові `#` відкидаємо:

```
             1
            / \
           2   3            "1,2,3,#,#,4,5"
              / \
             4   5
```

Відмінність від масиву купи (розділ 3): тут для `null` **не резервуються** місця під дітей, тож формат компактний навіть для виродженого дерева.

### 8.3 Друк дерева в консолі

Зручний «боковий» друк: праве піддерево — вгорі, ліве — внизу (поверніть голову ліворуч, щоб побачити звичний вигляд).

### 8.4 Код

```csharp
// Файл: BuildSerializePrint.cs
// Побудова з preorder+inorder, серіалізація/десеріалізація, друк у консолі.

int[] preorder = [3, 9, 20, 15, 7];
int[] inorder = [9, 3, 15, 20, 7];

var built = TreeBuilder.FromPreorderInorder(preorder, inorder);
Console.WriteLine("== Побудовано з preorder + inorder ==");
TreePrinter.Print(built);

var text = TreeCodec.Serialize(built);
Console.WriteLine($"Серіалізація: \"{text}\"");

var restored = TreeCodec.Deserialize(text);
Console.WriteLine($"Повторна серіалізація: \"{TreeCodec.Serialize(restored)}\"");

Console.WriteLine();
Console.WriteLine("== Десеріалізація \"1,2,3,#,4,5,6,7\" ==");
var other = TreeCodec.Deserialize("1,2,3,#,4,5,6,7");
TreePrinter.Print(other);
Console.WriteLine($"Серіалізація назад: \"{TreeCodec.Serialize(other)}\"");

Console.WriteLine();
Console.WriteLine($"Порожнє дерево: \"{TreeCodec.Serialize(null)}\", назад -> {(TreeCodec.Deserialize("") is null ? "null" : "не null")}");

public sealed class TreeNode(int value, TreeNode? left = null, TreeNode? right = null)
{
    public int Value { get; set; } = value;
    public TreeNode? Left { get; set; } = left;
    public TreeNode? Right { get; set; } = right;
}

public static class TreeBuilder
{
    public static TreeNode? FromPreorderInorder(int[] preorder, int[] inorder)
    {
        // Значення -> позиція в inorder, щоб знаходити корінь за O(1).
        var indexOf = new Dictionary<int, int>();
        for (var i = 0; i < inorder.Length; i++)
        {
            indexOf[inorder[i]] = i;
        }

        var preIndex = 0; // «курсор» по preorder, спільний для всіх викликів
        return Build(0, inorder.Length - 1);

        // Будує піддерево з елементів inorder[from..to].
        TreeNode? Build(int from, int to)
        {
            if (from > to)
            {
                return null;
            }

            var rootValue = preorder[preIndex++];
            var mid = indexOf[rootValue];

            // Порядок важливий: preorder = корінь, ПОТІМ ліве, ПОТІМ праве.
            var left = Build(from, mid - 1);
            var right = Build(mid + 1, to);
            return new TreeNode(rootValue, left, right);
        }
    }
}

public static class TreeCodec
{
    private const string NullMark = "#";

    public static string Serialize(TreeNode? root)
    {
        var parts = new List<string>();
        var queue = new Queue<TreeNode?>();
        queue.Enqueue(root);

        while (queue.Count > 0)
        {
            var node = queue.Dequeue();
            if (node is null)
            {
                parts.Add(NullMark);
                continue; // дітей у null немає — нічого не додаємо
            }

            parts.Add(node.Value.ToString());
            queue.Enqueue(node.Left);
            queue.Enqueue(node.Right);
        }

        // Хвостові "#" не несуть інформації.
        while (parts.Count > 0 && parts[^1] == NullMark)
        {
            parts.RemoveAt(parts.Count - 1);
        }

        return string.Join(",", parts);
    }

    public static TreeNode? Deserialize(string data)
    {
        if (string.IsNullOrWhiteSpace(data))
        {
            return null;
        }

        var tokens = data.Split(',');
        var root = new TreeNode(int.Parse(tokens[0]));
        var queue = new Queue<TreeNode>();
        queue.Enqueue(root);
        var index = 1;

        // Кожен вузол із черги «забирає» два наступні токени як своїх дітей.
        while (queue.Count > 0 && index < tokens.Length)
        {
            var parent = queue.Dequeue();

            if (index < tokens.Length && tokens[index] != NullMark)
            {
                parent.Left = new TreeNode(int.Parse(tokens[index]));
                queue.Enqueue(parent.Left);
            }

            index++;

            if (index < tokens.Length && tokens[index] != NullMark)
            {
                parent.Right = new TreeNode(int.Parse(tokens[index]));
                queue.Enqueue(parent.Right);
            }

            index++;
        }

        return root;
    }
}

public static class TreePrinter
{
    // Боковий друк: праве піддерево вгорі, ліве внизу.
    public static void Print(TreeNode? root)
    {
        if (root is null)
        {
            Console.WriteLine("(порожнє)");
            return;
        }

        PrintSubtree(root.Right, "", isRight: true);
        Console.WriteLine(root.Value);
        PrintSubtree(root.Left, "", isRight: false);
    }

    private static void PrintSubtree(TreeNode? node, string indent, bool isRight)
    {
        if (node is null)
        {
            return;
        }

        // Для правого сина вертикальна лінія потрібна під ним (до батька), для лівого — над ним.
        PrintSubtree(node.Right, indent + (isRight ? "    " : "│   "), isRight: true);
        Console.WriteLine(indent + (isRight ? "┌── " : "└── ") + node.Value);
        PrintSubtree(node.Left, indent + (isRight ? "│   " : "    "), isRight: false);
    }
}
```

**Приклад запуску:**

```text
== Побудовано з preorder + inorder ==
    ┌── 7
┌── 20
│   └── 15
3
└── 9
Серіалізація: "3,9,20,#,#,15,7"
Повторна серіалізація: "3,9,20,#,#,15,7"

== Десеріалізація "1,2,3,#,4,5,6,7" ==
    ┌── 6
┌── 3
│   └── 5
1
│   ┌── 4
│   │   └── 7
└── 2
Серіалізація назад: "1,2,3,#,4,5,6,7"

Порожнє дерево: "", назад -> null
```

### Типові помилки

- При побудові з preorder+inorder будувати **праве піддерево раніше лівого** — курсор `preIndex` зміститься неправильно.
- Лінійний пошук кореня в inorder → `O(n²)` на виродженому дереві.
- Серіалізувати без маркерів `null` лише один обхід (наприклад, тільки preorder) — дерево неможливо однозначно відновити.
- У десеріалізації не перевіряти `index < tokens.Length` для правого сина — `IndexOutOfRangeException`, коли хвостові `#` відкинуто.

### Міні-вправа 8.1

Відновіть дерево з **inorder + postorder**: `inorder = [9, 3, 15, 20, 7]`, `postorder = [9, 15, 7, 20, 3]`.

<details>
<summary>Розв'язок</summary>

Корінь — **останній** елемент postorder; курсор рухається справа наліво, тому будуємо **спочатку праве**, потім ліве піддерево.

```csharp
// Файл: BuildInPost.cs
int[] inorder = [9, 3, 15, 20, 7];
int[] postorder = [9, 15, 7, 20, 3];

var indexOf = inorder.Select((value, index) => (value, index)).ToDictionary(p => p.value, p => p.index);
var postIndex = postorder.Length - 1;
var root = Build(0, inorder.Length - 1);

Console.WriteLine("Preorder відновленого дерева: " + string.Join(" ", Preorder(root)));

Node? Build(int from, int to)
{
    if (from > to)
    {
        return null;
    }

    var value = postorder[postIndex--];
    var mid = indexOf[value];
    var right = Build(mid + 1, to); // спочатку праве!
    var left = Build(from, mid - 1);
    return new Node(value, left, right);
}

static IEnumerable<int> Preorder(Node? node) =>
    node is null ? [] : [node.Value, .. Preorder(node.Left), .. Preorder(node.Right)];

public sealed record Node(int Value, Node? Left = null, Node? Right = null);
```

```text
Preorder відновленого дерева: 3 9 20 15 7
```

</details>

---

> ## ☕ Перерва 2 (≈ 10 хв)
>
> Пройдено ≈ 98 хв: BFS, рекурсивні патерни, побудова та серіалізація. Далі — дерева пошуку.

---

## 9. Бінарне дерево пошуку (BST): базові операції

*≈ 20 хв*

### 9.1 Інваріант BST

**Бінарне дерево пошуку** — бінарне дерево, у якому для **кожного** вузла `x`:

```
  усі ключі в лівому піддереві  <  x.Key  <  усі ключі в правому піддереві
```

Наслідок: **inorder-обхід BST видає ключі у відсортованому порядку.**

```
                 8
               /   \
              3     10
             / \      \
            1   6      14
               / \     /
              4   7   13

Inorder: 1 3 4 6 7 8 10 13 14   (відсортовано)
```

> ⚠️ Інваріант стосується **всього піддерева**, а не лише безпосередніх дітей! Дерево нижче **не** є BST, хоча кожен вузол більший за лівого сина й менший за правого:
>
> ```
>         8
>        / \
>       3   10
>        \
>         9      ← 9 > 8, але лежить у лівому піддереві 8
> ```

**Дублікати.** Класичний BST зберігає унікальні ключі. Варіанти для дублікатів: лічильник у вузлі; правило «рівні — праворуч»; або (як `SortedSet<T>`) просто ігнорувати повторну вставку.

### 9.2 Пошук і вставка

Пошук: порівнюємо з поточним вузлом і йдемо **лише в одну** гілку — `O(h)`.

```
Пошук 7:   8 → (7<8) ліворуч → 3 → (7>3) праворуч → 6 → (7>6) праворуч → 7 ✓
Пошук 5:   8 → 3 → 6 → 4 → (5>4) праворуч → null ✗
```

Вставка: шукаємо ключ; місце, де пошук «впав» у `null`, і є позицією нового листа.

```
Вставка 5:
                 8                              8
               /   \                          /   \
              3     10                       3     10
             / \      \       ──►           / \      \
            1   6      14                  1   6      14
               / \     /                      / \     /
              4   7   13                     4   7   13
                                              \
                                               5   ← новий лист
```

### 9.3 Видалення — три випадки

**Випадок 1: лист.** Просто прибираємо посилання з батька.

```
Видалити 4 (лист):
        8                     8
       / \                   / \
      3   10                3   10
     / \    \      ──►     / \    \
    1   6    14           1   6    14
       / \   /                 \   /
      4   7 13                  7 13
```

`6.Left = null` — і все.

**Випадок 2: одна дитина.** Батько «перечіплюється» на єдину дитину вузла.

```
Видалити 10 (має лише правого сина 14):
        8                    8
       / \                  / \
      3   10      ──►      3   14
            \                  /
             14               13
            /
           13
```

**Випадок 3: дві дитини.** Знаходимо **inorder-наступника** (мінімум правого піддерева; у нього немає лівого сина), копіюємо його ключ у вузол і видаляємо наступника (для нього це випадок 1 або 2). Симетрично можна брати **попередника** — максимум лівого піддерева.

```
Видалити 3 (дві дитини):
        8                     8                      8
       / \                   / \                    / \
      3   10                4   10                 4   10
     / \    \      ──►     / \    \      ──►      / \    \
    1   6    14           1   6    14            1   6    14
       / \               (крок 1: 4 — мін.          \
      4   7               правого піддерева,         7
                          копіюємо ключ)       (крок 2: видаляємо старий 4)
```

Чому це коректно: наступник `s` більший за все ліве піддерево (бо він з правого) і менший за решту правого піддерева (бо він там мінімум) — отже на місці видаленого вузла інваріант зберігається.

### 9.4 Код: узагальнений `BinarySearchTree<T>`

```csharp
// Файл: BstBasics.cs
// Узагальнене BST: пошук, вставка, видалення (3 випадки), мінімум/максимум.

var tree = new BinarySearchTree<int>();
foreach (var key in new[] { 8, 3, 10, 1, 6, 14, 4, 7, 13 })
{
    tree.Add(key);
}

Console.WriteLine($"Inorder     : {string.Join(" ", tree.InOrder())}");
Console.WriteLine($"Count={tree.Count}, Min={tree.Min()}, Max={tree.Max()}, Height={tree.Height}");
Console.WriteLine($"Contains(7)={tree.Contains(7)}, Contains(5)={tree.Contains(5)}");
Console.WriteLine($"Add(6) повторно -> {tree.Add(6)} (дублікати ігноруються)");

Console.WriteLine();
Console.WriteLine($"Remove(4)  [лист]         -> {tree.Remove(4)}: {string.Join(" ", tree.InOrder())}");
Console.WriteLine($"Remove(10) [одна дитина]  -> {tree.Remove(10)}: {string.Join(" ", tree.InOrder())}");
Console.WriteLine($"Remove(3)  [дві дитини]   -> {tree.Remove(3)}: {string.Join(" ", tree.InOrder())}");
Console.WriteLine($"Remove(8)  [корінь]       -> {tree.Remove(8)}: {string.Join(" ", tree.InOrder())}");
Console.WriteLine($"Remove(42) [немає]        -> {tree.Remove(42)}");
Console.WriteLine($"Корінь тепер: {tree.RootKey}, Count={tree.Count}");

// Узагальненість: рядки порівнюються через IComparable<string> (ординально тут).
var words = new BinarySearchTree<string>(StringComparer.Ordinal);
foreach (var word in "груша яблуко вишня абрикос слива".Split(' '))
{
    words.Add(word);
}

Console.WriteLine();
Console.WriteLine($"Слова за алфавітом: {string.Join(", ", words.InOrder())}");

/// <summary>Незбалансоване бінарне дерево пошуку з унікальними ключами.</summary>
public sealed class BinarySearchTree<T>(IComparer<T>? comparer = null)
{
    private sealed class Node(T key)
    {
        public T Key { get; set; } = key;
        public Node? Left { get; set; }
        public Node? Right { get; set; }
    }

    // Comparer<T>.Default використовує IComparable<T>; можна передати власний.
    private readonly IComparer<T> _comparer = comparer ?? Comparer<T>.Default;
    private Node? _root;

    public int Count { get; private set; }

    public T RootKey => _root is null ? throw new InvalidOperationException("Дерево порожнє") : _root.Key;

    public int Height => HeightOf(_root);

    // Ітеративний пошук: O(h) часу, O(1) пам'яті.
    public bool Contains(T key)
    {
        var node = _root;
        while (node is not null)
        {
            var cmp = _comparer.Compare(key, node.Key);
            if (cmp == 0)
            {
                return true;
            }

            node = cmp < 0 ? node.Left : node.Right;
        }

        return false;
    }

    // Ітеративна вставка: запам'ятовуємо батька, щоб причепити новий лист.
    public bool Add(T key)
    {
        if (_root is null)
        {
            _root = new Node(key);
            Count = 1;
            return true;
        }

        var current = _root;
        while (true)
        {
            var cmp = _comparer.Compare(key, current.Key);
            if (cmp == 0)
            {
                return false; // такий ключ уже є
            }

            if (cmp < 0)
            {
                if (current.Left is null)
                {
                    current.Left = new Node(key);
                    break;
                }

                current = current.Left;
            }
            else
            {
                if (current.Right is null)
                {
                    current.Right = new Node(key);
                    break;
                }

                current = current.Right;
            }
        }

        Count++;
        return true;
    }

    public bool Remove(T key)
    {
        var removed = false;
        _root = RemoveFrom(_root, key, ref removed);
        if (removed)
        {
            Count--;
        }

        return removed;
    }

    // Рекурсивне видалення повертає НОВИЙ корінь піддерева — батьку лишається лише присвоїти.
    private Node? RemoveFrom(Node? node, T key, ref bool removed)
    {
        if (node is null)
        {
            return null; // ключ не знайдено
        }

        var cmp = _comparer.Compare(key, node.Key);
        if (cmp < 0)
        {
            node.Left = RemoveFrom(node.Left, key, ref removed);
            return node;
        }

        if (cmp > 0)
        {
            node.Right = RemoveFrom(node.Right, key, ref removed);
            return node;
        }

        removed = true;

        // Випадки 1 і 2: немає лівого (або обох) — піднімаємо правого; немає правого — лівого.
        if (node.Left is null)
        {
            return node.Right;
        }

        if (node.Right is null)
        {
            return node.Left;
        }

        // Випадок 3: дві дитини. Шукаємо мінімум правого піддерева.
        var successor = node.Right;
        while (successor.Left is not null)
        {
            successor = successor.Left;
        }

        node.Key = successor.Key;

        // Видаляємо наступника з правого піддерева; у нього немає лівого сина → випадок 1/2.
        var ignored = false;
        node.Right = RemoveFrom(node.Right, successor.Key, ref ignored);
        return node;
    }

    public T Min()
    {
        var node = _root ?? throw new InvalidOperationException("Дерево порожнє");
        while (node.Left is not null)
        {
            node = node.Left; // мінімум — найлівіший вузол
        }

        return node.Key;
    }

    public T Max()
    {
        var node = _root ?? throw new InvalidOperationException("Дерево порожнє");
        while (node.Right is not null)
        {
            node = node.Right; // максимум — найправіший вузол
        }

        return node.Key;
    }

    public IEnumerable<T> InOrder()
    {
        var stack = new Stack<Node>();
        var current = _root;
        while (current is not null || stack.Count > 0)
        {
            while (current is not null)
            {
                stack.Push(current);
                current = current.Left;
            }

            current = stack.Pop();
            yield return current.Key;
            current = current.Right;
        }
    }

    private static int HeightOf(Node? node) =>
        node is null ? -1 : 1 + Math.Max(HeightOf(node.Left), HeightOf(node.Right));
}
```

**Приклад запуску:**

```text
Inorder     : 1 3 4 6 7 8 10 13 14
Count=9, Min=1, Max=14, Height=3
Contains(7)=True, Contains(5)=False
Add(6) повторно -> False (дублікати ігноруються)

Remove(4)  [лист]         -> True: 1 3 6 7 8 10 13 14
Remove(10) [одна дитина]  -> True: 1 3 6 7 8 13 14
Remove(3)  [дві дитини]   -> True: 1 6 7 8 13 14
Remove(8)  [корінь]       -> True: 1 6 7 13 14
Remove(42) [немає]        -> False
Корінь тепер: 13, Count=5

Слова за алфавітом: абрикос, вишня, груша, слива, яблуко
```

### 9.5 Складність

| Операція | Середній випадок (випадкові вставки) | Найгірший (вироджене) |
|----------|-------------------------------------|-----------------------|
| Пошук | O(log n) | O(n) |
| Вставка | O(log n) | O(n) |
| Видалення | O(log n) | O(n) |
| Min / Max | O(log n) | O(n) |
| Inorder-обхід | O(n) | O(n) |

Для BST, побудованого з випадкової перестановки, **середня глибина** вузла ≈ `1.39·log₂ n`, а очікувана висота ≈ `4.311·ln n ≈ 2.99·log₂ n`. Головне: `O(log n)` лише **в середньому**; на відсортованих даних — `O(n)` (розділ 11).

### Типові помилки

- **Видалення з двома дітьми:** скопіювати ключ наступника, але забути видалити сам вузол-наступник → дублікат у дереві.
- **Шукати наступника в лівому піддереві** (там попередник!) і при цьому брати мінімум, а не максимум.
- **Рекурсивне видалення без присвоєння результату**: `RemoveFrom(node.Left, key)` замість `node.Left = RemoveFrom(...)` — вузол не від'єднується.
- **Порівняння рядків** за замовчуванням культурно-залежне (`Comparer<string>.Default`) — на різних машинах порядок може відрізнятися. Для детермінізму передавайте `StringComparer.Ordinal`.
- **Не оновлювати `Count`** при невдалій вставці/видаленні.

### Міні-вправа 9.1

Вставте в порожнє BST ключі `50, 30, 70, 20, 40, 60, 80`, потім видаліть `50`. Який ключ стане коренем і як виглядатиме дерево?

<details>
<summary>Розв'язок</summary>

Наступник `50` — мінімум правого піддерева, тобто `60`. Ключ `60` копіюється в корінь, лист `60` видаляється:

```
          50                         60
        /    \                     /    \
      30      70       ──►       30      70
     /  \    /  \               /  \       \
    20  40  60  80             20  40      80
```

</details>

---

## 10. BST: розширені операції

*≈ 18 хв*

### 10.1 Наступник і попередник

**Inorder-наступник** (successor) ключа `x` — найменший ключ, строго більший за `x`.

- Якщо у вузла є праве піддерево — наступник = мінімум правого піддерева.
- Інакше — найнижчий предок, для якого наш вузол лежить у **лівому** піддереві.

Без посилань на батька зручно йти від кореня: кожного разу, коли повертаємо **ліворуч**, поточний вузол — кандидат у наступники.

```
                 8
               /   \
              3     10
             / \      \
            1   6      14
               / \     /
              4   7   13

successor(7)  = 8    (праворуч нічого; останній поворот ліворуч був у 8)
successor(6)  = 7    (мінімум правого піддерева)
successor(14) = немає
predecessor(8) = 7,  predecessor(10) = 8
```

### 10.2 Floor і ceiling

- **floor(x)** — найбільший ключ `≤ x`;
- **ceiling(x)** — найменший ключ `≥ x`.

Відрізняються від predecessor/successor тим, що `x` сам може бути в дереві й тоді це і є відповідь.

```
floor(5) = 4,  ceiling(5) = 6,  floor(0) = немає,  ceiling(15) = немає,  floor(13) = 13
```

### 10.3 Перевірка, чи дерево є BST (через межі)

Кожен вузол має лежати в інтервалі `(low, high)`, що **успадковується від предків**:

```
               8 (−∞, +∞)
             /            \
       3 (−∞, 8)        10 (8, +∞)
            \
          9 (3, 8)   ✗  9 не менше за 8
```

Альтернатива: inorder має бути **строго зростаючим** — порівнюємо з попереднім значенням.

### 10.4 k-те найменше, запит діапазону

- **k-те найменше:** ітеративний inorder із зупинкою на k-му кроці. `O(h + k)`. (Якщо часто — зберігайте в кожному вузлі розмір піддерева → `O(h)`; це **order-statistic tree**.)
- **Діапазон `[lo, hi]`:** inorder з відсіканням: ліворуч спускаємося лише якщо `lo < node.Key`, праворуч — лише якщо `node.Key < hi`. Складність `O(h + m)`, де `m` — кількість знайдених ключів.

### 10.5 Збалансоване BST з відсортованого масиву

Беремо середній елемент як корінь, рекурсивно — ліву половину як ліве піддерево, праву — як праве. Висота `⌊log₂ n⌋`, час `O(n)`.

```
[1, 2, 3, 4, 5, 6, 7]  →          4
                                /   \
                               2     6
                              / \   / \
                             1   3 5   7
```

### 10.6 Ітератор BST (inorder зі стеком)

Ітератор тримає в стеку «лівий контур» ще не відвіданих вузлів. `MoveNext` — амортизовано `O(1)`, пам'ять — `O(h)`. Так влаштований перелічувач `SortedSet<T>` у .NET.

### 10.7 Код

```csharp
// Файл: BstAdvanced.cs
// Successor/predecessor, floor/ceiling, валідація, k-те, діапазон, побудова з масиву, ітератор.

var root = Bst.FromKeys(8, 3, 10, 1, 6, 14, 4, 7, 13);
Console.WriteLine("Дерево (inorder): " + string.Join(" ", new BstIterator(root)));

Console.WriteLine();
foreach (var x in new[] { 7, 6, 8, 14, 1, 5 })
{
    Console.WriteLine($"x={x,2}: successor={Show(Bst.Successor(root, x)),-5} predecessor={Show(Bst.Predecessor(root, x)),-5} " +
                      $"floor={Show(Bst.Floor(root, x)),-5} ceiling={Show(Bst.Ceiling(root, x))}");
}

Console.WriteLine();
Console.WriteLine($"k-те найменше: k=1 -> {Show(Bst.KthSmallest(root, 1))}, k=5 -> {Show(Bst.KthSmallest(root, 5))}, k=10 -> {Show(Bst.KthSmallest(root, 10))}");
Console.WriteLine($"Діапазон [4, 10]: {string.Join(" ", Bst.Range(root, 4, 10))}");

Console.WriteLine();
Console.WriteLine($"Валідне BST?                      {Bst.IsValid(root)}");
var broken = new Node(8, new Node(3, null, new Node(9)), new Node(10));
Console.WriteLine($"8(3(,9),10) — BST? (межі)         {Bst.IsValid(broken)}");
Console.WriteLine($"8(3(,9),10) — лише діти перевірити: {Bst.IsValidNaive(broken)}  ← хибний висновок");
var extremes = new Node(int.MinValue, null, new Node(int.MaxValue));
Console.WriteLine($"MinValue(,MaxValue) — BST?        {Bst.IsValid(extremes)}");

Console.WriteLine();
var balanced = Bst.FromSorted([1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15]);
Console.WriteLine($"З відсортованого [1..15]: корінь={balanced!.Value}, висота={Bst.Height(balanced)}, " +
                  $"рівні: {string.Join(" | ", Bst.Levels(balanced).Select(l => string.Join(" ", l)))}");

// Ручне використання ітератора в стилі Java: HasNext/Next.
var iterator = new BstIterator(root);
Console.Write("Перші три через HasNext/Next: ");
for (var i = 0; i < 3 && iterator.HasNext; i++)
{
    Console.Write(iterator.Next() + " ");
}

Console.WriteLine();

static string Show(int? value) => value?.ToString() ?? "—";

public sealed class Node(int value, Node? left = null, Node? right = null)
{
    public int Value { get; } = value;
    public Node? Left { get; set; } = left;
    public Node? Right { get; set; } = right;
}

public static class Bst
{
    public static Node? FromKeys(params int[] keys)
    {
        Node? root = null;
        foreach (var key in keys)
        {
            root = Insert(root, key);
        }

        return root;
    }

    private static Node Insert(Node? node, int key)
    {
        if (node is null)
        {
            return new Node(key);
        }

        if (key < node.Value)
        {
            node.Left = Insert(node.Left, key);
        }
        else if (key > node.Value)
        {
            node.Right = Insert(node.Right, key);
        }

        return node;
    }

    // Найменший ключ > x. Кожен поворот ліворуч дає кандидата.
    public static int? Successor(Node? node, int x)
    {
        int? candidate = null;
        while (node is not null)
        {
            if (x < node.Value)
            {
                candidate = node.Value;
                node = node.Left;
            }
            else
            {
                node = node.Right; // node.Value <= x — не підходить, шукаємо більші
            }
        }

        return candidate;
    }

    // Найбільший ключ < x. Кожен поворот праворуч дає кандидата.
    public static int? Predecessor(Node? node, int x)
    {
        int? candidate = null;
        while (node is not null)
        {
            if (x > node.Value)
            {
                candidate = node.Value;
                node = node.Right;
            }
            else
            {
                node = node.Left;
            }
        }

        return candidate;
    }

    // Найбільший ключ <= x.
    public static int? Floor(Node? node, int x)
    {
        int? candidate = null;
        while (node is not null)
        {
            if (node.Value == x)
            {
                return x;
            }

            if (node.Value < x)
            {
                candidate = node.Value;
                node = node.Right;
            }
            else
            {
                node = node.Left;
            }
        }

        return candidate;
    }

    // Найменший ключ >= x.
    public static int? Ceiling(Node? node, int x)
    {
        int? candidate = null;
        while (node is not null)
        {
            if (node.Value == x)
            {
                return x;
            }

            if (node.Value > x)
            {
                candidate = node.Value;
                node = node.Left;
            }
            else
            {
                node = node.Right;
            }
        }

        return candidate;
    }

    // Межі типу long, щоб коректно обробити вузли зі значеннями int.MinValue/int.MaxValue.
    public static bool IsValid(Node? root) => IsValid(root, long.MinValue, long.MaxValue);

    private static bool IsValid(Node? node, long low, long high)
    {
        if (node is null)
        {
            return true;
        }

        if (node.Value <= low || node.Value >= high)
        {
            return false;
        }

        // Ліворуч верхня межа стискається до node.Value, праворуч — нижня.
        return IsValid(node.Left, low, node.Value) && IsValid(node.Right, node.Value, high);
    }

    // НЕПРАВИЛЬНА версія — лише для демонстрації типової помилки.
    public static bool IsValidNaive(Node? node)
    {
        if (node is null)
        {
            return true;
        }

        if (node.Left is not null && node.Left.Value >= node.Value) return false;
        if (node.Right is not null && node.Right.Value <= node.Value) return false;
        return IsValidNaive(node.Left) && IsValidNaive(node.Right);
    }

    public static int? KthSmallest(Node? root, int k)
    {
        var stack = new Stack<Node>();
        var current = root;
        while (current is not null || stack.Count > 0)
        {
            while (current is not null)
            {
                stack.Push(current);
                current = current.Left;
            }

            current = stack.Pop();
            if (--k == 0)
            {
                return current.Value;
            }

            current = current.Right;
        }

        return null;
    }

    public static List<int> Range(Node? root, int low, int high)
    {
        var result = new List<int>();
        Collect(root);
        return result;

        void Collect(Node? node)
        {
            if (node is null)
            {
                return;
            }

            // Ліве піддерево може містити ключі >= low, лише якщо node.Value > low.
            if (low < node.Value)
            {
                Collect(node.Left);
            }

            if (low <= node.Value && node.Value <= high)
            {
                result.Add(node.Value);
            }

            if (node.Value < high)
            {
                Collect(node.Right);
            }
        }
    }

    public static Node? FromSorted(int[] sorted) => FromSorted(sorted, 0, sorted.Length - 1);

    private static Node? FromSorted(int[] sorted, int from, int to)
    {
        if (from > to)
        {
            return null;
        }

        var mid = from + (to - from) / 2; // без переповнення, на відміну від (from + to) / 2
        return new Node(sorted[mid], FromSorted(sorted, from, mid - 1), FromSorted(sorted, mid + 1, to));
    }

    public static int Height(Node? node) =>
        node is null ? -1 : 1 + Math.Max(Height(node.Left), Height(node.Right));

    public static List<List<int>> Levels(Node root)
    {
        var result = new List<List<int>>();
        var queue = new Queue<Node>([root]);
        while (queue.Count > 0)
        {
            var level = new List<int>();
            for (var size = queue.Count; size > 0; size--)
            {
                var node = queue.Dequeue();
                level.Add(node.Value);
                if (node.Left is not null) queue.Enqueue(node.Left);
                if (node.Right is not null) queue.Enqueue(node.Right);
            }

            result.Add(level);
        }

        return result;
    }
}

/// <summary>Лінивий inorder-ітератор: O(h) пам'яті, амортизовано O(1) на крок.</summary>
public sealed class BstIterator : IEnumerable<int>
{
    private readonly Node? _root;
    private readonly Stack<Node> _stack = new();

    public BstIterator(Node? root)
    {
        _root = root;
        PushLeftSpine(root);
    }

    public bool HasNext => _stack.Count > 0;

    public int Next()
    {
        if (_stack.Count == 0)
        {
            throw new InvalidOperationException("Елементів більше немає");
        }

        var node = _stack.Pop();

        // Наступні після node — ліве «ребро» його правого піддерева.
        PushLeftSpine(node.Right);
        return node.Value;
    }

    // Кожен вузол потрапляє в стек рівно раз → n push-ів на n викликів Next.
    private void PushLeftSpine(Node? node)
    {
        for (; node is not null; node = node.Left)
        {
            _stack.Push(node);
        }
    }

    // Для foreach створюємо новий незалежний ітератор.
    public IEnumerator<int> GetEnumerator()
    {
        var fresh = new BstIterator(_root);
        while (fresh.HasNext)
        {
            yield return fresh.Next();
        }
    }

    System.Collections.IEnumerator System.Collections.IEnumerable.GetEnumerator() => GetEnumerator();
}
```

**Приклад запуску:**

```text
Дерево (inorder): 1 3 4 6 7 8 10 13 14

x= 7: successor=8     predecessor=6     floor=7     ceiling=7
x= 6: successor=7     predecessor=4     floor=6     ceiling=6
x= 8: successor=10    predecessor=7     floor=8     ceiling=8
x=14: successor=—     predecessor=13    floor=14    ceiling=14
x= 1: successor=3     predecessor=—     floor=1     ceiling=1
x= 5: successor=6     predecessor=4     floor=4     ceiling=6

k-те найменше: k=1 -> 1, k=5 -> 7, k=10 -> —
Діапазон [4, 10]: 4 6 7 8 10

Валідне BST?                      True
8(3(,9),10) — BST? (межі)         False
8(3(,9),10) — лише діти перевірити: True  ← хибний висновок
MinValue(,MaxValue) — BST?        True

З відсортованого [1..15]: корінь=8, висота=3, рівні: 8 | 4 12 | 2 6 10 14 | 1 3 5 7 9 11 13 15
Перші три через HasNext/Next: 1 3 4 
```

### Типові помилки

- **Валідація з межами `int`**: `IsValid(root, int.MinValue, int.MaxValue)` відкидає правильне дерево, що містить `int.MinValue` або `int.MaxValue`. Використовуйте `long` або nullable-межі.
- **Нестроге порівняння** в валідації (`<` замість `<=`) пропускає дублікати, які в BST з унікальними ключами заборонені.
- **`(from + to) / 2`** переповнюється для великих індексів — пишіть `from + (to - from) / 2`.
- **Діапазонний запит без відсікання** — повний обхід `O(n)` замість `O(h + m)`.
- **Модифікація дерева під час ітерації** ітератором зі стеком — стек посилається на вже видалені вузли. `SortedSet<T>` у такому разі кидає `InvalidOperationException`.

### Міні-вправа 10.1

Два вузли BST **помилково поміняли місцями**. Як знайти їх за один inorder-обхід?

<details>
<summary>Розв'язок</summary>

В inorder шукаємо «спади» `prev > current`. Перший елемент першого спаду й другий елемент останнього спаду — шукані вузли (якщо спад один, це сусідні елементи). Міняємо значення назад.

```csharp
// Файл: RecoverBst.cs
//      3              правильне:    2
//     / \                          / \
//    1   4                        1   4
//       /                            /
//      2                            3
var root = new Node(3, new Node(1), new Node(4, new Node(2)));

Node? first = null, second = null, prev = null;
Inorder(root);
Console.WriteLine($"Переставлені: {first!.Value} і {second!.Value}");
(first.Value, second.Value) = (second.Value, first.Value);
Console.Write("Після виправлення inorder: ");
Print(root);
Console.WriteLine();

void Inorder(Node? node)
{
    if (node is null) return;
    Inorder(node.Left);
    if (prev is not null && prev.Value > node.Value)
    {
        first ??= prev;   // запам'ятовуємо лише на першому спаді
        second = node;    // оновлюємо на кожному спаді
    }

    prev = node;
    Inorder(node.Right);
}

static void Print(Node? node)
{
    if (node is null) return;
    Print(node.Left);
    Console.Write(node.Value + " ");
    Print(node.Right);
}

public sealed class Node(int value, Node? left = null, Node? right = null)
{
    public int Value { get; set; } = value;
    public Node? Left { get; } = left;
    public Node? Right { get; } = right;
}
```

```text
Переставлені: 3 і 2
Після виправлення inorder: 1 2 3 4 
```

</details>

---

## 11. Вироджений BST: чому потрібне балансування

*≈ 5 хв*

Форма BST повністю залежить від **порядку вставки**. Вставка `1, 2, 3, 4, 5` дає ланцюжок:

```
1
 \
  2
   \
    3            висота = n − 1,  пошук = O(n)
     \
      4
       \
        5
```

Реальні дані часто частково відсортовані: автоінкрементні ID, часові мітки, імена з уже відсортованого файлу. Перевіримо на практиці.

```csharp
// Файл: DegenerateBst.cs
// Висота BST для відсортованих і перемішаних ключів. Порівнюємо з log2(n).

const int n = 10_000;

var sortedKeys = Enumerable.Range(1, n).ToArray();

// Фіксоване зерно — однаковий результат при кожному запуску.
var shuffledKeys = sortedKeys.ToArray();
new Random(2025).Shuffle(shuffledKeys);

var sortedTree = new Bst();
var shuffledTree = new Bst();
foreach (var key in sortedKeys) sortedTree.Add(key);
foreach (var key in shuffledKeys) shuffledTree.Add(key);

Console.WriteLine($"n = {n}, log2(n) = {Math.Log2(n):F1}");
Console.WriteLine($"Відсортована вставка : висота = {sortedTree.Height()}, порівнянь для пошуку {n} = {sortedTree.Comparisons(n)}");
Console.WriteLine($"Перемішана вставка   : висота = {shuffledTree.Height()}, порівнянь для пошуку {n} = {shuffledTree.Comparisons(n)}");
Console.WriteLine($"Ідеально збалансоване: висота = {(int)Math.Floor(Math.Log2(n))}");

/// <summary>Мінімальне BST з ітеративними операціями (рекурсія на ланцюжку в 10 000 вузлів ризикована).</summary>
public sealed class Bst
{
    private sealed class Node(int key)
    {
        public int Key { get; } = key;
        public Node? Left { get; set; }
        public Node? Right { get; set; }
    }

    private Node? _root;

    public void Add(int key)
    {
        if (_root is null)
        {
            _root = new Node(key);
            return;
        }

        var node = _root;
        while (true)
        {
            if (key < node.Key)
            {
                if (node.Left is null) { node.Left = new Node(key); return; }
                node = node.Left;
            }
            else if (key > node.Key)
            {
                if (node.Right is null) { node.Right = new Node(key); return; }
                node = node.Right;
            }
            else
            {
                return;
            }
        }
    }

    public int Comparisons(int key)
    {
        var count = 0;
        for (var node = _root; node is not null; node = key < node.Key ? node.Left : node.Right)
        {
            count++;
            if (node.Key == key) break;
        }

        return count;
    }

    // Висота через BFS по рівнях — без рекурсії.
    public int Height()
    {
        if (_root is null) return -1;
        var queue = new Queue<Node>([_root]);
        var height = -1;
        while (queue.Count > 0)
        {
            height++;
            for (var size = queue.Count; size > 0; size--)
            {
                var node = queue.Dequeue();
                if (node.Left is not null) queue.Enqueue(node.Left);
                if (node.Right is not null) queue.Enqueue(node.Right);
            }
        }

        return height;
    }
}
```

**Приклад запуску:**

```text
n = 10000, log2(n) = 13.3
Відсортована вставка : висота = 9999, порівнянь для пошуку 10000 = 10000
Перемішана вставка   : висота = 29, порівнянь для пошуку 10000 = 4
Ідеально збалансоване: висота = 13
```

**Висновок:** на відсортованих даних BST деградує до списку — у сотні разів більше порівнянь. Потрібна структура, яка **гарантує** `O(log n)` незалежно від порядку вставки: **самобалансовані дерева** (AVL, червоно-чорні, B-дерева, treap, splay).

---

> ## ☕ Перерва 3 (≈ 10 хв)
>
> Пройдено ≈ 141 хв: BST повністю. Після перерви — AVL, червоно-чорні дерева та практичне використання в .NET.

---

## 12. AVL-дерева

*≈ 28 хв*

### 12.1 Ідея та фактор балансу

**AVL-дерево** (Адельсон-Вельський і Ландіс, 1962) — перше самобалансоване BST. Інваріант: для **кожного** вузла

```
balance(x) = height(x.Left) − height(x.Right)   ∈  { −1, 0, +1 }
```

У кожному вузлі зберігаємо його висоту (тут — **у вузлах**: `null` = 0, лист = 1), щоб фактор балансу обчислювався за `O(1)`.

```
            30 (h=3, bf=+1)
           /    \
   (h=2) 20      40 (h=1, bf=0)
        /
  (h=1) 10          ✓ AVL: скрізь |bf| ≤ 1

            30 (h=3, bf=+2)  ✗
           /
   (h=2) 20
        /
  (h=1) 10
```

Після звичайної BST-вставки чи видалення фактор балансу може стати `±2` лише у вузлах **на шляху** від зміненого місця до кореня. Ми піднімаємося цим шляхом (рекурсія робить це «безкоштовно» на поверненні), оновлюємо висоти й виправляємо порушення **поворотами**.

### 12.2 Повороти

Поворот — локальна перебудова трьох вузлів, яка **зберігає inorder-порядок** (а отже, BST-інваріант) і змінює висоти. Виконується за `O(1)`.

**Правий поворот навколо `y`** (rotate right):

```
          y                          x
         / \                        / \
        x   T3     ── right ──►    T1  y
       / \                            / \
      T1  T2                         T2  T3

  inorder до і після: T1 x T2 y T3
```

**Лівий поворот навколо `x`** — дзеркальний:

```
        x                              y
       / \                            / \
      T1  y        ── left ──►       x   T3
         / \                        / \
        T2  T3                     T1  T2
```

Зверніть увагу на піддерево `T2`: воно **переходить до іншого батька**. Це єдиний «хитрий» момент у коді.

### 12.3 Чотири випадки дисбалансу

Нехай `z` — найнижчий вузол з `|bf| = 2`.

**Випадок LL** (bf(z) = +2, bf(z.Left) ≥ 0) — один **правий** поворот навколо `z`:

```
            z(30)                       y(20)
            /                          /    \
         y(20)        ── R(z) ──►    x(10)  z(30)
         /
      x(10)
```

**Випадок RR** (bf(z) = −2, bf(z.Right) ≤ 0) — один **лівий** поворот навколо `z`:

```
      z(10)                             y(20)
         \                             /    \
         y(20)        ── L(z) ──►    z(10)  x(30)
            \
            x(30)
```

**Випадок LR** (bf(z) = +2, bf(z.Left) < 0) — спочатку **лівий** поворот навколо `z.Left`, потім **правий** навколо `z`:

```
        z(30)                  z(30)                  x(20)
        /                      /                     /    \
     y(10)    ── L(y) ──►   x(20)    ── R(z) ──►  y(10)  z(30)
        \                   /
        x(20)            y(10)
```

**Випадок RL** (bf(z) = −2, bf(z.Right) > 0) — спочатку **правий** навколо `z.Right`, потім **лівий** навколо `z`:

```
     z(10)                  z(10)                     x(20)
        \                      \                     /    \
        y(30)  ── R(y) ──►     x(20)  ── L(z) ──►  z(10)  y(30)
        /                         \
     x(20)                        y(30)
```

| Випадок | bf(z) | bf(дитини) | Дія |
|---------|-------|-----------|-----|
| LL | +2 | ≥ 0 | `RotateRight(z)` |
| RR | −2 | ≤ 0 | `RotateLeft(z)` |
| LR | +2 | < 0 | `z.Left = RotateLeft(z.Left)`, потім `RotateRight(z)` |
| RL | −2 | > 0 | `z.Right = RotateRight(z.Right)`, потім `RotateLeft(z)` |

> 💡 Випадок «bf(дитини) = 0» можливий **лише при видаленні**. Тоді достатньо одинарного повороту (саме тому в таблиці `≥ 0` і `≤ 0`, а не `> 0` / `< 0`).

### 12.4 Покрокове трасування вставки 10, 20, 30, 40, 50, 25

```
Вставка 10:     10

Вставка 20:     10
                  \
                   20

Вставка 30:     10 (bf=−2)             20
                  \                    /  \
                   20     ── RR ──►  10    30
                     \               L(10)
                      30

Вставка 40:       20
                 /  \
               10    30
                       \
                        40

Вставка 50:       20                        20
                 /  \                      /  \
               10    30 (bf=−2)          10    40
                       \       ── RR ──►      /  \
                        40      L(30)       30    50
                          \
                           50

Вставка 25:       20 (bf=−2)                 20                          30
                 /  \                       /  \                        /  \
               10    40 (bf=+1)           10    30                    20    40
                    /  \     ── R(40) ──►      /  \    ── L(20) ──►  / \     \
                  30    50                    25   40               10  25    50
                 /                                   \
               25                                     50
                          (випадок RL у вузлі 20)
```

Звичайне BST на тих самих ключах мало б висоту 4 (ланцюжок 10→20→30→40→50 + 25). AVL — висоту 2 (у ребрах).

### 12.5 Видалення

1. Видаляємо як у BST (три випадки).
2. На зворотному шляху до кореня **в кожному вузлі** оновлюємо висоту й за потреби робимо поворот.

На відміну від вставки (де достатньо **одного** повороту чи подвійного повороту на всю операцію), видалення може спричинити до `O(log n)` поворотів — по одному на кожному рівні.

### 12.6 Повна реалізація `AvlTree<T>`

```csharp
// Файл: AvlTreeDemo.cs
// Узагальнене AVL-дерево: вставка та видалення з ребалансуванням, трасування поворотів, перевірка інваріантів.

var tree = new AvlTree<int>();
tree.Log = message => Console.WriteLine("    " + message);

foreach (var key in new[] { 10, 20, 30, 40, 50, 25 })
{
    Console.WriteLine($"Insert({key})");
    tree.Add(key);
    Console.WriteLine($"  рівні: {tree.DescribeLevels()}");
}

Console.WriteLine();
Console.WriteLine($"Inorder: {string.Join(" ", tree)}; Count={tree.Count}; Height={tree.Height}; Valid={tree.IsValid()}");

Console.WriteLine();
foreach (var key in new[] { 10, 20, 30 })
{
    Console.WriteLine($"Remove({key}) -> {tree.Remove(key)}");
    Console.WriteLine($"  рівні: {tree.DescribeLevels()}; Valid={tree.IsValid()}");
}

// Стрес-тест: відсортовані ключі, потім видалення кожного другого.
tree.Log = null;
var big = new AvlTree<int>();
const int n = 100_000;
for (var i = 1; i <= n; i++)
{
    big.Add(i);
}

Console.WriteLine();
Console.WriteLine($"{n} відсортованих вставок: Height={big.Height} (у вузлах), 1.44*log2(n)={1.44 * Math.Log2(n):F1}, Valid={big.IsValid()}");
for (var i = 1; i <= n; i += 2)
{
    big.Remove(i);
}

Console.WriteLine($"Після видалення непарних: Count={big.Count}, Height={big.Height}, Valid={big.IsValid()}, Min={big.First()}");

// Рядки з власним компаратором.
var names = new AvlTree<string>(StringComparer.OrdinalIgnoreCase);
foreach (var name in new[] { "Olena", "andriy", "Bohdan", "ANDRIY", "iryna" })
{
    names.Add(name);
}

Console.WriteLine($"Імена без урахування регістру: {string.Join(", ", names)}");

/// <summary>AVL-дерево з унікальними ключами. Висота зберігається у вузлах: null = 0, лист = 1.</summary>
public sealed class AvlTree<T>(IComparer<T>? comparer = null) : IEnumerable<T>
{
    private sealed class Node(T key)
    {
        public T Key { get; set; } = key;
        public Node? Left { get; set; }
        public Node? Right { get; set; }
        public int Height { get; set; } = 1;
    }

    private readonly IComparer<T> _comparer = comparer ?? Comparer<T>.Default;
    private Node? _root;

    /// <summary>Необов'язковий журнал поворотів (для навчального трасування).</summary>
    public Action<string>? Log { get; set; }

    public int Count { get; private set; }

    public int Height => HeightOf(_root);

    // ---------- допоміжні ----------

    private static int HeightOf(Node? node) => node?.Height ?? 0;

    private static int BalanceOf(Node node) => HeightOf(node.Left) - HeightOf(node.Right);

    private static void UpdateHeight(Node node) =>
        node.Height = 1 + Math.Max(HeightOf(node.Left), HeightOf(node.Right));

    //        y                x
    //       / \              / \
    //      x   T3    ->     T1  y
    //     / \                  / \
    //    T1  T2               T2  T3
    private Node RotateRight(Node y)
    {
        var x = y.Left ?? throw new InvalidOperationException("Правий поворот потребує лівого сина");
        Log?.Invoke($"RotateRight({y.Key})");

        y.Left = x.Right; // T2 переходить до y
        x.Right = y;

        // Порядок важливий: спершу y (тепер нижче), потім x.
        UpdateHeight(y);
        UpdateHeight(x);
        return x; // новий корінь піддерева
    }

    //      x                    y
    //     / \                  / \
    //    T1  y       ->       x   T3
    //       / \              / \
    //      T2  T3           T1  T2
    private Node RotateLeft(Node x)
    {
        var y = x.Right ?? throw new InvalidOperationException("Лівий поворот потребує правого сина");
        Log?.Invoke($"RotateLeft({x.Key})");

        x.Right = y.Left; // T2 переходить до x
        y.Left = x;

        UpdateHeight(x);
        UpdateHeight(y);
        return y;
    }

    // Викликається на зворотному шляху для кожного вузла після вставки/видалення.
    private Node Rebalance(Node node)
    {
        UpdateHeight(node);
        var balance = BalanceOf(node);

        if (balance > 1)
        {
            // Ліве піддерево вище. node.Left гарантовано не null.
            var left = node.Left!;
            if (BalanceOf(left) < 0)
            {
                Log?.Invoke($"випадок LR у вузлі {node.Key}");
                node.Left = RotateLeft(left);
            }
            else
            {
                Log?.Invoke($"випадок LL у вузлі {node.Key}");
            }

            return RotateRight(node);
        }

        if (balance < -1)
        {
            var right = node.Right!;
            if (BalanceOf(right) > 0)
            {
                Log?.Invoke($"випадок RL у вузлі {node.Key}");
                node.Right = RotateRight(right);
            }
            else
            {
                Log?.Invoke($"випадок RR у вузлі {node.Key}");
            }

            return RotateLeft(node);
        }

        return node; // баланс у нормі
    }

    // ---------- вставка ----------

    public bool Add(T key)
    {
        var added = false;
        _root = Insert(_root, key, ref added);
        if (added)
        {
            Count++;
        }

        return added;
    }

    private Node Insert(Node? node, T key, ref bool added)
    {
        if (node is null)
        {
            added = true;
            return new Node(key);
        }

        var cmp = _comparer.Compare(key, node.Key);
        if (cmp < 0)
        {
            node.Left = Insert(node.Left, key, ref added);
        }
        else if (cmp > 0)
        {
            node.Right = Insert(node.Right, key, ref added);
        }
        else
        {
            return node; // дублікат: структура не змінилася
        }

        return Rebalance(node);
    }

    // ---------- видалення ----------

    public bool Remove(T key)
    {
        var removed = false;
        _root = Delete(_root, key, ref removed);
        if (removed)
        {
            Count--;
        }

        return removed;
    }

    private Node? Delete(Node? node, T key, ref bool removed)
    {
        if (node is null)
        {
            return null;
        }

        var cmp = _comparer.Compare(key, node.Key);
        if (cmp < 0)
        {
            node.Left = Delete(node.Left, key, ref removed);
        }
        else if (cmp > 0)
        {
            node.Right = Delete(node.Right, key, ref removed);
        }
        else
        {
            removed = true;

            // 0 або 1 дитина: піддерево дитини вже є коректним AVL.
            if (node.Left is null || node.Right is null)
            {
                return node.Left ?? node.Right;
            }

            // 2 дитини: копіюємо мінімум правого піддерева й видаляємо його звідти.
            var successor = node.Right;
            while (successor.Left is not null)
            {
                successor = successor.Left;
            }

            node.Key = successor.Key;
            var ignored = false;
            node.Right = Delete(node.Right, successor.Key, ref ignored);
        }

        // Ребалансування на кожному рівні шляху вгору.
        return Rebalance(node);
    }

    // ---------- перевірки та друк ----------

    /// <summary>Перевіряє збережені висоти, |bf| ≤ 1, строге зростання inorder та Count.</summary>
    public bool IsValid()
    {
        if (!CheckHeights(_root, out _))
        {
            return false;
        }

        // BST-порядок: inorder має строго зростати (перевірка всього піддерева, а не лише дітей).
        var seen = 0;
        var hasPrevious = false;
        T previous = default!;
        foreach (var key in this)
        {
            if (hasPrevious && _comparer.Compare(previous, key) >= 0)
            {
                return false;
            }

            previous = key;
            hasPrevious = true;
            seen++;
        }

        return seen == Count;
    }

    private static bool CheckHeights(Node? node, out int height)
    {
        height = 0;
        if (node is null)
        {
            return true;
        }

        if (!CheckHeights(node.Left, out var leftHeight) || !CheckHeights(node.Right, out var rightHeight))
        {
            return false;
        }

        height = 1 + Math.Max(leftHeight, rightHeight);
        return height == node.Height && Math.Abs(leftHeight - rightHeight) <= 1;
    }

    public string DescribeLevels()
    {
        if (_root is null)
        {
            return "(порожнє)";
        }

        var levels = new List<string>();
        var queue = new Queue<Node>([_root]);
        while (queue.Count > 0)
        {
            var level = new List<string>();
            for (var size = queue.Count; size > 0; size--)
            {
                var node = queue.Dequeue();
                level.Add($"{node.Key}(bf={BalanceOf(node):+0;-0;0})");
                if (node.Left is not null) queue.Enqueue(node.Left);
                if (node.Right is not null) queue.Enqueue(node.Right);
            }

            levels.Add(string.Join(" ", level));
        }

        return string.Join(" | ", levels);
    }

    public IEnumerator<T> GetEnumerator()
    {
        var stack = new Stack<Node>();
        var current = _root;
        while (current is not null || stack.Count > 0)
        {
            while (current is not null)
            {
                stack.Push(current);
                current = current.Left;
            }

            current = stack.Pop();
            yield return current.Key;
            current = current.Right;
        }
    }

    System.Collections.IEnumerator System.Collections.IEnumerable.GetEnumerator() => GetEnumerator();
}
```

**Приклад запуску:**

```text
Insert(10)
  рівні: 10(bf=0)
Insert(20)
  рівні: 10(bf=-1) | 20(bf=0)
Insert(30)
    випадок RR у вузлі 10
    RotateLeft(10)
  рівні: 20(bf=0) | 10(bf=0) 30(bf=0)
Insert(40)
  рівні: 20(bf=-1) | 10(bf=0) 30(bf=-1) | 40(bf=0)
Insert(50)
    випадок RR у вузлі 30
    RotateLeft(30)
  рівні: 20(bf=-1) | 10(bf=0) 40(bf=0) | 30(bf=0) 50(bf=0)
Insert(25)
    випадок RL у вузлі 20
    RotateRight(40)
    RotateLeft(20)
  рівні: 30(bf=0) | 20(bf=0) 40(bf=-1) | 10(bf=0) 25(bf=0) 50(bf=0)

Inorder: 10 20 25 30 40 50; Count=6; Height=3; Valid=True

Remove(10) -> True
  рівні: 30(bf=0) | 20(bf=-1) 40(bf=-1) | 25(bf=0) 50(bf=0); Valid=True
Remove(20) -> True
  рівні: 30(bf=-1) | 25(bf=0) 40(bf=-1) | 50(bf=0); Valid=True
Remove(30) -> True
  рівні: 40(bf=0) | 25(bf=0) 50(bf=0); Valid=True

100000 відсортованих вставок: Height=17 (у вузлах), 1.44*log2(n)=23.9, Valid=True
Після видалення непарних: Count=50000, Height=16, Valid=True, Min=2
Імена без урахування регістру: andriy, Bohdan, iryna, Olena
```

### 12.7 Чому висота AVL — O(log n): дерева Фібоначчі

Нехай `N(h)` — **мінімальна** кількість вузлів в AVL-дереві висоти `h` (у вузлах). Найбільш «худе» AVL-дерево має в корені одне піддерево висоти `h−1`, а інше — `h−2` (менше не можна, бо |bf| ≤ 1):

```
N(0) = 0,  N(1) = 1,  N(h) = N(h−1) + N(h−2) + 1
```

Це майже рекурентність Фібоначчі: `N(h) = F(h+2) − 1`. Оскільки `F(k) ≈ φ^k / √5`, де `φ ≈ 1.618`:

```
n ≥ N(h) ≈ φ^(h+2) / √5   ⟹   h ≤ log_φ(n) + O(1) ≈ 1.44 · log₂(n)
```

Отже, AVL-дерево **щонайбільше на 44% вище** за ідеально збалансоване. Такі мінімальні дерева називають **деревами Фібоначчі**:

```
h=1:  o      h=2:   o       h=3:     o          h=4:         o
                   /                /  \                   /    \
                  o                o    o                 o      o
                                  /                     /  \    /
                                 o                     o    o  o
                                                      /
                                                     o
N:    1            2                 4                     7
```

```csharp
// Файл: FibonacciTrees.cs
// Мінімальна кількість вузлів AVL-дерева висоти h та порівняння з log2.

Console.WriteLine(" h | N(h) мін. вузлів | 2^h - 1 макс. | 1.44*log2(N(h)+2) ");
Console.WriteLine("---+------------------+---------------+-------------------");

long previous = 0; // N(0)
long current = 1;  // N(1)
for (var h = 1; h <= 30; h++)
{
    if (h is <= 8 or 10 or 20 or 30)
    {
        var maxNodes = (1L << h) - 1;
        Console.WriteLine($"{h,2} | {current,16:N0} | {maxNodes,13:N0} | {1.44 * Math.Log2(current + 2),17:F2}");
    }

    // N(h+1) = N(h) + N(h-1) + 1
    (previous, current) = (current, current + previous + 1);
}

// Будуємо саме дерево Фібоначчі висоти 4 і перевіряємо, що це валідне AVL.
var fib4 = BuildFibonacciTree(4, ref KeyCounter.Next);
Console.WriteLine();
Console.WriteLine($"Дерево Фібоначчі h=4: вузлів={Count(fib4)}, висота={HeightOf(fib4)}, AVL={IsAvl(fib4)}");

// Ліве піддерево на 1 вище за праве — у КОЖНОМУ вузлі bf = +1 (крім листів).
static Node? BuildFibonacciTree(int h, ref int nextKey)
{
    if (h <= 0)
    {
        return null;
    }

    var left = BuildFibonacciTree(h - 1, ref nextKey);
    var key = nextKey++;               // inorder-нумерація дає валідний BST
    var right = BuildFibonacciTree(h - 2, ref nextKey);
    return new Node(key, left, right);
}

static int Count(Node? node) => node is null ? 0 : 1 + Count(node.Left) + Count(node.Right);

static int HeightOf(Node? node) => node is null ? 0 : 1 + Math.Max(HeightOf(node.Left), HeightOf(node.Right));

static bool IsAvl(Node? node) =>
    node is null || (Math.Abs(HeightOf(node.Left) - HeightOf(node.Right)) <= 1 && IsAvl(node.Left) && IsAvl(node.Right));

public sealed record Node(int Key, Node? Left, Node? Right);

// Статичне поле, щоб передати лічильник за посиланням із top-level коду.
public static class KeyCounter
{
    public static int Next;
}
```

**Приклад запуску:**

```text
 h | N(h) мін. вузлів | 2^h - 1 макс. | 1.44*log2(N(h)+2) 
---+------------------+---------------+-------------------
 1 |                1 |             1 |              2.28
 2 |                2 |             3 |              2.88
 3 |                4 |             7 |              3.72
 4 |                7 |            15 |              4.56
 5 |               12 |            31 |              5.48
 6 |               20 |            63 |              6.42
 7 |               33 |           127 |              7.39
 8 |               54 |           255 |              8.36
10 |              143 |         1,023 |             10.34
20 |           17,710 |     1,048,575 |             20.32
30 |        2,178,308 | 1,073,741,823 |             30.32

Дерево Фібоначчі h=4: вузлів=7, висота=4, AVL=True
```

### 12.8 Складність AVL

| Операція | Час | Повороти |
|----------|-----|----------|
| Пошук | O(log n) | 0 |
| Вставка | O(log n) | ≤ 1 одинарний або 1 подвійний |
| Видалення | O(log n) | до O(log n) |
| Пам'ять | O(n) | + `int Height` у кожному вузлі (або 2 біти для bf) |

### Типові помилки

- **Оновлювати висоти в неправильному порядку** при повороті: спочатку треба оновити вузол, що опинився **нижче**, потім новий корінь.
- **Забути повернути новий корінь** з `RotateLeft/RotateRight` і присвоїти його батьку (`node.Left = RotateLeft(left)`).
- **Переплутати LR і RL**: для LR перший поворот — **лівий** навколо **лівого** сина.
- **Використовувати строгі нерівності** `bf(child) > 0` для LL при видаленні — випадок `bf(child) = 0` залишиться неправильно обробленим (зайвий подвійний поворот ламає баланс).
- **Не ребалансувати після видалення** або ребалансувати лише один вузол, а не весь шлях до кореня.
- **Висота «в ребрах» з `null = 0`** — плутанина; тримайте одну домовленість (`null = 0`, лист = 1).

### Міні-вправа 12.1

Вставте в порожнє AVL-дерево `3, 2, 1, 4, 5, 6, 7`. Які повороти відбудуться і яке дерево вийде?

<details>
<summary>Розв'язок</summary>

- `1` → LL у `3` → `RotateRight(3)`: `2(1, 3)`.
- `5` → RR у `3` → `RotateLeft(3)`: `2(1, 4(3, 5))`.
- `6` → RR у `2` → `RotateLeft(2)`: `4(2(1, 3), 5(, 6))`.
- `7` → RR у `5` → `RotateLeft(5)`: `4(2(1, 3), 6(5, 7))`.

Отримали **досконале** дерево висоти 3 (у вузлах):

```
          4
        /   \
       2     6
      / \   / \
     1   3 5   7
```

Можна перевірити, додавши ці ключі в `AvlTree<int>` із розділу 12.6 з увімкненим `Log`.

</details>

---

## 13. Червоно-чорні дерева, B-дерева, префіксні дерева

*≈ 12 хв*

### 13.1 Червоно-чорне дерево (огляд)

**Червоно-чорне дерево (RB-tree)** — BST, у якому кожен вузол має колір, і виконуються **5 властивостей**:

1. Кожен вузол червоний або чорний.
2. Корінь — чорний.
3. Усі листи-заглушки `NIL` — чорні.
4. Червоний вузол не має червоних дітей (немає двох червоних поспіль).
5. Кожен шлях від вузла до його нащадків-`NIL` містить однакову кількість чорних вузлів (**чорна висота**).

```
                 13(B)
               /       \
           8(R)         17(R)
          /    \        /    \
       1(B)   11(B)  15(B)  25(B)
          \                 /    \
          6(R)           22(R)  27(R)
```

З властивостей 4 і 5: найдовший шлях (чергування червоних і чорних) не більше ніж **удвічі** довший за найкоротший (лише чорні) → `h ≤ 2·log₂(n + 1)`.

Балансування після вставки/видалення — **перефарбування** і **не більше 2 поворотів при вставці та 3 при видаленні**.

**Чому .NET `SortedSet<T>` та `SortedDictionary<TKey,TValue>` (а також `std::map` у C++, `TreeMap` у Java) використовують RB-дерева:**

- Оновлення **дешевші** за AVL: константна кількість поворотів, багато змін — лише зміна кольору.
- Колір — **1 біт** (у .NET — поле `NodeColor`) замість висоти.
- Гарантія `O(log n)` для всіх операцій при менших витратах на запис; для змішаного навантаження (часті вставки/видалення) це вигідніше за трохи коротшу висоту AVL.

### 13.2 AVL vs червоно-чорне дерево

| Критерій | AVL | Червоно-чорне |
|----------|-----|---------------|
| Інваріант | \|bf\| ≤ 1 у кожному вузлі | 5 властивостей кольорів |
| Максимальна висота | ≈ 1.44·log₂ n | ≈ 2·log₂ n |
| Пошук | трохи швидше (нижче дерево) | трохи повільніше |
| Повороти при вставці | ≤ 2 | ≤ 2 |
| Повороти при видаленні | до O(log n) | ≤ 3 |
| Додаткова пам'ять у вузлі | висота (int) або 2 біти | 1 біт кольору |
| Складність реалізації | помірна | вища (багато випадків) |
| Де краще | переважно читання (словники, індекси в пам'яті) | змішане навантаження, стандартні бібліотеки |
| Приклади | деякі БД в пам'яті, lookup-таблиці | .NET `SortedSet`/`SortedDictionary`, Java `TreeMap`, C++ `std::map`, планувальник Linux CFS |

### 13.3 B-дерева та B+-дерева для баз даних

Бінарні дерева погано підходять для **диска**: кожен перехід вузлом — потенційно окреме читання сторінки (≈ 0.1 мс SSD, ≈ 10 мс HDD). Для мільярда ключів AVL має висоту ≈ 43 → 43 читання диска.

**B-дерево порядку `m`**: вузол — ціла **сторінка** (4–16 КБ) з сотнями ключів і дітей.

- кожен вузол має від `⌈m/2⌉` до `m` дітей (крім кореня);
- ключі у вузлі відсортовані; діти між ключами містять проміжні значення;
- усі листи на **одній глибині** (дерево росте вгору, розщеплюючи корінь).

```
B-дерево порядку 4 (до 3 ключів у вузлі):

                    [ 20 | 40 ]
                  /      |      \
       [5 | 10 | 15]  [25 | 30]  [45 | 50 | 60]
```

**B+-дерево** (використовують PostgreSQL, MySQL InnoDB, SQL Server, SQLite, файлові системи NTFS/ext4/APFS):

- **усі значення** (або посилання на рядки) — лише в **листах**; внутрішні вузли містять тільки ключі-роздільники → більше дітей на сторінку;
- листи зв'язані в **список** → діапазонний запит `WHERE price BETWEEN 10 AND 20` = один спуск + послідовний прохід по листах.

```
                  [ 30 | 60 ]                   ← лише роздільники
                /      |      \
   [10|20] ⇄ [30|40|50] ⇄ [60|70|80]            ← листи з даними, зв'язані
```

При `m = 500` дерево висоти 4 вміщує `500⁴ ≈ 6·10¹⁰` ключів — **4 читання диска** на пошук, причому верхні рівні зазвичай у кеші.

### 13.4 Префіксне дерево (trie) — анонс

**Trie** зберігає рядки посимвольно: шлях від кореня до вузла — префікс. Пошук слова довжини `L` — `O(L)`, **незалежно** від кількості слів. Застосування: автодоповнення, перевірка орфографії, IP-маршрутизація (longest prefix match).

```
Слова: car, cat, cart, dog

          (root)
          /    \
         c      d
         |      |
         a      o
        / \     |
       r*  t*   g*
       |
       t*            * — кінець слова
```

```csharp
// Файл: TrieTeaser.cs
// Мінімальний trie: вставка, пошук слова, пошук за префіксом.

var trie = new Trie();
foreach (var word in new[] { "car", "cat", "cart", "care", "dog", "dot" })
{
    trie.Insert(word);
}

Console.WriteLine($"Contains(\"car\")  = {trie.Contains("car")}");
Console.WriteLine($"Contains(\"ca\")   = {trie.Contains("ca")}  (лише префікс)");
Console.WriteLine($"StartsWith(\"ca\") = {string.Join(", ", trie.WordsWithPrefix("ca"))}");
Console.WriteLine($"StartsWith(\"do\") = {string.Join(", ", trie.WordsWithPrefix("do"))}");
Console.WriteLine($"StartsWith(\"x\")  = [{string.Join(", ", trie.WordsWithPrefix("x"))}]");

public sealed class Trie
{
    private sealed class Node
    {
        // SortedDictionary — щоб автодоповнення видавало слова за алфавітом.
        public SortedDictionary<char, Node> Children { get; } = [];
        public bool IsWord { get; set; }
    }

    private readonly Node _root = new();

    public void Insert(string word)
    {
        var node = _root;
        foreach (var ch in word)
        {
            if (!node.Children.TryGetValue(ch, out var child))
            {
                child = new Node();
                node.Children[ch] = child;
            }

            node = child;
        }

        node.IsWord = true;
    }

    public bool Contains(string word) => Find(word)?.IsWord == true;

    public List<string> WordsWithPrefix(string prefix)
    {
        var result = new List<string>();
        var start = Find(prefix);
        if (start is not null)
        {
            Collect(start, new System.Text.StringBuilder(prefix), result);
        }

        return result;
    }

    private Node? Find(string prefix)
    {
        var node = _root;
        foreach (var ch in prefix)
        {
            if (!node.Children.TryGetValue(ch, out var child))
            {
                return null;
            }

            node = child;
        }

        return node;
    }

    // DFS (preorder) з backtracking по StringBuilder.
    private static void Collect(Node node, System.Text.StringBuilder current, List<string> result)
    {
        if (node.IsWord)
        {
            result.Add(current.ToString());
        }

        foreach (var (ch, child) in node.Children)
        {
            current.Append(ch);
            Collect(child, current, result);
            current.Length--;
        }
    }
}
```

**Приклад запуску:**

```text
Contains("car")  = True
Contains("ca")   = False  (лише префікс)
StartsWith("ca") = car, care, cart, cat
StartsWith("do") = dog, dot
StartsWith("x")  = []
```

Детальніше trie, сегментні дерева та дерева Фенвіка — у наступних лекціях.

---

## 14. Дерева в .NET: SortedSet і SortedDictionary

*≈ 8 хв*

### 14.1 Огляд колекцій

| Колекція | Структура | Add/Remove/Contains | Min/Max | Діапазон | Порядок перебору |
|----------|-----------|--------------------|---------|----------|------------------|
| `HashSet<T>` | хеш-таблиця | O(1) у середньому | O(n) | ❌ | довільний |
| `SortedSet<T>` | червоно-чорне дерево | O(log n) | O(log n) | ✅ `GetViewBetween` | відсортований |
| `Dictionary<K,V>` | хеш-таблиця | O(1) у середньому | O(n) | ❌ | довільний |
| `SortedDictionary<K,V>` | червоно-чорне дерево | O(log n) | O(log n)* | ❌ напряму | за ключем |
| `SortedList<K,V>` | два відсортовані масиви | O(n) вставка, O(log n) пошук | O(1) | через індекси | за ключем |
| `PriorityQueue<E,P>` | 4-арна купа в масиві | O(log n) Enqueue/Dequeue | O(1) лише мін. | ❌ | довільний |

\* через `First()`/`Last()` з LINQ: `First()` — O(log n), але `Last()` у `SortedDictionary` перебирає все, O(n). Для частого доступу до max використовуйте `SortedSet<T>` (`Max` — O(log n)).

### 14.2 Корисні можливості `SortedSet<T>`

- `Min`, `Max` — O(log n);
- `GetViewBetween(lower, upper)` — «живе» представлення діапазону `[lower, upper]` (не копія!); зміни в ньому видно в оригіналі;
- `Reverse()` — перебір у зворотному порядку;
- власний `IComparer<T>` — довільний порядок (наприклад, за спаданням очок);
- `Add` повертає `false` для дубліката (за компаратором!).

### 14.3 Код: базові операції та таблиця лідерів

```csharp
// Файл: DotNetTrees.cs
// SortedSet<T>, GetViewBetween, SortedDictionary і таблиця лідерів на SortedSet з компаратором.

var set = new SortedSet<int> { 50, 20, 80, 10, 30, 70, 90, 20 };
Console.WriteLine($"SortedSet: {string.Join(" ", set)} (Count={set.Count}, дублікат 20 проігноровано)");
Console.WriteLine($"Min={set.Min}, Max={set.Max}");

// Діапазон [25, 75] — представлення, а не копія.
var view = set.GetViewBetween(25, 75);
Console.WriteLine($"GetViewBetween(25, 75): {string.Join(" ", view)}; Min={view.Min}, Max={view.Max}");

set.Add(60);
Console.WriteLine($"Після set.Add(60) view: {string.Join(" ", view)}");

// Floor/ceiling через представлення: найбільший <= 65 та найменший >= 65.
Console.WriteLine($"floor(65)={set.GetViewBetween(int.MinValue, 65).Max}, ceiling(65)={set.GetViewBetween(65, int.MaxValue).Min}");
Console.WriteLine($"У зворотному порядку: {string.Join(" ", set.Reverse())}");

// SortedDictionary: ключі завжди впорядковані.
var stock = new SortedDictionary<string, int>(StringComparer.Ordinal)
{
    ["pear"] = 12,
    ["apple"] = 40,
    ["cherry"] = 7,
};
stock["banana"] = 25;
stock["apple"] -= 5;
Console.WriteLine();
Console.WriteLine("SortedDictionary: " + string.Join(", ", stock.Select(p => $"{p.Key}={p.Value}")));

// ---------- Таблиця лідерів ----------
var board = new Leaderboard();
board.Submit("olena", 1200);
board.Submit("andriy", 950);
board.Submit("bohdan", 1200);
board.Submit("iryna", 1430);
board.Submit("taras", 700);
board.Submit("andriy", 1500); // оновлення результату гравця

Console.WriteLine();
Console.WriteLine("Топ-3:");
foreach (var (rank, entry) in board.Top(3).Select((e, i) => (i + 1, e)))
{
    Console.WriteLine($"  {rank}. {entry.Player,-7} {entry.Score}");
}

Console.WriteLine($"Гравці з очками [900, 1300]: {string.Join(", ", board.InScoreRange(900, 1300).Select(e => $"{e.Player}({e.Score})"))}");
Console.WriteLine($"Місце bohdan: {board.RankOf("bohdan")}, місце taras: {board.RankOf("taras")}");

public sealed record Entry(string Player, int Score);

/// <summary>Таблиця лідерів: SortedSet тримає порядок, Dictionary — поточний результат гравця.</summary>
public sealed class Leaderboard
{
    // Порядок: більше очок — вище; при рівності — за ім'ям (інакше SortedSet вважатиме записи дублікатами!).
    private static readonly IComparer<Entry> ByScoreDesc = Comparer<Entry>.Create((a, b) =>
    {
        var byScore = b.Score.CompareTo(a.Score);
        return byScore != 0 ? byScore : string.CompareOrdinal(a.Player, b.Player);
    });

    private readonly SortedSet<Entry> _ranking = new(ByScoreDesc);
    private readonly Dictionary<string, Entry> _current = [];

    // O(log n): видаляємо старий запис гравця і вставляємо новий.
    public void Submit(string player, int score)
    {
        if (_current.TryGetValue(player, out var old))
        {
            _ranking.Remove(old);
        }

        var entry = new Entry(player, score);
        _ranking.Add(entry);
        _current[player] = entry;
    }

    public IEnumerable<Entry> Top(int count) => _ranking.Take(count);

    // Сортування за спаданням, тому «нижня» межа view — це БІЛЬШИЙ бал.
    // Ім'я string.Empty — найменше, char.MaxValue — «найбільше» серед імен при однаковому балі.
    public SortedSet<Entry> InScoreRange(int minScore, int maxScore) =>
        _ranking.GetViewBetween(new Entry(string.Empty, maxScore), new Entry(new string(char.MaxValue, 1), minScore));

    // Простий O(n) підрахунок місця; для O(log n) потрібне order-statistic дерево з розмірами піддерев.
    public int RankOf(string player)
    {
        var target = _current[player];
        return _ranking.TakeWhile(e => e != target).Count() + 1;
    }
}
```

**Приклад запуску:**

```text
SortedSet: 10 20 30 50 70 80 90 (Count=7, дублікат 20 проігноровано)
Min=10, Max=90
GetViewBetween(25, 75): 30 50 70; Min=30, Max=70
Після set.Add(60) view: 30 50 60 70
floor(65)=60, ceiling(65)=70
У зворотному порядку: 90 80 70 60 50 30 20 10

SortedDictionary: apple=35, banana=25, cherry=7, pear=12

Топ-3:
  1. andriy  1500
  2. iryna   1430
  3. bohdan  1200
Гравці з очками [900, 1300]: bohdan(1200), olena(1200)
Місце bohdan: 3, місце taras: 5
```

### Типові помилки

- **Компаратор, що повертає 0 для різних об'єктів** (наприклад, лише за очками) → `SortedSet` мовчки не додасть другого гравця з тим самим балом.
- **Змінювати поле, що бере участь у порівнянні**, коли об'єкт уже в `SortedSet` → дерево зламане, `Remove`/`Contains` перестають знаходити елемент. Спочатку `Remove`, потім зміна, потім `Add` (або незмінні `record`).
- **`GetViewBetween(lower, upper)` з `lower > upper`** за компаратором → `ArgumentException`.
- **`sortedDictionary.Last()`** у гарячому циклі — це O(n).
- Вибір `SortedList` для частих вставок у середину — O(n) на вставку через зсув масиву.

---

## 15. Підсумки, питання для самоперевірки, практичні задачі

*≈ 7 хв*

### 15.1 Головне

| Тема | Ключова ідея |
|------|--------------|
| Термінологія | глибина — від кореня (згори вниз), висота — до листа (знизу вгору) |
| Представлення | вузли з посиланнями (довільна форма), масив `2i+1/2i+2` (завершені дерева) |
| DFS-обходи | pre/in/post, рекурсивно або з `Stack<T>`; Morris — O(1) пам'яті |
| BFS | `Queue<T>`, фіксуємо `Count` на початку рівня |
| Рекурсивні патерни | «розв'яжи для дітей — скомбінуй»; «повертаю одне — оновлюю глобальне» |
| BST | ліве < вузол < праве для **всього** піддерева; inorder = відсортовано; O(h) |
| Видалення з BST | лист / одна дитина / дві дитини → наступник |
| Вироджений BST | відсортований вхід → h = n − 1 |
| AVL | \|bf\| ≤ 1; 4 випадки поворотів; h ≤ 1.44·log₂ n |
| RB-дерева | 5 властивостей; h ≤ 2·log₂ n; дешевші оновлення; основа `SortedSet` |
| B+-дерева | широкі вузли-сторінки, дані в листах, зв'язаний список листів — індекси БД |

### 15.2 Шпаргалка складності

| Структура | Пошук | Вставка | Видалення | Min/Max | Діапазон (m результатів) |
|-----------|-------|---------|-----------|---------|---------------------------|
| BST (середній) | O(log n) | O(log n) | O(log n) | O(log n) | O(log n + m) |
| BST (найгірший) | O(n) | O(n) | O(n) | O(n) | O(n) |
| AVL | O(log n) | O(log n) | O(log n) | O(log n) | O(log n + m) |
| Червоно-чорне | O(log n) | O(log n) | O(log n) | O(log n) | O(log n + m) |
| B+-дерево (порядок m) | O(log_m n) | O(log_m n) | O(log_m n) | O(log_m n) | O(log_m n + m) |
| Відсортований масив | O(log n) | O(n) | O(n) | O(1) | O(log n + m) |
| Хеш-таблиця | O(1)* | O(1)* | O(1)* | O(n) | O(n) |

### 15.3 Питання для самоперевірки

1. Чим відрізняються глибина вузла і висота вузла? Яка висота порожнього дерева в домовленості «в ребрах»?
2. Наведіть приклад дерева, яке є повним (full), але не завершеним (complete), і навпаки.
3. Скільки вузлів має досконале бінарне дерево висоти `h` (у ребрах)? Скільки з них листів?
4. За якими формулами знаходяться діти і батько вузла з індексом `i` у масивному представленні? Чому це представлення погане для виродженого дерева?
5. Який обхід використати, щоб: а) скопіювати дерево; б) звільнити ресурси всіх вузлів; в) отримати відсортовані ключі BST?
6. Чому ітеративний inorder має умову циклу `current != null || stack.Count > 0`?
7. Як Morris-обхід досягає O(1) додаткової пам'яті? Який у нього недолік?
8. Скільки пам'яті потребують DFS і BFS для досконалого і для виродженого дерева?
9. Чому діаметр дерева не можна обчислити як `height(left) + height(right) + 2` лише для кореня?
10. Як перевірити збалансованість за O(n), а не O(n²)?
11. Чому для однозначного відновлення дерева не достатньо preorder + postorder?
12. Сформулюйте інваріант BST. Чому перевірка лише батька і безпосередніх дітей недостатня?
13. Опишіть три випадки видалення з BST. Чому в третьому випадку можна взяти наступника?
14. Що таке floor і ceiling? Чим вони відрізняються від predecessor і successor?
15. Чому валідація BST з межами типу `int` може дати хибний результат?
16. Як побудувати BST мінімальної висоти з відсортованого масиву? Яка висота вийде для n = 1000?
17. Що таке фактор балансу? Які значення допустимі в AVL-дереві?
18. Опишіть чотири випадки дисбалансу AVL і відповідні повороти. Як визначити, що потрібен подвійний поворот?
19. Чому висота AVL-дерева не перевищує ≈ 1.44·log₂ n? Що таке дерево Фібоначчі?
20. Скільки поворотів може знадобитися при вставці та при видаленні в AVL?
21. Перелічіть 5 властивостей червоно-чорного дерева. Чому з них випливає h ≤ 2·log₂(n+1)?
22. Чому .NET використовує червоно-чорні дерева в `SortedSet<T>`, а не AVL?
23. Чому бази даних використовують B+-дерева, а не AVL чи RB-дерева?
24. Що станеться, якщо компаратор `SortedSet<T>` повертає 0 для двох різних об'єктів?
25. Чим `GetViewBetween` відрізняється від `Where(x => lo <= x && x <= hi)` за складністю і семантикою?

### 15.4 Практичні задачі

1. **Кузени.** Два вузли — кузени, якщо вони на одній глибині, але мають різних батьків. Напишіть `AreCousins(root, x, y)` за один BFS.
2. **Шляхи з сумою, що починаються будь-де.** Порахуйте кількість шляхів «униз» (не обов'язково від кореня і до листа) із сумою `k`. Підказка: префіксні суми + `Dictionary<long,int>` → O(n).
3. **Вид знизу.** Розширте вертикальний обхід так, щоб для кожного стовпчика виводився найнижчий вузол.
4. **Серіалізація preorder.** Реалізуйте серіалізацію/десеріалізацію через preorder із маркерами `#` (рекурсивно) і порівняйте розмір рядка з рівневим форматом на виродженому дереві.
5. **Злиття двох BST.** Дано два BST із `n` і `m` вузлами. Отримайте відсортований список усіх ключів за O(n + m) часу й O(h₁ + h₂) пам'яті (два ітератори зі стеком).
6. **Order-statistic tree.** Додайте до вузла BST поле `Size` і реалізуйте `Select(k)` та `Rank(key)` за O(h). Обережно з оновленням `Size` при видаленні.
7. **Рандомізоване тестування AVL.** Протестуйте на 10⁵ випадкових вставках/видаленнях `AvlTree<int>` з розділу 12.6 порівнянням із `SortedSet<int>`.
8. **AVL-словник.** Перетворіть `AvlTree<T>` на `AvlMap<TKey, TValue>` з індексатором, `TryGetValue` і `Floor/Ceiling`.
9. **Порівняння продуктивності.** Використовуючи `System.Diagnostics.Stopwatch`, виміряйте 10⁶ вставок відсортованих і випадкових ключів у: власне BST (лише випадкові!), `AvlTree<int>`, `SortedSet<int>`, `HashSet<int>`. Поясніть результати.
10. **Таблиця лідерів O(log n).** Модифікуйте `Leaderboard` так, щоб `RankOf` працював за O(log n) (підказка: задача 6).
11. **Trie з видаленням.** Додайте `Remove(word)` до `Trie`, що видаляє вузли, які більше не належать жодному слову.
12. **Нижній спільний предок із батьківськими посиланнями.** Для вузлів із полем `Parent` знайдіть LCA за O(h) часу та O(1) пам'яті (вирівняйте глибини або використайте прийом «двох вказівників» як для перетину списків).

<details>
<summary>Розв'язок задачі 1 (кузени)</summary>

```csharp
// Файл: Cousins.cs
//          1
//        /   \
//       2     3
//      / \     \
//     4   5     6
var root = new Node(1, new Node(2, new Node(4), new Node(5)), new Node(3, null, new Node(6)));

Console.WriteLine($"4 і 6 — кузени? {AreCousins(root, 4, 6)}");
Console.WriteLine($"4 і 5 — кузени? {AreCousins(root, 4, 5)} (брати)");
Console.WriteLine($"2 і 6 — кузени? {AreCousins(root, 2, 6)} (різні рівні)");

static bool AreCousins(Node root, int x, int y)
{
    var queue = new Queue<(Node Node, Node? Parent)>([(root, null)]);

    while (queue.Count > 0)
    {
        Node? parentX = null, parentY = null;

        for (var size = queue.Count; size > 0; size--)
        {
            var (node, parent) = queue.Dequeue();
            if (node.Value == x) parentX = parent ?? node;
            if (node.Value == y) parentY = parent ?? node;
            if (node.Left is not null) queue.Enqueue((node.Left, node));
            if (node.Right is not null) queue.Enqueue((node.Right, node));
        }

        // Обидва на цьому рівні — кузени, якщо батьки різні.
        if (parentX is not null && parentY is not null)
        {
            return parentX != parentY;
        }

        // Лише один знайдений на цьому рівні — глибини різні.
        if (parentX is not null || parentY is not null)
        {
            return false;
        }
    }

    return false;
}

public sealed record Node(int Value, Node? Left = null, Node? Right = null);
```

```text
4 і 6 — кузени? True
4 і 5 — кузени? False (брати)
2 і 6 — кузени? False (різні рівні)
```

</details>

<details>
<summary>Розв'язок задачі 2 (шляхи з сумою k)</summary>

Префіксна сума від кореня до поточного вузла `S`. Кількість шляхів, що закінчуються в поточному вузлі, дорівнює кількості предків (включно з «порожнім префіксом»), для яких префіксна сума дорівнює `S − k`.

```csharp
// Файл: PathSumAnywhere.cs
//          10
//         /  \
//        5    -3
//       / \     \
//      3   2     11
//     / \   \
//    3  -2   1          шляхи з сумою 8: 5->3, 5->2->1, -3->11
var root = new Node(10,
    new Node(5, new Node(3, new Node(3), new Node(-2)), new Node(2, null, new Node(1))),
    new Node(-3, null, new Node(11)));

Console.WriteLine($"Шляхів із сумою 8: {CountPaths(root, 8)}");

static int CountPaths(Node root, long target)
{
    var prefixCounts = new Dictionary<long, int> { [0] = 1 }; // порожній префікс
    return Dfs(root, 0);

    int Dfs(Node? node, long sum)
    {
        if (node is null)
        {
            return 0;
        }

        sum += node.Value;
        var count = prefixCounts.GetValueOrDefault(sum - target);

        prefixCounts[sum] = prefixCounts.GetValueOrDefault(sum) + 1;
        count += Dfs(node.Left, sum) + Dfs(node.Right, sum);
        prefixCounts[sum]--; // backtracking: префікс більше не на поточному шляху

        return count;
    }
}

public sealed record Node(int Value, Node? Left = null, Node? Right = null);
```

```text
Шляхів із сумою 8: 3
```

</details>

<details>
<summary>Розв'язок задачі 5 (злиття двох BST)</summary>

```csharp
// Файл: MergeTwoBsts.cs
var a = new Node(5, new Node(2, new Node(1), new Node(4)), new Node(9));
var b = new Node(6, new Node(3), new Node(8, new Node(7), new Node(10)));

Console.WriteLine(string.Join(" ", Merge(a, b)));

static IEnumerable<int> Merge(Node? first, Node? second)
{
    var s1 = new Stack<Node>();
    var s2 = new Stack<Node>();
    PushLeft(s1, first);
    PushLeft(s2, second);

    while (s1.Count > 0 || s2.Count > 0)
    {
        // Беремо менший з двох «поточних мінімумів».
        var fromFirst = s2.Count == 0 || (s1.Count > 0 && s1.Peek().Value <= s2.Peek().Value);
        var stack = fromFirst ? s1 : s2;
        var node = stack.Pop();
        PushLeft(stack, node.Right);
        yield return node.Value;
    }
}

static void PushLeft(Stack<Node> stack, Node? node)
{
    for (; node is not null; node = node.Left)
    {
        stack.Push(node);
    }
}

public sealed record Node(int Value, Node? Left = null, Node? Right = null);
```

```text
1 2 3 4 5 6 7 8 9 10
```

</details>

---

### Література

- Т. Кормен, Ч. Лейзерсон, Р. Рівест, К. Штайн. *Вступ до алгоритмів*, розділи 12 (BST), 13 (червоно-чорні дерева), 18 (B-дерева).
- Р. Седжвік, К. Вейн. *Алгоритми*, розділ 3.2–3.3 (BST, збалансовані дерева).
- Д. Кнут. *Мистецтво програмування*, т. 3, розділ 6.2.3 (AVL-дерева, дерева Фібоначчі).
- Документація .NET: [`SortedSet<T>`](https://learn.microsoft.com/dotnet/api/system.collections.generic.sortedset-1), [`SortedDictionary<TKey,TValue>`](https://learn.microsoft.com/dotnet/api/system.collections.generic.sorteddictionary-2).
