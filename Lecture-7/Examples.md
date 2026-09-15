# Лекція 7 — Хеш-таблиці: теорія, реалізація, застосування

> Тривалість: ~190 хвилин (з трьома перервами).
> Мова прикладів: C# (.NET, `Nullable` увімкнено).
> Практичне використання API `Dictionary<TKey,TValue>` / `HashSet<T>` детально розібране в Лекції 3 (колекції .NET). Тут ми зосереджуємося на **тому, як хеш-таблиці влаштовані всередині**, чому вони працюють швидко і коли перестають працювати.

---

## Зміст

| № | Розділ | Хв |
|---|--------|----|
| 1 | [Від прямої адресації до хешування](#1-від-прямої-адресації-до-хешування) | 10 |
| 2 | [Хеш-функції](#2-хеш-функції) | 25 |
| 3 | [Колізії та парадокс днів народження](#3-колізії-та-парадокс-днів-народження) | 10 |
| — | ☕ Перерва 1 | 5 |
| 4 | [Метод ланцюжків (separate chaining)](#4-метод-ланцюжків-separate-chaining) | 20 |
| 5 | [Відкрита адресація](#5-відкрита-адресація) | 25 |
| 6 | [Коефіцієнт заповнення та аналіз складності](#6-коефіцієнт-заповнення-та-аналіз-складності) | 10 |
| — | ☕ Перерва 2 | 5 |
| 7 | [Власний HashMap<TKey,TValue> крок за кроком](#7-власний-hashmaptkeytvalue-крок-за-кроком) | 15 |
| 8 | [Dictionary у .NET: внутрішня будова та контракт GetHashCode/Equals](#8-dictionary-у-net-внутрішня-будова-та-контракт-gethashcodeequals) | 20 |
| 9 | [HashSet<T> і операції над множинами](#9-hashsett-і-операції-над-множинами) | 5 |
| — | ☕ Перерва 3 | 5 |
| 10 | [Застосування: класичні задачі](#10-застосування-класичні-задачі) | 30 |
| 11 | [Досконале хешування та хешування зозулі](#11-досконале-хешування-та-хешування-зозулі) | 8 |
| 12 | [Порівняння: хеш-таблиця vs збалансоване дерево vs відсортований масив](#12-порівняння-хеш-таблиця-vs-збалансоване-дерево-vs-відсортований-масив) | 4 |
| 13 | [Типові помилки та підсумки](#13-типові-помилки-та-підсумки) | 3 |
| 14 | [Питання для самоперевірки та практичні завдання](#14-питання-для-самоперевірки-та-практичні-завдання) | 5 |
| | **Разом** | **~190** |

---

## 1. Від прямої адресації до хешування

### 1.1. Задача словника

**Словник** (асоціативний масив, map) — абстрактний тип даних, що зберігає пари **ключ → значення** і підтримує три операції:

| Операція | Опис |
|----------|------|
| `Insert(key, value)` | додати або замінити значення за ключем |
| `Search(key)` | знайти значення за ключем |
| `Delete(key)` | видалити пару за ключем |

Як можна реалізувати словник, і яка вартість операцій?

| Реалізація | Search | Insert | Delete | Примітка |
|------------|--------|--------|--------|----------|
| Несортований масив / список | O(n) | O(1) | O(n) | лінійний пошук |
| Відсортований масив | O(log n) | O(n) | O(n) | бінарний пошук, зсув елементів |
| Збалансоване дерево (AVL, червоно-чорне) | O(log n) | O(log n) | O(log n) | ключі впорядковані |
| **Хеш-таблиця** | **O(1)** в середньому | **O(1)** амортизовано | **O(1)** в середньому | порядку немає |

### 1.2. Таблиця прямої адресації

Найпростіший випадок: ключі — цілі числа з **маленького** діапазону `0..U-1`. Тоді беремо масив розміру `U` і використовуємо ключ **як індекс**.

```
Ключі: {2, 5, 7}, U = 10

індекс:  0     1     2     3     4     5     6     7     8     9
       ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
       │  –  │  –  │ "B" │  –  │  –  │ "E" │  –  │ "G" │  –  │  –  │
       └─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘
                      ▲                 ▲           ▲
                   key=2             key=5       key=7
```

Усі операції — **O(1) у гіршому випадку**, бо це просто звернення до масиву.

```csharp
// Таблиця прямої адресації: ключ = індекс масиву.
// Підходить, коли ключі — невеликі невід'ємні цілі числа.
var table = new DirectAddressTable<string>(universeSize: 10);
table.Insert(2, "B");
table.Insert(5, "E");
table.Insert(7, "G");

Console.WriteLine($"Search(5) = {table.Search(5) ?? "null"}");
Console.WriteLine($"Search(3) = {table.Search(3) ?? "null"}");
table.Delete(5);
Console.WriteLine($"Після Delete(5): Search(5) = {table.Search(5) ?? "null"}");
Console.WriteLine($"Пам'ять: {table.Capacity} комірок на {table.Count} елементи(ів)");

public sealed class DirectAddressTable<TValue> where TValue : class
{
    private readonly TValue?[] _slots; // одна комірка на кожен можливий ключ
    private int _count;

    public DirectAddressTable(int universeSize) => _slots = new TValue?[universeSize];

    public int Capacity => _slots.Length;
    public int Count => _count;

    public void Insert(int key, TValue value)
    {
        if (_slots[key] is null) _count++; // рахуємо лише нові ключі
        _slots[key] = value;               // O(1): прямий доступ за індексом
    }

    public TValue? Search(int key) => _slots[key]; // O(1)

    public void Delete(int key)
    {
        if (_slots[key] is not null) _count--;
        _slots[key] = null; // O(1)
    }
}
```

**Приклад запуску:**
```
Search(5) = E
Search(3) = null
Після Delete(5): Search(5) = null
Пам'ять: 10 комірок на 2 елементи(ів)
```

### 1.3. Проблема: великий універсум ключів

Пряма адресація ламається, коли **універсум U** (множина всіх можливих ключів) величезний:

| Тип ключа | Розмір універсуму U | Масив прямої адресації |
|-----------|---------------------|------------------------|
| `byte` | 256 | 256 комірок — ок |
| `short` | 65 536 | ок |
| `int` | 4 294 967 296 | ~16 ГБ лише на посилання — ні |
| `long` | 1.8 · 10¹⁹ | неможливо |
| `string` (до 10 латинських літер) | 26¹⁰ ≈ 1.4 · 10¹⁴ | неможливо |

А **реально використовуваних** ключів `n` зазвичай мало (тисячі, мільйони). Пам'ять `O(U)` при `n ≪ U` — марнотратство.

### 1.4. Ідея хешування

Замість того, щоб використовувати ключ як індекс, **обчислюємо** індекс за допомогою **хеш-функції**:

```
            h : U → {0, 1, ..., m-1}

   ключ ──► [ хеш-функція h ] ──► індекс кошика (bucket) 0..m-1

"apple" ──► h ──► 3
"pear"  ──► h ──► 0
"plum"  ──► h ──► 3   ← КОЛІЗІЯ: два ключі в один кошик!

       ┌───────────┐
    0  │ "pear"    │
    1  │           │
    2  │           │
    3  │ "apple" ? "plum" ?
    4  │           │
       └───────────┘   m = 5
```

Тепер пам'ять — `O(m + n)`, де `m` порядку `n`. Ціна: **колізії** — різні ключі можуть потрапити в один кошик (за принципом Діріхле це неминуче, якщо `|U| > m`). Уся подальша теорія — це:

1. як обрати **хорошу хеш-функцію**, щоб колізій було мало;
2. як **розв'язувати колізії**, коли вони все ж трапляються;
3. як **підтримувати розмір таблиці**, щоб колізій не ставало забагато.

> **Термінологія.** *Кошик* (bucket, slot) — комірка масиву таблиці. *Хеш-код* — число, яке повертає хеш-функція (часто 32-бітне). *Індекс* — хеш-код, зведений до діапазону `0..m-1`.

---

## 2. Хеш-функції

### 2.1. Двоетапна схема

На практиці хешування ділиться на два етапи:

```
  ключ (будь-який тип)
        │
        ▼  етап 1: хеш-код  (GetHashCode, FNV-1a, поліном…)
  32/64-бітне ціле число
        │
        ▼  етап 2: стискання (compression) — % m або & (m-1)
  індекс 0..m-1
```

Етап 1 залежить від **типу ключа** (рядок, структура, число). Етап 2 залежить від **розміру таблиці**. У .NET етап 1 — це `GetHashCode()`, етап 2 виконує сам `Dictionary`.

### 2.2. Властивості хорошої хеш-функції

| Властивість | Що означає | Чому важливо |
|-------------|-----------|--------------|
| **Детермінованість** | однаковий ключ → однаковий хеш (у межах процесу) | інакше ключ «загубиться» |
| **Узгодженість з Equals** | `a.Equals(b)` ⇒ `h(a) == h(b)` | інакше рівні ключі потраплять у різні кошики |
| **Рівномірність** | ключі розподіляються по кошиках приблизно порівну | довжини ланцюжків/проб малі |
| **Лавинний ефект** (avalanche) | зміна 1 біта входу змінює ~50% бітів виходу | схожі ключі (`"key1"`, `"key2"`) не скупчуються |
| **Швидкість** | O(довжина ключа), мала константа | хеш рахується на кожну операцію |

> Криптографічна стійкість (неможливість знайти колізію) для хеш-таблиць **зазвичай не потрібна** і занадто повільна. Але див. [HashDoS](#83-рандомізоване-хешування-рядків-і-hashdos).

### 2.3. Метод ділення

```
h(k) = k mod m
```

Просто і швидко. Але якість сильно залежить від `m`:

- **`m = 2^p`** — погано для «сирих» ключів: `k mod 2^p` бере лише **молодші p бітів**, решта бітів ігнорується.
- **`m = 10^p`** — те саме для десяткових цифр.
- **`m` — просте число, далеке від степенів 2** — хороший вибір: у результаті «беруть участь» усі біти ключа.

Продемонструємо: ключі — числа, кратні 8 (типово для адрес пам'яті або ідентифікаторів з кроком).

```csharp
// Метод ділення: h(k) = k mod m.
// Ключі кратні 8 (0, 8, 16, ...) — типовий "неприємний" шаблон.
int[] keys = Enumerable.Range(0, 32).Select(i => i * 8).ToArray();

PrintDistribution("m = 16 (степінь двійки)", keys, 16);
PrintDistribution("m = 17 (просте число)   ", keys, 17);

static void PrintDistribution(string title, int[] keys, int m)
{
    var counts = new int[m]; // кількість ключів у кожному кошику
    foreach (int k in keys)
        counts[k % m]++;     // метод ділення

    int used = counts.Count(c => c > 0);
    Console.WriteLine($"{title}: зайнято кошиків {used}/{m}, макс. у кошику {counts.Max()}");
    Console.WriteLine("  " + string.Join(" ", counts));
}
```

**Приклад запуску:**
```
m = 16 (степінь двійки): зайнято кошиків 2/16, макс. у кошику 16
  16 0 0 0 0 0 0 0 16 0 0 0 0 0 0 0
m = 17 (просте число)   : зайнято кошиків 17/17, макс. у кошику 2
  2 1 2 2 2 2 2 2 2 1 2 2 2 2 2 2 2
```

При `m = 16` усі ключі кратні 8 потрапляють лише в кошики 0 і 8 — молодші 3 біти ключів завжди нульові. Просте `m = 17` розподіляє ідеально рівномірно.

### 2.4. Метод множення (Кнута)

```
h(k) = ⌊ m · frac(k · A) ⌋,   0 < A < 1
```

де `frac(x)` — дробова частина. Кнут рекомендує `A ≈ (√5 − 1)/2 ≈ 0.6180339887` (золотий переріз). Перевага: **розмір `m` не критичний**, можна брати `m = 2^p`.

Цілочисельна версія (**Fibonacci hashing**) для 32-бітних ключів і `m = 2^p`:

```
h(k) = (k · 2654435769) >> (32 − p)       // 2654435769 ≈ 2^32 · 0.618…
```

Множення «перемішує» біти ключа у **старші** біти добутку, і ми беремо саме старші `p` бітів.

```csharp
// Порівняння: маскування молодших бітів vs Fibonacci hashing.
// Ключі кратні 8 — ті самі "погані" ключі, що й у попередньому прикладі.
const int P = 4;           // m = 2^4 = 16 кошиків
const int M = 1 << P;
uint[] keys = Enumerable.Range(0, 32).Select(i => (uint)(i * 8)).ToArray();

var maskCounts = new int[M];
var fibCounts = new int[M];
foreach (uint k in keys)
{
    maskCounts[k & (M - 1)]++;  // беремо молодші 4 біти — погано
    fibCounts[FibonacciHash(k, P)]++; // беремо старші 4 біти добутку — добре
}

Console.WriteLine("Маска   : " + string.Join(" ", maskCounts));
Console.WriteLine("Фібоначчі: " + string.Join(" ", fibCounts));

// Множимо на ⌊2^32 / φ⌋ і беремо старші p бітів 32-бітного результату.
static uint FibonacciHash(uint key, int p)
{
    const uint GoldenRatio32 = 2654435769u;
    uint product = unchecked(key * GoldenRatio32); // переповнення — це нормально (mod 2^32)
    return product >> (32 - p);
}
```

**Приклад запуску:**
```
Маска   : 16 0 0 0 0 0 0 0 16 0 0 0 0 0 0 0
Фібоначчі: 2 1 1 1 2 2 2 4 2 2 2 2 2 2 2 3
```

### 2.5. Переповнення цілих чисел і `unchecked`

Майже всі хеш-функції **навмисно** переповнюють цілі числа: множення за модулем `2^32` або `2^64` — це дешевий спосіб перемішати біти.

У C# арифметика за замовчуванням `unchecked` (переповнення тихо «загортається»), **але**:

- якщо проєкт зібрано з `<CheckForOverflowUnderflow>true</CheckForOverflowUnderflow>` — буде `OverflowException`;
- константні вирази перевіряються **під час компіляції** (помилка CS0220).

Тому в хеш-функціях **явно** пишемо `unchecked(...)` — це і документація намірів, і захист від налаштувань проєкту.

```csharp
// Демонстрація поведінки checked / unchecked при переповненні.
int big = int.MaxValue;

int wrapped = unchecked(big + 1); // загортається до int.MinValue
Console.WriteLine($"unchecked(int.MaxValue + 1) = {wrapped}");

try
{
    int boom = checked(big + 1); // у checked-контексті — виняток
    Console.WriteLine(boom);
}
catch (OverflowException)
{
    Console.WriteLine("checked(int.MaxValue + 1) → OverflowException");
}

// Типова хеш-функція в checked-контексті "впаде" на першому ж довгому рядку:
Console.WriteLine($"Хеш у unchecked: {HashUnchecked("overflow-me-please")}");
try
{
    Console.WriteLine(checked(HashChecked("overflow-me-please")));
}
catch (OverflowException)
{
    Console.WriteLine("Хеш у checked: OverflowException");
}

static uint HashUnchecked(string s)
{
    uint h = 17;
    foreach (char c in s) h = unchecked(h * 31 + c); // явно дозволяємо переповнення
    return h;
}

static uint HashChecked(string s)
{
    uint h = 17;
    foreach (char c in s) h = checked(h * 31 + c); // так робити НЕ треба
    return h;
}
```

**Приклад запуску:**
```
unchecked(int.MaxValue + 1) = -2147483648
checked(int.MaxValue + 1) → OverflowException
Хеш у unchecked: 1572204421
Хеш у checked: OverflowException
```

### 2.6. Поліноміальний хеш рядків

Рядок `s = s₀ s₁ … s_{n−1}` трактуємо як число в системі числення з основою `B`:

```
h(s) = s₀·B^(n−1) + s₁·B^(n−2) + … + s_{n−1}·B⁰   (mod M)
```

Обчислюється **схемою Горнера** за O(n): `h = h·B + sᵢ`.

- `B` — зазвичай невелике просте (31, 131) або випадкове.
- `M` — велике просте (`10⁹+7`) або неявно `2^64` через переповнення `ulong`.
- **Порядок символів важливий**: `"ab" ≠ "ba"` (на відміну від суми кодів символів).

```csharp
// Поліноміальний хеш: порівняння з "наївною" сумою кодів символів.
string[] words = ["listen", "silent", "enlist", "google", "gogole"];

Console.WriteLine($"{"слово",-8} {"сума",6} {"поліном mod 1e9+7",18}");
foreach (string w in words)
    Console.WriteLine($"{w,-8} {SumHash(w),6} {PolyHash(w),18}");

// Погана хеш-функція: будь-яка перестановка літер дає однаковий хеш.
static int SumHash(string s)
{
    int sum = 0;
    foreach (char c in s) sum += c;
    return sum;
}

// Поліноміальний хеш за модулем простого числа (схема Горнера).
static long PolyHash(string s)
{
    const long B = 131;           // основа
    const long Mod = 1_000_000_007; // велике просте
    long h = 0;
    foreach (char c in s)
        h = (h * B + c) % Mod;    // h·B < 1.4e11 — вміщається в long, переповнення немає
    return h;
}
```

**Приклад запуску:**
```
слово      сума  поліном mod 1e9+7
listen      655          767879116
silent      655          808311916
enlist      655          168029472
google      637          628217175
gogole      637          610369735
```

Сума кодів дає однакові значення для всіх анаграм — катастрофа для таблиці зі словами. Поліном розрізняє порядок.

### 2.7. FNV-1a

**FNV-1a** (Fowler–Noll–Vo) — проста і популярна некриптографічна хеш-функція з хорошим лавинним ефектом для коротких ключів:

```
hash = offset_basis
для кожного байта b:
    hash = hash XOR b
    hash = hash × FNV_prime
```

| Варіант | offset_basis | FNV_prime |
|---------|--------------|-----------|
| 32-біт | 2166136261 | 16777619 |
| 64-біт | 14695981039346656037 | 1099511628211 |

### 2.8. djb2

**djb2** (Daniel J. Bernstein): `hash = hash * 33 + c`, початкове значення 5381. Множення на 33 = `(hash << 5) + hash` — дуже швидко.

```csharp
using System.Text;

// Три хеш-функції для рядків: FNV-1a (32), djb2, поліноміальна (B=31).
string[] inputs = ["", "a", "b", "hello", "hellp"];

Console.WriteLine($"{"вхід",-8} {"FNV-1a",10} {"djb2",10} {"poly31",10}");
foreach (string s in inputs)
    Console.WriteLine($"{Quote(s),-8} {Fnv1a32(s),10:X8} {Djb2(s),10:X8} {Poly31(s),10:X8}");

static string Quote(string s) => $"\"{s}\"";

// FNV-1a над UTF-8 байтами рядка.
static uint Fnv1a32(string s)
{
    const uint OffsetBasis = 2166136261;
    const uint Prime = 16777619;
    uint hash = OffsetBasis;
    foreach (byte b in Encoding.UTF8.GetBytes(s))
    {
        hash ^= b;                        // спочатку XOR ...
        hash = unchecked(hash * Prime);   // ... потім множення (у FNV-1 — навпаки)
    }
    return hash;
}

// djb2: hash * 33 + c, записано через зсув.
static uint Djb2(string s)
{
    uint hash = 5381;
    foreach (char c in s)
        hash = unchecked((hash << 5) + hash + c); // (hash << 5) + hash == hash * 33
    return hash;
}

// Класичний Java-подібний поліноміальний хеш: h = 31*h + c.
static uint Poly31(string s)
{
    uint hash = 0;
    foreach (char c in s)
        hash = unchecked(hash * 31 + c);
    return hash;
}
```

**Приклад запуску:**
```
вхід         FNV-1a       djb2     poly31
""         811C9DC5   00001505   00000000
"a"        E40C292C   0002B606   00000061
"b"        E70C2DE5   0002B607   00000062
"hello"    4F9F2CAB   0F923099   05E918D2
"hellp"    5C9F4122   0F92309A   05E918D3
```

Зверніть увагу на `"hello"` vs `"hellp"` (різниця в останньому символі на 1): у `djb2` і `poly31` відрізняються лише **молодші** біти — лавинний ефект слабкий; у FNV-1a різниця трохи більша, але теж неідеальна, бо зміна в останньому байті проходить лише одне множення. Тому якісні таблиці додатково **перемішують** хеш перед стисканням.

### 2.9. Вимірювання лавинного ефекту

Лавинний ефект вимірюють так: змінюємо **один біт** входу і рахуємо, скільки бітів виходу змінилося. Ідеал — 50% (16 з 32).

Для цілих ключів використаємо **фіналізатор** (finalizer, «mixer») із MurmurHash3 — стандартний спосіб «дотиснути» лавинність:

```csharp
using System.Numerics;

// Лавинний ефект: скільки бітів виходу змінюється при зміні 1 біта входу.
// Порівнюємо тотожну функцію, множення на просте та фіналізатор MurmurHash3.
uint[] samples = [1, 42, 1000, 123456, 0xDEADBEEF];

Report("identity    ", k => k);
Report("k * 31      ", k => unchecked(k * 31));
Report("murmur fmix ", Fmix32);

void Report(string name, Func<uint, uint> hash)
{
    long totalFlipped = 0;
    int experiments = 0;
    foreach (uint key in samples)
    {
        uint baseline = hash(key);
        for (int bit = 0; bit < 32; bit++)
        {
            uint flipped = hash(key ^ (1u << bit));       // інвертуємо один біт входу
            totalFlipped += BitOperations.PopCount(baseline ^ flipped); // рахуємо різні біти виходу
            experiments++;
        }
    }
    double avg = (double)totalFlipped / experiments;
    Console.WriteLine($"{name}: у середньому змінюється {avg:F2} з 32 бітів ({avg / 32:P0})");
}

// Фіналізатор MurmurHash3: xor-shift + множення, двічі.
static uint Fmix32(uint h)
{
    h ^= h >> 16;
    h = unchecked(h * 0x85EBCA6B);
    h ^= h >> 13;
    h = unchecked(h * 0xC2B2AE35);
    h ^= h >> 16;
    return h;
}
```

**Приклад запуску:**
```
identity    : у середньому змінюється 1.00 з 32 бітів (3%)
k * 31      : у середньому змінюється 3.87 з 32 бітів (12%)
murmur fmix : у середньому змінюється 15.84 з 32 бітів (49%)
```

### 2.10. Стискання: `% простого` чи `& (2^p − 1)`?

| Підхід | Операція | Швидкість | Вимоги до хешу |
|--------|----------|-----------|----------------|
| Модуль простого | `(hash & 0x7FFFFFFF) % m` | ділення — повільніше (~20–40 тактів CPU) | терпимий до слабкого хешу: використовує всі біти |
| Маска степеня 2 | `hash & (m − 1)` | одна інструкція AND | потрібен **добре перемішаний** хеш, бо беруться лише молодші біти |

- .NET `Dictionary` використовує **прості розміри** і `%` (з оптимізацією через «fastmod» — множення замість ділення на 64-бітних платформах).
- Java `HashMap`, Rust `hashbrown`, Go `map` — **степені двійки** + перемішування хешу (`h ^ (h >>> 16)` у Java).

> **Чому `& 0x7FFFFFFF`?** `GetHashCode()` може бути від'ємним, а `%` у C# для від'ємних чисел повертає від'ємний залишок: `-7 % 5 == -2`. Такий індекс кине `IndexOutOfRangeException`. Альтернатива — `(uint)hash % (uint)m`.

```csharp
// Від'ємний хеш-код і стискання до індексу.
int hash = -7;
int m = 5;

Console.WriteLine($"hash % m                 = {hash % m}   ← від'ємний індекс!");
Console.WriteLine($"(hash & 0x7FFFFFFF) % m  = {(hash & 0x7FFFFFFF) % m}");
Console.WriteLine($"(uint)hash % (uint)m     = {(uint)hash % (uint)m}");
Console.WriteLine($"Math.Abs(hash) % m       = {Math.Abs(hash) % m}");

try
{
    Console.WriteLine(Math.Abs(int.MinValue)); // |int.MinValue| не вміщається в int
}
catch (OverflowException)
{
    Console.WriteLine("Math.Abs(int.MinValue) → OverflowException (тому Abs — погана ідея)");
}
```

**Приклад запуску:**
```
hash % m                 = -2   ← від'ємний індекс!
(hash & 0x7FFFFFFF) % m  = 1
(uint)hash % (uint)m     = 4
Math.Abs(hash) % m       = 2
Math.Abs(int.MinValue) → OverflowException (тому Abs — погана ідея)
```

### Типові помилки (хеш-функції)

- ❌ Сума або XOR кодів символів — анаграми дають однаковий хеш.
- ❌ `hash % m` без обробки знаку → від'ємний індекс.
- ❌ `Math.Abs(hash) % m` → `OverflowException` для `int.MinValue`.
- ❌ `m = 2^p` разом зі слабким хешем (наприклад, тотожним для `int`) → використовуються лише молодші біти.
- ❌ Хеш, що залежить від **змінних** полів об'єкта (див. [мутабельні ключі](#88-баг-мутабельного-ключа)).
- ❌ Хеш від `DateTime.Now`, `Random`, адреси в пам'яті, що може змінитися, — порушення детермінованості.
- ❌ Зберігання `string.GetHashCode()` у файл/БД — у .NET Core він **інший у кожному процесі**.

### Міні-вправа 2.1

Чому хеш `h(s) = s.Length` — погана хеш-функція для таблиці англійських слів, хоча вона детермінована і швидка?

<details>
<summary>Розв'язок</summary>

Вона дуже **нерівномірна**: довжини слів зосереджені в діапазоні 2–12, тож усі слова потраплять у ~10 кошиків, незалежно від `m`. Пошук деградує до O(n / 10) = O(n). Також немає лавинного ефекту: `"cat"` і `"dog"` мають однаковий хеш.
</details>

### Міні-вправа 2.2

Напишіть 64-бітний FNV-1a для рядка (працюйте з `char`, розбиваючи його на два байти) і обчисліть хеш `"abc"`.

<details>
<summary>Розв'язок</summary>

```csharp
// 64-бітний FNV-1a над UTF-16 кодовими одиницями (по 2 байти на char).
Console.WriteLine($"{Fnv1a64("abc"):X16}");
Console.WriteLine($"{Fnv1a64("abd"):X16}");

static ulong Fnv1a64(string s)
{
    const ulong OffsetBasis = 14695981039346656037;
    const ulong Prime = 1099511628211;
    ulong hash = OffsetBasis;
    foreach (char c in s)
    {
        hash ^= (byte)c;                     // молодший байт
        hash = unchecked(hash * Prime);
        hash ^= (byte)(c >> 8);              // старший байт
        hash = unchecked(hash * Prime);
    }
    return hash;
}
```

**Приклад запуску:**
```
CEC64E155111225D
CEBC1C15510878E2
```
</details>

---

## 3. Колізії та парадокс днів народження

### 3.1. Колізії неминучі

**Колізія** — ситуація `k₁ ≠ k₂`, але `h(k₁) = h(k₂)`. За **принципом Діріхле**: якщо `|U| > m`, то колізії **гарантовано існують** для будь-якої функції `h`. Питання лише — **як часто** вони трапляються на реальних даних.

### 3.2. Парадокс днів народження

Скільки людей треба зібрати, щоб імовірність збігу днів народження перевищила 50%? Інтуїція каже «~183», правильна відповідь — **23**.

Імовірність, що `n` ключів у `m` кошиках **не мають жодної колізії**:

```
P(без колізій) = 1 · (1 − 1/m) · (1 − 2/m) · … · (1 − (n−1)/m)
               ≈ e^(−n(n−1) / 2m)
```

Звідси 50% колізії досягається при `n ≈ 1.1774 · √m`. **Висновок для хеш-таблиць: колізії з'являються дуже рано — вже при заповненні ~√m кошиків.**

```csharp
// Парадокс днів народження: точна ймовірність хоча б однієї колізії
// для n ключів у m кошиках, та кількість ключів, що дає 50%.
int[] sizes = [365, 1_000, 65_536, 1_000_000];

Console.WriteLine($"{"m",10} {"n для 50%",10} {"1.1774·√m",10}");
foreach (int m in sizes)
{
    int n = KeysForHalfCollisionChance(m);
    Console.WriteLine($"{m,10} {n,10} {1.1774 * Math.Sqrt(m),10:F1}");
}

Console.WriteLine();
Console.WriteLine("m = 365 (дні року):");
foreach (int n in new[] { 10, 23, 30, 50, 70 })
    Console.WriteLine($"  n = {n,2}: P(колізія) = {CollisionProbability(n, 365):P1}");

// Точна ймовірність: 1 − ∏ (1 − i/m) для i = 0..n−1.
static double CollisionProbability(int n, int m)
{
    double noCollision = 1.0;
    for (int i = 0; i < n; i++)
        noCollision *= (m - i) / (double)m;
    return 1.0 - noCollision;
}

// Найменше n, для якого P(колізія) ≥ 0.5.
static int KeysForHalfCollisionChance(int m)
{
    double noCollision = 1.0;
    int n = 0;
    while (1.0 - noCollision < 0.5)
    {
        noCollision *= (m - n) / (double)m;
        n++;
    }
    return n;
}
```

**Приклад запуску:**
```
         m  n для 50%  1.1774·√m
       365         23       22.5
      1000         38       37.2
     65536        302      301.4
   1000000       1178     1177.4

m = 365 (дні року):
  n = 10: P(колізія) = 11.7%
  n = 23: P(колізія) = 50.7%
  n = 30: P(колізія) = 70.6%
  n = 50: P(колізія) = 97.0%
  n = 70: P(колізія) = 99.9%
```

### 3.3. Скільки колізій очікувати?

Для рівномірного хешування `n` ключів у `m` кошиків:

- очікувана кількість **пар, що колізують**: `n(n−1) / 2m`;
- очікувана кількість **порожніх кошиків**: `m · (1 − 1/m)ⁿ ≈ m · e^(−n/m)`;
- очікувана **довжина найдовшого ланцюжка** при `n = m`: `Θ(log n / log log n)` — росте, але дуже повільно.

```csharp
// Симуляція: кидаємо n = m ключів у m кошиків детермінованим генератором
// і порівнюємо з теорією (частка порожніх ≈ 1/e ≈ 36.8%).
const int M = 10_000;
var rng = new Random(Seed: 12345);  // фіксоване зерно → відтворюваний результат
var counts = new int[M];

for (int i = 0; i < M; i++)
    counts[rng.Next(M)]++;          // "ідеальна" хеш-функція — випадковий кошик

int empty = counts.Count(c => c == 0);
int maxChain = counts.Max();
Console.WriteLine($"Порожніх кошиків: {empty} ({(double)empty / M:P1}), теорія ≈ {Math.Exp(-1):P1}");
Console.WriteLine($"Найдовший ланцюжок: {maxChain}");

// Гістограма довжин ланцюжків і теоретичний розподіл Пуассона з λ = 1.
Console.WriteLine("довжина | кошиків | Пуассон(λ=1)");
for (int len = 0; len <= maxChain; len++)
{
    int actual = counts.Count(c => c == len);
    double poisson = M * Math.Exp(-1) / Factorial(len);
    Console.WriteLine($"{len,7} | {actual,7} | {poisson,8:F0}");
}

static double Factorial(int k) => k <= 1 ? 1 : k * Factorial(k - 1);
```

**Приклад запуску:**
```
Порожніх кошиків: 3677 (36.8%), теорія ≈ 36.8%
Найдовший ланцюжок: 6
довжина | кошиків | Пуассон(λ=1)
      0 |    3677 |     3679
      1 |    3717 |     3679
      2 |    1786 |     1839
      3 |     613 |      613
      4 |     170 |      153
      5 |      30 |       31
      6 |       7 |        5
```

Розподіл довжин ланцюжків добре описується **розподілом Пуассона** з λ = n/m.

### Міні-вправа 3.1

32-бітні хеш-коди (`m = 2³²`). Скільки різних об'єктів треба, щоб з імовірністю ~50% у двох із них збігся `GetHashCode()`?

<details>
<summary>Розв'язок</summary>

`n ≈ 1.1774 · √(2³²) = 1.1774 · 65536 ≈ 77 163`. Тобто **вже ~77 тисяч об'єктів** майже напевно мають збіг хеш-коду. Тому хеш-таблиця **завжди** мусить перевіряти `Equals`, а не лише порівнювати хеші.
</details>

---

## ☕ Перерва 1 (5 хв)

---


## 4. Метод ланцюжків (separate chaining)

### 4.1. Ідея

Кожен кошик зберігає **список** (ланцюжок) усіх пар, чиї ключі потрапили в цей кошик.

```
m = 5,  h(k) = k mod 5,  вставляємо 12, 7, 22, 3, 17, 10

 кошик
 ┌───┐
 │ 0 │──► [10]
 ├───┤
 │ 1 │──► ∅
 ├───┤
 │ 2 │──► [17] ──► [22] ──► [7] ──► [12]     ← нові елементи додаємо на початок
 ├───┤
 │ 3 │──► [3]
 ├───┤
 │ 4 │──► ∅
 └───┘
```

| Операція | Алгоритм | Час |
|----------|----------|-----|
| Insert | обчислити кошик, пройти ланцюжок (перевірка дубліката), додати вузол | O(1 + α) |
| Search | обчислити кошик, пройти ланцюжок, порівнюючи `Equals` | O(1 + α) |
| Delete | знайти вузол і «вирізати» його з ланцюжка | O(1 + α) |

де **α = n / m** — коефіцієнт заповнення (load factor). Якщо підтримувати α ≤ const (через рехешування), усі операції — **O(1) в середньому**.

**Переваги:** просто, видалення тривіальне, α може бути > 1, деградує плавно.
**Недоліки:** окрема алокація на кожен вузол, погана локальність кешу (перехід за вказівниками).

### 4.2. Реалізація: базова версія

Почнемо з мінімальної таблиці рядок → число, щоб побачити структуру ланцюжків.

```csharp
// Мінімальна хеш-таблиця з ланцюжками (без рехешування) — для візуалізації.
var table = new TinyChainedTable(bucketCount: 5);
foreach (string word in new[] { "apple", "pear", "plum", "kiwi", "fig", "lime", "date" })
    table.Add(word, word.Length);

table.Print();
Console.WriteLine($"plum → {table.Find("plum")}");
Console.WriteLine($"mango → {table.Find("mango")?.ToString() ?? "не знайдено"}");

public sealed class TinyChainedTable(int bucketCount)
{
    // Вузол однозв'язного списку.
    private sealed class Node(string key, int value, Node? next)
    {
        public string Key { get; } = key;
        public int Value { get; set; } = value;
        public Node? Next { get; set; } = next;
    }

    private readonly Node?[] _buckets = new Node?[bucketCount];

    public void Add(string key, int value)
    {
        int index = IndexFor(key);
        for (Node? n = _buckets[index]; n is not null; n = n.Next)
            if (n.Key == key) { n.Value = value; return; } // ключ є — оновлюємо

        _buckets[index] = new Node(key, value, _buckets[index]); // вставка на початок — O(1)
    }

    public int? Find(string key)
    {
        for (Node? n = _buckets[IndexFor(key)]; n is not null; n = n.Next)
            if (n.Key == key) return n.Value;
        return null;
    }

    public void Print()
    {
        for (int i = 0; i < _buckets.Length; i++)
        {
            var chain = new List<string>();
            for (Node? n = _buckets[i]; n is not null; n = n.Next)
                chain.Add($"{n.Key}={n.Value}");
            Console.WriteLine($"[{i}] " + (chain.Count == 0 ? "∅" : string.Join(" → ", chain)));
        }
    }

    // Власний детермінований хеш (djb2), а НЕ string.GetHashCode(),
    // щоб розкладка була однаковою при кожному запуску.
    private int IndexFor(string key)
    {
        uint h = 5381;
        foreach (char c in key) h = unchecked(h * 33 + c);
        return (int)(h % (uint)_buckets.Length);
    }
}
```

**Приклад запуску:**
```
[0] ∅
[1] kiwi=4 → plum=4
[2] date=4
[3] fig=3 → pear=4 → apple=5
[4] lime=4
plum → 4
mango → не знайдено
```

### 4.3. Повна узагальнена реалізація

Тепер — повноцінна `ChainedHashMap<TKey, TValue>`:

- узагальнена, з `IEqualityComparer<TKey>`;
- зберігає **хеш-код у вузлі** (не перераховуємо під час рехешування і швидко відсікаємо нерівні ключі);
- **рехешування** при α > 1.0 (подвоєння);
- `Remove`;
- **перелічувач** (`IEnumerable<KeyValuePair<TKey,TValue>>`) з перевіркою модифікації під час обходу.

Щоб вивід був детермінованим, у демонстрації передаємо власний компаратор рядків з FNV-1a.

```csharp
using System.Collections;

// Повна хеш-таблиця з ланцюжками: generic, resize, remove, enumerator.
var map = new ChainedHashMap<string, int>(new Fnv1aComparer());
string[] fruits = ["apple", "banana", "cherry", "date", "elder", "fig", "grape", "honeydew", "kiwi"];
for (int i = 0; i < fruits.Length; i++)
{
    map[fruits[i]] = i;
    Console.WriteLine($"+ {fruits[i],-9} Count={map.Count,-2} Buckets={map.BucketCount,-2} α={map.LoadFactor:F2}");
}

Console.WriteLine($"TryGetValue(\"fig\") → {map.TryGetValue("fig", out int fig)} {fig}");
Console.WriteLine($"Remove(\"banana\") → {map.Remove("banana")}");
Console.WriteLine($"Remove(\"banana\") → {map.Remove("banana")}");
Console.WriteLine($"ContainsKey(\"banana\") → {map.ContainsKey("banana")}");
Console.WriteLine($"Найдовший ланцюжок: {map.LongestChain()}");
Console.WriteLine("Вміст: " + string.Join(", ", map.OrderBy(p => p.Value).Select(p => $"{p.Key}:{p.Value}")));

try
{
    foreach (var pair in map)
        map[pair.Key + "!"] = 0; // модифікація під час обходу
}
catch (InvalidOperationException ex)
{
    Console.WriteLine($"Виняток: {ex.Message}");
}

// Детермінований компаратор рядків на основі FNV-1a.
public sealed class Fnv1aComparer : IEqualityComparer<string>
{
    public bool Equals(string? x, string? y) => string.Equals(x, y, StringComparison.Ordinal);

    public int GetHashCode(string s)
    {
        uint h = 2166136261;
        foreach (char c in s) h = unchecked((h ^ c) * 16777619);
        return (int)h;
    }
}

public sealed class ChainedHashMap<TKey, TValue> : IEnumerable<KeyValuePair<TKey, TValue>>
    where TKey : notnull
{
    private const int DefaultCapacity = 4;
    private const double MaxLoadFactor = 1.0;

    private sealed class Node(TKey key, TValue value, int hashCode, Node? next)
    {
        public TKey Key { get; } = key;
        public TValue Value { get; set; } = value;
        public int HashCode { get; } = hashCode; // кешований хеш
        public Node? Next { get; set; } = next;
    }

    private readonly IEqualityComparer<TKey> _comparer;
    private Node?[] _buckets;
    private int _count;
    private int _version; // збільшується при кожній модифікації — для перелічувача

    public ChainedHashMap(IEqualityComparer<TKey>? comparer = null, int capacity = DefaultCapacity)
    {
        _comparer = comparer ?? EqualityComparer<TKey>.Default;
        _buckets = new Node?[Math.Max(1, capacity)];
    }

    public int Count => _count;
    public int BucketCount => _buckets.Length;
    public double LoadFactor => (double)_count / _buckets.Length;

    public TValue this[TKey key]
    {
        get => TryGetValue(key, out TValue value) ? value : throw new KeyNotFoundException($"Ключ '{key}' відсутній");
        set => Insert(key, value, overwrite: true);
    }

    public void Add(TKey key, TValue value) => Insert(key, value, overwrite: false);

    public bool ContainsKey(TKey key) => FindNode(key) is not null;

    public bool TryGetValue(TKey key, out TValue value)
    {
        Node? node = FindNode(key);
        value = node is null ? default! : node.Value;
        return node is not null;
    }

    public bool Remove(TKey key)
    {
        int hash = Hash(key);
        int index = IndexFor(hash, _buckets.Length);
        Node? previous = null;
        for (Node? n = _buckets[index]; n is not null; previous = n, n = n.Next)
        {
            if (n.HashCode != hash || !_comparer.Equals(n.Key, key)) continue;

            if (previous is null) _buckets[index] = n.Next; // видаляємо голову ланцюжка
            else previous.Next = n.Next;                    // "вирізаємо" вузол із середини
            _count--;
            _version++;
            return true;
        }
        return false;
    }

    public int LongestChain()
    {
        int longest = 0;
        foreach (Node? head in _buckets)
        {
            int length = 0;
            for (Node? n = head; n is not null; n = n.Next) length++;
            longest = Math.Max(longest, length);
        }
        return longest;
    }

    public IEnumerator<KeyValuePair<TKey, TValue>> GetEnumerator()
    {
        int version = _version;
        foreach (Node? head in _buckets)
        {
            for (Node? n = head; n is not null; n = n.Next)
            {
                if (version != _version)
                    throw new InvalidOperationException("Колекцію змінено під час перелічення");
                yield return new KeyValuePair<TKey, TValue>(n.Key, n.Value);
            }
        }
        if (version != _version)
            throw new InvalidOperationException("Колекцію змінено під час перелічення");
    }

    IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();

    private void Insert(TKey key, TValue value, bool overwrite)
    {
        int hash = Hash(key);
        int index = IndexFor(hash, _buckets.Length);
        for (Node? n = _buckets[index]; n is not null; n = n.Next)
        {
            if (n.HashCode == hash && _comparer.Equals(n.Key, key)) // спершу дешеве порівняння хешів
            {
                if (!overwrite) throw new ArgumentException($"Ключ '{key}' уже існує");
                n.Value = value;
                _version++;
                return;
            }
        }

        _buckets[index] = new Node(key, value, hash, _buckets[index]);
        _count++;
        _version++;

        if (LoadFactor > MaxLoadFactor)
            Resize(_buckets.Length * 2);
    }

    // Рехешування: переносимо ВУЗЛИ (без нових алокацій) у новий масив кошиків.
    private void Resize(int newSize)
    {
        var newBuckets = new Node?[newSize];
        foreach (Node? head in _buckets)
        {
            Node? n = head;
            while (n is not null)
            {
                Node? next = n.Next;                        // запам'ятовуємо, бо Next перезапишемо
                int index = IndexFor(n.HashCode, newSize);  // хеш не перераховуємо — він кешований
                n.Next = newBuckets[index];
                newBuckets[index] = n;
                n = next;
            }
        }
        _buckets = newBuckets;
    }

    private Node? FindNode(TKey key)
    {
        int hash = Hash(key);
        for (Node? n = _buckets[IndexFor(hash, _buckets.Length)]; n is not null; n = n.Next)
            if (n.HashCode == hash && _comparer.Equals(n.Key, key))
                return n;
        return null;
    }

    private int Hash(TKey key) => _comparer.GetHashCode(key) & 0x7FFFFFFF; // невід'ємний хеш

    private static int IndexFor(int hash, int size) => hash % size;
}
```

**Приклад запуску:**
```
+ apple     Count=1  Buckets=4  α=0.25
+ banana    Count=2  Buckets=4  α=0.50
+ cherry    Count=3  Buckets=4  α=0.75
+ date      Count=4  Buckets=4  α=1.00
+ elder     Count=5  Buckets=8  α=0.62
+ fig       Count=6  Buckets=8  α=0.75
+ grape     Count=7  Buckets=8  α=0.88
+ honeydew  Count=8  Buckets=8  α=1.00
+ kiwi      Count=9  Buckets=16 α=0.56
TryGetValue("fig") → True 5
Remove("banana") → True
Remove("banana") → False
ContainsKey("banana") → False
Найдовший ланцюжок: 2
Вміст: apple:0, cherry:2, date:3, elder:4, fig:5, grape:6, honeydew:7, kiwi:8
Виняток: Колекцію змінено під час перелічення
```

### 4.4. Навіщо зберігати хеш-код у вузлі

1. **Рехешування без виклику `GetHashCode`** — для складних ключів (довгі рядки, записи) це суттєва економія.
2. **Швидке відсікання**: порівняння двох `int` дешевше за `Equals` рядків. У ланцюжку, де всі ключі мають різні хеші, `Equals` викликається лише для «справжнього» кандидата.
3. .NET `Dictionary` робить так само (поле `hashCode` в `Entry`).

### Типові помилки (ланцюжки)

- ❌ Забути перевірити наявність ключа перед вставкою → дублікати в ланцюжку.
- ❌ При рехешуванні використовувати **старий** розмір для індексу.
- ❌ У `Remove` забути випадок, коли видаляється **голова** ланцюжка.
- ❌ Порівнювати лише хеш-коди без `Equals` → різні ключі вважаються однаковими.
- ❌ Модифікувати таблицю під час `foreach` без захисту версією → пропущені або повторені елементи.

### Міні-вправа 4.1

Таблиця з ланцюжками, `m = 7`, `h(k) = k mod 7`, вставка на початок ланцюжка. Вставте 10, 22, 31, 4, 15, 28, 17, 88, 59. Намалюйте таблицю. Яка довжина найдовшого ланцюжка?

<details>
<summary>Розв'язок</summary>

```
10 mod 7 = 3    22 mod 7 = 1    31 mod 7 = 3    4 mod 7 = 4    15 mod 7 = 1
28 mod 7 = 0    17 mod 7 = 3    88 mod 7 = 4    59 mod 7 = 3

[0] 28
[1] 15 → 22
[2] ∅
[3] 59 → 17 → 31 → 10
[4] 88 → 4
[5] ∅
[6] ∅
```

Найдовший ланцюжок — кошик 3, довжина **4**. α = 9/7 ≈ 1.29.
</details>

---

## 5. Відкрита адресація

### 5.1. Ідея

Усі елементи зберігаються **безпосередньо в масиві** (без списків). При колізії шукаємо **іншу** вільну комірку за певною **послідовністю проб** (probe sequence):

```
h(k, i)  для i = 0, 1, 2, ..., m−1  — перестановка індексів 0..m−1
```

| Стратегія | Формула проби | Проблема |
|-----------|---------------|----------|
| Лінійне пробування | `h(k, i) = (h₁(k) + i) mod m` | первинна кластеризація |
| Квадратичне пробування | `h(k, i) = (h₁(k) + c₁i + c₂i²) mod m` | вторинна кластеризація, не всі комірки досяжні |
| Подвійне хешування | `h(k, i) = (h₁(k) + i · h₂(k)) mod m` | дорожче обчислення |

**Обов'язково α < 1** (таблиця не може містити більше елементів, ніж комірок); на практиці тримають **α ≤ 0.5–0.75**.

**Переваги:** немає алокацій на елемент, **чудова локальність кешу** (лінійне пробування — сусідні комірки в одній кеш-лінії).
**Недоліки:** чутливість до α і до якості хешу, складне видалення.

### 5.2. Лінійне пробування

```
m = 10, h(k) = k mod 10, вставляємо 23, 43, 13, 27, 33

  вставка 23: h=3 → [3] вільна
  вставка 43: h=3 → [3] зайнята → [4] вільна
  вставка 13: h=3 → [3] → [4] → [5] вільна
  вставка 27: h=7 → [7] вільна
  вставка 33: h=3 → [3] → [4] → [5] → [6] вільна

  індекс:  0    1    2    3    4    5    6    7    8    9
         ┌────┬────┬────┬────┬────┬────┬────┬────┬────┬────┐
         │    │    │    │ 23 │ 43 │ 13 │ 33 │ 27 │    │    │
         └────┴────┴────┴────┴────┴────┴────┴────┴────┴────┘
                         └──────── кластер ────────┘
```

**Первинна кластеризація**: зайняті комірки злипаються у суцільні блоки. Будь-який ключ, що хешується в кластер, **подовжує** його. Імовірність потрапити в кластер довжини `L` пропорційна `L+1` — «багаті багатшають».

Продемонструємо кількісно: однаковий набір випадкових ключів, три стратегії, рахуємо кількість проб і довжину найбільшого кластера.

```csharp
// Порівняння стратегій відкритої адресації: середня кількість проб
// при успішному пошуку та найдовший кластер зайнятих комірок.
const int M = 1009; // просте число — потрібне для квадратичного і подвійного хешування
var rng = new Random(Seed: 7);
double[] loadFactors = [0.5, 0.7, 0.8, 0.9, 0.95];

Console.WriteLine($"{"α",5} | {"лінійне",15} | {"квадратичне",15} | {"подвійне",15}");
Console.WriteLine($"{"",5} | {"проб / кластер",15} | {"проб / кластер",15} | {"проб / кластер",15}");
foreach (double alpha in loadFactors)
{
    int n = (int)(M * alpha);
    int[] keys = Enumerable.Range(0, n).Select(_ => rng.Next()).ToArray();

    string Run(Func<int, int, int> probe)
    {
        var table = new int?[M];
        long totalProbes = 0;
        foreach (int key in keys)
        {
            for (int i = 0; ; i++)
            {
                int index = probe(key, i);
                if (table[index] is null)
                {
                    table[index] = key;
                    totalProbes += i + 1; // пошук цього ключа потім зробить стільки ж проб
                    break;
                }
            }
        }
        return $"{(double)totalProbes / n,5:F2} / {LongestCluster(table),5}";
    }

    string linear = Run((k, i) => (H1(k) + i) % M);
    string quadratic = Run((k, i) => (int)((H1(k) + (long)i * i) % M));
    string doubleHash = Run((k, i) => (int)((H1(k) + (long)i * H2(k)) % M));
    Console.WriteLine($"{alpha,5:F2} | {linear,15} | {quadratic,15} | {doubleHash,15}");
}

static int H1(int key) => key % M;
static int H2(int key) => 1 + key % (M - 1); // ніколи не 0, інакше проба стоїть на місці

// Найдовший суцільний блок зайнятих комірок (з урахуванням "загортання" масиву).
static int LongestCluster(int?[] table)
{
    int longest = 0, current = 0;
    for (int i = 0; i < 2 * table.Length; i++) // два проходи — щоб врахувати кластер через кінець
    {
        current = table[i % table.Length] is null ? 0 : current + 1;
        longest = Math.Max(longest, Math.Min(current, table.Length));
    }
    return longest;
}
```

**Приклад запуску:**
```
    α |         лінійне |     квадратичне |        подвійне
      |  проб / кластер |  проб / кластер |  проб / кластер
 0.50 |    1.47 /    18 |    1.40 /    11 |    1.45 /     9
 0.70 |    1.89 /    27 |    1.78 /    20 |    1.64 /    14
 0.80 |    2.74 /    59 |    2.01 /    32 |    1.95 /    32
 0.90 |    5.97 /   325 |    2.85 /    81 |    2.64 /    45
 0.95 |    7.23 /   505 |    3.42 /    93 |    3.08 /   131
```

При α = 0.95 лінійне пробування дає кластери в сотні комірок, а подвійне хешування — у рази коротші.

### 5.3. Квадратичне пробування

```
h(k, i) = (h₁(k) + i²) mod m        послідовність зсувів: 0, 1, 4, 9, 16, 25, ...
```

Розриває первинні кластери, бо проби «стрибають» дедалі далі. Але:

- **Вторинна кластеризація**: ключі з однаковим `h₁(k)` мають **однакову** послідовність проб.
- **Не всі комірки досяжні**: при простому `m` і `i² mod m` досяжні лише ⌈m/2⌉ різних комірок. Гарантія: якщо `m` просте і α ≤ 0.5, вільна комірка **завжди** знайдеться.
- Варіант для `m = 2^p`: зсуви `i(i+1)/2` (трикутні числа) — обходять **усі** комірки.

```csharp
// Які комірки досяжні квадратичним пробуванням з однієї стартової позиції?
int[] sizes = [11, 16];
foreach (int m in sizes)
{
    var squares = new SortedSet<int>();
    var triangular = new SortedSet<int>();
    for (int i = 0; i < m; i++)
    {
        squares.Add(i * i % m);             // зсув i²
        triangular.Add(i * (i + 1) / 2 % m); // зсув i(i+1)/2
    }
    Console.WriteLine($"m = {m}");
    Console.WriteLine($"  i²       → {squares.Count,2} комірок: {string.Join(",", squares)}");
    Console.WriteLine($"  i(i+1)/2 → {triangular.Count,2} комірок: {string.Join(",", triangular)}");
}
```

**Приклад запуску:**
```
m = 11
  i²       →  6 комірок: 0,1,3,4,5,9
  i(i+1)/2 →  6 комірок: 0,1,3,4,6,10
m = 16
  i²       →  4 комірок: 0,1,4,9
  i(i+1)/2 → 16 комірок: 0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15
```

Для `m = 16` (степінь 2) трикутні зсуви покривають **усі 16** комірок — саме тому цю схему використовують у таблицях зі степенем двійки (наприклад, у Python `dict` використовується інша, псевдовипадкова схема з тією ж метою).

### 5.4. Подвійне хешування

```
h(k, i) = (h₁(k) + i · h₂(k)) mod m
```

Крок `h₂(k)` **залежить від ключа**, тож ключі з однаковим стартом розходяться. Вимоги:

- `h₂(k) ≠ 0` (інакше нескінченний цикл на одній комірці);
- `h₂(k)` взаємно просте з `m` (щоб обійти всі комірки). Найпростіше: `m` просте, `h₂(k) = 1 + (k mod (m−1))`. Або `m = 2^p` і `h₂(k)` завжди непарне.

Подвійне хешування найближче до ідеального **рівномірного хешування** (кожна перестановка проб рівноймовірна).

### 5.5. Видалення: надгробки (tombstones)

Не можна просто очистити комірку при відкритій адресації — це **розірве ланцюг проб** для інших ключів:

```
Лінійне пробування, h(k) = k mod 10. У таблиці 23, 43, 13 (всі з h=3):

  [3]=23  [4]=43  [5]=13

Delete(43) як "очистити":   [3]=23  [4]=∅  [5]=13
Search(13): h=3 → [3]=23 ≠ 13 → [4]=∅ → "не знайдено"   ✗ ПОМИЛКА! 13 є в [5]

Delete(43) як "надгробок":  [3]=23  [4]=†  [5]=13
Search(13): h=3 → [3]=23 → [4]=† (продовжуємо) → [5]=13  ✓ знайдено
```

Правила для станів комірки `Empty` / `Occupied` / `Deleted`:

| Операція | `Empty` | `Occupied` (інший ключ) | `Deleted` (†) |
|----------|---------|-------------------------|---------------|
| Search | **зупинитися**: не знайдено | продовжити | **продовжити** |
| Insert | вставити (або в перший запам'ятований †) | продовжити | запам'ятати як кандидата, продовжити пошук дубліката |

**Проблема надгробків**: вони накопичуються, і пошук сповільнюється, навіть якщо живих елементів мало. Рішення — рахувати `live + tombstones` у коефіцієнті заповнення і **рехешувати** (що прибирає всі надгробки).

```csharp
// Лінійне пробування з надгробками: демонстрація, чому не можна "просто очистити" комірку.
var broken = new LinearProbingDemo(10, useTombstones: false);
var correct = new LinearProbingDemo(10, useTombstones: true);

foreach (var table in new[] { broken, correct })
{
    foreach (int key in new[] { 23, 43, 13, 27 }) table.Insert(key);
    Console.WriteLine($"{table.Name}:");
    Console.WriteLine($"  до видалення:     {table}");
    table.Delete(43);
    Console.WriteLine($"  після Delete(43): {table}");
    Console.WriteLine($"  Contains(13) = {table.Contains(13)}, Contains(43) = {table.Contains(43)}");
}

public sealed class LinearProbingDemo(int size, bool useTombstones)
{
    private enum SlotState : byte { Empty, Occupied, Deleted }

    private readonly int[] _keys = new int[size];
    private readonly SlotState[] _states = new SlotState[size];

    public string Name => useTombstones ? "З надгробками" : "Без надгробків (НЕПРАВИЛЬНО)";

    public void Insert(int key)
    {
        int firstTombstone = -1;
        for (int i = 0; i < size; i++)
        {
            int index = (key % size + i) % size;
            switch (_states[index])
            {
                case SlotState.Occupied when _keys[index] == key:
                    return; // ключ уже є
                case SlotState.Deleted:
                    if (firstTombstone < 0) firstTombstone = index; // можна перевикористати
                    break;
                case SlotState.Empty:
                    int target = firstTombstone >= 0 ? firstTombstone : index;
                    _keys[target] = key;
                    _states[target] = SlotState.Occupied;
                    return;
            }
        }
        if (firstTombstone >= 0)
        {
            _keys[firstTombstone] = key;
            _states[firstTombstone] = SlotState.Occupied;
            return;
        }
        throw new InvalidOperationException("Таблиця переповнена");
    }

    public bool Contains(int key) => FindIndex(key) >= 0;

    public void Delete(int key)
    {
        int index = FindIndex(key);
        if (index < 0) return;
        _states[index] = useTombstones ? SlotState.Deleted : SlotState.Empty;
    }

    private int FindIndex(int key)
    {
        for (int i = 0; i < size; i++)
        {
            int index = (key % size + i) % size;
            if (_states[index] == SlotState.Empty) return -1;  // порожня → ключа точно немає
            if (_states[index] == SlotState.Occupied && _keys[index] == key) return index;
            // Deleted або інший ключ — продовжуємо пробування
        }
        return -1;
    }

    public override string ToString() =>
        string.Join(" ", Enumerable.Range(0, size).Select(i => _states[i] switch
        {
            SlotState.Occupied => _keys[i].ToString(),
            SlotState.Deleted => "†",
            _ => "·",
        }));
}
```

**Приклад запуску:**
```
Без надгробків (НЕПРАВИЛЬНО):
  до видалення:     · · · 23 43 13 · 27 · ·
  після Delete(43): · · · 23 · 13 · 27 · ·
  Contains(13) = False, Contains(43) = False
З надгробками:
  до видалення:     · · · 23 43 13 · 27 · ·
  після Delete(43): · · · 23 † 13 · 27 · ·
  Contains(13) = True, Contains(43) = False
```

> **Альтернатива надгробкам для лінійного пробування — зсув назад (backward shift deletion).** Після видалення проходимо кластер праворуч і переносимо елементи, які можна поставити ближче до їхнього «домашнього» кошика. Надгробків немає зовсім.

### 5.6. Хешування Робін Гуда (Robin Hood hashing)

Варіант лінійного пробування з **перерозподілом «багатства»**. Для кожного елемента зберігаємо **DIB** (distance to initial bucket) — наскільки далеко він від свого домашнього кошика.

**Правило вставки:** якщо в поточній комірці елемент «багатший» (його DIB **менший** за DIB того, кого вставляємо), — **забираємо** в нього місце, а його самого несемо далі.

```
Вставляємо X з h(X)=2. Числа під ключами — DIB.

 індекс:  2      3      4      5
        ┌──────┬──────┬──────┬──────┐
        │ A:0  │ B:1  │ C:0  │  ·   │
        └──────┴──────┴──────┴──────┘
  X у [2]: DIB(X)=0, DIB(A)=0 → не багатший, йдемо далі
  X у [3]: DIB(X)=1, DIB(B)=1 → далі
  X у [4]: DIB(X)=2, DIB(C)=0 → C "багатший" → X займає [4], несемо C
  C у [5]: DIB(C)=1, порожньо → ставимо

        ┌──────┬──────┬──────┬──────┐
        │ A:0  │ B:1  │ X:2  │ C:1  │
        └──────┴──────┴──────┴──────┘
```

**Результат:** дисперсія довжин проб різко зменшується — **максимальна** довжина проби росте як `O(log n)`, а середня майже не змінюється. Бонус: під час пошуку можна **зупинитися раніше** — якщо DIB поточного елемента менший за нашу поточну відстань, шуканого ключа точно немає.

```csharp
// Лінійне пробування vs Robin Hood: однакові ключі, порівнюємо середню і максимальну відстань
// від домашнього кошика (DIB) — тобто кількість проб для успішного пошуку мінус 1.
const int M = 1 << 12; // 4096 комірок
var rng = new Random(Seed: 2025);

Console.WriteLine($"{"α",5} | {"лінійне сер/макс",17} | {"Robin Hood сер/макс",19}");
foreach (double alpha in new[] { 0.5, 0.75, 0.9, 0.95 })
{
    int n = (int)(M * alpha);
    uint[] hashes = Enumerable.Range(0, n).Select(_ => (uint)rng.Next()).ToArray();

    (double avg, int max) linear = Simulate(hashes, robinHood: false);
    (double avg, int max) robin = Simulate(hashes, robinHood: true);
    Console.WriteLine($"{alpha,5:F2} | {linear.avg,8:F2} / {linear.max,-6} | {robin.avg,8:F2} / {robin.max,-8}");
}

static (double Average, int Max) Simulate(uint[] hashes, bool robinHood)
{
    var slots = new uint?[M];   // зберігаємо хеш елемента (ключ не потрібен для статистики)
    foreach (uint h in hashes)
    {
        uint carried = h;
        int dib = 0;                            // відстань елемента, який "несемо"
        int index = (int)(carried & (M - 1));
        while (true)
        {
            if (slots[index] is not uint occupant)
            {
                slots[index] = carried;
                break;
            }
            int occupantDib = (index - (int)(occupant & (M - 1)) + M) & (M - 1);
            if (robinHood && occupantDib < dib)
            {
                slots[index] = carried;  // "грабуємо багатого": займаємо його місце
                carried = occupant;      // і далі несемо витісненого
                dib = occupantDib;
            }
            index = (index + 1) & (M - 1);
            dib++;
        }
    }

    long total = 0;
    int max = 0;
    for (int i = 0; i < M; i++)
    {
        if (slots[i] is not uint h) continue;
        int d = (i - (int)(h & (M - 1)) + M) & (M - 1);
        total += d;
        max = Math.Max(max, d);
    }
    return ((double)total / hashes.Length, max);
}
```

**Приклад запуску:**
```
    α |  лінійне сер/макс | Robin Hood сер/макс
 0.50 |     0.46 / 12     |     0.46 / 4
 0.75 |     1.58 / 52     |     1.58 / 13
 0.90 |     4.12 / 268    |     4.12 / 22
 0.95 |    10.77 / 1012   |    10.77 / 39
```

Середнє **однакове** (сума DIB не залежить від порядку розміщення в лінійному пробуванні), а **максимум** у Robin Hood — у кілька разів менший. Саме тому Robin Hood (і його нащадки, як SwissTable) популярні в Rust, C++ (`absl::flat_hash_map`) тощо.

### Типові помилки (відкрита адресація)

- ❌ Видалення через «очищення» комірки без надгробка → «загублені» ключі.
- ❌ Не враховувати надгробки в коефіцієнті заповнення → при α_live = 0.1 пошук може проходити всю таблицю.
- ❌ α → 1 → кількість проб вибухає; при повній таблиці пошук відсутнього ключа — **нескінченний цикл**, якщо немає лічильника.
- ❌ Подвійне хешування з `h₂(k) = 0` або `gcd(h₂(k), m) > 1`.
- ❌ Квадратичне пробування при α > 0.5 без гарантій → вставка може не знайти вільну комірку, хоча вона є.

### Міні-вправа 5.1

`m = 11`, лінійне пробування, `h(k) = k mod 11`. Вставте 22, 1, 13, 11, 24, 33. Потім видаліть 13 (надгробок) і знайдіть 24. Скільки проб знадобиться?

<details>
<summary>Розв'язок</summary>

```
22 → h=0 → [0]
 1 → h=1 → [1]
13 → h=2 → [2]
11 → h=0 → [0]✗ [1]✗ [2]✗ [3]
24 → h=2 → [2]✗ [3]✗ [4]
33 → h=0 → [0]✗ [1]✗ [2]✗ [3]✗ [4]✗ [5]

 0   1   2   3   4   5   6 ...
22   1  13  11  24  33   ·

Delete(13): [2] = †
Search(24): h=2 → [2]=† (далі) → [3]=11 (далі) → [4]=24 ✓
```

**3 проби.**
</details>

### Міні-вправа 5.2

Той самий набір ключів вставте методом подвійного хешування з `h₁(k) = k mod 11`, `h₂(k) = 1 + (k mod 10)`.

<details>
<summary>Розв'язок</summary>

```
22: h₁=0 → [0]
 1: h₁=1 → [1]
13: h₁=2 → [2]
11: h₁=0, h₂=2 → [0]✗ → [2]✗ → [4]
24: h₁=2, h₂=5 → [2]✗ → [7]
33: h₁=0, h₂=4 → [0]✗ → [4]✗ → [8]

 0   1   2   3   4   5   6   7   8   9  10
22   1  13   ·  11   ·   ·  24  33   ·   ·
```

Кластерів немає: 11, 24, 33 розкидані по таблиці.
</details>

---

## 6. Коефіцієнт заповнення та аналіз складності

### 6.1. Очікувана кількість проб

Припущення **простого рівномірного хешування** (кожен ключ рівноймовірно потрапляє в будь-який кошик незалежно від інших). α = n / m.

| Метод | Невдалий пошук (ключа немає) | Успішний пошук |
|-------|------------------------------|----------------|
| Ланцюжки | `1 + α` | `1 + α/2` |
| Лінійне пробування (Кнут) | `½ · (1 + 1/(1−α)²)` | `½ · (1 + 1/(1−α))` |
| Рівномірне хешування / подвійне | `1/(1−α)` | `(1/α) · ln(1/(1−α))` |

```csharp
// Таблиця очікуваної кількості проб за теоретичними формулами.
double[] alphas = [0.25, 0.5, 0.75, 0.9, 0.95, 0.99];

Console.WriteLine("            |  ланцюжки   |   лінійне    |   подвійне");
Console.WriteLine("     α      | невд.  усп. | невд.   усп. | невд.   усп.");
Console.WriteLine("------------+-------------+--------------+-------------");
foreach (double a in alphas)
{
    double chainMiss = 1 + a;
    double chainHit = 1 + a / 2;
    double linearMiss = 0.5 * (1 + 1 / ((1 - a) * (1 - a)));
    double linearHit = 0.5 * (1 + 1 / (1 - a));
    double uniformMiss = 1 / (1 - a);
    double uniformHit = 1 / a * Math.Log(1 / (1 - a));
    Console.WriteLine($"   {a,5:F2}    | {chainMiss,5:F2} {chainHit,5:F2} | {linearMiss,6:F1} {linearHit,5:F1} | {uniformMiss,5:F1} {uniformHit,5:F2}");
}
```

**Приклад запуску:**
```
            |  ланцюжки   |   лінійне    |   подвійне
     α      | невд.  усп. | невд.   усп. | невд.   усп.
------------+-------------+--------------+-------------
    0.25    |  1.25  1.12 |    1.4   1.2 |   1.3  1.15
    0.50    |  1.50  1.25 |    2.5   1.5 |   2.0  1.39
    0.75    |  1.75  1.38 |    8.5   2.5 |   4.0  1.85
    0.90    |  1.90  1.45 |   50.5   5.5 |  10.0  2.56
    0.95    |  1.95  1.48 |  200.5  10.5 |  20.0  3.15
    0.99    |  1.99  1.50 | 5000.5  50.5 | 100.0  4.65
```

**Висновки:**

- Ланцюжки деградують **лінійно** за α — навіть α = 2 цілком працездатна.
- Лінійне пробування при α = 0.9 для невдалого пошуку — **~50 проб**, при 0.99 — **~5000**. Тому ліміт α ≈ 0.5–0.75.
- Подвійне хешування краще за лінійне при великих α, але лінійне часто **швидше на практиці** завдяки кешу CPU при малих α.

### 6.2. Рехешування та амортизований аналіз

Коли α перевищує поріг, створюємо таблицю **вдвічі** більшого розміру і переносимо всі елементи (кожен — з новим індексом). Одне рехешування коштує O(n), але трапляється рідко.

**Амортизований аналіз (метод сумування).** Нехай вставляємо `n` елементів, починаючи з розміру 1 і подвоюючи. Рехешування відбуваються при розмірах 1, 2, 4, …, 2^k ≤ n:

```
Загальна вартість переносів = 1 + 2 + 4 + … + 2^k < 2 · 2^k ≤ 2n
Загальна вартість вставок  = n
Разом                      < 3n  ⇒  O(1) амортизовано на операцію
```

**Чому саме множення, а не додавання?** Якщо збільшувати на константу `c`, переносів буде `c + 2c + 3c + … ≈ n²/(2c)` — O(n) амортизовано.

```csharp
// Підрахунок вартості рехешувань: множення на 2 vs додавання константи.
int[] sizes = [1_000, 10_000, 100_000, 1_000_000];

Console.WriteLine($"{"n",9} | {"×2: переносів",14} {"на елемент",10} | {"+100: переносів",16} {"на елемент",10}");
foreach (int n in sizes)
{
    long doubling = CountMoves(n, grow: cap => cap * 2);
    long additive = CountMoves(n, grow: cap => cap + 100);
    Console.WriteLine($"{n,9} | {doubling,14} {(double)doubling / n,10:F2} | {additive,16} {(double)additive / n,10:F1}");
}

// Моделюємо вставку n елементів: при переповненні (count == capacity) переносимо всі елементи.
static long CountMoves(int n, Func<long, long> grow)
{
    long capacity = 1, count = 0, moves = 0;
    for (int i = 0; i < n; i++)
    {
        if (count == capacity)
        {
            moves += count;          // рехешування: кожен елемент отримує новий індекс
            capacity = grow(capacity);
        }
        count++;
    }
    return moves;
}
```

**Приклад запуску:**
```
        n |  ×2: переносів на елемент |  +100: переносів на елемент
     1000 |           1023       1.02 |             4510        4.5
    10000 |          16383       1.64 |           495100       49.5
   100000 |         131071       1.31 |         49951000      499.5
  1000000 |        1048575       1.05 |       4999510000     4999.5
```

### 6.3. Практичні аспекти рехешування

- **Затримки (latency spikes).** Одна вставка може зайняти O(n) — погано для систем реального часу. Рішення: **інкрементальне рехешування** (Redis тримає дві таблиці й переносить по кілька кошиків за кожну операцію).
- **Попереднє виділення.** Якщо кількість елементів відома — `new Dictionary<K,V>(capacity)` уникає всіх рехешувань.
- **Зменшення.** Після масових видалень таблиця лишається великою. У .NET є `TrimExcess()`. Зменшувати варто з гістерезисом (наприклад, при α < 0.125 → розмір /2), щоб уникнути «пилкоподібних» рехешувань на межі.
- **Ітератори.** Рехешування змінює порядок елементів — ще одна причина не покладатися на порядок обходу хеш-таблиці.

### Міні-вправа 6.1

Таблиця з лінійним пробуванням тримає α ≤ 0.75. Скільки в середньому проб займе пошук **відсутнього** ключа безпосередньо перед рехешуванням? А якщо поріг підняти до 0.9?

<details>
<summary>Розв'язок</summary>

α = 0.75: `½ · (1 + 1/0.25²) = ½ · (1 + 16) = 8.5` проб.
α = 0.9: `½ · (1 + 1/0.1²) = ½ · 101 = 50.5` проб — майже в 6 разів більше. Економія пам'яті 15% коштує шестикратного сповільнення невдалих пошуків.
</details>

---

## ☕ Перерва 2 (5 хв)

---


## 7. Власний HashMap<TKey,TValue> крок за кроком

У розділі 4 ми побудували таблицю з ланцюжками на вузлах-об'єктах. Тепер зберемо **компактну** таблицю з відкритою адресацією — такою, як у промислових бібліотеках, — і пройдемо шлях від порожнього класу до повної реалізації.

### 7.1. Проєктні рішення

| Рішення | Вибір | Обґрунтування |
|---------|-------|---------------|
| Стратегія колізій | лінійне пробування | локальність кешу, простота |
| Розмір таблиці | степінь двійки | `& (m−1)` замість `%` |
| Перемішування хешу | фіналізатор Murmur | компенсує слабкі `GetHashCode` при маскуванні |
| Видалення | надгробки | простота; рехешування прибирає їх |
| Поріг заповнення | `(live + tombstones) / m ≤ 0.75` | ~8.5 проб на невдалий пошук у гіршому випадку |
| Зберігання | паралельні масиви `_keys`, `_values`, `_hashes`, `_states` | немає алокацій на елемент |
| Порівняння ключів | `IEqualityComparer<TKey>` | підтримка регістронезалежних ключів тощо |

### 7.2. Крок 1 — каркас і поля

```
         _states   _hashes     _keys      _values
 [0]     Empty        –          –           –
 [1]     Occupied  0x1A2B..   "apple"        5
 [2]     Deleted      –          –           –      ← надгробок
 [3]     Occupied  0x77C1..   "kiwi"         4
 ...
```

### 7.3. Крок 2 — пошук індексу

Єдиний метод `FindSlot` обслуговує і пошук, і вставку: повертає або індекс знайденого ключа, або індекс, **куди його вставити** (перший надгробок або перша порожня комірка).

### 7.4. Крок 3 — вставка, пошук, видалення, рехешування, перелічувач

Нижче — **повна** реалізація з демонстрацією. Код довгий, але кожен метод короткий і відповідає одному кроку.

```csharp
using System.Collections;
using System.Diagnostics.CodeAnalysis;

// Демонстрація власної хеш-таблиці з відкритою адресацією.
var ages = new OpenHashMap<string, int>(new OrdinalFnvComparer());
ages["Олена"] = 21;
ages["Тарас"] = 19;
ages["Марія"] = 22;
ages.Add("Іван", 20);
ages["Тарас"] = 20; // оновлення

Console.WriteLine($"Count = {ages.Count}, Capacity = {ages.Capacity}");
Console.WriteLine($"ages[\"Марія\"] = {ages["Марія"]}");
Console.WriteLine($"TryGetValue(\"Петро\") = {ages.TryGetValue("Петро", out _)}");

try { ages.Add("Іван", 99); }
catch (ArgumentException ex) { Console.WriteLine($"Add дубліката: {ex.Message}"); }

try { _ = ages["Петро"]; }
catch (KeyNotFoundException ex) { Console.WriteLine($"Індексатор: {ex.Message}"); }

Console.WriteLine($"Remove(\"Олена\") = {ages.Remove("Олена")}, Tombstones = {ages.Tombstones}");

// Масова вставка: спостерігаємо за рехешуваннями.
var numbers = new OpenHashMap<int, int>();
int lastCapacity = numbers.Capacity;
for (int i = 0; i < 1000; i++)
{
    numbers[i] = i * i;
    if (numbers.Capacity != lastCapacity)
    {
        Console.WriteLine($"  рехешування: {lastCapacity,4} → {numbers.Capacity,4} при Count = {numbers.Count}");
        lastCapacity = numbers.Capacity;
    }
}

// Видаляємо половину: надгробки накопичуються, потім прибираються при рехешуванні.
for (int i = 0; i < 1000; i += 2) numbers.Remove(i);
Console.WriteLine($"Після видалення парних: Count = {numbers.Count}, Tombstones = {numbers.Tombstones}");
for (int i = 1000; i < 1300; i++) numbers[i] = 0;
Console.WriteLine($"Після ще 300 вставок:    Count = {numbers.Count}, Tombstones = {numbers.Tombstones}, Capacity = {numbers.Capacity}");
Console.WriteLine($"numbers[999] = {numbers[999]}, ContainsKey(998) = {numbers.ContainsKey(998)}");
Console.WriteLine($"Сума ключів через перелічувач: {numbers.Sum(p => (long)p.Key)}");

public sealed class OrdinalFnvComparer : IEqualityComparer<string>
{
    public bool Equals(string? x, string? y) => string.Equals(x, y, StringComparison.Ordinal);

    public int GetHashCode([DisallowNull] string s)
    {
        uint h = 2166136261;
        foreach (char c in s) h = unchecked((h ^ c) * 16777619);
        return (int)h;
    }
}

public sealed class OpenHashMap<TKey, TValue> : IEnumerable<KeyValuePair<TKey, TValue>>
    where TKey : notnull
{
    private const int MinCapacity = 8;
    private const int MaxLoadNumerator = 3;   // поріг 3/4 — у цілих числах,
    private const int MaxLoadDenominator = 4; // без double

    private enum SlotState : byte { Empty, Occupied, Deleted }

    private readonly IEqualityComparer<TKey> _comparer;
    private SlotState[] _states;
    private int[] _hashes;
    private TKey[] _keys;
    private TValue[] _values;
    private int _count;
    private int _tombstones;
    private int _version;

    public OpenHashMap(IEqualityComparer<TKey>? comparer = null, int capacity = MinCapacity)
    {
        _comparer = comparer ?? EqualityComparer<TKey>.Default;
        int size = RoundUpToPowerOfTwo(Math.Max(capacity, MinCapacity));
        _states = new SlotState[size];
        _hashes = new int[size];
        _keys = new TKey[size];
        _values = new TValue[size];
    }

    public int Count => _count;
    public int Capacity => _states.Length;
    public int Tombstones => _tombstones;

    // ---------- Публічний API ----------

    public TValue this[TKey key]
    {
        get
        {
            int slot = FindExisting(key);
            return slot >= 0 ? _values[slot] : throw new KeyNotFoundException($"Ключ '{key}' відсутній");
        }
        set => Insert(key, value, overwrite: true);
    }

    public void Add(TKey key, TValue value) => Insert(key, value, overwrite: false);

    public bool ContainsKey(TKey key) => FindExisting(key) >= 0;

    public bool TryGetValue(TKey key, [MaybeNullWhen(false)] out TValue value)
    {
        int slot = FindExisting(key);
        if (slot < 0)
        {
            value = default;
            return false;
        }
        value = _values[slot];
        return true;
    }

    public bool Remove(TKey key)
    {
        int slot = FindExisting(key);
        if (slot < 0) return false;

        _states[slot] = SlotState.Deleted; // надгробок: ланцюг проб не розривається
        _keys[slot] = default!;            // звільняємо посилання для GC
        _values[slot] = default!;
        _count--;
        _tombstones++;
        _version++;
        return true;
    }

    public IEnumerator<KeyValuePair<TKey, TValue>> GetEnumerator()
    {
        int version = _version;
        for (int i = 0; i < _states.Length; i++)
        {
            if (version != _version)
                throw new InvalidOperationException("Колекцію змінено під час перелічення");
            if (_states[i] == SlotState.Occupied)
                yield return new KeyValuePair<TKey, TValue>(_keys[i], _values[i]);
        }
    }

    IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();

    // ---------- Крок 2: хеш і пробування ----------

    // Перемішуємо біти, бо індекс береться маскою (молодші біти).
    private int Hash(TKey key)
    {
        uint h = (uint)_comparer.GetHashCode(key);
        h ^= h >> 16;
        h = unchecked(h * 0x85EBCA6B);
        h ^= h >> 13;
        h = unchecked(h * 0xC2B2AE35);
        h ^= h >> 16;
        return (int)h;
    }

    // Повертає індекс існуючого ключа або −1.
    private int FindExisting(TKey key)
    {
        int hash = Hash(key);
        int mask = _states.Length - 1;
        for (int i = hash & mask, probes = 0; probes < _states.Length; i = (i + 1) & mask, probes++)
        {
            switch (_states[i])
            {
                case SlotState.Empty:
                    return -1; // порожня комірка завершує ланцюг проб
                case SlotState.Occupied when _hashes[i] == hash && _comparer.Equals(_keys[i], key):
                    return i;
            }
        }
        return -1;
    }

    // Повертає (індекс, знайдено?): або існуючий ключ, або найкраще місце для вставки.
    private (int Slot, bool Found) FindSlotForInsert(TKey key, int hash)
    {
        int mask = _states.Length - 1;
        int firstTombstone = -1;
        for (int i = hash & mask, probes = 0; probes < _states.Length; i = (i + 1) & mask, probes++)
        {
            switch (_states[i])
            {
                case SlotState.Empty:
                    return (firstTombstone >= 0 ? firstTombstone : i, false);
                case SlotState.Deleted:
                    if (firstTombstone < 0) firstTombstone = i;
                    break;
                case SlotState.Occupied when _hashes[i] == hash && _comparer.Equals(_keys[i], key):
                    return (i, true);
            }
        }
        return (firstTombstone, false); // таблиця без Empty — лише надгробки
    }

    // ---------- Крок 3: вставка ----------

    private void Insert(TKey key, TValue value, bool overwrite)
    {
        // Рехешуємо ЗАЗДАЛЕГІДЬ, щоб після вставки поріг не був перевищений.
        if ((_count + _tombstones + 1) * MaxLoadDenominator > _states.Length * MaxLoadNumerator)
            Resize();

        int hash = Hash(key);
        (int slot, bool found) = FindSlotForInsert(key, hash);
        if (found)
        {
            if (!overwrite) throw new ArgumentException($"Ключ '{key}' уже існує");
            _values[slot] = value;
            _version++;
            return;
        }

        if (_states[slot] == SlotState.Deleted) _tombstones--; // перевикористали надгробок
        _states[slot] = SlotState.Occupied;
        _hashes[slot] = hash;
        _keys[slot] = key;
        _values[slot] = value;
        _count++;
        _version++;
    }

    // ---------- Крок 4: рехешування ----------

    private void Resize()
    {
        // Якщо місце "з'їли" надгробки — достатньо перебудувати таблицю того ж розміру.
        int newSize = (_count + 1) * MaxLoadDenominator > _states.Length * MaxLoadNumerator / 2
            ? _states.Length * 2
            : _states.Length;

        SlotState[] oldStates = _states;
        int[] oldHashes = _hashes;
        TKey[] oldKeys = _keys;
        TValue[] oldValues = _values;

        _states = new SlotState[newSize];
        _hashes = new int[newSize];
        _keys = new TKey[newSize];
        _values = new TValue[newSize];
        _tombstones = 0;
        int mask = newSize - 1;

        for (int i = 0; i < oldStates.Length; i++)
        {
            if (oldStates[i] != SlotState.Occupied) continue; // надгробки не переносимо
            int j = oldHashes[i] & mask;                       // хеш кешований — не перераховуємо
            while (_states[j] == SlotState.Occupied) j = (j + 1) & mask;
            _states[j] = SlotState.Occupied;
            _hashes[j] = oldHashes[i];
            _keys[j] = oldKeys[i];
            _values[j] = oldValues[i];
        }
        _version++;
    }

    private static int RoundUpToPowerOfTwo(int value)
    {
        int result = 1;
        while (result < value) result <<= 1;
        return result;
    }
}
```

**Приклад запуску:**
```
Count = 4, Capacity = 8
ages["Марія"] = 22
TryGetValue("Петро") = False
Add дубліката: Ключ 'Іван' уже існує
Індексатор: Ключ 'Петро' відсутній
Remove("Олена") = True, Tombstones = 1
  рехешування:    8 →   16 при Count = 7
  рехешування:   16 →   32 при Count = 13
  рехешування:   32 →   64 при Count = 25
  рехешування:   64 →  128 при Count = 49
  рехешування:  128 →  256 при Count = 97
  рехешування:  256 →  512 при Count = 193
  рехешування:  512 → 1024 при Count = 385
  рехешування: 1024 → 2048 при Count = 769
Після видалення парних: Count = 500, Tombstones = 500
Після ще 300 вставок:    Count = 800, Tombstones = 396, Capacity = 2048
numbers[999] = 998001, ContainsKey(998) = False
Сума ключів через перелічувач: 594850
```

### 7.5. Що варто помітити

1. **Рехешування заздалегідь** (перед вставкою) — спрощує логіку: після `Resize` гарантовано є вільна комірка.
2. **Рехешування того самого розміру** — коли таблиця «забита» надгробками, а живих мало. Це прибирає надгробки без зайвої пам'яті.
3. **Очищення `_keys[slot]` і `_values[slot]` при видаленні** — інакше GC не зможе зібрати об'єкти, на які посилається «мертва» комірка (витік пам'яті).
4. **`[MaybeNullWhen(false)]`** — коректна анотація для nullable-аналізу, як у `Dictionary.TryGetValue`.

### Міні-вправа 7.1

Додайте до `OpenHashMap` метод `GetOrAdd(TKey key, Func<TKey, TValue> factory)`, який обчислює хеш **один раз** і не робить подвійного пошуку.

<details>
<summary>Розв'язок</summary>

```csharp
public TValue GetOrAdd(TKey key, Func<TKey, TValue> factory)
{
    if ((_count + _tombstones + 1) * MaxLoadDenominator > _states.Length * MaxLoadNumerator)
        Resize();

    int hash = Hash(key);                              // один виклик GetHashCode
    (int slot, bool found) = FindSlotForInsert(key, hash); // один прохід проб
    if (found) return _values[slot];

    TValue value = factory(key);
    if (_states[slot] == SlotState.Deleted) _tombstones--;
    _states[slot] = SlotState.Occupied;
    _hashes[slot] = hash;
    _keys[slot] = key;
    _values[slot] = value;
    _count++;
    _version++;
    return value;
}
```

Увага: якщо `factory` модифікує цю ж таблицю, знайдений `slot` стане недійсним — у промисловому коді це перевіряють через `_version`. У .NET аналогічну задачу розв'язує `CollectionsMarshal.GetValueRefOrAddDefault`.
</details>

---

## 8. Dictionary у .NET: внутрішня будова та контракт GetHashCode/Equals

### 8.1. Коротке нагадування API

(Детально — у Лекції 3.)

| Метод | Складність | Примітка |
|-------|-----------|----------|
| `dict[key] = value` | O(1) амортизовано | додає або замінює |
| `dict.Add(key, value)` | O(1) амортизовано | кидає, якщо ключ є |
| `dict.TryGetValue(key, out v)` | O(1) | **один** пошук замість `ContainsKey` + `[]` |
| `dict.TryAdd(key, value)` | O(1) | не кидає |
| `dict.Remove(key)` | O(1) | |
| `CollectionsMarshal.GetValueRefOrAddDefault` | O(1) | `ref` на значення — оновлення без повторного пошуку |
| `new Dictionary<K,V>(capacity)` | O(capacity) | уникнути рехешувань |
| `EnsureCapacity(n)`, `TrimExcess()` | O(n) | керування розміром |

### 8.2. Внутрішня будова: `_buckets` + `_entries`

`Dictionary<TKey,TValue>` — це **ланцюжки, але без вузлів-об'єктів**. Ланцюжки «вшиті» в масив структур через індекси:

```csharp
// Спрощено з вихідного коду .NET (System.Private.CoreLib/Dictionary.cs)
private int[]? _buckets;          // _buckets[i] = (індекс першого entry у ланцюжку) + 1; 0 = порожньо
private Entry[]? _entries;        // щільний масив записів
private int _count;               // скільки комірок _entries використано
private int _freeList;            // голова списку вільних entries (після Remove)
private int _freeCount;

private struct Entry
{
    public uint hashCode;
    public int next;              // індекс наступного entry в ланцюжку; -1 = кінець
    public TKey key;
    public TValue value;
}
```

```
Dictionary з 3 кошиками (просте число), вставлено A, B, C; hash(A)%3=1, hash(B)%3=1, hash(C)%3=0

 _buckets (int[3], зберігає index+1)        _entries (Entry[3])
 ┌───┐                                    ┌─────┬────────┬──────┬───────┐
 │ 3 │ ──► entry 2 ─────────────────────► │ [0] │ A      │ next │  -1   │
 ├───┤                                    ├─────┼────────┼──────┼───────┤
 │ 2 │ ──► entry 1 ──► next=0 ──► entry 0 │ [1] │ B      │ next │   0   │
 ├───┤                                    ├─────┼────────┼──────┼───────┤
 │ 0 │  (порожньо)                        │ [2] │ C      │ next │  -1   │
 └───┘                                    └─────┴────────┴──────┴───────┘
```

**Переваги такої схеми:**

- **Жодної алокації на елемент** — лише два масиви; `Entry` — структура.
- **Щільний `_entries`** → швидкий перелік (без пропуску порожніх кошиків).
- **Порядок перелічення = порядок вставки**, поки не було `Remove` (але це **деталь реалізації**, не гарантія!).
- **`Remove`** додає entry у `_freeList`; наступна вставка перевикористає цю комірку (тому після `Remove` + `Add` порядок змінюється).

### 8.3. Прості розміри, рехешування, рандомізоване хешування рядків і HashDoS

- **Розміри — прості числа** з таблиці `HashHelpers.Primes` (3, 7, 11, 17, 23, 29, 37, 47, 59, 71, 89, 107, 131, 163, 197, …). При переповненні `_entries` розмір стає **найменшим простим ≥ 2 × старий розмір**.
- Індекс: `hashCode % _buckets.Length`, на 64-бітних платформах через **FastMod** (множення на попередньо обчислений обернений множник).
- Рехешування викликається, коли **`_entries` заповнено** (тобто α досягає 1.0 — ланцюжки допускають це).

**HashDoS (Hash flooding).** Якщо зловмисник знає хеш-функцію, він може надіслати тисячі ключів **з однаковим хешем** (наприклад, у JSON-запиті чи параметрах форми). Таблиця деградує до списку: n вставок коштують O(n²). У 2011 році так «клали» веб-сервери на PHP, Java, Python, ASP.NET.

**Захист у .NET:**

1. `string.GetHashCode()` у .NET Core **рандомізований**: використовується Marvin hash із випадковим 64-бітним зерном, яке генерується **при старті процесу**. Тому хеш-коди рядків **різні між запусками** — ніколи не зберігайте їх і не друкуйте в тестах.
2. `Dictionary<string, TValue>` зі стандартним компаратором спочатку використовує **швидкий нерандомізований** хеш, а коли довжина ланцюжка перевищує поріг (100 колізій), **перемикається** на рандомізований компаратор і рехешує таблицю.

```csharp
// Атака на хеш-таблицю: усі ключі мають однаковий хеш → O(n²).
// Використовуємо "поганий" компаратор, щоб імітувати зловмисні ключі.
using System.Diagnostics;

foreach (int n in new[] { 2_000, 4_000, 8_000 })
{
    long goodComparisons = Fill(n, new CountingComparer(constantHash: false));
    long badComparisons = Fill(n, new CountingComparer(constantHash: true));
    Console.WriteLine($"n = {n,5}: порівнянь Equals — нормальний хеш {goodComparisons,9}, однаковий хеш {badComparisons,11}");
}

static long Fill(int n, CountingComparer comparer)
{
    var dict = new Dictionary<int, int>(comparer);
    for (int i = 0; i < n; i++) dict[i] = i;
    Debug.Assert(dict.Count == n);
    return comparer.EqualsCalls;
}

public sealed class CountingComparer(bool constantHash) : IEqualityComparer<int>
{
    public long EqualsCalls { get; private set; }

    public bool Equals(int x, int y)
    {
        EqualsCalls++;
        return x == y;
    }

    public int GetHashCode(int value) => constantHash ? 42 : value; // 42 — "атакуючий" хеш
}
```

**Приклад запуску:**
```
n =  2000: порівнянь Equals — нормальний хеш         0, однаковий хеш     1999000
n =  4000: порівнянь Equals — нормальний хеш         0, однаковий хеш     7998000
n =  8000: порівнянь Equals — нормальний хеш         0, однаковий хеш    31996000
```

Подвоєння `n` — **учетверо** більше порівнянь: класична квадратична деградація.

### 8.4. Контракт `GetHashCode` / `Equals`

Для будь-якого типу, що використовується як ключ:

| № | Правило | Наслідок порушення |
|---|---------|--------------------|
| 1 | `a.Equals(b) == true` ⇒ `a.GetHashCode() == b.GetHashCode()` | рівні ключі в різних кошиках → «не знаходить» |
| 2 | `GetHashCode()` не змінюється, поки об'єкт у таблиці | ключ «губиться» (див. 8.8) |
| 3 | `Equals` рефлексивний, симетричний, транзитивний | непередбачувані результати |
| 4 | Однаковий хеш **не** означає рівності | (це не правило, а нагадування: колізії — норма) |
| 5 | `GetHashCode` і `Equals` не кидають винятків | |

**За замовчуванням:**

- `class` — **посилальна** рівність: два різні об'єкти з однаковими полями **не рівні**.
- `struct` — `ValueType.Equals` порівнює поля **через рефлексію** (повільно!), а `GetHashCode` може використовувати лише **перше поле** → погана рівномірність. Завжди перевизначайте або використовуйте `record struct`.

```csharp
// Порушення контракту: Equals перевизначено, GetHashCode — ні.
var broken = new HashSet<PointBroken> { new(1, 2) };
var correct = new HashSet<PointCorrect> { new(1, 2) };

Console.WriteLine($"PointBroken(1,2).Equals(PointBroken(1,2)) = {new PointBroken(1, 2).Equals(new PointBroken(1, 2))}");
Console.WriteLine($"broken.Contains(new PointBroken(1,2))  = {broken.Contains(new PointBroken(1, 2))}  ← хеші різні (адреси об'єктів)");
Console.WriteLine($"correct.Contains(new PointCorrect(1,2)) = {correct.Contains(new PointCorrect(1, 2))}");

#pragma warning disable CS0659 // навмисно не перевизначаємо GetHashCode — демонстрація помилки
public sealed class PointBroken(int x, int y)
{
    public int X { get; } = x;
    public int Y { get; } = y;

    public override bool Equals(object? obj) => obj is PointBroken p && p.X == X && p.Y == Y;
}
#pragma warning restore CS0659

public sealed class PointCorrect(int x, int y) : IEquatable<PointCorrect>
{
    public int X { get; } = x;
    public int Y { get; } = y;

    public bool Equals(PointCorrect? other) => other is not null && other.X == X && other.Y == Y;
    public override bool Equals(object? obj) => Equals(obj as PointCorrect);
    public override int GetHashCode() => HashCode.Combine(X, Y); // узгоджено з Equals
}
```

**Приклад запуску:**
```
PointBroken(1,2).Equals(PointBroken(1,2)) = True
broken.Contains(new PointBroken(1,2))  = False  ← хеші різні (адреси об'єктів)
correct.Contains(new PointCorrect(1,2)) = True
```

> Формально `broken.Contains` **може** випадково повернути `True`, якщо хеш-коди двох об'єктів збіглися, але на практиці — `False`.

### 8.5. `HashCode.Combine` та `HashCode.Add`

`System.HashCode` — вбудований комбінатор на основі **xxHash32** з **рандомізованим зерном процесу**. Переваги над ручним `x * 31 + y`:

- хороший лавинний ефект;
- стійкість до порядку полів (`Combine(1, 2) != Combine(2, 1)`);
- не треба думати про `unchecked` і `null`.

```csharp
// HashCode.Combine і HashCode.Add: властивості (без друку самих значень — вони рандомізовані).
Console.WriteLine($"Детермінованість у процесі: {HashCode.Combine(1, 2) == HashCode.Combine(1, 2)}");
Console.WriteLine($"Порядок має значення:       {HashCode.Combine(1, 2) != HashCode.Combine(2, 1)}");
Console.WriteLine($"null допускається:          {HashCode.Combine<string?, int>(null, 5) == HashCode.Combine<string?, int>(null, 5)}");

// Для колекцій довільної довжини — HashCode.Add у циклі.
int[] a = [1, 2, 3, 4, 5];
int[] b = [1, 2, 3, 4, 5];
Console.WriteLine($"Посилання на масиви рівні:  {a.GetHashCode() == b.GetHashCode()}  ← масив не перевизначає GetHashCode");
Console.WriteLine($"Хеш за вмістом рівний:      {SequenceHash(a) == SequenceHash(b)}");

// Ручна "класична" формула для порівняння — дає колізії на простих шаблонах.
int collisions = 0;
var seen = new HashSet<int>();
for (int x = 0; x < 100; x++)
    for (int y = 0; y < 100; y++)
        if (!seen.Add(unchecked(x * 31 + y))) collisions++;
Console.WriteLine($"x*31+y для 100×100 точок: колізій {collisions} з 10000");

static int SequenceHash(int[] items)
{
    var hash = new HashCode();
    foreach (int item in items) hash.Add(item);
    return hash.ToHashCode();
}
```

**Приклад запуску:**
```
Детермінованість у процесі: True
Порядок має значення:       True
null допускається:          True
Посилання на масиви рівні:  False  ← масив не перевизначає GetHashCode
Хеш за вмістом рівний:      True
x*31+y для 100×100 точок: колізій 6831 з 10000
```

### 8.6. `record` і `record struct` — рівність «з коробки»

Компілятор генерує для записів `Equals`, `GetHashCode`, `==`, `!=`, `IEquatable<T>` на основі **усіх** полів.

```csharp
// Записи як ключі словника: рівність за значенням генерується компілятором.
var distances = new Dictionary<Route, int>
{
    [new Route("Київ", "Львів")] = 540,
    [new Route("Київ", "Одеса")] = 475,
};

var query = new Route("Київ", "Львів");  // НОВИЙ об'єкт з тими самими полями
Console.WriteLine($"Київ → Львів: {distances[query]} км");
Console.WriteLine($"ReferenceEquals: {ReferenceEquals(query, distances.Keys.First())}, Equals: {query == distances.Keys.First()}");

// record struct — те саме, але без алокації в купі.
var grid = new HashSet<Cell> { new(0, 0), new(1, 2), new(1, 2) };
Console.WriteLine($"Унікальних клітинок: {grid.Count}");

// Пастка: поле-масив у записі порівнюється ЗА ПОСИЛАННЯМ.
var tag1 = new Tagged("x", [1, 2]);
var tag2 = new Tagged("x", [1, 2]);
Console.WriteLine($"Записи з однаковими масивами рівні? {tag1 == tag2}");

public sealed record Route(string From, string To);
public readonly record struct Cell(int Row, int Col);
public sealed record Tagged(string Name, int[] Values);
```

**Приклад запуску:**
```
Київ → Львів: 540 км
ReferenceEquals: False, Equals: True
Унікальних клітинок: 2
Записи з однаковими масивами рівні? False
```

### 8.7. `IEqualityComparer<T>` — «зовнішня» рівність

Коли не можна (або не треба) змінювати сам тип, рівність і хеш задають окремим об'єктом:

```csharp
// Власні компаратори: регістронезалежні рядки, рівність масивів за вмістом, ключ за полем.
var headers = new Dictionary<string, string>(StringComparer.OrdinalIgnoreCase)
{
    ["Content-Type"] = "application/json",
};
Console.WriteLine($"headers[\"content-type\"] = {headers["content-type"]}");

var byContent = new HashSet<int[]>(new ArrayContentComparer()) { new[] { 1, 2, 3 } };
Console.WriteLine($"Contains([1,2,3]) за вмістом = {byContent.Contains([1, 2, 3])}");
Console.WriteLine($"Contains([3,2,1]) за вмістом = {byContent.Contains([3, 2, 1])}");

var students = new HashSet<Student>(new StudentIdComparer())
{
    new("S-001", "Олена"),
    new("S-002", "Тарас"),
    new("S-001", "Олена Коваль"), // той самий Id → дублікат
};
Console.WriteLine($"Студентів за Id: {students.Count}");

public sealed record Student(string Id, string Name);

public sealed class StudentIdComparer : IEqualityComparer<Student>
{
    public bool Equals(Student? x, Student? y) => x?.Id == y?.Id;
    public int GetHashCode(Student s) => s.Id.GetHashCode(); // узгоджено: хешуємо ТІЛЬКИ поле, яке порівнюємо
}

public sealed class ArrayContentComparer : IEqualityComparer<int[]>
{
    public bool Equals(int[]? x, int[]? y) =>
        ReferenceEquals(x, y) || (x is not null && y is not null && x.AsSpan().SequenceEqual(y));

    public int GetHashCode(int[] array)
    {
        var hash = new HashCode();
        foreach (int item in array) hash.Add(item);
        return hash.ToHashCode();
    }
}
```

**Приклад запуску:**
```
headers["content-type"] = application/json
Contains([1,2,3]) за вмістом = True
Contains([3,2,1]) за вмістом = False
Студентів за Id: 2
```

### 8.8. Баг мутабельного ключа

Якщо поле, від якого залежить `GetHashCode`, змінюється **після** додавання в таблицю, об'єкт лишається в **старому** кошику, а шукається в **новому**.

```
Add(key {Name="Anna"})           hash → кошик 3     [3]: key
key.Name = "Boris"               об'єкт той самий, але хеш тепер → кошик 7
Contains(key)                    шукаємо в [7] → порожньо → False   (хоча об'єкт у таблиці!)
Remove(key)                      теж шукає в [7] → False → "витік" запису
```

```csharp
// Баг мутабельного ключа: змінили поле після вставки — ключ "загубився".
var user = new MutableUser { Email = "anna@example.com" };
var sessions = new HashSet<MutableUser> { user };

Console.WriteLine($"До зміни:    Contains = {sessions.Contains(user)}");
user.Email = "boris@example.com";  // змінюємо поле, що бере участь у GetHashCode
Console.WriteLine($"Після зміни: Contains = {sessions.Contains(user)}, Count = {sessions.Count}");
Console.WriteLine($"Remove(user) = {sessions.Remove(user)}  ← запис неможливо видалити");
Console.WriteLine($"Але перелічення його бачить: {sessions.Single().Email}");

user.Email = "anna@example.com";   // повернули старе значення — знову знаходиться
Console.WriteLine($"Повернули email: Contains = {sessions.Contains(user)}");

// Рішення: незмінний ключ (record з init-властивостями).
var fixedKey = new ImmutableUser("anna@example.com");
var changed = fixedKey with { Email = "boris@example.com" }; // створює НОВИЙ об'єкт
var safeSessions = new HashSet<ImmutableUser> { fixedKey };
Console.WriteLine($"Незмінний ключ: Contains(original) = {safeSessions.Contains(fixedKey)}, Contains(changed) = {safeSessions.Contains(changed)}");

public sealed class MutableUser
{
    public required string Email { get; set; }

    public override bool Equals(object? obj) => obj is MutableUser u && u.Email == Email;
    public override int GetHashCode() => Email.GetHashCode();
}

public sealed record ImmutableUser(string Email);
```

**Приклад запуску:**
```
До зміни:    Contains = True
Після зміни: Contains = False, Count = 1
Remove(user) = False  ← запис неможливо видалити
Але перелічення його бачить: boris@example.com
Повернули email: Contains = True
Незмінний ключ: Contains(original) = True, Contains(changed) = False
```

**Правила:**

- Ключі мають бути **незмінними** (`record`, `readonly struct`, `string`, `init`-властивості).
- Якщо об'єкт мутабельний — хешуйте лише **незмінний ідентифікатор** (`Id`).
- Змінити ключ = `Remove(old)` → змінити → `Add(new)`.

### Типові помилки (.NET Dictionary)

- ❌ `if (dict.ContainsKey(k)) v = dict[k];` — **два** пошуки. Використовуйте `TryGetValue`.
- ❌ Перевизначити `Equals` без `GetHashCode` (компілятор попереджає CS0659 — не ігноруйте).
- ❌ `struct` як ключ без `IEquatable<T>` → боксинг і рефлексія на кожне порівняння.
- ❌ Покладатися на порядок перелічення `Dictionary`.
- ❌ Модифікувати `Dictionary` у `foreach` (виняток; виняток — `Remove` у .NET Core 3.0+, він дозволений).
- ❌ Спільний `Dictionary` з кількох потоків без синхронізації → може **зациклитися** при одночасному рехешуванні. Використовуйте `ConcurrentDictionary`.
- ❌ Друкувати/зберігати `string.GetHashCode()` — він рандомізований.

### Міні-вправа 8.1

Що виведе код і чому?

```csharp
var set = new HashSet<Money>();
set.Add(new Money(100, "UAH"));
Console.WriteLine(set.Contains(new Money(100, "uah")));

public readonly record struct Money(decimal Amount, string Currency);
```

<details>
<summary>Розв'язок</summary>

`False`. Рівність `record struct` генерується з **ординальним** порівнянням рядків, тож `"UAH" != "uah"`. Щоб ігнорувати регістр, або нормалізуйте валюту в конструкторі (`Currency.ToUpperInvariant()`), або передайте у `HashSet` власний `IEqualityComparer<Money>`, де і `Equals`, і `GetHashCode` використовують `StringComparer.OrdinalIgnoreCase`.
</details>

---

## 9. HashSet<T> і операції над множинами

`HashSet<T>` — це `Dictionary` **без значень**: та сама схема `_buckets` + `_entries`, O(1) `Add`/`Contains`/`Remove`.

| Операція | Метод (змінює множину) | Складність |
|----------|------------------------|-----------|
| A ∪ B | `a.UnionWith(b)` | O(\|B\|) |
| A ∩ B | `a.IntersectWith(b)` | O(\|A\|) якщо `b` — HashSet з тим самим компаратором, інакше O(\|A\| + \|B\|) |
| A \ B | `a.ExceptWith(b)` | O(\|B\|) |
| A △ B | `a.SymmetricExceptWith(b)` | O(\|B\|) |
| A ⊆ B | `a.IsSubsetOf(b)` | O(\|A\|) для HashSet |
| A ∩ B ≠ ∅ | `a.Overlaps(b)` | O(\|B\|) |

```csharp
// Операції над множинами на прикладі студентів, що записалися на курси.
string[] algorithms = ["Олена", "Тарас", "Марія", "Іван"];
string[] databases = ["Марія", "Іван", "Петро"];

Show("Алгоритми ∪ Бази даних", s => s.UnionWith(databases));
Show("Алгоритми ∩ Бази даних", s => s.IntersectWith(databases));
Show("Алгоритми \\ Бази даних", s => s.ExceptWith(databases));
Show("Алгоритми △ Бази даних", s => s.SymmetricExceptWith(databases));

var algoSet = new HashSet<string>(algorithms);
Console.WriteLine($"{{Марія, Іван}} ⊆ Алгоритми: {new HashSet<string> { "Марія", "Іван" }.IsSubsetOf(algoSet)}");
Console.WriteLine($"Алгоритми перетинається з {{Петро}}: {algoSet.Overlaps(["Петро"])}");

// Результат сортуємо: порядок у HashSet не гарантований.
void Show(string title, Action<HashSet<string>> operation)
{
    var set = new HashSet<string>(algorithms);
    operation(set);
    Console.WriteLine($"{title,-24}: {string.Join(", ", set.Order(StringComparer.Ordinal))}");
}
```

**Приклад запуску:**
```
Алгоритми ∪ Бази даних  : Іван, Марія, Олена, Петро, Тарас
Алгоритми ∩ Бази даних  : Іван, Марія
Алгоритми \ Бази даних  : Олена, Тарас
Алгоритми △ Бази даних  : Олена, Петро, Тарас
{Марія, Іван} ⊆ Алгоритми: True
Алгоритми перетинається з {Петро}: False
```

> Для **впорядкованої** множини — `SortedSet<T>` (червоно-чорне дерево, O(log n)), для незмінної — `FrozenSet<T>` (.NET 8+, оптимізована для читання: при створенні підбирає найкращу хеш-функцію під конкретні ключі).

### Міні-вправа 9.1

Дано два масиви по мільйону чисел. Чому `a.Intersect(b)` (LINQ) працює за O(n + m), а подвійний цикл `a.Where(x => b.Contains(x))` для масиву `b` — за O(n · m)?

<details>
<summary>Розв'язок</summary>

LINQ `Intersect` будує внутрішню хеш-множину з `b` (O(m)), а потім для кожного елемента `a` робить O(1) перевірку (O(n)). `b.Contains(x)` для **масиву** — лінійний пошук O(m), виконаний n разів. Для n = m = 10⁶ це 10¹² операцій проти ~2·10⁶.
</details>

---

## ☕ Перерва 3 (5 хв)

---


## 10. Застосування: класичні задачі

Хеш-таблиця — найчастіший «прискорювач» алгоритмів: вона перетворює внутрішній цикл пошуку O(n) на O(1). Типовий шаблон: **«чи бачили ми це раніше?»** або **«скільки разів ми це бачили?»**.

### 10.1. Підрахунок частот

```csharp
// Частоти слів у тексті: Dictionary<string,int> і CollectionsMarshal для одного пошуку.
using System.Runtime.InteropServices;

string text = "to be or not to be that is the question to be";
var frequency = new Dictionary<string, int>();

foreach (string word in text.Split(' '))
{
    // ref на значення: якщо ключа немає — створюється з default (0). Один пошук замість двох.
    ref int count = ref CollectionsMarshal.GetValueRefOrAddDefault(frequency, word, out _);
    count++;
}

foreach (var (word, count) in frequency.OrderByDescending(p => p.Value).ThenBy(p => p.Key, StringComparer.Ordinal).Take(4))
    Console.WriteLine($"{word,-8} {count}");

// Те саме через LINQ (зручно, але більше алокацій).
var top = text.Split(' ').CountBy(w => w).MaxBy(p => p.Value);
Console.WriteLine($"Найчастіше (CountBy): {top.Key} × {top.Value}");
```

**Приклад запуску:**
```
be       3
to       3
is       1
not      1
Найчастіше (CountBy): to × 3
```

### 10.2. Two Sum

**Задача.** Дано масив `nums` і число `target`. Знайти індекси двох елементів із сумою `target`.

- Наївно: подвійний цикл O(n²).
- З хешем: для кожного `x` перевіряємо, чи бачили `target − x`. **O(n) час, O(n) пам'ять.**

```
nums = [2, 7, 11, 15], target = 9

i=0: x=2, шукаємо 7 → немає;   seen = {2:0}
i=1: x=7, шукаємо 2 → є (0) →  відповідь (0, 1)
```

```csharp
// Two Sum за O(n): словник "значення → індекс".
Console.WriteLine(Format(TwoSum([2, 7, 11, 15], 9)));
Console.WriteLine(Format(TwoSum([3, 2, 4], 6)));
Console.WriteLine(Format(TwoSum([3, 3], 6)));
Console.WriteLine(Format(TwoSum([1, 2, 3], 100)));

static (int, int)? TwoSum(int[] nums, int target)
{
    var seen = new Dictionary<int, int>(nums.Length); // значення → індекс
    for (int i = 0; i < nums.Length; i++)
    {
        int complement = target - nums[i];
        if (seen.TryGetValue(complement, out int j))
            return (j, i);          // знайшли пару серед уже переглянутих
        seen[nums[i]] = i;          // додаємо ПІСЛЯ перевірки — щоб не взяти той самий елемент двічі
    }
    return null;
}

static string Format((int, int)? pair) => pair is { } p ? $"[{p.Item1}, {p.Item2}]" : "немає";
```

**Приклад запуску:**
```
[0, 1]
[1, 2]
[0, 1]
немає
```

### 10.3. Групування анаграм

**Задача.** Згрупувати слова, що є анаграмами одне одного. Ключ групи — **канонічна форма**: відсортовані літери (O(k log k)) або вектор частот (O(k)).

```csharp
// Групування анаграм: два способи побудувати канонічний ключ.
string[] words = ["eat", "tea", "tan", "ate", "nat", "bat"];

Print("Ключ — відсортовані літери", Group(words, SortedKey));
Print("Ключ — лічильник літер    ", Group(words, CountKey));

static List<List<string>> Group(string[] words, Func<string, string> keyOf)
{
    var groups = new Dictionary<string, List<string>>();
    foreach (string word in words)
    {
        string key = keyOf(word);
        if (!groups.TryGetValue(key, out var list))
            groups[key] = list = [];
        list.Add(word);
    }
    return groups.Values.ToList();
}

// "tea" → "aet": O(k log k)
static string SortedKey(string word)
{
    char[] letters = word.ToCharArray();
    Array.Sort(letters);
    return new string(letters);
}

// "tea" → "1#0#0#0#1#...": O(k), лише для латинських малих літер
static string CountKey(string word)
{
    Span<int> counts = stackalloc int[26];
    foreach (char c in word) counts[c - 'a']++;
    return string.Join('#', counts.ToArray());
}

static void Print(string title, List<List<string>> groups) =>
    Console.WriteLine($"{title}: " + string.Join(" | ", groups.Select(g => string.Join(",", g))));
```

**Приклад запуску:**
```
Ключ — відсортовані літери: eat,tea,ate | tan,nat | bat
Ключ — лічильник літер    : eat,tea,ate | tan,nat | bat
```

### 10.4. Перший неповторюваний символ

```csharp
// Перший символ, що зустрічається рівно один раз. Два проходи: підрахунок, потім пошук.
foreach (string s in new[] { "leetcode", "loveleetcode", "aabb", "хешування" })
{
    int index = FirstUniqueChar(s);
    Console.WriteLine(index >= 0 ? $"\"{s}\" → індекс {index} ('{s[index]}')" : $"\"{s}\" → немає");
}

static int FirstUniqueChar(string s)
{
    var counts = new Dictionary<char, int>();
    foreach (char c in s)
        counts[c] = counts.GetValueOrDefault(c) + 1;   // прохід 1: частоти

    for (int i = 0; i < s.Length; i++)
        if (counts[s[i]] == 1) return i;               // прохід 2: перший з частотою 1
    return -1;
}
```

**Приклад запуску:**
```
"leetcode" → індекс 0 ('l')
"loveleetcode" → індекс 2 ('v')
"aabb" → немає
"хешування" → індекс 0 ('х')
```

> Якщо алфавіт відомий і малий (наприклад, лише `'a'..'z'`), замість словника беріть `int[26]` — це та сама **пряма адресація** з розділу 1.

### 10.5. Найдовший підрядок без повторів (ковзне вікно + словник)

Тримаємо вікно `[left, right]` без повторів і словник **«символ → остання позиція»**. Якщо `s[right]` уже є у вікні — стрибком переносимо `left` за його попередню позицію.

```
s = "abcabcbb"

right=0 'a'  last={a:0}          вікно "a"      len 1
right=1 'b'  last={a:0,b:1}      вікно "ab"     len 2
right=2 'c'  last={..,c:2}       вікно "abc"    len 3  ← max
right=3 'a'  'a' був у 0 ≥ left → left=1, вікно "bca"   len 3
right=4 'b'  'b' був у 1 ≥ left → left=2, вікно "cab"   len 3
right=5 'c'  'c' був у 2 ≥ left → left=3, вікно "abc"   len 3
right=6 'b'  'b' був у 4 ≥ left → left=5, вікно "cb"    len 2
right=7 'b'  'b' був у 6 ≥ left → left=7, вікно "b"     len 1
```

```csharp
// Найдовший підрядок без повторюваних символів за O(n).
foreach (string s in new[] { "abcabcbb", "bbbbb", "pwwkew", "abba", "" })
{
    (int start, int length) = LongestUniqueSubstring(s);
    Console.WriteLine($"\"{s}\" → {length} (\"{s.Substring(start, length)}\")");
}

static (int Start, int Length) LongestUniqueSubstring(string s)
{
    var lastSeen = new Dictionary<char, int>(); // символ → останній індекс
    int left = 0, bestStart = 0, bestLength = 0;

    for (int right = 0; right < s.Length; right++)
    {
        // Зсуваємо left, лише якщо попереднє входження ВСЕРЕДИНІ вікна.
        // Без перевірки ">= left" рядок "abba" дасть неправильну відповідь (left відкотиться назад).
        if (lastSeen.TryGetValue(s[right], out int previous) && previous >= left)
            left = previous + 1;

        lastSeen[s[right]] = right;

        if (right - left + 1 > bestLength)
        {
            bestLength = right - left + 1;
            bestStart = left;
        }
    }
    return (bestStart, bestLength);
}
```

**Приклад запуску:**
```
"abcabcbb" → 3 ("abc")
"bbbbb" → 1 ("b")
"pwwkew" → 3 ("wke")
"abba" → 2 ("ab")
"" → 0 ("")
```

### 10.6. Кількість підмасивів із сумою k (префіксні суми + хеш)

**Ідея.** `prefix[i]` — сума перших `i` елементів. Сума підмасиву `(j, i]` дорівнює `prefix[i] − prefix[j]`. Отже, для кожного `i` треба знати, **скільки** було префіксів, рівних `prefix[i] − k`. Зберігаємо частоти префіксних сум у словнику.

```
nums = [1, 2, 3, -2, 2], k = 3

i   num  prefix  шукаємо prefix−k   знайдено   counts після
-   -     0         -                 -        {0:1}
0   1     1        -2                 0        {0:1, 1:1}
1   2     3         0                 1  ([1,2])            {.., 3:1}
2   3     6         3                 1  ([3])              {.., 6:1}
3  -2     4         1                 1  ([2,3,-2])         {.., 4:1}
4   2     6         3                 1  ([3,-2,2])         {.., 6:2}
                                     ─────
                                     разом 4
```

Працює і з **від'ємними** числами (на відміну від ковзного вікна). O(n) час.

```csharp
// Кількість неперервних підмасивів із сумою k: префіксні суми + словник частот.
Console.WriteLine(SubarraySum([1, 1, 1], 2));        // [1,1] двічі
Console.WriteLine(SubarraySum([1, 2, 3, -2, 2], 3));
Console.WriteLine(SubarraySum([3, 4, 7, 2, -3, 1, 4, 2], 7));
Console.WriteLine(SubarraySum([0, 0, 0], 0));        // 6 підмасивів із нулів

static int SubarraySum(int[] nums, int k)
{
    var prefixCounts = new Dictionary<long, int> { [0] = 1 }; // порожній префікс — важливо!
    long prefix = 0;
    int result = 0;

    foreach (int num in nums)
    {
        prefix += num;
        result += prefixCounts.GetValueOrDefault(prefix - k);        // скільки підмасивів закінчуються тут
        prefixCounts[prefix] = prefixCounts.GetValueOrDefault(prefix) + 1;
    }
    return result;
}
```

**Приклад запуску:**
```
2
4
4
6
```

### 10.7. LRU-кеш (Dictionary + LinkedList)

**LRU (Least Recently Used)** — кеш фіксованої ємності, що при переповненні викидає **найдавніше використаний** елемент. Вимога: `Get` і `Put` за **O(1)**.

- `Dictionary<TKey, LinkedListNode<...>>` — O(1) пошук вузла за ключем;
- `LinkedList` — O(1) переміщення вузла на початок і видалення з кінця.

```
capacity = 3

 Dictionary                 LinkedList (голова = найсвіжіший)
 ┌─────┬──────┐
 │ "a" │  ●───┼──────┐      ┌─────┐   ┌─────┐   ┌─────┐
 │ "b" │  ●───┼───┐  └────► │ a:1 │◄─►│ c:3 │◄─►│ b:2 │ ◄── кандидат на викидання
 │ "c" │  ●───┼─┐ │         └─────┘   └─────┘   └─────┘
 └─────┴──────┘ │ └────────────────────────────────▲
                └───────────────────▲
```

```csharp
// LRU-кеш з O(1) Get/Put.
var cache = new LruCache<string, int>(capacity: 3);
cache.Put("a", 1);
cache.Put("b", 2);
cache.Put("c", 3);
Console.WriteLine($"Порядок (свіжі → старі): {cache}");

Console.WriteLine($"Get(a) = {cache.GetOrDefault("a")}");   // a стає найсвіжішим
Console.WriteLine($"Порядок: {cache}");

cache.Put("d", 4);                                          // викидає b (найстаріший)
Console.WriteLine($"Put(d) → порядок: {cache}");
Console.WriteLine($"Get(b) = {cache.GetOrDefault("b", -1)}");

cache.Put("c", 30);                                         // оновлення існуючого
Console.WriteLine($"Put(c,30) → порядок: {cache}");

public sealed class LruCache<TKey, TValue>(int capacity) where TKey : notnull
{
    private readonly Dictionary<TKey, LinkedListNode<(TKey Key, TValue Value)>> _map = new(capacity);
    private readonly LinkedList<(TKey Key, TValue Value)> _order = new(); // First = найсвіжіший

    public TValue? GetOrDefault(TKey key, TValue? fallback = default)
    {
        if (!_map.TryGetValue(key, out var node))
            return fallback;
        _order.Remove(node);      // O(1): вузол знає своїх сусідів
        _order.AddFirst(node);    // O(1): робимо найсвіжішим
        return node.Value.Value;
    }

    public void Put(TKey key, TValue value)
    {
        if (_map.TryGetValue(key, out var existing))
        {
            _order.Remove(existing);
        }
        else if (_map.Count == capacity)
        {
            var oldest = _order.Last!;       // найдавніше використаний
            _order.RemoveLast();
            _map.Remove(oldest.Value.Key);   // тому в вузлі зберігаємо і ключ
        }

        _map[key] = _order.AddFirst((key, value));
    }

    public override string ToString() => string.Join(" ", _order.Select(e => $"{e.Key}:{e.Value}"));
}
```

**Приклад запуску:**
```
Порядок (свіжі → старі): c:3 b:2 a:1
Get(a) = 1
Порядок: a:1 c:3 b:2
Put(d) → порядок: d:4 a:1 c:3
Get(b) = -1
Put(c,30) → порядок: c:30 d:4 a:1
```

### 10.8. Узгоджене хешування (consistent hashing) — ідея

**Проблема.** Розподіляємо ключі між `N` серверами кешу за `hash(key) % N`. Додали сервер (`N → N+1`) — **майже всі** ключі змінили сервер (≈ N/(N+1) частка) → масові промахи кешу.

**Рішення.** Розмістимо і сервери, і ключі на **колі** хеш-значень `[0, 2³²)`. Ключ належить **першому серверу за годинниковою стрілкою**. Додавання/видалення сервера переносить лише ключі з **однієї дуги** — у середньому `1/N` частку.

```
                 0
             ┌───●───┐ S1
          ●k3        ● k1   → S2
      S3 ●              │
          │             ● S2
          ●k4       ●k2 → S3?  (перший сервер за годинниковою)
             └───────┘
```

**Віртуальні вузли:** кожен сервер розміщують на колі 100–200 разів (`"S1#0"`, `"S1#1"`, …) — інакше дуги нерівні й навантаження перекошене. Використовується в Amazon Dynamo, Cassandra, Memcached-клієнтах, CDN.

```csharp
// Порівняння: скільки ключів змінюють сервер при додаванні 5-го сервера.
// Узгоджене хешування — кільце на SortedDictionary з віртуальними вузлами.
string[] keys = Enumerable.Range(0, 10_000).Select(i => $"user:{i}").ToArray();
string[] servers4 = ["A", "B", "C", "D"];
string[] servers5 = ["A", "B", "C", "D", "E"];

int movedModulo = keys.Count(k => Fnv1a(k) % 4 != Fnv1a(k) % 5);
Console.WriteLine($"hash % N:           переміщено {movedModulo} з {keys.Length} ({(double)movedModulo / keys.Length:P0})");

var ring4 = new ConsistentHashRing(servers4, virtualNodes: 150);
var ring5 = new ConsistentHashRing(servers5, virtualNodes: 150);
int movedRing = keys.Count(k => ring4.GetServer(k) != ring5.GetServer(k));
Console.WriteLine($"узгоджене хешування: переміщено {movedRing} з {keys.Length} ({(double)movedRing / keys.Length:P0})");

Console.WriteLine("Навантаження на кільці з 5 серверів:");
foreach (var group in keys.GroupBy(ring5.GetServer).OrderBy(g => g.Key, StringComparer.Ordinal))
    Console.WriteLine($"  {group.Key}: {group.Count()}");

static uint Fnv1a(string s)
{
    uint h = 2166136261;
    foreach (char c in s) h = unchecked((h ^ c) * 16777619);
    // Фіналізатор для кращого перемішування коротких схожих рядків.
    h ^= h >> 16; h = unchecked(h * 0x85EBCA6B); h ^= h >> 13; h = unchecked(h * 0xC2B2AE35); h ^= h >> 16;
    return h;
}

public sealed class ConsistentHashRing
{
    private readonly SortedDictionary<uint, string> _ring = new(); // позиція на колі → сервер
    private readonly uint[] _positions;

    public ConsistentHashRing(IEnumerable<string> servers, int virtualNodes)
    {
        foreach (string server in servers)
            for (int v = 0; v < virtualNodes; v++)
                _ring[Hash($"{server}#{v}")] = server;
        _positions = [.. _ring.Keys]; // відсортовані позиції для бінарного пошуку
    }

    public string GetServer(string key)
    {
        uint h = Hash(key);
        int index = Array.BinarySearch(_positions, h);
        if (index < 0) index = ~index;             // перша позиція ≥ h
        if (index == _positions.Length) index = 0; // "загортання" кола
        return _ring[_positions[index]];
    }

    private static uint Hash(string s)
    {
        uint h = 2166136261;
        foreach (char c in s) h = unchecked((h ^ c) * 16777619);
        h ^= h >> 16; h = unchecked(h * 0x85EBCA6B); h ^= h >> 13; h = unchecked(h * 0xC2B2AE35); h ^= h >> 16;
        return h;
    }
}
```

**Приклад запуску:**
```
hash % N:           переміщено 7961 з 10000 (80%)
узгоджене хешування: переміщено 2436 з 10000 (24%)
Навантаження на кільці з 5 серверів:
  A: 1812
  B: 2030
  C: 1892
  D: 1830
  E: 2436
```

### 10.9. Пошук підрядка: алгоритм Рабіна–Карпа (rolling hash)

**Задача.** Знайти всі входження шаблону `P` (довжина m) у тексті `T` (довжина n).

**Ідея.** Порівнюємо **хеші** вікон довжини m, а посимвольно — лише при збігу хешів. Хеш наступного вікна отримуємо з попереднього за **O(1)** — «котимо» (rolling):

```
h("bcd") = (h("abc") − 'a'·B^(m−1)) · B + 'd'        (mod M)

 T:  a b c d e
     └─┬─┘                    h₀ = h("abc")
       └─┬─┘                  h₁ = (h₀ − 'a'·B²)·B + 'd'
         └─┬─┘                h₂ = (h₁ − 'b'·B²)·B + 'e'
```

Складність: **O(n + m)** в середньому, O(n·m) у гіршому (якщо багато хибних збігів хешів). Перевага — легко шукати **багато шаблонів однакової довжини** одночасно (хеші шаблонів у `HashSet`), а також знаходити повтори підрядків.

```csharp
// Рабін–Карп з поліноміальним rolling hash за модулем простого числа.
string text = "abracadabra cadabra abracadabra";
string pattern = "cadabra";

var (positions, spuriousHits) = RabinKarp(text, pattern);
Console.WriteLine($"\"{pattern}\" знайдено на позиціях: {string.Join(", ", positions)}");
Console.WriteLine($"Хибних збігів хешу: {spuriousHits}");

// Малий модуль → багато хибних збігів: видно, навіщо посимвольна перевірка.
var (positionsSmall, spuriousSmall) = RabinKarp(text, pattern, mod: 13);
Console.WriteLine($"Модуль 13: позиції {string.Join(", ", positionsSmall)}, хибних збігів {spuriousSmall}");

static (List<int> Positions, int SpuriousHits) RabinKarp(string text, string pattern, long mod = 1_000_000_007)
{
    const long B = 256;
    int n = text.Length, m = pattern.Length;
    var positions = new List<int>();
    int spurious = 0;
    if (m == 0 || m > n) return (positions, spurious);

    // B^(m−1) mod M — вага старшого символу, який "виходить" з вікна.
    long highPower = 1;
    for (int i = 0; i < m - 1; i++) highPower = highPower * B % mod;

    long patternHash = 0, windowHash = 0;
    for (int i = 0; i < m; i++)
    {
        patternHash = (patternHash * B + pattern[i]) % mod;
        windowHash = (windowHash * B + text[i]) % mod;
    }

    for (int start = 0; ; start++)
    {
        if (windowHash == patternHash)
        {
            // Хеші рівні — ще не означає рівності рядків. Перевіряємо посимвольно.
            if (text.AsSpan(start, m).SequenceEqual(pattern)) positions.Add(start);
            else spurious++;
        }
        if (start + m == n) break;

        // Котимо: прибрати text[start], додати text[start + m].
        windowHash = (windowHash - text[start] * highPower % mod + mod) % mod; // + mod — щоб не стати від'ємним
        windowHash = (windowHash * B + text[start + m]) % mod;
    }
    return (positions, spurious);
}
```

**Приклад запуску:**
```
"cadabra" знайдено на позиціях: 4, 12, 24
Хибних збігів хешу: 0
Модуль 13: позиції 4, 12, 24, хибних збігів 3
```

### 10.10. Фільтр Блума (Bloom filter)

**Фільтр Блума** — ймовірнісна структура для перевірки належності множині:

- `MightContain(x) == false` ⇒ `x` **точно немає**;
- `MightContain(x) == true` ⇒ `x` **можливо є** (хибнопозитивні спрацювання можливі, хибнонегативні — ні).

Займає **кілька бітів на елемент** (≈ 9.6 біта на 1% хибних спрацювань) незалежно від розміру самих елементів. Використання: перевірка «чи є ключ на диску» в LSM-деревах (RocksDB, Cassandra), фільтри шкідливих URL у браузерах, кеш-проксі.

```
Бітовий масив m = 16, k = 3 хеш-функції

Add("cat"):  h₁=2, h₂=7, h₃=13
Add("dog"):  h₁=5, h₂=7, h₃=11

 0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15
 0 0 1 0 0 1 0 1 0 0 0  1  0  1  0  0

Query("fox"): h₁=2, h₂=5, h₃=11 → усі біти = 1 → "можливо є"  ← хибнопозитивне!
Query("owl"): h₁=3, …           → біт 3 = 0     → "точно немає"
```

**Оптимальні параметри** для `n` елементів і бажаної ймовірності хибних спрацювань `p`:

```
m = −n · ln p / (ln 2)²        k = (m / n) · ln 2        p ≈ (1 − e^(−kn/m))^k
```

**`k` хеш-функцій з двох** (Kirsch–Mitzenmacher): `gᵢ(x) = h₁(x) + i · h₂(x)` — майже така ж якість, як у незалежних функцій.

> Детальніше про Bloom filter з точки зору ймовірнісних структур — у Лекції 9. Тут зосередимось на реалізації через хешування.

```csharp
using System.Collections;
using System.Text;

// Фільтр Блума: вимірюємо реальну частку хибнопозитивних і порівнюємо з теорією.
const int N = 10_000;
const double TargetFalsePositiveRate = 0.01;

var filter = BloomFilter.Create(N, TargetFalsePositiveRate);
Console.WriteLine($"m = {filter.BitCount} біт ({filter.BitCount / 8 / 1024.0:F1} КБ), k = {filter.HashCount}, біт на елемент = {(double)filter.BitCount / N:F1}");

for (int i = 0; i < N; i++) filter.Add($"user-{i}");

int falseNegatives = Enumerable.Range(0, N).Count(i => !filter.MightContain($"user-{i}"));
int falsePositives = Enumerable.Range(0, N).Count(i => filter.MightContain($"guest-{i}"));

Console.WriteLine($"Хибнонегативних: {falseNegatives} (завжди 0)");
Console.WriteLine($"Хибнопозитивних: {falsePositives} з {N} = {(double)falsePositives / N:P2} (ціль {TargetFalsePositiveRate:P0})");
Console.WriteLine($"Теоретична оцінка: {filter.EstimatedFalsePositiveRate(N):P2}");

public sealed class BloomFilter
{
    private readonly BitArray _bits;

    private BloomFilter(int bitCount, int hashCount)
    {
        _bits = new BitArray(bitCount);
        HashCount = hashCount;
    }

    public int BitCount => _bits.Length;
    public int HashCount { get; }

    // Підбір m і k за формулами для n елементів та ймовірності p.
    public static BloomFilter Create(int expectedItems, double falsePositiveRate)
    {
        double ln2 = Math.Log(2);
        int m = (int)Math.Ceiling(-expectedItems * Math.Log(falsePositiveRate) / (ln2 * ln2));
        int k = Math.Max(1, (int)Math.Round((double)m / expectedItems * ln2));
        return new BloomFilter(m, k);
    }

    public void Add(string item)
    {
        (uint h1, uint h2) = BaseHashes(item);
        for (int i = 0; i < HashCount; i++)
            _bits[IndexFor(h1, h2, i)] = true;
    }

    public bool MightContain(string item)
    {
        (uint h1, uint h2) = BaseHashes(item);
        for (int i = 0; i < HashCount; i++)
            if (!_bits[IndexFor(h1, h2, i)])
                return false; // хоч один нульовий біт → елемента точно не додавали
        return true;
    }

    public double EstimatedFalsePositiveRate(int insertedItems) =>
        Math.Pow(1 - Math.Exp(-(double)HashCount * insertedItems / BitCount), HashCount);

    // gᵢ = h₁ + i·h₂ (подвійне хешування, як у відкритій адресації).
    private int IndexFor(uint h1, uint h2, int i) => (int)(unchecked(h1 + (uint)i * h2) % (uint)_bits.Length);

    // Два незалежні 32-бітні хеші: FNV-1a з різними базисами + фіналізатор.
    private static (uint, uint) BaseHashes(string item)
    {
        byte[] bytes = Encoding.UTF8.GetBytes(item);
        return (Mix(Fnv1a(bytes, 2166136261)), Mix(Fnv1a(bytes, 0x9747B28C)) | 1); // h₂ непарний
    }

    private static uint Fnv1a(byte[] bytes, uint seed)
    {
        uint h = seed;
        foreach (byte b in bytes) h = unchecked((h ^ b) * 16777619);
        return h;
    }

    private static uint Mix(uint h)
    {
        h ^= h >> 16; h = unchecked(h * 0x85EBCA6B);
        h ^= h >> 13; h = unchecked(h * 0xC2B2AE35);
        return h ^ (h >> 16);
    }
}
```

**Приклад запуску:**
```
m = 95851 біт (11.7 КБ), k = 7, біт на елемент = 9.6
Хибнонегативних: 0 (завжди 0)
Хибнопозитивних: 81 з 10000 = 0.81% (ціль 1%)
Теоретична оцінка: 1.00%
```

**Обмеження:** з класичного фільтра Блума **не можна видаляти** (обнулення біта «зламає» інші елементи). Варіанти: *counting Bloom filter* (лічильники замість бітів), *cuckoo filter*.

### 10.11. Дедуплікація за хешем вмісту (SHA-256)

Для пошуку **однакових файлів / блоків даних** порівнювати вміст попарно — O(n²) порівнянь великих масивів. Замість цього рахуємо **криптографічний хеш** вмісту і групуємо за ним у словнику.

Чому саме криптографічний (SHA-256), а не FNV?

- 256 бітів → імовірність випадкової колізії для мільярда файлів ≈ `10¹⁸ / 2²⁵⁷` ≈ 10⁻⁵⁹ — практично нуль;
- **стійкість до навмисних колізій** — зловмисник не підсуне інший файл з тим самим хешем.

Так працюють Git (SHA-1 → SHA-256), Docker-шари, бекап-системи, Dropbox.

```csharp
using System.Security.Cryptography;
using System.Text;

// Дедуплікація "файлів" за SHA-256 вмісту: Dictionary<хеш, список імен>.
var files = new (string Name, string Content)[]
{
    ("report.txt", "Звіт за вересень"),
    ("photo.jpg", "ÿØbinary-image-data"),
    ("report-copy.txt", "Звіт за вересень"),
    ("notes.md", "# Нотатки"),
    ("backup/report.txt", "Звіт за вересень"),
    ("photo (1).jpg", "ÿØbinary-image-data"),
};

var byHash = new Dictionary<string, List<string>>();
foreach (var (name, content) in files)
{
    byte[] digest = SHA256.HashData(Encoding.UTF8.GetBytes(content)); // 32 байти
    string key = Convert.ToHexString(digest);                         // детермінований рядок-ключ
    if (!byHash.TryGetValue(key, out var names))
        byHash[key] = names = [];
    names.Add(name);
}

Console.WriteLine($"Файлів: {files.Length}, унікального вмісту: {byHash.Count}");
foreach (var (hash, names) in byHash.Where(p => p.Value.Count > 1))
    Console.WriteLine($"  {hash[..12]}… → {string.Join(", ", names)}");

Console.WriteLine($"SHA-256(\"abc\") = {Convert.ToHexString(SHA256.HashData("abc"u8))[..16]}…");
```

**Приклад запуску:**
```
Файлів: 6, унікального вмісту: 3
  B4DE7B7C2E89… → report.txt, report-copy.txt, backup/report.txt
  DC609B2972B0… → photo.jpg, photo (1).jpg
SHA-256("abc") = BA7816BF8F01CFEA…
```

> На практиці для великих файлів спочатку групують за **розміром** (дешево), потім за хешем перших 4 КБ, і лише потім — повний SHA-256.

### Міні-вправа 10.1 — Contains Duplicate II

Чи є в масиві два **однакові** елементи на відстані не більше `k` індексів? O(n).

<details>
<summary>Розв'язок</summary>

```csharp
// Ковзне вікно з HashSet розміру не більше k.
Console.WriteLine(ContainsNearbyDuplicate([1, 2, 3, 1], 3));       // True
Console.WriteLine(ContainsNearbyDuplicate([1, 0, 1, 1], 1));       // True
Console.WriteLine(ContainsNearbyDuplicate([1, 2, 3, 1, 2, 3], 2)); // False

static bool ContainsNearbyDuplicate(int[] nums, int k)
{
    var window = new HashSet<int>();
    for (int i = 0; i < nums.Length; i++)
    {
        if (!window.Add(nums[i])) return true; // Add повертає false, якщо елемент уже є
        if (window.Count > k) window.Remove(nums[i - k]); // тримаємо лише k останніх
    }
    return false;
}
```

**Приклад запуску:**
```
True
True
False
```
</details>

### Міні-вправа 10.2 — Longest Consecutive Sequence

Знайти довжину найдовшої послідовності **послідовних цілих** (у будь-якому порядку в масиві) за O(n). Приклад: `[100, 4, 200, 1, 3, 2]` → 4 (`1,2,3,4`).

<details>
<summary>Розв'язок</summary>

```csharp
// Починаємо рахувати лише з "початку" послідовності (x−1 немає в множині) — кожен елемент обробляється O(1) разів.
Console.WriteLine(LongestConsecutive([100, 4, 200, 1, 3, 2]));
Console.WriteLine(LongestConsecutive([0, 3, 7, 2, 5, 8, 4, 6, 0, 1]));

static int LongestConsecutive(int[] nums)
{
    var set = new HashSet<int>(nums);
    int best = 0;
    foreach (int x in set)
    {
        if (set.Contains(x - 1)) continue; // не початок послідовності
        int length = 1;
        while (set.Contains(x + length)) length++;
        best = Math.Max(best, length);
    }
    return best;
}
```

**Приклад запуску:**
```
4
9
```
</details>

---

## 11. Досконале хешування та хешування зозулі

### 11.1. Досконале хешування (perfect hashing)

Якщо множина ключів **статична** (відома наперед і не змінюється — ключові слова мови, словник, таблиця маршрутизації), можна побудувати хеш-функцію **без жодної колізії** → пошук **O(1) у гіршому випадку**.

**Схема FKS (Fredman–Komlós–Szemerédi), двохрівнева:**

```
Рівень 1: n ключів → m = n кошиків звичайною універсальною хеш-функцією
Рівень 2: кошик j з nⱼ ключами → власна таблиця розміру nⱼ² з окремою функцією hⱼ,
          підібраною випадково так, щоб НЕ було колізій (за парадоксом днів народження
          для nⱼ² комірок і nⱼ ключів імовірність колізії < 1/2 → кілька спроб)

 ┌───┐
 │ 0 │──► [ k5 ]                             n₀=1 → таблиця 1
 │ 1 │──► ∅
 │ 2 │──► [ · k1 · k7 ]                      n₂=2 → таблиця 4, h₂ підібрана
 │ 3 │──► [ · · k2 · · k3 · · k9 ]           n₃=3 → таблиця 9, h₃ підібрана
 └───┘
```

Загальна пам'ять `Σ nⱼ²` = **O(n)** в очікуванні.

**Мінімальне досконале хешування (MPHF)** — біекція n ключів на `0..n−1`, ≈ 2–3 біти на ключ. Утиліта `gperf` генерує такі функції для компіляторів. .NET `FrozenDictionary` застосовує схожу ідею: при створенні аналізує ключі (наприклад, шукає підрядок, що робить хеші унікальними) і обирає найшвидшу стратегію.

```csharp
// Пошук досконалої хеш-функції для статичного набору ключових слів:
// перебираємо зерно seed, доки h(key) = FNV(seed, key) % m не дасть нуль колізій.
string[] keywords = ["if", "else", "for", "while", "return", "class", "void", "int"];

foreach (int tableSize in new[] { 8, 12, 16 })
{
    (uint seed, int attempts) = FindPerfectSeed(keywords, tableSize);
    Console.WriteLine($"m = {tableSize,2}: зерно {seed,6} знайдено за {attempts,6} спроб");
    if (tableSize == 16)
    {
        var slots = new string?[tableSize];
        foreach (string k in keywords) slots[Hash(k, seed) % tableSize] = k;
        Console.WriteLine("  " + string.Join(" ", slots.Select(s => s ?? "·")));
    }
}

static (uint Seed, int Attempts) FindPerfectSeed(string[] keys, int m)
{
    for (uint seed = 1; ; seed++)
    {
        var used = new bool[m];
        bool ok = true;
        foreach (string key in keys)
        {
            int index = (int)(Hash(key, seed) % (uint)m);
            if (used[index]) { ok = false; break; } // колізія → пробуємо наступне зерно
            used[index] = true;
        }
        if (ok) return (seed, (int)seed);
    }
}

static uint Hash(string s, uint seed)
{
    uint h = 2166136261 ^ seed;
    foreach (char c in s) h = unchecked((h ^ c) * 16777619);
    h ^= h >> 15; h = unchecked(h * 0x2C1B3C6D); h ^= h >> 12;
    return h;
}
```

**Приклад запуску:**
```
m =  8: зерно    395 знайдено за    395 спроб
m = 12: зерно      4 знайдено за      4 спроб
m = 16: зерно     11 знайдено за     11 спроб
  for class return · else void int · while · if · · · · ·
```

При `m = n` (мінімальне) пошук зерна експоненційно довший: ймовірність випадкової біекції `n!/nⁿ` (для n = 8 ≈ 0.24%). Тому MPHF будують розумнішими алгоритмами (CHD, BBHash, RecSplit), а не перебором.

### 11.2. Хешування зозулі (cuckoo hashing)

Дві таблиці `T₁`, `T₂` і дві хеш-функції `h₁`, `h₂`. Кожен ключ може бути **лише** в `T₁[h₁(k)]` або в `T₂[h₂(k)]`.

- **Пошук / видалення — O(1) у гіршому випадку**: максимум 2 перевірки.
- **Вставка**: кладемо ключ у `T₁[h₁(k)]`. Якщо там був інший ключ — **виштовхуємо** його (як зозуля виштовхує яйця з гнізда) в його альтернативне місце в іншій таблиці, і так далі. Якщо цикл виштовхувань занадто довгий — **рехешуємо** з новими функціями.
- Працює при α ≤ 0.5 (сумарно на обидві таблиці); варіанти з кошиками на 4 ключі — до 0.95.

```
Вставка X: h₁(X)=1 зайнято A → X у T₁[1], A виштовхнуто
           A йде в T₂[h₂(A)]=3, там B → A у T₂[3], B виштовхнуто
           B йде в T₁[h₁(B)]=0 — вільно → готово

  T₁: [ B ][ X ][   ][   ]       T₂: [   ][   ][   ][ A ]
```

```csharp
// Хешування зозулі: дві таблиці, пошук ≤ 2 перевірки, вставка з виштовхуванням.
var cuckoo = new CuckooHashSet(capacityPerTable: 11);
int[] keys = [20, 50, 53, 75, 100, 67, 105, 3, 36, 39];

foreach (int key in keys)
{
    int kicks = cuckoo.Add(key);
    Console.WriteLine($"Add({key,3}): виштовхувань {kicks}{(cuckoo.Rehashes > 0 ? $", рехешувань {cuckoo.Rehashes}" : "")}");
}
Console.WriteLine(cuckoo);
Console.WriteLine($"Contains(67) = {cuckoo.Contains(67)}, Contains(68) = {cuckoo.Contains(68)}");

public sealed class CuckooHashSet(int capacityPerTable)
{
    private const int MaxKicks = 16;

    private int?[] _table1 = new int?[capacityPerTable];
    private int?[] _table2 = new int?[capacityPerTable];
    private int _seed = 1;

    public int Rehashes { get; private set; }

    public bool Contains(int key) =>
        _table1[H1(key)] == key || _table2[H2(key)] == key; // рівно дві перевірки

    // Повертає кількість виштовхувань під час вставки.
    public int Add(int key)
    {
        if (Contains(key)) return 0;

        int current = key;
        for (int kicks = 0; kicks < MaxKicks; kicks++)
        {
            int?[] table = kicks % 2 == 0 ? _table1 : _table2;   // по черзі T₁, T₂, T₁, ...
            int index = kicks % 2 == 0 ? H1(current) : H2(current);

            if (table[index] is not int evicted)
            {
                table[index] = current;
                return kicks;
            }
            table[index] = current; // займаємо місце
            current = evicted;      // і несемо виштовхнутий ключ в іншу таблицю
        }

        // Імовірно, цикл виштовхувань: змінюємо хеш-функції і перебудовуємо.
        Rehash();
        return MaxKicks + Add(current);
    }

    private void Rehash()
    {
        Rehashes++;
        int[] all = [.. _table1.Concat(_table2).OfType<int>()];
        _table1 = new int?[_table1.Length];
        _table2 = new int?[_table2.Length];
        _seed++;
        foreach (int key in all) Add(key);
    }

    private int H1(int key) => (int)((uint)(key * _seed) % (uint)_table1.Length);
    private int H2(int key) => (int)((uint)(key / _table2.Length * _seed + 7) % (uint)_table2.Length);

    public override string ToString() =>
        "T₁: " + string.Join(" ", _table1.Select(v => v?.ToString() ?? "·")) + Environment.NewLine +
        "T₂: " + string.Join(" ", _table2.Select(v => v?.ToString() ?? "·"));
}
```

**Приклад запуску:**
```
Add( 20): виштовхувань 0
Add( 50): виштовхувань 0
Add( 53): виштовхувань 1
Add( 75): виштовхувань 1
Add(100): виштовхувань 0
Add( 67): виштовхувань 1
Add(105): виштовхувань 3
Add(  3): виштовхувань 0
Add( 36): виштовхувань 1
Add( 39): виштовхувань 7
T₁: · 100 · 36 · · 50 · · 75 ·
T₂: 53 · 67 · · 105 · 3 20 · 39
Contains(67) = True, Contains(68) = False
```

| Схема | Пошук (гірший) | Вставка | Пам'ять | Динамічна? |
|-------|----------------|---------|---------|------------|
| Ланцюжки | O(n) | O(1) аморт. | вузли | так |
| Лінійне пробування | O(n) | O(1) аморт. | масив | так |
| Robin Hood | O(log n) з високою ймовірністю | O(1) аморт. | масив | так |
| Cuckoo | **O(1)** | O(1) аморт. (очікувано) | 2 масиви, α ≤ 0.5 | так |
| Досконале (FKS) | **O(1)** | — | O(n) | ні |

---

## 12. Порівняння: хеш-таблиця vs збалансоване дерево vs відсортований масив

| Критерій | Хеш-таблиця (`Dictionary`) | Збалансоване дерево (`SortedDictionary`) | Відсортований масив (`SortedList`, `Array.BinarySearch`) |
|----------|------------------|-------------------|------------------|
| Пошук | **O(1)** сер., O(n) гірший | O(log n) гарантовано | O(log n) гарантовано |
| Вставка | **O(1)** аморт. | O(log n) | O(n) (зсув) |
| Видалення | **O(1)** | O(log n) | O(n) |
| Мінімум / максимум | O(n) | O(log n) | **O(1)** |
| Обхід у порядку ключів | O(n log n) (сортування) | **O(n)** | **O(n)** |
| Діапазонний запит `[a, b]` | O(n) | O(log n + k) | O(log n + k) |
| Наступник / попередник | O(n) | O(log n) | O(log n) |
| Вимога до ключа | `GetHashCode` + `Equals` | `IComparable` / `IComparer` | `IComparable` / `IComparer` |
| Пам'ять | помірна (запас під α) | велика (вузли, вказівники) | **мінімальна** |
| Локальність кешу | добра (масиви) | погана | **відмінна** |
| Передбачуваність часу | стрибки при рехешуванні | стабільно | стабільно для читання |
| Стійкість до атак | потрібна рандомізація | стійке | стійке |

**Як обирати:**

- Лише «є / немає / отримати за ключем» → **хеш-таблиця**.
- Потрібен порядок, діапазони, найближчий ключ → **дерево**.
- Дані будуються один раз, потім лише читаються, важлива пам'ять → **відсортований масив** (або `FrozenDictionary` для точного пошуку).
- Жорсткі вимоги до гіршого випадку (реальний час) → **дерево** або cuckoo/досконале хешування.

```csharp
// Невеликий бенчмарк-ілюстрація: кількість порівнянь ключів при пошуку (детерміновано),
// а не час (час залежить від машини).
const int N = 100_000;
int[] keys = Enumerable.Range(0, N).Select(i => i * 7 + 3).ToArray(); // вже відсортовані

var hashComparer = new CountingEquality();
var dict = new Dictionary<int, int>(hashComparer);
foreach (int k in keys) dict[k] = k;

var treeComparer = new CountingComparer();
var tree = new SortedDictionary<int, int>(treeComparer);
foreach (int k in keys) tree[k] = k;

var arrayComparer = new CountingComparer();

hashComparer.Calls = treeComparer.Calls = 0;
var rng = new Random(Seed: 1);
int[] queries = Enumerable.Range(0, 1000).Select(_ => keys[rng.Next(N)]).ToArray();
foreach (int q in queries)
{
    _ = dict.ContainsKey(q);
    _ = tree.ContainsKey(q);
    _ = Array.BinarySearch(keys, q, arrayComparer);
}

Console.WriteLine($"N = {N}, 1000 успішних пошуків, log₂N ≈ {Math.Log2(N):F1}");
Console.WriteLine($"Dictionary:        {hashComparer.Calls / 1000.0,5:F2} Equals на пошук");
Console.WriteLine($"SortedDictionary:  {treeComparer.Calls / 1000.0,5:F2} Compare на пошук");
Console.WriteLine($"Array.BinarySearch:{arrayComparer.Calls / 1000.0,5:F2} Compare на пошук");

public sealed class CountingEquality : IEqualityComparer<int>
{
    public long Calls;
    public bool Equals(int x, int y) { Calls++; return x == y; }
    public int GetHashCode(int value) => value;
}

public sealed class CountingComparer : IComparer<int>
{
    public long Calls;
    public int Compare(int x, int y) { Calls++; return x.CompareTo(y); }
}
```

**Приклад запуску:**
```
N = 100000, 1000 успішних пошуків, log₂N ≈ 16.6
Dictionary:         1.00 Equals на пошук
SortedDictionary:  15.75 Compare на пошук
Array.BinarySearch:15.62 Compare на пошук
```

---

## 13. Типові помилки та підсумки

### 13.1. Зведений список типових помилок

| # | Помилка | Наслідок | Як правильно |
|---|---------|----------|--------------|
| 1 | `Equals` без `GetHashCode` | рівні ключі не знаходяться | перевизначати обидва / `record` |
| 2 | Мутабельний ключ | «загублені» записи, витоки | незмінні ключі |
| 3 | `hash % m` з від'ємним хешем | `IndexOutOfRangeException` | `& 0x7FFFFFFF` або `(uint)` |
| 4 | Слабкий хеш + маска `2^p` | скупчення в кількох кошиках | перемішування (fmix) або просте `m` |
| 5 | Видалення без надгробків (відкрита адресація) | загублені ключі | надгробки / backward shift |
| 6 | α → 1 при відкритій адресації | O(n) пробування | поріг 0.5–0.75 |
| 7 | Збільшення таблиці на константу | O(n) амортизовано | множення на 2 |
| 8 | `ContainsKey` + індексатор | подвійний пошук | `TryGetValue`, `GetValueRefOrAddDefault` |
| 9 | Порядок перелічення `Dictionary`/`HashSet` | крихкий код і тести | сортувати або `SortedDictionary` |
| 10 | Зберігання `string.GetHashCode()` | інші значення в іншому процесі | стабільний хеш (FNV, xxHash, SHA-256) |
| 11 | Хеш-таблиця з вхідними даними від користувача без рандомізації | HashDoS | рандомізовані хеші (.NET робить за замовчуванням для рядків) |
| 12 | Однаковий хеш ⇒ рівність | хибні збіги (Рабін–Карп, дедуплікація) | завжди `Equals` / посимвольна перевірка |
| 13 | `Dictionary` з кількох потоків | зациклення, пошкодження | `ConcurrentDictionary` / блокування |
| 14 | Масив або `List` як ключ | посилальна рівність | `IEqualityComparer` за вмістом або незмінний ключ-рядок/кортеж |

### 13.2. Підсумки

1. **Хешування** узагальнює пряму адресацію на великі універсуми: пам'ять O(n), операції O(1) в середньому.
2. **Хеш-функція** має бути детермінованою, узгодженою з `Equals`, рівномірною і лавинною. Для рядків — поліноміальна, FNV-1a, djb2; для якості при маскуванні — фіналізатор (Murmur fmix).
3. **Колізії неминучі** і з'являються рано (парадокс днів народження: ~1.18·√m ключів).
4. **Ланцюжки** — прості, стійкі до α > 1, але з алокаціями. **Відкрита адресація** — компактна і кеш-дружня, але чутлива до α і вимагає надгробків.
5. **Лінійне пробування** страждає від первинної кластеризації; **подвійне хешування** та **Robin Hood** її пом'якшують.
6. **Рехешування з множенням розміру** дає O(1) амортизовано.
7. **.NET `Dictionary`** — ланцюжки на масивах (`_buckets` + `_entries`), прості розміри, рандомізоване хешування рядків проти HashDoS.
8. **Контракт** `GetHashCode`/`Equals`: рівні об'єкти → рівні хеші; ключі не змінюються. `record` та `HashCode.Combine` роблять це легким.
9. Хеш-таблиця — універсальний прискорювач: частоти, two-sum, анаграми, ковзне вікно, префіксні суми, LRU, Рабін–Карп, Bloom filter, дедуплікація.
10. Для гарантованого O(1) — **cuckoo** або **досконале хешування**; для порядку і діапазонів — **дерево**.

---

## 14. Питання для самоперевірки та практичні завдання

### 14.1. Питання для самоперевірки

1. Чим таблиця прямої адресації відрізняється від хеш-таблиці? Коли пряма адресація все ж краща?
2. Сформулюйте п'ять властивостей хорошої хеш-функції. Яка з них обов'язкова для коректності, а які — лише для швидкості?
3. Чому при методі ділення `m = 2^p` вважається поганим вибором? За яких умов маска `& (m−1)` все ж працює добре?
4. Що таке лавинний ефект і як його виміряти?
5. Чому в хеш-функціях явно пишуть `unchecked`? Що станеться з `<CheckForOverflowUnderflow>true`?
6. Чому сума кодів символів — погана хеш-функція для рядків? Наведіть приклад колізії.
7. Скільки 32-бітних хеш-кодів потрібно, щоб імовірність колізії досягла 50%? Звідки ця оцінка?
8. Опишіть, як виконується `Remove` у таблиці з ланцюжками. Який граничний випадок легко пропустити?
9. Що таке первинна і вторинна кластеризація? Які стратегії пробування від них страждають?
10. Які умови на `h₂(k)` у подвійному хешуванні і чому?
11. Чому при відкритій адресації не можна просто очистити комірку під час видалення? Які недоліки надгробків?
12. Поясніть ідею Robin Hood hashing. Що вона покращує — середню чи максимальну довжину проби?
13. Запишіть очікувану кількість проб для невдалого пошуку при ланцюжках і при лінійному пробуванні. Обчисліть для α = 0.8.
14. Доведіть, що рехешування з подвоєнням дає O(1) амортизовано. Чому додавання константи не підходить?
15. Опишіть внутрішню будову .NET `Dictionary<TKey,TValue>`: навіщо два масиви і поле `next`?
16. Що таке HashDoS? Як .NET захищає `Dictionary<string, T>` від нього?
17. Сформулюйте контракт `GetHashCode`/`Equals`. Що станеться, якщо змінити поле ключа після вставки?
18. Чим `record` відрізняється від `class` щодо рівності? Яку пастку мають поля-масиви в записах?
19. Навіщо потрібен `IEqualityComparer<T>`? Наведіть два практичні приклади.
20. Чому у задачі «підмасиви із сумою k» словник ініціалізується парою `{0: 1}`?
21. Як реалізувати LRU-кеш з O(1) операціями? Навіщо у вузлі списку зберігати ключ?
22. Чому при додаванні сервера `hash % N` переміщує майже всі ключі, а узгоджене хешування — лише ~1/N?
23. Як обчислюється rolling hash у Рабіна–Карпа? Навіщо посимвольна перевірка?
24. Які помилки можливі у фільтра Блума, а які — ні? Як обрати `m` і `k`?
25. Чим cuckoo hashing відрізняється від лінійного пробування щодо гіршого часу пошуку?
26. Коли краще обрати `SortedDictionary` замість `Dictionary`? Назвіть три операції, які дерево робить асимптотично швидше.

### 14.2. Практичні завдання

**Завдання 1 — Хеш-функції (легке).** Реалізуйте djb2, FNV-1a і поліноміальний хеш. Хешуйте 10 000 англійських слів у таблицю з `m = 1024` (маска) і `m = 1021` (модуль) і виведіть для кожної комбінації: кількість порожніх кошиків, максимальну довжину ланцюжка, χ²-статистику рівномірності.

**Завдання 2 — Ланцюжки з видаленням (середнє).** Доповніть `ChainedHashMap` методами `Clear`, `Keys`, `Values`, `TrimExcess()` (зменшення з гістерезисом) і конструктором з `IEnumerable<KeyValuePair<TKey,TValue>>`. Напишіть тести на граничні випадки: видалення голови/середини/хвоста ланцюжка, модифікація під час перелічення.

**Завдання 3 — Відкрита адресація з backward shift (середнє).** Реалізуйте лінійне пробування **без надгробків**, де `Remove` зсуває наступні елементи кластера назад. Порівняйте середню кількість проб з версією з надгробками після 10⁵ випадкових вставок/видалень.

**Завдання 4 — Robin Hood HashMap (складне).** Перетворіть `OpenHashMap` з розділу 7 на Robin Hood: зберігайте DIB (або обчислюйте з хешу), реалізуйте ранню зупинку пошуку і backward shift при видаленні. Виміряйте максимальну довжину проби при α = 0.9.

**Завдання 5 — Контракт рівності (легке).** Створіть клас `Isbn` (рядок з дефісами або без: `"978-3-16-148410-0"` == `"9783161484100"`). Реалізуйте `IEquatable<Isbn>`, `GetHashCode`, оператори `==`/`!=`. Перевірте роботу в `HashSet<Isbn>`.

**Завдання 6 — Ізоморфні рядки (легке).** Визначте, чи можна отримати рядок `t` з `s` взаємно однозначною заміною символів (`"egg"`, `"add"` → так; `"foo"`, `"bar"` → ні). Два словники, O(n).

**Завдання 7 — Мінімальне вікно, що містить усі символи (складне).** Дано рядки `s` і `t`. Знайдіть найкоротший підрядок `s`, який містить усі символи `t` з урахуванням кратності. Ковзне вікно + словник частот, O(|s| + |t|).

**Завдання 8 — LFU-кеш (складне).** Реалізуйте кеш, що викидає **найрідше** використаний елемент (при рівності — найдавніший), з O(1) `Get`/`Put`. Підказка: `Dictionary<key, node>` + `Dictionary<frequency, LinkedList>` + змінна `minFrequency`.

**Завдання 9 — Найдовший повторюваний підрядок (складне).** Бінарним пошуком за довжиною `L` і rolling hash (`HashSet<long>` хешів підрядків довжини `L`) знайдіть найдовший підрядок, що зустрічається принаймні двічі. O(n log n) в середньому. Не забудьте перевірку при збігу хешів.

**Завдання 10 — Counting Bloom filter (середнє).** Замініть біти 4-бітними лічильниками, додайте `Remove`. Експериментально перевірте частку хибнопозитивних після 50% видалень.

**Завдання 11 — Узгоджене хешування (середнє).** Доповніть `ConsistentHashRing` методами `AddServer`/`RemoveServer` без повної перебудови. Побудуйте графік стандартного відхилення навантаження серверів залежно від кількості віртуальних вузлів (1, 10, 50, 100, 500).

**Завдання 12 — Дедуплікатор файлів (середнє).** Консольна утиліта: рекурсивно обходить каталог, групує файли за розміром, потім за SHA-256 (через `SHA256.HashDataAsync(stream)`), виводить групи дублікатів і сумарний обсяг, який можна звільнити.

**Завдання 13 — HashDoS-експеримент (середнє).** Згенеруйте 50 000 різних рядків з однаковим значенням **вашої** хеш-функції `Poly31` (підказка: `"Aa"` і `"BB"` мають однаковий хеш при B = 31, а конкатенації таких блоків — теж). Виміряйте час вставки в таблицю з ланцюжками, що використовує `Poly31`, і в стандартний `Dictionary<string,int>`.

**Завдання 14 — Cuckoo hashing (складне).** Реалізуйте узагальнений `CuckooHashMap<TKey,TValue>` з двома незалежними хешами (через різні зерна), рехешуванням при циклі і збільшенням розміру при α > 0.45. Порівняйте кількість виштовхувань на вставку при α = 0.3, 0.4, 0.49.

<details>
<summary>Розв'язок завдання 6 — ізоморфні рядки</summary>

```csharp
// Ізоморфність: відображення s→t і t→s мають бути узгодженими (біекція).
Console.WriteLine(IsIsomorphic("egg", "add"));
Console.WriteLine(IsIsomorphic("foo", "bar"));
Console.WriteLine(IsIsomorphic("badc", "baba"));
Console.WriteLine(IsIsomorphic("paper", "title"));

static bool IsIsomorphic(string s, string t)
{
    if (s.Length != t.Length) return false;
    var forward = new Dictionary<char, char>();
    var backward = new Dictionary<char, char>();
    for (int i = 0; i < s.Length; i++)
    {
        if (forward.TryGetValue(s[i], out char mapped) && mapped != t[i]) return false;
        if (backward.TryGetValue(t[i], out char source) && source != s[i]) return false;
        forward[s[i]] = t[i];
        backward[t[i]] = s[i];
    }
    return true;
}
```

**Приклад запуску:**
```
True
False
False
True
```
</details>

<details>
<summary>Розв'язок завдання 13 — генерація колізій для Poly31</summary>

```csharp
// "Aa" і "BB" мають однаковий Poly31: 'A'*31 + 'a' = 65*31 + 97 = 2112 = 66*31 + 66.
// Будь-яка конкатенація k таких блоків дає 2^k різних рядків з однаковим хешем.
Console.WriteLine($"Poly31(\"Aa\") = {Poly31("Aa")}, Poly31(\"BB\") = {Poly31("BB")}");

List<string> collisions = [""];
for (int block = 0; block < 4; block++)
    collisions = [.. collisions.SelectMany(prefix => new[] { prefix + "Aa", prefix + "BB" })];

Console.WriteLine($"Згенеровано {collisions.Count} рядків, різних хешів: {collisions.Select(Poly31).Distinct().Count()}");
Console.WriteLine(string.Join(" ", collisions.Take(6)) + " …");

static uint Poly31(string s)
{
    uint h = 0;
    foreach (char c in s) h = unchecked(h * 31 + c);
    return h;
}
```

**Приклад запуску:**
```
Poly31("Aa") = 2112, Poly31("BB") = 2112
Згенеровано 16 рядків, різних хешів: 1
AaAaAaAa AaAaAaBB AaAaBBAa AaAaBBBB AaBBAaAa AaBBAaBB …
```

Для 50 000 рядків достатньо 16 блоків (2¹⁶ = 65 536 рядків довжини 32).
</details>
