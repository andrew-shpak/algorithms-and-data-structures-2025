# Лекція 3 — .NET колекції

---

## Зміст

| № | Розділ | Орієнтовний час |
|---|--------|-----------------|
| 0 | [Вступ: колекції в .NET](#0-вступ-колекції-в-net) | 5 хв |
| 1 | [Ієрархія інтерфейсів колекцій](#1-ієрархія-інтерфейсів-колекцій) | 20 хв |
| 2 | [Послідовності: від масиву до FrozenSet](#2-послідовності-від-масиву-до-frozenset) | 10 хв |
| 3 | [Dictionary<TKey,TValue> зсередини](#3-dictionarytkeytvalue-зсередини) | 20 хв |
| 4 | [HashSet<T> і операції над множинами](#4-hashsett-і-операції-над-множинами) | 15 хв |
| 5 | [Відсортовані колекції](#5-відсортовані-колекції) | 20 хв |
| 6 | [PriorityQueue та інші спеціалізовані колекції](#6-priorityqueue-та-інші-спеціалізовані-колекції) | 15 хв |
| 7 | [LINQ над колекціями](#7-linq-над-колекціями) | 15 хв |
| 8 | [Потокобезпечні, незмінні та frozen колекції](#8-потокобезпечні-незмінні-та-frozen-колекції) | 30 хв |
| 9 | [Продуктивність і вибір колекції](#9-продуктивність-і-вибір-колекції) | 10 хв |
| 10 | [Практичні задачі](#10-практичні-задачі) | 15 хв |
| 11 | [Підсумок, питання та завдання](#11-підсумок-питання-та-завдання) | 5 хв |
| | **Разом** (з трьома перервами по 5 хв) | **≈ 195 хв** |

---

## 0. Вступ: колекції в .NET

Простір імен **`System.Collections.Generic`** містить узагальнені (generic) колекції, а **`System.Linq`** — алгоритми над ними. Бібліотеку можна розкласти на три шари:

- **Колекції** — узагальнені структури даних (`List<T>`, `LinkedList<T>`, `Dictionary<TKey,TValue>`, ...).
- **Перелічувачі** — універсальний спосіб обходу через інтерфейс `IEnumerable<T>` і цикл `foreach`.
- **Алгоритми** — методи LINQ (`OrderBy`, `Where`, `Sum`, ...) та вбудовані методи (`Sort`, `IndexOf`, `BinarySearch`, ...).

Головна ідея: метод LINQ не знає, з якою колекцією працює, — він бачить лише `IEnumerable<T>`.

```
                 ┌─────────────────────────────┐
                 │  Алгоритми (LINQ, Sort ...) │
                 └──────────────┬──────────────┘
                                │ працюють через
                 ┌──────────────▼──────────────┐
                 │ IEnumerable<T> / IEnumerator │
                 └──────────────┬──────────────┘
                                │ реалізують
   ┌───────────┬───────────┬────┴──────┬──────────────┬──────────────┐
   │ List<T>   │ HashSet<T>│ Dictionary│ SortedSet<T> │ PriorityQueue│ ...
   └───────────┴───────────┴───────────┴──────────────┴──────────────┘
```

Де живуть колекції:

| Простір імен | Що містить |
|--------------|------------|
| `System.Collections.Generic` | `List`, `Dictionary`, `HashSet`, `SortedSet`, `SortedDictionary`, `SortedList`, `LinkedList`, `Queue`, `Stack`, `PriorityQueue` |
| `System.Collections.ObjectModel` | `Collection<T>`, `ObservableCollection<T>`, `ReadOnlyCollection<T>`, `ReadOnlyDictionary` |
| `System.Collections.Immutable` | `ImmutableArray<T>`, `ImmutableList<T>`, `ImmutableDictionary`, ... |
| `System.Collections.Frozen` (.NET 8) | `FrozenSet<T>`, `FrozenDictionary<TKey,TValue>` |
| `System.Collections.Concurrent` | `ConcurrentDictionary`, `ConcurrentQueue`, `BlockingCollection` |
| `System.Collections` | старі негенеричні `ArrayList`, `Hashtable` (не використовуйте в новому коді) та `BitArray` |

> **Зв'язок з лекцією 2.** Внутрішню будову масиву, `List<T>` (місткість, подвоєння, амортизований O(1)), а також реалізації зв'язного списку, стека й черги ми вже розібрали в лекції 2. Тут ми дивимось на них як **користувачі бібліотеки** і зосереджуємось на тому, чого ще не було: інтерфейси, хеш-таблиці, дерева, купи, LINQ і вибір колекції.

Кожен приклад нижче — окрема консольна програма з top-level statements. Запуск: `dotnet new console`, вставити код у `Program.cs`, `dotnet run` (або в .NET 10+ просто `dotnet run app.cs`). Nullable-анотації увімкнено.

---

## 1. Ієрархія інтерфейсів колекцій

### 1.1. Загальна картина

```
IEnumerable<T>                     — «можна обійти foreach»
 ├── IReadOnlyCollection<T>        — + Count
 │    ├── IReadOnlyList<T>         — + this[int] (лише читання)
 │    ├── IReadOnlySet<T>          — + Contains, IsSubsetOf ... (.NET 5)
 │    └── IReadOnlyDictionary<K,V> — + TryGetValue, Keys, Values
 └── ICollection<T>                — + Count, Add, Remove, Contains, Clear, CopyTo
      ├── IList<T>                 — + this[int] get/set, Insert, RemoveAt, IndexOf
      ├── ISet<T>                  — + UnionWith, IntersectWith, ...
      └── IDictionary<K,V>         — ICollection<KeyValuePair<K,V>> + this[key], TryGetValue
```

Хто що реалізує:

| Тип | `IEnumerable<T>` | `ICollection<T>` | `IList<T>` | `ISet<T>` | `IDictionary` | `IReadOnlyList<T>` |
|-----|:---:|:---:|:---:|:---:|:---:|:---:|
| `T[]` | ✔ | ✔ | ✔ | | | ✔ |
| `List<T>` | ✔ | ✔ | ✔ | | | ✔ |
| `LinkedList<T>` | ✔ | ✔ | | | | |
| `HashSet<T>` / `SortedSet<T>` | ✔ | ✔ | | ✔ | | |
| `Dictionary<K,V>` / `SortedDictionary<K,V>` | ✔ | ✔ | | | ✔ | |
| `Queue<T>` / `Stack<T>` | ✔ | | | | | |
| `PriorityQueue<E,P>` | | | | | | |

`Queue<T>` і `Stack<T>` реалізують лише `IEnumerable<T>` та `IReadOnlyCollection<T>` — у них немає довільного `Add`/`Remove`. `PriorityQueue` не реалізує навіть `IEnumerable<T>` (є лише властивість `UnorderedItems`), бо «обхід купи» не має корисного порядку.

### 1.2. `IEnumerable<T>` та `IEnumerator<T>` — курсор

```csharp
// Спрощене оголошення з BCL (не програма)
public interface IEnumerable<out T>
{
    IEnumerator<T> GetEnumerator();   // кожен виклик дає НОВИЙ незалежний курсор
}

public interface IEnumerator<out T> : IDisposable
{
    T Current { get; }                // поточний елемент (валідний лише після MoveNext() == true)
    bool MoveNext();                  // крок уперед; false — елементи закінчились
    void Reset();                     // історичний метод, на практиці не використовується
}
```

Цикл `foreach` компілятор розгортає приблизно так:

```csharp
List<string> names = ["Ann", "Bob", "Cid"];

// foreach (string name in names) Console.WriteLine(name);
// перетворюється компілятором на:
using (List<string>.Enumerator e = names.GetEnumerator()) // struct-перелічувач List<T> — без алокацій
{
    while (e.MoveNext())                                  // спочатку курсор стоїть ПЕРЕД першим елементом
    {
        string name = e.Current;                          // читаємо поточний
        Console.WriteLine(name);
    }
}                                                         // Dispose викликається навіть при винятку
```

**Приклад запуску:**
```
Ann
Bob
Cid
```

> `foreach` не вимагає саме інтерфейсу — достатньо публічного методу `GetEnumerator()`, що повертає тип з `MoveNext()` і `Current` (duck typing). Саме тому `List<T>.Enumerator` — це `struct`, і `foreach` по `List<T>` не створює об'єктів у купі. Якщо ж змінна має тип `IEnumerable<T>`, перелічувач буде упакований (boxing).

### 1.3. Власний перелічувач вручну

Щоб зрозуміти, що робить `yield`, спочатку напишемо перелічувач «руками» — діапазон цілих чисел з кроком.

```csharp
var range = new StepRange(start: 1, end: 10, step: 3); // 1, 4, 7, 10
foreach (int x in range)                               // foreach викличе GetEnumerator()
{
    Console.Write($"{x} ");
}
Console.WriteLine();
Console.WriteLine($"Sum via LINQ = {range.Sum()}");     // LINQ бачить звичайний IEnumerable<int>

// Послідовність цілих чисел [start..end] з кроком step
public sealed class StepRange(int start, int end, int step) : IEnumerable<int>
{
    public IEnumerator<int> GetEnumerator() => new Enumerator(start, end, step);

    // Негенерична версія потрібна для сумісності зі старим IEnumerable
    System.Collections.IEnumerator System.Collections.IEnumerable.GetEnumerator() => GetEnumerator();

    private sealed class Enumerator(int start, int end, int step) : IEnumerator<int>
    {
        private int _current = start - step; // стоїмо «перед» першим елементом
        private bool _started;

        public int Current => _started ? _current : throw new InvalidOperationException("Call MoveNext first");
        object System.Collections.IEnumerator.Current => Current;

        public bool MoveNext()
        {
            if (_started && _current + step > end)
            {
                return false;                // далі виходимо за межу
            }
            _started = true;
            _current += step;                // робимо крок
            return _current <= end;
        }

        public void Reset() => throw new NotSupportedException();
        public void Dispose() { }            // ресурсів немає
    }
}
```

**Приклад запуску:**
```
1 4 7 10 
Sum via LINQ = 22
```

Багато шаблонного коду: стан, прапорець, два `Current`, `Dispose`. Ключове слово `yield` генерує саме такий клас автоматично.

### 1.4. Ітератори з `yield` і відкладене виконання

```csharp
// Той самий діапазон, але через ітератор: компілятор згенерує state machine
static IEnumerable<int> StepRange(int start, int end, int step)
{
    Console.WriteLine("  [start iterating]");          // виконається лише при першому MoveNext()
    for (int i = start; i <= end; i += step)
    {
        Console.WriteLine($"  [yield {i}]");
        yield return i;                                 // «заморожуємо» метод і віддаємо значення
    }
    Console.WriteLine("  [done]");
}

IEnumerable<int> seq = StepRange(1, 7, 3);             // ЖОДЕН рядок тіла ще не виконано!
Console.WriteLine("Sequence created");

foreach (int x in seq)
{
    Console.WriteLine($"got {x}");                      // значення приходять по одному «на вимогу»
}

Console.WriteLine("Take only first:");
foreach (int x in StepRange(1, 7, 3))
{
    Console.WriteLine($"got {x}");
    break;                                              // break викликає Dispose — тіло ітератора не дійде до [done]
}
```

**Приклад запуску:**
```
Sequence created
  [start iterating]
  [yield 1]
got 1
  [yield 4]
got 4
  [yield 7]
got 7
  [done]
Take only first:
  [start iterating]
  [yield 1]
got 1
```

Відкладене (lazy) виконання дає змогу описувати навіть **нескінченні** послідовності:

```csharp
static IEnumerable<long> Fibonacci()
{
    long a = 0, b = 1;
    while (true)                         // нескінченний цикл — це нормально для ітератора
    {
        yield return a;
        (a, b) = (b, a + b);             // кортеж: одночасне присвоєння
    }
}

// Беремо рівно стільки, скільки потрібно: Take зупинить перелічення
Console.WriteLine(string.Join(", ", Fibonacci().Take(10)));

// Перше число Фібоначчі, більше за мільйон
Console.WriteLine(Fibonacci().First(f => f > 1_000_000));

// yield break — дострокове завершення послідовності
static IEnumerable<int> Digits(int n)
{
    if (n < 0)
    {
        yield break;                     // від'ємні числа — порожня послідовність
    }
    foreach (char c in n.ToString())
    {
        yield return c - '0';
    }
}
Console.WriteLine($"digits: [{string.Join(",", Digits(9075))}], negative: [{string.Join(",", Digits(-5))}]");
```

**Приклад запуску:**
```
0, 1, 1, 2, 3, 5, 8, 13, 21, 34
1346269
digits: [9,0,7,5], negative: []
```

**Типові помилки з ітераторами:**

1. **Перевірка аргументів «запізнюється».** Виняток у тілі ітератора виникне не при виклику методу, а при першому `MoveNext()`. Рішення — зовнішній метод перевіряє аргументи, внутрішня локальна функція містить `yield`.
2. **Повторне перелічення виконує тіло знову** (див. розділ 7.3).
3. **`yield` не можна** використовувати в блоці `catch`, у `finally`, в анонімних методах та в методах з `ref`/`out` параметрами.

```csharp
static IEnumerable<int> Repeat(int value, int count)
{
    ArgumentOutOfRangeException.ThrowIfNegative(count); // перевірка виконується ОДРАЗУ при виклику
    return Iterator();

    IEnumerable<int> Iterator()                         // локальна функція-ітератор
    {
        for (int i = 0; i < count; i++)
        {
            yield return value;
        }
    }
}

try
{
    IEnumerable<int> bad = Repeat(7, -1);               // виняток тут, а не десь пізніше
    Console.WriteLine("not reached");
}
catch (ArgumentOutOfRangeException)
{
    Console.WriteLine("caught immediately");
}
Console.WriteLine(string.Join(" ", Repeat(7, 3)));
```

**Приклад запуску:**
```
caught immediately
7 7 7
```

### 1.5. Модифікація під час обходу

Перелічувачі `List<T>`, `Dictionary`, `HashSet` зберігають «версію» колекції. Будь-яка зміна (`Add`, `Remove`) збільшує версію, і наступний `MoveNext()` кидає виняток.

```csharp
List<int> numbers = [1, 2, 3, 4, 5, 6];

try
{
    foreach (int n in numbers)
    {
        if (n % 2 == 0)
        {
            numbers.Remove(n);          // змінюємо колекцію під час обходу
        }
    }
}
catch (InvalidOperationException ex)
{
    Console.WriteLine($"Error: {ex.GetType().Name}");
}

// Правильно 1: RemoveAll — один прохід O(n)
List<int> a = [1, 2, 3, 4, 5, 6];
a.RemoveAll(n => n % 2 == 0);
Console.WriteLine(string.Join(" ", a));

// Правильно 2: обхід індексами з кінця (видалення не зсуває ще не переглянуті елементи)
List<int> b = [1, 2, 3, 4, 5, 6];
for (int i = b.Count - 1; i >= 0; i--)
{
    if (b[i] % 2 == 0)
    {
        b.RemoveAt(i);
    }
}
Console.WriteLine(string.Join(" ", b));

// Правильно 3: для словника — спочатку зібрати ключі, потім видалити
var stock = new Dictionary<string, int> { ["apple"] = 0, ["pear"] = 5, ["plum"] = 0 };
foreach (string key in stock.Where(p => p.Value == 0).Select(p => p.Key).ToList()) // ToList робить копію
{
    stock.Remove(key);
}
Console.WriteLine(string.Join(", ", stock.Keys));
```

**Приклад запуску:**
```
Error: InvalidOperationException
1 3 5
1 3 5
pear
```

> Починаючи з .NET Core 3.0, `Dictionary.Remove` під час `foreach` по самому словнику **дозволено** (він не змінює версію). Але покладатися на це в навчальному коді не варто — варіант з копією ключів працює для будь-якої колекції.

### 1.6. `ICollection<T>`, `IList<T>` та read-only інтерфейси

```csharp
// Один метод працює з будь-якою колекцією, яка має Count і Add
static void AddRange<T>(ICollection<T> target, params T[] items)
{
    foreach (T item in items)
    {
        target.Add(item);                           // List.Add, HashSet.Add, LinkedList.AddLast ...
    }
}

var list = new List<int>();
var set = new HashSet<int>();
var linked = new LinkedList<int>();

AddRange(list, 3, 1, 3);
AddRange(set, 3, 1, 3);                             // ICollection<T>.Add для HashSet просто ігнорує дублікат
AddRange(linked, 3, 1, 3);

Console.WriteLine($"list={list.Count} set={set.Count} linked={linked.Count}");

// IList<T> — додає індексатор. Масив реалізує IList<T>, але має фіксований розмір!
IList<int> array = new int[] { 10, 20, 30 };
array[1] = 25;                                      // запис за індексом — ок
Console.WriteLine($"array[1]={array[1]}, IsReadOnly={array.IsReadOnly}"); // True: розмір масиву змінити не можна
try
{
    array.Add(40);                                  // масив не може рости
}
catch (NotSupportedException)
{
    Console.WriteLine("int[] as IList<int>: Add is not supported");
}

// IReadOnlyList<T> — «обіцянка» не змінювати: у інтерфейсі просто немає методів запису
IReadOnlyList<int> view = list;
Console.WriteLine($"view[0]={view[0]}, view.Count={view.Count}");
// view.Add(5);                                     // помилка компіляції: немає такого методу
list.Add(99);                                       // але власник колекції МОЖЕ її змінити
Console.WriteLine($"view sees change: Count={view.Count}");
```

**Приклад запуску:**
```
list=3 set=2 linked=3
array[1]=25, IsReadOnly=True
int[] as IList<int>: Add is not supported
view[0]=3, view.Count=3
view sees change: Count=4
```

> **Read-only ≠ immutable.** `IReadOnlyList<T>` означає «*ти* через це посилання не зміниш», але хтось інший може. Незмінність гарантують лише `ImmutableArray<T>`, `FrozenSet<T>` тощо (розділ 2).

### 1.7. `ISet<T>`, `IDictionary<TKey,TValue>`, `IReadOnlyDictionary<TKey,TValue>`

```csharp
ISet<string> tags = new SortedSet<string> { "csharp", "dotnet" };   // можна підмінити на HashSet
bool added = tags.Add("linq");                                     // ISet<T>.Add повертає bool
bool again = tags.Add("linq");
Console.WriteLine($"added={added} again={again} tags={string.Join(",", tags)}");

IDictionary<string, int> ages = new Dictionary<string, int>();
ages["Ann"] = 20;                                                  // індексатор: додати або перезаписати
ages.Add("Bob", 25);                                               // Add: кине виняток, якщо ключ існує
Console.WriteLine($"Ann={ages["Ann"]} hasBob={ages.ContainsKey("Bob")}");

// Метод, що лише читає словник, повинен приймати IReadOnlyDictionary
static int TotalAge(IReadOnlyDictionary<string, int> people) => people.Values.Sum();

Console.WriteLine($"total={TotalAge((Dictionary<string, int>)ages)}");
```

**Приклад запуску:**
```
added=True again=False tags=csharp,dotnet,linq
Ann=20 hasBob=True
total=45
```

> Цікава деталь: `IDictionary<K,V>` **не успадковує** `IReadOnlyDictionary<K,V>` (історичні причини сумісності), тому в прикладі потрібне приведення до конкретного `Dictionary`, який реалізує обидва.

### 1.8. Які типи приймати й повертати

**Правило Постела для API:** *приймай найзагальніше, повертай найконкретніше, що не порушує інкапсуляцію.*

| Ситуація | Параметр | Повернення |
|----------|----------|------------|
| Лише обхід один раз | `IEnumerable<T>` | — |
| Потрібні `Count` / кілька проходів | `IReadOnlyCollection<T>` | `IReadOnlyCollection<T>` |
| Потрібен доступ за індексом | `IReadOnlyList<T>` | `IReadOnlyList<T>` або `T[]` |
| Метод змінює передану колекцію | `ICollection<T>` / `IList<T>` | — |
| Перевірка належності | `IReadOnlySet<T>` | `IReadOnlySet<T>` |
| Пошук за ключем | `IReadOnlyDictionary<K,V>` | `IReadOnlyDictionary<K,V>` |
| Внутрішнє поле класу | — | не віддавайте `List<T>` напряму! |

```csharp
var course = new Course("Algorithms");
course.Enroll("Ann");
course.Enroll("Bob");
course.Enroll("Ann");                                    // дублікат ігнорується всередині класу

IReadOnlyList<string> students = course.Students;        // зовні — лише читання
Console.WriteLine($"{course.Title}: {string.Join(", ", students)} ({students.Count})");

// ((List<string>)students).Clear(); — так «зламати» можна, тому для повної безпеки є AsReadOnly()
Console.WriteLine($"Is wrapper: {students is System.Collections.ObjectModel.ReadOnlyCollection<string>}");

public sealed class Course(string title)
{
    private readonly List<string> _students = [];        // внутрішній стан — конкретний List<T>

    public string Title { get; } = title;

    // AsReadOnly() повертає обгортку ReadOnlyCollection<T>: приведення до List<T> вже не спрацює
    public IReadOnlyList<string> Students => _students.AsReadOnly();

    public void Enroll(string name)
    {
        if (!_students.Contains(name))                   // бізнес-правило живе в одному місці
        {
            _students.Add(name);
        }
    }
}
```

**Приклад запуску:**
```
Algorithms: Ann, Bob (2)
Is wrapper: True
```

**Типові помилки розділу 1:**

- Приймати `List<T>` там, де вистачає `IEnumerable<T>` — метод стає непридатним для масивів, множин і LINQ-запитів.
- Приймати `IEnumerable<T>` і тричі його обходити (`Count()`, `First()`, `foreach`) — див. розділ 7.3.
- Повертати внутрішній `List<T>` з властивості — будь-хто може змінити стан об'єкта.
- Плутати `IReadOnlyList<T>` з незмінним списком.

**Міні-вправа 1.** Напишіть ітератор `IEnumerable<T> EveryNth<T>(IEnumerable<T> source, int n)`, що повертає кожен n-й елемент (1-й, (n+1)-й, ...). Перевірка `n <= 0` має кидати виняток одразу.

<details>
<summary>Розв'язок</summary>

```csharp
static IEnumerable<T> EveryNth<T>(IEnumerable<T> source, int n)
{
    ArgumentNullException.ThrowIfNull(source);
    ArgumentOutOfRangeException.ThrowIfNegativeOrZero(n); // eager-перевірка
    return Iterate();

    IEnumerable<T> Iterate()
    {
        int index = 0;
        foreach (T item in source)
        {
            if (index % n == 0)
            {
                yield return item;                         // lazy-частина
            }
            index++;
        }
    }
}

Console.WriteLine(string.Join(" ", EveryNth("abcdefghij", 3)));  // string — це IEnumerable<char>
Console.WriteLine(string.Join(" ", EveryNth(Enumerable.Range(1, 10), 4)));
```

**Приклад запуску:**
```
a d g j
1 5 9
```

</details>

---

## 2. Послідовності: від масиву до FrozenSet

### 2.1. Коротке нагадування (детально — лекція 2)

| Операція | `T[]` | `List<T>` | `LinkedList<T>` |
|----------|:-----:|:---------:|:---------------:|
| Доступ за індексом | O(1) | O(1) | O(n) (індексатора немає) |
| Додати в кінець | — | O(1) амортизовано | O(1) |
| Додати на початок | — | O(n) | O(1) |
| Вставка / видалення в середині | — | O(n) | O(1) за відомим вузлом |
| Пошук значення | O(n) / O(log n) `BinarySearch` | O(n) / O(log n) `BinarySearch` | O(n) |
| Пам'ять на елемент (`int`, x64) | 4 байти | 4 байти (+ резерв місткості) | ≈ 48 байт (об'єкт-вузол) |
| Локальність кешу | відмінна | відмінна | погана |

Висновок лекції 2 залишається в силі: **за замовчуванням — `List<T>` або масив.**

Кілька корисних методів `List<T>`, яких не було в лекції 2:

```csharp
List<int> data = [5, 3, 8, 1, 9, 2];

data.Sort();                                                  // інтроспективне сортування (IntroSort), O(n log n)
int idx = data.BinarySearch(8);                               // лише для відсортованого списку
int missing = data.BinarySearch(4);                           // від'ємне число: ~missing — позиція вставки
Console.WriteLine($"sorted={string.Join(",", data)} idx(8)={idx} insertAt(4)={~missing}");

data.Insert(~missing, 4);                                     // вставка зі збереженням порядку
Console.WriteLine($"after insert: {string.Join(",", data)}");

List<int> big = data.FindAll(x => x > 4);                     // новий список за предикатом
int firstBig = data.Find(x => x > 4);                         // перший підхожий (або default)
bool anyNeg = data.Exists(x => x < 0);
Console.WriteLine($"big={string.Join(",", big)} first={firstBig} anyNeg={anyNeg}");

List<int> slice = data.GetRange(1, 3);                        // копія підсписку [1..4)
data.Reverse();                                               // на місці
Console.WriteLine($"slice={string.Join(",", slice)} reversed={string.Join(",", data)}");

data.Sort((x, y) => y.CompareTo(x));                          // Comparison<T>: за спаданням
Console.WriteLine($"desc={string.Join(",", data)}");

int[] array = data.ToArray();                                 // копія у масив
Span<int> span = System.Runtime.InteropServices.CollectionsMarshal.AsSpan(data); // Span над внутрішнім масивом — без копії
span[0] = 100;                                                // змінюємо сам список!
Console.WriteLine($"array[0]={array[0]} data[0]={data[0]}");
```

**Приклад запуску:**
```
sorted=1,2,3,5,8,9 idx(8)=4 insertAt(4)=3
after insert: 1,2,3,4,5,8,9
big=5,8,9 first=5 anyNeg=False
slice=2,3,4 reversed=9,8,5,4,3,2,1
desc=9,8,5,4,3,2,1
array[0]=9 data[0]=100
```

### 2.2. `Collection<T>` і `ObservableCollection<T>` — точки розширення

`List<T>` **не має віртуальних методів** — успадкувати його й «перехопити» `Add` неможливо. Для цього існує `Collection<T>` з простору `System.Collections.ObjectModel`: це обгортка над `IList<T>` з віртуальними `InsertItem`, `SetItem`, `RemoveItem`, `ClearItems`.

```csharp
using System.Collections.ObjectModel;
using System.Collections.Specialized;

var grades = new GradeCollection { 90, 75 };        // ініціалізатор колекції викликає Add -> InsertItem
try
{
    grades.Add(120);                                 // перехоплюємо некоректне значення
}
catch (ArgumentOutOfRangeException ex)
{
    Console.WriteLine($"rejected: {ex.ActualValue}");   // ActualValue — значення, що не пройшло перевірку
}
Console.WriteLine($"grades: {string.Join(", ", grades)}");

// ObservableCollection<T> повідомляє підписників про кожну зміну (використовується у WPF/MAUI для UI)
var todo = new ObservableCollection<string>();
todo.CollectionChanged += (_, e) =>
{
    string items = string.Join(",", (e.NewItems ?? e.OldItems)?.Cast<string>() ?? []);
    Console.WriteLine($"  event: {e.Action} [{items}]");
};
todo.Add("buy milk");
todo.Add("learn C#");
todo.Move(1, 0);                                     // переміщення теж подія
todo.Remove("buy milk");
Console.WriteLine($"todo: {string.Join(", ", todo)}");

// Колекція оцінок, що відкидає значення поза [0, 100]
public sealed class GradeCollection : Collection<int>
{
    protected override void InsertItem(int index, int item)
    {
        ArgumentOutOfRangeException.ThrowIfGreaterThan(item, 100);
        ArgumentOutOfRangeException.ThrowIfNegative(item);
        base.InsertItem(index, item);                // справжня вставка у внутрішній List<int>
    }

    protected override void SetItem(int index, int item)
    {
        ArgumentOutOfRangeException.ThrowIfGreaterThan(item, 100);
        base.SetItem(index, item);
    }
}
```

**Приклад запуску:**
```
rejected: 120
grades: 90, 75
  event: Add [buy milk]
  event: Add [learn C#]
  event: Move [learn C#]
  event: Remove [buy milk]
todo: learn C#
```

### 2.3. `ReadOnlyCollection<T>`, immutable- та frozen-колекції

Три різні рівні «незмінності»:

| Тип | Хто може змінити | Вартість «зміни» | Для чого |
|-----|------------------|------------------|----------|
| `ReadOnlyCollection<T>` (`list.AsReadOnly()`) | власник оригінального `List<T>` | — (обгортка, O(1) створення) | безпечно віддати назовні |
| `ImmutableArray<T>` | ніхто | O(n) — копіювання масиву | маленькі незмінні набори, швидке читання |
| `ImmutableList<T>` | ніхто | O(log n) — AVL-дерево зі спільними вузлами | часті «зміни» зі збереженням старих версій |
| `FrozenSet<T>` / `FrozenDictionary<K,V>` (.NET 8) | ніхто, змін немає взагалі | створення дороге, читання — найшвидше | довідники, що будуються один раз при старті |

> Тут — лише огляд. Детально (спільні вузли, builder-и, `ImmutableInterlocked`, оптимізації frozen-колекцій і порівняння з конкурентними) — у розділі 8.

```csharp
using System.Collections.Frozen;
using System.Collections.Immutable;
using System.Collections.ObjectModel;

// 1) ReadOnlyCollection — вікно на живий список
var source = new List<int> { 1, 2, 3 };
ReadOnlyCollection<int> readOnly = source.AsReadOnly();
source.Add(4);                                              // змінюємо оригінал
Console.WriteLine($"readOnly sees: {string.Join(",", readOnly)}");

// 2) ImmutableArray — кожна «зміна» повертає НОВИЙ масив
ImmutableArray<int> v1 = [1, 2, 3];                         // вираз колекції працює і тут
ImmutableArray<int> v2 = v1.Add(4);                         // v1 не змінився
ImmutableArray<int> v3 = v2.SetItem(0, 100);
Console.WriteLine($"v1={string.Join(",", v1)} v2={string.Join(",", v2)} v3={string.Join(",", v3)}");

// Builder — ефективно накопичити багато змін і «заморозити» один раз
ImmutableArray<int>.Builder builder = ImmutableArray.CreateBuilder<int>();
for (int i = 0; i < 5; i++)
{
    builder.Add(i * i);
}
ImmutableArray<int> squares = builder.ToImmutable();
Console.WriteLine($"squares={string.Join(",", squares)}");

// 3) ImmutableList — дерево: старі й нові версії ділять більшість вузлів
ImmutableList<string> history0 = ImmutableList<string>.Empty;
ImmutableList<string> history1 = history0.Add("open");
ImmutableList<string> history2 = history1.Add("edit");
ImmutableList<string> undo = history1;                      // «відкат» — просто посилання на стару версію
Console.WriteLine($"h2={string.Join("->", history2)} undo={string.Join("->", undo)}");

// 4) FrozenSet / FrozenDictionary — оптимізовані під читання після одноразової побудови
FrozenSet<string> keywords = new[] { "if", "else", "for", "while", "return" }.ToFrozenSet();
FrozenDictionary<string, int> httpCodes = new Dictionary<string, int>
{
    ["OK"] = 200,
    ["NotFound"] = 404,
    ["Teapot"] = 418,
}.ToFrozenDictionary();

Console.WriteLine($"'for' is keyword: {keywords.Contains("for")}, 'var': {keywords.Contains("var")}");
Console.WriteLine($"NotFound -> {httpCodes["NotFound"]}");
// keywords.Add("var");                                     // методу немає: FrozenSet не має API зміни
```

**Приклад запуску:**
```
readOnly sees: 1,2,3,4
v1=1,2,3 v2=1,2,3,4 v3=100,2,3,4
squares=0,1,4,9,16
h2=open->edit undo=open
'for' is keyword: True, 'var': False
NotFound -> 404
```

**Типові помилки розділу 2:**

- Використовувати `ImmutableList<T>` «бо незмінний = швидкий». Насправді читання за індексом там O(log n) і повільніше за `List<T>` у рази; для рідко змінюваних даних кращий `ImmutableArray<T>`.
- Будувати `ImmutableArray` в циклі через `arr = arr.Add(x)` — це O(n²). Використовуйте `Builder`.
- Створювати `FrozenDictionary` на кожен запит — побудова дорожча за звичайний `Dictionary`, виграш є лише при багатьох читаннях.
- Вважати `AsReadOnly()` знімком даних — це вікно, воно бачить подальші зміни.

**Міні-вправа 2.** Клас `Playlist` зберігає пісні у `List<string>`. Зробіть властивість `Songs`, через яку неможливо змінити плейлист навіть приведенням типу, і метод `Snapshot()`, що повертає незмінну копію, яка **не** бачить подальших змін.

<details>
<summary>Розв'язок</summary>

```csharp
using System.Collections.Immutable;

var playlist = new Playlist();
playlist.Add("Song A");
ImmutableArray<string> snapshot = playlist.Snapshot();
playlist.Add("Song B");

Console.WriteLine($"live: {string.Join(", ", playlist.Songs)}");
Console.WriteLine($"snapshot: {string.Join(", ", snapshot)}");
Console.WriteLine($"cast to List works? {playlist.Songs is List<string>}");

public sealed class Playlist
{
    private readonly List<string> _songs = [];

    public IReadOnlyList<string> Songs => _songs.AsReadOnly();      // обгортка — «вікно»

    public void Add(string song) => _songs.Add(song);

    public ImmutableArray<string> Snapshot() => [.. _songs];         // копія O(n)
}
```

**Приклад запуску:**
```
live: Song A, Song B
snapshot: Song A
cast to List works? False
```

</details>

---

## 3. Dictionary<TKey,TValue> зсередини

`Dictionary<TKey,TValue>` — найважливіша колекція після `List<T>`. Пошук, вставка й видалення за ключем — **O(1) у середньому**. Щоб розуміти, коли ця оцінка ламається, треба знати, як він влаштований.

### 3.1. Ідея хеш-таблиці

1. Ключ перетворюється на ціле число — **хеш-код** (`key.GetHashCode()`).
2. Хеш-код зводиться до номера **кошика** (bucket): `bucket = hash % buckets.Length`.
3. У кошику шукаємо ключ, порівнюючи через `Equals`.

Якщо два різні ключі потрапили в один кошик — це **колізія**. .NET розв'язує колізії **методом ланцюжків**, але ланцюжки зберігаються не у вузлах, а в масиві `entries` через індекс `next`:

```
Dictionary<int, string> після Add(3,"c"), Add(10,"j"), Add(7,"g"), Add(17,"q"); buckets.Length = 7

buckets (1-based індекс у entries; 0 = порожньо)
  idx:   0    1    2    3    4    5    6
       ┌────┬────┬────┬────┬────┬────┬────┐
       │ 3  │ 0  │ 0  │ 4  │ 0  │ 0  │ 0  │
       └─┬──┴────┴────┴─┬──┴────┴────┴────┘
         │              │
         │              └──────────────┐
         ▼                             ▼
entries  #0              #1              #2              #3
       ┌───────────────┬───────────────┬───────────────┬───────────────┐
 hash  │ 3             │ 10            │ 7             │ 17            │
 key   │ 3             │ 10            │ 7             │ 17            │
 value │ "c"           │ "j"           │ "g"           │ "q"           │
 next  │ -1            │ 0 ──►#0       │ -1            │ 1 ──►#1       │
       └───────────────┴───────────────┴───────────────┴───────────────┘

 3 % 7 = 3, 10 % 7 = 3, 17 % 7 = 3  → ланцюжок у кошику 3: #3 → #1 → #0
 7 % 7 = 0                          → кошик 0: #2
```

Ключові деталі реалізації .NET:

| Деталь | Як зроблено |
|--------|-------------|
| Розмір `buckets` | просте число (3, 7, 17, 37, 79, 163, ...) — краще розсіює хеші |
| Вставка | новий запис кладеться в кінець `entries` (або у «дірку» з free list) і стає головою ланцюжка |
| Видалення | запис позначається вільним і додається до **free list**; масив не зсувається |
| Коефіцієнт заповнення | ≤ 1: коли `entries` заповнений, обидва масиви збільшуються ≈ удвічі (до наступного простого) |
| Resize | O(n): усі записи перерозподіляються по новому масиву кошиків |
| Кешування хешу | `hashCode` зберігається в записі, щоб при resize не викликати `GetHashCode` повторно |
| Захист від атак | для `string`-ключів використовується рандомізований хеш (Marvin), тож хеш-коди рядків різні між запусками |

### 3.2. Навчальна модель хеш-таблиці

Спрощена реалізація тієї ж ідеї (кошики + масив записів з `next`). Вона не замінює бібліотечну, але показує все, що відбувається всередині.

```csharp
var map = new MiniHashMap<int, string>(capacity: 3);
foreach (int key in new[] { 3, 10, 7, 17, 5 })
{
    map.Put(key, $"v{key}");
    Console.WriteLine($"Put {key,2}: count={map.Count} buckets={map.BucketCount}");
}
map.Put(10, "ten");                                   // перезапис існуючого ключа
Console.WriteLine($"Get 10 = {map.Get(10)}, Get 4 = {map.Get(4) ?? "null"}");
map.PrintLayout();

// Спрощена хеш-таблиця з ланцюжками в масиві записів (як у .NET Dictionary)
public sealed class MiniHashMap<TKey, TValue> where TKey : notnull
{
    private static readonly int[] Primes = [3, 7, 17, 37, 79, 163, 331];

    private int[] _buckets;           // 1-based індекс першого запису в ланцюжку; 0 — порожній кошик
    private Entry[] _entries;         // самі дані
    private int _count;

    private struct Entry
    {
        public int HashCode;
        public int Next;              // індекс наступного запису в ланцюжку або -1
        public TKey Key;
        public TValue Value;
    }

    public MiniHashMap(int capacity)
    {
        int size = Primes.First(p => p >= capacity);
        _buckets = new int[size];
        _entries = new Entry[size];
    }

    public int Count => _count;
    public int BucketCount => _buckets.Length;

    public void Put(TKey key, TValue value)
    {
        int hash = key.GetHashCode() & 0x7FFFFFFF;           // прибираємо знак
        int bucket = hash % _buckets.Length;

        for (int i = _buckets[bucket] - 1; i >= 0; i = _entries[i].Next)  // йдемо по ланцюжку
        {
            if (_entries[i].HashCode == hash && EqualityComparer<TKey>.Default.Equals(_entries[i].Key, key))
            {
                _entries[i].Value = value;                   // ключ уже є — перезаписуємо
                return;
            }
        }

        if (_count == _entries.Length)
        {
            Resize();                                        // місця немає — росте вся таблиця
            bucket = hash % _buckets.Length;                 // номер кошика змінився!
        }

        ref Entry entry = ref _entries[_count];
        entry.HashCode = hash;
        entry.Key = key;
        entry.Value = value;
        entry.Next = _buckets[bucket] - 1;                   // новий запис стає головою ланцюжка
        _buckets[bucket] = _count + 1;
        _count++;
    }

    public TValue? Get(TKey key)
    {
        int hash = key.GetHashCode() & 0x7FFFFFFF;
        for (int i = _buckets[hash % _buckets.Length] - 1; i >= 0; i = _entries[i].Next)
        {
            if (_entries[i].HashCode == hash && EqualityComparer<TKey>.Default.Equals(_entries[i].Key, key))
            {
                return _entries[i].Value;
            }
        }
        return default;
    }

    private void Resize()
    {
        int newSize = Primes.First(p => p >= _entries.Length * 2);
        Array.Resize(ref _entries, newSize);                 // копіюємо записи
        _buckets = new int[newSize];                         // кошики будуємо заново
        for (int i = 0; i < _count; i++)
        {
            int bucket = _entries[i].HashCode % newSize;     // використовуємо збережений хеш
            _entries[i].Next = _buckets[bucket] - 1;
            _buckets[bucket] = i + 1;
        }
    }

    public void PrintLayout()
    {
        for (int b = 0; b < _buckets.Length; b++)
        {
            var chain = new List<string>();
            for (int i = _buckets[b] - 1; i >= 0; i = _entries[i].Next)
            {
                chain.Add($"{_entries[i].Key}:{_entries[i].Value}");
            }
            if (chain.Count > 0)
            {
                Console.WriteLine($"  bucket {b}: {string.Join(" -> ", chain)}");
            }
        }
    }
}
```

**Приклад запуску:**
```
Put  3: count=1 buckets=3
Put 10: count=2 buckets=3
Put  7: count=3 buckets=3
Put 17: count=4 buckets=7
Put  5: count=5 buckets=7
Get 10 = ten, Get 4 = null
  bucket 0: 7:v7
  bucket 3: 17:v17 -> 10:ten -> 3:v3
  bucket 5: 5:v5
```

Складність операцій `Dictionary<TKey,TValue>`:

| Операція | Середній випадок | Найгірший випадок | Коли найгірший |
|----------|:---------------:|:-----------------:|----------------|
| `TryGetValue`, `ContainsKey`, `this[key]` get | O(1) | O(n) | усі ключі в одному кошику (поганий `GetHashCode`) |
| `Add`, `TryAdd`, `this[key]` set | O(1) амортизовано | O(n) | resize або колізії |
| `Remove` | O(1) | O(n) | колізії |
| `ContainsValue` | O(n) | O(n) | завжди лінійний пошук |
| Перелічення | O(n + capacity) | — | після масових видалень «діри» теж обходяться |
| `new Dictionary(capacity)` | O(capacity) | — | заздалегідь виділяє масиви, уникаючи resize |

### 3.3. Контракт `GetHashCode` / `Equals`

Будь-який тип, що використовується як ключ, має виконувати правила:

1. Якщо `a.Equals(b)`, то **обов'язково** `a.GetHashCode() == b.GetHashCode()`.
2. Рівні хеш-коди **не означають** рівність (колізії дозволені).
3. Хеш-код ключа **не повинен змінюватися**, поки ключ лежить у словнику.
4. `GetHashCode` має бути швидким і добре розподіляти значення.

| Тип ключа | Рівність за замовчуванням | Придатний як ключ? |
|-----------|---------------------------|--------------------|
| `int`, `long`, `char`, `enum` | за значенням | ✔ ідеально |
| `string` | за вмістом (ordinal, з урахуванням регістру) | ✔ |
| `record` / `record struct` | за значеннями всіх властивостей (генерується) | ✔ якщо властивості незмінні |
| кортеж `(int, int)` | за значеннями компонентів | ✔ зручно для координат |
| звичайний `class` | **за посиланням** | ⚠ лише якщо потрібна саме ідентичність об'єкта |
| звичайний `struct` без перевизначення | рефлексія по полях — повільно | ⚠ перевизначте `Equals`/`GetHashCode` або використайте `record struct` |

Що буде, якщо порушити правило 1:

```csharp
var broken = new Dictionary<BadPoint, string>();
broken[new BadPoint(1, 2)] = "A";

// Інший об'єкт з тими самими координатами: Equals каже «рівні», але хеш-коди різні
bool found = broken.ContainsKey(new BadPoint(1, 2));
Console.WriteLine($"BadPoint found: {found}");

var good = new Dictionary<GoodPoint, string>();
good[new GoodPoint(1, 2)] = "A";
Console.WriteLine($"GoodPoint found: {good.ContainsKey(new GoodPoint(1, 2))}");

var tuple = new Dictionary<(int X, int Y), string> { [(1, 2)] = "A" };  // кортеж — теж правильний ключ
Console.WriteLine($"Tuple found: {tuple.ContainsKey((1, 2))}");

// Перевизначено лише Equals — класична помилка (компілятор навіть дає попередження CS0659)
#pragma warning disable CS0659
public sealed class BadPoint(int x, int y)
{
    public int X { get; } = x;
    public int Y { get; } = y;

    public override bool Equals(object? obj) => obj is BadPoint p && p.X == X && p.Y == Y;
    // GetHashCode не перевизначено -> береться хеш посилання, різний для різних об'єктів
}
#pragma warning restore CS0659

// Правильно: record struct генерує Equals, GetHashCode, ==, != за всіма полями
public readonly record struct GoodPoint(int X, int Y);
```

**Приклад запуску:**
```
BadPoint found: False
GoodPoint found: True
Tuple found: True
```

Ручна реалізація для класу (коли `record` не підходить):

```csharp
var a = new Money(100, "UAH");
var b = new Money(100, "UAH");
var prices = new HashSet<Money> { a };

Console.WriteLine($"ReferenceEquals={ReferenceEquals(a, b)} Equals={a.Equals(b)} ==: {a == b}");
Console.WriteLine($"Same hash: {a.GetHashCode() == b.GetHashCode()}, set contains b: {prices.Contains(b)}");

public sealed class Money(decimal amount, string currency) : IEquatable<Money>
{
    public decimal Amount { get; } = amount;       // лише get — ключ незмінний
    public string Currency { get; } = currency;

    // IEquatable<T>.Equals — типізована версія без упаковки; її викликає EqualityComparer<T>.Default
    public bool Equals(Money? other) =>
        other is not null && Amount == other.Amount && Currency == other.Currency;

    public override bool Equals(object? obj) => Equals(obj as Money);

    // HashCode.Combine якісно змішує хеші кількох полів
    public override int GetHashCode() => HashCode.Combine(Amount, Currency);

    public static bool operator ==(Money? left, Money? right) => Equals(left, right);
    public static bool operator !=(Money? left, Money? right) => !Equals(left, right);
}
```

**Приклад запуску:**
```
ReferenceEquals=False Equals=True ==: True
Same hash: True, set contains b: True
```

### 3.4. Поганий хеш: від O(1) до O(n)

```csharp
const int N = 2_000;

var goodComparer = new CountingComparer(badHash: false);
var badComparer = new CountingComparer(badHash: true);
var good = new Dictionary<int, int>(goodComparer);
var bad = new Dictionary<int, int>(badComparer);

for (int i = 0; i < N; i++)
{
    good[i] = i;
    bad[i] = i;
}
goodComparer.EqualsCalls = 0;
badComparer.EqualsCalls = 0;

for (int i = 0; i < N; i++)                          // N пошуків в кожному словнику
{
    _ = good.ContainsKey(i);
    _ = bad.ContainsKey(i);
}

Console.WriteLine($"good hash: {goodComparer.EqualsCalls,9:N0} Equals calls (~{goodComparer.EqualsCalls / N} per lookup)");
Console.WriteLine($"bad hash:  {badComparer.EqualsCalls,9:N0} Equals calls (~{badComparer.EqualsCalls / N} per lookup)");

// Порівнювач, що рахує виклики Equals і за бажанням повертає однаковий хеш для всіх ключів
public sealed class CountingComparer(bool badHash) : IEqualityComparer<int>
{
    public long EqualsCalls { get; set; }

    public bool Equals(int x, int y)
    {
        EqualsCalls++;
        return x == y;
    }

    public int GetHashCode(int obj) => badHash ? 42 : obj;   // 42 для всіх -> один довгий ланцюжок
}
```

**Приклад запуску:**
```
good hash:     2,000 Equals calls (~1 per lookup)
bad hash:  2,001,000 Equals calls (~1000 per lookup)
```

Кожен пошук у «поганому» словнику в середньому проходить половину ланцюжка з 2000 елементів — це вже не хеш-таблиця, а повільний зв'язний список.

### 3.5. Власні порівнювачі: `IEqualityComparer<T>` і `StringComparer`

Рівність ключів можна задати **ззовні**, не змінюючи сам тип, — через конструктор словника.

```csharp
// Регістронезалежний словник: "Kyiv" і "KYIV" — один ключ
var population = new Dictionary<string, int>(StringComparer.OrdinalIgnoreCase)
{
    ["Kyiv"] = 2_950_000,
    ["Lviv"] = 717_000,
};
Console.WriteLine($"KYIV -> {population["KYIV"]:N0}");
Console.WriteLine($"TryAdd lviv: {population.TryAdd("lviv", 1)}");   // false — такий ключ уже є

// Порівнювач за довжиною рядка (демонстраційний): групує слова однакової довжини
var byLength = new Dictionary<string, List<string>>(new LengthComparer());
foreach (string w in new[] { "cat", "dog", "house", "tree", "mouse", "sun" })
{
    if (!byLength.TryGetValue(w, out List<string>? group))
    {
        group = [];
        byLength[w] = group;                                 // ключем стане перше слово такої довжини
    }
    group.Add(w);
}
foreach (var (key, words) in byLength)
{
    Console.WriteLine($"len {key.Length}: {string.Join(", ", words)}");
}

// Порівнювач масивів за вмістом (масиви за замовчуванням порівнюються за посиланням)
var seen = new HashSet<int[]>(new ArrayContentComparer());
Console.WriteLine($"add [1,2]: {seen.Add([1, 2])}, add [1,2] again: {seen.Add([1, 2])}");

public sealed class LengthComparer : IEqualityComparer<string>
{
    public bool Equals(string? x, string? y) => x?.Length == y?.Length;
    public int GetHashCode(string obj) => obj.Length;        // рівні за Equals -> рівні хеші ✔
}

public sealed class ArrayContentComparer : IEqualityComparer<int[]>
{
    public bool Equals(int[]? x, int[]? y) =>
        ReferenceEquals(x, y) || (x is not null && y is not null && x.AsSpan().SequenceEqual(y));

    public int GetHashCode(int[] obj)
    {
        var hash = new HashCode();
        foreach (int item in obj)
        {
            hash.Add(item);                                  // накопичуємо хеш по всіх елементах
        }
        return hash.ToHashCode();
    }
}
```

**Приклад запуску:**
```
KYIV -> 2,950,000
TryAdd lviv: False
len 3: cat, dog, sun
len 5: house, mouse
len 4: tree
add [1,2]: True, add [1,2] again: False
```

| `StringComparer` | Коли використовувати |
|------------------|----------------------|
| `Ordinal` (за замовчуванням) | ідентифікатори, ключі, шляхи в Linux — найшвидший |
| `OrdinalIgnoreCase` | імена файлів у Windows, HTTP-заголовки, команди — швидкий і передбачуваний |
| `CurrentCulture(IgnoreCase)` | лише для відображення користувачу; результат залежить від мови системи |
| `InvariantCulture` | культурно-обізнане, але стабільне порівняння (рідко потрібне для ключів) |

### 3.6. Ідіоматичний API: `TryGetValue`, `TryAdd`, `GetValueOrDefault`

```csharp
var stock = new Dictionary<string, int> { ["apple"] = 5 };

// ❌ Два пошуки: ContainsKey + індексатор
if (stock.ContainsKey("apple"))
{
    Console.WriteLine($"apple (2 lookups): {stock["apple"]}");
}

// ✔ Один пошук
if (stock.TryGetValue("apple", out int apples))
{
    Console.WriteLine($"apple (1 lookup): {apples}");
}

// Індексатор при читанні відсутнього ключа кидає виняток
try
{
    Console.WriteLine(stock["pear"]);
}
catch (KeyNotFoundException)
{
    Console.WriteLine("pear: KeyNotFoundException");
}

Console.WriteLine($"pear or default: {stock.GetValueOrDefault("pear")}, or -1: {stock.GetValueOrDefault("pear", -1)}");

// Add кидає виняток на дублікат, TryAdd — повертає false, індексатор — перезаписує
Console.WriteLine($"TryAdd apple: {stock.TryAdd("apple", 100)} -> {stock["apple"]}");
stock["apple"] = 100;
Console.WriteLine($"indexer set -> {stock["apple"]}");

// Remove з out-параметром: видалити й одразу отримати значення
if (stock.Remove("apple", out int removed))
{
    Console.WriteLine($"removed apple={removed}, count={stock.Count}");
}
```

**Приклад запуску:**
```
apple (2 lookups): 5
apple (1 lookup): 5
pear: KeyNotFoundException
pear or default: 0, or -1: -1
TryAdd apple: False -> 5
indexer set -> 100
removed apple=100, count=0
```

### 3.7. `CollectionsMarshal.GetValueRefOrAddDefault` — один пошук на «прочитати-змінити-записати»

`freq[word] = freq.GetValueOrDefault(word) + 1` робить **два** хешування. У гарячих циклах можна отримати `ref` на значення прямо всередині словника:

```csharp
using System.Runtime.InteropServices;

string text = "the quick brown fox jumps over the lazy dog the end";
var freq = new Dictionary<string, int>();

foreach (string word in text.Split(' '))
{
    // ref на слот значення; якщо ключа не було — він додається зі значенням default (0)
    ref int count = ref CollectionsMarshal.GetValueRefOrAddDefault(freq, word, out bool existed);
    count++;                                   // змінюємо значення прямо в масиві entries — без другого пошуку
}

foreach (var (word, count) in freq.Where(p => p.Value > 1))
{
    Console.WriteLine($"{word}: {count}");
}

// Для структур-значень це ще корисніше: можна змінити поле без копіювання struct
var stats = new Dictionary<string, Stats>();
foreach (int score in new[] { 7, 3, 9 })
{
    ref Stats s = ref CollectionsMarshal.GetValueRefOrAddDefault(stats, "player1", out _);
    s.Total += score;
    s.Games++;
}
Console.WriteLine($"player1: total={stats["player1"].Total} games={stats["player1"].Games}");

public struct Stats
{
    public int Total;
    public int Games;
}
```

**Приклад запуску:**
```
the: 3
player1: total=19 games=3
```

> ⚠ Не додавайте й не видаляйте елементи словника, поки тримаєте такий `ref`: resize перемістить записи, і посилання вказуватиме на старий масив.

### 3.8. Порядок перелічення

```csharp
var d = new Dictionary<int, string>();
d[5] = "five";
d[1] = "one";
d[3] = "three";
Console.WriteLine($"after adds:   {string.Join(", ", d.Keys)}");   // зараз збігається з порядком вставки...

d.Remove(1);
d[42] = "forty-two";                                               // ...але новий запис зайняв «дірку» від 1
Console.WriteLine($"after remove: {string.Join(", ", d.Keys)}");

// Якщо порядок важливий — сортуйте явно або беріть SortedDictionary
Console.WriteLine($"sorted keys:  {string.Join(", ", d.Keys.Order())}");

// .NET 9+: OrderedDictionary<TKey,TValue> гарантує порядок вставки і має доступ за індексом
var ordered = new OrderedDictionary<string, int> { ["b"] = 2, ["a"] = 1 };
ordered.Remove("b");
ordered["c"] = 3;
Console.WriteLine($"ordered:      {string.Join(", ", ordered.Keys)}; index 0 = {ordered.GetAt(0).Key}");
```

**Приклад запуску:**
```
after adds:   5, 1, 3
after remove: 5, 42, 3
sorted keys:  3, 5, 42
ordered:      a, c; index 0 = a
```

**Документація прямо каже: порядок елементів `Dictionary` не визначено.** Те, що без видалень він збігається з порядком вставки, — деталь реалізації.

### 3.9. Баг зі змінюваним ключем

```csharp
var student = new StudentKey { Name = "Ann", Group = "KN-21" };
var grades = new Dictionary<StudentKey, int> { [student] = 95 };

Console.WriteLine($"before: contains={grades.ContainsKey(student)}");

student.Group = "KN-31";                        // змінили поле, яке бере участь у GetHashCode!

Console.WriteLine($"after:  contains={grades.ContainsKey(student)} count={grades.Count}");
Console.WriteLine($"key is still inside: {grades.Keys.Any(k => ReferenceEquals(k, student))}");

// Запис «загубився»: він лежить у кошику, розрахованому за СТАРИМ хешем
grades[student] = 100;                          // додасть ДРУГИЙ запис для того самого об'єкта
Console.WriteLine($"after re-set: count={grades.Count}");

// Змінюваний клас-ключ — так робити НЕ треба. Виправлення: init-властивості або record
public sealed class StudentKey
{
    public required string Name { get; set; }
    public required string Group { get; set; }

    public override bool Equals(object? obj) => obj is StudentKey s && s.Name == Name && s.Group == Group;
    public override int GetHashCode() => HashCode.Combine(Name, Group);
}
```

**Приклад запуску:**
```
before: contains=True
after:  contains=False count=1
key is still inside: True
after re-set: count=2
```

**Типові помилки розділу 3:**

- Перевизначити `Equals` без `GetHashCode` (або навпаки).
- Змінювати поля об'єкта, що вже є ключем словника чи елементом `HashSet`.
- `ContainsKey` + індексатор замість одного `TryGetValue`.
- Покладатися на порядок перелічення `Dictionary`.
- Використовувати `CurrentCulture` для ключів — програма поводиться по-різному на різних комп'ютерах (відома «турецька I»).
- Виводити/зберігати `string.GetHashCode()` — він відрізняється між запусками процесу.
- Шукати значення через `ContainsValue` в циклі — це O(n) на кожен виклик; потрібен зворотний словник.

**Міні-вправа 3.** Реалізуйте двосторонній словник `BiMap` (логін ↔ id), у якому обидва пошуки O(1), а спроба додати вже зайнятий логін **або** id повертає `false`.

<details>
<summary>Розв'язок</summary>

```csharp
var users = new BiMap();
Console.WriteLine(users.TryAdd("ann", 1));
Console.WriteLine(users.TryAdd("bob", 2));
Console.WriteLine(users.TryAdd("ANN", 3));     // логін регістронезалежний -> false
Console.WriteLine(users.TryAdd("cid", 2));     // id 2 зайнятий -> false
Console.WriteLine($"id of Bob = {users.GetId("Bob")}, login of 1 = {users.GetLogin(1)}");

public sealed class BiMap
{
    private readonly Dictionary<string, int> _byLogin = new(StringComparer.OrdinalIgnoreCase);
    private readonly Dictionary<int, string> _byId = [];

    public bool TryAdd(string login, int id)
    {
        if (_byLogin.ContainsKey(login) || _byId.ContainsKey(id))
        {
            return false;                       // перевіряємо обидва напрями ДО зміни стану
        }
        _byLogin.Add(login, id);
        _byId.Add(id, login);
        return true;
    }

    public int GetId(string login) => _byLogin[login];
    public string GetLogin(int id) => _byId[id];
}
```

**Приклад запуску:**
```
True
True
False
False
id of Bob = 2, login of 1 = ann
```

</details>

---

> ☕ **Перерва (5 хв).** Позаду ≈ 55 хв: інтерфейси, послідовності та хеш-таблиці.

---

## 4. HashSet<T> і операції над множинами

`HashSet<T>` — це `Dictionary` без значень: ті самі кошики, записи, `GetHashCode`/`Equals`, ті самі O(1) у середньому. Усе з розділу 3 про контракт рівності, порівнювачі та змінювані ключі стосується й множин.

### 4.1. Основні операції

```csharp
var visited = new HashSet<string>();

Console.WriteLine(visited.Add("home"));      // true — додано
Console.WriteLine(visited.Add("about"));     // true
Console.WriteLine(visited.Add("home"));      // false — вже було (жодного винятку)
Console.WriteLine($"Count={visited.Count} Contains(about)={visited.Contains("about")}");

visited.Remove("about");
Console.WriteLine($"after Remove: Count={visited.Count}");

// TryGetValue повертає ЕКЗЕМПЛЯР, що лежить у множині (корисно з регістронезалежним порівнювачем)
var names = new HashSet<string>(StringComparer.OrdinalIgnoreCase) { "Kyiv" };
if (names.TryGetValue("KYIV", out string? original))
{
    Console.WriteLine($"stored as: {original}");
}

// Дедуплікація зі збереженням порядку першої появи
int[] input = [4, 1, 4, 2, 1, 3, 2];
var seen = new HashSet<int>();
List<int> unique = [.. input.Where(seen.Add)];   // seen.Add повертає true лише для нового елемента
Console.WriteLine($"unique in order: {string.Join(" ", unique)}");
```

**Приклад запуску:**
```
True
True
False
Count=2 Contains(about)=True
after Remove: Count=1
stored as: Kyiv
unique in order: 4 1 2 3
```

### 4.2. Теоретико-множинні операції

```
   A = {1,2,3,4}     B = {3,4,5,6}

   UnionWith            A ∪ B = {1,2,3,4,5,6}
   IntersectWith        A ∩ B = {3,4}
   ExceptWith           A \ B = {1,2}
   SymmetricExceptWith  A △ B = {1,2,5,6}

        ┌───────A───────┐
        │  1  2  ┌──────┼──────B──┐
        │        │ 3  4 │  5  6   │
        └────────┼──────┘         │
                 └────────────────┘
```

Усі `...With`-методи **змінюють множину на місці** (на відміну від LINQ `Union`/`Intersect`, які створюють нову послідовність).

```csharp
// Допоміжна функція: множина -> відсортований рядок (порядок HashSet не гарантований)
static string Show(IEnumerable<int> set) => "{" + string.Join(",", set.Order()) + "}";

HashSet<int> a = [1, 2, 3, 4];
HashSet<int> b = [3, 4, 5, 6];

var union = new HashSet<int>(a);          // копія, щоб не зіпсувати a
union.UnionWith(b);

var intersect = new HashSet<int>(a);
intersect.IntersectWith(b);

var except = new HashSet<int>(a);
except.ExceptWith(b);

var symmetric = new HashSet<int>(a);
symmetric.SymmetricExceptWith(b);

Console.WriteLine($"A ∪ B = {Show(union)}");
Console.WriteLine($"A ∩ B = {Show(intersect)}");
Console.WriteLine($"A \\ B = {Show(except)}");
Console.WriteLine($"A △ B = {Show(symmetric)}");

// Перевірки відношень — нічого не змінюють
HashSet<int> small = [3, 4];
Console.WriteLine($"{Show(small)} ⊆ A: {small.IsSubsetOf(a)}, proper: {small.IsProperSubsetOf(a)}");
Console.WriteLine($"A ⊇ {Show(small)}: {a.IsSupersetOf(small)}");
Console.WriteLine($"A overlaps B: {a.Overlaps(b)}, A equals {{4,3,2,1}}: {a.SetEquals([4, 3, 2, 1])}");

// Аргументом може бути будь-який IEnumerable<T>, навіть із дублікатами
a.ExceptWith(new List<int> { 1, 1, 1 });
Console.WriteLine($"A after ExceptWith([1,1,1]) = {Show(a)}");
```

**Приклад запуску:**
```
A ∪ B = {1,2,3,4,5,6}
A ∩ B = {3,4}
A \ B = {1,2}
A △ B = {1,2,5,6}
{3,4} ⊆ A: True, proper: True
A ⊇ {3,4}: True
A overlaps B: True, A equals {4,3,2,1}: True
A after ExceptWith([1,1,1]) = {2,3,4}
```

| Метод | Складність (m — розмір аргументу) | Результат |
|-------|:----------------------------------:|-----------|
| `Add` / `Remove` / `Contains` | O(1) avg | `bool` |
| `UnionWith` | O(m) | змінює `this` |
| `IntersectWith` | O(n + m) | змінює `this` |
| `ExceptWith` | O(m) | змінює `this` |
| `SymmetricExceptWith` | O(n + m) | змінює `this` |
| `IsSubsetOf` / `IsSupersetOf` / `Overlaps` / `SetEquals` | O(n + m) | `bool` |
| LINQ `Union` / `Intersect` / `Except` | O(n + m) | нова лінива послідовність |

### 4.3. Практичні задачі на множини

```csharp
// 1) Чи є в масиві дублікати? O(n) замість O(n²) подвійного циклу
static bool HasDuplicates(int[] values)
{
    var seen = new HashSet<int>(values.Length);   // місткість заздалегідь — без resize
    foreach (int v in values)
    {
        if (!seen.Add(v))
        {
            return true;                          // Add повернув false -> вже бачили
        }
    }
    return false;
}

Console.WriteLine($"[1,2,3,1] has duplicates: {HasDuplicates([1, 2, 3, 1])}");
Console.WriteLine($"[1,2,3]   has duplicates: {HasDuplicates([1, 2, 3])}");

// 2) Спільні друзі двох користувачів
var friends = new Dictionary<string, HashSet<string>>
{
    ["ann"] = ["bob", "cid", "dan", "eve"],
    ["bob"] = ["ann", "cid", "eve", "fay"],
};
var mutual = new HashSet<string>(friends["ann"]);
mutual.IntersectWith(friends["bob"]);
Console.WriteLine($"mutual friends: {string.Join(", ", mutual.Order())}");

// 3) Найдовша послідовність послідовних чисел за O(n): [100, 4, 200, 1, 3, 2] -> 1,2,3,4 (довжина 4)
static int LongestConsecutive(int[] nums)
{
    var set = new HashSet<int>(nums);
    int best = 0;
    foreach (int x in set)
    {
        if (set.Contains(x - 1))
        {
            continue;                             // x — не початок послідовності, пропускаємо
        }
        int length = 1;
        while (set.Contains(x + length))          // кожен елемент «пройдемо» лише раз за весь алгоритм
        {
            length++;
        }
        best = Math.Max(best, length);
    }
    return best;
}
Console.WriteLine($"longest consecutive: {LongestConsecutive([100, 4, 200, 1, 3, 2])}");

// 4) Відсутні й зайві елементи між двома знімками
string[] yesterday = ["a.txt", "b.txt", "c.txt"];
string[] today = ["b.txt", "c.txt", "d.txt"];
var deleted = new HashSet<string>(yesterday);
deleted.ExceptWith(today);
var created = new HashSet<string>(today);
created.ExceptWith(yesterday);
Console.WriteLine($"deleted: {string.Join(",", deleted)}; created: {string.Join(",", created)}");
```

**Приклад запуску:**
```
[1,2,3,1] has duplicates: True
[1,2,3]   has duplicates: False
mutual friends: cid, eve
longest consecutive: 4
deleted: a.txt; created: d.txt
```

**Типові помилки розділу 4:**

- Виводити `HashSet` без сортування і чекати стабільного порядку.
- Використовувати `List<T>.Contains` у циклі — O(n²) там, де `HashSet` дає O(n).
- Забути, що `UnionWith` змінює множину на місці, і випадково зіпсувати оригінал.
- `HashSet<int[]>` або `HashSet<List<int>>` без порівнювача — порівнюються посилання.
- Плутати LINQ `Distinct()` (нова послідовність) з `HashSet` (структура для повторних перевірок).

**Міні-вправа 4.** Дано два рядки. Виведіть у алфавітному порядку літери, які є рівно в одному з них (без урахування регістру).

<details>
<summary>Розв'язок</summary>

```csharp
static string OnlyInOne(string first, string second)
{
    var set = new HashSet<char>(first.ToLowerInvariant().Where(char.IsLetter));
    set.SymmetricExceptWith(second.ToLowerInvariant().Where(char.IsLetter)); // A △ B
    return new string([.. set.Order()]);
}

Console.WriteLine(OnlyInOne("Hello", "World"));
Console.WriteLine(OnlyInOne("abc", "CBA"));
```

**Приклад запуску:**
```
dehrw

```

(Другий рядок порожній: множини однакові.)

</details>

---

## 5. Відсортовані колекції

Хеш-таблиця швидка, але не вміє відповідати на питання «**що більше за x**», «**найменший елемент**», «**усі ключі в діапазоні [a, b]**». Для цього потрібне **збалансоване бінарне дерево пошуку** або відсортований масив.

| Колекція | Будова | Пошук | Вставка | Видалення | Min/Max | Діапазон | Індекс `[i]` |
|----------|--------|:-----:|:-------:|:---------:|:-------:|:--------:|:------------:|
| `SortedSet<T>` | червоно-чорне дерево | O(log n) | O(log n) | O(log n) | O(log n) | O(log n + k) | — |
| `SortedDictionary<K,V>` | червоно-чорне дерево (над `SortedSet` пар) | O(log n) | O(log n) | O(log n) | через `First()` | — (лише через LINQ) | — |
| `SortedList<K,V>` | два відсортовані масиви `keys[]`, `values[]` | O(log n) | O(n) | O(n) | O(1) | через бінарний пошук | O(1) |
| `List<T>` + `Sort` + `BinarySearch` | відсортований масив | O(log n) | O(n) | O(n) | O(1) | O(log n + k) | O(1) |

### 5.1. Червоно-чорне дерево — що треба знати користувачу

```
Вставляємо 10, 20, 30, 15, 25, 5, 1 у SortedSet<int>:

                 20(B)
               /       \
           10(R)       30(B)
          /    \       /
       5(B)   15(B)  25(R)
       /
     1(R)

Правила:  1) корінь чорний;  2) у червоного вузла немає червоних дітей;
          3) на кожному шляху від кореня до листа однакова кількість чорних вузлів.
Наслідок: висота ≤ 2·log₂(n+1)  →  усі операції O(log n), навіть для відсортованого вводу.
```

Звичайне BST на відсортованому вводі 1, 2, 3, ... вироджується у «палицю» висоти n. Червоно-чорне дерево після кожної вставки робить повороти й перефарбування, тож глибина завжди логарифмічна. Детально дерева розбиратимемо в окремій лекції; тут важливі наслідки для складності.

### 5.2. `SortedSet<T>`: порядок, Min/Max, `GetViewBetween`

```csharp
var set = new SortedSet<int> { 50, 10, 40, 20, 30, 20 };   // дублікат 20 відкидається

Console.WriteLine($"set: {string.Join(" ", set)}");        // обхід in-order — завжди за зростанням
Console.WriteLine($"Min={set.Min} Max={set.Max} Count={set.Count}");
Console.WriteLine($"reverse: {string.Join(" ", set.Reverse())}");   // SortedSet.Reverse() — лінивий обхід у зворотному порядку

// GetViewBetween(lo, hi) — «вікно» [lo, hi] на те саме дерево, без копіювання
SortedSet<int> view = set.GetViewBetween(15, 40);
Console.WriteLine($"view [15..40]: {string.Join(" ", view)}, Min={view.Min}, Max={view.Max}");

set.Add(35);                                               // зміна оригіналу видна у вікні
Console.WriteLine($"after Add(35) view: {string.Join(" ", view)}");

view.Add(25);                                              // додавання через вікно змінює оригінал
Console.WriteLine($"after view.Add(25) set: {string.Join(" ", set)}");

try
{
    view.Add(100);                                         // поза межами вікна — виняток
}
catch (ArgumentOutOfRangeException)
{
    Console.WriteLine("view.Add(100): out of range");
}

// RemoveWhere — видалити за умовою
int removed = set.RemoveWhere(x => x % 20 == 0);
Console.WriteLine($"removed {removed}, set: {string.Join(" ", set)}");
```

**Приклад запуску:**
```
set: 10 20 30 40 50
Min=10 Max=50 Count=5
reverse: 50 40 30 20 10
view [15..40]: 20 30 40, Min=20, Max=40
after Add(35) view: 20 30 35 40
after view.Add(25) set: 10 20 25 30 35 40 50
view.Add(100): out of range
removed 2, set: 10 25 30 35 50
```

### 5.3. Floor / Ceiling — «найближчий» елемент

У Java є `TreeSet.floor/ceiling`, у C++ — `lower_bound`. У .NET їх можна виразити через `GetViewBetween`:

```csharp
var prices = new SortedSet<int> { 100, 250, 400, 999 };

Console.WriteLine($"ceiling(260) = {Ceiling(prices, 260)}");   // найменший >= 260
Console.WriteLine($"floor(260)   = {Floor(prices, 260)}");     // найбільший <= 260
Console.WriteLine($"ceiling(1000)= {Ceiling(prices, 1000)?.ToString() ?? "none"}");
Console.WriteLine($"floor(50)    = {Floor(prices, 50)?.ToString() ?? "none"}");
Console.WriteLine($"exact floor(400) = {Floor(prices, 400)}");

// Найменший елемент >= value або null
static int? Ceiling(SortedSet<int> set, int value)
{
    if (set.Count == 0 || value > set.Max)
    {
        return null;                                   // GetViewBetween кине виняток, якщо lo > hi
    }
    SortedSet<int> tail = set.GetViewBetween(value, set.Max);
    return tail.Count > 0 ? tail.Min : null;           // Min вікна — O(log n)
}

// Найбільший елемент <= value або null
static int? Floor(SortedSet<int> set, int value)
{
    if (set.Count == 0 || value < set.Min)
    {
        return null;
    }
    SortedSet<int> head = set.GetViewBetween(set.Min, value);
    return head.Count > 0 ? head.Max : null;
}
```

**Приклад запуску:**
```
ceiling(260) = 400
floor(260)   = 250
ceiling(1000)= none
floor(50)    = none
exact floor(400) = 400
```

> ⚠ `view.Count` для вікна рахує елементи обходом — O(k). Для перевірки «вікно не порожнє» у гарячому коді краще порівнювати `Min`/`Max` з межами.

### 5.4. Власний `IComparer<T>`

Порядок у відсортованих колекціях задається через `IComparer<T>`: `Compare(x, y)` < 0, якщо x раніше, 0 — якщо **рівні**, > 0 — якщо пізніше.

```csharp
// 1) Comparer.Create з лямбди: за спаданням
var desc = new SortedSet<int>(Comparer<int>.Create((a, b) => b.CompareTo(a))) { 3, 1, 2 };
Console.WriteLine($"desc: {string.Join(" ", desc)}");

// 2) Слова: спочатку за довжиною, потім за алфавітом
var words = new SortedSet<string>(new LengthThenAlphabet()) { "pear", "fig", "apple", "kiwi", "banana" };
Console.WriteLine($"by length: {string.Join(" ", words)}");

// 3) ⚠ ПАСТКА: компаратор лише за довжиною вважає "kiwi" і "pear" РІВНИМИ -> один з них зникне
var trap = new SortedSet<string>(Comparer<string>.Create((a, b) => a.Length.CompareTo(b.Length)))
{
    "pear", "fig", "kiwi", "plum",
};
Console.WriteLine($"trap: {string.Join(" ", trap)} (Count={trap.Count})");

// 4) Кортежі порівнюються покомпонентно — зручний спосіб зробити порядок «повним»
var tasks = new SortedSet<(int Priority, string Name)>
{
    (2, "write tests"), (1, "fix bug"), (2, "code review"), (3, "refactor"),
};
foreach (var (priority, name) in tasks)
{
    Console.WriteLine($"  P{priority}: {name}");
}

public sealed class LengthThenAlphabet : IComparer<string>
{
    public int Compare(string? x, string? y)
    {
        int byLength = (x?.Length ?? 0).CompareTo(y?.Length ?? 0);
        return byLength != 0 ? byLength : string.CompareOrdinal(x, y);  // «тай-брейк» — щоб різні рядки не були рівними
    }
}
```

**Приклад запуску:**
```
desc: 3 2 1
by length: fig kiwi pear apple banana
trap: fig pear (Count=2)
  P1: fix bug
  P2: code review
  P2: write tests
  P3: refactor
```

**Правило:** для `SortedSet`/`SortedDictionary` компаратор повертає 0 **лише** для елементів, які ви вважаєте однаковими. Завжди додавайте «тай-брейк» за унікальним полем.

### 5.5. `SortedDictionary` проти `SortedList`

```
SortedDictionary<int,string>             SortedList<int,string>
(дерево вузлів)                          (два паралельні масиви)

        20:"b"                           index:   0     1     2     3
       /      \                          keys:  [ 10 | 20  | 30  | 40 ]
   10:"a"    30:"c"                      values:["a" | "b" | "c" | "d"]
                \
               40:"d"                    Вставка 25: бінарний пошук O(log n),
                                         зсув хвостів обох масивів O(n)
Вставка 25: спуск O(log n) + поворот
```

| Критерій | `SortedDictionary<K,V>` | `SortedList<K,V>` |
|----------|:-----------------------:|:-----------------:|
| Пошук за ключем | O(log n) | O(log n) |
| Вставка у випадковому порядку | **O(log n)** | O(n) |
| Вставка у зростаючому порядку | O(log n) | **O(log n)** (додавання в кінець) |
| Видалення | **O(log n)** | O(n) |
| Доступ за індексом (`Keys[i]`, `GetKeyAtIndex`) | — | **O(1)** |
| Пам'ять | об'єкт-вузол на кожен елемент (≈ 40+ байт накладних) | **лише масиви** |
| Кеш процесора | погано | **добре** |
| Коли обирати | багато вставок/видалень | будується один раз або дані надходять відсортованими; мало пам'яті |

```csharp
var tree = new SortedDictionary<string, double>(StringComparer.Ordinal)
{
    ["USD"] = 41.5, ["EUR"] = 45.1, ["PLN"] = 10.4, ["GBP"] = 52.7,
};
Console.WriteLine($"SortedDictionary: {string.Join(", ", tree.Select(p => $"{p.Key}={p.Value}"))}");
Console.WriteLine($"first: {tree.First().Key}, last: {tree.Last().Key}");   // First/Last — через LINQ, Last() тут O(n)!

var list = new SortedList<string, double>(tree, StringComparer.Ordinal);    // копія в масиви
Console.WriteLine($"SortedList index of PLN = {list.IndexOfKey("PLN")}");
Console.WriteLine($"key at 1 = {list.GetKeyAtIndex(1)}, value at 1 = {list.GetValueAtIndex(1)}");
Console.WriteLine($"last via index: {list.Keys[^1]}");                       // O(1) — Keys є IList<string>

// k-й за порядком елемент і «сусіди» — сильна сторона SortedList
int i = list.IndexOfKey("GBP");
Console.WriteLine($"neighbours of GBP: {list.Keys[i - 1]} and {list.Keys[i + 1]}");
```

**Приклад запуску:**
```
SortedDictionary: EUR=45.1, GBP=52.7, PLN=10.4, USD=41.5
first: EUR, last: USD
SortedList index of PLN = 2
key at 1 = GBP, value at 1 = 52.7
last via index: USD
neighbours of GBP: EUR and PLN
```

> Числа на кшталт `45.1` виводяться з крапкою, бо консольні приклади запускались з інваріантною/англійською культурою. В українській локалі буде кома.

### 5.6. Запити на діапазон: журнал подій

Задача: події приходять із часовими мітками; треба швидко відповідати «що сталося між `from` і `to`» і «яка остання подія до моменту t».

```csharp
var log = new EventLog();
log.Add(new TimeOnly(9, 0), "server started");
log.Add(new TimeOnly(9, 15), "user ann logged in");
log.Add(new TimeOnly(10, 5), "backup finished");
log.Add(new TimeOnly(10, 40), "user bob logged in");
log.Add(new TimeOnly(12, 0), "deploy v2");

Console.WriteLine("Between 09:10 and 10:40:");
foreach (var (time, message) in log.Between(new TimeOnly(9, 10), new TimeOnly(10, 40)))
{
    Console.WriteLine($"  {time:HH\\:mm} {message}");
}
Console.WriteLine($"Last before 11:00: {log.LastBefore(new TimeOnly(11, 0))}");
Console.WriteLine($"Last before 08:00: {log.LastBefore(new TimeOnly(8, 0)) ?? "nothing"}");

public sealed class EventLog
{
    // Ключ — (час, номер), щоб дві події в одну хвилину не вважались однаковими
    private readonly SortedSet<(TimeOnly Time, int Seq, string Message)> _events = [];
    private int _seq;

    public void Add(TimeOnly time, string message) => _events.Add((time, _seq++, message));

    public IEnumerable<(TimeOnly Time, string Message)> Between(TimeOnly from, TimeOnly to)
    {
        // Межі вікна: найменший і найбільший можливі кортежі для цих моментів
        var lo = (from, int.MinValue, "");
        var hi = (to, int.MaxValue, "");
        return _events.GetViewBetween(lo, hi).Select(e => (e.Time, e.Message));  // O(log n + k)
    }

    public string? LastBefore(TimeOnly time)
    {
        if (_events.Count == 0 || _events.Min.Time >= time)
        {
            return null;
        }
        var hi = (time.Add(TimeSpan.FromTicks(-1)), int.MaxValue, "");
        return _events.GetViewBetween(_events.Min, hi).Max.Message;             // O(log n)
    }
}
```

**Приклад запуску:**
```
Between 09:10 and 10:40:
  09:15 user ann logged in
  10:05 backup finished
  10:40 user bob logged in
Last before 11:00: user bob logged in
Last before 08:00: nothing
```

**Типові помилки розділу 5:**

- Компаратор повертає 0 для різних об'єктів — «зникаючі» елементи.
- `SortedList` для потоку випадкових вставок — O(n²) сумарно.
- `sortedDictionary.Last()` чи `ElementAt(i)` — це LINQ, O(n), а не O(log n).
- `GetViewBetween(lo, hi)` з `lo > hi` кидає `ArgumentException` — перевіряйте межі.
- Змінювати поля елемента, від яких залежить порядок, поки він у дереві (аналог бага зі змінюваним ключем).

**Міні-вправа 5.** Система бронювання: інтервали `[start, end)` не можуть перетинатися. Реалізуйте `bool TryBook(int start, int end)` за O(log n) на `SortedList<int,int>` (start → end) або `SortedDictionary`.

<details>
<summary>Розв'язок</summary>

```csharp
var calendar = new Calendar();
Console.WriteLine(calendar.TryBook(10, 20));   // True
Console.WriteLine(calendar.TryBook(15, 25));   // False — перетин з [10,20)
Console.WriteLine(calendar.TryBook(20, 30));   // True — дотик кінця не є перетином
Console.WriteLine(calendar.TryBook(5, 10));    // True
Console.WriteLine(calendar.TryBook(8, 12));    // False

public sealed class Calendar
{
    private readonly SortedSet<(int Start, int End)> _booked = [];

    public bool TryBook(int start, int end)
    {
        // Попередник: останній інтервал зі Start < end. Якщо його End > start — перетин.
        if (_booked.Count > 0 && _booked.Min.Start < end)
        {
            var previous = _booked.GetViewBetween(_booked.Min, (end - 1, int.MaxValue)).Max;
            if (previous.End > start)
            {
                return false;
            }
        }
        _booked.Add((start, end));
        return true;
    }
}
```

**Приклад запуску:**
```
True
False
True
True
False
```

Інтервали не перетинаються між собою, тому достатньо перевірити лише **один** найближчий інтервал, що починається раніше за `end`.

</details>

---

> ☕ **Перерва (5 хв).** Позаду ≈ 95 хв: хеш-множини й дерева.

---

## 6. PriorityQueue та інші спеціалізовані колекції

### 6.1. `PriorityQueue<TElement,TPriority>` — бінарна min-купа

З'явилась у .NET 6. Завжди видає елемент з **найменшим пріоритетом**. Всередині — масив, що зберігає повне бінарне дерево (насправді 4-арну купу, але ідея та сама):

```
Пріоритети після Enqueue 5, 3, 8, 1, 4 (бінарна купа для наочності):

              1                  масив: [1, 3, 8, 5, 4]
            /   \                індекси: батько i → діти 2i+1, 2i+2
           3     8
          / \
         5   4

Dequeue: забрати корінь (1), останній елемент (4) поставити в корінь і «просіяти вниз»:

              3                  масив: [3, 4, 8, 5]
            /   \
           4     8
          /
         5
```

| Операція | Складність |
|----------|:----------:|
| `Enqueue(element, priority)` | O(log n) |
| `Dequeue()` / `TryDequeue` | O(log n) |
| `Peek()` / `TryPeek` | O(1) |
| `EnqueueDequeue` / `DequeueEnqueue` | O(log n) — одна операція замість двох |
| конструктор з `IEnumerable` / `EnqueueRange` | O(n) — heapify |
| пошук / зміна пріоритету довільного елемента | O(n) (`Remove` з .NET 9) |
| `UnorderedItems` | обхід у **довільному** порядку |

```csharp
var er = new PriorityQueue<string, int>();          // елемент — пацієнт, пріоритет — терміновість (1 — найвища)
er.Enqueue("broken arm", 2);
er.Enqueue("cold", 5);
er.Enqueue("heart attack", 1);
er.Enqueue("cut finger", 4);

Console.WriteLine($"Count={er.Count}, next: {er.Peek()}");

while (er.TryDequeue(out string? patient, out int urgency))
{
    Console.WriteLine($"  treating [{urgency}] {patient}");
}

// Max-купа: інвертуємо порівняння пріоритетів
var maxHeap = new PriorityQueue<string, int>(Comparer<int>.Create((a, b) => b.CompareTo(a)));
maxHeap.EnqueueRange([("low", 1), ("high", 10), ("mid", 5)]);  // heapify за O(n)
Console.WriteLine($"max-heap order: {maxHeap.Dequeue()}, {maxHeap.Dequeue()}, {maxHeap.Dequeue()}");

// ⚠ Порядок елементів з ОДНАКОВИМ пріоритетом не гарантований (купа не стабільна).
// Рішення: складений пріоритет (priority, sequenceNumber)
var fair = new PriorityQueue<string, (int Priority, long Seq)>();
long seq = 0;
foreach (string job in new[] { "A", "B", "C", "D" })
{
    fair.Enqueue(job, (1, seq++));                  // однаковий пріоритет -> FIFO за seq
}
fair.Enqueue("urgent", (0, seq++));
Console.Write("fair order:");
while (fair.Count > 0)
{
    Console.Write($" {fair.Dequeue()}");
}
Console.WriteLine();
```

**Приклад запуску:**
```
Count=4, next: heart attack
  treating [1] heart attack
  treating [2] broken arm
  treating [4] cut finger
  treating [5] cold
max-heap order: high, mid, low
fair order: urgent A B C D
```

### 6.2. Top-k за O(n log k)

Знайти k найбільших серед n чисел можна сортуванням за O(n log n). Але якщо n — мільйони, а k = 10, краще тримати **min-купу розміру k**: її корінь — найменший з поточних «чемпіонів».

```csharp
static List<int> TopK(IEnumerable<int> source, int k)
{
    var heap = new PriorityQueue<int, int>(k + 1);
    foreach (int x in source)
    {
        if (heap.Count < k)
        {
            heap.Enqueue(x, x);                      // купа ще не заповнена
        }
        else if (x > heap.Peek())
        {
            heap.DequeueEnqueue(x, x);               // витісняємо найслабшого з k — O(log k)
        }
    }

    var result = new List<int>(heap.Count);
    while (heap.Count > 0)
    {
        result.Add(heap.Dequeue());                  // виходять за зростанням
    }
    result.Reverse();                                // найбільший першим
    return result;
}

int[] scores = [42, 7, 99, 13, 58, 71, 3, 88, 64, 25];
Console.WriteLine($"top-3: {string.Join(", ", TopK(scores, 3))}");

var random = new Random(2025);                       // фіксований seed — детермінований результат
int[] big = [.. Enumerable.Range(0, 1_000_000).Select(_ => random.Next())];
List<int> top5 = TopK(big, 5);
bool sameAsSort = top5.SequenceEqual(big.OrderDescending().Take(5));
Console.WriteLine($"1M numbers, top-5 equals sort-based result: {sameAsSort}");

// .NET 9+: вбудований PriorityQueue-підхід доступний і як Enumerable.OrderDescending().Take(k),
// який LINQ оптимізує частковим сортуванням — але пам'ять O(n), а купа — O(k)
```

**Приклад запуску:**
```
top-3: 99, 88, 71
1M numbers, top-5 equals sort-based result: True
```

### 6.3. Злиття k відсортованих списків

```csharp
static List<int> MergeSorted(IReadOnlyList<IReadOnlyList<int>> lists)
{
    // Елемент купи — (номер списку, позиція в ньому); пріоритет — саме значення
    var heap = new PriorityQueue<(int List, int Index), int>();
    for (int i = 0; i < lists.Count; i++)
    {
        if (lists[i].Count > 0)
        {
            heap.Enqueue((i, 0), lists[i][0]);        // голови всіх списків
        }
    }

    var result = new List<int>();
    while (heap.TryDequeue(out var cursor, out int value))
    {
        result.Add(value);                            // найменша з голів
        int next = cursor.Index + 1;
        if (next < lists[cursor.List].Count)
        {
            heap.Enqueue((cursor.List, next), lists[cursor.List][next]);  // підставляємо наступний з того ж списку
        }
    }
    return result;                                    // O(N log k), N — сумарна довжина
}

int[][] data =
[
    [1, 4, 7, 10],
    [2, 5, 8],
    [0, 3, 6, 9, 12],
    [],
];
Console.WriteLine(string.Join(" ", MergeSorted(data)));
```

**Приклад запуску:**
```
0 1 2 3 4 5 6 7 8 9 10 12
```

Та сама схема лежить в основі **зовнішнього сортування** (файли, що не вміщуються в пам'ять) і злиття логів з кількох серверів.

### 6.4. Найкоротший шлях: Dijkstra на `PriorityQueue`

```csharp
// Граф міст: список суміжності (сусід, відстань)
var graph = new Dictionary<string, List<(string To, int Km)>>
{
    ["Lviv"] = [("Ternopil", 127), ("Rivne", 211)],
    ["Ternopil"] = [("Khmelnytskyi", 112), ("Rivne", 158)],
    ["Rivne"] = [("Zhytomyr", 187)],
    ["Khmelnytskyi"] = [("Vinnytsia", 120), ("Zhytomyr", 180)],
    ["Zhytomyr"] = [("Kyiv", 140)],
    ["Vinnytsia"] = [("Kyiv", 268)],
    ["Kyiv"] = [],
};

var dist = new Dictionary<string, int> { ["Lviv"] = 0 };
var previous = new Dictionary<string, string>();
var queue = new PriorityQueue<string, int>();
queue.Enqueue("Lviv", 0);

while (queue.TryDequeue(out string? city, out int d))
{
    if (d > dist[city])
    {
        continue;                                          // застарілий запис: уже знайшли коротший шлях
    }
    foreach (var (to, km) in graph[city])
    {
        int candidate = d + km;
        if (candidate < dist.GetValueOrDefault(to, int.MaxValue))
        {
            dist[to] = candidate;
            previous[to] = city;
            queue.Enqueue(to, candidate);                  // «ліниве» оновлення замість decrease-key
        }
    }
}

var path = new Stack<string>();                             // відновлюємо шлях з кінця
for (string? c = "Kyiv"; c is not null; c = previous.GetValueOrDefault(c))
{
    path.Push(c);
}
Console.WriteLine($"Lviv -> Kyiv: {dist["Kyiv"]} km via {string.Join(" -> ", path)}");
```

**Приклад запуску:**
```
Lviv -> Kyiv: 538 km via Lviv -> Rivne -> Zhytomyr -> Kyiv
```

### 6.5. `Stack<T>` і `Queue<T>` — коротке нагадування

Реалізації стека й черги ми писали в лекції 2; бібліотечні версії мають той самий API і O(1) операції. Два класичні застосування:

```csharp
// Стек: перевірка дужок
static bool IsBalanced(string s)
{
    var pairs = new Dictionary<char, char> { [')'] = '(', [']'] = '[', ['}'] = '{' };
    var stack = new Stack<char>();
    foreach (char c in s)
    {
        if (c is '(' or '[' or '{')
        {
            stack.Push(c);
        }
        else if (pairs.TryGetValue(c, out char open))
        {
            if (!stack.TryPop(out char top) || top != open)   // TryPop — без винятку на порожньому стеку
            {
                return false;
            }
        }
    }
    return stack.Count == 0;
}

Console.WriteLine($"{{[()]}}: {IsBalanced("{[()]}")}, ([)]: {IsBalanced("([)]")}, ((: {IsBalanced("((")}");

// Черга: BFS — мінімальна кількість ходів коня з a1 до h8
static int KnightMoves((int X, int Y) from, (int X, int Y) to)
{
    var queue = new Queue<((int X, int Y) Cell, int Steps)>();
    var visited = new HashSet<(int, int)> { from };
    queue.Enqueue((from, 0));
    (int, int)[] jumps = [(1, 2), (2, 1), (2, -1), (1, -2), (-1, -2), (-2, -1), (-2, 1), (-1, 2)];

    while (queue.TryDequeue(out var item))
    {
        if (item.Cell == to)
        {
            return item.Steps;
        }
        foreach (var (dx, dy) in jumps)
        {
            var next = (X: item.Cell.X + dx, Y: item.Cell.Y + dy);
            if (next.X is >= 0 and < 8 && next.Y is >= 0 and < 8 && visited.Add(next))
            {
                queue.Enqueue((next, item.Steps + 1));
            }
        }
    }
    return -1;
}

Console.WriteLine($"knight a1 -> h8: {KnightMoves((0, 0), (7, 7))} moves");
```

**Приклад запуску:**
```
{[()]}: True, ([)]: False, ((: False
knight a1 -> h8: 6 moves
```

### 6.6. `BitArray` — мільйон прапорців у 125 КБ

`bool[]` витрачає 1 байт на прапорець, `BitArray` — 1 біт (усередині `int[]`).

```csharp
using System.Collections;

// Решето Ератосфена до 1 000 000
const int Limit = 1_000_000;
var composite = new BitArray(Limit + 1);              // усі false; true = «складене»
for (int i = 2; (long)i * i <= Limit; i++)
{
    if (!composite[i])
    {
        for (int j = i * i; j <= Limit; j += i)
        {
            composite[j] = true;
        }
    }
}

int primeCount = 0;
for (int i = 2; i <= Limit; i++)
{
    if (!composite[i])
    {
        primeCount++;
    }
}
Console.WriteLine($"primes <= {Limit:N0}: {primeCount:N0}");
Console.WriteLine($"memory: BitArray ≈ {(Limit + 1) / 8 / 1024} KB vs bool[] ≈ {(Limit + 1) / 1024} KB");

// Побітові операції над цілими масивами прапорців
var a = new BitArray(new[] { true, true, false, false });
var b = new BitArray(new[] { true, false, true, false });
static string Bits(BitArray bits) => string.Concat(bits.Cast<bool>().Select(x => x ? '1' : '0'));
Console.WriteLine($"a AND b = {Bits(new BitArray(a).And(b))}");   // And змінює об'єкт — працюємо з копією
Console.WriteLine($"a OR  b = {Bits(new BitArray(a).Or(b))}");
Console.WriteLine($"a XOR b = {Bits(new BitArray(a).Xor(b))}");
Console.WriteLine($"NOT a   = {Bits(new BitArray(a).Not())}");
```

**Приклад запуску:**
```
primes <= 1,000,000: 78,498
memory: BitArray ≈ 122 KB vs bool[] ≈ 976 KB
a AND b = 1000
a OR  b = 1110
a XOR b = 0110
NOT a   = 0011
```

### 6.7. Коли `LinkedList<T>` справді потрібен

У реальному коді `LinkedList<T>` рідкісний, але є випадки, де він незамінний:

| Сценарій | Чому саме зв'язний список |
|----------|---------------------------|
| **LRU-кеш** (розділ 10.4) | `Dictionary` зберігає `LinkedListNode`, переміщення в голову — O(1) |
| Черга з видаленням із середини за посиланням | `Remove(node)` — O(1) без пошуку |
| Редактор тексту / історія з курсором | вставка перед/після поточної позиції O(1) |
| Злиття/розрізання списків без копіювання | переприв'язування вузлів |
| Двобічна черга (deque) | у .NET немає `Deque<T>`, `LinkedList` дає O(1) з обох кінців |

```csharp
// Плейлист з курсором: вставка «грати наступною» і видалення поточної — O(1)
var playlist = new LinkedList<string>(["intro", "song A", "song B", "outro"]);
LinkedListNode<string> current = playlist.Find("song A")!;   // Find — O(n), але робимо один раз

playlist.AddAfter(current, "play next!");                     // O(1)
LinkedListNode<string> toSkip = current;
current = current.Next!;
playlist.Remove(toSkip);                                      // O(1): вузол відомий

Console.WriteLine($"now playing: {current.Value}");
Console.WriteLine($"playlist: {string.Join(" | ", playlist)}");
Console.WriteLine($"prev: {current.Previous?.Value}, next: {current.Next?.Value}");
```

**Приклад запуску:**
```
now playing: play next!
playlist: intro | play next! | song B | outro
prev: intro, next: song B
```

**Типові помилки розділу 6:**

- Чекати, що `PriorityQueue` з однаковими пріоритетами поверне елементи в порядку додавання.
- Перелічувати `pq.UnorderedItems` і вважати, що вони відсортовані.
- Для max-купи ставити пріоритет `-x` — працює для `int`, але ламається на `int.MinValue` (переповнення); компаратор надійніший.
- Шукати в `PriorityQueue` конкретний елемент, щоб змінити пріоритет, — O(n); використовуйте «ліниве» видалення (як у Dijkstra).
- `Stack<T>` з `foreach` обходиться від вершини до дна, а `new Stack<T>(collection)` «перевертає» порядок — легко заплутатися.

**Міні-вправа 6.** Дано потік чисел. Після кожного числа виведіть поточну **медіану**. Використайте дві купи: max-купу для меншої половини і min-купу для більшої.

<details>
<summary>Розв'язок</summary>

```csharp
var median = new RunningMedian();
foreach (int x in new[] { 5, 15, 1, 3, 8, 7 })
{
    median.Add(x);
    Console.WriteLine($"add {x,2} -> median {median.Median}");
}

public sealed class RunningMedian
{
    private readonly PriorityQueue<int, int> _low = new(Comparer<int>.Create((a, b) => b.CompareTo(a))); // max-купа
    private readonly PriorityQueue<int, int> _high = new();                                               // min-купа

    public void Add(int x)
    {
        if (_low.Count == 0 || x <= _low.Peek())
        {
            _low.Enqueue(x, x);
        }
        else
        {
            _high.Enqueue(x, x);
        }

        // Балансуємо: розміри відрізняються не більше ніж на 1, і _low не менша
        if (_low.Count > _high.Count + 1)
        {
            int moved = _low.Dequeue();
            _high.Enqueue(moved, moved);
        }
        else if (_high.Count > _low.Count)
        {
            int moved = _high.Dequeue();
            _low.Enqueue(moved, moved);
        }
    }

    public double Median => _low.Count > _high.Count
        ? _low.Peek()
        : (_low.Peek() + _high.Peek()) / 2.0;
}
```

**Приклад запуску:**
```
add  5 -> median 5
add 15 -> median 10
add  1 -> median 5
add  3 -> median 4
add  8 -> median 5
add  7 -> median 6
```

</details>

---

## 7. LINQ над колекціями

LINQ (Language Integrated Query) — набір методів-розширень для `IEnumerable<T>` у `System.Linq`. Майже всі вони реалізовані як ітератори з `yield` (розділ 1.4), тому успадковують **відкладене виконання**.

### 7.1. Основні оператори

```csharp
var students = new List<Student>
{
    new("Ann", "KN-21", 92),
    new("Bob", "KN-22", 67),
    new("Cid", "KN-21", 78),
    new("Dan", "KN-22", 85),
    new("Eve", "KN-21", 92),
    new("Fay", "KN-23", 55),
};

// Where + Select: фільтр і проєкція
IEnumerable<string> passed = students.Where(s => s.Score >= 60).Select(s => s.Name);
Console.WriteLine($"passed: {string.Join(", ", passed)}");

// OrderBy + ThenBy: стабільне сортування за кількома ключами
var ranking = students.OrderByDescending(s => s.Score).ThenBy(s => s.Name);
Console.WriteLine($"ranking: {string.Join(", ", ranking.Select(s => $"{s.Name}({s.Score})"))}");

// GroupBy: групи з ключем
foreach (IGrouping<string, Student> group in students.GroupBy(s => s.Group).OrderBy(g => g.Key))
{
    Console.WriteLine($"  {group.Key}: count={group.Count()}, avg={group.Average(s => s.Score):F1}, best={group.MaxBy(s => s.Score)!.Name}");
}

// ToDictionary: ключі мають бути унікальними, інакше ArgumentException
Dictionary<string, int> scoreByName = students.ToDictionary(s => s.Name, s => s.Score);
Console.WriteLine($"Dan -> {scoreByName["Dan"]}");

// ToLookup: «словник списків», відсутній ключ дає порожню послідовність (без винятку)
ILookup<string, string> namesByGroup = students.ToLookup(s => s.Group, s => s.Name);
Console.WriteLine($"KN-21: {string.Join(",", namesByGroup["KN-21"])}; KN-99: [{string.Join(",", namesByGroup["KN-99"])}]");

// Distinct / DistinctBy
Console.WriteLine($"distinct scores: {string.Join(",", students.Select(s => s.Score).Distinct())}");
Console.WriteLine($"first per group: {string.Join(",", students.DistinctBy(s => s.Group).Select(s => s.Name))}");

// Chunk: розбити на пакети фіксованого розміру (.NET 6)
foreach (Student[] batch in students.Chunk(4))
{
    Console.WriteLine($"  batch: {string.Join(",", batch.Select(s => s.Name))}");
}

// Zip: попарне поєднання двох послідовностей (довжина — за коротшою)
string[] seats = ["A1", "A2", "A3"];
Console.WriteLine($"seats: {string.Join(", ", students.Zip(seats, (s, seat) => $"{s.Name}@{seat}"))}");

// Aggregate / CountBy (.NET 9) / Sum / Any / All
Console.WriteLine($"names joined: {students.Select(s => s.Name).Aggregate((acc, n) => acc + "+" + n)}");
Console.WriteLine($"count by group: {string.Join(", ", students.CountBy(s => s.Group).Select(p => $"{p.Key}={p.Value}"))}");
Console.WriteLine($"any < 60: {students.Any(s => s.Score < 60)}, all > 50: {students.All(s => s.Score > 50)}");

public sealed record Student(string Name, string Group, int Score);
```

**Приклад запуску:**
```
passed: Ann, Bob, Cid, Dan, Eve
ranking: Ann(92), Eve(92), Dan(85), Cid(78), Bob(67), Fay(55)
  KN-21: count=3, avg=87.3, best=Ann
  KN-22: count=2, avg=76.0, best=Dan
  KN-23: count=1, avg=55.0, best=Fay
Dan -> 85
KN-21: Ann,Cid,Eve; KN-99: []
distinct scores: 92,67,78,85,55
first per group: Ann,Bob,Fay
  batch: Ann,Bob,Cid,Dan
  batch: Eve,Fay
seats: Ann@A1, Bob@A2, Cid@A3
names joined: Ann+Bob+Cid+Dan+Eve+Fay
count by group: KN-21=3, KN-22=2, KN-23=1
any < 60: True, all > 50: True
```

### 7.2. Відкладене (deferred) та негайне (immediate) виконання

| Тип | Оператори | Коли виконується |
|-----|-----------|------------------|
| **Відкладені, потокові** | `Where`, `Select`, `SelectMany`, `Take`, `Skip`, `Zip`, `Chunk`, `Concat`, `Distinct`* | по одному елементу під час `foreach` |
| **Відкладені, буферизовані** | `OrderBy`, `GroupBy`, `Reverse`, `Join`, `Except`, `Intersect` | при першому `MoveNext` читають **усе** джерело |
| **Негайні (materialize)** | `ToList`, `ToArray`, `ToDictionary`, `ToHashSet`, `ToLookup` | одразу, створюють нову колекцію |
| **Негайні (скаляр)** | `Count`, `Sum`, `Max`, `First`, `Any`, `All`, `Contains`, `Aggregate` | одразу, повертають одне значення |

\* `Distinct` потоковий, але накопичує `HashSet` уже бачених елементів.

```csharp
var numbers = new List<int> { 1, 2, 3 };

IEnumerable<int> query = numbers.Where(n =>
{
    Console.WriteLine($"  checking {n}");
    return n % 2 == 1;
});
Console.WriteLine("query built (nothing checked yet)");

numbers.Add(5);                                   // запит «побачить» цю зміну — він ще не виконувався
numbers.Add(7);

List<int> snapshot = query.ToList();              // негайне виконання
Console.WriteLine($"snapshot: {string.Join(",", snapshot)}");

numbers.Add(9);
Console.WriteLine($"snapshot after Add(9): {string.Join(",", snapshot)}");  // копія не змінюється
Console.WriteLine($"query count now: {query.Count()}");                      // запит виконується ЗНОВУ

// Пастка із захопленою змінною
int threshold = 2;
IEnumerable<int> greater = numbers.Where(n => n > threshold);
threshold = 6;                                    // лямбда читає змінну в момент виконання, а не створення
Console.WriteLine($"greater than threshold: {string.Join(",", greater)}");
```

**Приклад запуску:**
```
query built (nothing checked yet)
  checking 1
  checking 2
  checking 3
  checking 5
  checking 7
snapshot: 1,3,5,7
snapshot after Add(9): 1,3,5,7
  checking 1
  checking 2
  checking 3
  checking 5
  checking 7
  checking 9
query count now: 5
greater than threshold: 7,9
```

### 7.3. Пастка багаторазового перелічення

```csharp
int generated = 0;

IEnumerable<int> LoadScores()                    // імітація «дорогого» джерела: БД, файл, мережа
{
    foreach (int s in new[] { 70, 85, 90 })
    {
        generated++;
        yield return s;
    }
}

IEnumerable<int> scores = LoadScores();

// ❌ Три перелічення = три «запити до бази»
if (scores.Any())
{
    Console.WriteLine($"count={scores.Count()}, avg={scores.Average():F1}");
}
Console.WriteLine($"elements generated (bad): {generated}");

// ✔ Матеріалізуємо один раз
generated = 0;
IReadOnlyList<int> cached = LoadScores().ToList();
if (cached.Count > 0)
{
    Console.WriteLine($"count={cached.Count}, avg={cached.Average():F1}");
}
Console.WriteLine($"elements generated (good): {generated}");

// Випадкові дані перелічуються по-різному щоразу — ще гірше, ніж просто повільно
var rnd = new Random(1);
IEnumerable<int> dice = Enumerable.Range(0, 3).Select(_ => rnd.Next(1, 7));
Console.WriteLine($"same sequence twice equal? {dice.SequenceEqual(dice)}");
```

**Приклад запуску:**
```
count=3, avg=81.7
elements generated (bad): 7
count=3, avg=81.7
elements generated (good): 3
same sequence twice equal? False
```

Рахуємо «погану» версію: `Any()` зупинився після першого елемента (1 генерація), `Count()` пройшов усю послідовність (3), `Average()` — ще раз усю (3). Разом 1 + 3 + 3 = **7** замість 3. Для справжньої бази даних це три окремі запити, а дані між ними можуть змінитися.

### 7.4. Складність операторів LINQ

| Оператор | Час | Додаткова пам'ять | Примітка |
|----------|:---:|:-----------------:|----------|
| `Where`, `Select`, `Take`, `Skip` | O(n) на повний обхід | O(1) | потокові |
| `Count()` | O(1) для `ICollection<T>`, інакше O(n) | O(1) | LINQ перевіряє тип джерела |
| `Any()` | O(1) | O(1) | кращий за `Count() > 0` |
| `ElementAt(i)`, `Last()` | O(1) для `IList<T>`, інакше O(n) | O(1) | |
| `Contains(x)` | O(1) для `HashSet`, O(n) для списку | O(1) | делегує `ICollection<T>.Contains` |
| `OrderBy` + обхід | O(n log n) | O(n) | стабільне сортування |
| `OrderBy(...).First()` | O(n) | O(n) | оптимізовано до пошуку мінімуму |
| `Min`, `Max`, `MinBy`, `MaxBy` | O(n) | O(1) | |
| `Distinct`, `ToHashSet` | O(n) avg | O(n) | хеш-множина |
| `GroupBy`, `ToLookup`, `ToDictionary` | O(n) avg | O(n) | хеш-таблиця |
| `Join` (a, b) | O(n + m) avg | O(m) | lookup по внутрішній послідовності |
| `Except`, `Intersect`, `Union` | O(n + m) avg | O(m) | |
| `a.Where(x => b.Contains(x))` при `b` — `List` | **O(n·m)** | O(1) | класичний прихований квадрат! |
| `Reverse()` | O(n) | O(n) | буферизує |
| `SequenceEqual` | O(n) | O(1) | |

```csharp
var ids = Enumerable.Range(0, 20_000).ToList();
var banned = Enumerable.Range(0, 20_000).Where(i => i % 3 == 0).ToList();

long comparisons = 0;
bool InList(int x)                          // рахуємо «кроки» лінійного пошуку в List
{
    int idx = banned.IndexOf(x);
    comparisons += idx >= 0 ? idx + 1 : banned.Count;
    return idx >= 0;
}

int slow = ids.Count(x => !InList(x));      // O(n·m)
Console.WriteLine($"List.Contains:    allowed={slow}, comparisons≈{comparisons:N0}");

var bannedSet = banned.ToHashSet();         // O(m) один раз
int fast = ids.Count(x => !bannedSet.Contains(x));       // O(n)
Console.WriteLine($"HashSet.Contains: allowed={fast}, lookups={ids.Count:N0}");

int viaExcept = ids.Except(banned).Count(); // Except усередині теж будує HashSet
Console.WriteLine($"Except:           allowed={viaExcept}");
```

**Приклад запуску:**
```
List.Contains:    allowed=13333, comparisons≈111,118,889
HashSet.Contains: allowed=13333, lookups=20,000
Except:           allowed=13333
```

### 7.5. Query-синтаксис і `SelectMany`

```csharp
var orders = new[]
{
    new Order(1, "Ann", ["pen", "book"]),
    new Order(2, "Bob", ["laptop"]),
    new Order(3, "Ann", ["book", "lamp"]),
};

// SelectMany «розплющує» послідовність послідовностей
var allItems = orders.SelectMany(o => o.Items).Distinct().Order();
Console.WriteLine($"items: {string.Join(", ", allItems)}");

// Той самий запит у query-синтаксисі (компілятор перетворює його на виклики методів)
var customerItems =
    from order in orders
    from item in order.Items
    group item by order.Customer into g
    orderby g.Key
    select $"{g.Key}: {string.Join("/", g.Distinct())}";
Console.WriteLine(string.Join("; ", customerItems));

// Join двох колекцій за ключем
var prices = new Dictionary<string, decimal> { ["pen"] = 20m, ["book"] = 250m, ["laptop"] = 30000m, ["lamp"] = 800m };
var totals = orders
    .Select(o => (o.Id, Total: o.Items.Sum(i => prices[i])))
    .OrderByDescending(t => t.Total);
foreach (var (id, total) in totals)
{
    Console.WriteLine($"  order #{id}: {total} UAH");
}

public sealed record Order(int Id, string Customer, string[] Items);
```

**Приклад запуску:**
```
items: book, lamp, laptop, pen
Ann: pen/book/lamp; Bob: laptop
  order #2: 30000 UAH
  order #3: 1050 UAH
  order #1: 270 UAH
```

**Типові помилки розділу 7:**

- Повертати з методу лінивий запит над локальним ресурсом (відкритим файлом, `using`) — на момент перелічення ресурс уже закритий.
- Багаторазово перелічувати `IEnumerable<T>` (7.3).
- `list.Where(...).Count() > 0` замість `list.Any(...)`; `First()` на порожній послідовності замість `FirstOrDefault()`.
- `ToDictionary` на даних з дублікатами ключів → `ArgumentException`; використовуйте `ToLookup`/`GroupBy`.
- Приховане O(n²): `Contains`/`IndexOf` у списку всередині `Where`/`Select`.
- `OrderBy(x).OrderBy(y)` замість `OrderBy(x).ThenBy(y)` — друге сортування «перебиває» перше.
- Побічні ефекти в лямбдах `Select` — вони виконаються стільки разів, скільки разів перелічено запит.

**Міні-вправа 7.** Дано список покупок `(Customer, Product, Price)`. Одним LINQ-запитом отримайте для кожного покупця: суму покупок і найдорожчий товар, відсортувавши за сумою спаданням.

<details>
<summary>Розв'язок</summary>

```csharp
(string Customer, string Product, int Price)[] purchases =
[
    ("ann", "phone", 12000), ("bob", "book", 300), ("ann", "case", 500),
    ("cid", "tv", 20000), ("bob", "lamp", 900),
];

var report = purchases
    .GroupBy(p => p.Customer)
    .Select(g => new
    {
        Customer = g.Key,
        Total = g.Sum(p => p.Price),
        Top = g.MaxBy(p => p.Price).Product,
    })
    .OrderByDescending(r => r.Total);

foreach (var r in report)
{
    Console.WriteLine($"{r.Customer}: total={r.Total}, top={r.Top}");
}
```

**Приклад запуску:**
```
cid: total=20000, top=tv
ann: total=12500, top=phone
bob: total=1200, top=lamp
```

</details>

---

> ☕ **Перерва (5 хв).** Позаду ≈ 130 хв: купи, LINQ. Залишилось: конкурентні, незмінні та frozen колекції, продуктивність і практика.

---

## 8. Потокобезпечні, незмінні та frozen колекції

Три різні відповіді на питання «як безпечно ділити дані між частинами програми»:

| Підхід | Ідея | Типові типи |
|--------|------|-------------|
| **Конкурентні** | колекція змінюється, але внутрішньо синхронізована | `ConcurrentDictionary`, `ConcurrentQueue`, `Channel<T>` |
| **Незмінні (immutable)** | колекцію неможливо змінити; «зміна» створює нову версію, що ділить більшість даних зі старою | `ImmutableList`, `ImmutableDictionary`, `ImmutableArray` |
| **Заморожені (frozen)** | будується один раз, змін немає взагалі, зате читання максимально оптимізоване | `FrozenDictionary`, `FrozenSet` |

### 8.1. Чому `Dictionary` не потокобезпечний

`Dictionary.Add` — це кілька кроків: знайти кошик, записати запис у `entries[_count]`, оновити `buckets[b]`, збільшити `_count`, а при заповненні — **resize** (новий масив і перерозподіл). Якщо два потоки роблять це одночасно:

```
Потік A                                   Потік B
───────                                   ───────
index = _count          (= 5)
                                          index = _count          (= 5)   ← той самий слот!
entries[5] = ("x", 1)
                                          entries[5] = ("y", 2)           ← запис A втрачено
_count++                (= 6)
                                          _count++                (= 7)   ← Count бреше: 7, а записів 6
                                          resize: копіює entries...
buckets[3] = 6  ← пише в СТАРИЙ масив                                     ← ланцюжок може замкнутися в цикл
```

Можливі наслідки: втрачені елементи, неправильний `Count`, `IndexOutOfRangeException`, `InvalidOperationException` («Operations that change non-concurrent collections must have exclusive-access»), а в старих версіях .NET — **нескінченний цикл** у `TryGetValue` через зациклений ланцюжок. Важливо: помилка **недетермінована** — тести можуть проходити роками.

```csharp
// ⚠ Навмисно некоректний код: запис у звичайний Dictionary з кількох потоків
var unsafeCounts = new Dictionary<int, int>();
string outcome;
try
{
    Parallel.For(0, 100_000, i =>
    {
        int key = i % 100;
        unsafeCounts[key] = unsafeCounts.GetValueOrDefault(key) + 1;  // читання + запис без синхронізації
    });
    outcome = $"no exception, total = {unsafeCounts.Values.Sum()} (expected 100000)";
}
catch (Exception ex)
{
    outcome = $"exception: {ex.GetType().Name}";
}
Console.WriteLine($"Dictionary:      {outcome}");

// ✔ Та сама задача з lock — коректно, але всі потоки стоять у черзі за одним замком
var lockedCounts = new Dictionary<int, int>();
var gate = new Lock();                                     // System.Threading.Lock (.NET 9); у старих версіях — object
Parallel.For(0, 100_000, i =>
{
    lock (gate)
    {
        int key = i % 100;
        lockedCounts[key] = lockedCounts.GetValueOrDefault(key) + 1;
    }
});
Console.WriteLine($"Dictionary+lock: total = {lockedCounts.Values.Sum()}");
```

**Орієнтовний вивід** (перший рядок щоразу різний — у цьому й проблема):
```
Dictionary:      no exception, total = 91300 (expected 100000)
Dictionary+lock: total = 100000
```

> Лише **читання** з кількох потоків звичайного `Dictionary`, який ніхто не змінює, — безпечне. Небезпечне будь-яке поєднання запису з читанням чи іншим записом.

### 8.2. `ConcurrentDictionary<TKey,TValue>`: API

Усі методи атомарні — «перевірка + дія» відбувається як одна операція:

| Метод | Що робить | Замість небезпечного |
|-------|-----------|----------------------|
| `TryAdd(k, v)` | додати, якщо ключа немає; `bool` | `if (!ContainsKey) Add` |
| `TryGetValue(k, out v)` | прочитати без блокування | — |
| `TryUpdate(k, new, expected)` | замінити, лише якщо поточне значення == `expected` (compare-and-swap) | `if (d[k] == old) d[k] = new` |
| `TryRemove(k, out v)` / `TryRemove(pair)` | видалити (або видалити, лише якщо значення збігається) | `if (TryGetValue) Remove` |
| `GetOrAdd(k, factory)` | повернути існуюче або створити | `if (!TryGetValue) d[k] = Create()` |
| `AddOrUpdate(k, addValue, updateFactory)` | додати або оновити на основі старого значення | `d[k] = d[k] + 1` |
| `this[k] = v` | безумовний запис (атомарний сам по собі) | — |

```csharp
using System.Collections.Concurrent;

var sessions = new ConcurrentDictionary<string, int>();

Console.WriteLine($"TryAdd ann:    {sessions.TryAdd("ann", 1)}");
Console.WriteLine($"TryAdd ann:    {sessions.TryAdd("ann", 99)} (value stays {sessions["ann"]})");

int bob = sessions.GetOrAdd("bob", key => key.Length * 10);           // фабрика отримує ключ
Console.WriteLine($"GetOrAdd bob:  {bob}");

// Перевантаження з аргументом фабрики — без замикання (менше алокацій)
int cid = sessions.GetOrAdd("cid", static (key, bonus) => key.Length + bonus, factoryArgument: 1000);
Console.WriteLine($"GetOrAdd cid:  {cid}");

int newAnn = sessions.AddOrUpdate("ann", addValue: 1, updateValueFactory: (_, old) => old + 1);
Console.WriteLine($"AddOrUpdate:   ann = {newAnn}");

// Compare-and-swap: успіх лише якщо значення не змінилося з моменту читання
Console.WriteLine($"TryUpdate 2->5: {sessions.TryUpdate("ann", 5, comparisonValue: 2)}");
Console.WriteLine($"TryUpdate 2->7: {sessions.TryUpdate("ann", 7, comparisonValue: 2)} (ann = {sessions["ann"]})");

// Видалити, лише якщо значення все ще те, яке ми бачили
bool removedWrong = sessions.TryRemove(KeyValuePair.Create("bob", 0));
bool removedRight = sessions.TryRemove("bob", out int bobValue);
Console.WriteLine($"TryRemove bob=0: {removedWrong}; TryRemove bob: {removedRight} ({bobValue})");

Console.WriteLine($"final: {string.Join(", ", sessions.OrderBy(p => p.Key).Select(p => $"{p.Key}={p.Value}"))}");
```

**Приклад запуску:**
```
TryAdd ann:    True
TryAdd ann:    False (value stays 1)
GetOrAdd bob:  30
GetOrAdd cid:  1003
AddOrUpdate:   ann = 2
TryUpdate 2->5: True
TryUpdate 2->7: False (ann = 5)
TryRemove bob=0: False; TryRemove bob: True (30)
final: ann=5, cid=1003
```

### 8.3. Пастка: фабрика може виконатися двічі

`GetOrAdd` і `AddOrUpdate` викликають делегат **поза внутрішнім замком** (щоб чужий код не тримав блокування). Тому два потоки можуть одночасно не знайти ключ, обидва виконати фабрику, а в словник потрапить лише одне значення. Для дешевих обчислень це нормально; для дорогих (запит у БД, відкриття з'єднання) або з побічними ефектами — ні.

```csharp
using System.Collections.Concurrent;

// Barrier змушує обидва потоки опинитися всередині фабрики одночасно — гонка гарантовано відтворюється
int factoryCalls = 0;
var barrier = new Barrier(2);
var cache = new ConcurrentDictionary<string, string>();

string Load(string key)
{
    Interlocked.Increment(ref factoryCalls);
    barrier.SignalAndWait();                           // чекаємо, поки другий потік теж зайде сюди
    return $"data-for-{key}";
}

string[] results = await Task.WhenAll(
    Task.Run(() => cache.GetOrAdd("config", Load)),
    Task.Run(() => cache.GetOrAdd("config", Load)));

Console.WriteLine($"plain GetOrAdd: factory calls = {factoryCalls}, same value = {results[0] == results[1]}, count = {cache.Count}");

// ✔ Рішення: зберігати Lazy<T>. Створити «обгортку» двічі не страшно — вона дешева,
// а Lazy з режимом ExecutionAndPublication гарантує ОДИН запуск дорогої функції
int lazyCalls = 0;
var lazyBarrier = new Barrier(2);
var lazyCache = new ConcurrentDictionary<string, Lazy<string>>();

Lazy<string> MakeLazy(string key)
{
    lazyBarrier.SignalAndWait();                       // обидва потоки створюють свій Lazy...
    return new Lazy<string>(() =>
    {
        Interlocked.Increment(ref lazyCalls);          // ...але виконується лише той, що потрапив у словник
        return $"data-for-{key}";
    }, LazyThreadSafetyMode.ExecutionAndPublication);
}

string[] lazyResults = await Task.WhenAll(
    Task.Run(() => lazyCache.GetOrAdd("config", MakeLazy).Value),
    Task.Run(() => lazyCache.GetOrAdd("config", MakeLazy).Value));

Console.WriteLine($"Lazy<T> GetOrAdd: expensive calls = {lazyCalls}, same value = {lazyResults[0] == lazyResults[1]}");
```

**Приклад запуску:**
```
plain GetOrAdd: factory calls = 2, same value = True, count = 1
Lazy<T> GetOrAdd: expensive calls = 1, same value = True
```

Те саме стосується `updateValueFactory` в `AddOrUpdate`: при конфлікті вона **повторюється** (оптимістичний цикл compare-and-swap). Тож фабрика має бути **чистою функцією** — без `Console.WriteLine`, запису в інші колекції чи інкременту лічильників.

### 8.4. Атомарні лічильники: `AddOrUpdate` проти `Interlocked`

```csharp
using System.Collections.Concurrent;

const int Operations = 200_000;
string[] pages = ["/home", "/about", "/blog", "/contact"];

// ❌ Не атомарно: між читанням і записом інший потік встигає змінити значення
var racy = new ConcurrentDictionary<string, int>();
Parallel.For(0, Operations, i =>
{
    string page = pages[i % pages.Length];
    racy[page] = racy.GetValueOrDefault(page) + 1;
});

// ✔ Варіант 1: AddOrUpdate — простий, але при конфліктах фабрика повторюється, int — копіюється
var viaAddOrUpdate = new ConcurrentDictionary<string, int>();
Parallel.For(0, Operations, i =>
{
    viaAddOrUpdate.AddOrUpdate(pages[i % pages.Length], 1, static (_, old) => old + 1);
});

// ✔ Варіант 2: значення — змінний «лічильник-коробка», інкремент через Interlocked (без повторів)
var viaInterlocked = new ConcurrentDictionary<string, Counter>();
Parallel.For(0, Operations, i =>
{
    Counter counter = viaInterlocked.GetOrAdd(pages[i % pages.Length], static _ => new Counter());
    counter.Increment();                                  // атомарний інкремент одного поля
});

int racyTotal = racy.Values.Sum();
Console.WriteLine($"racy total:        {(racyTotal == Operations ? "lucky" : "wrong")} (expected {Operations})");
Console.WriteLine($"AddOrUpdate total: {viaAddOrUpdate.Values.Sum()}");
Console.WriteLine($"Interlocked total: {viaInterlocked.Values.Sum(c => c.Value)}");
Console.WriteLine($"per page:          {string.Join(", ", viaInterlocked.OrderBy(p => p.Key).Select(p => $"{p.Key}={p.Value.Value}"))}");

public sealed class Counter
{
    private long _value;

    public long Value => Interlocked.Read(ref _value);

    public void Increment() => Interlocked.Increment(ref _value);
}
```

**Орієнтовний вивід** (перший рядок може бути `lucky` або `wrong`, решта — завжди однакові):
```
racy total:        wrong (expected 200000)
AddOrUpdate total: 200000
Interlocked total: 200000
per page:          /about=50000, /blog=50000, /contact=50000, /home=50000
```

| Спосіб | Коректність | Алокації | Поведінка під навантаженням |
|--------|:-----------:|:--------:|-----------------------------|
| `d[k] = d[k] + 1` | ❌ | — | втрачає оновлення |
| `AddOrUpdate(k, 1, (_, v) => v + 1)` | ✔ | делегат (static — без алокацій) | повтори при конфліктах на одному ключі |
| `GetOrAdd(k, new Counter())` + `Interlocked.Increment` | ✔ | один об'єкт на ключ | без повторів, найшвидше для «гарячих» ключів |

### 8.5. Перелічення — знімок «на льоту»

Перелічувач `ConcurrentDictionary` **не кидає** `InvalidOperationException` при змінах: він не блокує словник і може побачити (або не побачити) зміни, зроблені під час обходу. Це не атомарний знімок.

```csharp
using System.Collections.Concurrent;

var dict = new ConcurrentDictionary<int, string>();
for (int i = 0; i < 5; i++)
{
    dict[i] = $"v{i}";
}

int seen = 0;
foreach (var (key, _) in dict)
{
    seen++;
    if (key < 100)
    {
        dict.TryAdd(100 + key, "added during foreach");  // звичайний Dictionary тут кинув би виняток
    }
}
// Скільки нових елементів побачив обхід — не визначено (від 5 до 10), тому друкуємо лише діапазон
string seenRange = seen is >= 5 and <= 10 ? "between 5 and 10" : "unexpected";
Console.WriteLine($"no exception; items seen during foreach: {seenRange}; count now = {dict.Count}");

// Потрібен узгоджений знімок? ToArray() бере всі внутрішні замки на мить копіювання
KeyValuePair<int, string>[] snapshot = dict.ToArray();
dict.Clear();
Console.WriteLine($"snapshot length = {snapshot.Length}, dictionary count = {dict.Count}");

// Count, IsEmpty, Keys, Values, ToArray беруть УСІ замки — дорого викликати в гарячому циклі
Console.WriteLine($"IsEmpty = {dict.IsEmpty}");
```

**Приклад запуску:**
```
no exception; items seen during foreach: between 5 and 10; count now = 10
snapshot length = 10, dictionary count = 0
IsEmpty = True
```

| Операція | Замки | Вартість |
|----------|-------|----------|
| `TryGetValue`, `ContainsKey`, індексатор get | жодних | дуже дешево |
| `TryAdd`, `TryUpdate`, `TryRemove`, set | один замок сегмента | дешево |
| `foreach` | жодних | O(n), не знімок |
| `Count`, `IsEmpty`*, `Keys`, `Values`, `ToArray`, `Clear` | **усі** замки | дорого, блокує записи |

\* `IsEmpty` у нових версіях спершу пробує перевірку без замків.

### 8.6. Всередині: розділені замки (lock striping)

```
ConcurrentDictionary (concurrencyLevel = 4 замки, 8 кошиків)

  locks:   [L0]      [L1]      [L2]      [L3]
            │  ╲      │  ╲      │  ╲      │  ╲
  buckets: [b0][b4]  [b1][b5]  [b2][b6]  [b3][b7]      замок = bucket % locks.Length
            │                   │
            ▼                   ▼
          node→node           node                     ланцюжки з незмінних вузлів (key, value, next)

  Потік A пише в b1 (бере L1) ║ Потік B пише в b6 (бере L2)  → працюють паралельно
  Потік C читає b1 без замків: вузли не змінюються «на місці» — лише атомарна заміна посилань
```

- Кількість замків за замовчуванням ≈ кількість процесорних ядер; при рості таблиці число замків може збільшуватися.
- Записи у **різні** сегменти йдуть паралельно; читання взагалі не блокуються.
- Кожен елемент — окремий об'єкт-вузол (на відміну від масиву структур у `Dictionary`), тому пам'яті більше, а однопотокові операції повільніші приблизно в 1.5–3 рази.
- Resize бере всі замки — ще одна причина задавати `capacity` у конструкторі.

**Продуктивність, коротко:** в однопотоковому коді `Dictionary` швидший і економніший; при інтенсивних читаннях з багатьох потоків `ConcurrentDictionary` масштабується майже лінійно; при переважанні записів у кілька «гарячих» ключів — вузьким місцем стають замки сегментів. Якщо дані змінюються рідко, а читаються постійно, часто вигідніше «незмінна колекція + атомарна заміна посилання» (8.10).

### 8.7. Конкурентна множина: трьома способами

У .NET **немає** `ConcurrentHashSet<T>`. Типові заміни:

```csharp
using System.Collections.Concurrent;
using System.Collections.Immutable;

int[] input = [.. Enumerable.Range(0, 100_000).Select(i => i % 1_000)];   // 1000 унікальних значень, кожне 100 разів

// 1) ConcurrentDictionary<T, byte>: ключі — множина, значення ігноруємо
var cdSet = new ConcurrentDictionary<int, byte>();
int firstSeenCd = 0;
Parallel.ForEach(input, x =>
{
    if (cdSet.TryAdd(x, 0))                       // true рівно один раз для кожного значення
    {
        Interlocked.Increment(ref firstSeenCd);
    }
});

// 2) lock навколо звичайного HashSet — просто і швидко при малій конкуренції
var hashSet = new HashSet<int>();
var gate = new Lock();
int firstSeenLock = 0;
Parallel.ForEach(input, x =>
{
    lock (gate)
    {
        if (hashSet.Add(x))
        {
            firstSeenLock++;                      // всередині lock — Interlocked не потрібен
        }
    }
});

// 3) ImmutableHashSet + атомарна заміна посилання (copy-on-write)
ImmutableHashSet<int> immutableSet = [];
Parallel.ForEach(input, x =>
{
    ImmutableInterlocked.Update(ref immutableSet, static (set, item) => set.Add(item), x);
});

Console.WriteLine($"ConcurrentDictionary<int,byte>: count={cdSet.Count}, first-seen={firstSeenCd}");
Console.WriteLine($"lock + HashSet:                 count={hashSet.Count}, first-seen={firstSeenLock}");
Console.WriteLine($"ImmutableHashSet + Update:      count={immutableSet.Count}");
Console.WriteLine($"same elements: {cdSet.Keys.ToHashSet().SetEquals(hashSet) && hashSet.SetEquals(immutableSet)}");
```

**Приклад запуску:**
```
ConcurrentDictionary<int,byte>: count=1000, first-seen=1000
lock + HashSet:                 count=1000, first-seen=1000
ImmutableHashSet + Update:      count=1000
same elements: True
```

| Варіант | Запис | Читання | Перелічення | Коли обрати |
|---------|-------|---------|-------------|-------------|
| `ConcurrentDictionary<T, byte>` | O(1), паралельно в різні сегменти | O(1) без замків | не знімок | багато потоків, багато записів |
| `lock` + `HashSet<T>` | O(1), але послідовно | під тим самим замком | лише під замком (або копія) | мало потоків, складні складені операції (`UnionWith` тощо) |
| `ImmutableHashSet<T>` + `ImmutableInterlocked` | O(log n) + алокації, повтори при конфліктах | O(log n) без замків | **справжній знімок** | записи рідкі, читань і обходів багато |

### 8.8. `ConcurrentQueue`, `ConcurrentStack`, `ConcurrentBag`, `BlockingCollection`, `Channel`

```csharp
using System.Collections.Concurrent;

// ConcurrentQueue: lock-free FIFO. Лише Try-методи — між перевіркою і дією інший потік може забрати елемент
var queue = new ConcurrentQueue<int>();
Parallel.For(0, 1_000, i => queue.Enqueue(i));
long queueSum = 0;
Parallel.For(0, 4, _ =>
{
    while (queue.TryDequeue(out int item))
    {
        Interlocked.Add(ref queueSum, item);
    }
});
Console.WriteLine($"ConcurrentQueue: sum={queueSum}, empty={queue.IsEmpty}");

// ConcurrentStack: lock-free LIFO, пакетні PushRange/TryPopRange
var stack = new ConcurrentStack<string>();
stack.PushRange(["a", "b", "c", "d"]);
string[] popped = new string[3];
int count = stack.TryPopRange(popped);
Console.WriteLine($"ConcurrentStack: popped {count}: {string.Join(",", popped)}; left={stack.Count}");

// ConcurrentBag: неупорядкований «мішок»; кожен потік має локальний список — ідеально, коли той самий потік і кладе, і забирає
var bag = new ConcurrentBag<int>();
Parallel.For(0, 100, i => bag.Add(i * i));
Console.WriteLine($"ConcurrentBag: count={bag.Count}, sum={bag.Sum()}, max={bag.Max()}");
```

**Приклад запуску:**
```
ConcurrentQueue: sum=499500, empty=True
ConcurrentStack: popped 3: d,c,b; left=1
ConcurrentBag: count=100, sum=328350, max=9801
```

**Producer/consumer.** Для передачі даних між потоками-виробниками і споживачами використовують обмежену (bounded) чергу з «зворотним тиском» (back-pressure): коли черга повна, виробник чекає.

```csharp
using System.Collections.Concurrent;
using System.Threading.Channels;

// 1) BlockingCollection: синхронний API, Take блокує потік
using var queue = new BlockingCollection<int>(boundedCapacity: 10);

Task producer = Task.Run(() =>
{
    for (int i = 1; i <= 100; i++)
    {
        queue.Add(i);                       // заблокується, якщо в черзі вже 10 елементів
    }
    queue.CompleteAdding();                 // сигнал «більше нічого не буде»
});

long sum = 0;
Task consumer = Task.Run(() =>
{
    foreach (int item in queue.GetConsumingEnumerable())   // завершиться після CompleteAdding і спорожнення
    {
        sum += item;
    }
});
await Task.WhenAll(producer, consumer);
Console.WriteLine($"BlockingCollection sum = {sum}");

// 2) Channel: асинхронний API — потоки не блокуються, а звільняються через await
Channel<string> channel = Channel.CreateBounded<string>(new BoundedChannelOptions(capacity: 5)
{
    FullMode = BoundedChannelFullMode.Wait,               // що робити, коли канал повний
});

Task writer = Task.Run(async () =>
{
    foreach (string file in new[] { "a.txt", "b.txt", "c.txt" })
    {
        await channel.Writer.WriteAsync(file);
    }
    channel.Writer.Complete();
});

var processed = new ConcurrentBag<string>();
Task[] readers = [.. Enumerable.Range(0, 2).Select(_ => Task.Run(async () =>
{
    await foreach (string file in channel.Reader.ReadAllAsync())   // кілька читачів конкурують за елементи
    {
        processed.Add(file.ToUpperInvariant());
    }
}))];

await writer;
await Task.WhenAll(readers);
Console.WriteLine($"Channel processed: {string.Join(", ", processed.Order())}");
```

**Приклад запуску:**
```
BlockingCollection sum = 5050
Channel processed: A.TXT, B.TXT, C.TXT
```

| Колекція | Порядок | Синхронізація | Коли |
|----------|---------|---------------|------|
| `ConcurrentQueue<T>` | FIFO | lock-free | черга задач без очікування |
| `ConcurrentStack<T>` | LIFO | lock-free | пули об'єктів, «останній — першим» |
| `ConcurrentBag<T>` | не визначено | локальні списки потоків | той самий потік додає й забирає |
| `BlockingCollection<T>` | обгортка (FIFO за замовчуванням) | блокує потоки | синхронний producer/consumer |
| `Channel<T>` | FIFO | асинхронне очікування | `async` код, рекомендований у нових проєктах |

> `async`/`await`, `Task` і `Parallel` ми згадували в лекції 1 (сучасні можливості C#). Тут важливо лише: `BlockingCollection` займає потік на час очікування, `Channel` — ні.

---

### 8.9. Незмінні колекції: `System.Collections.Immutable`

Незмінна колекція після створення **ніколи** не змінюється. Методи `Add`, `Remove`, `SetItem` повертають **новий екземпляр**, а старий залишається валідним.

| Тип | Внутрішня будова | Аналог |
|-----|------------------|--------|
| `ImmutableArray<T>` | `struct` над звичайним масивом `T[]` | `T[]` |
| `ImmutableList<T>` | AVL-дерево | `List<T>` |
| `ImmutableDictionary<K,V>` | AVL-дерево кошиків за хешем | `Dictionary<K,V>` |
| `ImmutableHashSet<T>` | AVL-дерево кошиків за хешем | `HashSet<T>` |
| `ImmutableSortedDictionary<K,V>` | AVL-дерево за ключем | `SortedDictionary<K,V>` |
| `ImmutableSortedSet<T>` | AVL-дерево | `SortedSet<T>` |
| `ImmutableQueue<T>` | два незмінні стеки | `Queue<T>` |
| `ImmutableStack<T>` | незмінний однозв'язний список | `Stack<T>` |

```csharp
using System.Collections.Immutable;

// Add повертає НОВИЙ екземпляр — результат обов'язково треба присвоїти
ImmutableList<string> empty = ImmutableList<string>.Empty;
empty.Add("lost");                                          // ⚠ результат відкинуто — нічого не змінилося
ImmutableList<string> fruits = empty.Add("apple").Add("pear").Insert(0, "kiwi");
Console.WriteLine($"empty.Count={empty.Count}, fruits=[{string.Join(",", fruits)}]");

ImmutableDictionary<string, int> prices = ImmutableDictionary<string, int>.Empty
    .Add("apple", 30)
    .Add("pear", 45);
ImmutableDictionary<string, int> discounted = prices.SetItem("pear", 40).Remove("apple");
Console.WriteLine($"prices: pear={prices["pear"]}, count={prices.Count}; discounted: pear={discounted["pear"]}, count={discounted.Count}");

ImmutableHashSet<int> set = [3, 1, 2];                      // вирази колекцій працюють з immutable-типами
ImmutableHashSet<int> bigger = set.Union([4, 5]);
Console.WriteLine($"set={set.Count} items, bigger={string.Join(",", bigger.Order())}");

ImmutableSortedSet<int> sorted = ImmutableSortedSet.Create(50, 10, 30);
Console.WriteLine($"sorted=[{string.Join(",", sorted.Add(20))}], min={sorted.Min}, index of 30={sorted.IndexOf(30)}");

ImmutableSortedDictionary<string, int> ranks = ImmutableSortedDictionary.CreateRange(
    new Dictionary<string, int> { ["zed"] = 3, ["amy"] = 1, ["kim"] = 2 });
Console.WriteLine($"ranks=[{string.Join(",", ranks.Keys)}]");

ImmutableQueue<string> q1 = ImmutableQueue.Create("first", "second");
ImmutableQueue<string> q2 = q1.Dequeue(out string head).Enqueue("third");
Console.WriteLine($"dequeued={head}; q1=[{string.Join(",", q1)}]; q2=[{string.Join(",", q2)}]");

ImmutableStack<int> s1 = ImmutableStack.Create(1, 2);       // вершина — 2
ImmutableStack<int> s2 = s1.Push(3);
Console.WriteLine($"s1.Peek={s1.Peek()}, s2.Peek={s2.Peek()}, s2.Pop() is s1: {ReferenceEquals(s2.Pop(), s1)}");
```

**Приклад запуску:**
```
empty.Count=0, fruits=[kiwi,apple,pear]
prices: pear=45, count=2; discounted: pear=40, count=1
set=3 items, bigger=1,2,3,4,5
sorted=[10,20,30,50], min=10, index of 30=1
ranks=[amy,kim,zed]
dequeued=first; q1=[first,second]; q2=[second,third]
s1.Peek=2, s2.Peek=3, s2.Pop() is s1: True
```

Останній рядок показує **структурне спільне використання** (structural sharing): `s2` не копіює `s1`, а просто додає новий вузол, що вказує на старий стек.

### 8.10. Персистентні структури та спільні вузли

Як `ImmutableList` робить «зміну» за O(log n), не копіюючи всі n елементів? Змінюється лише **шлях від кореня до зміненого вузла**, решта дерева спільна:

```
v1 = [A, B, C, D, E, F, G]                    v2 = v1.SetItem(4, "X")   // замінити E на X

              D                                         D'   ← нова копія кореня
           /     \                                   /     \
          B       F                          (спільне) B       F'  ← нова копія
         / \     / \                                  / \     / \
        A   C   E   G                                A   C   X   G  ← (G спільний)
                                                                ↑ новий вузол

Скопійовано 3 вузли з 7 (висота дерева ≈ log₂ n). v1 повністю валідний і не змінився.
Для n = 1 000 000 — ≈ 20 нових вузлів замість мільйона елементів.
```

```csharp
using System.Collections.Immutable;

ImmutableList<int> v1 = ImmutableList.CreateRange(Enumerable.Range(1, 1_000_000));

long before = GC.GetAllocatedBytesForCurrentThread();
ImmutableList<int> v2 = v1.SetItem(500_000, -1);             // «зміна» одного елемента
long allocated = GC.GetAllocatedBytesForCurrentThread() - before;

Console.WriteLine($"v1[500000]={v1[500_000]}, v2[500000]={v2[500_000]}");
Console.WriteLine($"allocated for SetItem: {(allocated < 10_000 ? "< 10 KB" : "too much")} (a full copy would be ~40 MB)");

// Версії можна зберігати як історію — undo/redo майже безкоштовно
var history = new Stack<ImmutableList<int>>();
ImmutableList<int> doc = [1, 2, 3];
history.Push(doc);
doc = doc.Add(4);
history.Push(doc);
doc = doc.RemoveAt(0);
Console.WriteLine($"current=[{string.Join(",", doc)}]");
doc = history.Pop();
Console.WriteLine($"undo   =[{string.Join(",", doc)}]");
doc = history.Pop();
Console.WriteLine($"undo   =[{string.Join(",", doc)}]");
```

**Приклад запуску:**
```
v1[500000]=500001, v2[500000]=-1
allocated for SetItem: < 10 KB (a full copy would be ~40 MB)
current=[2,3,4]
undo   =[1,2,3,4]
undo   =[1,2,3]
```

### 8.11. Builders: масові зміни без зайвих алокацій

Кожен `Add` на незмінній колекції створює проміжну версію. Для побудови з тисяч елементів використовують **builder** — змінну «чернетку», яку потім заморожують одним викликом.

```csharp
using System.Collections.Immutable;

const int N = 10_000;

// ❌ N проміжних версій: для ImmutableArray кожен Add копіює весь масив — O(n²)
long before = GC.GetAllocatedBytesForCurrentThread();
ImmutableArray<int> slow = [];
for (int i = 0; i < N; i++)
{
    slow = slow.Add(i);
}
long slowBytes = GC.GetAllocatedBytesForCurrentThread() - before;

// ✔ Builder: одна змінна структура, потім ToImmutable / MoveToImmutable
before = GC.GetAllocatedBytesForCurrentThread();
ImmutableArray<int>.Builder builder = ImmutableArray.CreateBuilder<int>(initialCapacity: N);
for (int i = 0; i < N; i++)
{
    builder.Add(i);
}
ImmutableArray<int> fast = builder.MoveToImmutable();     // без копіювання, якщо Count == Capacity
long fastBytes = GC.GetAllocatedBytesForCurrentThread() - before;

Console.WriteLine($"equal: {slow.SequenceEqual(fast)}");
Console.WriteLine($"Add in loop allocated {(slowBytes > 100_000_000 ? "> 100 MB" : "< 100 MB")}, builder ~{fastBytes / 1024} KB");

// ToBuilder: взяти існуючу незмінну колекцію, змінити пакетно, заморозити знову
ImmutableDictionary<string, int> stock = ImmutableDictionary.CreateRange(
    [KeyValuePair.Create("apple", 5), KeyValuePair.Create("pear", 0), KeyValuePair.Create("plum", 7)]);

ImmutableDictionary<string, int>.Builder draft = stock.ToBuilder();
foreach (var (name, qty) in stock)
{
    if (qty == 0)
    {
        draft.Remove(name);                                // змінюємо чернетку, а не оригінал
    }
    else
    {
        draft[name] = qty * 10;
    }
}
ImmutableDictionary<string, int> updated = draft.ToImmutable();

Console.WriteLine($"stock:   {string.Join(", ", stock.OrderBy(p => p.Key).Select(p => $"{p.Key}={p.Value}"))}");
Console.WriteLine($"updated: {string.Join(", ", updated.OrderBy(p => p.Key).Select(p => $"{p.Key}={p.Value}"))}");
```

**Приклад запуску:**
```
equal: True
Add in loop allocated > 100 MB, builder ~39 KB
stock:   apple=5, pear=0, plum=7
updated: apple=50, plum=70
```

### 8.12. `ImmutableArray<T>` проти `ImmutableList<T>`

| Операція | `ImmutableArray<T>` | `ImmutableList<T>` | `List<T>` (для порівняння) |
|----------|:-------------------:|:------------------:|:--------------------------:|
| Доступ `[i]` | **O(1)**, як масив | O(log n) | O(1) |
| `foreach` | **дуже швидко** (struct-перелічувач, суміжна пам'ять) | повільніше в рази (обхід дерева) | швидко |
| `Add` / `Insert` / `SetItem` | O(n) — копія масиву | **O(log n)** | O(1)* / O(n) |
| Пам'ять на елемент | мінімальна | вузол дерева ≈ 40+ байт | мінімальна |
| Тип | `struct` (обгортка над масивом; `default` ≠ порожній!) | `class` | `class` |
| Коли | створили один раз, багато читаєте | часто «змінюєте», зберігаєте версії | однопотокова змінна колекція |

```csharp
using System.Collections.Immutable;

// ⚠ ImmutableArray — struct: неініціалізоване значення має IsDefault == true
ImmutableArray<int> notInitialized = default;
Console.WriteLine($"IsDefault={notInitialized.IsDefault}, IsDefaultOrEmpty={notInitialized.IsDefaultOrEmpty}");
try
{
    Console.WriteLine(notInitialized.Length);
}
catch (NullReferenceException)
{
    Console.WriteLine("Length on default ImmutableArray -> NullReferenceException");
}

ImmutableArray<int> ok = [];                                 // порожній, але ініціалізований
Console.WriteLine($"[] -> IsDefault={ok.IsDefault}, Length={ok.Length}");
```

**Приклад запуску:**
```
IsDefault=True, IsDefaultOrEmpty=True
Length on default ImmutableArray -> NullReferenceException
[] -> IsDefault=False, Length=0
```

### 8.13. Потокобезпечність через незмінність і `ImmutableInterlocked`

Незмінний об'єкт можна **читати з будь-якої кількості потоків без замків** — його ніхто не змінить. Для «оновлень» використовують шаблон *copy-on-write*: зчитати посилання → побудувати нову версію → атомарно замінити посилання, якщо його ніхто не підмінив (compare-and-swap). `ImmutableInterlocked` робить цей цикл за вас.

```csharp
using System.Collections.Immutable;

var registry = new ServiceRegistry();

Parallel.For(0, 1_000, i => registry.Register($"svc-{i % 50}", $"10.0.0.{i % 50}"));
Parallel.For(0, 10, i => registry.Unregister($"svc-{i}"));

// Читачі бачать цілісний знімок, навіть якщо в цей момент хтось реєструє сервіси
ImmutableDictionary<string, string> snapshot = registry.Snapshot;
Console.WriteLine($"services: {snapshot.Count}");
Console.WriteLine($"svc-42 -> {snapshot["svc-42"]}, svc-3 registered: {snapshot.ContainsKey("svc-3")}");
Console.WriteLine($"generations: {registry.Generation}");

public sealed class ServiceRegistry
{
    private ImmutableDictionary<string, string> _services = ImmutableDictionary<string, string>.Empty;
    private int _generation;

    // Читання — просто повернути поточне посилання. Жодних замків.
    public ImmutableDictionary<string, string> Snapshot => _services;

    public int Generation => _generation;

    public void Register(string name, string address)
    {
        // Атомарно: _services = _services.SetItem(name, address), з повтором при конфлікті
        ImmutableInterlocked.AddOrUpdate(ref _services, name, address, (_, _) => address);
        Interlocked.Increment(ref _generation);
    }

    public void Unregister(string name)
    {
        // Загальний варіант: будь-яке перетворення «стара версія -> нова версія»
        ImmutableInterlocked.Update(ref _services, static (map, key) => map.Remove(key), name);
        Interlocked.Increment(ref _generation);
    }
}
```

**Приклад запуску:**
```
services: 40
svc-42 -> 10.0.0.42, svc-3 registered: False
generations: 1010
```

Що всередині `ImmutableInterlocked.Update` (спрощено):

```csharp
// Спрощена логіка ImmutableInterlocked.Update (не програма)
public static bool Update<T>(ref T location, Func<T, T> transformer) where T : class
{
    while (true)
    {
        T original = Volatile.Read(ref location);          // 1) прочитати поточну версію
        T updated = transformer(original);                  // 2) побудувати нову (може виконатися кілька разів!)
        if (ReferenceEquals(original, updated))
        {
            return false;                                   // нічого не змінилося
        }
        if (ReferenceEquals(Interlocked.CompareExchange(ref location, updated, original), original))
        {
            return true;                                    // 3) атомарно підмінили — успіх
        }
        // інший потік встиг змінити location — повторюємо з новою версією
    }
}
```

### 8.14. `ReadOnlyCollection` — обгортка, а не незмінність

```csharp
using System.Collections.Frozen;
using System.Collections.Immutable;
using System.Collections.ObjectModel;

var source = new Dictionary<string, int> { ["a"] = 1 };
var sourceList = new List<int> { 1, 2 };

ReadOnlyDictionary<string, int> readOnly = source.AsReadOnly();      // .NET 7+: вікно на той самий словник
ReadOnlyCollection<int> readOnlyList = sourceList.AsReadOnly();
ImmutableDictionary<string, int> immutable = source.ToImmutableDictionary();   // копія
FrozenDictionary<string, int> frozen = source.ToFrozenDictionary();            // копія

source["b"] = 2;                                                    // змінюємо ОРИГІНАЛ
sourceList.Add(3);

Console.WriteLine($"ReadOnlyDictionary:  count={readOnly.Count}  (sees the change)");
Console.WriteLine($"ReadOnlyCollection:  count={readOnlyList.Count}  (sees the change)");
Console.WriteLine($"ImmutableDictionary: count={immutable.Count}  (independent copy)");
Console.WriteLine($"FrozenDictionary:    count={frozen.Count}  (independent copy)");

// Через інтерфейс IDictionary «зміна» read-only обгортки кидає виняток
try
{
    ((IDictionary<string, int>)readOnly).Add("c", 3);
}
catch (NotSupportedException)
{
    Console.WriteLine("ReadOnlyDictionary.Add -> NotSupportedException");
}

// Immutable через той самий інтерфейс теж не змінюється — для змін є власні методи, що повертають нову версію
try
{
    ((IDictionary<string, int>)immutable).Add("c", 3);
}
catch (NotSupportedException)
{
    Console.WriteLine("ImmutableDictionary via IDictionary.Add -> NotSupportedException");
}
```

**Приклад запуску:**
```
ReadOnlyDictionary:  count=2  (sees the change)
ReadOnlyCollection:  count=3  (sees the change)
ImmutableDictionary: count=1  (independent copy)
FrozenDictionary:    count=1  (independent copy)
ReadOnlyDictionary.Add -> NotSupportedException
ImmutableDictionary via IDictionary.Add -> NotSupportedException
```

Наслідок для потоків: `ReadOnlyCollection` над `List`, який хтось змінює в іншому потоці, **так само небезпечна**, як і сам `List`. Незмінні та frozen колекції — безпечні.

---

### 8.15. Frozen-колекції (.NET 8)

`FrozenDictionary<TKey,TValue>` і `FrozenSet<T>` із `System.Collections.Frozen` створюються методами `ToFrozenDictionary()` / `ToFrozenSet()` і після цього **не мають жодних методів зміни** (навіть «повернути нову версію»). Натомість під час побудови вони аналізують **конкретний набір ключів** і обирають найшвидшу стратегію пошуку:

| Оптимізація при побудові | Приклад |
|--------------------------|---------|
| Ключі — невеликі цілі числа | прямий масив або компактна хеш-таблиця без `%` |
| Мало елементів (≤ ~10) | лінійний пошук по масиву — швидший за хеш |
| Рядкові ключі | хешується не весь рядок, а **мінімальний підрядок**, що розрізняє ключі (напр., 2 символи з певної позиції) |
| Рядки різної довжини | спершу відсікання за довжиною (якщо довжини немає серед ключів — одразу «ні») |
| `OrdinalIgnoreCase` для ASCII | спеціалізовані порівнювачі без культурних таблиць |
| Розмір хеш-таблиці | підбирається так, щоб колізій майже не було |

```csharp
using System.Collections.Frozen;

// Типове застосування: довідник, що будується один раз при старті програми
FrozenDictionary<string, string> mimeTypes = new Dictionary<string, string>(StringComparer.OrdinalIgnoreCase)
{
    [".html"] = "text/html",
    [".css"] = "text/css",
    [".js"] = "text/javascript",
    [".png"] = "image/png",
    [".json"] = "application/json",
}.ToFrozenDictionary(StringComparer.OrdinalIgnoreCase);

Console.WriteLine($".PNG -> {mimeTypes[".PNG"]}");
Console.WriteLine($".exe known: {mimeTypes.ContainsKey(".exe")}");
Console.WriteLine($"TryGetValue .css: {(mimeTypes.TryGetValue(".css", out string? css) ? css : "none")}");

FrozenSet<string> reservedWords = FrozenSet.ToFrozenSet(["class", "struct", "record", "interface", "enum"]);
Console.WriteLine($"'record' reserved: {reservedWords.Contains("record")}, 'value': {reservedWords.Contains("value")}");

// Keys і Values — ImmutableArray: індексований доступ без алокацій
Console.WriteLine($"keys type: {mimeTypes.Keys.GetType().Name}, count={mimeTypes.Keys.Length}");
```

**Приклад запуску:**
```
.PNG -> image/png
.exe known: False
TryGetValue .css: text/css
'record' reserved: True, 'value': False
keys type: ImmutableArray`1, count=5
```

**Вартість побудови проти швидкості читання:**

```csharp
using System.Collections.Frozen;
using System.Collections.Immutable;
using System.Diagnostics;

const int Keys = 10_000;
const int Lookups = 5_000_000;
string[] keys = [.. Enumerable.Range(0, Keys).Select(i => $"user:{i:D6}")];

var sw = Stopwatch.StartNew();
Dictionary<string, int> dictionary = keys.Select((k, i) => (k, i)).ToDictionary(p => p.k, p => p.i);
TimeSpan buildDictionary = sw.Elapsed;

sw.Restart();
ImmutableDictionary<string, int> immutable = dictionary.ToImmutableDictionary();
TimeSpan buildImmutable = sw.Elapsed;

sw.Restart();
FrozenDictionary<string, int> frozen = dictionary.ToFrozenDictionary();
TimeSpan buildFrozen = sw.Elapsed;

Console.WriteLine($"build: Dictionary {buildDictionary.TotalMilliseconds,6:F2} ms | Immutable {buildImmutable.TotalMilliseconds,6:F2} ms | Frozen {buildFrozen.TotalMilliseconds,6:F2} ms");

long Lookup(IReadOnlyDictionary<string, int> map)
{
    long sum = 0;
    for (int i = 0; i < Lookups; i++)
    {
        sum += map[keys[i % Keys]];
    }
    return sum;
}

foreach (var (name, map) in new (string, IReadOnlyDictionary<string, int>)[] { ("Dictionary", dictionary), ("Immutable", immutable), ("Frozen", frozen) })
{
    Lookup(map);                                            // прогрів
    sw.Restart();
    long sum = Lookup(map);
    Console.WriteLine($"{Lookups:N0} lookups: {name,-10} {sw.Elapsed.TotalMilliseconds,7:F1} ms (checksum {sum})");
}
```

**Орієнтовний вивід** (один запуск без `-c Release`; час побудови сильно залежить від прогріву JIT, співвідношення пошуків — показове):
```
build: Dictionary   5.63 ms | Immutable   5.24 ms | Frozen   6.32 ms
5,000,000 lookups: Dictionary    89.8 ms (checksum 24997500000)
5,000,000 lookups: Immutable    426.8 ms (checksum 24997500000)
5,000,000 lookups: Frozen        60.4 ms (checksum 24997500000)
```

Frozen будується довше за `Dictionary` (у холодному запуску різницю маскує JIT; у BenchmarkDotNet вона зазвичай кратна), але кожен пошук швидший, а `ImmutableDictionary` на пошуках — найповільніший. Виграш окупається, коли пошуків **значно більше**, ніж елементів, — типово для конфігурації, маршрутів, довідників кодів, списків дозволених значень.

### 8.16. Порівняння «словників лише для читання»

| Критерій | `Dictionary` | `ReadOnlyDictionary` | `ImmutableDictionary` | `FrozenDictionary` | `ConcurrentDictionary` |
|----------|:------------:|:--------------------:|:---------------------:|:------------------:|:----------------------:|
| Змінюваність | так | ні через обгортку, **оригінал — так** | ні; «зміни» → нова версія | ні, взагалі | так |
| Потокобезпечність | лише читання без записів | як в оригіналу | ✔ повна | ✔ повна | ✔ повна |
| Пошук | O(1), швидко | O(1) + виклик через обгортку | **O(log n)**, повільніше в 3–5 разів | O(1), **найшвидше** | O(1), трохи повільніше за `Dictionary` |
| Створення | O(n), дешево | O(1) — обгортка | O(n log n), дорого | O(n), найдорожче | O(n) |
| «Зміна» | O(1) | — | O(log n), спільні вузли | — (лише перебудова) | O(1) |
| Знімок для перелічення | ні | ні | ✔ | ✔ | ні |
| Пам'ять | мала | + обгортка | велика (вузли) | мала–середня | велика (вузли + замки) |
| Типове застосування | локальні дані методу/класу | віддати назовні без копії | стан, історія версій, copy-on-write | довідники, конфіг при старті | спільний кеш, лічильники |

### 8.17. Як обрати

```
                        Колекцію використовують кілька потоків?
                          ні │                         │ так
                             ▼                         ▼
                  Звичайна колекція          Чи змінюється вона після побудови?
                  (List, Dictionary...)        ні │                  │ так
                  + AsReadOnly() назовні         ▼                   ▼
                                     Frozen* (часті пошуки)   Записів більше, ніж читань / записи часті?
                                     ImmutableArray (обходи)    так │                    │ ні (рідкі записи,
                                                                   ▼                    ▼  потрібні знімки)
                                                    Потрібна черга між      Immutable* + ImmutableInterlocked
                                                    виробником і споживачем?  (copy-on-write)
                                                      так │         │ ні
                                                          ▼         ▼
                                                   Channel<T>   ConcurrentDictionary /
                                                  (async) або   ConcurrentQueue / lock
                                                  BlockingCollection
```

**Типові помилки розділу 8:**

- Вважати, що `ConcurrentDictionary` робить **послідовність** викликів атомарною: `if (!d.ContainsKey(k)) d.TryAdd(k, Create())` — гонка між двома викликами.
- Побічні ефекти у `valueFactory`/`updateValueFactory` — вони можуть виконатися кілька разів; для дорогих обчислень — `Lazy<T>`.
- Часто викликати `Count`, `Keys`, `Values` у `ConcurrentDictionary` — кожен виклик бере всі замки.
- Забути присвоїти результат `immutable.Add(x)` — зміна «зникає» (аналізатори попереджають, але не завжди).
- Будувати `ImmutableArray`/`ImmutableList` через `Add` у циклі замість builder — O(n²) або гори сміття.
- `default(ImmutableArray<T>)` замість `[]` — `NullReferenceException` при першому ж зверненні.
- Вважати `ReadOnlyCollection`/`AsReadOnly()` потокобезпечними — вони лише обгортка.
- `ImmutableDictionary` «бо читань багато» — пошук там O(log n); для читання після одноразової побудови потрібен `FrozenDictionary`.
- Перебудовувати `FrozenDictionary` на кожен запит або часто — вартість побудови з'їдає виграш.
- Використовувати конкурентні колекції в однопотоковому коді «про запас».
- Забути `CompleteAdding()` / `Writer.Complete()` — споживач чекатиме вічно.

**Міні-вправа 8.1.** Веб-краулер з 8 потоками отримує URL-адреси (з повторами). Кожну адресу треба обробити **рівно один раз**. Напишіть метод `bool TryClaim(string url)` без `lock`, що повертає `true` лише першому потоку, та перевірте на 100 000 викликах з 5 000 унікальних адрес (регістр не важливий).

<details>
<summary>Розв'язок</summary>

```csharp
using System.Collections.Concurrent;

var crawler = new Crawler();
int claimed = 0;

Parallel.For(0, 100_000, new ParallelOptions { MaxDegreeOfParallelism = 8 }, i =>
{
    string url = (i % 2 == 0 ? "HTTPS://SITE/page" : "https://site/page") + (i % 5_000);
    if (crawler.TryClaim(url))
    {
        Interlocked.Increment(ref claimed);
    }
});

Console.WriteLine($"claimed={claimed}, visited={crawler.VisitedCount}");

public sealed class Crawler
{
    // ConcurrentDictionary<T, byte> як конкурентна множина; порівнювач — регістронезалежний
    private readonly ConcurrentDictionary<string, byte> _visited = new(StringComparer.OrdinalIgnoreCase);

    public bool TryClaim(string url) => _visited.TryAdd(url, 0);   // атомарно: true рівно для одного потоку

    public int VisitedCount => _visited.Count;
}
```

**Приклад запуску:**
```
claimed=5000, visited=5000
```

</details>

**Міні-вправа 8.2.** Налаштування застосунку (`FeatureFlags`) читають тисячі запитів на секунду, а змінюють рідко (адміністратор вмикає прапорець). Реалізуйте клас з методами `bool IsEnabled(string flag)` (без замків) і `void Set(string flag, bool enabled)` (потокобезпечно), де читачі завжди бачать цілісний набір. Використайте `FrozenDictionary` та атомарну заміну посилання.

<details>
<summary>Розв'язок</summary>

```csharp
using System.Collections.Frozen;

var flags = new FeatureFlags(new Dictionary<string, bool> { ["dark-mode"] = false, ["beta-search"] = true });

Parallel.For(0, 20, i => flags.Set($"experiment-{i % 5}", i % 2 == 0));
flags.Set("dark-mode", true);

Console.WriteLine($"dark-mode={flags.IsEnabled("dark-mode")}, beta-search={flags.IsEnabled("BETA-SEARCH")}, unknown={flags.IsEnabled("nope")}");
Console.WriteLine($"flags count={flags.Count}");

public sealed class FeatureFlags
{
    private FrozenDictionary<string, bool> _current;          // завжди повністю побудований незмінний знімок
    private readonly Lock _writeGate = new();                  // серіалізує лише рідкісні записи

    public FeatureFlags(IDictionary<string, bool> initial) =>
        _current = initial.ToFrozenDictionary(StringComparer.OrdinalIgnoreCase);

    // Читання: одне звернення до поля + пошук у frozen-словнику, жодних замків
    public bool IsEnabled(string flag) => _current.GetValueOrDefault(flag);

    public int Count => _current.Count;

    public void Set(string flag, bool enabled)
    {
        lock (_writeGate)
        {
            var draft = new Dictionary<string, bool>(_current, StringComparer.OrdinalIgnoreCase)
            {
                [flag] = enabled,
            };
            _current = draft.ToFrozenDictionary(StringComparer.OrdinalIgnoreCase);  // запис посилання атомарний
        }
    }
}
```

**Приклад запуску:**
```
dark-mode=True, beta-search=True, unknown=False
flags count=7
```

Для записів використано `lock` (а не `ImmutableInterlocked`), бо frozen-колекція не має методу «повернути версію з одним зміненим ключем», а перебудова дорога — не варто виконувати її повторно при конфліктах.

</details>

---

## 9. Продуктивність і вибір колекції

### 9.1. Велика порівняльна таблиця

| Колекція | Доступ `[i]` | Пошук | Вставка | Видалення | Порядок | Дублікати | Пам'ять |
|----------|:-----------:|:-----:|:-------:|:---------:|---------|:---------:|---------|
| `T[]` | O(1) | O(n) | — | — | індексний | ✔ | мінімальна |
| `List<T>` | O(1) | O(n) | O(1)* кінець / O(n) | O(n) / O(1) кінець | індексний | ✔ | мала |
| `LinkedList<T>` | — | O(n) | O(1) за вузлом | O(1) за вузлом | вставки | ✔ | велика |
| `Queue<T>` / `Stack<T>` | — | O(n) | O(1)* | O(1) | FIFO / LIFO | ✔ | мала |
| `HashSet<T>` | — | O(1) avg | O(1)* avg | O(1) avg | не визначено | ✗ | середня |
| `Dictionary<K,V>` | — | O(1) avg | O(1)* avg | O(1) avg | не визначено | ключі ✗ | середня |
| `SortedSet<T>` | — | O(log n) | O(log n) | O(log n) | відсортований | ✗ | велика |
| `SortedDictionary<K,V>` | — | O(log n) | O(log n) | O(log n) | відсортований | ключі ✗ | велика |
| `SortedList<K,V>` | O(1) за індексом | O(log n) | O(n) | O(n) | відсортований | ключі ✗ | мала |
| `PriorityQueue<E,P>` | — | O(n) | O(log n) | O(log n) min | лише мінімум | ✔ | мала |
| `ImmutableArray<T>` | O(1) | O(n) | O(n) | O(n) | індексний | ✔ | мінімальна |
| `ImmutableList<T>` | O(log n) | O(n) | O(log n) | O(log n) | індексний | ✔ | велика |
| `FrozenDictionary<K,V>` | — | O(1), найшвидший | — | — | не визначено | ключі ✗ | середня, дорога побудова |
| `ConcurrentDictionary<K,V>` | — | O(1) avg | O(1) avg | O(1) avg | не визначено | ключі ✗ | велика |

\* амортизовано.

### 9.2. Вимірювання: `Stopwatch` і BenchmarkDotNet

Асимптотика — лише половина правди. Константи, кеш процесора й алокації часто важать більше. **Вимірюйте**, але правильно:

- Запускайте в `Release` (`dotnet run -c Release`), не під дебагером.
- Робіть «прогрів» (JIT компілює метод при першому виклику, а tiered JIT потім ще раз оптимізує).
- Повторюйте вимір кілька разів і дивіться на медіану.
- Для серйозних порівнянь використовуйте **BenchmarkDotNet** — він робить усе це автоматично.

Швидкий експеримент зі `Stopwatch`:

```csharp
using System.Diagnostics;

const int N = 20_000;
int[] data = [.. Enumerable.Range(0, N)];
int[] queries = [.. Enumerable.Range(0, N).Select(i => i * 7 % (2 * N))];   // частина є в даних, частина — ні

var list = new List<int>(data);
var set = new HashSet<int>(data);
var sorted = new SortedSet<int>(data);

Measure("List.Contains", () => queries.Count(list.Contains));
Measure("HashSet.Contains", () => queries.Count(set.Contains));
Measure("SortedSet.Contains", () => queries.Count(sorted.Contains));
Measure("Array.BinarySearch", () => queries.Count(q => Array.BinarySearch(data, q) >= 0));

static void Measure(string name, Func<int> action)
{
    action();                                          // прогрів: JIT-компіляція
    var sw = Stopwatch.StartNew();
    int found = 0;
    const int Repeats = 5;
    for (int r = 0; r < Repeats; r++)
    {
        found = action();
    }
    sw.Stop();
    Console.WriteLine($"{name,-20} found={found,6}  {sw.Elapsed.TotalMilliseconds / Repeats,10:F3} ms");
}
```

**Орієнтовний вивід** (час залежить від машини; Apple M-серії, Release):
```
List.Contains        found= 11429     25.945 ms
HashSet.Contains     found= 11429      0.229 ms
SortedSet.Contains   found= 11429      4.954 ms
Array.BinarySearch   found= 11429      1.379 ms
```

Порядок величин передбачуваний: лінійний пошук у списку — O(n·q), хеш — O(q), дерево й бінарний пошук — O(q log n), але бінарний пошук по масиву помітно швидший за дерево завдяки локальності даних.

Той самий порівняльний тест на BenchmarkDotNet (потрібен пакет `BenchmarkDotNet`, запуск у Release):

```csharp
// Шаблон бенчмарку для окремого проєкту з пакетом BenchmarkDotNet (не програма)
using BenchmarkDotNet.Attributes;
using BenchmarkDotNet.Running;

BenchmarkRunner.Run<ContainsBenchmarks>();

[MemoryDiagnoser]                                   // показує алокації
public class ContainsBenchmarks
{
    [Params(100, 10_000)]                           // прогнати для різних розмірів
    public int N;

    private List<int> _list = [];
    private HashSet<int> _set = [];

    [GlobalSetup]
    public void Setup()
    {
        _list = [.. Enumerable.Range(0, N)];
        _set = [.. _list];
    }

    [Benchmark(Baseline = true)]
    public bool ListContains() => _list.Contains(N - 1);   // найгірший випадок для списку

    [Benchmark]
    public bool HashSetContains() => _set.Contains(N - 1);
}
```

Цікавий результат, який зазвичай показує BenchmarkDotNet: для **N ≈ 10–20** `List.Contains` може бути швидшим за `HashSet.Contains` — обчислення хешу та перехід по кошиках коштують більше, ніж кілька порівнянь у суміжній пам'яті.

### 9.3. Попереднє виділення місткості

```csharp
// Рахуємо, скільки разів List і Dictionary перевиділяли внутрішні масиви
const int N = 1_000;

var grown = new List<int>();
int listResizes = 0;
int lastCapacity = grown.Capacity;
for (int i = 0; i < N; i++)
{
    grown.Add(i);
    if (grown.Capacity != lastCapacity)
    {
        listResizes++;                                // кожна зміна Capacity = новий масив + копіювання
        lastCapacity = grown.Capacity;
    }
}
Console.WriteLine($"List without capacity: {listResizes} reallocations, final Capacity={grown.Capacity}");

var preallocated = new List<int>(N);
int before = preallocated.Capacity;
for (int i = 0; i < N; i++)
{
    preallocated.Add(i);
}
Console.WriteLine($"List(N): Capacity {before} -> {preallocated.Capacity}, 0 reallocations");

// Dictionary і HashSet мають EnsureCapacity / TrimExcess
var dict = new Dictionary<int, int>();
int dictCapacity = dict.EnsureCapacity(N);           // повертає фактичну місткість (просте число)
Console.WriteLine($"Dictionary.EnsureCapacity({N}) -> {dictCapacity}");

for (int i = 0; i < N; i++)
{
    dict[i] = i;
}
for (int i = 10; i < N; i++)
{
    dict.Remove(i);                                  // видалення НЕ зменшує масиви
}
dict.TrimExcess();                                   // стискаємо після масового видалення
Console.WriteLine($"after TrimExcess: Count={dict.Count}, Capacity={dict.EnsureCapacity(0)}");

// ⚠ ToList після Select не завжди знає точний розмір — місткість може бути більшою за Count
List<int> fromArray = new int[N].Select(x => x + 1).ToList();
Console.WriteLine($"ToList over array.Select: Capacity={fromArray.Capacity}");
```

**Приклад запуску:**
```
List without capacity: 9 reallocations, final Capacity=1024
List(N): Capacity 1000 -> 1000, 0 reallocations
Dictionary.EnsureCapacity(1000) -> 1103
after TrimExcess: Count=10, Capacity=11
ToList over array.Select: Capacity=1024
```

### 9.4. `struct` проти `class` як елемент колекції

```
List<PointClass> (посилання)                 List<PointStruct> (значення inline)

 _items: [ ref | ref | ref | ref ]            _items: [ X Y | X Y | X Y | X Y ]
            │     │     │     │                        одна неперервна ділянка пам'яті
            ▼     ▼     ▼     ▼
          ┌───┐ ┌───┐ ┌───┐ ┌───┐
          │hdr│ │hdr│ │hdr│ │hdr│  окремі об'єкти в купі:
          │X Y│ │X Y│ │X Y│ │X Y│  заголовок 16 байт + дані, розкидані по пам'яті
          └───┘ └───┘ └───┘ └───┘
```

```csharp
const int N = 1_000_000;

long before = GC.GetAllocatedBytesForCurrentThread();
var classes = new List<PointClass>(N);
for (int i = 0; i < N; i++)
{
    classes.Add(new PointClass(i, i));               // мільйон окремих об'єктів у купі
}
long classBytes = GC.GetAllocatedBytesForCurrentThread() - before;

before = GC.GetAllocatedBytesForCurrentThread();
var structs = new List<PointStruct>(N);
for (int i = 0; i < N; i++)
{
    structs.Add(new PointStruct(i, i));              // значення копіюються прямо у внутрішній масив
}
long structBytes = GC.GetAllocatedBytesForCurrentThread() - before;

Console.WriteLine($"class : ~{classBytes / 1024 / 1024} MB");
Console.WriteLine($"struct: ~{structBytes / 1024 / 1024} MB");

// ⚠ Пастка struct: індексатор List повертає КОПІЮ, змінити поле «на місці» не можна
// structs[0].X = 5;                                 // помилка компіляції CS1612
var copy = structs[0];
copy = copy with { X = 5 };
structs[0] = copy;                                   // треба записати назад
Console.WriteLine($"structs[0] = {structs[0]}");

public sealed class PointClass(int x, int y)
{
    public int X { get; } = x;
    public int Y { get; } = y;
}

public readonly record struct PointStruct(int X, int Y);
```

**Приклад запуску:**
```
class : ~30 MB
struct: ~7 MB
structs[0] = PointStruct { X = 5, Y = 0 }
```

| | Елементи-`struct` | Елементи-`class` |
|--|------------------|-----------------|
| Пам'ять | дані inline, без заголовків | посилання 8 байт + об'єкт ≥ 24 байти |
| Навантаження на GC | один масив | мільйони об'єктів |
| Швидкість обходу | висока (локальність) | нижча (переходи по посиланнях) |
| Зміна елемента | копіювання; потрібно записати назад (або `CollectionsMarshal.AsSpan`) | через посилання «на місці» |
| Великі типи (> ~32 байти) | дороге копіювання при кожному читанні | копіюється лише посилання |
| Як ключ словника | `record struct` / `IEquatable<T>`, інакше повільно | за посиланням або перевизначені `Equals`/`GetHashCode` |

### 9.5. Як обрати колекцію: блок-схема

```
                         ┌────────────────────────────┐
                         │ Потрібен доступ з кількох   │
                         │ потоків із записом?         │
                         └─────────────┬──────────────┘
                              так ┌────┴────┐ ні
                  ┌───────────────┘         └──────────────┐
                  ▼                                        ▼
   Concurrent* / Channel<T>              ┌──────────────────────────────┐
                                         │ Дані змінюються після         │
                                         │ побудови?                     │
                                         └──────────────┬───────────────┘
                                         ні ┌───────────┴────────┐ так
                          ┌─────────────────┘                    │
                          ▼                                      ▼
          FrozenSet / FrozenDictionary /          ┌──────────────────────────────┐
          ImmutableArray                          │ Шукаємо за ключем / значенням?│
                                                  └──────────────┬───────────────┘
                                                  так ┌──────────┴──────────┐ ні
                                    ┌─────────────────┘                     │
                                    ▼                                       ▼
                     ┌─────────────────────────┐          ┌──────────────────────────────┐
                     │ Потрібен порядок ключів │          │ Особливий порядок вилучення?  │
                     │ або запити на діапазон? │          └──────────────┬───────────────┘
                     └────────────┬────────────┘           FIFO │ LIFO │ пріоритет │ ні
                          так ┌───┴───┐ ні                      ▼      ▼      ▼       ▼
                              ▼       ▼                    Queue  Stack  Priority  ┌─────────────────────┐
         ┌─────────────────────┐  ┌──────────────────┐                  Queue      │ Часті вставки в     │
         │ Багато вставок?     │  │ Лише «чи є»?     │                             │ середину за вузлом? │
         └──────────┬──────────┘  └────────┬─────────┘                             └──────────┬──────────┘
            так ┌───┴───┐ ні          так ┌─┴──┐ ні                                    так ┌───┴───┐ ні
                ▼       ▼                 ▼    ▼                                           ▼       ▼
       SortedSet /  SortedList       HashSet  Dictionary                          LinkedList  List<T> / T[]
       SortedDictionary
```

**Короткі правила:**

1. За замовчуванням — `List<T>` (або `T[]`, якщо розмір не змінюється).
2. Пошук за ключем — `Dictionary`; «чи бачили вже» — `HashSet`.
3. Порядок / діапазони / мін-макс з оновленнями — `SortedSet` / `SortedDictionary`.
4. «Найменший/найбільший наступним» без повного порядку — `PriorityQueue`.
5. Дані для читання, побудовані один раз, — `Frozen*`; незмінні знімки — `ImmutableArray`.
6. Потоки — `Concurrent*` та `Channel<T>`.
7. Сумніваєтесь — **виміряйте** на реальних даних.

**Типові помилки розділу 9:**

- Оптимізувати без вимірів, або міряти в Debug-збірці.
- Порівнювати асимптотику для n = 10 — там виграють прості масиви.
- Забувати `capacity` при відомому розмірі — зайві копіювання і сміття.
- Великі змінювані `struct` у колекціях — дороге копіювання та «зміни, що губляться».

---

## 10. Практичні задачі

Кожна задача: умова → ідея та вибір колекції → код → складність.

### 10.1. Частота слів

**Умова.** Дано текст. Виведіть 5 найчастіших слів (без урахування регістру й розділових знаків); при однаковій частоті — за алфавітом.

**Ідея.** `Dictionary<string,int>` з `OrdinalIgnoreCase` для підрахунку, потім сортування пар.

```csharp
using System.Runtime.InteropServices;

const string Text = """
    It was the best of times, it was the worst of times, it was the age of wisdom,
    it was the age of foolishness, it was the epoch of belief, it was the epoch of incredulity.
    """;

var freq = new Dictionary<string, int>(StringComparer.OrdinalIgnoreCase);
foreach (string raw in Text.Split([' ', '\n', '\r', ',', '.'], StringSplitOptions.RemoveEmptyEntries))
{
    ref int count = ref CollectionsMarshal.GetValueRefOrAddDefault(freq, raw.ToLowerInvariant(), out _);
    count++;                                                   // один пошук на слово
}

Console.WriteLine($"distinct words: {freq.Count}");
foreach (var (word, count) in freq
             .OrderByDescending(p => p.Value)                  // спочатку за частотою
             .ThenBy(p => p.Key, StringComparer.Ordinal)       // тай-брейк за алфавітом — детермінований результат
             .Take(5))
{
    Console.WriteLine($"  {word,-6} {count}");
}
```

**Приклад запуску:**
```
distinct words: 13
  it     6
  of     6
  the    6
  was    6
  age    2
```

Складність: O(n) на підрахунок + O(u log u) на сортування унікальних слів.

### 10.2. Групування анаграм

**Умова.** Згрупуйте слова, що є анаграмами одне одного: `["eat","tea","tan","ate","nat","bat"]` → `[ate,eat,tea] [nat,tan] [bat]`.

**Ідея.** Канонічний ключ анаграми — відсортовані літери. `Dictionary<string, List<string>>`.

```csharp
string[] words = ["eat", "tea", "tan", "ate", "nat", "bat"];

var groups = new Dictionary<string, List<string>>();
foreach (string word in words)
{
    char[] letters = word.ToCharArray();
    Array.Sort(letters);                                  // "tea" -> "aet"
    string key = new(letters);

    if (!groups.TryGetValue(key, out List<string>? group))
    {
        group = [];
        groups.Add(key, group);
    }
    group.Add(word);
}

foreach (List<string> group in groups.Values.OrderByDescending(g => g.Count).ThenBy(g => g.Min(StringComparer.Ordinal)))
{
    group.Sort(StringComparer.Ordinal);
    Console.WriteLine($"[{string.Join(",", group)}]");
}

// Альтернатива в один вираз через LINQ GroupBy
var linqGroups = words.GroupBy(w => string.Concat(w.Order())).Select(g => g.Count());
Console.WriteLine($"group sizes via LINQ: {string.Join(",", linqGroups)}");
```

**Приклад запуску:**
```
[ate,eat,tea]
[nat,tan]
[bat]
group sizes via LINQ: 3,2,1
```

Складність: O(n · k log k), де k — довжина слова. Для латиниці можна ключем взяти рядок-лічильник 26 літер — O(n · k).

### 10.3. Two Sum

**Умова.** Знайти індекси двох чисел масиву, сума яких дорівнює `target`.

**Ідея.** Проходимо один раз; словник «значення → індекс». Для кожного `x` перевіряємо, чи бачили `target - x`.

```csharp
static (int, int)? TwoSum(int[] nums, int target)
{
    var seen = new Dictionary<int, int>(nums.Length);        // значення -> індекс
    for (int i = 0; i < nums.Length; i++)
    {
        if (seen.TryGetValue(target - nums[i], out int j))
        {
            return (j, i);                                    // знайшли пару за O(1)
        }
        seen.TryAdd(nums[i], i);                              // TryAdd: при дублікатах зберігаємо перший індекс
    }
    return null;
}

Console.WriteLine(TwoSum([2, 7, 11, 15], 9));
Console.WriteLine(TwoSum([3, 2, 4], 6));
Console.WriteLine(TwoSum([3, 3], 6));
Console.WriteLine(TwoSum([1, 2, 3], 100)?.ToString() ?? "no pair");
```

**Приклад запуску:**
```
(0, 1)
(1, 2)
(0, 1)
no pair
```

| Підхід | Час | Пам'ять |
|--------|:---:|:-------:|
| Два вкладені цикли | O(n²) | O(1) |
| Сортування + два вказівники | O(n log n) | O(n) (щоб зберегти індекси) |
| **Словник за один прохід** | **O(n)** | O(n) |

### 10.4. LRU-кеш (Dictionary + LinkedList)

**Умова.** Кеш на `capacity` елементів: `Get(key)` і `Put(key, value)` за O(1). При переповненні видаляється елемент, до якого **найдовше не звертались** (Least Recently Used).

**Ідея.**

```
 Dictionary<key, LinkedListNode>          LinkedList (голова — найсвіжіший)
 ┌─────┬──────┐
 │  1  │  ●───┼──────────────┐
 │  3  │  ●───┼────┐         │
 │  4  │  ●───┼─┐  │         │
 └─────┴──────┘ │  │         │
                ▼  ▼         ▼
       First → [4:D] ⇄ [3:C] ⇄ [1:A] ← Last (кандидат на видалення)

 Get(1): знайти вузол у словнику O(1) → Remove(node) O(1) → AddFirst(node) O(1)
```

```csharp
var cache = new LruCache<int, string>(capacity: 3);
cache.Put(1, "A");
cache.Put(2, "B");
cache.Put(3, "C");
Console.WriteLine($"state: {cache}");

Console.WriteLine($"Get(1) = {cache.Get(1)}");          // 1 стає найсвіжішим
Console.WriteLine($"state: {cache}");

cache.Put(4, "D");                                       // переповнення: витісняємо найстаріший — 2
Console.WriteLine($"Put(4) state: {cache}");
Console.WriteLine($"Get(2) = {cache.Get(2) ?? "miss"}");

cache.Put(3, "C2");                                      // оновлення існуючого ключа теж «освіжає» його
Console.WriteLine($"Put(3,C2) state: {cache}");

public sealed class LruCache<TKey, TValue>(int capacity) where TKey : notnull
{
    private readonly Dictionary<TKey, LinkedListNode<(TKey Key, TValue Value)>> _map = new(capacity);
    private readonly LinkedList<(TKey Key, TValue Value)> _order = new();   // First — найсвіжіший

    public TValue? Get(TKey key)
    {
        if (!_map.TryGetValue(key, out var node))
        {
            return default;
        }
        MoveToFront(node);
        return node.Value.Value;
    }

    public void Put(TKey key, TValue value)
    {
        if (_map.TryGetValue(key, out var node))
        {
            node.Value = (key, value);                   // оновлюємо значення у вузлі
            MoveToFront(node);
            return;
        }

        if (_map.Count == capacity)
        {
            LinkedListNode<(TKey Key, TValue Value)> oldest = _order.Last!;
            _order.RemoveLast();                         // O(1)
            _map.Remove(oldest.Value.Key);               // ключ у вузлі потрібен саме для цього
        }

        _map[key] = _order.AddFirst((key, value));       // AddFirst повертає створений вузол
    }

    private void MoveToFront(LinkedListNode<(TKey Key, TValue Value)> node)
    {
        _order.Remove(node);                             // O(1): вузол відомий
        _order.AddFirst(node);                           // той самий об'єкт вузла — посилання в словнику валідне
    }

    public override string ToString() => string.Join(" ", _order.Select(e => $"{e.Key}:{e.Value}"));
}
```

**Приклад запуску:**
```
state: 3:C 2:B 1:A
Get(1) = A
state: 1:A 3:C 2:B
Put(4) state: 4:D 1:A 3:C
Get(2) = miss
Put(3,C2) state: 3:C2 4:D 1:A
```

> У .NET 9+ можна зробити LRU і на `OrderedDictionary<K,V>`, але видалення з його початку — O(n) (масив). Пара `Dictionary` + `LinkedList` дає справжні O(1).

### 10.5. Top-k частих елементів

**Умова.** Повернути k найчастіших чисел. При однаковій частоті — менше число першим.

**Ідея.** Підрахунок у `Dictionary`, потім min-купа розміру k з пріоритетом `(частота, -число)`. Альтернатива — «кошикове сортування» за частотою за O(n).

```csharp
static int[] TopKFrequent(int[] nums, int k)
{
    var counts = new Dictionary<int, int>();
    foreach (int x in nums)
    {
        counts[x] = counts.GetValueOrDefault(x) + 1;
    }

    // Пріоритет — (частота, -значення): у корені min-купи «найслабший» кандидат
    var heap = new PriorityQueue<int, (int Count, int NegValue)>();
    foreach (var (value, count) in counts)
    {
        heap.Enqueue(value, (count, -value));
        if (heap.Count > k)
        {
            heap.Dequeue();                                   // викидаємо найслабшого — розмір купи ≤ k
        }
    }

    var result = new int[heap.Count];
    for (int i = result.Length - 1; i >= 0; i--)
    {
        result[i] = heap.Dequeue();                           // найсильніші виходять останніми
    }
    return result;
}

// Варіант O(n): bucket sort за частотою
static int[] TopKFrequentBuckets(int[] nums, int k)
{
    var counts = nums.CountBy(x => x).ToDictionary();         // .NET 9: CountBy -> пари (ключ, кількість)
    var buckets = new List<int>[nums.Length + 1];             // buckets[f] — числа з частотою f
    foreach (var (value, count) in counts)
    {
        (buckets[count] ??= []).Add(value);
    }

    var result = new List<int>(k);
    for (int f = buckets.Length - 1; f > 0 && result.Count < k; f--)
    {
        if (buckets[f] is { } bucket)
        {
            bucket.Sort();                                    // тай-брейк: менше число першим
            result.AddRange(bucket.Take(k - result.Count));
        }
    }
    return [.. result];
}

int[] data = [1, 1, 1, 2, 2, 3, 4, 4, 5, 5, 5, 5];
Console.WriteLine($"heap:    {string.Join(",", TopKFrequent(data, 3))}");
Console.WriteLine($"buckets: {string.Join(",", TopKFrequentBuckets(data, 3))}");
```

**Приклад запуску:**
```
heap:    5,1,2
buckets: 5,1,2
```

Складність: купа — O(n + u log k), кошики — O(n + u log u) у найгіршому випадку через сортування всередині кошика (без тай-брейку — O(n)).

### 10.6. Кількість різних елементів у ковзному вікні

**Умова.** Для кожного вікна довжини k виведіть кількість різних елементів: `[1,2,1,3,4,2,3]`, k = 4 → `3 4 4 3`.

**Ідея.** `Dictionary<int,int>` «значення → скільки разів у вікні». Зсув вікна — одне додавання й одне видалення, O(1). Кількість різних — `Count` словника.

```csharp
static List<int> DistinctInWindows(int[] a, int k)
{
    var inWindow = new Dictionary<int, int>();
    var result = new List<int>(Math.Max(0, a.Length - k + 1));

    for (int i = 0; i < a.Length; i++)
    {
        inWindow[a[i]] = inWindow.GetValueOrDefault(a[i]) + 1;     // новий правий елемент

        if (i >= k)
        {
            int left = a[i - k];                                     // елемент, що вийшов зліва
            if (--inWindow[left] == 0)
            {
                inWindow.Remove(left);                               // інакше Count рахував би «нулі»
            }
        }

        if (i >= k - 1)
        {
            result.Add(inWindow.Count);
        }
    }
    return result;
}

Console.WriteLine(string.Join(" ", DistinctInWindows([1, 2, 1, 3, 4, 2, 3], 4)));
Console.WriteLine(string.Join(" ", DistinctInWindows([5, 5, 5, 5], 2)));

// Варіація: максимум у ковзному вікні — монотонна двобічна черга на LinkedList (O(n) сумарно)
static List<int> MaxInWindows(int[] a, int k)
{
    var deque = new LinkedList<int>();                               // індекси; значення спадають від голови до хвоста
    var result = new List<int>();
    for (int i = 0; i < a.Length; i++)
    {
        while (deque.Count > 0 && a[deque.Last!.Value] <= a[i])
        {
            deque.RemoveLast();                                      // менші за новий — вже ніколи не будуть максимумом
        }
        deque.AddLast(i);
        if (deque.First!.Value <= i - k)
        {
            deque.RemoveFirst();                                     // вийшов за межі вікна
        }
        if (i >= k - 1)
        {
            result.Add(a[deque.First.Value]);
        }
    }
    return result;
}

Console.WriteLine(string.Join(" ", MaxInWindows([1, 3, -1, -3, 5, 3, 6, 7], 3)));
```

**Приклад запуску:**
```
3 4 4 3
1 1 1
3 3 5 5 6 7
```

Складність: O(n) час, O(k) пам'ять для обох варіантів.

### 10.7. Таблиця лідерів на `SortedSet`

**Умова.** Онлайн-гра: гравці оновлюють рахунок; треба швидко отримувати топ-N і змінювати рахунок будь-якого гравця.

**Ідея.** `SortedSet<(int Score, string Name)>` з компаратором «рахунок спаданням, ім'я зростанням» + `Dictionary<string,int>` для поточного рахунку (щоб знайти й видалити старий запис з дерева за O(log n)).

```csharp
var board = new Leaderboard();
board.Submit("ann", 120);
board.Submit("bob", 95);
board.Submit("cid", 150);
board.Submit("dan", 95);
board.Submit("eve", 80);

Console.WriteLine($"top-3: {string.Join(", ", board.Top(3))}");

board.Submit("eve", 200);                               // оновлення: видалити старий запис, вставити новий
board.Submit("cid", 10);
Console.WriteLine($"top-3: {string.Join(", ", board.Top(3))}");
Console.WriteLine($"rank of bob: {board.RankOf("bob")}, rank of cid: {board.RankOf("cid")}");
Console.WriteLine($"players with 90..130: {string.Join(", ", board.InRange(90, 130))}");

public sealed class Leaderboard
{
    // Порядок: більший рахунок раніше; при рівності — за іменем (тай-брейк, щоб не втрачати гравців)
    private readonly SortedSet<(int Score, string Name)> _ranking = new(
        Comparer<(int Score, string Name)>.Create((a, b) =>
        {
            int byScore = b.Score.CompareTo(a.Score);
            return byScore != 0 ? byScore : string.CompareOrdinal(a.Name, b.Name);
        }));

    private readonly Dictionary<string, int> _scores = [];

    public void Submit(string name, int score)
    {
        if (_scores.TryGetValue(name, out int old))
        {
            _ranking.Remove((old, name));                 // O(log n): знаємо точний старий запис
        }
        _scores[name] = score;
        _ranking.Add((score, name));                      // O(log n)
    }

    public IEnumerable<string> Top(int n) => _ranking.Take(n).Select(e => $"{e.Name}={e.Score}");  // O(log n + n)

    // Ранг = кількість гравців «попереду» + 1. Через вікно дерева — O(log n + rank)
    public int RankOf(string name)
    {
        int score = _scores[name];
        return _ranking.GetViewBetween(_ranking.Min, (score, name)).Count;
    }

    // Порядок компаратора спадаючий, тому «нижня» межа вікна — більший рахунок
    public IEnumerable<string> InRange(int minScore, int maxScore) =>
        _ranking.GetViewBetween((maxScore, ""), (minScore, "￿")).Select(e => e.Name);
}
```

**Приклад запуску:**
```
top-3: cid=150, ann=120, bob=95
top-3: eve=200, ann=120, bob=95
rank of bob: 3, rank of cid: 5
players with 90..130: ann, bob, dan
```

| Операція | `SortedSet` + `Dictionary` | Лише `List` + сортування |
|----------|:-------------------------:|:------------------------:|
| Оновити рахунок | O(log n) | O(n log n) (пересортувати) |
| Топ-N | O(log n + N) | O(N) |
| Ранг гравця | O(log n + rank) | O(n) |
| Діапазон рахунків | O(log n + k) | O(log n + k) |

> Для рангу за O(log n) потрібне дерево з розміром піддерев (order-statistic tree) — у стандартній бібліотеці .NET його немає; це тема лекції про дерева.

**Міні-вправа 10.** Реалізуйте `FirstUniqueChar(string s)` — індекс першого символу, що зустрічається рівно один раз, або -1. Два проходи, O(n).

<details>
<summary>Розв'язок</summary>

```csharp
static int FirstUniqueChar(string s)
{
    var counts = new Dictionary<char, int>();
    foreach (char c in s)
    {
        counts[c] = counts.GetValueOrDefault(c) + 1;   // прохід 1: частоти
    }
    for (int i = 0; i < s.Length; i++)
    {
        if (counts[s[i]] == 1)                         // прохід 2: перший з частотою 1 у порядку рядка
        {
            return i;
        }
    }
    return -1;
}

Console.WriteLine(FirstUniqueChar("leetcode"));
Console.WriteLine(FirstUniqueChar("loveleetcode"));
Console.WriteLine(FirstUniqueChar("aabb"));
```

**Приклад запуску:**
```
0
2
-1
```

</details>

---

## 11. Підсумок, питання та завдання

### 11.1. Головне

- .NET розділяє **колекції**, **перелічення** (`IEnumerable<T>`) та **алгоритми** (LINQ); інтерфейси з'єднують їх між собою.
- Приймайте **найзагальніший** інтерфейс (`IEnumerable<T>`, `IReadOnlyList<T>`), зберігайте конкретний тип всередині класу, не віддавайте змінювані колекції назовні.
- `yield` будує ліниві ітератори; LINQ успадковує **відкладене виконання** — пам'ятайте про повторне перелічення та захоплені змінні.
- `Dictionary`/`HashSet` — хеш-таблиця: O(1) у середньому, але лише за **коректних і незмінних** `Equals`/`GetHashCode`; порядок не гарантований.
- `SortedSet`/`SortedDictionary` — червоно-чорне дерево: O(log n), порядок, Min/Max, `GetViewBetween`; `SortedList` — компактні масиви з O(n) вставкою.
- `PriorityQueue` — min-купа: O(log n) вставка/вилучення, top-k, злиття k списків, Dijkstra; нестабільна при рівних пріоритетах.
- Незмінність: `ReadOnlyCollection` (вікно) < `ImmutableArray/List` (нові версії) < `Frozen*` (оптимізовані під читання).
- Для потоків — `ConcurrentDictionary` з атомарними `AddOrUpdate`/`GetOrAdd` (фабрика може виконатися двічі → `Lazy<T>`), `ConcurrentDictionary<T,byte>` як множина, `Channel<T>` для producer/consumer.
- Незмінні колекції ділять вузли між версіями (O(log n) «зміни»), масові зміни — через builder, атомарні оновлення — `ImmutableInterlocked`; `Frozen*` — найшвидше читання після дорогої одноразової побудови.
- Асимптотика — не все: локальність, алокації, константи. Вимірюйте в Release, використовуйте BenchmarkDotNet.

### 11.2. Питання для самоперевірки

1. Чому `Add` у `List<T>` має амортизовану складність O(1), а не строго O(1)?
2. Чим відрізняється читання `freq[key]` від `freq.TryGetValue(key, out var v)` у `Dictionary<TKey,TValue>`?
3. Коли `SortedSet<T>` кращий за `HashSet<T>`, незважаючи на гіршу асимптотику пошуку?
4. Чому `LinkedList<T>` не має методу `Sort` і що використати натомість?
5. Що станеться, якщо видалити елемент зі `List<T>` всередині `foreach`, і як правильно видаляти елементи під час обходу?
6. Що генерує компілятор для методу з `yield return` і коли починає виконуватися його тіло?
7. Чому перевірку аргументів у ітераторі виносять в окремий неітераторний метод?
8. Чим `IReadOnlyList<T>` відрізняється від `ImmutableArray<T>`? Наведіть приклад, коли «read-only» колекція змінюється.
9. Опишіть, як `Dictionary` знаходить значення за ключем: хеш, кошик, ланцюжок, `Equals`.
10. Сформулюйте контракт `Equals`/`GetHashCode`. Що станеться, якщо перевизначити лише `Equals`?
11. Чому не можна змінювати поля об'єкта, який уже є ключем словника? Як захиститися від цієї помилки?
12. Навіщо передавати `StringComparer.OrdinalIgnoreCase` у конструктор словника і чим він кращий за `ToLower()` ключів?
13. Чим відрізняються `HashSet.IntersectWith` і LINQ `Intersect`?
14. Чому компаратор для `SortedSet<T>` повинен повертати 0 лише для однакових елементів? Що буде інакше?
15. Коли обрати `SortedList<K,V>`, а коли `SortedDictionary<K,V>`?
16. Як отримати max-купу з `PriorityQueue<TElement,TPriority>` і як зробити порядок стабільним для рівних пріоритетів?
17. Чому пошук top-k через купу розміру k ефективніший за повне сортування? Оцініть складність.
18. Які оператори LINQ відкладені, а які виконуються негайно? Що таке «багаторазове перелічення»?
19. Яка складність `list.Where(x => other.Contains(x))`, якщо `other` — `List<T>`? Як виправити?
20. Чому `counts[key]++` небезпечне на `ConcurrentDictionary` і що використати натомість?
21. Коли `FrozenDictionary` дає виграш, а коли — лише зайві витрати?
22. Навіщо вказувати `capacity` у конструкторі `List<T>`/`Dictionary`? Що робить `TrimExcess`?
23. Чим відрізняється `List<struct>` від `List<class>` за пам'яттю та поведінкою при зміні елемента?
24. Як поєднати `Dictionary` і `LinkedList`, щоб отримати LRU-кеш з O(1) операціями?
25. Чому `valueFactory` у `ConcurrentDictionary.GetOrAdd` може виконатися кілька разів і як `Lazy<T>` це виправляє?
26. Чому в .NET немає `ConcurrentHashSet<T>` і які є три заміни? Порівняйте їх за записом, читанням і переліченням.
27. Що таке структурне спільне використання (structural sharing) і чому `ImmutableList.SetItem` не копіює весь список?
28. Чим відрізняються `ReadOnlyDictionary`, `ImmutableDictionary` і `FrozenDictionary` за змінюваністю, потокобезпечністю, швидкістю пошуку та вартістю створення?
29. Навіщо потрібні builder-и незмінних колекцій і що станеться з продуктивністю без них?

### 11.3. Завдання для практики

| № | Задача | Ключова колекція | Рівень |
|---|--------|------------------|:------:|
| 1 | Ітератор `Batch<T>(IEnumerable<T>, int size)` без використання `Chunk` | `yield`, `List<T>` | ★ |
| 2 | Перевірити, чи є рядок перестановкою паліндрому | `HashSet<char>` / `Dictionary` | ★ |
| 3 | Знайти всі пари чисел з різницею k за O(n) | `HashSet<int>` | ★ |
| 4 | Ізоморфні рядки (`egg` ↔ `add`) | два `Dictionary<char,char>` | ★ |
| 5 | Регістронезалежний телефонний довідник з пошуком за префіксом | `SortedDictionary` + `GetViewBetween` | ★★ |
| 6 | Найдовший підрядок без повторюваних символів | `Dictionary<char,int>`, ковзне вікно | ★★ |
| 7 | Планувальник задач з пріоритетами та FIFO для рівних пріоритетів | `PriorityQueue` з `(priority, seq)` | ★★ |
| 8 | k найближчих точок до початку координат | max-купа розміру k | ★★ |
| 9 | Мінімальна кількість конференц-залів для інтервалів | сортування + `PriorityQueue` | ★★ |
| 10 | LFU-кеш (Least Frequently Used) з O(1) операціями | `Dictionary` + `Dictionary<int, LinkedList>` | ★★★ |
| 11 | Медіана ковзного вікна | дві `SortedSet` з тай-брейком за індексом | ★★★ |
| 12 | «Календар» з підрахунком максимального перекриття бронювань | `SortedDictionary<int,int>` (різницевий масив) | ★★★ |
| 13 | Порівняти BenchmarkDotNet-ом `Dictionary`, `SortedDictionary`, `FrozenDictionary` для 10, 1 000, 100 000 ключів | BenchmarkDotNet | ★★ |
| 14 | Паралельний підрахунок слів у 100 файлах | `Channel<string>` + `ConcurrentDictionary` | ★★★ |
