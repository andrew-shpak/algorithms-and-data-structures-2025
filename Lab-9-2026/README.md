# Lab-9-2026 — Збалансовані дерева пошуку (AVL)

## [IDE](https://onecompiler.com/csharp)

---

## Завдання 1: AVL-дерево з перевіркою збалансованості

### Мета

Реалізувати AVL-дерево з автоматичним балансуванням при вставці та видаленні елементів. Створити функцію перевірки AVL-властивості (різниця висот лівого та правого піддерев не перевищує 1 для кожного вузла).

- **Складність:**
  - Вставка: O(log n) завдяки збалансованості
  - Видалення: O(log n) з перебалансуванням
  - Перевірка збалансованості: O(n)

### Підказка

AVL-балансування потребує чотирьох типів обертань:
- **Ліве обертання** — коли правий нащадок важчий (Right-Right випадок)
- **Праве обертання** — коли лівий нащадок важчий (Left-Left випадок)
- **Ліве-праве обертання** — спочатку ліве обертання лівого нащадка, потім праве обертання вузла (Left-Right випадок)
- **Право-ліве обертання** — спочатку праве обертання правого нащадка, потім ліве обертання вузла (Right-Left випадок)

Баланс-фактор вузла = висота(лівий) - висота(правий). Якщо |баланс-фактор| > 1, потрібне обертання.

Видалення з AVL = BST-видалення + балансування на зворотному шляху рекурсії. Три випадки видалення: листок, один нащадок, два нащадки (заміна на inorder-наступника). Після видалення може знадобитися балансування кількох вузлів на шляху до кореня.

### Приклад `main`

```csharp
Console.OutputEncoding = System.Text.Encoding.UTF8;

var tree = new AvlTree();

Console.WriteLine("=== ВСТАВКА ЕЛЕМЕНТІВ ===");
int[] values = [10, 20, 30, 40, 50, 25];

foreach (int val in values)
{
    tree.Insert(val);
    Console.WriteLine($"Вставлено: {val}");
    Console.WriteLine($"Висота дерева: {tree.Height}");
    Console.WriteLine($"Збалансоване: {(tree.IsBalanced() ? "Так" : "Ні")}");
    Console.WriteLine();
}

Console.WriteLine("=== СТРУКТУРА ДЕРЕВА ===");
tree.PrintTree();

Console.WriteLine("\n=== ФІНАЛЬНА ПЕРЕВІРКА ===");
Console.WriteLine($"Загальна висота: {tree.Height}");
Console.WriteLine($"Дерево збалансоване: {(tree.IsBalanced() ? "Так" : "Ні")}");

Console.WriteLine("\n=== ВИДАЛЕННЯ ЕЛЕМЕНТІВ ===");
Console.Write("Inorder до видалень: ");
tree.Inorder();
Console.WriteLine();

// Видалення листка
tree.Remove(50);
Console.Write("Після видалення 50 (листок): ");
tree.Inorder();
Console.WriteLine($"| Збалансоване: {(tree.IsBalanced() ? "Так" : "Ні")}");

// Видалення вузла з одним нащадком
tree.Remove(40);
Console.Write("Після видалення 40 (один нащадок): ");
tree.Inorder();
Console.WriteLine($"| Збалансоване: {(tree.IsBalanced() ? "Так" : "Ні")}");

// Видалення вузла з двома нащадками
tree.Remove(25);
Console.Write("Після видалення 25 (два нащадки): ");
tree.Inorder();
Console.WriteLine($"| Збалансоване: {(tree.IsBalanced() ? "Так" : "Ні")}");

public class Node(int key)
{
    public int Key { get; set; } = key;
    public Node? Left { get; set; }
    public Node? Right { get; set; }
    public int Height { get; set; } = 1;
}

public class AvlTree
{
    private Node? _root;

    // Отримати висоту вузла
    private static int GetHeight(Node? node)
    {
        // Ваш код тут
    }

    // Обчислити баланс-фактор вузла
    private static int GetBalance(Node? node)
    {
        // Ваш код тут
    }

    // Праве обертання
    private static Node RotateRight(Node y)
    {
        // Ваш код тут
    }

    // Ліве обертання
    private static Node RotateLeft(Node x)
    {
        // Ваш код тут
    }

    // Вставка з балансуванням
    private static Node InsertHelper(Node? node, int key)
    {
        // Ваш код тут
    }

    // Перевірка збалансованості
    private static bool IsBalancedHelper(Node? node, out int height)
    {
        // Ваш код тут
    }

    // Знайти вузол з мінімальним ключем
    private static Node FindMin(Node node)
    {
        // Ваш код тут
    }

    // Видалення з перебалансуванням
    private static Node? DeleteHelper(Node? node, int key)
    {
        // Ваш код тут
    }

    // Інфіксний обхід
    private static void InorderHelper(Node? node)
    {
        if (node is null) return;
        InorderHelper(node.Left);
        Console.Write($"{node.Key} ");
        InorderHelper(node.Right);
    }

    // Вивід дерева
    private static void PrintHelper(Node? node, string indent, bool last)
    {
        // Ваш код тут
    }

    public void Insert(int key) => _root = InsertHelper(_root, key);

    public void Remove(int key) => _root = DeleteHelper(_root, key);

    public void Inorder() => InorderHelper(_root);

    public bool IsBalanced() => IsBalancedHelper(_root, out _);

    public int Height => GetHeight(_root);

    public void PrintTree()
    {
        // Ваш код тут
    }
}
```

### Приклад запуску

```text
=== ВСТАВКА ЕЛЕМЕНТІВ ===
Вставлено: 10
Висота дерева: 1
Збалансоване: Так

Вставлено: 20
Висота дерева: 2
Збалансоване: Так

Вставлено: 30
Висота дерева: 2
Збалансоване: Так

Вставлено: 40
Висота дерева: 3
Збалансоване: Так

Вставлено: 50
Висота дерева: 3
Збалансоване: Так

Вставлено: 25
Висота дерева: 3
Збалансоване: Так

=== СТРУКТУРА ДЕРЕВА ===
│   ┌── 50
┌── 40
30
    ┌── 25
└── 20
    └── 10

=== ФІНАЛЬНА ПЕРЕВІРКА ===
Загальна висота: 3
Дерево збалансоване: Так

=== ВИДАЛЕННЯ ЕЛЕМЕНТІВ ===
Inorder до видалень: 10 20 25 30 40 50
Після видалення 50 (листок): 10 20 25 30 40 | Збалансоване: Так
Після видалення 40 (один нащадок): 10 20 25 30 | Збалансоване: Так
Після видалення 25 (два нащадки): 10 20 30 | Збалансоване: Так
```

---

## Завдання 2: K-й найменший елемент (Order Statistics Tree)

### Мета

Розширити AVL-дерево з Завдання 1, додавши до кожного вузла поле `Size` — кількість вузлів у його піддереві (включаючи сам вузол). Використовуючи це поле, реалізувати ефективний пошук k-го найменшого елемента за O(log n).

- **Складність:** O(log n) для пошуку k-го елемента

### Підказка

Для кожного вузла `Size = 1 + Size(лівий) + Size(правий)`. Щоб знайти k-й найменший:
- Нехай `leftSize` = кількість вузлів у лівому піддереві
- Якщо `k == leftSize + 1` — поточний вузол і є відповідь
- Якщо `k <= leftSize` — шукаємо в лівому піддереві
- Якщо `k > leftSize + 1` — шукаємо в правому піддереві з `k = k - leftSize - 1`

Не забудьте оновлювати `Size` після вставки та обертань.

### Приклад `main`

```csharp
var tree = new OrderStatTree();

Console.WriteLine("=== ПОБУДОВА ДЕРЕВА ===");
foreach (int v in new[] { 50, 30, 70, 20, 40, 60, 80, 10, 25, 35 })
    tree.Insert(v);

Console.WriteLine($"Кількість елементів: {tree.Size}");
// Відсортований порядок: 10 20 25 30 35 40 50 60 70 80

Console.WriteLine("\n=== ПОШУК K-ГО НАЙМЕНШОГО ===");
for (int k = 1; k <= tree.Size; k++)
{
    Console.WriteLine($"{k}-й найменший: {tree.KthSmallest(k)}");
}

Console.WriteLine("\n=== ГРАНИЧНІ ВИПАДКИ ===");
tree.KthSmallest(0);   // поза межами
tree.KthSmallest(11);  // поза межами

public class Node(int key)
{
    public int Key { get; set; } = key;
    public Node? Left { get; set; }
    public Node? Right { get; set; }
    public int Height { get; set; } = 1;
    public int Size { get; set; } = 1;  // кількість вузлів у піддереві
}

public class OrderStatTree
{
    private Node? _root;

    private static int GetHeight(Node? node) => node?.Height ?? 0;

    private static int GetSize(Node? node) => node?.Size ?? 0;

    // Оновити висоту та розмір вузла
    private static void Update(Node node)
    {
        // Ваш код тут
    }

    private static int GetBalance(Node? node) =>
        node is null ? 0 : GetHeight(node.Left) - GetHeight(node.Right);

    // Праве обертання (не забудьте оновити Size!)
    private static Node RotateRight(Node y)
    {
        // Ваш код тут
    }

    // Ліве обертання (не забудьте оновити Size!)
    private static Node RotateLeft(Node x)
    {
        // Ваш код тут
    }

    private static Node InsertHelper(Node? node, int key)
    {
        // Ваш код тут (аналогічно до Завдання 1, але з оновленням Size)
    }

    // Пошук k-го найменшого елемента
    private static int KthSmallestHelper(Node node, int k)
    {
        // Ваш код тут
    }

    public void Insert(int key) => _root = InsertHelper(_root, key);

    // Повертає k-й найменший елемент (1-індексація)
    public int KthSmallest(int k)
    {
        if (_root is null || k < 1 || k > GetSize(_root))
        {
            Console.WriteLine($"k = {k} поза межами [1, {GetSize(_root)}]");
            return -1;
        }
        return KthSmallestHelper(_root, k);
    }

    public int Size => GetSize(_root);
}
```

### Приклад запуску

```text
=== ПОБУДОВА ДЕРЕВА ===
Кількість елементів: 10

=== ПОШУК K-ГО НАЙМЕНШОГО ===
1-й найменший: 10
2-й найменший: 20
3-й найменший: 25
4-й найменший: 30
5-й найменший: 35
6-й найменший: 40
7-й найменший: 50
8-й найменший: 60
9-й найменший: 70
10-й найменший: 80

=== ГРАНИЧНІ ВИПАДКИ ===
k = 0 поза межами [1, 10]
k = 11 поза межами [1, 10]
```

---

## Завдання

| # | Тема |
|---|------|
| Завдання 1 | AVL-дерево з перевіркою збалансованості |
| Завдання 2 | K-й найменший елемент (Order Statistics Tree) |
