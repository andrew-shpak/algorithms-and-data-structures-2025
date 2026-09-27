# Лабораторна робота 2 — виконайте 2 завдання для однієї структури: реалізацію (3 бали) та алгоритм середньої складності (2 бали)

> **C# / .NET 10.** [Налаштування та запуск](../LABS.md).

Оберіть **одну структуру даних** і виконайте **обидва завдання відповідної пари**:

- **Перше:** власна реалізація обраної структури даних, **3 бали**.
- **Друге:** алгоритм середньої складності для цієї самої структури, **2 бали**.

| Обрана структура | Реалізація — 3 бали | Алгоритм — 2 бали |
|---|---|---|
| Стек | Завдання **1** | Завдання **4** |
| Динамічний список | Завдання **2** | Завдання **5** |
| Двозв'язний список | Завдання **3** | Завдання **6** |

**Разом: 5 балів за одну пару.** Дозволені лише пари **1 + 4**, **2 + 5**
або **3 + 6**. Наприклад, реалізація списку обов'язково доповнюється алгоритмом
для списку — завданнями 2 і 5. Виконувати всі шість завдань не потрібно.

| № | Завдання | Група | Бали |
|---|---|---|---|
| 1 | Власний стек `LabStack<T>` | Реалізація | 3 |
| 2 | Власний динамічний список `LabList<T>` | Реалізація | 3 |
| 3 | Власний двозв'язний список `LabLinkedList<T>` | Реалізація | 3 |
| 4 | Сортування `Stack<int>` допоміжним стеком | Середня складність | 2 |
| 5 | Наступна перестановка `List<int>` | Середня складність | 2 |
| 6 | Розворот `LinkedList<int>` групами по k вузлів | Середня складність | 2 |

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
У завданнях **4–6** реалізуйте алгоритм над готовою колекцією .NET:
**4 — `Stack<int>`, 5 — `List<int>`, 6 — `LinkedList<int>`**. У кожному випадку
потрібно змінити передану колекцію за правилами задачі. Власна структура з
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

## Завдання 4. Сортування стека допоміжним стеком — 2 бали

Реалізуйте алгоритм сортування **`Stack<int>`** так, щоб після завершення
найменший елемент був на вершині. Послідовні виклики `Pop()` мають повертати
значення за неспаданням. Працюйте з тим самим об'єктом стека.

### Умова

- Реалізуйте `SortStack(Stack<int> stack)` без повернення нової колекції.
- Дозволено створити **лише один допоміжний `Stack<int>`** та сталу кількість
  локальних змінних. Для роботи з обома стеками використовуйте тільки
  `Push`, `Pop`, `Peek` і `Count`.
- Заборонені масиви, списки, LINQ, готові сортування, рекурсія та обхід стека
  через `foreach` усередині алгоритму. Обхід у демонстрації потрібен лише для друку.
- Збережіть усі значення та кількість повторів. Від'ємні числа дозволені.
  Порожній стек і стек з одним елементом не змінюються.
- Вхід містить не більше 1 000 елементів. Значення можуть бути будь-якими `int`.
- **Складність:** не гірше O(n²) часу та O(n) додаткової пам'яті.

### `Program.cs` — заготовка без реалізації

```csharp
// Конструктор додає значення зліва направо: останнє стане вершиною.
int[][] examples = [[3, 1, 4, 2], [2, -1, 2, 0], [5], []];

foreach (int[] input in examples)
{
    var stack = new Stack<int>(input);
    string before = stack.Count == 0 ? "(порожньо)" : string.Join(' ', stack);
    SortStack(stack);
    string after = stack.Count == 0 ? "(порожньо)" : string.Join(' ', stack);
    Console.WriteLine($"Вершина → дно: {before}; після: {after}");
}

static void SortStack(Stack<int> stack)
{
    throw new NotImplementedException();
}
```

### Очікуваний вивід після реалізації

```text
Вершина → дно: 2 4 1 3; після: 1 2 3 4
Вершина → дно: 0 2 -1 2; після: -1 0 2 2
Вершина → дно: 5; після: 5
Вершина → дно: (порожньо); після: (порожньо)
```

Додатково перевірте вже відсортований стек, зворотний порядок, однакові
значення та `int.MinValue` / `int.MaxValue`. Поясніть, у якому порядку
зберігаються елементи допоміжного стека під час роботи алгоритму.

## Завдання 5. Наступна лексикографічна перестановка списку — 2 бали

Реалізуйте алгоритм пошуку **наступної перестановки `List<int>`** на місці.
Потрібно отримати найменшу послідовність, що лексикографічно більша за поточну
і містить ті самі значення з тими самими кількостями повторів.

Лексикографічний порядок визначається першою відмінною позицією: наприклад,
`[1, 3, 2] < [2, 1, 3]`, тому що `1 < 2`. Для `[1, 3, 2]` наступною
перестановкою є `[2, 1, 3]`.

### Умова

- Реалізуйте `bool NextPermutation(List<int> values)`.
- Якщо більша перестановка існує, запишіть її в **той самий список** і поверніть `true`.
- Якщо більшої перестановки немає, упорядкуйте значення за неспаданням
  і поверніть `false`: це перехід до найменшої перестановки.
- Порожній список, один елемент та список однакових значень залишаються
  незмінними; результат — `false`.
- Дозволені порівняння, доступ через індексатор, `Count` та обмін окремих
  елементів. Не створюйте додаткових колекцій і не змінюйте довжину списку.
- Заборонені `Sort`, `Reverse`, LINQ, рекурсія та перебір усіх перестановок.
  Усі необхідні перестановки елементів виконайте самостійно.
- Вхід містить не більше 100 000 елементів; дозволені будь-які значення `int`,
  зокрема від'ємні й повторювані.
- **Складність:** O(n) часу та O(1) додаткової пам'яті.

### `Program.cs` — заготовка без реалізації

```csharp
List<int>[] examples =
[
    [1, 2, 3],
    [1, 3, 2],
    [3, 2, 1],
    [1, 1, 5],
    [2, 2],
    [7],
    [],
];

foreach (List<int> values in examples)
{
    string before = values.Count == 0 ? "(порожньо)" : string.Join(' ', values);
    bool hasNext = NextPermutation(values);
    string after = values.Count == 0 ? "(порожньо)" : string.Join(' ', values);
    Console.WriteLine($"{before} → {after}; наступна існує: {hasNext}");
}

static bool NextPermutation(List<int> values)
{
    throw new NotImplementedException();
}
```

### Очікуваний вивід після реалізації

```text
1 2 3 → 1 3 2; наступна існує: True
1 3 2 → 2 1 3; наступна існує: True
3 2 1 → 1 2 3; наступна існує: False
1 1 5 → 1 5 1; наступна існує: True
2 2 → 2 2; наступна існує: False
7 → 7; наступна існує: False
(порожньо) → (порожньо); наступна існує: False
```

Додатково перевірте від'ємні числа та довгий спадний фрагмент у кінці списку.
Для `[1, 2, 3]` послідовними викликами отримайте всі шість перестановок і
повернення до початкової. Поясніть, чому алгоритм знаходить саме найближчу більшу.

## Завдання 6. Розворот зв'язного списку групами по k вузлів — 2 бали

Реалізуйте алгоритм зміни порядку вузлів **`LinkedList<int>`**: розділіть
послідовність від початку на групи по k вузлів і розверніть кожну повну групу.
Якщо в останній групі менше ніж k вузлів, збережіть її початковий порядок.

### Умова

- Реалізуйте `ReverseGroups(LinkedList<int> values, int groupSize)`.
  Змінюйте **той самий список**.
- Переставляйте наявні об'єкти `LinkedListNode<int>` через операції
  від'єднання та приєднання (`Remove`, `AddBefore`, `AddAfter`, за потреби
  `AddFirst` / `AddLast`). Використовуйте перевантаження, що приймають вузол.
- Не змінюйте `Value`, не створюйте нових вузлів і не копіюйте значення
  в масив, стек або іншу колекцію. Рекурсія та LINQ усередині алгоритму заборонені.
- Використовуйте `First`, `Last`, `Next`, `Previous` і сталу кількість
  посилань на вузли. Кожен початковий вузол має залишитися в списку рівно один раз.
- `groupSize == 1`, `groupSize > Count` та порожній список не змінюються.
- `groupSize <= 0` — `ArgumentOutOfRangeException`; список при цьому не змінюється.
- Вхід містить не більше 100 000 вузлів; значення можуть повторюватися.
- **Складність:** O(n) часу та O(1) додаткової пам'яті.
  Не починайте пошук кожної наступної групи з початку списку.

### `Program.cs` — заготовка без реалізації

```csharp
(int[] Values, int GroupSize)[] examples =
[
    ([1, 2, 3, 4, 5, 6, 7, 8], 3),
    ([1, 2, 3, 4, 5], 2),
    ([1, 2, 3], 1),
    ([1, 2], 3),
    ([], 2),
];

foreach (var example in examples)
{
    var values = new LinkedList<int>(example.Values);
    ReverseGroups(values, example.GroupSize);
    string result = values.Count == 0 ? "(порожньо)" : string.Join(' ', values);
    Console.WriteLine($"k = {example.GroupSize}: {result}");
}

static void ReverseGroups(LinkedList<int> values, int groupSize)
{
    throw new NotImplementedException();
}
```

### Очікуваний вивід після реалізації

```text
k = 3: 3 2 1 6 5 4 7 8
k = 2: 2 1 4 3 5
k = 1: 1 2 3
k = 3: 1 2
k = 2: (порожньо)
```

Додатково перевірте одну повну групу, повторювані значення, обхід через
`Previous` та некоректний `groupSize`. Збережіть посилання на вузли перед
викликом і переконайтеся, що після перестановки це ті самі об'єкти, їхні
значення збережені, а властивість `List` указує на початковий список.

## Що здати та як оцінюється робота

Здайте два консольні проєкти з обраною парою (**1 + 4**, **2 + 5** або **3 + 6**), результатами запуску
на наведених і додаткових граничних прикладах та коротким поясненням складності.
Для власної структури реалізуйте **весь** зазначений API, а не лише методи,
які викликаються у демонстрації.

| Обрана частина | Критерій | Бали |
|---|---|---|
| Реалізація, одне із завдань 1–3 | Базові операції та інваріанти структури | 1 |
| Реалізація, одне із завдань 1–3 | Усі додаткові методи та заявлена складність | 1 |
| Реалізація, одне із завдань 1–3 | Граничні випадки, винятки, перевірки з різними типами | 1 |
| Алгоритм для тієї самої структури: 4 до 1, 5 до 2 або 6 до 3 | Правильний результат на звичайних і граничних даних | 1 |
| Алгоритм для тієї самої структури: 4 до 1, 5 до 2 або 6 до 3 | Виконання обмежень і пояснення складності | 1 |
| **Разом за одну обрану пару** | | **5** |
