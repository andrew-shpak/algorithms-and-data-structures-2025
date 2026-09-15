# Лекція 6 — Стек та черга: реалізації та застосування

> **Тривалість:** ~195 хвилин (з трьома перервами).
> **Передумови:** масиви, зв'язні списки та базові `Stack<T>`/`Queue<T>` з [Лекції 2](../Lecture-2/Examples.md), асимптотична складність.
> Тут ми **не повторюємо** основи дослівно, а йдемо глибше: контракти, внутрішня будова, амортизований аналіз і, головне, — **десятки задач**, які розв'язуються стеком, чергою, деком та купою.
>
> Усі приклади — повні програми C# (top-level statements, `Nullable` увімкнено). Кожен можна скопіювати у `Program.cs` порожнього консольного проєкту і виконати `dotnet run`. Блоки «Приклад запуску» містять **точний** вивід (випадкові числа — з фіксованим seed).

---

## Зміст

| № | Розділ | Хв |
|---|--------|----|
| 1 | [АТД проти реалізації](#1-атд-проти-реалізації) | 10 |
| 2 | [Контракти, винятки та Try-методи](#2-контракти-винятки-та-try-методи) | 10 |
| 3 | [Реалізації стека](#3-реалізації-стека) | 15 |
| 4 | [Реалізації черги та деку](#4-реалізації-черги-та-деку) | 15 |
| — | ☕ **Перерва 1** | — |
| 5 | [Колекції .NET зсередини](#5-колекції-net-зсередини) | 10 |
| 6 | [Стек: дужки та вирази](#6-стек-дужки-та-вирази) | 20 |
| 7 | [Стек викликів і рекурсія → явний стек](#7-стек-викликів-і-рекурсія--явний-стек) | 10 |
| 8 | [Min-stack та монотонний стек](#8-min-stack-та-монотонний-стек) | 15 |
| — | ☕ **Перерва 2** | — |
| 9 | [Стек у прикладних задачах: undo/redo, браузер, шляхи, декодування](#9-стек-у-прикладних-задачах) | 15 |
| 10 | [Черга: BFS](#10-черга-bfs) | 15 |
| 11 | [Черга: симуляції](#11-черга-симуляції) | 15 |
| 12 | [Дек і ковзаючі вікна: максимум, rate limiter, логер](#12-дек-і-ковзаючі-вікна) | 15 |
| — | ☕ **Перерва 3** | — |
| 13 | [Черга з пріоритетом і бінарна купа](#13-черга-з-пріоритетом-і-бінарна-купа) | 20 |
| 14 | [Конкурентні черги: ConcurrentQueue та Channel<T>](#14-конкурентні-черги) | 7 |
| 15 | [Підсумки](#15-підсумки) | 3 |
| 16 | [Питання для самоперевірки](#16-питання-для-самоперевірки) | — |
| 17 | [Практичні завдання](#17-практичні-завдання) | — |
| | **Разом** | **≈195** |

---

## 1. АТД проти реалізації

*(≈10 хв)*

### 1.1. Короткий повтор

У [Лекції 2](../Lecture-2/Examples.md) ми вже бачили:

- **Стек** — LIFO (Last In, First Out): `Push`, `Pop`, `Peek`.
- **Черга** — FIFO (First In, First Out): `Enqueue`, `Dequeue`, `Peek`.

```
   Стек (LIFO)                 Черга (FIFO)
                               
   Push ─┐  ┌─▶ Pop            Enqueue ─▶ ┌───┬───┬───┬───┐ ─▶ Dequeue
         ▼  │                  (tail)     │ 4 │ 3 │ 2 │ 1 │    (head)
       ┌─────┐                            └───┴───┴───┴───┘
       │  3  │ ◀─ top
       ├─────┤
       │  2  │
       ├─────┤
       │  1  │
       └─────┘
```

### 1.2. Абстрактний тип даних (АТД)

**Абстрактний тип даних** (Abstract Data Type, ADT) — це *опис поведінки*: набір операцій і правил (аксіом), яким вони підкоряються. АТД нічого не каже про те, **як** дані зберігаються.

**Реалізація** (implementation) — конкретна структура в пам'яті (масив, зв'язний список, кільцевий буфер), яка виконує контракт АТД з певною складністю.

| АТД | Аксіоми (неформально) | Можливі реалізації |
|-----|-----------------------|--------------------|
| Стек | `Pop(Push(s, x)) == x`; `Peek` не змінює стан; `Pop` на порожньому — помилка | динамічний масив, зв'язний список |
| Черга | елементи виходять у порядку входу; `Dequeue` на порожній — помилка | кільцевий буфер, зв'язний список, два стеки |
| Дек | вставка/видалення з обох кінців | кільцевий буфер, двозв'язний список |
| Черга з пріоритетом | `Dequeue` повертає мінімальний (за пріоритетом) елемент | бінарна купа, відсортований масив, BST |

**Навіщо розділяти?** Клієнтський код, написаний проти інтерфейсу, не залежить від реалізації. Можна замінити масив на список (або навпаки) без зміни алгоритму — і виміряти, що швидше.

### 1.3. Інтерфейси АТД у C#

Опишемо АТД як інтерфейси. Далі в лекції всі наші реалізації будуть їх виконувати.

### Приклад 1. Інтерфейси `IStack<T>` та `IQueue<T>` і клієнт, що не знає реалізації

```csharp
// Клієнтський код працює лише з інтерфейсом — він не знає, що всередині.
IStack<string> stack = new ListBackedStack<string>();
Reverse(stack, ["a", "b", "c"]);

// Функція розвертає послідовність за допомогою БУДЬ-ЯКОГО стека.
static void Reverse(IStack<string> s, string[] items)
{
    foreach (var item in items)
    {
        s.Push(item); // кладемо елементи по черзі: a, b, c
    }

    var result = new List<string>();
    while (!s.IsEmpty)
    {
        result.Add(s.Pop()); // виймаємо у зворотному порядку: c, b, a
    }

    Console.WriteLine(string.Join(" ", result));
}

// АТД «стек»: лише операції, жодного слова про масиви чи вузли.
public interface IStack<T>
{
    int Count { get; }
    bool IsEmpty { get; }
    void Push(T item);
    T Pop();              // кидає InvalidOperationException, якщо порожній
    T Peek();             // кидає InvalidOperationException, якщо порожній
    bool TryPop(out T item);
    bool TryPeek(out T item);
}

// АТД «черга».
public interface IQueue<T>
{
    int Count { get; }
    bool IsEmpty { get; }
    void Enqueue(T item);
    T Dequeue();
    T Peek();
    bool TryDequeue(out T item);
}

// Найпростіша реалізація «на швидку руку» — поверх List<T>.
public sealed class ListBackedStack<T> : IStack<T>
{
    private readonly List<T> _items = [];

    public int Count => _items.Count;
    public bool IsEmpty => _items.Count == 0;

    public void Push(T item) => _items.Add(item); // додаємо в кінець списку — O(1) амортизовано

    public T Pop()
    {
        var item = Peek();                  // Peek сам перевірить порожнечу
        _items.RemoveAt(_items.Count - 1);  // видалення з кінця — O(1), без зсуву
        return item;
    }

    public T Peek() => _items.Count > 0
        ? _items[^1]
        : throw new InvalidOperationException("Stack is empty.");

    public bool TryPop(out T item)
    {
        if (_items.Count == 0)
        {
            item = default!;
            return false;
        }

        item = Pop();
        return true;
    }

    public bool TryPeek(out T item)
    {
        if (_items.Count == 0)
        {
            item = default!;
            return false;
        }

        item = _items[^1];
        return true;
    }
}
```

**Приклад запуску:**

```text
c b a
```

> **Зверніть увагу:** видаляємо з **кінця** списку. Якби вершиною стека був індекс 0, кожен `Pop` зсував би весь масив — O(n) замість O(1).

---

## 2. Контракти, винятки та Try-методи

*(≈10 хв)*

### 2.1. Що робити, коли структура порожня?

Операції `Pop`/`Dequeue`/`Peek` мають **передумову**: структура не порожня. Є три стратегії поведінки при її порушенні:

| Стратегія | Приклад | Плюси | Мінуси |
|-----------|---------|-------|--------|
| Кинути виняток | `Stack<T>.Pop()` → `InvalidOperationException` | помилку не проґавиш | винятки дорогі; `try/catch` для звичайного сценарію — антипатерн |
| Try-метод | `TryPop(out T item)` → `bool` | дешево, явно | треба перевіряти результат |
| Повернути «спецзначення» | `null`, `-1` | коротко | неоднозначно (а якщо `-1` — валідний елемент?) |

**Правило .NET:** якщо порожнеча — *очікувана* ситуація (цикл «поки є елементи»), використовуйте `TryPop`/`TryDequeue`/`TryPeek`. Якщо порожнеча означає *баг* у програмі — `Pop`/`Dequeue` з винятком.

**Чому саме `InvalidOperationException`?** За гайдлайнами .NET цей виняток означає «виклик некоректний для **поточного стану** об'єкта». Аргументів у `Pop()` немає, тож `ArgumentException` не підходить.

### Приклад 2. Виняток проти Try-методу

```csharp
var stack = new Stack<int>();
stack.Push(10);

// 1) Try-методи: порожнеча — нормальна ситуація.
while (stack.TryPop(out int value))
{
    Console.WriteLine($"TryPop -> {value}");
}

Console.WriteLine($"TryPeek on empty: {stack.TryPeek(out _)}");

// 2) Pop на порожньому стеку — порушення контракту → виняток.
try
{
    stack.Pop();
}
catch (InvalidOperationException ex)
{
    Console.WriteLine($"{ex.GetType().Name}: {ex.Message}");
}

// 3) Те саме для черги.
var queue = new Queue<string>();
Console.WriteLine($"TryDequeue on empty: {queue.TryDequeue(out string? s)}, s is null: {s is null}");

try
{
    queue.Dequeue();
}
catch (InvalidOperationException ex)
{
    Console.WriteLine($"{ex.GetType().Name}: {ex.Message}");
}
```

**Приклад запуску:**

```text
TryPop -> 10
TryPeek on empty: False
InvalidOperationException: Stack empty.
TryDequeue on empty: False, s is null: True
InvalidOperationException: Queue empty.
```

### 2.2. Інваріанти реалізації

Крім зовнішнього контракту, кожна реалізація має **інваріанти** — умови, що істинні між будь-якими двома викликами публічних методів. Наприклад, для стека на масиві:

- `0 <= _count <= _items.Length`;
- елементи стека — це рівно `_items[0.._count]`, вершина — `_items[_count - 1]`;
- комірки `_items[_count..]` не тримають посилань (щоб GC міг зібрати об'єкти).

Останній пункт часто забувають — див. «Типові помилки» нижче.

### Типові помилки

1. **`try/catch` замість `TryPop`.** Обробка винятку в циклі в сотні разів повільніша за перевірку `bool`.
2. **`Count > 0` + `Pop` у багатопотоковому коді.** Між перевіркою і викликом інший потік може забрати елемент. Використовуйте `ConcurrentStack<T>.TryPop` (розділ 14).
3. **Ігнорування результату `TryPop`.** Після `false` значення `out`-параметра — `default(T)`, а не «щось корисне».
4. **Повернення `-1` як ознаки порожнечі** у стеку цілих чисел, де `-1` — валідне значення.

---

## 3. Реалізації стека

*(≈15 хв)*

### 3.1. Стек на динамічному масиві

```
 _items (Capacity = 4)            Push(40): масив повний → подвоюємо
 ┌────┬────┬────┬────┐            ┌────┬────┬────┬────┬────┬────┬────┬────┐
 │ 10 │ 20 │ 30 │ 35 │   ──────▶  │ 10 │ 20 │ 30 │ 35 │ 40 │    │    │    │
 └────┴────┴────┴────┘            └────┴────┴────┴────┴────┴────┴────┴────┘
                   ▲ top (_count=4)                    ▲ top (_count=5)
```

**Амортизований аналіз подвоєння.** Нехай починаємо з місткості 1 і робимо n операцій `Push`. Копіювання відбувається на розмірах 1, 2, 4, …, 2^k < n. Сумарно копіюємо `1 + 2 + 4 + … + 2^k < 2n` елементів. Отже n операцій коштують `n (запис) + 2n (копіювання) = O(n)`, тобто **O(1) амортизовано** на операцію.

> Чому саме **множення** (×2, ×1.5), а не додавання (+10)? При `+c` копіювання сумарно дає `c + 2c + 3c + … ≈ n²/(2c)` — це O(n²) на n операцій.

### Приклад 3. `ArrayStack<T>` з ростом і лічильником копіювань

```csharp
var stack = new ArrayStack<int>(capacity: 1);

for (int i = 1; i <= 17; i++)
{
    int before = stack.Capacity;
    stack.Push(i * 10);
    if (stack.Capacity != before)
    {
        // Друкуємо лише моменти росту масиву.
        Console.WriteLine($"Push #{i,2}: capacity {before,2} -> {stack.Capacity,2}");
    }
}

Console.WriteLine($"Count = {stack.Count}, total copies = {stack.TotalCopies} (< 2 * 17 = 34)");
Console.WriteLine($"Peek = {stack.Peek()}");
Console.Write("Pop all: ");
while (stack.TryPop(out int x))
{
    Console.Write($"{x} ");
}

Console.WriteLine();

public sealed class ArrayStack<T>
{
    private T[] _items;
    private int _count;

    public ArrayStack(int capacity = 4)
    {
        ArgumentOutOfRangeException.ThrowIfNegativeOrZero(capacity);
        _items = new T[capacity];
    }

    public int Count => _count;
    public int Capacity => _items.Length;
    public long TotalCopies { get; private set; } // скільки елементів скопійовано при ростах

    public void Push(T item)
    {
        if (_count == _items.Length)
        {
            Grow(); // рідкісна дорога операція
        }

        _items[_count++] = item; // звичайна дешева операція O(1)
    }

    public T Pop()
    {
        if (_count == 0)
        {
            throw new InvalidOperationException("Stack is empty.");
        }

        T item = _items[--_count];
        _items[_count] = default!; // ВАЖЛИВО: звільняємо посилання для GC
        return item;
    }

    public bool TryPop(out T item)
    {
        if (_count == 0)
        {
            item = default!;
            return false;
        }

        item = Pop();
        return true;
    }

    public T Peek() => _count > 0
        ? _items[_count - 1]
        : throw new InvalidOperationException("Stack is empty.");

    private void Grow()
    {
        var bigger = new T[_items.Length * 2];     // подвоюємо місткість
        Array.Copy(_items, bigger, _count);        // O(n) копіювання
        TotalCopies += _count;
        _items = bigger;
    }
}
```

**Приклад запуску:**

```text
Push # 2: capacity  1 ->  2
Push # 3: capacity  2 ->  4
Push # 5: capacity  4 ->  8
Push # 9: capacity  8 -> 16
Push #17: capacity 16 -> 32
Count = 17, total copies = 31 (< 2 * 17 = 34)
Peek = 170
Pop all: 170 160 150 140 130 120 110 100 90 80 70 60 50 40 30 20 10 
```

### 3.2. Стек на зв'язному списку

```
 Push(30):   top ──▶ [30] ──▶ [20] ──▶ [10] ──▶ null
 Pop():      top ──────────▶ [20] ──▶ [10] ──▶ null     (повертає 30)
```

Кожен `Push` — нова алокація вузла, зате **немає копіювань** і гарантоване O(1) у найгіршому випадку (не лише амортизовано). Мінус — більше пам'яті на елемент (посилання + заголовок об'єкта) і гірша локальність кешу.

### Приклад 4. `LinkedStack<T>` з перелічуванням від вершини

```csharp
using System.Collections;

var stack = new LinkedStack<string>();
stack.Push("first");
stack.Push("second");
stack.Push("third");

// foreach працює, бо реалізовано IEnumerable<T>: обхід від вершини до дна.
Console.WriteLine($"Items (top→bottom): {string.Join(", ", stack)}");
Console.WriteLine($"Pop: {stack.Pop()}, Peek: {stack.Peek()}, Count: {stack.Count}");

public sealed class LinkedStack<T> : IEnumerable<T>
{
    // Вузол односпрямованого списку. Незмінний: Next задається лише в конструкторі.
    private sealed class Node(T value, Node? next)
    {
        public T Value { get; } = value;
        public Node? Next { get; } = next;
    }

    private Node? _top;

    public int Count { get; private set; }

    public void Push(T item)
    {
        _top = new Node(item, _top); // новий вузол вказує на стару вершину
        Count++;
    }

    public T Pop()
    {
        var node = _top ?? throw new InvalidOperationException("Stack is empty.");
        _top = node.Next; // вершиною стає наступний вузол; старий збере GC
        Count--;
        return node.Value;
    }

    public T Peek() => _top is null
        ? throw new InvalidOperationException("Stack is empty.")
        : _top.Value;

    public IEnumerator<T> GetEnumerator()
    {
        for (var node = _top; node is not null; node = node.Next)
        {
            yield return node.Value;
        }
    }

    IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();
}
```

**Приклад запуску:**

```text
Items (top→bottom): third, second, first
Pop: third, Peek: second, Count: 2
```

### 3.3. Порівняння

| Критерій | Масив | Зв'язний список |
|----------|-------|-----------------|
| `Push` | O(1) амортизовано, O(n) у найгіршому | O(1) завжди |
| `Pop`, `Peek` | O(1) | O(1) |
| Пам'ять на елемент | лише значення (+ запас місткості) | значення + посилання + заголовок об'єкта (~24+ байт) |
| Кеш процесора | відмінно (суцільний блок) | погано (вузли розкидані) |
| Алокації | рідко (при рості) | на кожен `Push` |

**Висновок:** на практиці майже завжди перемагає масив — тому `Stack<T>` у .NET саме такий.

### Міні-вправа 3.1

Додайте до `ArrayStack<T>` метод `TrimExcess()`, який зменшує масив до `Count`, якщо заповнено менше ніж 90%. Чому **небезпечно** автоматично зменшувати масив удвічі, щойно `Count == Capacity / 2`?

<details>
<summary>Розв'язок</summary>

```csharp
var s = new ArrayStack<int>();
for (int i = 0; i < 10; i++) s.Push(i);
for (int i = 0; i < 8; i++) s.Pop();
Console.WriteLine($"Before: Count={s.Count}, Capacity={s.Capacity}");
s.TrimExcess();
Console.WriteLine($"After:  Count={s.Count}, Capacity={s.Capacity}");

public sealed class ArrayStack<T>
{
    private T[] _items = new T[4];
    private int _count;

    public int Count => _count;
    public int Capacity => _items.Length;

    public void Push(T item)
    {
        if (_count == _items.Length) Array.Resize(ref _items, _items.Length * 2);
        _items[_count++] = item;
    }

    public T Pop()
    {
        if (_count == 0) throw new InvalidOperationException("Stack is empty.");
        T item = _items[--_count];
        _items[_count] = default!;
        return item;
    }

    public void TrimExcess()
    {
        // Як і в .NET: не чіпаємо масив, якщо він заповнений на ≥ 90%.
        if (_count >= (int)(_items.Length * 0.9)) return;
        Array.Resize(ref _items, Math.Max(_count, 1));
    }
}
```

```text
Before: Count=2, Capacity=16
After:  Count=2, Capacity=2
```

**Чому не зменшувати на половині?** Уявіть `Count == Capacity/2` і чергування `Push`, `Pop`, `Push`, `Pop`… Кожен `Push` подвоює масив, кожен `Pop` — зменшує: кожна операція O(n) — «пилка» (thrashing). Правильно — зменшувати, коли заповнено **чверть** (гістерезис), або лише за явним викликом.

</details>

---

## 4. Реалізації черги та деку

*(≈15 хв)*

### 4.1. Чому не просто масив зі зсувом?

Якщо `Dequeue` видаляє `_items[0]` і зсуває решту, кожна операція — O(n). Рішення — **кільцевий буфер** (circular buffer, ring buffer): індекси `_head` і `_tail` рухаються вперед і «загортаються» через `% Capacity`.

```
 Capacity = 5, після Enqueue(1..5), Dequeue×2, Enqueue(6, 7):

 індекс:   0     1     2     3     4
         ┌─────┬─────┬─────┬─────┬─────┐
         │  6  │  7  │  3  │  4  │  5  │
         └─────┴─────┴─────┴─────┴─────┘
                  ▲     ▲
         tail=2 ──┘     └── head=2        Count = 5 (повна)
         (куди писати)      (звідки читати)

 Логічний порядок: 3, 4, 5, 6, 7   (від head по колу)
```

Щоб розрізнити «порожня» і «повна» (у обох випадках `head == tail`), зберігаємо окремий `_count`.

### Приклад 5. `CircularQueue<T>` з ростом і візуалізацією буфера

```csharp
var q = new CircularQueue<int>(capacity: 5);
for (int i = 1; i <= 5; i++) q.Enqueue(i);
Console.WriteLine($"after 1..5       {q.Dump()}");

q.Dequeue();
q.Dequeue();
Console.WriteLine($"after Dequeue×2  {q.Dump()}");

q.Enqueue(6);
q.Enqueue(7);
Console.WriteLine($"after 6,7 (wrap) {q.Dump()}");

q.Enqueue(8); // повна → ріст, елементи перекладаються у правильному порядку
Console.WriteLine($"after 8 (grow)   {q.Dump()}");

Console.Write("Dequeue all: ");
while (q.TryDequeue(out int x)) Console.Write($"{x} ");
Console.WriteLine();

public sealed class CircularQueue<T>
{
    private T[] _buffer;
    private int _head;   // індекс першого елемента (звідки Dequeue)
    private int _tail;   // індекс, куди запишемо наступний Enqueue
    private int _count;

    public CircularQueue(int capacity = 4)
    {
        ArgumentOutOfRangeException.ThrowIfNegativeOrZero(capacity);
        _buffer = new T[capacity];
    }

    public int Count => _count;

    public void Enqueue(T item)
    {
        if (_count == _buffer.Length)
        {
            Grow();
        }

        _buffer[_tail] = item;
        _tail = (_tail + 1) % _buffer.Length; // «загортання» індексу по колу
        _count++;
    }

    public T Dequeue()
    {
        if (_count == 0)
        {
            throw new InvalidOperationException("Queue is empty.");
        }

        T item = _buffer[_head];
        _buffer[_head] = default!;             // звільняємо посилання
        _head = (_head + 1) % _buffer.Length;
        _count--;
        return item;
    }

    public bool TryDequeue(out T item)
    {
        if (_count == 0)
        {
            item = default!;
            return false;
        }

        item = Dequeue();
        return true;
    }

    public T Peek() => _count > 0
        ? _buffer[_head]
        : throw new InvalidOperationException("Queue is empty.");

    private void Grow()
    {
        var bigger = new T[_buffer.Length * 2];
        // Копіюємо у ЛОГІЧНОМУ порядку: від head по колу. Простий Array.Copy(0..) зламав би порядок.
        for (int i = 0; i < _count; i++)
        {
            bigger[i] = _buffer[(_head + i) % _buffer.Length];
        }

        _buffer = bigger;
        _head = 0;
        _tail = _count;
    }

    // Показує фізичний вміст буфера та позиції head/tail — для навчання.
    public string Dump()
    {
        var cells = new string[_buffer.Length];
        for (int i = 0; i < _buffer.Length; i++)
        {
            bool occupied = _count > 0 && ((i - _head + _buffer.Length) % _buffer.Length) < _count;
            cells[i] = occupied ? $"{_buffer[i]}" : ".";
        }

        return $"[{string.Join(" ", cells)}] head={_head} tail={_tail} count={_count}";
    }
}
```

**Приклад запуску:**

```text
after 1..5       [1 2 3 4 5] head=0 tail=0 count=5
after Dequeue×2  [. . 3 4 5] head=2 tail=0 count=3
after 6,7 (wrap) [6 7 3 4 5] head=2 tail=2 count=5
after 8 (grow)   [3 4 5 6 7 8 . . . .] head=0 tail=6 count=6
Dequeue all: 3 4 5 6 7 8 
```

### 4.2. Черга на двох стеках і амортизований аналіз

Ідея: `_in` приймає нові елементи, `_out` віддає. Коли `_out` порожній, перекладаємо **весь** `_in` у `_out` — порядок розвертається, і найстаріший елемент опиняється на вершині.

```
 Enqueue 1,2,3:     _in: [1 2 3>     _out: [>
 Dequeue:           переливаємо →    _in: [>        _out: [3 2 1>   → Pop = 1
 Enqueue 4:         _in: [4>         _out: [3 2>
 Dequeue:           _out не порожній → Pop = 2 (без переливання)
```

**Амортизований аналіз (метод бухгалтерського обліку).** Кожен елемент за життя: 1 `Push` у `_in`, щонайбільше 1 `Pop` з `_in`, 1 `Push` у `_out`, 1 `Pop` з `_out` — **не більше 4** елементарних операцій. Отже n операцій коштують O(n), тобто кожна — **O(1) амортизовано**, хоча окремий `Dequeue` може бути O(n).

**Метод потенціалів.** Візьмемо Φ = 2·|_in|. `Enqueue`: реальна вартість 1, ΔΦ = +2 → амортизована 3. `Dequeue` з переливанням k елементів: реальна 2k + 1, ΔΦ = −2k → амортизована 1. Обидві O(1).

> ⚠️ Амортизоване O(1) ≠ O(1) у найгіршому випадку. Для систем реального часу (аудіо, ігровий кадр) пік O(n) може бути неприйнятним.

### Приклад 6. `TwoStackQueue<T>` з підрахунком елементарних операцій

```csharp
var q = new TwoStackQueue<int>();
const int n = 1000;
int dequeued = 0;

// Змішане навантаження: 2 Enqueue, 1 Dequeue, повторити.
for (int i = 0; i < n; i++)
{
    q.Enqueue(i);
    q.Enqueue(i + n);
    q.Dequeue();
    dequeued++;
}

while (q.Count > 0)
{
    q.Dequeue();
    dequeued++;
}

int totalOps = 2 * n + dequeued;
Console.WriteLine($"Queue operations: {totalOps}");
Console.WriteLine($"Primitive stack ops: {q.PrimitiveOps}");
Console.WriteLine($"Ratio: {(double)q.PrimitiveOps / totalOps:F2} (bounded by a constant)");
Console.WriteLine($"Longest single transfer: {q.MaxTransfer}");

var small = new TwoStackQueue<char>();
foreach (char c in "abc") small.Enqueue(c);
Console.Write($"{small.Dequeue()} ");
small.Enqueue('d');
while (small.Count > 0) Console.Write($"{small.Dequeue()} ");
Console.WriteLine();

public sealed class TwoStackQueue<T>
{
    private readonly Stack<T> _in = new();   // сюди додаємо
    private readonly Stack<T> _out = new();  // звідси забираємо

    public int Count => _in.Count + _out.Count;
    public long PrimitiveOps { get; private set; }
    public int MaxTransfer { get; private set; }

    public void Enqueue(T item)
    {
        _in.Push(item);
        PrimitiveOps++;
    }

    public T Dequeue()
    {
        if (_out.Count == 0)
        {
            if (_in.Count == 0)
            {
                throw new InvalidOperationException("Queue is empty.");
            }

            // Переливання: розвертає порядок, найстаріший стає вершиною _out.
            MaxTransfer = Math.Max(MaxTransfer, _in.Count);
            while (_in.TryPop(out T? x))
            {
                _out.Push(x);
                PrimitiveOps += 2; // Pop + Push
            }
        }

        PrimitiveOps++;
        return _out.Pop();
    }
}
```

**Приклад запуску:**

```text
Queue operations: 4000
Primitive stack ops: 8000
Ratio: 2.00 (bounded by a constant)
Longest single transfer: 978
a b c d 
```

Кожен з 2000 елементів пройшов рівно 4 примітивні операції — `Ratio` обмежене константою.

### 4.3. Черга на зв'язному списку

Тримаємо **два** посилання: `_head` (звідки забираємо) і `_tail` (куди додаємо). Обидві операції — O(1) у найгіршому випадку.

```
 _head                         _tail
   │                             │
   ▼                             ▼
 [ A ] ──▶ [ B ] ──▶ [ C ] ──▶ [ D ] ──▶ null

 Enqueue(E): _tail.Next = new(E); _tail = _tail.Next
 Dequeue():  value = _head.Value; _head = _head.Next; якщо _head == null → _tail = null
```

### Приклад 7. `LinkedQueue<T>`

```csharp
var q = new LinkedQueue<string>();
q.Enqueue("A");
q.Enqueue("B");
Console.WriteLine(q.Dequeue()); // A
Console.WriteLine(q.Dequeue()); // B — черга стала порожньою, _tail теж скинуто
q.Enqueue("C");                 // без скидання _tail тут був би баг
Console.WriteLine($"{q.Peek()} (Count = {q.Count})");

public sealed class LinkedQueue<T>
{
    private sealed class Node(T value)
    {
        public T Value { get; } = value;
        public Node? Next { get; set; }
    }

    private Node? _head;
    private Node? _tail;

    public int Count { get; private set; }

    public void Enqueue(T item)
    {
        var node = new Node(item);
        if (_tail is null)
        {
            _head = _tail = node; // перший елемент — одночасно голова і хвіст
        }
        else
        {
            _tail.Next = node;    // причіплюємо в кінець
            _tail = node;
        }

        Count++;
    }

    public T Dequeue()
    {
        var node = _head ?? throw new InvalidOperationException("Queue is empty.");
        _head = node.Next;
        if (_head is null)
        {
            _tail = null; // КЛЮЧОВИЙ рядок: інакше _tail вказує на «мертвий» вузол
        }

        Count--;
        return node.Value;
    }

    public T Peek() => _head is null
        ? throw new InvalidOperationException("Queue is empty.")
        : _head.Value;
}
```

**Приклад запуску:**

```text
A
B
C (Count = 1)
```

### 4.4. Дек на кільцевому буфері

**Дек** (deque, double-ended queue) підтримує `PushFront`, `PushBack`, `PopFront`, `PopBack` за O(1). На кільцевому буфері `PushFront` просто зсуває `_head` **назад**: `(_head - 1 + Capacity) % Capacity`.

```
 PushBack(3), PushBack(4), PushFront(2), PushFront(1)    (Capacity = 8)

 індекс:  0    1    2    3    4    5    6    7
        ┌────┬────┬────┬────┬────┬────┬────┬────┐
        │ 3  │ 4  │    │    │    │    │ 1  │ 2  │
        └────┴────┴────┴────┴────┴────┴────┴────┘
                                        ▲ head=6
 Логічно: 1 2 3 4
```

> У .NET немає класу `Deque<T>`. Варіанти: `LinkedList<T>` (O(1) з обох кінців, але алокація на вузол) або власний кільцевий дек, як нижче. Він знадобиться нам у розділі 12.

### Приклад 8. `RingDeque<T>`

```csharp
var d = new RingDeque<int>(capacity: 2);
d.PushBack(3);
d.PushBack(4);
d.PushFront(2);   // тут відбудеться ріст
d.PushFront(1);
d.PushBack(5);
Console.WriteLine($"deque: {string.Join(" ", d.ToArray())}  front={d.PeekFront()} back={d.PeekBack()}");
Console.WriteLine($"PopFront={d.PopFront()}, PopBack={d.PopBack()}");
Console.WriteLine($"deque: {string.Join(" ", d.ToArray())}  d[1]={d[1]}");

public sealed class RingDeque<T>
{
    private T[] _buffer;
    private int _head;
    private int _count;

    public RingDeque(int capacity = 8) => _buffer = new T[Math.Max(1, capacity)];

    public int Count => _count;

    // Доступ за логічним індексом 0..Count-1 — теж O(1).
    public T this[int index]
    {
        get
        {
            if ((uint)index >= (uint)_count) throw new ArgumentOutOfRangeException(nameof(index));
            return _buffer[(_head + index) % _buffer.Length];
        }
    }

    public void PushBack(T item)
    {
        if (_count == _buffer.Length) Grow();
        _buffer[(_head + _count) % _buffer.Length] = item; // tail обчислюємо з head і count
        _count++;
    }

    public void PushFront(T item)
    {
        if (_count == _buffer.Length) Grow();
        _head = (_head - 1 + _buffer.Length) % _buffer.Length; // крок назад по колу
        _buffer[_head] = item;
        _count++;
    }

    public T PopFront()
    {
        if (_count == 0) throw new InvalidOperationException("Deque is empty.");
        T item = _buffer[_head];
        _buffer[_head] = default!;
        _head = (_head + 1) % _buffer.Length;
        _count--;
        return item;
    }

    public T PopBack()
    {
        if (_count == 0) throw new InvalidOperationException("Deque is empty.");
        int tail = (_head + _count - 1) % _buffer.Length;
        T item = _buffer[tail];
        _buffer[tail] = default!;
        _count--;
        return item;
    }

    public T PeekFront() => _count > 0 ? _buffer[_head] : throw new InvalidOperationException("Deque is empty.");

    public T PeekBack() => _count > 0
        ? _buffer[(_head + _count - 1) % _buffer.Length]
        : throw new InvalidOperationException("Deque is empty.");

    public T[] ToArray()
    {
        var result = new T[_count];
        for (int i = 0; i < _count; i++) result[i] = this[i];
        return result;
    }

    private void Grow()
    {
        var bigger = new T[_buffer.Length * 2];
        for (int i = 0; i < _count; i++) bigger[i] = _buffer[(_head + i) % _buffer.Length];
        _buffer = bigger;
        _head = 0;
    }
}
```

**Приклад запуску:**

```text
deque: 1 2 3 4 5  front=1 back=5
PopFront=1, PopBack=5
deque: 2 3 4  d[1]=3
```

### 4.5. Зведена таблиця реалізацій

| Реалізація | Enqueue / PushBack | Dequeue / PopFront | Найгірший випадок | Примітка |
|------------|--------------------|--------------------|-------------------|----------|
| Масив зі зсувом | O(1)* | **O(n)** | O(n) | не використовувати |
| Кільцевий буфер | O(1)* | O(1) | O(n) при рості | `Queue<T>` у .NET |
| Два стеки | O(1)* | O(1)* | O(n) при переливанні | зручно у функціональних мовах |
| Зв'язний список | O(1) | O(1) | O(1) | алокація на елемент |
| Кільцевий дек | O(1)* | O(1) | O(n) при рості | обидва кінці |

\* амортизовано.

### Типові помилки

1. **`(_head - 1) % n` для від'ємних чисел.** У C# `-1 % 5 == -1`, а не 4! Завжди додавайте `n`: `(_head - 1 + n) % n`.
2. **Ріст кільцевого буфера через `Array.Copy(old, new, count)`.** Якщо дані «загорнуті», порядок зламається.
3. **`_head == _tail` як умова порожнечі** без окремого лічильника — не відрізнити порожню й повну чергу.
4. **Незкинутий `_tail`** у черзі на списку після видалення останнього елемента.
5. **Переливання в `TwoStackQueue` при непорожньому `_out`** — порушує порядок FIFO.

### Міні-вправа 4.1

Реалізуйте **стек на двох чергах** (`Push` — O(1), `Pop` — O(n)). Виведіть результат `Push(1), Push(2), Push(3), Pop(), Push(4), Pop(), Pop()`.

<details>
<summary>Розв'язок</summary>

```csharp
var s = new QueueStack<int>();
s.Push(1); s.Push(2); s.Push(3);
Console.Write($"{s.Pop()} ");
s.Push(4);
Console.Write($"{s.Pop()} ");
Console.WriteLine(s.Pop());

public sealed class QueueStack<T>
{
    private Queue<T> _main = new();
    private Queue<T> _helper = new();

    public void Push(T item) => _main.Enqueue(item);

    public T Pop()
    {
        if (_main.Count == 0) throw new InvalidOperationException("Stack is empty.");

        // Переносимо всі, крім останнього, у допоміжну чергу.
        while (_main.Count > 1) _helper.Enqueue(_main.Dequeue());
        T last = _main.Dequeue(); // останній доданий — це вершина стека

        (_main, _helper) = (_helper, _main); // міняємо ролі черг місцями
        return last;
    }
}
```

```text
3 4 2
```

</details>

---

> ## ☕ Перерва 1 (≈50 хв від початку)
>
> 10 хвилин відпочинку. Після перерви — як це влаштовано в .NET і перші задачі на стек.

---

## 5. Колекції .NET зсередини

*(≈10 хв)*

### 5.1. Огляд

| Тип | Всередині | Ключові операції | Складність |
|-----|-----------|------------------|------------|
| `Stack<T>` | `T[] _array`, `int _size` | `Push`, `Pop`, `Peek`, `TryPop`, `TryPeek`, `Contains`, `ToArray`, `TrimExcess`, `EnsureCapacity` | O(1)*, `Contains` O(n) |
| `Queue<T>` | кільцевий буфер: `_array`, `_head`, `_tail`, `_size` | `Enqueue`, `Dequeue`, `Peek`, `TryDequeue`, `TryPeek`, `EnsureCapacity` | O(1)* |
| `LinkedList<T>` | **двозв'язний кільцевий** список `LinkedListNode<T>` | `AddFirst`, `AddLast`, `RemoveFirst`, `RemoveLast`, `AddAfter(node)`, `Remove(node)` | O(1); `Find` O(n) |
| `PriorityQueue<TElement,TPriority>` | 4-арна мін-купа в масиві | `Enqueue`, `Dequeue`, `Peek`, `EnqueueDequeue`, `DequeueEnqueue`, `TryDequeue`, `UnorderedItems` | O(log n); `Peek` O(1) |

\* амортизовано, ріст — подвоєння.

Корисні деталі:

- `Stack<T>.ToArray()` і `foreach` по `Stack<T>` йдуть **від вершини до дна**. Тому `new Stack<T>(stack)` **розвертає** порядок!
- `Queue<T>` і `Stack<T>` мають «версію» (`_version`): зміна колекції під час `foreach` кидає `InvalidOperationException`.
- `PriorityQueue` **не стабільна**: елементи з однаковим пріоритетом виходять у довільному порядку. Не реалізує `IEnumerable<TElement>` у порядку пріоритету — `UnorderedItems` повертає масив купи «як є».
- `PriorityQueue` не підтримує зміну пріоритету (decrease-key) — у Дейкстрі додають дублікат і пропускають застарілі записи (розділ 13).

### Приклад 9. API та «пастки» стандартних колекцій

```csharp
// 1) Порядок перелічування Stack<T> і пастка копіювання.
var stack = new Stack<int>([1, 2, 3]);                 // Push 1, 2, 3 → вершина 3
Console.WriteLine($"stack foreach: {string.Join(" ", stack)}");
var copy = new Stack<int>(stack);                       // перелічує 3,2,1 і пушить → вершина 1!
Console.WriteLine($"naive copy:    {string.Join(" ", copy)}");
var correctCopy = new Stack<int>(stack.Reverse());      // правильна копія
Console.WriteLine($"correct copy:  {string.Join(" ", correctCopy)}");

// 2) Модифікація під час перелічування.
var queue = new Queue<string>(["a", "b"]);
try
{
    foreach (var item in queue)
    {
        queue.Enqueue(item + "!");
    }
}
catch (InvalidOperationException ex)
{
    Console.WriteLine($"foreach+Enqueue: {ex.GetType().Name}");
}

// 3) LinkedList<T> як дек і вставка за вузлом за O(1).
var list = new LinkedList<string>();
list.AddLast("B");
list.AddFirst("A");
LinkedListNode<string> nodeC = list.AddLast("C");
list.AddBefore(nodeC, "B+");   // O(1), бо вузол уже відомий
list.RemoveFirst();
Console.WriteLine($"linked list:   {string.Join(" ", list)}  First={list.First!.Value} Last={list.Last!.Value}");

// 4) PriorityQueue: мінімальний пріоритет виходить першим.
var pq = new PriorityQueue<string, int>();
pq.Enqueue("low", 5);
pq.Enqueue("urgent", 1);
pq.Enqueue("normal", 3);
Console.WriteLine($"pq Peek:       {pq.Peek()}  Count={pq.Count}");
Console.Write("pq order:      ");
while (pq.TryDequeue(out string? task, out int priority))
{
    Console.Write($"{task}({priority}) ");
}

Console.WriteLine();

// 5) EnsureCapacity / TrimExcess — керування масивом.
var big = new Queue<int>();
Console.WriteLine($"EnsureCapacity(100) -> {big.EnsureCapacity(100)}");
```

**Приклад запуску:**

```text
stack foreach: 3 2 1
naive copy:    1 2 3
correct copy:  3 2 1
foreach+Enqueue: InvalidOperationException
linked list:   B B+ C  First=B Last=C
pq Peek:       urgent  Count=3
pq order:      urgent(1) normal(3) low(5) 
EnsureCapacity(100) -> 100
```

### 5.2. Чому 4-арна купа?

У `PriorityQueue` кожен вузол має 4 нащадки: дерево нижче (log₄ n замість log₂ n рівнів), тож `Enqueue` (просіювання вгору) швидший, а на `Dequeue` порівнюємо 4 дітей — але вони лежать поруч у масиві, що дружньо до кешу. Ми ж у розділі 13 напишемо класичну **бінарну** купу — принцип той самий.

---

## 6. Стек: дужки та вирази

*(≈20 хв)*

### 6.1. Перевірка дужок з позицією помилки

Для кожної **відкриваючої** дужки кладемо на стек її символ і позицію. Для **закриваючої** — вершина мусить бути відповідною відкриваючою. Три види помилок:

1. Закриваюча без пари (стек порожній).
2. Невідповідний тип (`(]`).
3. Незакриті дужки наприкінці (стек непорожній) — повідомляємо позицію **найглибшої** незакритої, тобто вершини.

```
 Вхід:  { [ ( ) ] ( }
 поз.:  0 1 2 3 4 5 6

 i=0 '{' push       стек: {0
 i=1 '[' push       стек: {0 [1
 i=2 '(' push       стек: {0 [1 (2
 i=3 ')' pop (2 ✓   стек: {0 [1
 i=4 ']' pop [1 ✓   стек: {0
 i=5 '(' push       стек: {0 (5
 i=6 '}' вершина (5 ≠ { → ПОМИЛКА на позиції 6: очікувалось ')'
```

### Приклад 10. Валідатор дужок

```csharp
string[] inputs =
[
    "{[()]}",
    "if (a[i] == b) { return; }",
    "{[()](}",
    "(a + b))",
    "((x)",
    "\"(\" + ')'",   // дужки в рядкових літералах ігноруємо
];

foreach (var input in inputs)
{
    var result = BracketValidator.Validate(input);
    Console.WriteLine($"{input,-28} -> {result}");
}

public readonly record struct BracketResult(bool IsValid, int ErrorPosition, string Message)
{
    public override string ToString() => IsValid ? "OK" : $"error at {ErrorPosition}: {Message}";
}

public static class BracketValidator
{
    private static readonly Dictionary<char, char> Pairs = new()
    {
        [')'] = '(',
        [']'] = '[',
        ['}'] = '{',
    };

    public static BracketResult Validate(string text)
    {
        // Зберігаємо не лише символ, а й позицію — для гарного повідомлення.
        var stack = new Stack<(char Bracket, int Position)>();
        char? quote = null; // якщо всередині літерала — його лапка

        for (int i = 0; i < text.Length; i++)
        {
            char c = text[i];

            if (quote is not null)
            {
                if (c == quote) quote = null; // кінець літерала
                continue;
            }

            if (c is '"' or '\'')
            {
                quote = c;
                continue;
            }

            if (c is '(' or '[' or '{')
            {
                stack.Push((c, i));
            }
            else if (Pairs.TryGetValue(c, out char expectedOpen))
            {
                if (!stack.TryPop(out var top))
                {
                    return new(false, i, $"unexpected '{c}'");
                }

                if (top.Bracket != expectedOpen)
                {
                    char expectedClose = Pairs.First(p => p.Value == top.Bracket).Key;
                    return new(false, i, $"expected '{expectedClose}' but found '{c}'");
                }
            }
        }

        if (stack.TryPeek(out var unclosed))
        {
            return new(false, unclosed.Position, $"'{unclosed.Bracket}' is never closed");
        }

        return new(true, -1, "");
    }
}
```

**Приклад запуску:**

```text
{[()]}                       -> OK
if (a[i] == b) { return; }   -> OK
{[()](}                      -> error at 6: expected ')' but found '}'
(a + b))                     -> error at 7: unexpected ')'
((x)                         -> error at 0: '(' is never closed
"(" + ')'                    -> OK
```

### 6.2. Нотації виразів

| Нотація | Приклад | Особливість |
|---------|---------|-------------|
| Інфіксна | `3 + 4 * 2` | потрібні пріоритети та дужки |
| Постфіксна (RPN, зворотна польська) | `3 4 2 * +` | без дужок; обчислюється одним стеком зліва направо |
| Префіксна (польська) | `+ 3 * 4 2` | без дужок; обчислюється стеком справа наліво |

### 6.3. Алгоритм сортувальної станції (shunting-yard, Дейкстра)

Правила (для кожного токена зліва направо):

1. **Число** → одразу у вихід.
2. **Оператор `o1`** → поки на вершині стека оператор `o2` і (`o2` має **більший** пріоритет, або **рівний** пріоритет і `o1` **лівоасоціативний**) — виштовхуємо `o2` у вихід. Потім кладемо `o1`.
3. **`(`** → на стек.
4. **`)`** → виштовхуємо у вихід до `(`; саму `(` викидаємо. Не знайшли — помилка дужок.
5. Кінець → виштовхуємо все; якщо трапилась `(` — помилка.

**Асоціативність:** `8 - 3 - 2 = (8 - 3) - 2` (ліва), але `2 ^ 3 ^ 2 = 2 ^ (3 ^ 2) = 512` (права).

**Унарний мінус:** `-` є унарним, якщо стоїть на початку, після `(` або після іншого оператора. Позначимо його `~` з найвищим пріоритетом і **правою** асоціативністю (щоб `--x` працювало), але нижчим за `^`, щоб `-2^2 = -(2^2) = -4` — як у математиці.

```
 Вхід: 3 + 4 * ( 2 - 1 )

 токен │ дія                        │ вихід            │ стек
 ──────┼────────────────────────────┼──────────────────┼────────
 3     │ у вихід                    │ 3                │
 +     │ push                       │ 3                │ +
 4     │ у вихід                    │ 3 4              │ +
 *     │ * > + → push               │ 3 4              │ + *
 (     │ push                       │ 3 4              │ + * (
 2     │ у вихід                    │ 3 4 2            │ + * (
 -     │ вершина ( → push           │ 3 4 2            │ + * ( -
 1     │ у вихід                    │ 3 4 2 1          │ + * ( -
 )     │ pop до (                   │ 3 4 2 1 -        │ + *
 кінець│ pop все                    │ 3 4 2 1 - * +    │
```

### Приклад 11. Shunting-yard + обчислення постфіксного та префіксного виразу

```csharp
using System.Globalization;
using System.Text;

string[] expressions =
[
    "3 + 4 * (2 - 1)",
    "8 - 3 - 2",
    "2 ^ 3 ^ 2",
    "-2 ^ 2",
    "(-3 + 5) * -(2)",
    "10 / 4 - --1",
];

foreach (var expr in expressions)
{
    var tokens = Tokenizer.Tokenize(expr);
    var postfix = ShuntingYard.ToPostfix(tokens);
    double value = RpnEvaluator.EvaluatePostfix(postfix);
    Console.WriteLine($"{expr,-18} => {string.Join(" ", postfix),-22} = {value.ToString(CultureInfo.InvariantCulture)}");
}

// Префіксний вираз: читаємо справа наліво, операнди знімаємо у прямому порядку.
string prefix = "- * 2 + 3 4 / 10 5";
Console.WriteLine($"prefix '{prefix}' = {RpnEvaluator.EvaluatePrefix(prefix.Split(' '))}");

try
{
    ShuntingYard.ToPostfix(Tokenizer.Tokenize("(1 + 2"));
}
catch (FormatException ex)
{
    Console.WriteLine($"FormatException: {ex.Message}");
}

public static class Tokenizer
{
    // Розбиває рядок на токени: числа, оператори, дужки. Унарний мінус → "~".
    public static List<string> Tokenize(string text)
    {
        var tokens = new List<string>();
        int i = 0;
        while (i < text.Length)
        {
            char c = text[i];
            if (char.IsWhiteSpace(c))
            {
                i++;
                continue;
            }

            if (char.IsDigit(c) || c == '.')
            {
                var sb = new StringBuilder();
                while (i < text.Length && (char.IsDigit(text[i]) || text[i] == '.'))
                {
                    sb.Append(text[i++]);
                }

                tokens.Add(sb.ToString());
                continue;
            }

            if (c == '-')
            {
                // Унарний, якщо попереднього токена немає, або це оператор чи '('.
                bool unary = tokens.Count == 0 || tokens[^1] is "(" or "+" or "-" or "*" or "/" or "^" or "~";
                tokens.Add(unary ? "~" : "-");
            }
            else if ("+*/^()".Contains(c))
            {
                tokens.Add(c.ToString());
            }
            else
            {
                throw new FormatException($"Unexpected character '{c}' at {i}.");
            }

            i++;
        }

        return tokens;
    }
}

public static class ShuntingYard
{
    private static int Precedence(string op) => op switch
    {
        "+" or "-" => 1,
        "*" or "/" => 2,
        "~" => 3,        // унарний мінус
        "^" => 4,        // степінь сильніший за унарний мінус: -2^2 = -(2^2)
        _ => throw new ArgumentException($"Not an operator: {op}"),
    };

    private static bool IsRightAssociative(string op) => op is "^" or "~";

    public static bool IsOperator(string token) => token is "+" or "-" or "*" or "/" or "^" or "~";

    public static List<string> ToPostfix(List<string> tokens)
    {
        var output = new List<string>();
        var ops = new Stack<string>();

        foreach (var token in tokens)
        {
            if (IsOperator(token))
            {
                // Унарний оператор-префікс нічого не виштовхує: у нього ще немає операнда.
                while (token != "~"
                       && ops.TryPeek(out var top) && IsOperator(top)
                       && (Precedence(top) > Precedence(token)
                           || (Precedence(top) == Precedence(token) && !IsRightAssociative(token))))
                {
                    output.Add(ops.Pop());
                }

                ops.Push(token);
            }
            else if (token == "(")
            {
                ops.Push(token);
            }
            else if (token == ")")
            {
                while (ops.TryPeek(out var top) && top != "(")
                {
                    output.Add(ops.Pop());
                }

                if (!ops.TryPop(out _))
                {
                    throw new FormatException("Unmatched ')'.");
                }
            }
            else
            {
                output.Add(token); // число
            }
        }

        while (ops.TryPop(out var op))
        {
            if (op == "(") throw new FormatException("Unmatched '('.");
            output.Add(op);
        }

        return output;
    }
}

public static class RpnEvaluator
{
    public static double EvaluatePostfix(IEnumerable<string> postfix)
    {
        var stack = new Stack<double>();
        foreach (var token in postfix)
        {
            if (token == "~")
            {
                stack.Push(-Pop(stack));
            }
            else if (ShuntingYard.IsOperator(token))
            {
                double right = Pop(stack); // УВАГА: спершу правий операнд!
                double left = Pop(stack);
                stack.Push(Apply(token, left, right));
            }
            else
            {
                stack.Push(double.Parse(token, CultureInfo.InvariantCulture));
            }
        }

        return stack.Count == 1 ? stack.Pop() : throw new FormatException("Malformed expression.");
    }

    public static double EvaluatePrefix(IReadOnlyList<string> prefix)
    {
        var stack = new Stack<double>();
        for (int i = prefix.Count - 1; i >= 0; i--) // справа наліво
        {
            string token = prefix[i];
            if (ShuntingYard.IsOperator(token))
            {
                double left = Pop(stack);  // у префіксі спершу знімаємо ЛІВИЙ
                double right = Pop(stack);
                stack.Push(Apply(token, left, right));
            }
            else
            {
                stack.Push(double.Parse(token, CultureInfo.InvariantCulture));
            }
        }

        return stack.Count == 1 ? stack.Pop() : throw new FormatException("Malformed expression.");
    }

    private static double Pop(Stack<double> stack) =>
        stack.TryPop(out double v) ? v : throw new FormatException("Missing operand.");

    private static double Apply(string op, double a, double b) => op switch
    {
        "+" => a + b,
        "-" => a - b,
        "*" => a * b,
        "/" => a / b,
        "^" => Math.Pow(a, b),
        _ => throw new FormatException($"Unknown operator {op}"),
    };
}
```

**Приклад запуску:**

```text
3 + 4 * (2 - 1)    => 3 4 2 1 - * +          = 7
8 - 3 - 2          => 8 3 - 2 -              = 3
2 ^ 3 ^ 2          => 2 3 2 ^ ^              = 512
-2 ^ 2             => 2 2 ^ ~                = -4
(-3 + 5) * -(2)    => 3 ~ 5 + 2 ~ *          = -4
10 / 4 - --1       => 10 4 / 1 ~ ~ -         = 1.5
prefix '- * 2 + 3 4 / 10 5' = 12
FormatException: Unmatched '('.
```

### 6.4. Дерево виразу з постфіксного запису

Постфіксний запис легко перетворити на **дерево виразу**: число → листок на стек; бінарний оператор → знімаємо два піддерева і створюємо вузол. Обхід дерева дає всі три нотації: in-order (з дужками) — інфікс, pre-order — префікс, post-order — постфікс.

```
 Постфікс: 3 4 2 1 - * +

            (+)
           /   \
         3     (*)
              /   \
            4     (-)
                 /   \
                2     1
```

### Приклад 12. Побудова та обходи дерева виразу

```csharp
string[] postfix = "3 4 2 1 - * +".Split(' ');
ExprNode root = ExpressionTree.Build(postfix);

Console.WriteLine($"infix:   {root.ToInfix()}");
Console.WriteLine($"prefix:  {root.ToPrefix()}");
Console.WriteLine($"postfix: {root.ToPostfix()}");
Console.WriteLine($"value:   {root.Evaluate()}");
Console.WriteLine($"height:  {root.Height()}");

// Абстрактний вузол і два конкретні — число та бінарна операція.
public abstract record ExprNode
{
    public abstract double Evaluate();
    public abstract string ToInfix();
    public abstract string ToPrefix();
    public abstract string ToPostfix();
    public abstract int Height();
}

public sealed record NumberNode(double Value) : ExprNode
{
    public override double Evaluate() => Value;
    public override string ToInfix() => Value.ToString();
    public override string ToPrefix() => Value.ToString();
    public override string ToPostfix() => Value.ToString();
    public override int Height() => 1;
}

public sealed record BinaryNode(char Op, ExprNode Left, ExprNode Right) : ExprNode
{
    public override double Evaluate() => Op switch
    {
        '+' => Left.Evaluate() + Right.Evaluate(),
        '-' => Left.Evaluate() - Right.Evaluate(),
        '*' => Left.Evaluate() * Right.Evaluate(),
        '/' => Left.Evaluate() / Right.Evaluate(),
        _ => throw new InvalidOperationException($"Unknown op {Op}"),
    };

    public override string ToInfix() => $"({Left.ToInfix()} {Op} {Right.ToInfix()})";
    public override string ToPrefix() => $"{Op} {Left.ToPrefix()} {Right.ToPrefix()}";
    public override string ToPostfix() => $"{Left.ToPostfix()} {Right.ToPostfix()} {Op}";
    public override int Height() => 1 + Math.Max(Left.Height(), Right.Height());
}

public static class ExpressionTree
{
    public static ExprNode Build(IEnumerable<string> postfix)
    {
        var stack = new Stack<ExprNode>();
        foreach (var token in postfix)
        {
            if (token.Length == 1 && "+-*/".Contains(token[0]))
            {
                var right = stack.Pop(); // порядок важливий: права гілка знімається першою
                var left = stack.Pop();
                stack.Push(new BinaryNode(token[0], left, right));
            }
            else
            {
                stack.Push(new NumberNode(double.Parse(token)));
            }
        }

        return stack.Count == 1 ? stack.Pop() : throw new FormatException("Malformed postfix.");
    }
}
```

**Приклад запуску:**

```text
infix:   (3 + (4 * (2 - 1)))
prefix:  + 3 * 4 - 2 1
postfix: 3 4 2 1 - * +
value:   7
height:  4
```

### Типові помилки

1. **Переплутані операнди** при обчисленні RPN: для `a b -` першим знімається `b`. `left = Pop(); right = Pop();` дає `b - a`.
2. **Однакова обробка асоціативності** для всіх операторів: `2^3^2` стане `64` замість `512`.
3. **Забутий унарний мінус**: `-3 + 5` дає «Missing operand».
4. **Перевірка лише кількості дужок** замість стека: `([)]` має однакову кількість, але невалідний.
5. **Культура при `double.Parse`**: на системі з українською локаллю `"2.5"` не розпарситься. Використовуйте `CultureInfo.InvariantCulture`.

### Міні-вправа 6.1

Для постфіксного виразу `5 1 2 + 4 * + 3 -` запишіть стан стека після кожного токена та відповідь.

<details>
<summary>Розв'язок</summary>

| Токен | Стек (дно → вершина) |
|-------|----------------------|
| 5 | 5 |
| 1 | 5 1 |
| 2 | 5 1 2 |
| + | 5 3 |
| 4 | 5 3 4 |
| * | 5 12 |
| + | 17 |
| 3 | 17 3 |
| - | 14 |

Відповідь: **14**. Інфіксна форма: `5 + (1 + 2) * 4 - 3`.

</details>

---

## 7. Стек викликів і рекурсія → явний стек

*(≈10 хв)*

### 7.1. Стек викликів

Кожен виклик методу створює **кадр стека** (stack frame): параметри, локальні змінні, адреса повернення. Повернення з методу знімає кадр.

```
 Factorial(3)

 │ Factorial(1): n=1 → return 1        │ ◀─ вершина
 │ Factorial(2): n=2, чекає 2*F(1)     │
 │ Factorial(3): n=3, чекає 3*F(2)     │
 │ Main                                │
 └─────────────────────────────────────┘
```

Розмір стека потоку в .NET обмежений (типово 1 МБ для головного потоку на Windows, 8 МБ на Linux/macOS). Глибока рекурсія (скажімо, DFS по ланцюжку з 1 000 000 вершин) закінчиться **`StackOverflowException`**, який **неможливо перехопити** — процес завершиться.

**Рішення:** замінити неявний стек викликів **явним** `Stack<T>` у купі (heap), де пам'яті значно більше.

### 7.2. Техніка перетворення

1. Кадр = запис із параметрами та «станом» (на якому кроці методу ми зупинились).
2. Замість виклику — `Push` кадру; замість повернення — `Pop` і передача результату.
3. Для «хвостових» задач (DFS без пост-обробки) стан не потрібен — достатньо класти вершини.

### Приклад 13. Факторіал, DFS і пост-порядок без рекурсії

```csharp
using System.Numerics;

Console.WriteLine($"FactorialRecursive(20) = {Recursion.FactorialRecursive(20)}");
Console.WriteLine($"FactorialExplicit(20)  = {Recursion.FactorialExplicit(20)}");

// Граф:  0 → 1, 2;  1 → 3;  2 → 3, 4;  3 → 5;  4 → 5
int[][] graph = [[1, 2], [3], [3, 4], [5], [5], []];
Console.WriteLine($"DFS recursive: {string.Join(" ", Recursion.DfsRecursive(graph, 0))}");
Console.WriteLine($"DFS explicit:  {string.Join(" ", Recursion.DfsExplicit(graph, 0))}");
Console.WriteLine($"Post-order:    {string.Join(" ", Recursion.PostOrderExplicit(graph, 0))}");

// Ланцюжок з 1 000 000 вершин: рекурсія тут впала б зі StackOverflow.
const int n = 1_000_000;
int[][] chain = new int[n][];
for (int i = 0; i < n; i++) chain[i] = i + 1 < n ? [i + 1] : [];
Console.WriteLine($"Chain DFS visited: {Recursion.DfsExplicit(chain, 0).Count}");

public static class Recursion
{
    public static BigInteger FactorialRecursive(int n) => n <= 1 ? 1 : n * FactorialRecursive(n - 1);

    // Явний стек імітує дві фази: «спуск» (кладемо n, n-1, ...) і «підйом» (множимо).
    public static BigInteger FactorialExplicit(int n)
    {
        var frames = new Stack<int>();
        while (n > 1)
        {
            frames.Push(n--); // фаза виклику
        }

        BigInteger result = 1; // базовий випадок
        while (frames.TryPop(out int k))
        {
            result *= k;      // фаза повернення: кадри знімаються у зворотному порядку
        }

        return result;
    }

    public static List<int> DfsRecursive(int[][] graph, int start)
    {
        var visited = new bool[graph.Length];
        var order = new List<int>();
        Visit(start);
        return order;

        void Visit(int v)
        {
            visited[v] = true;
            order.Add(v);
            foreach (int next in graph[v])
            {
                if (!visited[next]) Visit(next);
            }
        }
    }

    public static List<int> DfsExplicit(int[][] graph, int start)
    {
        var visited = new bool[graph.Length];
        var order = new List<int>();
        var stack = new Stack<int>();
        stack.Push(start);

        while (stack.TryPop(out int v))
        {
            if (visited[v]) continue; // вершину могли покласти кілька разів
            visited[v] = true;
            order.Add(v);

            // Кладемо сусідів у ЗВОРОТНОМУ порядку, щоб першим обробився graph[v][0] —
            // тоді порядок збігається з рекурсивним варіантом.
            for (int i = graph[v].Length - 1; i >= 0; i--)
            {
                if (!visited[graph[v][i]]) stack.Push(graph[v][i]);
            }
        }

        return order;
    }

    // Пост-порядок потребує «стану кадру»: індекс наступного сусіда, якого треба обробити.
    public static List<int> PostOrderExplicit(int[][] graph, int start)
    {
        var visited = new bool[graph.Length];
        var order = new List<int>();
        var stack = new Stack<(int Vertex, int NextChild)>();
        stack.Push((start, 0));
        visited[start] = true;

        while (stack.TryPop(out var frame))
        {
            var (v, childIndex) = frame;
            if (childIndex < graph[v].Length)
            {
                stack.Push((v, childIndex + 1)); // «зберігаємо» кадр: продовжимо з наступного сусіда
                int child = graph[v][childIndex];
                if (!visited[child])
                {
                    visited[child] = true;
                    stack.Push((child, 0));      // «викликаємо» дитину
                }
            }
            else
            {
                order.Add(v); // усі діти оброблені — «повертаємось»
            }
        }

        return order;
    }
}
```

**Приклад запуску:**

```text
FactorialRecursive(20) = 2432902008176640000
FactorialExplicit(20)  = 2432902008176640000
DFS recursive: 0 1 3 5 2 4
DFS explicit:  0 1 3 5 2 4
Post-order:    5 3 1 4 2 0
Chain DFS visited: 1000000
```

> Пост-порядок DFS — це основа **топологічного сортування** (у зворотному порядку): `5 3 1 4 2 0` → реверс `0 2 4 1 3 5`.

### Типові помилки

1. **Позначати `visited` при виштовхуванні без перевірки** — вершина обробиться кілька разів.
2. **Прямий порядок сусідів** у явному DFS — обхід коректний, але порядок відрізнятиметься від рекурсивного (часто плутає студентів при порівнянні з еталоном).
3. **Сподівання перехопити `StackOverflowException`** через `try/catch` — у .NET це неможливо.

---

## 8. Min-stack та монотонний стек

*(≈15 хв)*

### 8.1. Min-stack: мінімум за O(1)

Задача: стек з операцією `GetMin()` за O(1). Ідея — поруч з кожним значенням зберігати **мінімум на момент вставки**.

```
 Push 5   → [(5,5)]
 Push 3   → [(5,5) (3,3)]
 Push 7   → [(5,5) (3,3) (7,3)]    GetMin = 3
 Pop      → [(5,5) (3,3)]          GetMin = 3
 Pop      → [(5,5)]                GetMin = 5
```

### Приклад 14. `MinStack`

```csharp
var s = new MinStack();
foreach (int x in new[] { 5, 3, 7, 3, 1 })
{
    s.Push(x);
    Console.WriteLine($"Push {x}: min = {s.Min}");
}

while (s.Count > 1)
{
    int popped = s.Pop();
    Console.WriteLine($"Pop  {popped}: min = {s.Min}");
}

public sealed class MinStack
{
    // Кожен запис знає мінімум усього стека «під собою і включно».
    private readonly Stack<(int Value, int MinSoFar)> _stack = new();

    public int Count => _stack.Count;

    public int Min => _stack.TryPeek(out var top)
        ? top.MinSoFar
        : throw new InvalidOperationException("Stack is empty.");

    public void Push(int value)
    {
        int min = _stack.TryPeek(out var top) ? Math.Min(value, top.MinSoFar) : value;
        _stack.Push((value, min));
    }

    public int Pop() => _stack.Pop().Value;
}
```

**Приклад запуску:**

```text
Push 5: min = 5
Push 3: min = 3
Push 7: min = 3
Push 3: min = 3
Push 1: min = 1
Pop  1: min = 3
Pop  3: min = 3
Pop  7: min = 3
Pop  3: min = 5
```

### 8.2. Монотонний стек

**Монотонний стек** — стек, у якому елементи завжди впорядковані (наприклад, спадають від дна до вершини). Перед `Push(x)` виштовхуємо всі елементи, що порушують порядок — і **саме в момент виштовхування** ми дізнаємось для них відповідь: «`x` — перший більший за мене праворуч».

Кожен індекс кладеться і знімається **рівно раз** → O(n) замість наївного O(n²).

```
 Next Greater Element для [2, 1, 2, 4, 3]
 (стек зберігає ІНДЕКСИ, значення спадають)

 i=0 v=2  стек: [0]
 i=1 v=1  1<2 → push              стек: [0 1]
 i=2 v=2  2>1 → pop 1, ans[1]=2   стек: [0]; 2==2 не більше → push  стек: [0 2]
 i=3 v=4  pop 2 → ans[2]=4; pop 0 → ans[0]=4                       стек: [3]
 i=4 v=3  push                    стек: [3 4]
 кінець: ans[3] = ans[4] = -1
 Відповідь: [4, 2, 4, -1, -1]
```

### Приклад 15. Next Greater Element, Daily Temperatures, Stock Span

```csharp
int[] nums = [2, 1, 2, 4, 3];
Console.WriteLine($"NGE {Fmt(nums)} -> {Fmt(Monotonic.NextGreater(nums))}");

// Daily temperatures: скільки днів чекати на теплішу погоду.
int[] temps = [73, 74, 75, 71, 69, 72, 76, 73];
Console.WriteLine($"Wait {Fmt(temps)} -> {Fmt(Monotonic.DailyTemperatures(temps))}");

// Stock span: скільки днів поспіль (включно з сьогодні) ціна була <= сьогоднішньої.
int[] prices = [100, 80, 60, 70, 60, 75, 85];
Console.WriteLine($"Span {Fmt(prices)} -> {Fmt(Monotonic.StockSpan(prices))}");

static string Fmt(int[] a) => $"[{string.Join(", ", a)}]";

public static class Monotonic
{
    public static int[] NextGreater(int[] a)
    {
        var answer = new int[a.Length];
        Array.Fill(answer, -1);          // за замовчуванням — «немає більшого»
        var stack = new Stack<int>();    // індекси; a[stack] строго спадають... або рівні

        for (int i = 0; i < a.Length; i++)
        {
            // Поточний елемент — «перший більший» для всіх менших на вершині.
            while (stack.TryPeek(out int top) && a[top] < a[i])
            {
                answer[stack.Pop()] = a[i];
            }

            stack.Push(i);
        }

        return answer;
    }

    public static int[] DailyTemperatures(int[] t)
    {
        var wait = new int[t.Length];
        var stack = new Stack<int>();
        for (int i = 0; i < t.Length; i++)
        {
            while (stack.TryPeek(out int top) && t[top] < t[i])
            {
                stack.Pop();
                wait[top] = i - top; // відстань у днях, а не саме значення
            }

            stack.Push(i);
        }

        return wait;
    }

    public static int[] StockSpan(int[] prices)
    {
        var span = new int[prices.Length];
        var stack = new Stack<int>(); // індекси днів зі строго спадаючими цінами
        for (int i = 0; i < prices.Length; i++)
        {
            // Знімаємо всі дні, коли ціна була <= сьогоднішньої: вони «поглинаються».
            while (stack.TryPeek(out int top) && prices[top] <= prices[i])
            {
                stack.Pop();
            }

            // Попередній більший день — на вершині (або його немає: -1).
            int previousGreater = stack.TryPeek(out int p) ? p : -1;
            span[i] = i - previousGreater;
            stack.Push(i);
        }

        return span;
    }
}
```

**Приклад запуску:**

```text
NGE [2, 1, 2, 4, 3] -> [4, 2, 4, -1, -1]
Wait [73, 74, 75, 71, 69, 72, 76, 73] -> [1, 1, 4, 2, 1, 1, 0, 0]
Span [100, 80, 60, 70, 60, 75, 85] -> [1, 1, 1, 2, 1, 4, 6]
```

### 8.3. Найбільший прямокутник у гістограмі

Для кожного стовпця `h[i]` найбільший прямокутник висоти `h[i]` простягається від **попереднього меншого** до **наступного меншого** стовпця. Монотонний **зростаючий** стек дає обидві межі за один прохід: коли стовпець `top` знімається поточним `i` (бо `h[i] < h[top]`), права межа — `i`, ліва — новий елемент на вершині.

```
 h = [2, 1, 5, 6, 2, 3]

        ┌──┐
     ┌──┤  │
     │▓▓│▓▓│
     │▓▓│▓▓│  ┌──┐
 ┌──┐│▓▓│▓▓├──┤  │
 │  ├┤▓▓│▓▓│  │  │
 └──┴┴──┴──┴──┴──┘
  0  1  2  3  4  5
 Найбільший: висота 5, стовпці 2..3, площа 10
```

### Приклад 16. Largest Rectangle in Histogram з трасуванням

```csharp
int[] heights = [2, 1, 5, 6, 2, 3];
var (area, left, right) = Histogram.LargestRectangle(heights, trace: true);
Console.WriteLine($"Max area = {area} (bars {left}..{right})");

int[] staircase = [1, 2, 3, 4, 5];
Console.WriteLine($"Staircase max area = {Histogram.LargestRectangle(staircase, trace: false).Area}");

public static class Histogram
{
    public static (int Area, int Left, int Right) LargestRectangle(int[] h, bool trace)
    {
        var stack = new Stack<int>(); // індекси з НЕСПАДАЮЧИМИ висотами
        (int Area, int Left, int Right) best = (0, -1, -1);

        // i == h.Length — «сторожовий» стовпець висоти 0, який виштовхне все, що лишилось.
        for (int i = 0; i <= h.Length; i++)
        {
            int current = i < h.Length ? h[i] : 0;
            while (stack.TryPeek(out int top) && h[top] > current)
            {
                stack.Pop();
                int height = h[top];
                int leftBoundary = stack.TryPeek(out int prev) ? prev : -1; // попередній менший
                int width = i - leftBoundary - 1;                            // між межами (не включно)
                int area = height * width;
                if (trace)
                {
                    Console.WriteLine($"  i={i}: pop bar {top} (h={height}), width={width}, area={area}");
                }

                if (area > best.Area)
                {
                    best = (area, leftBoundary + 1, i - 1);
                }
            }

            stack.Push(i);
        }

        return best;
    }
}
```

**Приклад запуску:**

```text
  i=1: pop bar 0 (h=2), width=1, area=2
  i=4: pop bar 3 (h=6), width=1, area=6
  i=4: pop bar 2 (h=5), width=2, area=10
  i=6: pop bar 5 (h=3), width=1, area=3
  i=6: pop bar 4 (h=2), width=4, area=8
  i=6: pop bar 1 (h=1), width=6, area=6
Max area = 10 (bars 2..3)
Staircase max area = 9
```

### Типові помилки

1. **Зберігати в стеку значення замість індексів** — неможливо обчислити відстань чи ширину.
2. **Строга/нестрога нерівність**: у Stock Span потрібно `<=` (рівні ціни поглинаються), у NGE — `<` (рівний не є «більшим»). Помилка дає зсув на одиницю для дублікатів.
3. **Забутий «сторожовий» 0** у гістограмі — стовпці, що лишились у стеку, не будуть оброблені (для зростаючого масиву відповідь буде 0).

### Міні-вправа 8.1

Напишіть **Previous Smaller Element**: для кожного елемента — найближчий ліворуч строго менший (або `-1`). Вхід: `[4, 5, 2, 10, 8]`.

<details>
<summary>Розв'язок</summary>

```csharp
int[] a = [4, 5, 2, 10, 8];
var result = new int[a.Length];
var stack = new Stack<int>(); // значення, строго зростають від дна до вершини

for (int i = 0; i < a.Length; i++)
{
    // Все, що >= a[i], ніколи не буде «попереднім меншим» для наступних елементів.
    while (stack.TryPeek(out int top) && top >= a[i]) stack.Pop();
    result[i] = stack.TryPeek(out int smaller) ? smaller : -1;
    stack.Push(a[i]);
}

Console.WriteLine(string.Join(", ", result));
```

```text
-1, 4, -1, 2, 2
```

</details>

---

> ## ☕ Перерва 2 (≈105 хв від початку)
>
> 10 хвилин відпочинку. Далі — стек у «побутових» задачах і перехід до черг.

---

## 9. Стек у прикладних задачах

*(≈15 хв)*

### 9.1. Undo/Redo у текстовому редакторі

Класичний патерн **Command** + два стеки:

- `_undo` — виконані команди; `Undo()` знімає команду, викликає `Revert()` і кладе її в `_redo`.
- `_redo` — скасовані команди; `Redo()` повторює і повертає в `_undo`.
- **Будь-яка нова дія очищує `_redo`** — «майбутнє» після розгалуження історії втрачається.

```
 type "Hello"  type " World"  Undo          Undo         Redo         type "!"
 undo: [H]     undo: [H W]    undo: [H]     undo: []     undo: [H]    undo: [H !]
 redo: []      redo: []       redo: [W]     redo: [W H]  redo: [W]    redo: []  ← очищено
```

### Приклад 17. Редактор з undo/redo та обмеженою історією

```csharp
using System.Text;

var editor = new TextEditor(historyLimit: 3);
editor.Execute(new InsertCommand(0, "Hello"));
editor.Execute(new InsertCommand(5, " World"));
editor.Execute(new DeleteCommand(0, 1));
Print("3 edits");
editor.Undo();
Print("undo");
editor.Undo();
Print("undo");
editor.Redo();
Print("redo");
editor.Execute(new InsertCommand(editor.Text.Length, "!"));
Print("type '!'");
Console.WriteLine($"Redo possible: {editor.CanRedo}");

// Перевірка ліміту історії: після 5 дій скасувати можна лише 3.
var limited = new TextEditor(historyLimit: 3);
for (int i = 0; i < 5; i++) limited.Execute(new InsertCommand(i, i.ToString()));
int undone = 0;
while (limited.Undo()) undone++;
Console.WriteLine($"Limited editor: undone {undone}, text '{limited.Text}'");

void Print(string label) =>
    Console.WriteLine($"{label,-10} -> \"{editor.Text}\" (undo={editor.UndoCount}, redo={editor.RedoCount})");

public interface IEditCommand
{
    void Apply(StringBuilder text);
    void Revert(StringBuilder text);
}

public sealed class InsertCommand(int position, string value) : IEditCommand
{
    public void Apply(StringBuilder text) => text.Insert(position, value);
    public void Revert(StringBuilder text) => text.Remove(position, value.Length);
}

public sealed class DeleteCommand(int position, int length) : IEditCommand
{
    private string _removed = ""; // запам'ятовуємо видалене, щоб уміти відновити

    public void Apply(StringBuilder text)
    {
        _removed = text.ToString(position, length);
        text.Remove(position, length);
    }

    public void Revert(StringBuilder text) => text.Insert(position, _removed);
}

public sealed class TextEditor(int historyLimit)
{
    private readonly StringBuilder _text = new();
    // LinkedList як «стек з дном, яке можна обрізати»: вершина — Last, найстаріше — First.
    private readonly LinkedList<IEditCommand> _undo = new();
    private readonly Stack<IEditCommand> _redo = new();

    public string Text => _text.ToString();
    public int UndoCount => _undo.Count;
    public int RedoCount => _redo.Count;
    public bool CanRedo => _redo.Count > 0;

    public void Execute(IEditCommand command)
    {
        command.Apply(_text);
        _undo.AddLast(command);
        if (_undo.Count > historyLimit)
        {
            _undo.RemoveFirst(); // найстаріша дія «забувається»
        }

        _redo.Clear(); // нова гілка історії — redo більше неможливий
    }

    public bool Undo()
    {
        if (_undo.Last is not { } node) return false;
        _undo.RemoveLast();
        node.Value.Revert(_text);
        _redo.Push(node.Value);
        return true;
    }

    public bool Redo()
    {
        if (!_redo.TryPop(out var command)) return false;
        command.Apply(_text);
        _undo.AddLast(command);
        return true;
    }
}
```

**Приклад запуску:**

```text
3 edits    -> "ello World" (undo=3, redo=0)
undo       -> "Hello World" (undo=2, redo=1)
undo       -> "Hello" (undo=1, redo=2)
redo       -> "Hello World" (undo=2, redo=1)
type '!'   -> "Hello World!" (undo=3, redo=0)
Redo possible: False
Limited editor: undone 3, text '01'
```

> **Чому `LinkedList` для undo?** Звичайний `Stack<T>` не дає видалити **дно** (найстарішу дію) за O(1). Обмежена історія — це фактично **дек**.

### 9.2. Кнопки «Назад / Вперед» у браузері

Та сама ідея: поточна сторінка + стек `back` + стек `forward`. Перехід за новим посиланням очищує `forward`.

### Приклад 18. `BrowserHistory`

```csharp
var browser = new BrowserHistory("home.com");
browser.Visit("news.com");
browser.Visit("sport.com");
browser.Visit("weather.com");
Console.WriteLine($"Back:    {browser.Back(1)}");
Console.WriteLine($"Back 5:  {browser.Back(5)}");     // не можна піти далі за першу сторінку
Console.WriteLine($"Forward: {browser.Forward(2)}");
browser.Visit("music.com");                            // forward очищується
Console.WriteLine($"Forward: {browser.Forward(1)}");
Console.WriteLine($"Back 2:  {browser.Back(2)}");

public sealed class BrowserHistory(string homepage)
{
    private readonly Stack<string> _back = new();
    private readonly Stack<string> _forward = new();
    private string _current = homepage;

    public void Visit(string url)
    {
        _back.Push(_current);
        _current = url;
        _forward.Clear();
    }

    public string Back(int steps)
    {
        // Кожен крок: поточна → forward, вершина back → поточна.
        for (int i = 0; i < steps && _back.TryPop(out var previous); i++)
        {
            _forward.Push(_current);
            _current = previous;
        }

        return _current;
    }

    public string Forward(int steps)
    {
        for (int i = 0; i < steps && _forward.TryPop(out var next); i++)
        {
            _back.Push(_current);
            _current = next;
        }

        return _current;
    }
}
```

**Приклад запуску:**

```text
Back:    sport.com
Back 5:  home.com
Forward: sport.com
Forward: music.com
Back 2:  news.com
```

### 9.3. Спрощення шляху Unix

Розбиваємо шлях за `/`. `""` і `"."` пропускаємо, `".."` — `Pop` (якщо є що), інше — `Push`. Результат — вміст стека **від дна до вершини**.

```
 "/a/./b/../../c/"  →  частини: "", a, ., b, .., .., c, ""
 a → [a]   . → [a]   b → [a b]   .. → [a]   .. → []   c → [c]      → "/c"
```

### Приклад 19. `SimplifyPath`

```csharp
string[] paths = ["/a/./b/../c", "/a/./b/../../c/", "/../", "/home//user/./docs/..", "/a/b/c/../../../../x"];
foreach (var path in paths)
{
    Console.WriteLine($"{path,-24} -> {PathTools.Simplify(path)}");
}

public static class PathTools
{
    public static string Simplify(string path)
    {
        var stack = new Stack<string>();
        foreach (var part in path.Split('/'))
        {
            switch (part)
            {
                case "" or ".":
                    break;                  // порожні сегменти (//) і поточна тека — ігноруємо
                case "..":
                    stack.TryPop(out _);    // вгору; з кореня вище не піти — TryPop просто поверне false
                    break;
                default:
                    stack.Push(part);
                    break;
            }
        }

        // Stack перелічується від вершини, тому розвертаємо, щоб отримати шлях від кореня.
        return "/" + string.Join("/", stack.Reverse());
    }
}
```

**Приклад запуску:**

```text
/a/./b/../c              -> /a/c
/a/./b/../../c/          -> /c
/../                     -> /
/home//user/./docs/..    -> /home/user
/a/b/c/../../../../x     -> /x
```

### 9.4. Декодування рядка `k[encoded]`

`"3[a2[c]]"` → `"accaccacc"`. Коли зустрічаємо `[`, **зберігаємо контекст** (що вже побудовано і скільки повторити) на стек і починаємо новий рядок. На `]` — знімаємо контекст і дописуємо поточний рядок `k` разів.

```
 3[a2[c]]
 '3'  k=3
 '['  push ("", 3);  cur=""
 'a'  cur="a"
 '2'  k=2
 '['  push ("a", 2); cur=""
 'c'  cur="c"
 ']'  pop ("a",2) → cur = "a" + "c"×2 = "acc"
 ']'  pop ("",3)  → cur = "" + "acc"×3 = "accaccacc"
```

### Приклад 20. `DecodeString`

```csharp
using System.Text;

string[] inputs = ["3[a]2[bc]", "3[a2[c]]", "2[abc]3[cd]ef", "10[x]", "ab"];
foreach (var s in inputs)
{
    Console.WriteLine($"{s,-14} -> {Decoder.Decode(s)}");
}

public static class Decoder
{
    public static string Decode(string s)
    {
        var stack = new Stack<(StringBuilder Prefix, int Repeat)>();
        var current = new StringBuilder();
        int k = 0;

        foreach (char c in s)
        {
            if (char.IsDigit(c))
            {
                k = k * 10 + (c - '0'); // число може бути багатоцифровим: "10[x]"
            }
            else if (c == '[')
            {
                stack.Push((current, k)); // зберігаємо «що було до дужки» і множник
                current = new StringBuilder();
                k = 0;
            }
            else if (c == ']')
            {
                var (prefix, repeat) = stack.Pop();
                string inner = current.ToString();
                for (int i = 0; i < repeat; i++)
                {
                    prefix.Append(inner);
                }

                current = prefix; // продовжуємо дописувати у зовнішній рядок
            }
            else
            {
                current.Append(c);
            }
        }

        return current.ToString();
    }
}
```

**Приклад запуску:**

```text
3[a]2[bc]      -> aaabcbc
3[a2[c]]       -> accaccacc
2[abc]3[cd]ef  -> abcabccdcdcdef
10[x]          -> xxxxxxxxxx
ab             -> ab
```

### Типові помилки

1. **Забути очистити redo/forward** при новій дії — «Вперед» відкриє сторінку з іншої гілки історії.
2. **`string.Join("/", stack)`** без `Reverse()` — шлях у зворотному порядку.
3. **Однозначні множники** (`k = c - '0'`) у decode — `10[x]` зламається.
4. **Команда Delete без збереження видаленого тексту** — `Revert` неможливий.

### Міні-вправа 9.1

Видаліть **усі суміжні дублікати** у рядку повторно: `"abbaca"` → `"ca"` (видаляємо `bb` → `"aaca"` → видаляємо `aa` → `"ca"`). Складність має бути O(n).

<details>
<summary>Розв'язок</summary>

```csharp
using System.Text;

Console.WriteLine(RemoveDuplicates("abbaca"));
Console.WriteLine(RemoveDuplicates("azxxzy"));

// StringBuilder як стек символів: Append = Push, Length-- = Pop.
static string RemoveDuplicates(string s)
{
    var sb = new StringBuilder();
    foreach (char c in s)
    {
        if (sb.Length > 0 && sb[^1] == c)
        {
            sb.Length--; // пара знайшлась — знищуємо обидва
        }
        else
        {
            sb.Append(c);
        }
    }

    return sb.ToString();
}
```

```text
ca
ay
```

</details>

---

## 10. Черга: BFS

*(≈15 хв)*

### 10.1. Чому BFS знаходить найкоротший шлях?

**Пошук у ширину** (Breadth-First Search) обробляє вершини **хвилями**: спочатку всі на відстані 0, потім 1, потім 2… Черга FIFO гарантує, що вершина на відстані `d+1` ніколи не буде оброблена раніше за вершину на відстані `d`. Тому в **незваженому** графі перше відвідування вершини — по найкоротшому шляху.

```
 Лабіринт (S — старт, E — фініш, # — стіна); числа — відстань BFS

  S . . # .        0 1 2 # 8
  # # . # .        # # 3 # 7
  . . . . .   →    6 5 4 5 6
  . # # # .        7 # # # 7
  . . . # E        8 9 10# 8
```

Для **відновлення шляху** зберігаємо `parent[cell]` — звідки ми прийшли. Від фінішу йдемо по `parent` до старту і розвертаємо (або кладемо у стек!).

### Приклад 21. Найкоротший шлях у лабіринті з відновленням

```csharp
string[] maze =
[
    "S..#.",
    "##.#.",
    ".....",
    ".###.",
    "...#E",
];

var result = MazeSolver.ShortestPath(maze);
if (result is null)
{
    Console.WriteLine("No path");
}
else
{
    Console.WriteLine($"Distance: {result.Value.Distance}");
    Console.WriteLine($"Path: {string.Join(" -> ", result.Value.Path.Select(p => $"({p.Row},{p.Col})"))}");
    foreach (var line in MazeSolver.Draw(maze, result.Value.Path))
    {
        Console.WriteLine(line);
    }
}

Console.WriteLine($"Blocked maze: {(MazeSolver.ShortestPath(["S#", "#E"]) is null ? "No path" : "path")}");

public readonly record struct Cell(int Row, int Col);

public static class MazeSolver
{
    private static readonly (int Dr, int Dc)[] Directions = [(-1, 0), (0, 1), (1, 0), (0, -1)]; // вгору, вправо, вниз, вліво

    public static (int Distance, List<Cell> Path)? ShortestPath(string[] grid)
    {
        int rows = grid.Length, cols = grid[0].Length;
        Cell start = Find(grid, 'S'), end = Find(grid, 'E');

        var parent = new Cell?[rows, cols];
        var visited = new bool[rows, cols];
        var queue = new Queue<Cell>();

        queue.Enqueue(start);
        visited[start.Row, start.Col] = true; // позначаємо ПРИ ДОДАВАННІ, а не при вийманні

        while (queue.TryDequeue(out var cell))
        {
            if (cell == end)
            {
                return (BuildPath(parent, end).Count - 1, BuildPath(parent, end));
            }

            foreach (var (dr, dc) in Directions)
            {
                var next = new Cell(cell.Row + dr, cell.Col + dc);
                bool inside = next.Row >= 0 && next.Row < rows && next.Col >= 0 && next.Col < cols;
                if (inside && grid[next.Row][next.Col] != '#' && !visited[next.Row, next.Col])
                {
                    visited[next.Row, next.Col] = true;
                    parent[next.Row, next.Col] = cell;
                    queue.Enqueue(next);
                }
            }
        }

        return null; // фініш недосяжний
    }

    private static List<Cell> BuildPath(Cell?[,] parent, Cell end)
    {
        // Йдемо від фінішу до старту по parent — стек розверне порядок.
        var stack = new Stack<Cell>();
        for (Cell? c = end; c is not null; c = parent[c.Value.Row, c.Value.Col])
        {
            stack.Push(c.Value);
        }

        return [.. stack]; // перелічення Stack — від вершини, тобто від старту
    }

    public static IEnumerable<string> Draw(string[] grid, List<Cell> path)
    {
        var chars = grid.Select(r => r.ToCharArray()).ToArray();
        foreach (var c in path.Skip(1).SkipLast(1))
        {
            chars[c.Row][c.Col] = '*';
        }

        return chars.Select(r => new string(r));
    }

    private static Cell Find(string[] grid, char target)
    {
        for (int r = 0; r < grid.Length; r++)
        {
            int c = grid[r].IndexOf(target);
            if (c >= 0) return new Cell(r, c);
        }

        throw new ArgumentException($"'{target}' not found.");
    }
}
```

**Приклад запуску:**

```text
Distance: 8
Path: (0,0) -> (0,1) -> (0,2) -> (1,2) -> (2,2) -> (2,3) -> (2,4) -> (3,4) -> (4,4)
S**#.
##*#.
..***
.###*
...#E
Blocked maze: No path
```

### 10.2. Multi-source BFS: гнилі апельсини

Решітка: `0` — порожньо, `1` — свіжий апельсин, `2` — гнилий. Щохвилини гнилий псує 4 сусідів. За скільки хвилин зіпсуються всі (або `-1`)?

**Ідея:** покласти в чергу **всі** гнилі одразу (кілька джерел) — хвилі поширюються одночасно. Кількість хвиль (рівнів) = хвилини.

```
 хв 0        хв 1        хв 2        хв 3        хв 4
 2 1 1       2 2 1       2 2 2       2 2 2       2 2 2
 1 1 0   →   2 1 0   →   2 2 0   →   2 2 0   →   2 2 0
 0 1 1       0 1 1       0 1 1       0 2 1       0 2 2
```

### Приклад 22. Rotting Oranges з обробкою по рівнях

```csharp
int[][] grid1 = [[2, 1, 1], [1, 1, 0], [0, 1, 1]];
int[][] grid2 = [[2, 1, 1], [0, 1, 1], [1, 0, 1]]; // лівий нижній недосяжний
int[][] grid3 = [[0, 2]];

Console.WriteLine($"grid1: {Oranges.MinutesToRot(grid1, verbose: true)}");
Console.WriteLine($"grid2: {Oranges.MinutesToRot(grid2, verbose: false)}");
Console.WriteLine($"grid3: {Oranges.MinutesToRot(grid3, verbose: false)}");

public static class Oranges
{
    public static int MinutesToRot(int[][] grid, bool verbose)
    {
        int rows = grid.Length, cols = grid[0].Length;
        var queue = new Queue<(int R, int C)>();
        int fresh = 0;

        // 1) Усі джерела — одразу в чергу.
        for (int r = 0; r < rows; r++)
        {
            for (int c = 0; c < cols; c++)
            {
                if (grid[r][c] == 2) queue.Enqueue((r, c));
                else if (grid[r][c] == 1) fresh++;
            }
        }

        int minutes = 0;
        (int, int)[] dirs = [(1, 0), (-1, 0), (0, 1), (0, -1)];

        // 2) Обробка ПО РІВНЯХ: фіксуємо розмір черги на початку хвилини.
        while (queue.Count > 0 && fresh > 0)
        {
            int levelSize = queue.Count;
            var rottedNow = new List<string>();
            for (int i = 0; i < levelSize; i++)
            {
                var (r, c) = queue.Dequeue();
                foreach (var (dr, dc) in dirs)
                {
                    int nr = r + dr, nc = c + dc;
                    if (nr >= 0 && nr < rows && nc >= 0 && nc < cols && grid[nr][nc] == 1)
                    {
                        grid[nr][nc] = 2; // псуємо одразу, щоб не додати двічі
                        fresh--;
                        queue.Enqueue((nr, nc));
                        rottedNow.Add($"({nr},{nc})");
                    }
                }
            }

            minutes++;
            if (verbose) Console.WriteLine($"  minute {minutes}: {string.Join(" ", rottedNow)}");
        }

        return fresh == 0 ? minutes : -1;
    }
}
```

**Приклад запуску:**

```text
  minute 1: (1,0) (0,1)
  minute 2: (1,1) (0,2)
  minute 3: (2,1)
  minute 4: (2,2)
grid1: 4
grid2: -1
grid3: 0
```

### 10.3. Обробка по рівнях у дереві

Той самий трюк `levelSize = queue.Count` дає **level-order** обхід дерева: друк по рядках, середнє на рівні, «вид справа», зигзаг.

### Приклад 23. Level-order: рівні, правий вид, зигзаг

```csharp
//            1
//          /   \
//         2     3
//        / \     \
//       4   5     6
//          /
//         7
var root = new TreeNode(1,
    new TreeNode(2, new TreeNode(4), new TreeNode(5, new TreeNode(7))),
    new TreeNode(3, null, new TreeNode(6)));

var levels = LevelOrder.Levels(root);
for (int i = 0; i < levels.Count; i++)
{
    Console.WriteLine($"level {i}: {string.Join(" ", levels[i])}  avg={levels[i].Average():F2}");
}

Console.WriteLine($"right view: {string.Join(" ", levels.Select(l => l[^1]))}");
Console.WriteLine($"zigzag:     {string.Join(" | ", levels.Select((l, i) => string.Join(" ", i % 2 == 0 ? l : Enumerable.Reverse(l))))}");

public sealed record TreeNode(int Value, TreeNode? Left = null, TreeNode? Right = null);

public static class LevelOrder
{
    public static List<List<int>> Levels(TreeNode? root)
    {
        var result = new List<List<int>>();
        if (root is null) return result;

        var queue = new Queue<TreeNode>();
        queue.Enqueue(root);
        while (queue.Count > 0)
        {
            int size = queue.Count;          // стільки вузлів на ПОТОЧНОМУ рівні
            var level = new List<int>(size);
            for (int i = 0; i < size; i++)
            {
                var node = queue.Dequeue();
                level.Add(node.Value);
                if (node.Left is not null) queue.Enqueue(node.Left);   // діти — вже наступний рівень
                if (node.Right is not null) queue.Enqueue(node.Right);
            }

            result.Add(level);
        }

        return result;
    }
}
```

**Приклад запуску:**

```text
level 0: 1  avg=1.00
level 1: 2 3  avg=2.50
level 2: 4 5 6  avg=5.00
level 3: 7  avg=7.00
right view: 1 3 6 7
zigzag:     1 | 3 2 | 4 5 6 | 7
```

### Типові помилки

1. **Позначати `visited` при вийманні** з черги — клітинка потрапить у чергу багато разів (експоненційний ріст на решітці).
2. **`while (queue.Count > 0) { for (i < queue.Count) ... }`** — `queue.Count` змінюється всередині циклу! Спершу збережіть `levelSize`.
3. **BFS на зваженому графі** — перше відвідування вже не найкоротше. Потрібна Дейкстра (розділ 13) або 0-1 BFS з деком.
4. **Використання стека замість черги** «бо так само працює» — вийде DFS, який не гарантує найкоротшого шляху.

### Міні-вправа 10.1

Знайдіть мінімальну кількість ходів **коня** з `(0,0)` до `(7,7)` на шахівниці 8×8.

<details>
<summary>Розв'язок</summary>

```csharp
Console.WriteLine(KnightMoves(8, (0, 0), (7, 7)));
Console.WriteLine(KnightMoves(8, (0, 0), (1, 2)));

static int KnightMoves(int n, (int R, int C) from, (int R, int C) to)
{
    (int, int)[] jumps = [(1, 2), (2, 1), (2, -1), (1, -2), (-1, -2), (-2, -1), (-2, 1), (-1, 2)];
    var dist = new int[n, n];
    for (int r = 0; r < n; r++) for (int c = 0; c < n; c++) dist[r, c] = -1;

    var queue = new Queue<(int R, int C)>();
    queue.Enqueue(from);
    dist[from.R, from.C] = 0;

    while (queue.TryDequeue(out var cell))
    {
        if (cell == to) return dist[cell.R, cell.C];
        foreach (var (dr, dc) in jumps)
        {
            int r = cell.R + dr, c = cell.C + dc;
            if (r >= 0 && r < n && c >= 0 && c < n && dist[r, c] < 0)
            {
                dist[r, c] = dist[cell.R, cell.C] + 1; // масив відстаней замінює visited
                queue.Enqueue((r, c));
            }
        }
    }

    return -1;
}
```

```text
6
1
```

</details>

---

## 11. Черга: симуляції

*(≈15 хв)*

### 11.1. Round-robin планувальник

Операційна система дає кожному процесу **квант часу** (quantum). Якщо процес не встиг — він стає в **кінець черги**. Це чесно (ніхто не голодує) і просто.

```
 quantum = 3;  A(5) B(2) C(4)

 t=0..3   A  (лишилось 2)  → у кінець    черга: B C A
 t=3..5   B  (готово)                    черга: C A
 t=5..8   C  (лишилось 1)  → у кінець    черга: A C
 t=8..10  A  (готово)                    черга: C
 t=10..11 C  (готово)
```

Метрики: **turnaround** = завершення − прибуття; **waiting** = turnaround − час виконання.

### Приклад 24. Round-robin з прибуттям процесів

```csharp
Job[] jobs =
[
    new("A", Arrival: 0, Burst: 5),
    new("B", Arrival: 1, Burst: 3),
    new("C", Arrival: 2, Burst: 1),
    new("D", Arrival: 3, Burst: 2),
    new("E", Arrival: 4, Burst: 3),
];

var stats = RoundRobin.Run(jobs, quantum: 2);
Console.WriteLine("Job  Finish  Turnaround  Waiting");
foreach (var s in stats)
{
    Console.WriteLine($"{s.Name,-4} {s.Finish,6} {s.Turnaround,11} {s.Waiting,8}");
}

Console.WriteLine($"Average waiting: {stats.Average(s => s.Waiting):F2}");

public sealed record Job(string Name, int Arrival, int Burst);

public sealed record JobStats(string Name, int Finish, int Turnaround, int Waiting);

public static class RoundRobin
{
    public static List<JobStats> Run(Job[] jobs, int quantum)
    {
        var pending = new Queue<Job>(jobs.OrderBy(j => j.Arrival)); // ще не прибули
        var ready = new Queue<(Job Job, int Remaining)>();           // черга готових
        var stats = new List<JobStats>();
        var timeline = new List<string>();
        int time = 0;

        while (pending.Count > 0 || ready.Count > 0)
        {
            AdmitArrived(); // процеси, що прибули до цього моменту

            if (!ready.TryDequeue(out var current))
            {
                time = pending.Peek().Arrival; // процесор простоює до наступного прибуття
                continue;
            }

            int run = Math.Min(quantum, current.Remaining);
            timeline.Add($"{time}-{time + run}:{current.Job.Name}");
            time += run;

            // ВАЖЛИВО: спершу додаємо тих, хто прибув під час кванту, і лише ПОТІМ
            // ставимо перерваний процес у кінець — це стандартна домовленість.
            AdmitArrived();

            if (current.Remaining > run)
            {
                ready.Enqueue((current.Job, current.Remaining - run));
            }
            else
            {
                int turnaround = time - current.Job.Arrival;
                stats.Add(new(current.Job.Name, time, turnaround, turnaround - current.Job.Burst));
            }
        }

        Console.WriteLine($"Timeline: {string.Join(" ", timeline)}");
        return stats.OrderBy(s => s.Name).ToList();

        void AdmitArrived()
        {
            while (pending.TryPeek(out var job) && job.Arrival <= time)
            {
                ready.Enqueue((pending.Dequeue(), job.Burst));
            }
        }
    }
}
```

**Приклад запуску:**

```text
Timeline: 0-2:A 2-4:B 4-5:C 5-7:A 7-9:D 9-11:E 11-12:B 12-13:A 13-14:E
Job  Finish  Turnaround  Waiting
A        13          13        8
B        12          11        8
C         5           3        2
D         9           6        4
E        14          10        7
Average waiting: 5.80
```

### 11.2. Симуляція банку з кількома касами

**Дискретно-подієве моделювання**: час іде «тіками» (хвилинами). Щохвилини з певною ймовірністю приходить клієнт і стає в чергу; вільна каса бере наступного. Використовуємо `new Random(seed)` — однаковий seed дає однакову послідовність, тож експеримент **відтворюваний**.

> `new Random(42)` у .NET використовує стабільний алгоритм для явного seed, тож вивід нижче однаковий на всіх машинах. `Random.Shared` і `new Random()` без seed — ні.

### Приклад 25. Банк: 1 каса проти 2 кас

```csharp
foreach (int tellers in new[] { 1, 2 })
{
    var report = BankSimulation.Run(tellers, minutes: 480, arrivalProbability: 0.3, seed: 42);
    Console.WriteLine($"Tellers={tellers}: served={report.Served}, avg wait={report.AverageWait:F1} min, " +
                      $"max wait={report.MaxWait} min, max queue={report.MaxQueueLength}, left in queue={report.LeftInQueue}");
}

public sealed record BankReport(int Served, double AverageWait, int MaxWait, int MaxQueueLength, int LeftInQueue);

public static class BankSimulation
{
    public static BankReport Run(int tellers, int minutes, double arrivalProbability, int seed)
    {
        var random = new Random(seed);
        var queue = new Queue<int>();          // зберігаємо ХВИЛИНУ прибуття клієнта
        var busyUntil = new int[tellers];      // до якої хвилини зайнята кожна каса
        int served = 0, totalWait = 0, maxWait = 0, maxQueue = 0;

        for (int t = 0; t < minutes; t++)
        {
            // 1) Прибуття: з імовірністю p новий клієнт.
            if (random.NextDouble() < arrivalProbability)
            {
                queue.Enqueue(t);
            }

            // 2) Кожна вільна каса бере наступного з черги.
            for (int i = 0; i < tellers; i++)
            {
                if (busyUntil[i] <= t && queue.TryDequeue(out int arrival))
                {
                    int wait = t - arrival;
                    totalWait += wait;
                    maxWait = Math.Max(maxWait, wait);
                    served++;
                    busyUntil[i] = t + random.Next(2, 6); // обслуговування 2..5 хв
                }
            }

            maxQueue = Math.Max(maxQueue, queue.Count);
        }

        return new BankReport(served, served == 0 ? 0 : (double)totalWait / served, maxWait, maxQueue, queue.Count);
    }
}
```

**Приклад запуску:**

```text
Tellers=1: served=139, avg wait=40.6 min, max wait=72 min, max queue=21, left in queue=16
Tellers=2: served=157, avg wait=0.4 min, max wait=4 min, max queue=2, left in queue=0
```

Середній час обслуговування 3.5 хв → одна каса обслуговує ≈0.29 клієнта/хв, а приходить 0.3/хв. Навантаження **ρ ≈ 1.05 > 1** — черга росте необмежено. Друга каса знижує ρ удвічі, і черга практично зникає. Це ключовий висновок **теорії масового обслуговування**: при ρ → 1 час очікування зростає нелінійно.

### 11.3. Черга друку з обмеженою місткістю

Принтер має буфер на `N` завдань. Якщо буфер повний — нове завдання **відхиляється** (backpressure). Кожне завдання друкується зі швидкістю 1 сторінка/тік.

### Приклад 26. Print queue з відмовами

```csharp
var printer = new PrintQueue(capacity: 2);
var random = new Random(7);

for (int tick = 0; tick < 12; tick++)
{
    if (random.Next(100) < 80) // 80% імовірність нового документа
    {
        var doc = new PrintJob($"doc{tick}", Pages: random.Next(1, 4));
        bool accepted = printer.Submit(doc);
        Console.WriteLine($"t={tick,2}: submit {doc.Name}({doc.Pages}p) {(accepted ? "accepted" : "REJECTED")}");
    }

    if (printer.Tick() is { } finished)
    {
        Console.WriteLine($"t={tick,2}: printed {finished}");
    }
}

Console.WriteLine($"Rejected: {printer.Rejected}, still queued: {printer.Queued}");

public sealed record PrintJob(string Name, int Pages);

public sealed class PrintQueue(int capacity)
{
    private readonly Queue<PrintJob> _queue = new(capacity);
    private int _pagesLeft; // скільки лишилось надрукувати в поточного документа

    public int Rejected { get; private set; }
    public int Queued => _queue.Count;

    public bool Submit(PrintJob job)
    {
        if (_queue.Count >= capacity)
        {
            Rejected++;
            return false; // буфер повний — відмова замість нескінченного росту
        }

        _queue.Enqueue(job);
        return true;
    }

    // Один тік принтера: друкує сторінку поточного документа (голови черги).
    public string? Tick()
    {
        if (!_queue.TryPeek(out var current)) return null;
        if (_pagesLeft == 0) _pagesLeft = current.Pages; // починаємо новий документ

        _pagesLeft--;
        if (_pagesLeft > 0) return null;

        _queue.Dequeue(); // документ залишає чергу лише ПІСЛЯ повного друку
        return current.Name;
    }
}
```

**Приклад запуску:**

```text
t= 0: submit doc0(3p) accepted
t= 1: submit doc1(1p) accepted
t= 2: submit doc2(3p) REJECTED
t= 2: printed doc0
t= 3: submit doc3(3p) accepted
t= 3: printed doc1
t= 6: submit doc6(3p) accepted
t= 6: printed doc3
t= 7: submit doc7(1p) accepted
t= 9: submit doc9(3p) REJECTED
t= 9: printed doc6
t=10: submit doc10(3p) accepted
t=10: printed doc7
t=11: submit doc11(3p) accepted
Rejected: 2, still queued: 2
```

### Типові помилки

1. **`new Random()` без seed** у тестах чи лекційних прикладах — результат щоразу інший.
2. **Порядок подій у тіці** (прибуття → обслуговування або навпаки) змінює результати. Зафіксуйте й задокументуйте домовленість.
3. **Необмежена черга** у реальній системі при ρ > 1 — рано чи пізно `OutOfMemoryException`. Потрібні ліміти та відмови (backpressure).
4. **Round-robin: повернення перерваного процесу перед новими прибулими** — інший (теж валідний, але несумісний з підручником) розклад.

---

## 12. Дек і ковзаючі вікна

*(≈15 хв)*

### 12.1. Максимум у ковзаючому вікні

Для масиву і вікна `k` знайти максимум у кожному вікні. Наївно — O(n·k). З **монотонним деком** — O(n).

**Інваріант:** дек зберігає індекси, значення по них **спадають** від голови до хвоста. Голова — максимум поточного вікна.

1. Видалити з **голови** індекс, що вийшов за вікно (`i - k`).
2. Видалити з **хвоста** всі індекси зі значенням ≤ `a[i]` — вони вже ніколи не стануть максимумом (новий елемент і більший, і «живе» довше).
3. Додати `i` у хвіст. Якщо `i >= k - 1` — відповідь `a[голова]`.

```
 a = [1, 3, -1, -3, 5, 3, 6, 7], k = 3

 i  a[i]  дек (значення)   max
 0   1    [1]
 1   3    [3]               
 2  -1    [3 -1]            3
 3  -3    [3 -1 -3]         3
 4   5    [5]   (3 вийшов)  5
 5   3    [5 3]             5
 6   6    [6]               6
 7   7    [7]               7
```

### Приклад 27. Sliding Window Maximum на `RingDeque<int>`

```csharp
int[] a = [1, 3, -1, -3, 5, 3, 6, 7];
Console.WriteLine($"max  k=3: {string.Join(" ", Window.Maximums(a, 3))}");
Console.WriteLine($"max  k=1: {string.Join(" ", Window.Maximums(a, 1))}");
Console.WriteLine($"max  k=8: {string.Join(" ", Window.Maximums(a, 8))}");

public static class Window
{
    public static List<int> Maximums(int[] a, int k)
    {
        var result = new List<int>();
        var deque = new RingDeque<int>(); // індекси; a[...] спадають від голови до хвоста

        for (int i = 0; i < a.Length; i++)
        {
            // 1) Голова вийшла за ліву межу вікна [i-k+1, i].
            if (deque.Count > 0 && deque.PeekFront() <= i - k)
            {
                deque.PopFront();
            }

            // 2) Хвости, не більші за a[i], безнадійні — прибираємо.
            while (deque.Count > 0 && a[deque.PeekBack()] <= a[i])
            {
                deque.PopBack();
            }

            deque.PushBack(i);

            // 3) Вікно сформоване — голова дає максимум.
            if (i >= k - 1)
            {
                result.Add(a[deque.PeekFront()]);
            }
        }

        return result;
    }
}

// Спрощений кільцевий дек з розділу 4.4 (лише потрібні операції).
public sealed class RingDeque<T>
{
    private T[] _buffer = new T[4];
    private int _head;

    public int Count { get; private set; }

    public void PushBack(T item)
    {
        if (Count == _buffer.Length) Grow();
        _buffer[(_head + Count) % _buffer.Length] = item;
        Count++;
    }

    public T PopFront()
    {
        T item = PeekFront();
        _head = (_head + 1) % _buffer.Length;
        Count--;
        return item;
    }

    public T PopBack()
    {
        T item = PeekBack();
        Count--;
        return item;
    }

    public T PeekFront() => Count > 0 ? _buffer[_head] : throw new InvalidOperationException("Deque is empty.");

    public T PeekBack() => Count > 0
        ? _buffer[(_head + Count - 1) % _buffer.Length]
        : throw new InvalidOperationException("Deque is empty.");

    private void Grow()
    {
        var bigger = new T[_buffer.Length * 2];
        for (int i = 0; i < Count; i++) bigger[i] = _buffer[(_head + i) % _buffer.Length];
        _buffer = bigger;
        _head = 0;
    }
}
```

**Приклад запуску:**

```text
max  k=3: 3 3 5 5 6 7
max  k=1: 1 3 -1 -3 5 3 6 7
max  k=8: 7
```

**Складність:** кожен індекс один раз додається і щонайбільше один раз видаляється → O(n) час, O(k) пам'ять.

### 12.2. Recent Counter і Rate Limiter (sliding log)

**Recent Counter:** скільки запитів надійшло за останні 3000 мс (включно). Черга часових міток: додаємо нову, з голови прибираємо застарілі.

**Rate limiter «ковзаючий журнал»:** не більше `N` запитів за вікно `W` **для кожного клієнта**. Точний, але пам'ять O(N) на клієнта. (Альтернативи: fixed window, token bucket; у .NET є `System.Threading.RateLimiting`.)

```
 limit = 3 за 10 с

 t:   0    1    2    5    10   11   12
      ✓    ✓    ✓    ✗    ✓    ✓    ✓
                     │    │
                     │    └── t=10: мітка 0 застаріла (0 <= 10-10) → лишились [1 2] → дозволено
                     └── t=5: у журналі [0 1 2] — 3 ≥ limit → відмова
```

### Приклад 28. `RecentCounter` та `SlidingLogRateLimiter`

```csharp
var counter = new RecentCounter(windowMs: 3000);
foreach (int t in new[] { 1, 100, 3001, 3002, 7000 })
{
    Console.WriteLine($"ping({t}) -> {counter.Ping(t)}");
}

var limiter = new SlidingLogRateLimiter(limit: 3, window: TimeSpan.FromSeconds(10));
var start = new DateTime(2025, 1, 1, 12, 0, 0, DateTimeKind.Utc);
(string User, int Second)[] requests =
[
    ("alice", 0), ("alice", 1), ("bob", 1), ("alice", 2), ("alice", 5),
    ("alice", 10), ("alice", 11), ("alice", 12), ("alice", 12), ("bob", 12),
];

foreach (var (user, second) in requests)
{
    bool allowed = limiter.TryAcquire(user, start.AddSeconds(second));
    Console.WriteLine($"t={second,2}s {user,-5} -> {(allowed ? "allowed" : "rejected")}");
}

public sealed class RecentCounter(int windowMs)
{
    private readonly Queue<int> _pings = new();

    public int Ping(int t)
    {
        _pings.Enqueue(t);
        // Мітки строго старші за [t - window] вже поза вікном.
        while (_pings.Peek() < t - windowMs)
        {
            _pings.Dequeue();
        }

        return _pings.Count;
    }
}

public sealed class SlidingLogRateLimiter(int limit, TimeSpan window)
{
    // Окремий журнал (черга часових міток) для кожного клієнта.
    private readonly Dictionary<string, Queue<DateTime>> _logs = new();

    public bool TryAcquire(string clientId, DateTime now)
    {
        if (!_logs.TryGetValue(clientId, out var log))
        {
            log = new Queue<DateTime>();
            _logs[clientId] = log;
        }

        // Вікно (now - window, now]: мітки <= now - window видаляємо з голови.
        while (log.TryPeek(out var oldest) && oldest <= now - window)
        {
            log.Dequeue();
        }

        if (log.Count >= limit)
        {
            return false; // відхилений запит НЕ записуємо в журнал
        }

        log.Enqueue(now);
        return true;
    }
}
```

**Приклад запуску:**

```text
ping(1) -> 1
ping(100) -> 2
ping(3001) -> 3
ping(3002) -> 3
ping(7000) -> 1
t= 0s alice -> allowed
t= 1s alice -> allowed
t= 1s bob   -> allowed
t= 2s alice -> allowed
t= 5s alice -> rejected
t=10s alice -> allowed
t=11s alice -> allowed
t=12s alice -> allowed
t=12s alice -> rejected
t=12s bob   -> allowed
```

### 12.3. Кільцевий логер «останні N повідомлень»

Кільцевий буфер фіксованого розміру, що **перезаписує** найстаріше — ідеальний для «чорної скриньки»: зберігати останні N записів логу в пам'яті і скинути їх при аварії. Пам'ять O(N) незалежно від тривалості роботи, без алокацій після старту.

### Приклад 29. `RingBufferLogger`

```csharp
var logger = new RingBufferLogger(capacity: 4);
for (int i = 1; i <= 7; i++)
{
    logger.Log($"event #{i}");
}

Console.WriteLine($"Total logged: {logger.TotalLogged}, kept: {logger.Count}");
Console.WriteLine("Crash dump (oldest → newest):");
foreach (var line in logger.Snapshot())
{
    Console.WriteLine($"  {line}");
}

public sealed class RingBufferLogger(int capacity)
{
    private readonly string[] _buffer = new string[capacity];
    private int _next;   // куди писати наступний запис

    public int Count { get; private set; }
    public long TotalLogged { get; private set; }

    public void Log(string message)
    {
        _buffer[_next] = $"[{TotalLogged + 1:D3}] {message}";
        _next = (_next + 1) % capacity; // по колу: найстаріший запис буде перезаписано
        Count = Math.Min(Count + 1, capacity);
        TotalLogged++;
    }

    public IEnumerable<string> Snapshot()
    {
        // Найстаріший запис: якщо буфер повний — там, куди писатимемо далі; інакше — з 0.
        int oldest = Count == capacity ? _next : 0;
        for (int i = 0; i < Count; i++)
        {
            yield return _buffer[(oldest + i) % capacity];
        }
    }
}
```

**Приклад запуску:**

```text
Total logged: 7, kept: 4
Crash dump (oldest → newest):
  [004] event #4
  [005] event #5
  [006] event #6
  [007] event #7
```

### Типові помилки

1. **`if` замість `while`** при видаленні застарілих міток — видалиться лише одна, лічильник буде завищений.
2. **Записувати відхилені запити в журнал** — клієнт, що «спамить», ніколи не розблокується.
3. **Межі вікна `<` проти `<=`** — визначте, чи включна межа, і перевірте тестом на граничному значенні (як `ping(3001)` вище).
4. **Максимум у вікні з `PriorityQueue`** без видалення застарілих — O(n log n) і складна логіка «лінивого» видалення; монотонний дек простіший і швидший.

### Міні-вправа 12.1

Модифікуйте `Window.Maximums`, щоб знаходити **мінімуми** у вікні `k = 2` для `[4, 2, 12, 3, 8]`. Що змінюється?

<details>
<summary>Розв'язок</summary>

Змінюється лише знак порівняння з хвостом: прибираємо значення **≥** `a[i]` (дек стає **зростаючим**).

```csharp
int[] a = [4, 2, 12, 3, 8];
int k = 2;
var deque = new LinkedList<int>(); // тут для різноманіття — LinkedList як дек
var mins = new List<int>();

for (int i = 0; i < a.Length; i++)
{
    if (deque.Count > 0 && deque.First!.Value <= i - k) deque.RemoveFirst();
    while (deque.Count > 0 && a[deque.Last!.Value] >= a[i]) deque.RemoveLast();
    deque.AddLast(i);
    if (i >= k - 1) mins.Add(a[deque.First!.Value]);
}

Console.WriteLine(string.Join(" ", mins));
```

```text
2 2 3 3
```

</details>

---

> ## ☕ Перерва 3 (≈150 хв від початку)
>
> 10 хвилин відпочинку. Попереду — купи, черги з пріоритетом і трохи конкурентності.

---

## 13. Черга з пріоритетом і бінарна купа

*(≈20 хв)*

### 13.1. Від АТД до купи

**Черга з пріоритетом** віддає не найстаріший, а **найважливіший** (з мінімальним пріоритетом) елемент.

| Реалізація | Enqueue | Dequeue (min) | Peek |
|------------|---------|---------------|------|
| Невідсортований масив | O(1) | O(n) | O(n) |
| Відсортований масив | O(n) | O(1) | O(1) |
| **Бінарна купа** | **O(log n)** | **O(log n)** | **O(1)** |

**Бінарна мін-купа** — повне бінарне дерево, у якому кожен батько ≤ своїх дітей. Зберігається **в масиві** без вказівників:

```
 індекси:  parent(i) = (i - 1) / 2     left(i) = 2i + 1     right(i) = 2i + 2

                 1 [0]
              /        \
          3 [1]        2 [2]
          /   \        /
      7 [3]  4 [4]  5 [5]

 масив: [1, 3, 2, 7, 4, 5]
```

**Enqueue (sift up):** кладемо в кінець масиву і «спливаємо» вгору, поки батько більший.
**Dequeue (sift down):** забираємо корінь, на його місце — останній елемент, і «топимо» вниз, міняючи з **меншою** з дітей.

```
 Enqueue(0):                          Dequeue():
 [1,3,2,7,4,5,0]                      корінь 0 забрали; останній (2) → в корінь
  0 на [6], батько [2]=2 > 0 → swap   [2,3,1,7,4,5]
 [1,3,0,7,4,5,2]                      діти 3 і 1 → менша 1 < 2 → swap
  батько [0]=1 > 0 → swap             [1,3,2,7,4,5]
 [0,3,1,7,4,5,2]                      [2] діти [5]=5 → 2 < 5 → стоп
```

**Heapify** (побудова купи з масиву) — sift down від `n/2 - 1` до 0 — працює за **O(n)**, а не O(n log n): більшість вузлів — листки або майже листки і топляться на мало рівнів.

### Приклад 30. `BinaryHeap<T>` з нуля + heapsort

```csharp
var heap = new BinaryHeap<int>();
foreach (int x in new[] { 5, 3, 8, 1, 4, 2, 7 })
{
    heap.Push(x);
    Console.WriteLine($"push {x}: [{string.Join(", ", heap.RawArray())}]");
}

Console.Write("pop order: ");
while (heap.Count > 0) Console.Write($"{heap.Pop()} ");
Console.WriteLine();

// Heapify за O(n) і max-heap через компаратор.
var maxHeap = new BinaryHeap<int>([4, 10, 3, 5, 1], Comparer<int>.Create((a, b) => b.CompareTo(a)));
Console.WriteLine($"max-heap after heapify: [{string.Join(", ", maxHeap.RawArray())}], top = {maxHeap.Peek()}");

// Рядки: купа працює з будь-яким T, для якого є компаратор.
var words = new BinaryHeap<string>(["pear", "apple", "fig", "banana"], StringComparer.Ordinal);
Console.WriteLine($"smallest word: {words.Pop()}, next: {words.Pop()}");

public sealed class BinaryHeap<T>
{
    private readonly List<T> _items;
    private readonly IComparer<T> _comparer;

    public BinaryHeap(IComparer<T>? comparer = null)
    {
        _items = [];
        _comparer = comparer ?? Comparer<T>.Default;
    }

    // Побудова купи з готової колекції за O(n).
    public BinaryHeap(IEnumerable<T> items, IComparer<T>? comparer = null)
    {
        _items = [.. items];
        _comparer = comparer ?? Comparer<T>.Default;
        for (int i = _items.Count / 2 - 1; i >= 0; i--)
        {
            SiftDown(i); // листки (індекси >= n/2) вже є купами розміру 1
        }
    }

    public int Count => _items.Count;

    public IReadOnlyList<T> RawArray() => _items;

    public T Peek() => _items.Count > 0 ? _items[0] : throw new InvalidOperationException("Heap is empty.");

    public void Push(T item)
    {
        _items.Add(item);
        SiftUp(_items.Count - 1);
    }

    public T Pop()
    {
        T top = Peek();
        int last = _items.Count - 1;
        _items[0] = _items[last];   // останній елемент — у корінь
        _items.RemoveAt(last);
        if (_items.Count > 0)
        {
            SiftDown(0);
        }

        return top;
    }

    private void SiftUp(int i)
    {
        while (i > 0)
        {
            int parent = (i - 1) / 2;
            if (_comparer.Compare(_items[i], _items[parent]) >= 0)
            {
                break; // батько не більший — властивість купи відновлена
            }

            (_items[i], _items[parent]) = (_items[parent], _items[i]);
            i = parent;
        }
    }

    private void SiftDown(int i)
    {
        int n = _items.Count;
        while (true)
        {
            int left = 2 * i + 1, right = left + 1, smallest = i;
            if (left < n && _comparer.Compare(_items[left], _items[smallest]) < 0) smallest = left;
            if (right < n && _comparer.Compare(_items[right], _items[smallest]) < 0) smallest = right;
            if (smallest == i)
            {
                break; // обидві дитини не менші — стоп
            }

            (_items[i], _items[smallest]) = (_items[smallest], _items[i]);
            i = smallest;
        }
    }
}
```

**Приклад запуску:**

```text
push 5: [5]
push 3: [3, 5]
push 8: [3, 5, 8]
push 1: [1, 3, 8, 5]
push 4: [1, 3, 8, 5, 4]
push 2: [1, 3, 2, 5, 4, 8]
push 7: [1, 3, 2, 5, 4, 8, 7]
pop order: 1 2 3 4 5 7 8 
max-heap after heapify: [10, 5, 3, 4, 1], top = 10
smallest word: apple, next: banana
```

### 13.2. `PriorityQueue<TElement, TPriority>`: min і max

`PriorityQueue` у .NET — **мін**-купа. Для макс-купи передаємо компаратор, що інвертує порівняння. Пріоритет — окремий від елемента параметр, тож можна мати `PriorityQueue<string, (int Severity, DateTime Created)>` з кортежем-пріоритетом (порівнюється лексикографічно).

### Приклад 31. Мін-, макс-купа та складений пріоритет

```csharp
// 1) Min: найменший пріоритет — першим.
var minPq = new PriorityQueue<string, int>([("B", 2), ("C", 3), ("A", 1)]);
Console.WriteLine($"min order: {DrainAll(minPq)}");

// 2) Max: інвертований компаратор.
var maxPq = new PriorityQueue<string, int>(Comparer<int>.Create((x, y) => y.CompareTo(x)));
maxPq.EnqueueRange([("B", 2), ("C", 3), ("A", 1)]);
Console.WriteLine($"max order: {DrainAll(maxPq)}");

// 3) Складений пріоритет: спершу серйозність (менше = важливіше), потім порядковий номер
//    — це робить чергу СТАБІЛЬНОЮ для однакової серйозності.
var tickets = new PriorityQueue<string, (int Severity, long Sequence)>();
long seq = 0;
foreach (var (title, severity) in new[] { ("typo", 3), ("outage", 1), ("slow page", 2), ("data loss", 1), ("color", 3) })
{
    tickets.Enqueue(title, (severity, seq++));
}

Console.Write("tickets:   ");
while (tickets.TryDequeue(out var t, out var p)) Console.Write($"{t}(sev {p.Severity}) ");
Console.WriteLine();

// 4) EnqueueDequeue: додати і одразу забрати мінімум — ефективніше за дві операції.
var pq = new PriorityQueue<int, int>([(5, 5), (7, 7)]);
Console.WriteLine($"EnqueueDequeue(3) -> {pq.EnqueueDequeue(3, 3)}; EnqueueDequeue(6) -> {pq.EnqueueDequeue(6, 6)}");

static string DrainAll(PriorityQueue<string, int> pq)
{
    var parts = new List<string>();
    while (pq.TryDequeue(out var element, out var priority)) parts.Add($"{element}:{priority}");
    return string.Join(" ", parts);
}
```

**Приклад запуску:**

```text
min order: A:1 B:2 C:3
max order: C:3 B:2 A:1
tickets:   outage(sev 1) data loss(sev 1) slow page(sev 2) typo(sev 3) color(sev 3) 
EnqueueDequeue(3) -> 3; EnqueueDequeue(6) -> 5
```

### 13.3. Top-K за O(n log k)

Задача: k найбільших чисел із потоку з n елементів. Сортування — O(n log n) і потребує всіх даних у пам'яті. Краще: **мін-купа розміру k**. Корінь — найменший з поточних «топ-k»; новий елемент заходить, лише якщо він більший за корінь.

### Приклад 32. Top-K найбільших і K найчастіших слів

```csharp
int[] stream = [7, 10, 4, 3, 20, 15, 8, 1, 25, 9];
Console.WriteLine($"top-3 largest: {string.Join(" ", TopK.Largest(stream, 3))}");

string text = "the quick brown fox jumps over the lazy dog the fox barks and the dog runs";
Console.WriteLine($"top-2 words:   {string.Join(", ", TopK.FrequentWords(text.Split(' '), 2))}");

public static class TopK
{
    public static IEnumerable<int> Largest(IEnumerable<int> source, int k)
    {
        var heap = new PriorityQueue<int, int>(k + 1);
        foreach (int x in source)
        {
            if (heap.Count < k)
            {
                heap.Enqueue(x, x);
            }
            else if (x > heap.Peek())
            {
                heap.DequeueEnqueue(x, x); // викидаємо найменший з топу, додаємо новий
            }
        }

        // Купа не впорядкована — сортуємо k елементів у кінці (O(k log k)).
        return heap.UnorderedItems.Select(item => item.Element).OrderDescending();
    }

    public static IEnumerable<string> FrequentWords(string[] words, int k)
    {
        var counts = new Dictionary<string, int>();
        foreach (var w in words) counts[w] = counts.GetValueOrDefault(w) + 1;

        // Пріоритет — (частота, слово): при рівній частоті «гіршим» вважаємо алфавітно більше слово.
        var comparer = Comparer<(int Count, string Word)>.Create((a, b) =>
            a.Count != b.Count ? a.Count.CompareTo(b.Count) : string.CompareOrdinal(b.Word, a.Word));
        var heap = new PriorityQueue<string, (int Count, string Word)>(comparer);

        foreach (var (word, count) in counts)
        {
            heap.Enqueue(word, (count, word));
            if (heap.Count > k) heap.Dequeue(); // тримаємо не більше k «найкращих»
        }

        var result = new List<string>();
        while (heap.TryDequeue(out var word, out var priority)) result.Add($"{word}={priority.Count}");
        result.Reverse(); // з купи виходять від гіршого до кращого
        return result;
    }
}
```

**Приклад запуску:**

```text
top-3 largest: 25 20 15
top-2 words:   the=4, dog=2
```

### 13.4. Злиття k відсортованих списків

Кладемо в купу **голову** кожного списку. Забираємо мінімум, додаємо до результату і кладемо наступний елемент **того самого** списку. Час O(N log k), де N — загальна кількість елементів.

```
 L0: 1 4 7      купа: (1,L0) (2,L1) (3,L2)
 L1: 2 5 8      pop 1 → push 4 з L0      купа: (2,L1) (3,L2) (4,L0)
 L2: 3 6 9      pop 2 → push 5 з L1  ...  результат: 1 2 3 4 5 6 7 8 9
```

### Приклад 33. Merge K Sorted

```csharp
int[][] lists = [[1, 4, 7, 10], [2, 5, 8], [0, 6, 9, 11, 12], []];
Console.WriteLine(string.Join(" ", Merger.MergeK(lists)));

// Той самий алгоритм для «лінивих» послідовностей (наприклад, файлів логів, відсортованих за часом).
IEnumerable<string>[] logs =
[
    ["09:00 web start", "09:05 web request"],
    ["09:01 db start", "09:03 db backup", "09:07 db done"],
];
foreach (var line in Merger.MergeK(logs, StringComparer.Ordinal)) Console.WriteLine(line);

public static class Merger
{
    public static IEnumerable<int> MergeK(int[][] lists)
    {
        // Елемент купи — (номер списку, індекс у ньому); пріоритет — саме значення.
        var heap = new PriorityQueue<(int List, int Index), int>();
        for (int i = 0; i < lists.Length; i++)
        {
            if (lists[i].Length > 0) heap.Enqueue((i, 0), lists[i][0]);
        }

        while (heap.TryDequeue(out var pos, out int value))
        {
            yield return value;
            int nextIndex = pos.Index + 1;
            if (nextIndex < lists[pos.List].Length)
            {
                heap.Enqueue((pos.List, nextIndex), lists[pos.List][nextIndex]);
            }
        }
    }

    public static IEnumerable<T> MergeK<T>(IEnumerable<IEnumerable<T>> sources, IComparer<T> comparer)
    {
        var heap = new PriorityQueue<IEnumerator<T>, T>(comparer);
        var enumerators = new List<IEnumerator<T>>();
        try
        {
            foreach (var source in sources)
            {
                var e = source.GetEnumerator();
                enumerators.Add(e);
                if (e.MoveNext()) heap.Enqueue(e, e.Current);
            }

            while (heap.TryDequeue(out var e, out var current))
            {
                yield return current;
                if (e.MoveNext()) heap.Enqueue(e, e.Current); // наступний з того ж джерела
            }
        }
        finally
        {
            foreach (var e in enumerators) e.Dispose();
        }
    }
}
```

**Приклад запуску:**

```text
0 1 2 4 5 6 7 8 9 10 11 12
09:00 web start
09:01 db start
09:03 db backup
09:05 web request
09:07 db done
```

### 13.5. Планувальник задач з пріоритетами

Задачі мають пріоритет і час надходження. Процесор виконує задачу до кінця (без переривань), обираючи серед **тих, що вже надійшли,** найпріоритетнішу (менше число — важливіше; при рівності — та, що прийшла раніше). Надходження — у звичайній черзі за часом, готові — у купі.

### Приклад 34. Non-preemptive priority scheduler

```csharp
TaskItem[] tasks =
[
    new("backup",   Arrival: 0, Duration: 4, Priority: 3),
    new("email",    Arrival: 1, Duration: 1, Priority: 2),
    new("payment",  Arrival: 2, Duration: 2, Priority: 1),
    new("report",   Arrival: 3, Duration: 3, Priority: 2),
    new("alert",    Arrival: 9, Duration: 1, Priority: 1),
    new("cleanup",  Arrival: 9, Duration: 2, Priority: 5),
];

foreach (var line in PriorityScheduler.Run(tasks))
{
    Console.WriteLine(line);
}

public sealed record TaskItem(string Name, int Arrival, int Duration, int Priority);

public static class PriorityScheduler
{
    public static IEnumerable<string> Run(TaskItem[] tasks)
    {
        var arrivals = new Queue<TaskItem>(tasks.OrderBy(t => t.Arrival));
        var ready = new PriorityQueue<TaskItem, (int Priority, int Arrival)>();
        int time = 0;

        while (arrivals.Count > 0 || ready.Count > 0)
        {
            // Усі задачі, що надійшли до поточного моменту, — у купу.
            while (arrivals.TryPeek(out var t) && t.Arrival <= time)
            {
                arrivals.Dequeue();
                ready.Enqueue(t, (t.Priority, t.Arrival));
            }

            if (!ready.TryDequeue(out var task, out _))
            {
                yield return $"t={time,2}: idle until {arrivals.Peek().Arrival}";
                time = arrivals.Peek().Arrival;
                continue;
            }

            yield return $"t={time,2}: run {task.Name,-8} (prio {task.Priority}, waited {time - task.Arrival})";
            time += task.Duration;
        }

        yield return $"t={time,2}: all done";
    }
}
```

**Приклад запуску:**

```text
t= 0: run backup   (prio 3, waited 0)
t= 4: run payment  (prio 1, waited 2)
t= 6: run email    (prio 2, waited 5)
t= 7: run report   (prio 2, waited 4)
t=10: run alert    (prio 1, waited 1)
t=11: run cleanup  (prio 5, waited 2)
t=13: all done
```

### 13.6. Тизер: алгоритм Дейкстри

Дейкстра — це «BFS для зважених графів», де звичайну чергу замінено на **чергу з пріоритетом** за поточною відстанню. Оскільки `PriorityQueue` не вміє зменшувати пріоритет, використовуємо «ліниве видалення»: додаємо дублікат з кращою відстанню, а застарілі записи пропускаємо. Детально — у лекції про графи.

```
        4
   A ───────── B
   │ \         │
 1 │   \ 2     │ 1
   │     \     │
   C ───── D ──┘
       5     
 A→C=1, A→D=2, A→B=4, C→D=5, D→B=1   найкоротше A→B: A→D→B = 3
```

### Приклад 35. Дейкстра на `PriorityQueue`

```csharp
var graph = new Dictionary<string, List<(string To, int Weight)>>
{
    ["A"] = [("B", 4), ("C", 1), ("D", 2)],
    ["B"] = [("D", 1)],
    ["C"] = [("A", 1), ("D", 5)],
    ["D"] = [("B", 1), ("C", 5)],
};

var (dist, prev) = Dijkstra.Run(graph, "A");
foreach (var v in dist.Keys.Order())
{
    var path = new Stack<string>();                       // стек знову розвертає шлях
    for (string? at = v; at is not null; at = prev.GetValueOrDefault(at)) path.Push(at);
    Console.WriteLine($"A -> {v}: {dist[v]}  via {string.Join("→", path)}");
}

public static class Dijkstra
{
    public static (Dictionary<string, int> Dist, Dictionary<string, string> Prev) Run(
        Dictionary<string, List<(string To, int Weight)>> graph, string source)
    {
        var dist = new Dictionary<string, int> { [source] = 0 };
        var prev = new Dictionary<string, string>();
        var pq = new PriorityQueue<string, int>();
        pq.Enqueue(source, 0);
        int skipped = 0;

        while (pq.TryDequeue(out var u, out int d))
        {
            if (d > dist[u])
            {
                skipped++;
                continue; // застарілий запис — вже знайшли коротший шлях
            }

            foreach (var (v, w) in graph[u])
            {
                int candidate = d + w;
                if (candidate < dist.GetValueOrDefault(v, int.MaxValue))
                {
                    dist[v] = candidate;
                    prev[v] = u;
                    pq.Enqueue(v, candidate); // замість decrease-key — дублікат
                }
            }
        }

        Console.WriteLine($"(stale entries skipped: {skipped})");
        return (dist, prev);
    }
}
```

**Приклад запуску:**

```text
(stale entries skipped: 1)
A -> A: 0  via A
A -> B: 3  via A→D→B
A -> C: 1  via A→C
A -> D: 2  via A→D
```

### Типові помилки

1. **Вважати `PriorityQueue` стабільною.** Однакові пріоритети виходять у довільному порядку — додайте лічильник у пріоритет.
2. **Перелічувати `UnorderedItems` і чекати впорядкованості.** Це сирий масив купи.
3. **Змінювати поле об'єкта, від якого залежить пріоритет,** поки він у купі — купа «ламається» мовчки.
4. **Top-K з макс-купою розміру n** — O(n + k log n) пам'яті O(n); для потоку краща мін-купа розміру k.
5. **Індекс дітей `2i` і `2i+1`** — формула для масиву з 1, а не з 0.
6. **Дейкстра з від'ємними вагами** — не працює; потрібен Беллман–Форд.

### Міні-вправа 13.1

**Медіана потоку.** Підтримуйте дві купи: макс-купу «нижньої половини» і мін-купу «верхньої». Виведіть медіану після кожного з чисел `5, 15, 1, 3, 8`.

<details>
<summary>Розв'язок</summary>

```csharp
var lower = new PriorityQueue<int, int>(Comparer<int>.Create((a, b) => b.CompareTo(a))); // макс-купа
var upper = new PriorityQueue<int, int>();                                              // мін-купа

foreach (int x in new[] { 5, 15, 1, 3, 8 })
{
    // 1) Кладемо в нижню, потім її максимум переносимо у верхню — щоб порядок між купами зберігся.
    lower.Enqueue(x, x);
    int moved = lower.Dequeue();
    upper.Enqueue(moved, moved);

    // 2) Балансуємо: нижня може бути більшою на 1, але не меншою.
    if (upper.Count > lower.Count)
    {
        int back = upper.Dequeue();
        lower.Enqueue(back, back);
    }

    double median = lower.Count > upper.Count ? lower.Peek() : (lower.Peek() + upper.Peek()) / 2.0;
    Console.WriteLine($"add {x,2}: median = {median}");
}
```

```text
add  5: median = 5
add 15: median = 10
add  1: median = 5
add  3: median = 4
add  8: median = 5
```

</details>

---

## 14. Конкурентні черги

*(≈7 хв)*

Звичайні `Queue<T>` і `Stack<T>` **не потокобезпечні**: одночасний `Enqueue` з двох потоків може пошкодити внутрішній масив. Варіанти:

| Тип | Коли використовувати |
|-----|----------------------|
| `ConcurrentQueue<T>`, `ConcurrentStack<T>` | кілька потоків додають/забирають; без блокувань (lock-free); лише `TryDequeue`/`TryPop` |
| `BlockingCollection<T>` | старіший API: блокуюче очікування, обмежена місткість |
| **`Channel<T>`** (`System.Threading.Channels`) | сучасний **асинхронний** producer/consumer: `await WriteAsync`/`ReadAllAsync`, backpressure через `BoundedChannel` |

**Патерн producer/consumer:** виробники кладуть роботу в чергу, споживачі її забирають. Черга розв'язує їх за швидкістю: якщо споживачі повільніші — обмежений канал змушує виробника **чекати** (backpressure), а не з'їдати пам'ять.

```
 Producer 1 ──┐                         ┌──▶ Consumer A
              ├──▶ [ Channel (cap 3) ] ─┤
 Producer 2 ──┘    WriteAsync чекає,    └──▶ Consumer B
                   якщо канал повний
```

### Приклад 36. `ConcurrentQueue<T>` та `Channel<T>`

```csharp
using System.Collections.Concurrent;
using System.Threading.Channels;

// 1) ConcurrentQueue: 4 потоки по 10 000 елементів без жодного lock.
var concurrent = new ConcurrentQueue<int>();
Parallel.For(0, 4, worker =>
{
    for (int i = 0; i < 10_000; i++) concurrent.Enqueue(worker * 100_000 + i);
});
long sum = 0;
while (concurrent.TryDequeue(out int item)) sum += item;
Console.WriteLine($"ConcurrentQueue: dequeued sum = {sum}");

// 2) Channel: обмежений канал, 2 виробники, 2 споживачі.
var channel = Channel.CreateBounded<int>(new BoundedChannelOptions(capacity: 3)
{
    FullMode = BoundedChannelFullMode.Wait, // повний канал → WriteAsync асинхронно чекає
});

var processed = new ConcurrentBag<(string Consumer, int Item)>();

async Task ProduceAsync(int id)
{
    for (int i = 1; i <= 5; i++)
    {
        await channel.Writer.WriteAsync(id * 100 + i);
    }
}

async Task ConsumeAsync(string name)
{
    // ReadAllAsync завершиться, коли канал закриють і він спорожніє.
    await foreach (int item in channel.Reader.ReadAllAsync())
    {
        await Task.Delay(1); // імітація роботи
        processed.Add((name, item));
    }
}

var consumers = new[] { ConsumeAsync("A"), ConsumeAsync("B") };
await Task.WhenAll(ProduceAsync(1), ProduceAsync(2));
channel.Writer.Complete(); // сигнал «більше даних не буде»
await Task.WhenAll(consumers);

// Розподіл між споживачами недетермінований, тому друкуємо лише стабільні факти.
var items = processed.Select(p => p.Item).Order().ToList();
Console.WriteLine($"Channel: processed {items.Count} items: {string.Join(" ", items)}");
Console.WriteLine($"Every item exactly once: {items.Distinct().Count() == 10}");
```

**Приклад запуску:**

```text
ConcurrentQueue: dequeued sum = 6199980000
Channel: processed 10 items: 101 102 103 104 105 201 202 203 204 205
Every item exactly once: True
```

### Типові помилки

1. **`lock` навколо `ConcurrentQueue`** — зайво, вона вже потокобезпечна; а от **складені** операції («перевірити і взяти») треба робити через `TryDequeue`, а не `IsEmpty` + `TryDequeue`.
2. **Забутий `Writer.Complete()`** — `ReadAllAsync` чекатиме вічно, програма «зависне».
3. **`Channel.CreateUnbounded`** для швидкого виробника й повільного споживача — неконтрольований ріст пам'яті.
4. **Покладатися на порядок обробки** між кількома споживачами — його немає; порядок FIFO гарантується лише для одного читача.

---

## 15. Підсумки

*(≈3 хв)*

| Структура | Принцип | Найкраща реалізація | Типові задачі |
|-----------|---------|---------------------|---------------|
| Стек | LIFO | динамічний масив (`Stack<T>`) | дужки, вирази, DFS/рекурсія, undo, монотонний стек |
| Черга | FIFO | кільцевий буфер (`Queue<T>`) | BFS, симуляції, планувальники, rate limiting |
| Дек | обидва кінці | кільцевий буфер (власний) / `LinkedList<T>` | ковзаюче вікно, обмежена історія |
| Черга з пріоритетом | мінімум першим | бінарна/4-арна купа (`PriorityQueue`) | top-k, merge k, Дейкстра, планувальники |
| Конкурентна черга | FIFO між потоками | `ConcurrentQueue<T>`, `Channel<T>` | producer/consumer, backpressure |

**Головні ідеї лекції:**

1. АТД — це контракт, реалізація — компроміс. Пишіть код проти інтерфейсу.
2. `Try`-методи для очікуваної порожнечі, винятки — для багів.
3. Подвоєння місткості дає O(1) амортизовано; два стеки дають чергу за O(1) амортизовано.
4. Стек «відкладає» незавершену роботу (дужки, оператори, кадри, індекси без відповіді).
5. Монотонний стек/дек перетворює O(n²) на O(n): кожен елемент входить і виходить один раз.
6. Черга обробляє «хвилями» → найкоротший шлях у незваженому графі; `levelSize` → обробка по рівнях.
7. Купа — O(log n) на вставку/видалення мінімуму; мін-купа розміру k — для top-k.
8. У багатопотоковому коді — лише потокобезпечні колекції та обмежені канали.

---

## 16. Питання для самоперевірки

1. Чим абстрактний тип даних відрізняється від структури даних? Наведіть по дві реалізації АТД «черга» та «стек».
2. Чому `Pop` на порожньому стеку кидає саме `InvalidOperationException`? Коли слід використовувати `TryPop`?
3. Доведіть, що n операцій `Push` у стек з подвоєнням місткості виконуються за O(n). Що буде, якщо збільшувати місткість на 100?
4. Навіщо в `Pop` для масивного стека записувати `default` у звільнену комірку?
5. Як у кільцевому буфері розрізнити порожню та повну чергу? Чому `(i - 1) % n` небезпечно в C#?
6. Поясніть амортизований аналіз черги на двох стеках методом потенціалів. Яка найгірша вартість одного `Dequeue`?
7. Чому `new Stack<int>(existingStack)` дає стек у зворотному порядку?
8. Опишіть правила shunting-yard для лівої та правої асоціативності. Як обробити унарний мінус?
9. У якому порядку знімаються операнди при обчисленні постфіксного і префіксного виразів?
10. Чому глибока рекурсія небезпечна в .NET і як перетворити рекурсивний DFS на ітеративний, зберігши порядок обходу?
11. Як реалізувати `GetMin()` за O(1) у стеку? Скільки додаткової пам'яті це потребує?
12. Чому монотонний стек працює за O(n), хоча всередині `for` є `while`?
13. Чому нова дія в редакторі очищує стек redo?
14. Чому BFS знаходить найкоротший шлях у незваженому графі, а DFS — ні? Чому `visited` треба ставити при додаванні в чергу?
15. Що таке multi-source BFS і як з його допомогою порахувати час «зараження» решітки?
16. Що таке коефіцієнт завантаження ρ у симуляції черги і що відбувається при ρ > 1?
17. Опишіть інваріант монотонного деку в задачі про максимум у ковзаючому вікні.
18. Чим відрізняються rate limiter'и «sliding log», «fixed window» та «token bucket»?
19. Виведіть формули `parent`, `left`, `right` для купи в масиві з індексацією з 0. Чому heapify — O(n)?
20. Як отримати макс-купу з `PriorityQueue<TElement, TPriority>`? Як зробити її стабільною?
21. Чому для top-k використовують мін-купу розміру k, а не макс-купу?
22. Чому в Дейкстрі на `PriorityQueue` у .NET потрібна перевірка `if (d > dist[u]) continue`?
23. Навіщо `Channel<T>` обмежена місткість і що станеться, якщо не викликати `Writer.Complete()`?

---

## 17. Практичні завдання

### Базовий рівень

1. **Узагальнений АТД.** Зробіть `ArrayStack<T>`, `LinkedStack<T>` реалізаціями `IStack<T>` з розділу 1, а `CircularQueue<T>`, `LinkedQueue<T>`, `TwoStackQueue<T>` — `IQueue<T>`. Напишіть один набір тестів, що запускається для кожної реалізації.
2. **Стек з обмеженою місткістю.** `BoundedStack<T>(capacity)`: `Push` при переповненні кидає `InvalidOperationException`, `TryPush` повертає `false`.
3. **Перевірка паліндрома** стеком і чергою одночасно (ігноруючи регістр і небукви): `"A man, a plan, a canal: Panama"` → `true`.
4. **Кільцева черга фіксованого розміру** `MyCircularQueue(k)` з методами `EnQueue`, `DeQueue`, `Front`, `Rear`, `IsEmpty`, `IsFull` без використання `_count` (підказка: залиште одну комірку порожньою).
5. **Двійкові числа від 1 до n** за допомогою черги: `"1"`, далі для кожного `s` додавати `s + "0"` та `s + "1"`.

### Середній рівень

6. **Калькулятор.** Розширте приклад 11: функції `max(a, b)`, `sqrt(x)`, змінні (`x = 3; 2 * x + 1`), повідомлення про помилку з позицією токена.
7. **Asteroid collision.** Масив астероїдів (знак — напрям, модуль — розмір). Змоделюйте зіткнення стеком: `[5, 10, -5]` → `[5, 10]`; `[10, 2, -5]` → `[10]`.
8. **Trapping Rain Water** монотонним стеком: `[0,1,0,2,1,0,1,3,2,1,2,1]` → `6`.
9. **Maximal Rectangle у бінарній матриці** через гістограму (приклад 16) для кожного рядка.
10. **Відкрити замок** (Open the Lock): 4 колеса 0–9, заборонені комбінації; мінімум обертань від `"0000"` до цілі — BFS.
11. **Двоколірність графа** (bipartite) через BFS.
12. **Hit counter за 5 хвилин** з O(1) пам'яті: кільцевий буфер з 300 комірок (секунди) замість черги міток.
13. **Token bucket rate limiter** з місткістю `B` та швидкістю поповнення `r` токенів/с; порівняйте поведінку зі sliding log на однаковому сценарії.

### Високий рівень

14. **d-арна купа.** Узагальніть `BinaryHeap<T>` до `DHeap<T>(d)`; порівняйте кількість порівнянь для `d = 2, 4, 8` на 1 000 000 операцій (фіксований seed).
15. **Indexed priority queue** з операцією `DecreaseKey(element, newPriority)` за O(log n) (словник «елемент → індекс у масиві»). Перепишіть на ній Дейкстру без дублікатів.
16. **Task Scheduler з cooldown:** задачі `A A A B B B`, між однаковими — щонайменше `n = 2` тіки. Знайдіть мінімальний час (макс-купа + черга «охолодження»).
17. **Симуляція супермаркету:** k кас, клієнт обирає найкоротшу чергу; порівняйте з однією спільною чергою на k кас. Зробіть висновок, який варіант дає менший середній та максимальний час очікування (seed = 2025).
18. **Pipeline на `Channel<T>`:** три етапи (завантаження → обробка → запис), кожен — окремий обмежений канал і кілька споживачів; скасування через `CancellationToken`.
19. **Undo з групуванням:** послідовні вставки символів протягом 1 с об'єднуються в одну команду (як у справжніх редакторах).
20. **Deque-based 0-1 BFS:** решітка, де рух прямо коштує 0, а поворот — 1. Знайдіть мінімальну кількість поворотів від старту до фінішу.
