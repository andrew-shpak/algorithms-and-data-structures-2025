# Lab-8-2026 — Бінарні дерева

## [IDE](https://onecompiler.com/csharp)

> Локально: створіть проєкт командою `dotnet new console`, вставте код у `Program.cs` і запустіть `dotnet run`.

---

## Завдання 1: Обходи дерева

### Мета

Реалізувати три основні рекурсивні обходи бінарного дерева пошуку: інфіксний (inorder), префіксний (preorder) та постфіксний (postorder).

- **Бали:** 1
- **Складність:** O(n) для кожного обходу, де n — кількість вузлів

### Приклад `Program.cs`

```csharp
Node? root = null;
foreach (int v in new[] { 50, 30, 70, 20, 40, 60, 80 })
    root = Insert(root, v);

Console.Write("Інфіксний обхід (Inorder): ");
Inorder(root);
Console.WriteLine();

Console.Write("Префіксний обхід (Preorder): ");
Preorder(root);
Console.WriteLine();

Console.Write("Постфіксний обхід (Postorder): ");
Postorder(root);
Console.WriteLine();

static Node Insert(Node? root, int value)
{
    // Ваш код тут
    throw new NotImplementedException();
}

static void Inorder(Node? root)
{
    // Ваш код тут
}

static void Preorder(Node? root)
{
    // Ваш код тут
}

static void Postorder(Node? root)
{
    // Ваш код тут
}

class Node(int data)
{
    public int Data { get; set; } = data;
    public Node? Left { get; set; }
    public Node? Right { get; set; }
}
```

### Приклад запуску

```text
Інфіксний обхід (Inorder): 20 30 40 50 60 70 80
Префіксний обхід (Preorder): 50 30 20 40 70 60 80
Постфіксний обхід (Postorder): 20 40 30 60 80 70 50
```

---

## Завдання 2: Діаметр бінарного дерева

### Мета

Обчислити діаметр бінарного дерева — найдовший шлях між будь-якими двома вузлами дерева, виміряний у кількості ребер. Цей шлях може не проходити через корінь.

- **Бали:** 1
- **Складність:** O(n), де n — кількість вузлів

### Підказка

Для кожного вузла найдовший шлях, що проходить через нього, дорівнює сумі висот його лівого та правого піддерев (+ 2 ребра до кожного нащадка). Діаметр дерева — максимум серед усіх таких шляхів. Задачу можна розв'язати за один прохід, обчислюючи висоту та діаметр одночасно (висоту повертайте через параметр `out`).

### Приклад `Program.cs`

```csharp
// Дерево 1 (збалансоване):
//         50
//        /  \
//      30    70
//     / \   / \
//    20  40 60  80
Node? root1 = null;
foreach (int v in new[] { 50, 30, 70, 20, 40, 60, 80 })
    root1 = Insert(root1, v);

Console.WriteLine("Дерево 1 (збалансоване):");
Console.WriteLine($"Діаметр: {Diameter(root1, out _)}");
Console.WriteLine();

// Дерево 2 (діаметр НЕ проходить через корінь):
//         50
//        /
//      30
//     / \
//    20   40
//   /      \
//  10       45
Node? root2 = null;
foreach (int v in new[] { 50, 30, 20, 10, 40, 45 })
    root2 = Insert(root2, v);

Console.WriteLine("Дерево 2 (діаметр не через корінь):");
Console.WriteLine($"Діаметр: {Diameter(root2, out _)}");

static Node Insert(Node? root, int value)
{
    // Ваш код тут
    throw new NotImplementedException();
}

// Повертає діаметр дерева
// height — вихідний параметр для висоти піддерева
static int Diameter(Node? root, out int height)
{
    // Ваш код тут
    throw new NotImplementedException();
}

class Node(int data)
{
    public int Data { get; set; } = data;
    public Node? Left { get; set; }
    public Node? Right { get; set; }
}
```

### Приклад запуску

```text
Дерево 1 (збалансоване):
Діаметр: 4

Дерево 2 (діаметр не через корінь):
Діаметр: 4
```

---

## Завдання 3: Перевірка коректності BST

### Мета

Реалізувати функцію перевірки, чи є бінарне дерево коректним деревом пошуку (BST). Наївна перевірка (порівняння вузла лише з прямими нащадками) є недостатньою — потрібно використовувати діапазонний підхід із мінімальними та максимальними допустимими значеннями.

- **Бали:** 2
- **Складність:** O(n), де n — кількість вузлів

### Приклад `Program.cs`

```csharp
// Дерево 1: коректне BST
//       50
//      /  \
//    30    70
//   / \   / \
//  20  40 60  80
var t1 = new Node(50)
{
    Left = new Node(30) { Left = new Node(20), Right = new Node(40) },
    Right = new Node(70) { Left = new Node(60), Right = new Node(80) },
};

Console.WriteLine($"Дерево 1: {(IsValidBst(t1) ? "Коректне BST" : "Не BST")}");

// Дерево 2: НЕ коректне BST
//       50
//      /  \
//    30    70
//   / \
//  20  60  <-- 60 > 50, але знаходиться в лівому піддереві кореня
var t2 = new Node(50)
{
    Left = new Node(30) { Left = new Node(20), Right = new Node(60) },
    Right = new Node(70),
};

Console.WriteLine($"Дерево 2: {(IsValidBst(t2) ? "Коректне BST" : "Не BST")}");

// Дерево 3: НЕ коректне BST
//       50
//      /  \
//    30    70
//         / \
//        40  80  <-- 40 < 50, але знаходиться в правому піддереві кореня
var t3 = new Node(50)
{
    Left = new Node(30),
    Right = new Node(70) { Left = new Node(40), Right = new Node(80) },
};

Console.WriteLine($"Дерево 3: {(IsValidBst(t3) ? "Коректне BST" : "Не BST")}");

static bool IsValidBst(Node? root) => IsValidBstInRange(root, long.MinValue, long.MaxValue);

// Перевірка BST з допустимим діапазоном [minValue, maxValue]
static bool IsValidBstInRange(Node? root, long minValue, long maxValue)
{
    // Ваш код тут
    throw new NotImplementedException();
}

class Node(int data)
{
    public int Data { get; set; } = data;
    public Node? Left { get; set; }
    public Node? Right { get; set; }
}
```

### Приклад запуску

```text
Дерево 1: Коректне BST
Дерево 2: Не BST
Дерево 3: Не BST
```

---

## Завдання 4: Видалення вузла з BST

### Мета

Реалізувати операцію видалення вузла з бінарного дерева пошуку з урахуванням трьох випадків:

1. **Вузол-листок** — просто видалити (прибрати посилання; пам'ять звільнить збирач сміття)
2. **Вузол з одним нащадком** — замінити вузол його нащадком
3. **Вузол з двома нащадками** — знайти inorder-наступника (найменший елемент у правому піддереві), скопіювати його значення та видалити наступника

- **Бали:** 2
- **Складність:** O(h), де h — висота дерева

### Приклад `Program.cs`

```csharp
Node? root = null;
foreach (int v in new[] { 50, 30, 70, 20, 40, 60, 80 })
    root = Insert(root, v);

//         50
//        /  \
//      30    70
//     / \   / \
//    20  40 60  80

Console.Write("Початкове дерево: ");
Inorder(root);
Console.WriteLine();

// Випадок 1: видалення листка (20)
root = DeleteNode(root, 20);
Console.Write("Після видалення 20 (листок): ");
Inorder(root);
Console.WriteLine();

// Випадок 2: видалення вузла з одним нащадком (30 → має лише 40)
root = DeleteNode(root, 30);
Console.Write("Після видалення 30 (один нащадок): ");
Inorder(root);
Console.WriteLine();

// Випадок 3: видалення вузла з двома нащадками (50 → наступник 60)
root = DeleteNode(root, 50);
Console.Write("Після видалення 50 (два нащадки): ");
Inorder(root);
Console.WriteLine();

static Node Insert(Node? root, int value)
{
    // Ваш код тут
    throw new NotImplementedException();
}

static void Inorder(Node? root)
{
    // Ваш код тут
}

// Знаходить вузол з мінімальним значенням у дереві
static Node FindMin(Node root)
{
    // Ваш код тут
    throw new NotImplementedException();
}

// Видаляє вузол зі значенням key з дерева
static Node? DeleteNode(Node? root, int key)
{
    // Ваш код тут
    throw new NotImplementedException();
}

class Node(int data)
{
    public int Data { get; set; } = data;
    public Node? Left { get; set; }
    public Node? Right { get; set; }
}
```

### Приклад запуску

```text
Початкове дерево: 20 30 40 50 60 70 80
Після видалення 20 (листок): 30 40 50 60 70 80
Після видалення 30 (один нащадок): 40 50 60 70 80
Після видалення 50 (два нащадки): 40 60 70 80
```

---

## Завдання 5: Серіалізація та десеріалізація BST

### Мета

Реалізувати функції для серіалізації бінарного дерева пошуку у рядок та його відновлення (десеріалізації) з рядка. Використовувати префіксний обхід (preorder) із маркером `#` для позначення відсутніх вузлів.

- **Бали:** 2
- **Складність:** O(n) для обох операцій, де n — кількість вузлів

### Підказка

Серіалізація: обхід дерева у префіксному порядку, записуючи значення вузлів через пробіл (зручно використовувати `StringBuilder` або `List<string>` + `string.Join`). Для `null` записуйте `#`. Десеріалізація: розбийте рядок на токени (`Split`), покладіть їх у `Queue<string>` і рекурсивно будуйте дерево у тому ж порядку, вилучаючи токени по одному через `Dequeue()`.

### Приклад `Program.cs`

```csharp
Node? root = null;
foreach (int v in new[] { 50, 30, 70, 20, 40, 60, 80 })
    root = Insert(root, v);

//         50
//        /  \
//      30    70
//     / \   / \
//    20  40 60  80

Console.Write("Оригінальне дерево (inorder): ");
Inorder(root);
Console.WriteLine();

string data = Serialize(root);
Console.WriteLine($"Серіалізовано: {data}");

Node? restored = Deserialize(data);
Console.Write("Відновлене дерево (inorder): ");
Inorder(restored);
Console.WriteLine();

// Тест з порожнім деревом
string emptyData = Serialize(null);
Console.WriteLine($"Порожнє дерево серіалізовано: \"{emptyData}\"");

Node? emptyRestored = Deserialize(emptyData);
Console.Write("Відновлене порожнє дерево (inorder): ");
Inorder(emptyRestored);
Console.WriteLine("(порожньо)");

static Node Insert(Node? root, int value)
{
    // Ваш код тут
    throw new NotImplementedException();
}

static void Inorder(Node? root)
{
    // Ваш код тут
}

// Серіалізує дерево у рядок (префіксний обхід, "#" для null)
static string Serialize(Node? root)
{
    // Ваш код тут
    throw new NotImplementedException();
}

// Допоміжна функція для десеріалізації
static Node? DeserializeHelper(Queue<string> tokens)
{
    // Ваш код тут
    throw new NotImplementedException();
}

// Відновлює дерево з рядка
static Node? Deserialize(string data)
{
    var tokens = new Queue<string>(data.Split(' ', StringSplitOptions.RemoveEmptyEntries));
    return DeserializeHelper(tokens);
}

class Node(int data)
{
    public int Data { get; set; } = data;
    public Node? Left { get; set; }
    public Node? Right { get; set; }
}
```

### Приклад запуску

```text
Оригінальне дерево (inorder): 20 30 40 50 60 70 80
Серіалізовано: 50 30 20 # # 40 # # 70 60 # # 80 # #
Відновлене дерево (inorder): 20 30 40 50 60 70 80
Порожнє дерево серіалізовано: "#"
Відновлене порожнє дерево (inorder): (порожньо)
```

---

## Сумарні бали

| Завдання | Тема | Бали |
|----------|------|------|
| Завдання 1 | Обходи дерева | 1 |
| Завдання 2 | Діаметр бінарного дерева | 1 |
| Завдання 3 | Перевірка коректності BST | 2 |
| Завдання 4 | Видалення вузла з BST | 2 |
| Завдання 5 | Серіалізація та десеріалізація BST | 2 |

**Всього: 8 балів**
