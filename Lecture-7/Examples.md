# Лекція 7 — Хеш-таблиці

## Зміст

1. [Навіщо хеш-таблиці](#1-навіщо-хеш-таблиці)
2. [Хеш-функція](#2-хеш-функція)
3. [Колізії: ланцюжки та відкрита адресація](#3-колізії-ланцюжки-та-відкрита-адресація)
4. [Коефіцієнт заповнення та рехешування](#4-коефіцієнт-заповнення-та-рехешування)
5. [Dictionary та HashSet](#5-dictionary-та-hashset)
6. [Власний хеш для структури](#6-власний-хеш-для-структури)
7. [Розбір задач](#7-розбір-задач)
8. [Підсумки](#8-підсумки)
9. [Питання для самоперевірки](#9-питання-для-самоперевірки)

## 1. Навіщо хеш-таблиці

Хеш-таблиця зберігає пари **ключ → значення** і дозволяє виконувати пошук, вставку та видалення в середньому за **O(1)**.

| Структура | Пошук | Вставка | Порядок ключів |
|-----------|-------|---------|----------------|
| Масив (несортований) | O(n) | O(1) | немає |
| `SortedDictionary` (дерево) | O(log n) | O(log n) | відсортовані |
| `Dictionary` (хеш) | O(1) в середньому, O(n) у гіршому | O(1) в середньому | немає |

Ідея: перетворити ключ на номер «кошика» (bucket) у масиві — `index = hash(key) % capacity`.

## 2. Хеш-функція

Хороша хеш-функція:
- **детермінована** — однаковий ключ завжди дає однаковий хеш;
- **швидка** — O(довжина ключа);
- **рівномірна** — розкидає ключі по кошиках без скупчень.

Для рядків часто використовують **поліноміальний хеш**: `h = s[0]·p^(n-1) + s[1]·p^(n-2) + ... + s[n-1]`.

> У .NET `string.GetHashCode()` **рандомізований для кожного процесу** — між запусками значення різні. Тому для друку використовуємо власну функцію.

### Приклад 1 — поліноміальний хеш рядка

```csharp
const ulong Capacity = 7; // кількість кошиків у таблиці
string[] words = ["cat", "dog", "act", "bird", "fish"];
foreach (string w in words) // індекс кошика — залишок від ділення хешу на розмір таблиці
    Console.WriteLine($"{w} -> hash={PolyHash(w)}, bucket={PolyHash(w) % Capacity}");
// Зверніть увагу: "cat" і "act" мають різні хеші — порядок символів важливий

// Поліноміальний хеш: h = h * P + символ (за модулем 2^64 через переповнення)
static ulong PolyHash(string s)
{
    const ulong P = 31; // невелике просте число
    ulong h = 0;
    foreach (char c in s) h = unchecked(h * P + c); // "зсуваємо" і додаємо символ
    return h;
}
```

**Приклад запуску:**
```
cat -> hash=98262, bucket=3
dog -> hash=99644, bucket=6
act -> hash=96402, bucket=5
bird -> hash=3024057, bucket=1
fish -> hash=3143256, bucket=4
```

## 3. Колізії: ланцюжки та відкрита адресація

**Колізія** — два різні ключі потрапили в один кошик. Уникнути їх неможливо (ключів більше, ніж кошиків), тому їх треба **обробляти**.

| Метод | Ідея | Плюси | Мінуси |
|-------|------|-------|--------|
| **Ланцюжки** (separate chaining) | кожен кошик — список елементів | просто, α може бути > 1 | додаткова пам'ять на вузли |
| **Відкрита адресація** (open addressing) | при зайнятій клітинці шукаємо наступну вільну (лінійне зондування: i+1, i+2, ...) | кеш-дружньо, без вказівників | кластеризація, складне видалення, α < 1 |

### Приклад 2 — хеш-таблиця з ланцюжками (з рехешуванням)

```csharp
using System.Globalization;

var m = new ChainedHashMap<string, int>();
string[] names = ["Anna", "Bohdan", "Iryna", "Oleh", "Taras", "Olena", "Maksym"];
int age = 20;
foreach (string n in names) { Console.WriteLine($"put {n}"); m.Put(n, age++); }
m.Put("Anna", 99); // оновлення існуючого ключа, розмір не змінюється

Console.WriteLine($"Anna = {(m.TryGet("Anna", out int v) ? v : -1)}");
Console.WriteLine($"Petro знайдено? {(m.TryGet("Petro", out _) ? "так" : "ні")}");
Console.WriteLine($"size = {m.Count}, load = {m.LoadFactor.ToString(CultureInfo.InvariantCulture)}");

class ChainedHashMap<TKey, TValue>(int capacity = 4) where TKey : notnull
{
    private const double MaxLoad = 0.75; // поріг коефіцієнта заповнення
    // Масив кошиків, кожен — зв'язний список пар (ключ, значення)
    private LinkedList<KeyValuePair<TKey, TValue>>[] _buckets = CreateBuckets(capacity);
    private readonly IEqualityComparer<TKey> _comparer = EqualityComparer<TKey>.Default;
    public int Count { get; private set; } // кількість збережених елементів
    public double LoadFactor => (double)Count / _buckets.Length;

    private static LinkedList<KeyValuePair<TKey, TValue>>[] CreateBuckets(int cap) =>
        Enumerable.Range(0, cap).Select(_ => new LinkedList<KeyValuePair<TKey, TValue>>()).ToArray();

    // Маскуємо знаковий біт, щоб індекс не був від'ємним
    private int IndexFor(TKey key, int cap) => (_comparer.GetHashCode(key) & int.MaxValue) % cap;

    public void Put(TKey key, TValue value)
    {
        var bucket = _buckets[IndexFor(key, _buckets.Length)];
        for (var node = bucket.First; node is not null; node = node.Next)
            if (_comparer.Equals(node.Value.Key, key)) { node.Value = new(key, value); return; } // оновлення
        bucket.AddLast(new KeyValuePair<TKey, TValue>(key, value)); // новий ключ — у кінець ланцюжка
        Count++;
        if (LoadFactor <= MaxLoad) return; // поріг не перевищено — рехешування не потрібне
        Console.WriteLine($"  rehash: {_buckets.Length} -> {_buckets.Length * 2}");
        Rehash();
    }

    // Повертає true і значення, якщо ключ знайдено
    public bool TryGet(TKey key, out TValue value)
    {
        foreach (var kv in _buckets[IndexFor(key, _buckets.Length)])
            if (_comparer.Equals(kv.Key, key)) { value = kv.Value; return true; }
        value = default!; return false;
    }

    private void Rehash()
    {
        // Створюємо вдвічі більший масив і переносимо всі елементи.
        // Індекси змінюються, бо змінився модуль (capacity)!
        var bigger = CreateBuckets(_buckets.Length * 2);
        foreach (var bucket in _buckets)
            foreach (var kv in bucket)
                bigger[IndexFor(kv.Key, bigger.Length)].AddLast(kv);
        _buckets = bigger;
    }
}
```

**Приклад запуску:**
```
put Anna
put Bohdan
put Iryna
put Oleh
  rehash: 4 -> 8
put Taras
put Olena
put Maksym
  rehash: 8 -> 16
Anna = 99
Petro знайдено? ні
size = 7, load = 0.4375
```

### Приклад 3 — відкрита адресація (лінійне зондування)

```csharp
var s = new LinearProbingSet<int>(7, k => (uint)k); // простий хеш для int — саме число
// 10 % 7 = 3, 17 % 7 = 3 (колізія!), 24 % 7 = 3 (ще одна), 5 % 7 = 5
foreach (int k in new[] { 10, 17, 24, 5 })
    Console.WriteLine($"insert {k} -> cell {s.Insert(k)}");
foreach (int k in new[] { 24, 31 }) Console.WriteLine($"contains {k}: {s.Contains(k).ToString().ToLower()}");

class LinearProbingSet<T>(int capacity, Func<T, uint> hash)
{
    private readonly T[] _keys = new T[capacity];
    private readonly bool[] _busy = new bool[capacity]; // стан клітинки: зайнята чи порожня
    private readonly EqualityComparer<T> _comparer = EqualityComparer<T>.Default;

    // Початкова клітинка — хеш за модулем розміру таблиці
    private int StartIndex(T key) => (int)(hash(key) % (uint)capacity);

    // Повертає індекс, куди потрапив ключ, або null, якщо таблиця повна
    public int? Insert(T key)
    {
        int start = StartIndex(key);
        for (int step = 0; step < capacity; step++)
        {
            int i = (start + step) % capacity;        // i, i+1, i+2, ... по колу
            if (!_busy[i]) { _keys[i] = key; _busy[i] = true; return i; } // вільна — займаємо
            if (_comparer.Equals(_keys[i], key)) return i; // вже є — нічого не робимо
        }
        return null;
    }

    public bool Contains(T key)
    {
        int start = StartIndex(key);
        for (int step = 0; step < capacity; step++)
        {
            int i = (start + step) % capacity;
            if (!_busy[i]) return false;               // дійшли до порожньої — ключа немає
            if (_comparer.Equals(_keys[i], key)) return true;
        }
        return false;
    }
}
```

**Приклад запуску:**
```
insert 10 -> cell 3
insert 17 -> cell 4
insert 24 -> cell 5
insert 5 -> cell 6
contains 24: true
contains 31: false
```

> Ключ 5 мав потрапити в клітинку 5, але її вже зайняв 24 — це і є **кластеризація**.

## 4. Коефіцієнт заповнення та рехешування

**Коефіцієнт заповнення** α = n / m (n — елементів, m — кошиків).

- Ланцюжки: середня довжина ланцюжка = α, тому пошук O(1 + α).
- Відкрита адресація: при α → 1 кількість зондувань різко зростає; зазвичай тримають α ≤ 0.5–0.7.

Коли α перевищує поріг, виконують **рехешування**: створюють більшу таблицю (зазвичай ×2) і перевставляють **усі** елементи. Одна операція коштує O(n), але амортизовано вставка залишається O(1) — як у `List<T>.Add`.

## 5. Dictionary та HashSet

### Приклад 4 — базові операції

```csharp
using System.Runtime.InteropServices;

var stock = new Dictionary<string, int>(); // товар -> кількість
stock["apple"] = 5;                          // індексатор створює або перезаписує ключ
stock.TryAdd("pear", 3);                     // TryAdd не перезаписує існуючий ключ
stock.Add("plum", 7);                        // Add кидає виняток, якщо ключ уже є
stock["apple"] += 2;                         // оновлення значення

// TryGetValue — один пошук замість ContainsKey + індексатор
if (stock.TryGetValue("pear", out int pear)) Console.WriteLine($"pear: {pear}");

Console.WriteLine($"ContainsKey(kiwi) = {stock.ContainsKey("kiwi")}");
// УВАГА: stock["kiwi"] для відсутнього ключа кине KeyNotFoundException!

// Посилання на значення: додає 0, якщо ключа немає, і дає змінити значення на місці
ref int grapes = ref CollectionsMarshal.GetValueRefOrAddDefault(stock, "grape", out bool existed);
grapes += 10;
Console.WriteLine($"grape = {stock["grape"]}, existed = {existed}");

stock.Remove("plum"); stock.Remove("grape");
Console.WriteLine($"apple = {stock["apple"]}, size = {stock.Count}");

// Заздалегідь виділити місце — менше рехешувань
Console.WriteLine($"capacity >= 100: {stock.EnsureCapacity(100) >= 100}");

var seen = new HashSet<int>();
foreach (int x in new[] { 3, 1, 3, 2, 1 })
    if (!seen.Add(x)) Console.WriteLine($"дублікат: {x}"); // Add повертає false, якщо елемент уже був
Console.WriteLine($"унікальних: {seen.Count}");
```

**Приклад запуску:**
```
pear: 3
ContainsKey(kiwi) = False
grape = 10, existed = False
apple = 7, size = 2
capacity >= 100: True
дублікат: 3
дублікат: 1
унікальних: 3
```

> Порядок обходу `Dictionary` **не гарантований** — не покладайтесь на нього. Потрібен порядок — використовуйте `SortedDictionary` або сортуйте результат.

## 6. Власний хеш для структури

Для власного типу ключа `Dictionary` потребує двох узгоджених речей: **`GetHashCode`** і **`Equals`**. `record struct` генерує обидва автоматично; вручну — через `IEquatable<T>` і `HashCode.Combine`.

### Приклад 5 — хеш для `Point`

```csharp
var cityAt = new Dictionary<Point, string>
{
    [new(0, 0)] = "Kyiv",
    [new(3, -2)] = "Lviv",
    [new(-1, 4)] = "Kharkiv",
};

var q = new Point(3, -2);
Console.WriteLine($"(3,-2): {(cityAt.TryGetValue(q, out var city) ? city : "немає")}");
Console.WriteLine($"(1,1) є? {(cityAt.ContainsKey(new(1, 1)) ? "так" : "ні")}");
Console.WriteLine($"hash(1,2) != hash(2,1): {new Point(1, 2).GetHashCode() != new Point(2, 1).GetHashCode()}");

// Найкоротше: `readonly record struct Point(int X, int Y);` — Equals і GetHashCode генеруються самі.
// Нижче — те саме вручну, щоб бачити, що саме потрібно словнику.
readonly struct Point(int x, int y) : IEquatable<Point>
{
    public int X => x; public int Y => y;
    // Рівність обов'язкова: хеш знаходить кошик, а Equals — конкретний ключ у ньому
    public bool Equals(Point other) => X == other.X && Y == other.Y;
    public override bool Equals(object? obj) => obj is Point p && Equals(p);

    // HashCode.Combine перемішує біти полів, тому (1,2) і (2,1) дають різні хеші
    // (значення рандомізоване між запусками, тому саме число не друкуємо)
    public override int GetHashCode() => HashCode.Combine(X, Y);
}
```

**Приклад запуску:**
```
(3,-2): Lviv
(1,1) є? ні
hash(1,2) != hash(2,1): True
```

## 7. Розбір задач

### Приклад 6 — частота слів

```csharp
using System.Runtime.InteropServices;

string text = "The cat and the dog. The DOG runs, and the cat sleeps!";
var freq = CountWords(text);

// Сортуємо: спершу за частотою (спадання), потім за алфавітом
foreach (var (w, c) in freq.OrderByDescending(kv => kv.Value).ThenBy(kv => kv.Key, StringComparer.Ordinal))
    Console.WriteLine($"{w}: {c}");

static Dictionary<string, int> CountWords(string text)
{
    var freq = new Dictionary<string, int>();
    foreach (string word in text.Split(' ', StringSplitOptions.RemoveEmptyEntries))
    {
        // Лишаємо тільки літери і приводимо до нижнього регістру
        string clean = string.Concat(word.Where(char.IsLetter).Select(char.ToLowerInvariant));
        if (clean.Length > 0)
            CollectionsMarshal.GetValueRefOrAddDefault(freq, clean, out _)++; // створить 0, потім ++
    }
    return freq; // O(n) в середньому
}
```

**Приклад запуску:**
```
the: 4
and: 2
cat: 2
dog: 2
runs: 1
sleeps: 1
```

### Приклад 7 — Two Sum

Задача: знайти індекси двох чисел, сума яких дорівнює `target`. Наївно — O(n²) (усі пари). З хеш-таблицею — **O(n)**: для кожного `x` перевіряємо, чи бачили вже `target - x`.

```csharp
int[] nums = [2, 7, 11, 15, -3, 4];
foreach (int target in new[] { 9, 1, 8, 100 })
{
    var (i, j) = TwoSum(nums, target);
    Console.WriteLine($"target={target} -> " + (i == -1 ? "немає" : $"[{i}, {j}] ({nums[i]} + {nums[j]})"));
}

// Повертає пару індексів або (-1, -1), якщо розв'язку немає
static (int, int) TwoSum(int[] nums, int target)
{
    var indexOf = new Dictionary<int, int>(); // значення -> індекс, де ми його бачили
    for (int i = 0; i < nums.Length; i++)
    {
        int need = target - nums[i];              // яке число доповнює поточне
        if (indexOf.TryGetValue(need, out int j))
            return (j, i);                        // знайшли пару за O(1)
        indexOf[nums[i]] = i;                     // запам'ятовуємо ПІСЛЯ перевірки,
                                                  // щоб не використати елемент двічі
    }
    return (-1, -1);
}
```

**Приклад запуску:**
```
target=9 -> [0, 1] (2 + 7)
target=1 -> [4, 5] (-3 + 4)
target=8 -> [2, 4] (11 + -3)
target=100 -> немає
```

## 8. Підсумки

- Хеш-таблиця = масив кошиків + хеш-функція; операції O(1) в середньому, O(n) у гіршому випадку.
- Колізії неминучі: **ланцюжки** (списки в кошиках) або **відкрита адресація** (пошук вільної клітинки).
- Коефіцієнт заповнення α керує швидкістю; при перевищенні порогу — **рехешування** (амортизовано O(1)).
- `Dictionary<TKey,TValue>` / `HashSet<T>` — готові хеш-таблиці .NET; порядок обходу не гарантовано.
- Для власного ключа потрібні узгоджені `GetHashCode` і `Equals` (або `record struct`).
- Типові застосування: підрахунок частот, пошук дублікатів, Two Sum, кешування.

## 9. Питання для самоперевірки

1. Чому пошук у хеш-таблиці у гіршому випадку O(n)? Коли це трапляється?
2. Чим відрізняються ланцюжки від відкритої адресації? Чому видалення при відкритій адресації складніше?
3. Що таке коефіцієнт заповнення і чому після рехешування змінюються індекси елементів?
4. Чому для перевірки наявності ключа краще `TryGetValue`/`ContainsKey`, а не індексатор?
5. Що станеться, якщо для власної структури перевизначити `GetHashCode`, але не `Equals` (або зробити їх неузгодженими)?
