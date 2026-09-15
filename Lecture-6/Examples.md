# Лекція 6 — Стек та черга

---

## Зміст

1. [LIFO та FIFO](#1-lifo-та-fifo)
2. [Колекції .NET](#2-колекції-net)
3. [Реалізація стека](#3-реалізація-стека)
4. [Застосування стека](#4-застосування-стека)
5. [Застосування черги](#5-застосування-черги)
6. [Підсумки](#6-підсумки)
7. [Питання для самоперевірки](#7-питання-для-самоперевірки)

---

## 1. LIFO та FIFO

**Стек** (stack) — структура даних за принципом **LIFO** (Last In, First Out): останній доданий елемент виймається першим. Аналогія — стос тарілок.

**Черга** (queue) — структура даних за принципом **FIFO** (First In, First Out): першим виймається той, хто прийшов першим. Аналогія — черга в касу.

| Операція | Стек (`Stack<T>`) | Черга (`Queue<T>`) | Складність |
|----------|------|-------|-----------|
| Додати | `Push` (на вершину) | `Enqueue` (у кінець) | O(1)* |
| Видалити | `Pop` (з вершини) | `Dequeue` (з початку) | O(1) |
| Переглянути | `Peek` | `Peek` | O(1) |
| Розмір | `Count` | `Count` | O(1) |

\* амортизовано: внутрішній масив іноді збільшується.

**Дек** (deque, double-ended queue) дозволяє додавати й видаляти з обох кінців за O(1); у .NET цю роль виконує `LinkedList<T>`.
**Черга з пріоритетом** (`PriorityQueue<TElement, TPriority>`) завжди віддає елемент з найменшим пріоритетом; `Enqueue`/`Dequeue` — O(log n).

---

## 2. Колекції .NET

`Stack<T>` та `Queue<T>` з простору імен `System.Collections.Generic` побудовані на масиві (черга — на кільцевому буфері) і відкривають лише потрібні операції.

### Приклад 1. `Stack<T>` та `Queue<T>`

```csharp
var stack = new Stack<int>();  // стек цілих чисел (всередині — масив)
var queue = new Queue<int>();  // черга цілих чисел (кільцевий буфер)

// Додаємо однакові елементи 1, 2, 3 в обидві структури
for (int i = 1; i <= 3; i++)
{
    stack.Push(i);     // кладемо на вершину стека
    queue.Enqueue(i);  // ставимо в кінець черги
}

Console.Write("Stack (LIFO): ");
// TryPop повертає false, коли стек порожній, — без винятку
while (stack.TryPop(out int top))
{
    Console.Write($"{top} ");  // Pop/TryPop видаляє вершину і повертає її
}

Console.Write("\nQueue (FIFO): ");
while (queue.Count > 0)  // поки черга не порожня
{
    Console.Write($"{queue.Dequeue()} ");  // Dequeue — видаляє і повертає перший елемент
}
Console.WriteLine();
```

**Приклад запуску:**
```
Stack (LIFO): 3 2 1
Queue (FIFO): 1 2 3
```

> Увага: `Pop()`/`Peek()`/`Dequeue()` на порожній колекції кидають `InvalidOperationException`. Перевіряйте `Count` або використовуйте `TryPop`/`TryPeek`/`TryDequeue`.

### Приклад 2. `LinkedList<T>` як дек та `PriorityQueue<TElement, TPriority>`

```csharp
// --- LinkedList<T>: вставка/видалення з обох кінців за O(1) ---
var deque = new LinkedList<int>();
deque.AddLast(2);    // [2]
deque.AddLast(3);    // [2, 3]
deque.AddFirst(1);   // [1, 2, 3]
deque.RemoveLast();  // [1, 2]
// First/Last — вузли; індексації немає, доступ до середини — O(n)
Console.WriteLine($"deque: {deque.First!.Value} {deque.Last!.Value}, count = {deque.Count}");

int[] values = [5, 1, 8, 3];

// --- PriorityQueue: за замовчуванням min-heap (менший пріоритет — раніше) ---
var minPq = new PriorityQueue<int, int>();
foreach (int x in values) minPq.Enqueue(x, x);  // елемент і його пріоритет; O(log n)
Console.Write("min-heap: ");
while (minPq.TryDequeue(out int item, out _))
{
    Console.Write($"{item} ");  // щоразу виймається найменший
}

// max-heap: передаємо компаратор, що інвертує порівняння пріоритетів
var maxPq = new PriorityQueue<int, int>(Comparer<int>.Create((a, b) => b.CompareTo(a)));
foreach (int x in values) maxPq.Enqueue(x, x);
Console.Write("\nmax-heap: ");
while (maxPq.Count > 0)
{
    Console.Write($"{maxPq.Dequeue()} ");  // тепер виймається найбільший
}
Console.WriteLine();
```

**Приклад запуску:**
```
deque: 1 2, count = 2
min-heap: 1 3 5 8
max-heap: 8 5 3 1
```

---

## 3. Реалізація стека

Два класичні підходи:

| Підхід | Переваги | Недоліки |
|--------|----------|----------|
| Динамічний масив | Локальність кешу, мало пам'яті на елемент | Амортизоване O(1) через перевиділення |
| Зв'язний список | Строге O(1), без перевиділень | Додаткове посилання та об'єкт на кожен вузол |

### Приклад 3. Стек на масиві

```csharp
var s = new ArrayStack<string>();
s.Push("a");
s.Push("b");
s.Push("c");
Console.WriteLine($"top = {s.Peek()}, size = {s.Count}");
s.Pop();
Console.WriteLine($"after pop: top = {s.Peek()}");

public class ArrayStack<T>
{
    private T[] _items = new T[4];  // внутрішній масив; вершина — індекс _count - 1
    private int _count;             // кількість елементів у стеку

    public int Count => _count;
    public bool IsEmpty => _count == 0;

    public void Push(T value)
    {
        // Масив заповнений — подвоюємо ємність (амортизоване O(1))
        if (_count == _items.Length) Array.Resize(ref _items, _items.Length * 2);
        _items[_count++] = value;  // записуємо на вершину і збільшуємо лічильник
    }

    public T Pop()
    {
        if (IsEmpty) throw new InvalidOperationException("Pop from empty stack");
        T value = _items[--_count];  // беремо вершину, зменшуючи лічильник
        _items[_count] = default!;   // обнуляємо комірку, щоб GC міг зібрати об'єкт
        return value;
    }

    public T Peek()
    {
        if (IsEmpty) throw new InvalidOperationException("Peek at empty stack");
        return _items[_count - 1];  // лише читаємо вершину
    }
}
```

**Приклад запуску:**
```
top = c, size = 3
after pop: top = b
```

### Приклад 4. Стек на зв'язному списку

```csharp
var s = new LinkedStack<int>();
for (int i = 10; i <= 30; i += 10) s.Push(i);
while (!s.IsEmpty)
{
    Console.Write($"{s.Pop()} ");
}
Console.WriteLine();

public class LinkedStack<T>
{
    // Вузол списку: значення + посилання на наступний (нижчий) вузол
    private sealed class Node(T value, Node? next)
    {
        public T Value { get; } = value;
        public Node? Next { get; } = next;
    }

    private Node? _head;  // вершина стека = голова списку
    private int _count;

    public int Count => _count;
    public bool IsEmpty => _head is null;

    public void Push(T value)
    {
        // Новий вузол вказує на стару вершину і сам стає вершиною — O(1)
        _head = new Node(value, _head);
        _count++;
    }

    public T Pop()
    {
        if (_head is null) throw new InvalidOperationException("Pop from empty stack");
        T value = _head.Value;
        _head = _head.Next;  // вершиною стає наступний вузол; старий збере GC
        _count--;
        return value;
    }

    public T Peek() =>
        _head is null ? throw new InvalidOperationException("Peek at empty stack") : _head.Value;
}
```

**Приклад запуску:**
```
30 20 10
```

---

## 4. Застосування стека

### Приклад 5. Перевірка дужок

Ідея: відкриваючу дужку кладемо в стек; закриваюча має відповідати вершині стека. Наприкінці стек має бути порожнім. Складність — O(n).

```csharp
foreach (string s in new[] { "{[()]}", "([)]", "((a+b)*c", "f(x[1])" })
{
    Console.WriteLine($"{s} -> {(IsBalanced(s) ? "OK" : "ERROR")}");
}

static bool IsBalanced(string s)
{
    var stack = new Stack<char>();
    foreach (char c in s)
    {
        if (c is '(' or '[' or '{')
        {
            stack.Push(c);  // запам'ятовуємо відкриваючу дужку
        }
        else if (c is ')' or ']' or '}')
        {
            if (!stack.TryPop(out char open)) return false;  // закриваюча без пари
            // Перевіряємо, що типи дужок збігаються
            char expected = c switch { ')' => '(', ']' => '[', _ => '{' };
            if (open != expected) return false;
        }
        // інші символи ігноруємо
    }
    return stack.Count == 0;  // лишились незакриті дужки => false
}
```

**Приклад запуску:**
```
{[()]} -> OK
([)] -> ERROR
((a+b)*c -> ERROR
f(x[1]) -> OK
```

### Приклад 6. Обчислення постфіксного виразу

У **постфіксному записі** (зворотна польська нотація) оператор стоїть після операндів: `(3 + 4) * 2` → `3 4 + 2 *`. Дужки не потрібні.

Алгоритм: число — у стек; оператор — дістаємо два операнди, обчислюємо, результат кладемо назад.

```csharp
Console.WriteLine(EvalPostfix("3 4 + 2 *"));         // (3 + 4) * 2
Console.WriteLine(EvalPostfix("5 1 2 + 4 * + 3 -")); // 5 + (1 + 2) * 4 - 3

static long EvalPostfix(string expr)
{
    var stack = new Stack<long>();
    // Токени розділені пробілами; порожні частини пропускаємо
    foreach (string token in expr.Split(' ', StringSplitOptions.RemoveEmptyEntries))
    {
        if (token is "+" or "-" or "*" or "/")
        {
            if (stack.Count < 2) throw new InvalidOperationException("Not enough operands");
            // Порядок важливий: спочатку дістаємо ПРАВИЙ операнд
            long b = stack.Pop();
            long a = stack.Pop();
            stack.Push(token switch
            {
                "+" => a + b,
                "-" => a - b,
                "*" => a * b,
                _ => a / b,
            });
        }
        else
        {
            stack.Push(long.Parse(token));  // операнд — просто в стек
        }
    }
    if (stack.Count != 1) throw new InvalidOperationException("Invalid expression");
    return stack.Peek();
}
```

**Приклад запуску:**
```
14
14
```

---

## 5. Застосування черги

Черга моделює обслуговування в порядку надходження: принтер, запити до сервера, BFS у графах.

### Приклад 7. Симуляція каси

Клієнти приходять у певний час і потребують певний час обслуговування. Одна каса обслуговує їх по черзі. Порахуємо час очікування.

```csharp
using System.Globalization;

var line = new Queue<Customer>();
// Клієнти вже впорядковані за часом приходу
line.Enqueue(new Customer("Anna", 0, 3));
line.Enqueue(new Customer("Bohdan", 1, 2));
line.Enqueue(new Customer("Olha", 2, 4));
line.Enqueue(new Customer("Taras", 10, 1));

int clock = 0;      // поточний час каси
int totalWait = 0;
int served = 0;

while (line.TryDequeue(out Customer? c))  // обслуговуємо першого в черзі (FIFO)
{
    // Якщо каса вільна раніше, ніж клієнт прийшов, — чекаємо на клієнта
    int start = Math.Max(clock, c.Arrival);
    int wait = start - c.Arrival;  // скільки клієнт простояв у черзі
    clock = start + c.Service;     // каса звільниться в цей момент

    Console.WriteLine($"{c.Name}: start={start}, wait={wait}, done={clock}");
    totalWait += wait;
    served++;
}
// InvariantCulture — щоб дробова частина відділялась крапкою незалежно від локалі
double average = (double)totalWait / served;
Console.WriteLine($"Average wait: {average.ToString(CultureInfo.InvariantCulture)}");

// Arrival — момент приходу, Service — тривалість обслуговування
public record Customer(string Name, int Arrival, int Service);
```

**Приклад запуску:**
```
Anna: start=0, wait=0, done=3
Bohdan: start=3, wait=2, done=5
Olha: start=5, wait=3, done=9
Taras: start=10, wait=0, done=11
Average wait: 1.25
```

---

## 6. Підсумки

- **Стек** — LIFO, операції `Push`/`Pop`/`Peek` за O(1). Застосування: дужки, обчислення виразів, стек викликів, скасування дій (undo).
- **Черга** — FIFO, операції `Enqueue`/`Dequeue`/`Peek` за O(1). Застосування: симуляції, буфери, BFS.
- **Дек** — вставка/видалення з обох кінців; у .NET — `LinkedList<T>` (`AddFirst`/`AddLast`/`RemoveFirst`/`RemoveLast`).
- **Черга з пріоритетом** — купа; `Enqueue`/`Dequeue` за O(log n), `Peek` за O(1). За замовчуванням min-heap.
- Стек реалізується на масиві (амортизоване O(1), кеш-дружньо) або на списку (строге O(1)).

---

## 7. Питання для самоперевірки

1. Чим відрізняються принципи LIFO і FIFO? Наведіть по два приклади застосування.
2. Чим відрізняються `Pop()` і `TryPop(out T)`? Коли варто використовувати кожен?
3. Порівняйте реалізацію стека на масиві та на зв'язному списку за часом і пам'яттю.
4. Обчисліть постфіксний вираз `2 3 4 * + 5 -` і покажіть стан стека після кожного токена.
5. Як створити `PriorityQueue<TElement, TPriority>`, що повертає найбільший елемент? Яка складність `Enqueue` і `Dequeue`?
