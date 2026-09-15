# Лекція 4 — Рекурсія, узагальнення (generics) та лямбда-вирази

---

## Зміст

1. [Рекурсія: базовий і рекурсивний випадок](#1-рекурсія-базовий-і-рекурсивний-випадок)
2. [Стек викликів](#2-стек-викликів)
3. [Класичні приклади рекурсії](#3-класичні-приклади-рекурсії)
4. [Рекурсія vs ітерація](#4-рекурсія-vs-ітерація)
5. [Узагальнені методи і класи](#5-узагальнені-методи-і-класи)
6. [Лямбда-вирази та замикання](#6-лямбда-вирази-та-замикання)
7. [Підсумки](#7-підсумки)
8. [Питання для самоперевірки](#8-питання-для-самоперевірки)

---

## 1. Рекурсія: базовий і рекурсивний випадок

**Рекурсія** — це прийом, коли метод викликає сам себе для розв'язання меншої версії тієї ж задачі.

- **Базовий випадок** (base case) — умова, за якої метод повертає результат **без** рекурсивного виклику. Без нього рекурсія ніколи не завершиться.
- **Рекурсивний випадок** (recursive case) — метод викликає себе з **меншими** аргументами, наближаючись до базового випадку.

```
Factorial(n) = 1                     якщо n <= 1   (базовий випадок)
Factorial(n) = n * Factorial(n - 1)  інакше        (рекурсивний випадок)
```

---

## 2. Стек викликів

Кожен виклик методу створює на **стеку викликів** (call stack) новий кадр: аргументи, локальні змінні, адресу повернення. Кадр знищується, коли метод повертає значення.

Розмір стеку потоку обмежений (у .NET зазвичай 1 МБ для головного потоку). Занадто глибока рекурсія (або відсутній базовий випадок) призводить до **`StackOverflowException`**. Цей виняток **неможливо перехопити** через `try/catch` — процес .NET просто аварійно завершується.

### Приклад 1. Факторіал із трасуванням стеку

```csharp
long f = Factorial(4);  // спочатку рахуємо (з трасуванням), потім друкуємо
Console.WriteLine($"4! = {f}");

// depth — глибина рекурсії, використовуємо лише для відступів у виводі
static long Factorial(int n, int depth = 0)
{
    string indent = new(' ', depth * 2);  // відступ: 2 пробіли на рівень
    Console.WriteLine($"{indent}-> Factorial({n})");

    // Базовий випадок: 0! = 1! = 1, далі не заглиблюємось
    if (n <= 1)
    {
        Console.WriteLine($"{indent}<- 1");
        return 1;
    }

    // Рекурсивний випадок: задача зводиться до меншої (n - 1)
    long result = n * Factorial(n - 1, depth + 1);
    Console.WriteLine($"{indent}<- {result}");
    return result;  // кадр стеку для цього n знищується тут
}
```

**Приклад запуску:**

```
-> Factorial(4)
  -> Factorial(3)
    -> Factorial(2)
      -> Factorial(1)
      <- 1
    <- 2
  <- 6
<- 24
4! = 24
```

---

## 3. Класичні приклади рекурсії

### Приклад 2. Числа Фібоначчі: наївна рекурсія vs мемоізація

```csharp
long calls = 0;  // лічильник викликів, щоб побачити вартість наївного підходу
int n = 30;

Console.WriteLine($"FibNaive({n}) = {FibNaive(n)}, викликів: {calls}");

long[] memo = new long[n + 1];
Array.Fill(memo, -1);  // -1 означає "ще не пораховано"
Console.WriteLine($"FibMemo({n})  = {FibMemo(n, memo)}");

// Наївна рекурсія: O(2^n) — одні й ті самі значення рахуються багато разів.
// Локальна функція (не static) бачить змінну calls із зовнішньої області.
long FibNaive(int k)
{
    calls++;
    if (k <= 1) return k;                    // базові випадки: F(0)=0, F(1)=1
    return FibNaive(k - 1) + FibNaive(k - 2); // два рекурсивні виклики
}

// Мемоізація: запам'ятовуємо вже пораховані значення — O(n)
static long FibMemo(int k, long[] cache)
{
    if (k <= 1) return k;
    if (cache[k] != -1) return cache[k];     // уже рахували — повертаємо з кешу
    cache[k] = FibMemo(k - 1, cache) + FibMemo(k - 2, cache);
    return cache[k];
}
```

**Приклад запуску:**

```
FibNaive(30) = 832040, викликів: 2692537
FibMemo(30)  = 832040
```

---

## 4. Рекурсія vs ітерація

| Критерій | Рекурсія | Ітерація |
|----------|----------|----------|
| Читабельність | Природна для дерев, графів, "розділяй і володарюй" | Природна для простих лінійних проходів |
| Пам'ять | O(глибина) на стеку викликів | Зазвичай O(1) додаткової пам'яті |
| Ризики | `StackOverflowException`, повторні обчислення | Складніше виразити обхід дерев |
| Швидкість | Накладні витрати на виклики | Зазвичай швидша |

Будь-яку рекурсію можна переписати ітеративно (у крайньому разі — з явним `Stack<T>`). C# **не гарантує** оптимізацію хвостової рекурсії, тож на глибину покладатися не варто.

### Приклад 3. Факторіал і Фібоначчі ітеративно

```csharp
Console.WriteLine($"20! = {FactorialIter(20)}");
Console.WriteLine($"F(90) = {FibIter(90)}");

// Ітеративний факторіал: один цикл, жодних додаткових кадрів стеку
static long FactorialIter(int n)
{
    long result = 1;
    for (int i = 2; i <= n; i++)
        result *= i;
    return result;
}

// Ітеративний Фібоначчі: зберігаємо лише два попередні значення — O(n) час, O(1) пам'ять
static long FibIter(int n)
{
    long prev = 0, curr = 1;  // F(0), F(1)
    if (n == 0) return prev;
    for (int i = 2; i <= n; i++)
        (prev, curr) = (curr, prev + curr);  // кортежне присвоєння замість тимчасової змінної
    return curr;
}
```

**Приклад запуску:**

```
20! = 2432902008176640000
F(90) = 2880067194370816120
```

---

## 5. Узагальнені методи і класи

**Узагальнення** (generics) дозволяють писати код один раз для багатьох типів. На відміну від простого `object`, вони зберігають **типобезпеку** і не потребують упаковки (boxing) для типів-значень: для кожного типу-значення (`int`, `long`) JIT створює окрему спеціалізовану версію.

**Обмеження** (`where`) кажуть компілятору, що вміє тип `T`:

| Обмеження | Значення |
|-----------|----------|
| `where T : IComparable<T>` | `T` можна порівнювати через `CompareTo` |
| `where T : INumber<T>` | `T` — число: доступні `+`, `T.Zero`, `T.One` (generic math, .NET 7+) |
| `where T : class` / `struct` | посилальний тип / тип-значення |
| `where T : new()` | є конструктор без параметрів |

### Приклад 4. Узагальнені методи з обмеженнями

```csharp
using System.Numerics;

Console.WriteLine(MaxOf(3, 7));                  // T = int (виведено автоматично)
Console.WriteLine(MaxOf("apple", "banana"));     // T = string (порівняння рядків)
Console.WriteLine(MaxOf<long>(10, 2));           // T задано явно

int[] ints = [1, 2, 3, 4];
long[] longs = [5_000_000_000, 7];
Console.WriteLine($"{SumRecursive(ints)} {SumRecursive(longs)}");

// Без обмеження компілятор не дозволить викликати CompareTo для довільного T
static T MaxOf<T>(T a, T b) where T : IComparable<T>
    => a.CompareTo(b) > 0 ? a : b;

// INumber<T> дає оператор + і T.Zero — рекурсивна сума для будь-якого числового типу
static T SumRecursive<T>(T[] items, int i = 0) where T : INumber<T>
{
    if (i == items.Length) return T.Zero;         // базовий випадок: "нуль" для типу
    return items[i] + SumRecursive(items, i + 1); // рекурсивний випадок
}
```

**Приклад запуску:**

```
7
banana
10
10 5000000007
```

### Приклад 5. Узагальнений клас — стек

```csharp
var numbers = new MyStack<int>();
for (int i = 1; i <= 3; i++) numbers.Push(i * 10);

var words = new MyStack<string>();
words.Push("hello");
words.Push("world");

Console.WriteLine($"numbers.Count = {numbers.Count}");
var popped = new List<int>();
while (!numbers.IsEmpty) popped.Add(numbers.Pop());  // LIFO: 30 20 10
Console.WriteLine(string.Join(" ", popped));
Console.WriteLine($"{words.Pop()} {words.Pop()}");

// MyStack<int> і MyStack<string> — різні закриті типи, створені з одного визначення
class MyStack<T>
{
    private readonly List<T> _data = [];  // сховище елементів; T відоме при використанні

    public int Count => _data.Count;
    public bool IsEmpty => _data.Count == 0;

    public void Push(T value) => _data.Add(value);

    public T Pop()
    {
        if (IsEmpty) throw new InvalidOperationException("Stack is empty");
        T top = _data[^1];            // ^1 — останній елемент
        _data.RemoveAt(_data.Count - 1);
        return top;
    }
}
```

**Приклад запуску:**

```
numbers.Count = 3
30 20 10
world hello
```

---

## 6. Лямбда-вирази та замикання

**Лямбда-вираз** — анонімна функція, записана прямо в місці використання:

```
(параметри) => вираз
(параметри) => { інструкції; }
```

Лямбду зберігають у делегаті: `Func<T1, ..., TResult>` (повертає значення), `Action<T1, ...>` (нічого не повертає), `Predicate<T>` (повертає `bool`).

**Замикання (closure).** Якщо лямбда використовує зовнішню змінну, C# захоплює **саму змінну, а не її копію**: компілятор переносить її в прихований об'єкт, спільний для методу й лямбди. Тому лямбда бачить усі пізніші зміни і сама може змінювати змінну. Щоб "заморозити" значення, скопіюйте його в окрему локальну змінну. Модифікатор `static` у лямбди забороняє будь-яке захоплення.

### Приклад 6. Func, Action і семантика захоплення

```csharp
int[] v = [5, 3, 8, 1, 9, 2];

// Comparison<int> як лямбда: сортування за спаданням
Array.Sort(v, (a, b) => b.CompareTo(a));
Console.WriteLine(string.Join(" ", v));

// Лямбда захоплює threshold і використовується як предикат LINQ
int threshold = 4;
int count = v.Count(x => x > threshold);
Console.WriteLine($"Більших за {threshold}: {count}");

// Action змінює зовнішню змінну sum — захоплена сама змінна
int sum = 0;
Action<int> add = x => sum += x;
foreach (int x in v) add(x);
Console.WriteLine($"Сума: {sum}");

// Захоплення змінної vs копія значення
int value = 1;
int snapshot = value;                  // окрема змінна-копія
Func<int> live = () => value;          // бачить живу змінну
Func<int> frozen = () => snapshot;     // бачить копію
value = 100;
Console.WriteLine($"live: {live()}, frozen: {frozen()}");

// Фабрика лічильників: кожен виклик MakeCounter створює нову змінну n
Func<int> counter = MakeCounter();
counter();
counter();
Console.WriteLine($"counter: {counter()}");

// Рекурсивна лямбда: спершу оголошуємо змінну, потім присвоюємо лямбду,
// яка захоплює цю ж змінну і викликає себе через неї
Func<int, long> fact = null!;
fact = n => n <= 1 ? 1 : n * fact(n - 1);
Console.WriteLine($"fact(10) = {fact(10)}");

static Func<int> MakeCounter()
{
    int n = 0;          // живе стільки, скільки живе лямбда, що її захопила
    return () => ++n;
}
```

**Приклад запуску:**

```
9 8 5 3 2 1
Більших за 4: 3
Сума: 28
live: 100, frozen: 1
counter: 3
fact(10) = 3628800
```

### Приклад 7. Узагальнення + лямбда: власний фільтр

```csharp
int[] nums = [1, 2, 3, 4, 5, 6, 7, 8];
List<int> evens = Filter(nums, x => x % 2 == 0);  // T = int виводиться з nums

string[] names = ["Ann", "Bohdan", "Iryna", "Oleh"];
int minLen = 4;
// Захоплюємо minLen, щоб умову можна було налаштувати ззовні
List<string> longNames = Filter(names, s => s.Length > minLen);

Console.WriteLine(string.Join(" ", evens));
Console.WriteLine(string.Join(" ", longNames));

// predicate — будь-яка функція T -> bool: лямбда, метод, група методів
static List<T> Filter<T>(IEnumerable<T> items, Func<T, bool> predicate)
{
    var result = new List<T>();
    foreach (T item in items)
        if (predicate(item)) result.Add(item);  // викликаємо переданий делегат
    return result;
}
```

**Приклад запуску:**

```
2 4 6 8
Bohdan Iryna
```

---

## 7. Підсумки

- Рекурсія = **базовий випадок** + **рекурсивний випадок**, що наближає до базового.
- Кожен виклик займає кадр на **стеку викликів**; надто глибока рекурсія дає `StackOverflowException`, який не перехоплюється.
- Наївна рекурсія може бути експоненційною (Фібоначчі) — допомагає **мемоізація** або ітерація.
- **Generics** дають типобезпечний узагальнений код; **обмеження** `where` (`IComparable<T>`, `INumber<T>`) визначають, що можна робити з `T`.
- **Лямбди** зберігаються у `Func`/`Action`; замикання захоплює **змінну**, а не значення.

---

## 8. Питання для самоперевірки

1. Що станеться, якщо в рекурсивному методі немає базового випадку? Чи можна перехопити цю помилку?
2. Чому наївний `FibNaive(30)` робить мільйони викликів, а `FibMemo(30)` — лише кілька десятків?
3. Яка пам'ять потрібна рекурсивному та ітеративному факторіалу для `n`?
4. Навіщо в `MaxOf<T>` обмеження `where T : IComparable<T>` і що буде без нього?
5. Чому `live()` у прикладі 6 повертає 100? Як отримати значення на момент створення лямбди?
