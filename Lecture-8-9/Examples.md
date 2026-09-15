# Лекція 8-9 — Бінарні дерева, BST та AVL-дерева

---

## Зміст

1. [Бінарне дерево: вузол і термінологія](#1-бінарне-дерево-вузол-і-термінологія)
2. [Обходи дерева](#2-обходи-дерева)
3. [Висота та кількість вузлів](#3-висота-та-кількість-вузлів)
4. [Бінарне дерево пошуку (BST)](#4-бінарне-дерево-пошуку-bst)
5. [AVL-дерева](#5-avl-дерева)
6. [Підсумки](#6-підсумки)
7. [Питання для самоперевірки](#7-питання-для-самоперевірки)

---

## 1. Бінарне дерево: вузол і термінологія

**Бінарне дерево** — це структура, де кожен вузол має не більше двох дітей: **лівого** та **правого**.

| Термін | Значення |
|--------|----------|
| Корінь (root) | Верхній вузол, не має батька |
| Лист (leaf) | Вузол без дітей |
| Висота (height) | Кількість вузлів на найдовшому шляху від вузла до листа |
| Глибина (depth) | Відстань від кореня до вузла |

У C# вузол — це клас із nullable-посиланнями на дітей (`Node?`). Звільняти пам'ять вручну не потрібно: коли на вузол не лишається посилань, його прибере **збирач сміття (GC)**.

### Приклад 1. Структура вузла

```csharp
//        1
//       / \
//      2   3
var root = new Node(1) { Left = new Node(2), Right = new Node(3) };
Console.WriteLine($"{root.Key} {root.Left.Key} {root.Right.Key}"); // delete не потрібен: пам'ять звільнить GC

// Вузол бінарного дерева: значення + два посилання на дітей
class Node(int key)
{
    public int Key { get; set; } = key; // значення, що зберігається у вузлі
    public Node? Left { get; set; }     // лівий нащадок (null, якщо немає)
    public Node? Right { get; set; }    // правий нащадок (null, якщо немає)
}
```

**Приклад запуску:**
```
1 2 3
```

---

## 2. Обходи дерева

- **Pre-order** (прямий): вузол → ліве → праве. Зручно для копіювання дерева.
- **In-order** (симетричний): ліве → вузол → праве. Для BST дає **відсортовану** послідовність.
- **Post-order** (зворотний): ліве → праве → вузол. Зручно для видалення дерева.
- **Level-order** (по рівнях): BFS із чергою `Queue<T>`.

Усі обходи працюють за **O(n)**.

### Приклад 2. Рекурсивні обходи (DFS)

```csharp
//        4
//       / \
//      2   6
//     / \
//    1   3
var root = new Node(4) { Left = new Node(2) { Left = new Node(1), Right = new Node(3) }, Right = new Node(6) };

Console.Write("pre:  "); PreOrder(root);  Console.WriteLine();
Console.Write("in:   "); InOrder(root);   Console.WriteLine();
Console.Write("post: "); PostOrder(root); Console.WriteLine();

// Прямий обхід: спочатку обробляємо вузол, потім піддерева
static void PreOrder(Node? n)
{
    if (n is null) return;       // базовий випадок: порожнє піддерево
    Console.Write($"{n.Key} ");
    PreOrder(n.Left);
    PreOrder(n.Right);
}

// Симетричний обхід: ліве піддерево, вузол, праве піддерево
static void InOrder(Node? n)
{
    if (n is null) return;
    InOrder(n.Left);
    Console.Write($"{n.Key} ");
    InOrder(n.Right);
}

// Зворотний обхід: вузол обробляється останнім
static void PostOrder(Node? n)
{
    if (n is null) return;
    PostOrder(n.Left);
    PostOrder(n.Right);
    Console.Write($"{n.Key} ");
}

// Той самий вузол, що й у прикладі 1
class Node(int key) { public int Key { get; set; } = key; public Node? Left { get; set; } public Node? Right { get; set; } }
```

**Приклад запуску:**
```
pre:  4 2 1 3 6 
in:   1 2 3 4 6 
post: 1 3 2 6 4 
```

### Приклад 3. Обхід по рівнях (BFS)

```csharp
var root = new Node(4) { Left = new Node(2) { Left = new Node(1), Right = new Node(3) }, Right = new Node(6) };
LevelOrder(root);

// Обхід по рівнях: черга гарантує порядок "зверху вниз, зліва направо"
static void LevelOrder(Node? root)
{
    if (root is null) return;
    var queue = new Queue<Node>();
    queue.Enqueue(root);
    while (queue.Count > 0)
    {
        int levelSize = queue.Count; // скільки вузлів на поточному рівні
        for (int i = 0; i < levelSize; i++)
        {
            Node n = queue.Dequeue();
            Console.Write($"{n.Key} ");
            // додаємо дітей у чергу — вони будуть оброблені на наступному рівні
            if (n.Left is not null) queue.Enqueue(n.Left);
            if (n.Right is not null) queue.Enqueue(n.Right);
        }
        Console.WriteLine(); // кінець рівня
    }
}

// Той самий вузол, що й у прикладі 1
class Node(int key) { public int Key { get; set; } = key; public Node? Left { get; set; } public Node? Right { get; set; } }
```

**Приклад запуску:**
```
4 
2 6 
1 3 
```

---

## 3. Висота та кількість вузлів

Обидві функції — природна рекурсія: відповідь для вузла будується з відповідей для піддерев.

### Приклад 4. Висота та кількість вузлів

```csharp
var root = new Node(4) { Left = new Node(2) { Left = new Node(1) }, Right = new Node(6) };
Console.WriteLine($"height = {Height(root)}");
Console.WriteLine($"count  = {CountNodes(root)}");

// Висота: порожнє дерево має висоту 0, лист — 1
static int Height(Node? n) =>
    n is null ? 0 : 1 + Math.Max(Height(n.Left), Height(n.Right));

// Кількість вузлів: 1 (сам вузол) + вузли в обох піддеревах
static int CountNodes(Node? n) =>
    n is null ? 0 : 1 + CountNodes(n.Left) + CountNodes(n.Right);

// Той самий вузол, що й у прикладі 1
class Node(int key) { public int Key { get; set; } = key; public Node? Left { get; set; } public Node? Right { get; set; } }
```

**Приклад запуску:**
```
height = 3
count  = 4
```

---

## 4. Бінарне дерево пошуку (BST)

**Властивість BST:** для кожного вузла всі ключі лівого піддерева **менші**, а правого — **більші**.

| Операція | Середній випадок | Найгірший випадок |
|----------|------------------|-------------------|
| Пошук | O(log n) | O(n) |
| Вставка | O(log n) | O(n) |
| Видалення | O(log n) | O(n) |

Найгірший випадок — вставка відсортованих даних: дерево вироджується в "список".

**Видалення** має три випадки:
1. Вузол — лист: просто видаляємо.
2. Один нащадок: замінюємо вузол нащадком.
3. Два нащадки: копіюємо ключ **наступника** (мінімум правого піддерева) і видаляємо наступника.

### Приклад 5. BST: вставка, пошук, видалення

```csharp
Node? root = null;
foreach (int x in new[] { 50, 30, 70, 20, 40, 60, 80 }) root = Insert(root, x);

Console.Write("inorder: "); InOrder(root); Console.WriteLine();
Console.WriteLine($"contains 40? {Contains(root, 40)}");
Console.WriteLine($"contains 45? {Contains(root, 45)}");

root = Erase(root, 30); // вузол з двома нащадками
root = Erase(root, 80); // лист
Console.Write("after erase: "); InOrder(root); Console.WriteLine();

// Вставка: йдемо вліво/вправо, доки не знайдемо порожнє місце
static Node Insert(Node? n, int key)
{
    if (n is null) return new Node(key);            // знайшли місце для нового вузла
    if (key < n.Key) n.Left = Insert(n.Left, key);
    else if (key > n.Key) n.Right = Insert(n.Right, key);
    return n;                                        // дублікати ігноруємо
}

// Пошук: на кожному кроці відкидаємо половину (у збалансованому дереві)
static bool Contains(Node? n, int key)
{
    while (n is not null)
    {
        if (key == n.Key) return true;
        n = key < n.Key ? n.Left : n.Right;
    }
    return false;
}

// Видалення ключа; повертає новий корінь піддерева
static Node? Erase(Node? n, int key)
{
    if (n is null) return null;                      // ключа немає
    if (key < n.Key) n.Left = Erase(n.Left, key);
    else if (key > n.Key) n.Right = Erase(n.Right, key);
    else
    {
        // Випадки 1 і 2: немає лівого або правого нащадка —
        // повертаємо іншого; старий вузол забере GC
        if (n.Left is null) return n.Right;
        if (n.Right is null) return n.Left;
        // Випадок 3: два нащадки — беремо наступника (найлівіший у правому піддереві)
        Node succ = n.Right;
        while (succ.Left is not null) succ = succ.Left;
        n.Key = succ.Key;                            // копіюємо ключ наступника
        n.Right = Erase(n.Right, succ.Key);          // видаляємо наступника
    }
    return n;
}

// Симетричний обхід (див. приклад 2) — для BST друкує ключі за зростанням
static void InOrder(Node? n) { if (n is null) return; InOrder(n.Left); Console.Write($"{n.Key} "); InOrder(n.Right); }

// Той самий вузол, що й у прикладі 1
class Node(int key) { public int Key { get; set; } = key; public Node? Left { get; set; } public Node? Right { get; set; } }
```

**Приклад запуску:**
```
inorder: 20 30 40 50 60 70 80 
contains 40? True
contains 45? False
after erase: 20 40 50 60 70 
```

---

## 5. AVL-дерева

**AVL-дерево** — самобалансоване BST. Для кожного вузла:

```
balance factor = height(left) - height(right),   |bf| <= 1
```

Отже, висота завжди **O(log n)**, і всі операції гарантовано O(log n).

> У .NET готові збалансовані дерева пошуку — `SortedSet<T>` та `SortedDictionary<TKey, TValue>` (усередині — **червоно-чорне дерево**, яке, як і AVL, гарантує O(log n)).

Після вставки перевіряємо баланс і виконуємо **повороти**:

| Випадок | Умова | Дія |
|---------|-------|-----|
| LL | bf > 1, ключ у лівому-лівому | правий поворот |
| RR | bf < -1, ключ у правому-правому | лівий поворот |
| LR | bf > 1, ключ у лівому-правому | лівий поворот лівого сина, потім правий |
| RL | bf < -1, ключ у правому-лівому | правий поворот правого сина, потім лівий |

```
Правий поворот (LL):
        z                y
       / \             /   \
      y   T4    →     x     z
     / \             / \   / \
    x   T3          T1 T2 T3 T4
```

### Приклад 6. AVL: повороти та вставка

```csharp
Node? root = null;
// Відсортовані дані зробили б звичайне BST "списком" висоти 7
for (int x = 1; x <= 7; x++) root = Insert(root, x);
Console.WriteLine($"height = {H(root)}");
Print(root);
Console.WriteLine();
static int H(Node? n) => n?.Height ?? 0;

// Перераховуємо висоту вузла за висотами дітей
static void Update(Node n) => n.Height = 1 + Math.Max(H(n.Left), H(n.Right));

// Balance factor: різниця висот лівого та правого піддерев
static int Balance(Node? n) => n is null ? 0 : H(n.Left) - H(n.Right);

// Правий поворот навколо y: лівий син x стає новим коренем
static Node RotateRight(Node y)
{
    Node x = y.Left!;      // при bf > 1 лівий син гарантовано існує
    y.Left = x.Right;      // праве піддерево x переходить до y
    x.Right = y;           // y стає правим сином x
    Update(y);             // спершу нижній вузол,
    Update(x);             // потім верхній
    return x;
}

// Лівий поворот навколо x: правий син y стає новим коренем
static Node RotateLeft(Node x)
{
    Node y = x.Right!;
    x.Right = y.Left;
    y.Left = x;
    Update(x);
    Update(y);
    return y;
}

static Node Insert(Node? n, int key)
{
    // 1. Звичайна вставка в BST
    if (n is null) return new Node(key);
    if (key < n.Key) n.Left = Insert(n.Left, key);
    else if (key > n.Key) n.Right = Insert(n.Right, key);
    else return n;

    // 2. Оновлюємо висоту на шляху вгору
    Update(n);
    int bf = Balance(n);

    // 3. Відновлюємо баланс одним із чотирьох випадків
    if (bf > 1 && key < n.Left!.Key) return RotateRight(n);          // LL
    if (bf < -1 && key > n.Right!.Key) return RotateLeft(n);         // RR
    if (bf > 1) { n.Left = RotateLeft(n.Left!); return RotateRight(n); }      // LR
    if (bf < -1) { n.Right = RotateRight(n.Right!); return RotateLeft(n); }   // RL
    return n; // вузол збалансований
}

// Друкуємо ключ і balance factor кожного вузла (pre-order)
static void Print(Node? n) { if (n is null) return; Console.Write($"{n.Key}(bf={Balance(n)}) "); Print(n.Left); Print(n.Right); }

class Node(int key)
{
    public int Key { get; } = key;
    public int Height { get; set; } = 1; // висота піддерева, лист має висоту 1
    public Node? Left { get; set; }
    public Node? Right { get; set; }
}
```

**Приклад запуску:**
```
height = 3
4(bf=0) 2(bf=0) 1(bf=0) 3(bf=0) 6(bf=0) 5(bf=0) 7(bf=0) 
```

---

## 6. Підсумки

- Бінарне дерево — вузол із двома посиланнями `Node?`; більшість алгоритмів на ньому рекурсивні, пам'ять звільняє GC.
- Обходи pre/in/post-order — це DFS, level-order — BFS із `Queue<T>`; всі за O(n).
- BST дає O(log n) у середньому, але може виродитися до O(n).
- AVL-дерево підтримує |bf| ≤ 1 за допомогою поворотів і гарантує O(log n); у .NET є готові `SortedSet<T>`/`SortedDictionary<TKey, TValue>` на червоно-чорному дереві.

---

## 7. Питання для самоперевірки

1. Який обхід BST повертає ключі у відсортованому порядку і чому?
2. Чим level-order обхід відрізняється від DFS-обходів і яка структура даних для нього потрібна?
3. Опишіть три випадки видалення вузла з BST.
4. Коли звичайне BST вироджується в список і як AVL-дерево цього уникає?
5. Яку послідовність поворотів виконують у випадку LR і чому одного повороту недостатньо?
