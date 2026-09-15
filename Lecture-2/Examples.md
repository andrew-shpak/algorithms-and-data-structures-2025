# Лекція 2 — Масиви, стек і черга

---

## Зміст

1. [Динамічний масив: як він росте](#1-динамічний-масив-як-він-росте)
2. [Коли використовувати List\<T\>](#2-коли-використовувати-listt)
3. [LinkedList, Stack, Queue](#3-linkedlist-stack-queue)
4. [Приклади коду](#4-приклади-коду)
5. [Підсумки](#5-підсумки)
6. [Питання для самоперевірки](#6-питання-для-самоперевірки)

---

## 1. Динамічний масив: як він росте

Звичайний масив `T[]` має **фіксований розмір** і лежить у пам'яті одним неперервним блоком — тому доступ `a[i]` займає O(1).
Динамічний масив (`List<T>` у .NET, `std::vector` у C++) — це обгортка над таким масивом із двома числами:

- **`Count`** — скільки елементів реально збережено;
- **`Capacity`** — скільки місця виділено.

Коли `Count == Capacity`, виділяється **новий масив удвічі більший**, старі елементи копіюються (O(n)), і лише потім додається новий.
Копіювання трапляється рідко (на розмірах 1, 2, 4, 8, ...), тому сумарно n додавань коштують O(n), а один `Add` — **амортизовано O(1)**.

---

## 2. Коли використовувати List\<T\>

| Операція | `List<T>` | `LinkedList<T>` | `Stack<T>` / `Queue<T>` |
|----------|-----------|-----------------|-------------------------|
| Доступ за індексом | O(1) | O(n) | — |
| Додавання в кінець | O(1) амортизовано | O(1) | `Push`/`Enqueue` O(1) амортизовано |
| Вставка/видалення на початку чи в середині | O(n) (зсув) | O(1) за готовим вузлом | — |
| Пошук значення | O(n), `BinarySearch` O(log n) на відсортованому | O(n) | — |

**Використовуйте `List<T>`, коли:** потрібен доступ за індексом, більшість операцій — додавання в кінець, потрібні `Sort`/`BinarySearch`, важлива ефективність кешу (елементи поруч у пам'яті).

**Не варто, коли:** часті вставки/видалення на початку або в середині (O(n)); дуже великі обсяги, де копіювання при розширенні дороге (задайте `Capacity` наперед або оберіть іншу структуру).

---

## 3. LinkedList, Stack, Queue

- **`LinkedList<T>`** — двозв'язний список. Вставка `AddAfter`/`AddBefore` і `Remove(node)` за O(1), але немає індексатора.
  У C++ `std::list::splice` переносить діапазон вузлів між списками за O(1). У .NET **аналога немає**: вузол знає свій список (`node.List`), тому переносимо вузли по одному — `RemoveFirst` + `AddLast(node)`, без копіювання значень, але за O(k).
- **`Stack<T>`** (LIFO — останній увійшов, перший вийшов): баланс дужок, Undo/Redo, DFS без рекурсії, обчислення RPN, перевертання послідовності.
- **`Queue<T>`** (FIFO — перший увійшов, перший вийшов): BFS, черга клієнтів/завдань, планувальники, черга друку. Реалізована як кільцевий буфер.
- Двобічної черги (deque) у стандартній бібліотеці немає — її роль виконує `LinkedList<T>` (`AddFirst`/`AddLast`/`RemoveFirst`/`RemoveLast`).

Якщо потрібен довільний доступ або перебір з індексами — стек і черга не підходять, беріть `List<T>`.

---

## 4. Приклади коду

Кожен приклад — окрема консольна програма (.NET 8+, C# 12). Запуск: `dotnet new console`, вставити код у `Program.cs`, `dotnet run`.

### Приклад 1 — власний `DynamicArray<T>` з подвоєнням місткості

```csharp
var numbers = new DynamicArray<int>();   // старт: Capacity = 1, Count = 0
for (int i = 1; i <= 5; i++)
{
    numbers.Add(i * 10);                 // коли місця немає — масив подвоюється
    Console.WriteLine($"Add({i * 10}): Count={numbers.Count} Capacity={numbers.Capacity}");
}
Console.WriteLine($"numbers[3]={numbers[3]}"); // доступ за індексом — O(1)

// Мінімальний динамічний масив: неперервний буфер + лічильник елементів
public sealed class DynamicArray<T>
{
    private T[] _items = new T[1];       // внутрішній буфер фіксованого розміру
    public int Count { get; private set; } // скільки елементів реально зайнято
    public int Capacity => _items.Length;  // скільки місця виділено

    public void Add(T item)
    {
        if (Count == Capacity)           // буфер заповнений
        {
            var bigger = new T[Capacity * 2];         // виділяємо вдвічі більший масив
            Array.Copy(_items, bigger, Count);        // копіюємо старі елементи — O(n), але рідко
            _items = bigger;                          // старий масив забере збирач сміття
        }
        _items[Count++] = item;          // звичайний запис у кінець — O(1)
    }

    public T this[int index]
    {
        get
        {
            // перевіряємо межі саме за Count, а не за Capacity
            ArgumentOutOfRangeException.ThrowIfGreaterThanOrEqual((uint)index, (uint)Count, nameof(index));
            return _items[index];
        }
    }
}
```

### Приклад 2 — `List<T>`: сортування та бінарний пошук

```csharp
List<int> scores = [42, 7, 19, 88, 3, 19];   // collection expression (C# 12)
scores.Sort();                                // сортування на місці, O(n log n)
Console.WriteLine(string.Join(' ', scores));

// BinarySearch працює лише на ВІДСОРТОВАНОМУ списку, O(log n)
int found = scores.BinarySearch(42);
Console.WriteLine($"index of 42 = {found}");
int missing = scores.BinarySearch(20);        // немає → від'ємне число
Console.WriteLine($"20 missing, insert at {~missing}"); // ~ дає позицію для вставки
scores.Insert(~missing, 20);                  // вставка в середину — O(n) через зсув
Console.WriteLine(string.Join(' ', scores));
```

### Приклад 3 — матриця: `int[,]` та jagged `int[][]`

```csharp
// Прямокутна матриця: один неперервний блок пам'яті rows × cols
int[,] grid = new int[3, 4];
for (int r = 0; r < grid.GetLength(0); r++)      // GetLength(0) — кількість рядків
    for (int c = 0; c < grid.GetLength(1); c++)  // GetLength(1) — кількість стовпців
        grid[r, c] = r * 10 + c;
Console.WriteLine($"grid[2,3]={grid[2, 3]} total={grid.Length}");

// Jagged-масив: масив масивів, рядки можуть мати різну довжину
int[][] triangle = new int[4][];
for (int r = 0; r < triangle.Length; r++)
{
    triangle[r] = new int[r + 1];                // кожен рядок — окремий масив у купі
    triangle[r][0] = triangle[r][r] = 1;         // краї трикутника Паскаля
    for (int c = 1; c < r; c++)
        triangle[r][c] = triangle[r - 1][c - 1] + triangle[r - 1][c];
}
foreach (int[] row in triangle)
    Console.WriteLine(string.Join(' ', row));
```

### Приклад 4 — список користувачів і буфер під файл

```csharp
List<User> users =
[
    new("olena", 31),
    new("taras", 24),
    new("ivan", 28),
];
users.Add(new User("maria", 24));                // додавання в кінець — амортизовано O(1)

// Сортуємо за віком, а при рівному віці — за іменем
users.Sort((a, b) => a.Age != b.Age ? a.Age.CompareTo(b.Age) : string.CompareOrdinal(a.Name, b.Name));
foreach (User u in users)
    Console.WriteLine($"{u.Name} ({u.Age})");

// Лінійний пошук за умовою — O(n)
User? firstAdult = users.Find(u => u.Age > 30);
Console.WriteLine($"older than 30: {firstAdult?.Name}");

// Буфер під файл: весь вміст файлу — один масив байтів
string path = Path.Combine(Path.GetTempPath(), "lecture2.bin");
File.WriteAllBytes(path, [0x48, 0x69, 0x21]);   // записуємо 3 байти ("Hi!")
byte[] buffer = File.ReadAllBytes(path);         // читаємо назад одним викликом
Console.WriteLine($"{buffer.Length} bytes: {Convert.ToHexString(buffer)} = {System.Text.Encoding.ASCII.GetString(buffer)}");
File.Delete(path);

// record — незмінний тип із автоматичними Equals/ToString
public sealed record User(string Name, int Age);
```

### Приклад 5 — `LinkedList<T>`: `AddAfter`, `Remove` і перенесення вузлів

```csharp
var tasks = new LinkedList<string>(["read", "code", "test"]);
LinkedListNode<string> code = tasks.Find("code")!; // пошук вузла — O(n)
tasks.AddAfter(code, "review");                    // вставка за вузлом — O(1), без зсуву
tasks.AddFirst("plan");                            // початок — O(1)
tasks.Remove(tasks.First!.Next!);                  // видалення вузла "read" — O(1)
Console.WriteLine(string.Join(" -> ", tasks));

// Аналога splice немає: вузол належить одному списку, тож переносимо по одному
var done = new LinkedList<string>();
while (tasks.First is { } node && node.Value != "test")
{
    tasks.RemoveFirst();       // від'єднуємо вузол від старого списку
    done.AddLast(node);        // той самий об'єкт вузла — без копіювання значення
}
Console.WriteLine($"done: {string.Join(", ", done)} | left: {string.Join(", ", tasks)}");
```

### Приклад 6 — `Stack<T>`: баланс дужок і обчислення RPN

```csharp
Console.WriteLine($"{{[()]}} balanced? {IsBalanced("{[()]}")}");
Console.WriteLine($"([)] balanced? {IsBalanced("([)]")}");
Console.WriteLine($"RPN '3 4 + 2 *' = {EvalRpn("3 4 + 2 *")}");

// Баланс дужок: відкриваюча — push, закриваюча — має збігтися з вершиною
static bool IsBalanced(string text)
{
    var stack = new Stack<char>();
    foreach (char ch in text)
    {
        switch (ch)
        {
            case '(' or '[' or '{':
                stack.Push(ch);                    // O(1)
                break;
            case ')' or ']' or '}':
                char open = ch switch { ')' => '(', ']' => '[', _ => '{' };
                if (!stack.TryPop(out char top) || top != open)
                    return false;                  // порожній стек або чужа дужка
                break;
        }
    }
    return stack.Count == 0;                       // залишились незакриті — дисбаланс
}

// Постфіксний вираз: числа — у стек, оператор забирає два верхні
static int EvalRpn(string expression)
{
    var stack = new Stack<int>();
    foreach (string token in expression.Split(' '))
    {
        if (int.TryParse(token, out int value)) { stack.Push(value); continue; }
        int right = stack.Pop(), left = stack.Pop(); // порядок важливий для - та /
        stack.Push(token switch
        {
            "+" => left + right,
            "-" => left - right,
            "*" => left * right,
            _ => left / right,
        });
    }
    return stack.Pop();
}
```

### Приклад 7 — `Queue<T>`: черга завдань і BFS

```csharp
// Черга завдань: обробляємо у порядку надходження
var jobs = new Queue<string>();
jobs.Enqueue("print report");   // у кінець — O(1)
jobs.Enqueue("send email");
jobs.Enqueue("backup");
Console.WriteLine($"next: {jobs.Peek()}, total: {jobs.Count}");
while (jobs.TryDequeue(out string? job)) // з початку — O(1)
    Console.WriteLine($"done: {job}");

// BFS: сусіди вершини ставляться в чергу → обхід «шарами»
int[][] graph = [[1, 2], [3], [3, 4], [5], [5], []];
var distance = new int[graph.Length];
Array.Fill(distance, -1);                  // -1 = ще не відвідана
var queue = new Queue<int>([0]);
distance[0] = 0;
while (queue.Count > 0)
{
    int v = queue.Dequeue();
    foreach (int next in graph[v])
    {
        if (distance[next] != -1) continue; // вже бачили
        distance[next] = distance[v] + 1;
        queue.Enqueue(next);
    }
}
Console.WriteLine($"distances: {string.Join(' ', distance)}");
```

### Приклад запуску

```
# Приклад 1
Add(10): Count=1 Capacity=1
Add(20): Count=2 Capacity=2
Add(30): Count=3 Capacity=4
Add(40): Count=4 Capacity=4
Add(50): Count=5 Capacity=8
numbers[3]=40
# Приклад 2
3 7 19 19 42 88
index of 42 = 4
20 missing, insert at 4
3 7 19 19 20 42 88
# Приклад 3
grid[2,3]=23 total=12
1
1 1
1 2 1
1 3 3 1
# Приклад 4
maria (24)
taras (24)
ivan (28)
olena (31)
older than 30: olena
3 bytes: 486921 = Hi!
# Приклад 5
plan -> code -> review -> test
done: plan, code, review | left: test
# Приклад 6
{[()]} balanced? True
([)] balanced? False
RPN '3 4 + 2 *' = 14
# Приклад 7
next: print report, total: 3
done: print report
done: send email
done: backup
distances: 0 1 1 2 2 3
```

---

## 5. Підсумки

- Динамічний масив подвоює місткість при заповненні, тому `Add` — амортизовано O(1), а доступ за індексом — O(1).
- `List<T>` — вибір за замовчуванням: кеш-дружній, має `Sort`, `BinarySearch`, `Find`; слабке місце — вставки в середину O(n).
- `int[,]` — один неперервний блок; `int[][]` — масив окремих рядків різної довжини.
- `LinkedList<T>` дає O(1) вставку/видалення за вузлом, але не має `splice` та індексатора.
- `Stack<T>` — LIFO (дужки, RPN, DFS), `Queue<T>` — FIFO (BFS, черги завдань); основні операції O(1).

---

## 6. Питання для самоперевірки

1. Чому при подвоєнні місткості n додавань коштують O(n) сумарно, а при збільшенні на сталу величину — O(n²)?
2. Що повертає `List<T>.BinarySearch`, якщо елемента немає, і як отримати з цього позицію для вставки?
3. Чим `int[,]` відрізняється від `int[][]` за будовою в пам'яті та гнучкістю?
4. Чому в .NET немає аналога `std::list::splice` і як перенести вузли з одного `LinkedList<T>` в інший?
5. Яку структуру — стек чи чергу — використовують для BFS і для DFS без рекурсії, і чому?
