# Лабораторна робота 2 — виконайте 2 завдання: одну реалізацію (3 бали) та одну задачу середньої складності (2 бали)

> **C# / .NET 10.** [Налаштування та запуск](../LABS.md).

Із **шести варіантів** оберіть **рівно два завдання**:

- **Перше:** одне із завдань **1–3** — власна реалізація структури даних, **3 бали**.
- **Друге:** одне із завдань **4–6** — прикладна задача середньої складності, **2 бали**.

**Разом: 5 балів.** Наприклад, можна обрати завдання 1 і 5 або 3 і 4.
Виконувати всі шість не потрібно; вибір другої задачі не залежить від першої.

| № | Завдання | Група | Бали |
|---|---|---|---|
| 1 | Власний стек `LabStack<T>` | Реалізація | 3 |
| 2 | Власний динамічний список `LabList<T>` | Реалізація | 3 |
| 3 | Власний двозв'язний список `LabLinkedList<T>` | Реалізація | 3 |
| 4 | Власний час виконання функцій | Середня складність | 2 |
| 5 | Доступні проміжки після блокування | Середня складність | 2 |
| 6 | Гра з кульками у кільці | Середня складність | 2 |

Кожен блок **`Program.cs`** можна повністю скопіювати в окремий консольний
проєкт. Заготовки компілюються, але навчальні методи, конструктори й обчислювані
властивості містять лише `throw new NotImplementedException()`.
Додайте приватні поля та замініть заглушки власним кодом.
Демонстраційні виклики й типи даних уже задано; очікуваний вивід наведено для
завершеної реалізації. Готових розв'язків у цьому матеріалі немає.

Для завдань **1–3** заборонено зберігати елементи всередині готових `Stack<T>`,
`List<T>`, `LinkedList<T>`, `Queue<T>` або будувати реалізацію через LINQ.
Для стека й динамічного списку використовуйте власний масив `T[]`, для
двозв'язного списку — власні вузли. Дозволені `Array.Copy`, `Array.Clear`,
`Array.Resize` та `EqualityComparer<T>.Default`.
Для завдань **4–6** дозволені стандартні колекції .NET; власна структура з
першого завдання не є обов'язковою.

## Завдання 1. Власний стек `LabStack<T>` — 3 бали

Реалізуйте узагальнений стек на динамічному масиві. Вершина — останній
зайнятий елемент. Потрібно реалізувати **6 методів**, конструктор і дві
властивості. До базових операцій додано безпечне вилучення, копіювання та очищення.

### Обов'язковий API

| Член | Поведінка |
|---|---|
| `LabStack(int capacity = 0)` | Порожній стек із заданою початковою місткістю |
| `Count`, `IsEmpty` | Кількість елементів та ознака порожнього стека |
| `Push(T value)` | Додати на вершину |
| `Pop()` | Вилучити вершину та повернути її значення |
| `Peek()` | Прочитати вершину без вилучення |
| `TryPop(out T value)` | На порожньому стеку повернути `false` і `default` через `out`; інакше — `true` та вилучене значення |
| `ToArray()` | Повернути незалежну копію від вершини до дна |
| `Clear()` | Очистити стек, зберігши місткість буфера |

- Від'ємна початкова місткість — `ArgumentOutOfRangeException`.
- `Pop` / `Peek` на порожньому стеку — `InvalidOperationException`.
  Невдала операція не змінює стек.
- Коли буфер заповнено, збільшуйте його вдвічі; якщо початкова місткість
  нульова, перший буфер має вміщувати щонайменше 4 елементи.
- Вилучені елементи в масиві замінюйте на `default`, щоб не утримувати посилання.
- **Складність:** `Push` — амортизовано O(1), `Pop` / `Peek` / `TryPop` —
  O(1), `ToArray` / `Clear` — O(n).

### `Program.cs` — заготовка без реалізації

```csharp
var stack = new LabStack<int>(2);
foreach (int value in new[] { 10, 20, 30, 40 }) stack.Push(value);
Console.WriteLine($"Стек: {string.Join(' ', stack.ToArray())}");
Console.WriteLine($"Вершина: {stack.Peek()}");
Console.WriteLine($"Pop: {stack.Pop()}");
Console.WriteLine($"TryPop: {stack.TryPop(out int top)}, {top}");
Console.WriteLine($"Після вилучень: {string.Join(' ', stack.ToArray())}");
stack.Clear();
Console.WriteLine($"Після очищення: {stack.Count}, {stack.IsEmpty}");
Console.WriteLine($"Порожній TryPop: {stack.TryPop(out top)}, {top}");

var words = new LabStack<string>();
words.Push("кіт");
words.Push("пес");
Console.WriteLine($"Рядки: {string.Join(' ', words.ToArray())}");

public sealed class LabStack<T>
{
    // Додайте власний масив та лічильник елементів.
    public LabStack(int capacity = 0) => throw new NotImplementedException();
    public int Count => throw new NotImplementedException();
    public bool IsEmpty => throw new NotImplementedException();
    public void Push(T value) => throw new NotImplementedException();
    public T Pop() => throw new NotImplementedException();
    public T Peek() => throw new NotImplementedException();
    public bool TryPop(out T value) => throw new NotImplementedException();
    public T[] ToArray() => throw new NotImplementedException();
    public void Clear() => throw new NotImplementedException();
}
```

### Очікуваний вивід після реалізації

```text
Стек: 40 30 20 10
Вершина: 40
Pop: 40
TryPop: True, 30
Після вилучень: 20 10
Після очищення: 0, True
Порожній TryPop: False, 0
Рядки: пес кіт
```

Додатково перевірте незалежність `ToArray`, ріст буфера, `Pop` / `Peek`
на порожньому стеку та повторне використання після очищення.

## Завдання 2. Власний динамічний список `LabList<T>` — 3 бали

Реалізуйте список із доступом за індексом на власному масиві.
Потрібно реалізувати **7 методів**, конструктор, дві властивості та індексатор.
Окрім одиничних операцій, список має підтримувати вилучення діапазону.

### Обов'язковий API

| Член | Поведінка |
|---|---|
| `LabList(int capacity = 0)` | Порожній список із заданою місткістю |
| `Count`, `Capacity`, індексатор `this[int index]` | Кількість, місткість, читання та заміна елемента |
| `Add(T value)` | Додати в кінець |
| `Insert(int index, T value)` | Додати перед заданою позицією |
| `RemoveAt(int index)` | Вилучити за індексом |
| `IndexOf(T value)` | Індекс першого збігу або `-1` |
| `RemoveRange(int index, int count)` | Вилучити діапазон одним зсувом хвоста |
| `ToArray()` | Незалежна копія елементів у порядку списку |
| `Clear()` | Очистити список, зберігши місткість буфера |

- Доступ і вилучення: `0 <= index < Count`; вставка: `0 <= index <= Count`.
- Діапазон: `0 <= index <= Count`, `0 <= count <= Count - index`.
  Нульова довжина дозволена, зокрема при `index == Count`.
- Некоректний індекс, діапазон чи початкова місткість —
  `ArgumentOutOfRangeException`. Стан не змінюється.
- Порівнюйте значення через `EqualityComparer<T>.Default`.
- Коли буфер заповнено, збільшуйте його вдвічі; якщо початкова місткість
  нульова, перший буфер має вміщувати щонайменше 4 елементи.
- Вилучення зберігає порядок решти елементів. Очищайте зайняті раніше
  комірки, щоб не утримувати посилання на вилучені об'єкти.
- **Складність:** індексатор — O(1), `Add` — амортизовано O(1), пошук,
  вставка, вилучення, копіювання й очищення — O(n).

### `Program.cs` — заготовка без реалізації

```csharp
var list = new LabList<int>(2);
list.Add(1);
list.Add(3);
list.Insert(1, 2);
list.Add(4);
list.Add(5);
Console.WriteLine($"Після вставок: {string.Join(' ', list.ToArray())}");
list[0] = 10;
list.RemoveRange(1, 2);
list.RemoveAt(1);
list.Add(20);
list.Add(30);
Console.WriteLine($"Список: {string.Join(' ', list.ToArray())}");
Console.WriteLine($"Індекси 20 і 99: {list.IndexOf(20)}, {list.IndexOf(99)}");
int[] copy = list.ToArray();
copy[0] = 99;
Console.WriteLine($"Копія: {string.Join(' ', copy)}");
Console.WriteLine($"Перший в оригіналі: {list[0]}");
int capacity = list.Capacity;
list.Clear();
Console.WriteLine($"Після Clear: {list.Count}, місткість збережена: {list.Capacity == capacity}");

public sealed class LabList<T>
{
    // Додайте власний масив та лічильник елементів.
    public LabList(int capacity = 0) => throw new NotImplementedException();
    public int Count => throw new NotImplementedException();
    public int Capacity => throw new NotImplementedException();
    public T this[int index]
    {
        get => throw new NotImplementedException();
        set => throw new NotImplementedException();
    }
    public void Add(T value) => throw new NotImplementedException();
    public void Insert(int index, T value) => throw new NotImplementedException();
    public void RemoveAt(int index) => throw new NotImplementedException();
    public int IndexOf(T value) => throw new NotImplementedException();
    public void RemoveRange(int index, int count) => throw new NotImplementedException();
    public T[] ToArray() => throw new NotImplementedException();
    public void Clear() => throw new NotImplementedException();
}
```

### Очікуваний вивід після реалізації

```text
Після вставок: 1 2 3 4 5
Список: 10 5 20 30
Індекси 20 і 99: 2, -1
Копія: 99 5 20 30
Перший в оригіналі: 10
Після Clear: 0, місткість збережена: True
```

Додатково перевірте тип `string`, порожні діапазони, вставку в початок і кінець,
вилучення всіх елементів, дублікати, пошук відсутнього значення й помилкові індекси.

## Завдання 3. Власний двозв'язний список `LabLinkedList<T>` — 3 бали

Реалізуйте список із власними вузлами. Кожен вузол зберігає значення,
посилання на попередній і наступний вузли та список-власник.
Потрібно реалізувати **7 методів**, конструктори та властивості з наведеної
заготовки. Вставка й вилучення за готовим вузлом мають працювати без пошуку.

### Обов'язковий API

| Член | Поведінка |
|---|---|
| `Count`, `First`, `Last` | Кількість та крайні вузли; у порожньому списку обидва посилання — `null` |
| `AddFirst(T value)` | Створити вузол на початку та повернути його |
| `AddLast(T value)` | Створити вузол у кінці та повернути його |
| `AddAfter(node, value)` | Створити вузол після вказаного та повернути його |
| `Remove(node)` | Від'єднати саме цей вузол |
| `Find(T value)` | Перший вузол зі значенням або `null` |
| `ToArray()` | Незалежна копія значень від початку до кінця |
| `Clear()` | Від'єднати всі вузли та очистити список |

- `First.Previous` і `Last.Next` дорівнюють `null`. Для кожного приєднаного
  вузла `Owner` указує на його список. Після вилучення або `Clear` його
  `Owner`, `Next` і `Previous` мають дорівнювати `null`.
- `null` замість вузла — `ArgumentNullException`.
  Чужий або вже від'єднаний вузол — `InvalidOperationException`.
  Перевіряйте аргументи до зміни зв'язків.
- Порівняння значень у `Find` — `EqualityComparer<T>.Default`.
- Зберігайте ідентичність вузлів: `Find` повертає наявний вузол, а `Remove`
  від'єднує саме переданий вузол. Не підміняйте вилучення копіюванням значень.
- Після вставки й вилучення узгоджено оновлюйте зв'язки в обох напрямках,
  крайні вузли та `Count`.
- **Складність:** вставки та вилучення за готовим вузлом — O(1),
  `Find`, `ToArray`, `Clear` — O(n).

### `Program.cs` — заготовка без реалізації

```csharp
var list = new LabLinkedList<string>();
var a = list.AddLast("A");
var c = list.AddLast("C");
var b = list.AddAfter(a, "B");
list.AddFirst("Head");
Console.WriteLine($"Список: {string.Join(' ', list.ToArray())}");
Console.WriteLine($"Знайдено той самий B: {ReferenceEquals(list.Find("B"), b)}");
list.Remove(b);
Console.WriteLine($"B від'єднаний: {b.Owner is null && b.Next is null && b.Previous is null}");
Console.WriteLine($"Після вилучення B: {string.Join(' ', list.ToArray())}");
list.Remove(list.First!);
list.Remove(c);
Console.WriteLine($"Після вилучення країв: {string.Join(' ', list.ToArray())}");
Console.WriteLine($"Немає Z: {list.Find("Z") is null}");
list.Clear();
Console.WriteLine($"Порожні межі: {list.Count == 0 && list.First is null && list.Last is null}");

public sealed class LabNode<T>
{
    internal LabNode(T value) => throw new NotImplementedException();
    public T Value { get; set; } = default!;
    public LabNode<T>? Previous { get; internal set; }
    public LabNode<T>? Next { get; internal set; }
    public LabLinkedList<T>? Owner { get; internal set; }
}

public sealed class LabLinkedList<T>
{
    // Додайте посилання на крайні вузли та лічильник елементів.
    public LabLinkedList() => throw new NotImplementedException();
    public int Count => throw new NotImplementedException();
    public LabNode<T>? First => throw new NotImplementedException();
    public LabNode<T>? Last => throw new NotImplementedException();
    public LabNode<T> AddFirst(T value) => throw new NotImplementedException();
    public LabNode<T> AddLast(T value) => throw new NotImplementedException();
    public LabNode<T> AddAfter(LabNode<T> node, T value) => throw new NotImplementedException();
    public void Remove(LabNode<T> node) => throw new NotImplementedException();
    public LabNode<T>? Find(T value) => throw new NotImplementedException();
    public T[] ToArray() => throw new NotImplementedException();
    public void Clear() => throw new NotImplementedException();
}
```

### Очікуваний вивід після реалізації

```text
Список: Head A B C
Знайдено той самий B: True
B від'єднаний: True
Після вилучення B: Head A C
Після вилучення країв: A
Немає Z: True
Порожні межі: True
```

Додатково перевірте тип `int`, один вузол, обхід через `Previous`, вставку після
останнього вузла, чужі й від'єднані вузли та повторне використання списку після очищення.

## Завдання 4. Власний час виконання функцій — 2 бали

Однопотокова програма записує початок і завершення викликів функцій у журнал.
Виклик може запускати іншу функцію або рекурсивно викликати себе. Потрібно
порахувати **власний час** кожної функції: час, коли працювала саме вона,
без часу вкладених викликів. Для рекурсії всі виклики одного ідентифікатора
додають час до спільного результату.

### Умова

- Функції мають ідентифікатори від `0` до `functionCount - 1`.
- Запис має вигляд `id:start:time` або `id:end:time`.
- `start:t` означає початок на початку такту t, `end:t` — завершення
  **в кінці** такту t. Виклик від `start:2` до `end:2` триває один такт.
- Журнал уже впорядкований; вкладеність коректна, усі виклики завершені.
  Функції не працюють паралельно. Між окремими викликами можливий простій.
- Поверніть `int[]` довжини `functionCount`. Для функції без викликів — 0.
  Час простою не належить жодній функції. Порожній журнал дає всі нулі.
- Межі: `1 <= functionCount <= 100`, не більше 100 000 записів,
  `0 <= time <= 1_000_000`. Перевірка формату коректних вхідних даних не потрібна.
- Використайте стек активних викликів. Не імітуйте кожен окремий такт часу.
- **Складність:** O(L + F) часу та O(D + F) пам'яті, де L — кількість
  записів, F — кількість функцій, D — максимальна глибина викликів.

### `Program.cs` — заготовка без реалізації

```csharp
(int FunctionCount, string[] Logs)[] examples =
[
    (2, ["0:start:0", "1:start:2", "1:end:5", "0:end:6"]),
    (1, ["0:start:0", "0:start:2", "0:end:3", "0:end:5"]),
    (3, ["0:start:0", "0:end:0", "2:start:4", "2:end:5"]),
    (2, []),
];

foreach (var example in examples)
{
    int[] times = ExclusiveTimes(example.FunctionCount, example.Logs);
    Console.WriteLine($"Час: {string.Join(' ', times)}");
}

static int[] ExclusiveTimes(int functionCount, string[] logs)
{
    throw new NotImplementedException();
}
```

### Очікуваний вивід після реалізації

```text
Час: 3 4
Час: 6
Час: 1 0 2
Час: 0 0
```

У першому прикладі функція 0 працює на тактах 0, 1 і 6, а функція 1 —
на тактах 2–5. У другому прикладі рекурсивний виклик тієї самої функції
не повинен призводити до подвійного підрахунку тактів 2 і 3.
Додатково перевірте три рівні вкладеності та кілька викликів однієї функції.

## Завдання 5. Доступні проміжки після блокування — 2 бали

Система має впорядкований список доступних проміжків і список заблокованих
проміжків. Знайдіть усі частини доступних проміжків, які залишилися після
блокування. Один блок може перекривати кілька доступних проміжків, а один
доступний проміжок може розділитися на кілька частин.

### Умова

- Проміжок `[Start, End)` включає ліву межу й не включає праву;
  `0 <= Start < End <= 1_000_000`.
- Кожен вхідний масив уже відсортований за `Start`. Усередині кожного
  масиву проміжки розділені: `previous.End < next.Start`.
- Масиви можуть бути порожніми; у кожному не більше 100 000 проміжків.
  Вхідні дані коректні, змінювати їх не можна.
- Поверніть `List<Interval>` із залишками за зростанням `Start`.
  Не додавайте порожні проміжки й не включайте заблоковані ділянки.
- Дотик меж не є перекриттям: блок `[10, 15)` не змінює `[0, 10)`.
- Якщо блоків немає, результат містить усі доступні проміжки;
  повне блокування або відсутність доступних проміжків дають порожній список.
- Не перебирайте кожну точку, не сортуйте повторно та не починайте
  сканування всіх блоків заново для кожного доступного проміжку.
- **Складність:** O(n + m + r) часу, O(r) пам'яті для результату та O(1)
  додаткової пам'яті; n і m — довжини входів, r — кількість залишків.

### `Program.cs` — заготовка без реалізації

```csharp
Interval[] available = [new(0, 10), new(15, 25)];
(string Name, Interval[] Available, Interval[] Blocked)[] examples =
[
    ("Часткове блокування", available, [new(3, 5), new(8, 18), new(22, 30)]),
    ("Лише дотик меж", available, [new(10, 15)]),
    ("Повне блокування", available, [new(0, 30)]),
    ("Без блоків", available, []),
    ("Немає доступних", [], [new(2, 4)]),
];

foreach (var example in examples)
{
    List<Interval> result = SubtractBlocked(example.Available, example.Blocked);
    string text = result.Count == 0
        ? "(порожньо)"
        : string.Join(' ', result.Select(interval => $"[{interval.Start}, {interval.End})"));
    Console.WriteLine($"{example.Name}: {text}");
}

static List<Interval> SubtractBlocked(Interval[] available, Interval[] blocked)
{
    throw new NotImplementedException();
}

public readonly record struct Interval(int Start, int End);
```

### Очікуваний вивід після реалізації

```text
Часткове блокування: [0, 3) [5, 8) [18, 22)
Лише дотик меж: [0, 10) [15, 25)
Повне блокування: (порожньо)
Без блоків: [0, 10) [15, 25)
Немає доступних: (порожньо)
```

Додатково перевірте блоки повністю ліворуч і праворуч від доступних проміжків,
кілька блоків усередині одного проміжку та незмінність обох вхідних масивів.

## Завдання 6. Гра з кульками у кільці — 2 бали

Гравці по черзі додають пронумеровані кульки до кільця. На спеціальних ходах
потрібно рухатися у зворотному напрямку й вилучати кульку. Обчисліть підсумкові
бали **кожного** гравця, використовуючи двозв'язний список.

### Правила

1. Спочатку кільце містить лише кульку **0**; вона є поточною.
2. Далі по черзі розігруються кульки `1, 2, ..., lastMarble`.
   Хід із кулькою m виконує гравець `(m - 1) % playerCount`; нумерація гравців — від 0.
3. Якщо m **не кратне 23**, від поточної кульки пройдіть одну позицію за
   годинниковою стрілкою й вставте нову кульку **після** знайденої.
   Нова кулька стає поточною; бали за цей хід не нараховуються.
4. Якщо m **кратне 23**, цю кульку не вставляйте. Від поточної кульки
   пройдіть **7 позицій проти годинникової стрілки**, вилучіть знайдену
   кульку й додайте гравцеві `m + значення вилученої кульки` балів.
   Поточною стає наступна за вилученою кулька за годинниковою стрілкою.
5. Після останнього вузла перехід за годинниковою стрілкою веде до першого;
   перед першим вузлом у зворотному напрямку розташований останній.

- Реалізуйте `PlayMarbles(playerCount, lastMarble)`, що повертає `long[]`
  із балами гравців у порядку їхніх номерів.
- Вхідні дані коректні: `1 <= playerCount <= 100`, `0 <= lastMarble <= 1_000_000`.
  При `lastMarble == 0` усі бали дорівнюють нулю.
- Використайте `LinkedList<int>` або власні двозв'язні вузли й зберігайте
  посилання на поточний вузол. Не шукайте його за значенням на кожному ході,
  не перетворюйте кільце на масив і не використовуйте `List<int>` для кільця.
- **Складність:** O(M + P) часу та пам'яті, де M — `lastMarble`, P — кількість
  гравців. Один хід потребує лише сталої кількості переходів між вузлами.

### `Program.cs` — заготовка без реалізації

```csharp
(int Players, int LastMarble)[] examples = [(3, 0), (1, 22), (2, 23), (3, 50)];

foreach (var example in examples)
{
    long[] scores = PlayMarbles(example.Players, example.LastMarble);
    Console.WriteLine($"Гравців: {example.Players}, остання кулька: {example.LastMarble}; бали: {string.Join(' ', scores)}");
}

static long[] PlayMarbles(int playerCount, int lastMarble)
{
    throw new NotImplementedException();
}
```

### Очікуваний вивід після реалізації

```text
Гравців: 3, остання кулька: 0; бали: 0 0 0
Гравців: 1, остання кулька: 22; бали: 0
Гравців: 2, остання кулька: 23; бали: 32 0
Гравців: 3, остання кулька: 50; бали: 63 32 0
```

На ході 23 вилучається кулька 9, тому гравець отримує `23 + 9 = 32` бали.
На ході 46 вилучається кулька 17, тому нараховується `46 + 17 = 63` бали.
Для трьох гравців ці ходи належать відповідно гравцям 1 і 0.
Додатково перевірте гру з одним гравцем, переходи через межі списку та довгу гру.

## Що здати та як оцінюється робота

Здайте два консольні проєкти з обраними завданнями, результатами запуску
на наведених і додаткових граничних прикладах та коротким поясненням складності.
Для власної структури реалізуйте **весь** зазначений API, а не лише методи,
які викликаються у демонстрації.

| Обрана частина | Критерій | Бали |
|---|---|---|
| Реалізація, одне із завдань 1–3 | Базові операції та інваріанти структури | 1 |
| Реалізація, одне із завдань 1–3 | Усі додаткові методи та заявлена складність | 1 |
| Реалізація, одне із завдань 1–3 | Граничні випадки, винятки, перевірки з різними типами | 1 |
| Прикладна задача, одне із завдань 4–6 | Правильний результат на звичайних і граничних даних | 1 |
| Прикладна задача, одне із завдань 4–6 | Виконання обмежень і пояснення складності | 1 |
| **Разом за два обрані завдання** | | **5** |
