# Lab 9 - Збалансовані дерева пошуку

Цей лабораторний практикум присвячений вивченню та реалізації збалансованих дерев пошуку: AVL-дерев. Ви навчитесь створювати ці структури даних з нуля та перевіряти їх властивості збалансованості.

---

## Завдання 1: AVL-дерево з перевіркою збалансованості

### Мета

Реалізувати AVL-дерево від основи з автоматичним балансуванням при вставці елементів. Створити функцію для перевірки дотримання AVL-властивості (різниця висот лівого та правого піддерев не перевищує 1 для кожного вузла).

### Вимоги

- Створіть клас `AvlTree` та клас вузла `Node`:
  - Кожен вузол містить ключ (int), посилання `Node?` на лівого та правого нащадків (пам'ять звільняє GC), висоту
- Реалізуйте метод `Insert(int key)` з автоматичним балансуванням:
  - Використовуйте ліві та праві обертання (rotations)
  - Оновлюйте висоту вузлів після вставки
  - Застосовуйте обертання для підтримки AVL-властивості
- Реалізуйте метод `bool IsBalanced()` для перевірки збалансованості:
  - Для кожного вузла: |height(лівий) - height(правий)| ≤ 1
  - Повертає `true`, якщо дерево збалансоване, інакше `false`
- Реалізуйте властивість `int Height` для обчислення висоти дерева
- Реалізуйте метод `void PrintTree()` для візуалізації структури дерева
- **Бали:** 10 балів
- **Складність:**
  - Вставка: O(log n) завдяки збалансованості
  - Перевірка збалансованості: O(n), де n - кількість вузлів
  - Обчислення висоти: O(n)

### Приклад `Program.cs`

```csharp
// Program.cs (.NET 8+, <Nullable>enable</Nullable>)
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

Console.WriteLine();
Console.WriteLine("=== ФІНАЛЬНА ПЕРЕВІРКА ===");
Console.WriteLine($"Загальна висота: {tree.Height}");
Console.WriteLine($"Дерево збалансоване: {(tree.IsBalanced() ? "Так" : "Ні")}");

public sealed class Node(int key)
{
    public int Key { get; } = key;
    public Node? Left { get; set; }
    public Node? Right { get; set; }
    public int Height { get; set; } = 1;
}

public sealed class AvlTree
{
    private Node? _root;

    public int Height => GetHeight(_root);

    public void Insert(int key) => _root = InsertHelper(_root, key);

    public bool IsBalanced() => IsBalancedHelper(_root, out _);

    public void PrintTree()
    {
        // Ваш код тут
    }

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

    // Вивід дерева
    private static void PrintHelper(Node? node, string indent, bool last)
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
        ┌── 50
    ┌── 40
30
        ┌── 25
    └── 20
        └── 10

=== ФІНАЛЬНА ПЕРЕВІРКА ===
Загальна висота: 3
Дерево збалансоване: Так
```
---
