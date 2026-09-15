# Лекція 1 — Нагадування: можливості C#

---

## Зміст
1. [Вступ: курс і мова C#](#1-вступ-курс-і-мова-c)
2. [Мінімальна програма](#2-мінімальна-програма)
3. [Типи даних, const, readonly, var](#3-типи-даних-const-readonly-var)
4. [Умови, switch і pattern matching](#4-умови-switch-і-pattern-matching)
5. [Цикли та масиви](#5-цикли-та-масиви)
6. [Методи: ref, out, in, необов'язкові параметри](#6-методи-ref-out-in-необовязкові-параметри)
7. [Класи, record, struct, властивості, nullable](#7-класи-record-struct-властивості-nullable)
8. [Винятки та LINQ](#8-винятки-та-linq)
9. [Алгоритми: O(n) проти O(log n)](#9-алгоритми-on-проти-olog-n)
10. [Форматування та стиль коду](#10-форматування-та-стиль-коду)
11. [15 сучасних можливостей C#](#11-15-сучасних-можливостей-c)
12. [Підсумки](#12-підсумки)
13. [Питання для самоперевірки](#13-питання-для-самоперевірки)

---

## 1. Вступ: курс і мова C#
**Програма курсу:** 11 лекцій, 11 практичних, Екзамен.

Де застосовується C# / .NET:

- **Веб і хмара:** ASP.NET Core, мікросервіси, Azure; **ігри:** Unity, Godot.
- **Десктоп/мобільні:** WPF, .NET MAUI; **фінанси та enterprise:** банки, трейдинг, ERP.
- **Алгоритми та змагання:** LeetCode, Codeforces.

Версії: C# 7 → 8 (nullable) → 9 (record, top-level) → 10 → 11 → 12. **Фокус курсу: .NET 8+ / C# 12.**

Відмінності від C++, які варто пам'ятати: пам'ять звільняє **збирач сміття** (немає `delete`), замість `template` — **generics**, замість множинного наслідування класів — **інтерфейси**.

---

## 2. Мінімальна програма
Запуск: `dotnet new console`, код у `Program.cs`, `dotnet run`. Сучасний варіант — **top-level statements** (компілятор сам створює `Main`):

```csharp
Console.WriteLine("Hello, world!"); // вивід у консоль; using System підключено неявно
```

Класичний варіант з явним методом `Main` (у проєкті має бути лише одна точка входу):

```csharp
internal static class Program
{
    private static int Main(string[] args) // args — аргументи командного рядка
    {
        Console.WriteLine("Hello, world!");
        return 0;                           // код успішного завершення (можна void)
    }
}
```

**Приклад запуску:**
```
Hello, world!
```

---

## 3. Типи даних, const, readonly, var
| Тип | Розмір | Примітка |
|-----|--------|----------|
| `byte` / `short` / `int` / `long` | 1 / 2 / 4 / 8 байт | розміри **фіксовані** (на відміну від C++) |
| `uint` / `ulong` | 4 / 8 байт | тільки ≥ 0 |
| `float` / `double` | 4 / 8 байт | двійкова плаваюча кома |
| `decimal` | 16 байт | десяткова, для грошей |
| `char` | 2 байти | UTF-16 |
| `bool` | 1 байт | лише `true`/`false`, не число |
| `string` | посилання | незмінний (immutable) рядок |

- `const` — константа **часу компіляції** (аналог `constexpr`).
- `readonly` — поле, яке задається один раз у конструкторі (**час виконання**).
- `var` — тип виводиться компілятором, але залишається **статичним**.

```csharp
int i = -1000;                  // 4 байти
long big = 9_000_000_000L;      // 8 байт; _ — роздільник розрядів
uint u = 4_000_000_000U;        // беззнаковий
double d = 2.718281828;         // 8 байт
decimal price = 19.99m;         // суфікс m обов'язковий для decimal
char c = 'A';
bool ok = true;
string name = "Andrii";
Console.WriteLine($"int={sizeof(int)} long={sizeof(long)} double={sizeof(double)} decimal={sizeof(decimal)} char={sizeof(char)} bool={sizeof(bool)}");
Console.WriteLine($"int.MaxValue = {int.MaxValue}"); // межі типу — статичні поля
Console.WriteLine($"{i} {big} {u} {d} {price} {c} {ok} {name}");
const int N = 16;               // const: значення відоме компілятору
int[] arr = new int[N];         // розмір масиву може бути й змінною — масиви живуть у heap
var total = N * 2;              // var → int (виводиться з виразу)
Console.WriteLine($"arr.Length={arr.Length}, total={total}, тип={total.GetType().Name}");
Console.WriteLine($"PI ≈ {MathConsts.Pi:F4}"); // :F4 — 4 знаки після коми
static class MathConsts
{
    public static readonly double Pi = Math.Round(Math.PI, 6); // readonly: обчислюється під час виконання
}
```

**Приклад запуску:**
```
int=4 long=8 double=8 decimal=16 char=2 bool=1
int.MaxValue = 2147483647
-1000 9000000000 4000000000 2.718281828 19.99 A True Andrii
arr.Length=16, total=32, тип=Int32
PI ≈ 3.1416
```

---

## 4. Умови, switch і pattern matching
Калькулятор зі слайдів: у C# немає «провалювання» між `case` — кожна гілка закінчується `break`/`return`. Компактніше — **switch-вираз** з патернами.

```csharp
// Рядок замість Console.ReadLine(), щоб приклад був відтворюваним
string input = "3 + 5";                      // приклад вводу
string[] parts = input.Split(' ');           // ["3", "+", "5"]
double a = double.Parse(parts[0]);           // Parse кидає FormatException на поганих даних
char op = parts[1][0];
double b = double.Parse(parts[2]);
// Класичний switch-оператор
switch (op)
{
    case '+': Console.WriteLine(a + b); break;
    case '/' when b == 0: Console.WriteLine("Помилка: ділення на 0"); break; // when — додаткова умова
    case '/': Console.WriteLine(a / b); break;
    default: Console.WriteLine("Невідомий оператор"); break;
}
// switch-вираз: кожна гілка повертає значення
string Sign(int x) => x switch
{
    > 0 => "Positive",                       // реляційний патерн
    < 0 => "Negative",
    _ => "Zero"                              // _ — будь-яке інше значення
};
Console.WriteLine($"{Sign(7)} {Sign(-2)} {Sign(0)}");
// TryParse не кидає виняток: повертає bool і результат через out
if (int.TryParse("abc", out int n)) Console.WriteLine(n);
else Console.WriteLine("Не число");
// Патерн типу + логічні патерни and/or/not
object value = 42;
if (value is int k and >= 10 and <= 99) Console.WriteLine($"Двозначне ціле: {k}");
```

**Приклад запуску:**
```
8
Positive Negative Zero
Не число
Двозначне ціле: 42
```

---

## 5. Цикли та масиви
Масиви в C# — об'єкти з властивістю `Length` і перевіркою меж (`IndexOutOfRangeException`). Є три види: одновимірні `int[]`, прямокутні `int[,]` та «зубчасті» (jagged) `int[][]` — масив масивів різної довжини.

```csharp
for (int i = 0; i < 5; ++i) Console.Write($"{i} ");   // 0 1 2 3 4
Console.WriteLine();
int j = 3;
while (j > 0) Console.Write($"{j--} ");               // 3 2 1
Console.WriteLine();
int k = 0;
do { Console.Write($"{k++} "); } while (k < 3);       // тіло виконується хоча б раз
Console.WriteLine();
int[] arr = [1, 2, 3, 4, 5];                          // collection expression (C# 12)
foreach (int v in arr) Console.Write($"{v} ");        // foreach — лише читання елементів
Console.WriteLine();
int[,] grid = new int[2, 3];                          // прямокутний 2×3
for (int r = 0; r < grid.GetLength(0); r++)           // GetLength(0) — кількість рядків
    for (int col = 0; col < grid.GetLength(1); col++)
        grid[r, col] = r * 10 + col;
Console.WriteLine($"grid[1,2] = {grid[1, 2]}, всього {grid.Length} клітинок");
int[][] jagged = [[1], [2, 3], [4, 5, 6]];            // рядки різної довжини
foreach (int[] row in jagged) Console.WriteLine(string.Join(",", row));
```

**Приклад запуску:**
```
0 1 2 3 4 
3 2 1 
0 1 2 
1 2 3 4 5 
grid[1,2] = 12, всього 6 клітинок
1
2,3
4,5,6
```

---

## 6. Методи: ref, out, in, необов'язкові параметри
| C++ | C# | Сенс |
|-----|----|------|
| `void f(int x)` | `void F(int x)` | копія, оригінал не змінюється |
| `void f(int& x)` | `void F(ref int x)` | змінює оригінал |
| — | `void F(out int x)` | метод **зобов'язаний** присвоїти значення |
| `void f(const T& x)` | `void F(in T x)` | передача за посиланням лише для читання |

Вказівників і `new/delete` у звичайному коді немає: класи — **посилальні типи** (змінна зберігає посилання), пам'ять звільняє GC.

```csharp
int x = 7;
ByValue(x); Console.WriteLine($"ByValue: {x}");      // 7 — змінювалась копія
ByRef(ref x); Console.WriteLine($"ByRef: {x}");      // 17 — ref також на місці виклику
Split(17, 5, out int q, out int rem);                // out-змінні оголошуються прямо у виклику
Console.WriteLine($"17 = 5*{q} + {rem}");
Console.WriteLine(Area(in x));                       // in: не копіює, але й не дозволяє змінити
Console.WriteLine(Greet("Andrii"));                  // greeting бере значення за замовчуванням
Console.WriteLine(Greet(greeting: "Привіт", name: "Olena")); // іменовані параметри — будь-який порядок
Console.WriteLine($"{Add(2, 3)} {Add(2.5, 1.5)}");   // generic-метод, аналог template
static void ByValue(int v) => v += 10;
static void ByRef(ref int v) => v += 10;
static void Split(int a, int b, out int quotient, out int remainder) =>
    (quotient, remainder) = (a / b, a % b);          // обидва out-параметри мають бути присвоєні
static int Area(in int side) => side * side;
static string Greet(string name, string greeting = "Hello") => $"{greeting}, {name}!";
static T Add<T>(T a, T b) where T : System.Numerics.INumber<T> => a + b; // обмеження: T — число
```

**Приклад запуску:**
```
ByValue: 7
ByRef: 17
17 = 5*3 + 2
289
Hello, Andrii!
Привіт, Olena!
5 4
```

---

## 7. Класи, record, struct, властивості, nullable
- **Інкапсуляція:** приватні поля + **властивості** (`get`/`set`/`init`).
- **Наслідування:** лише від **одного** класу, але багатьох **інтерфейсів** (замість множинного наслідування).
- **Поліморфізм:** `virtual` + `override`; **абстракція:** `abstract` / `interface`.
- `static` — член належить типу, а не об'єкту. Перевантаження операторів — `public static T operator +`.
- `class` — посилальний тип; `struct` — значущий (копіюється); `record` — порівнюється **за значенням**.
- **Nullable reference types:** `string?` може бути `null`, `string` — ні (компілятор попереджає).

```csharp
var acc = new BankAccount(100m);
acc.Deposit(50m);
Console.WriteLine($"Баланс: {acc.Balance}");         // acc.Balance = 0; — помилка: private set
IShape[] shapes = [new Circle(1), new Square(2)];    // поліморфізм через інтерфейс
foreach (IShape s in shapes) Console.WriteLine($"{s.Name}: {s.Area():F2}");
var p1 = new Point(1, 2);                            // record struct: значущий тип + рівність за значенням
var p2 = new Point(1, 2);
Console.WriteLine($"{p1 + p2}, рівні: {p1 == p2}");  // ToString генерується автоматично
Console.WriteLine($"Створено рахунків: {BankAccount.Count}");
string? nickname = null;                             // ? — явно дозволяємо null
Console.WriteLine($"Довжина: {nickname?.Length ?? 0}"); // ?. — безпечний доступ, ?? — значення за замовчуванням
public class BankAccount
{
    public static int Count { get; private set; }    // static: спільний для всіх об'єктів
    public decimal Balance { get; private set; }     // читати можна всім, змінювати — лише класу
    public BankAccount(decimal initial) { Balance = initial; Count++; }
    public void Deposit(decimal amount)
    {
        if (amount <= 0) throw new ArgumentOutOfRangeException(nameof(amount)); // захист інваріанта
        Balance += amount;
    }
}
public interface IShape { string Name { get; } double Area(); } // абстракція: лише контракт
public abstract class Shape : IShape                 // абстрактний клас не можна створити через new
{
    public virtual string Name => "Shape";           // virtual — можна перевизначити
    public abstract double Area();                   // abstract — зобов'язані перевизначити
}
public class Circle(double radius) : Shape           // primary constructor (C# 12)
{
    public override string Name => "Circle";
    public override double Area() => Math.PI * radius * radius;
}
public sealed class Square(double side) : Shape      // sealed — забороняє подальше наслідування
{
    public override string Name => "Square";
    public override double Area() => side * side;
}
public readonly record struct Point(int X, int Y)
{
    public static Point operator +(Point a, Point b) => new(a.X + b.X, a.Y + b.Y); // перевантаження +
}
```

**Приклад запуску:**
```
Баланс: 150
Circle: 3.14
Square: 4.00
Point { X = 2, Y = 4 }, рівні: True
Створено рахунків: 1
Довжина: 0
```

---

## 8. Винятки та LINQ
`try` / `catch` / `finally`; `finally` виконується завжди. **LINQ** — декларативні запити над будь-якою колекцією (`IEnumerable<T>`).

```csharp
try
{
    Console.WriteLine(Divide(10, 2));
    Console.WriteLine(Divide(10, 0));                // кидає виняток — наступний рядок не виконається
}
catch (DivideByZeroException e)                      // найконкретніший тип — першим
{
    Console.WriteLine($"Помилка: {e.Message}");
}
finally
{
    Console.WriteLine("finally виконується завжди");
}
int[] nums = [5, 12, 7, 150, 8, 3];
var evens = nums.Where(n => n % 2 == 0);             // фільтр (лямбда n => ...)
var squares = nums.Select(n => n * n).Take(3);       // проєкція + перші 3
Console.WriteLine($"Парні: {string.Join(" ", evens)}");
Console.WriteLine($"Квадрати: {string.Join(" ", squares)}");
Console.WriteLine($"Sum={nums.Sum()} Min={nums.Min()} Any>100={nums.Any(n => n > 100)}");
Console.WriteLine($"Відсортовані: {string.Join(" ", nums.Order())}");
static double Divide(double a, double b) =>
    b == 0 ? throw new DivideByZeroException("Division by zero!") : a / b; // throw як вираз
```

**Приклад запуску:**
```
5
Помилка: Division by zero!
finally виконується завжди
Парні: 12 150 8
Квадрати: 25 144 49
Sum=185 Min=3 Any>100=True
Відсортовані: 3 5 7 8 12 150
```

---

## 9. Алгоритми: O(n) проти O(log n)
**Алгоритм** — скінченна послідовність однозначних кроків, що з вхідних даних дає результат. **Big-O** описує, як росте час: O(1), O(log n), O(n), O(n²). Для n = 1 000 000 лінійний пошук робить до мільйона перевірок, бінарний (на **відсортованому** масиві) — близько 20.

```csharp
int[] sorted = [1, 3, 5, 7, 9, 11, 13, 15];
Console.WriteLine($"LinearSearch(9) = {LinearSearch(sorted, 9)}");
Console.WriteLine($"BinarySearch(9) = {BinarySearch(sorted, 9)}");
Console.WriteLine($"Array.BinarySearch(9) = {Array.BinarySearch(sorted, 9)}"); // вбудована версія
static int LinearSearch(int[] arr, int target)       // O(n): у гіршому випадку переглядає все
{
    for (int i = 0; i < arr.Length; i++)
        if (arr[i] == target) return i;
    return -1;
}
static int BinarySearch(int[] arr, int target)       // O(log n): щоразу відкидає половину
{
    int left = 0, right = arr.Length - 1;
    while (left <= right)
    {
        int mid = left + (right - left) / 2;         // без переповнення, на відміну від (left+right)/2
        Console.WriteLine($"  [{left}..{right}] mid={arr[mid]}");
        if (arr[mid] == target) return mid;
        if (arr[mid] < target) left = mid + 1;       // шукаємо праворуч
        else right = mid - 1;                        // шукаємо ліворуч
    }
    return -1;
}
```

**Приклад запуску:**
```
LinearSearch(9) = 4
  [0..7] mid=7
  [4..7] mid=11
  [4..4] mid=9
BinarySearch(9) = 4
Array.BinarySearch(9) = 4
```

**Інтуїція росту** — приблизна кількість кроків:

| n | O(1) | O(log n) | O(n) | O(n log n) | O(n²) |
|---|------|----------|------|------------|-------|
| 10 | 1 | ~3 | 10 | ~33 | 100 |
| 1 000 | 1 | ~10 | 1 000 | ~10 000 | 1 000 000 |
| 1 000 000 | 1 | ~20 | 1 000 000 | ~20 000 000 | 10¹² (години) |

**Підраховуємо порівняння, lower bound і вбудовані методи.** Типові пастки бінарного пошуку:

- **Передумова:** масив має бути **відсортований**, інакше результат хибний (без помилки!).
- **Переповнення:** `(lo + hi) / 2` переповнює `int` для великих індексів — пишіть `lo + (hi - lo) / 2`.
- **Off-by-one:** для включних меж `[lo, hi]` умова `lo <= hi` і зсув `mid ± 1`; для напіввідкритих `[lo, hi)` — `lo < hi` і `hi = mid`. Змішування стилів дає пропущений елемент або нескінченний цикл.
- `Array.BinarySearch` / `List<T>.BinarySearch` повертають **від'ємне** число, якщо елемент не знайдено: `~result` — позиція для вставки.

```csharp
int[] sorted = [1, 3, 5, 7, 9, 11, 13, 15];
int[] big = Enumerable.Range(0, 1_000_000).Select(i => 2 * i).ToArray(); // 0, 2, 4, ... відсортований
Console.WriteLine($"Linear: index={LinearCount(big, 1_999_998, out int c1)}, порівнянь={c1}");
Console.WriteLine($"Binary: index={BinaryCount(big, 1_999_998, out int c2)}, порівнянь={c2}");
// lower bound: перший індекс, де arr[i] >= target (позиція для вставки)
int[] dup = [1, 3, 3, 3, 8];
Console.WriteLine($"LowerBound(3)={LowerBound(dup, 3)}, LowerBound(4)={LowerBound(dup, 4)}, LowerBound(100)={LowerBound(dup, 100)}");
// Вбудовані: якщо не знайдено — від'ємне число ~index (побітове доповнення точки вставки)
int r = Array.BinarySearch(sorted, 6);
Console.WriteLine($"Array.BinarySearch(6) = {r}, точка вставки = {~r}");
var list = new List<int>(sorted);
Console.WriteLine($"List.BinarySearch(13) = {list.BinarySearch(13)}");
// Помилка: бінарний пошук на невідсортованому масиві дає хибний результат
int[] unsorted = [9, 1, 7, 3, 5];
Console.WriteLine($"На невідсортованому: BinarySearch(9) = {Array.BinarySearch(unsorted, 9)}"); // від'ємне, хоча 9 є
// HashSet.Contains — O(1) у середньому, List.Contains — O(n)
var bigList = new List<int>(big);
var bigSet = new HashSet<int>(big);
var sw = System.Diagnostics.Stopwatch.StartNew();
for (int i = 0; i < 1000; i++) bigList.Contains(-1);        // 1000 × 1 000 000 порівнянь
long listMs = sw.ElapsedMilliseconds; sw.Restart();
for (int i = 0; i < 1000; i++) bigSet.Contains(-1);         // 1000 × ~1 обчислення хешу
long setMs = sw.ElapsedMilliseconds;
Console.WriteLine($"HashSet швидший за List: {setMs < listMs}");

static int LinearCount(int[] arr, int target, out int comparisons)
{
    comparisons = 0;
    for (int i = 0; i < arr.Length; i++)
    {
        comparisons++;
        if (arr[i] == target) return i;
    }
    return -1;
}
static int BinaryCount(int[] arr, int target, out int comparisons)
{
    comparisons = 0;
    int lo = 0, hi = arr.Length - 1;                 // межі включні: [lo, hi]
    while (lo <= hi)                                 // <=, інакше пропустимо останній елемент
    {
        int mid = lo + (hi - lo) / 2;                // (lo + hi) / 2 може переповнити int
        comparisons++;
        if (arr[mid] == target) return mid;
        if (arr[mid] < target) lo = mid + 1;         // +1 / -1 — інакше можливий нескінченний цикл
        else hi = mid - 1;
    }
    return -1;
}
static int LowerBound(int[] arr, int target)
{
    int lo = 0, hi = arr.Length;                     // напіввідкритий інтервал [lo, hi)
    while (lo < hi)
    {
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] < target) lo = mid + 1;
        else hi = mid;                               // mid може бути відповіддю — не відкидаємо
    }
    return lo;                                       // arr.Length, якщо всі елементи менші
}
```

**Приклад запуску:**
```
Linear: index=999999, порівнянь=1000000
Binary: index=999999, порівнянь=20
LowerBound(3)=1, LowerBound(4)=4, LowerBound(100)=5
Array.BinarySearch(6) = -4, точка вставки = 3
List.BinarySearch(13) = 6
На невідсортованому: BinarySearch(9) = -6
HashSet швидший за List: True
```

**Коли O(n) — нормально:**
- **Мале n** (десятки–сотні елементів): лінійний прохід простіший і часто швидший через кеш процесора.
- **Невідсортовані дані й один пошук:** сортування коштує O(n log n) — це дорожче за один лінійний пошук O(n). Сортувати (або будувати `HashSet`) вигідно, коли пошуків **багато**.
- Для частих перевірок «чи є елемент» — `HashSet<T>.Contains` за O(1) у середньому замість `List<T>.Contains` за O(n).

---

## 10. Форматування та стиль коду
Єдиний стиль робить код передбачуваним: читач одразу бачить, що є типом, полем чи локальною змінною. Нижче — загальноприйняті конвенції .NET (Microsoft / Roslyn).

| Що | Стиль | Приклад |
|----|-------|---------|
| Типи, методи, властивості, константи | `PascalCase` | `BinarySearch`, `MaxValue` |
| Локальні змінні, параметри | `camelCase` | `left`, `targetIndex` |
| Приватні поля | `_camelCase` | `_students` |
| Інтерфейси | префікс `I` | `IShape`, `IComparable<T>` |
| Параметри generic-типів | префікс `T` | `T`, `TKey`, `TValue` |
| Асинхронні методи | суфікс `Async` | `LoadAsync` |

- **Дужки в стилі Allman** — `{` на новому рядку; **відступ 4 пробіли**, без табуляцій.
- **Один тип — один файл**, ім'я файлу збігається з іменем типу (`Student.cs`).
- **File-scoped namespace:** `namespace Course.Lecture1;` замість блоку `{ }` — на один рівень відступу менше.
- `var` — коли тип **очевидний** з правої частини (`var list = new List<int>();`), інакше — явний тип.
- **Expression-bodied members** (`=>`) — для коротких однорядкових методів і властивостей.
- **Завжди фігурні дужки** для `if`/`for`/`while` у робочому коді — захист від помилок при додаванні рядка.
- **Global usings:** `global using System.Text;` в одному файлі (напр. `GlobalUsings.cs`) діє на весь проєкт; SDK вже додає неявні `System`, `System.Linq`, `System.Collections.Generic` тощо (`<ImplicitUsings>enable</ImplicitUsings>`).
- **Порядок `using`:** спочатку `System.*`, потім інші простори імен за алфавітом; `using` — поза `namespace`.

Погано — змішані стилі, K&R-дужки, `if` без дужок, неінформативні імена (компілюється, але читати важко):

```csharp
// Погано: змішані стилі іменування, K&R-дужки, if без дужок, незрозумілі імена
public class student_repo {
    private Dictionary<int, string> Students = new();
    public int count() { return Students.Count; }
    public void add(int ID, string n) {
        if (Students.Count > 100) throw new Exception("err");
        Students[ID] = n;
    }
}
```

Добре — той самий код за конвенціями:

```csharp
namespace Course.Lecture1;                               // file-scoped namespace: без зайвого рівня відступу

public interface IStudentRepository                      // інтерфейс — префікс I
{
    Task<Student?> FindAsync(int id);                    // асинхронний метод — суфікс Async
}

public sealed class InMemoryStudentRepository : IStudentRepository
{
    public const int MaxStudents = 100;                  // константа — PascalCase
    private readonly Dictionary<int, Student> _students = new(); // приватне поле — _camelCase

    public int Count => _students.Count;                 // expression-bodied властивість

    public void Add(Student student)                      // параметр — camelCase
    {
        if (_students.Count >= MaxStudents)               // фігурні дужки завжди, навіть для одного рядка
        {
            throw new InvalidOperationException("Забагато студентів");
        }

        _students[student.Id] = student;
    }

    public Task<Student?> FindAsync(int id)
    {
        var found = _students.GetValueOrDefault(id);     // var — тип очевидний з виразу
        return Task.FromResult(found);
    }
}

public record Student(int Id, string Name);

public static class Program
{
    public static async Task Main()
    {
        var repository = new InMemoryStudentRepository();
        repository.Add(new Student(1, "Olena"));
        Student? student = await repository.FindAsync(1);
        Console.WriteLine($"{student?.Name}, всього: {repository.Count}");
    }
}
```

**Приклад запуску:**
```
Olena, всього: 1
```

Правила фіксуються у файлі `.editorconfig` у корені репозиторію — IDE і `dotnet format` читають його автоматично:

```ini
root = true

[*.cs]
indent_style = space
indent_size = 4
csharp_new_line_before_open_brace = all              # Allman
csharp_style_namespace_declarations = file_scoped:warning
csharp_style_var_when_type_is_apparent = true:suggestion
csharp_prefer_braces = true:warning
dotnet_sort_system_directives_first = true
dotnet_style_qualification_for_field = false:suggestion
```

Автоматичне форматування всього проєкту:

```bash
dotnet format                        # виправити стиль і форматування
dotnet format --verify-no-changes    # перевірка в CI: помилка, якщо код не відформатовано
```

---

## 11. 15 сучасних можливостей C#
Можливості C# 7–12, які постійно трапляються в сучасному коді та в прикладах курсу. Кожен приклад — окрема програма з top-level statements.

### 11.1. Span<T> і ReadOnlySpan<T>
`Span<T>` — «вікно» на неперервну ділянку пам'яті (масив, рядок, стек): зрізи робляться **без алокацій і копіювання**. `stackalloc` виділяє буфер на стеку, а `ReadOnlySpan<char>` дозволяє парсити рядки без створення підрядків.

```csharp
int[] data = [10, 20, 30, 40, 50];
Span<int> middle = data.AsSpan(1, 3);            // "вікно" на елементи 1..3 без копіювання
middle[0] = 99;                                  // змінює вихідний масив
Console.WriteLine(string.Join(" ", data));       // 10 99 30 40 50
Span<int> buffer = stackalloc int[4];            // пам'ять на стеку — без GC
for (int i = 0; i < buffer.Length; i++) buffer[i] = i * i;
Console.WriteLine($"Сума buffer: {Sum(buffer)}");
ReadOnlySpan<char> text = "2026-09-15";
int year = int.Parse(text[..4]);                 // зріз рядка без створення нового string
Console.WriteLine($"Рік: {year}");
static int Sum(ReadOnlySpan<int> values)         // приймає масив, Span, stackalloc
{
    int total = 0;
    foreach (int v in values) total += v;
    return total;
}
```

**Приклад запуску:**
```
10 99 30 40 50
Сума buffer: 14
Рік: 2026
```

### 11.2. Memory<T>
`Span<T>` живе лише на стеку (не можна зберегти в полі чи використати після `await`). `Memory<T>` — його «довгоживучий» аналог; доступ до даних — через `.Span`.

```csharp
Memory<int> memory = new int[] { 1, 2, 3, 4, 5, 6 };  // Memory<T> можна зберігати в полях і між await
Memory<int> tail = memory[3..];                      // зріз без копіювання
int sum = await SumLaterAsync(tail);
Console.WriteLine($"Сума хвоста: {sum}");
static async Task<int> SumLaterAsync(ReadOnlyMemory<int> values)
{
    await Task.Delay(10);                            // Span тут заборонений, Memory — можна
    int total = 0;
    foreach (int v in values.Span) total += v;       // .Span — швидкий доступ, коли потрібно
    return total;
}
```

**Приклад запуску:**
```
Сума хвоста: 15
```

### 11.3. Індекси та діапазони (`^`, `..`)
`^n` — n-й елемент з кінця, `a..b` — діапазон `[a, b)`. Працює з масивами, рядками, `Span<T>`.

```csharp
int[] a = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9];
Console.WriteLine(a[^1]);                          // ^1 — останній елемент: 9
Console.WriteLine(string.Join(",", a[2..5]));      // [2, 5) — 2,3,4
Console.WriteLine(string.Join(",", a[^3..]));      // останні три: 7,8,9
Range firstHalf = ..(a.Length / 2);                // діапазон можна зберегти у змінну
Console.WriteLine(string.Join(",", a[firstHalf])); // 0,1,2,3,4
Console.WriteLine("Hello, world"[7..]);            // працює і з рядками: world
```

**Приклад запуску:**
```
9
2,3,4
7,8,9
0,1,2,3,4
world
```

### 11.4. record і `with`
`record` — тип з рівністю за значенням, автоматичним `ToString` і деконструкцією. `with` створює змінену копію незмінного об'єкта.

```csharp
var p1 = new Person("Andrii", 30);
var p2 = p1 with { Age = 31 };             // копія зі зміненою властивістю; p1 не змінюється
Console.WriteLine(p1);                     // ToString згенеровано автоматично
Console.WriteLine(p2);
Console.WriteLine(p1 == new Person("Andrii", 30)); // рівність за значенням: True
public record Person(string Name, int Age);        // позиційний record: незмінні властивості
```

**Приклад запуску:**
```
Person { Name = Andrii, Age = 30 }
Person { Name = Andrii, Age = 31 }
True
```

### 11.5. Pattern matching: property, list, relational
Патерни перевіряють форму даних: властивості `{ Total: > 1000 }`, списки `[1, .., var last]`, порівняння `>`/`<`, умови `when`.

```csharp
object[] items = [new Order("UA", 1500m), new Order("PL", 50m), new int[] { 1, 2, 3 }, 42];
foreach (object item in items) Console.WriteLine(Describe(item));
static string Describe(object o) => o switch
{
    Order { Country: "UA", Total: > 1000m } => "Велике замовлення з України", // property + relational
    Order { Total: < 100m } o2             => $"Мале замовлення: {o2.Total}",
    int[] and [1, .., var last]            => $"Масив з 1 на початку, останній {last}", // list pattern
    int n when n % 2 == 0                  => $"Парне число {n}",
    _                                      => "Інше"
};
public record Order(string Country, decimal Total);
```

**Приклад запуску:**
```
Велике замовлення з України
Мале замовлення: 50
Масив з 1 на початку, останній 3
Парне число 42
```

### 11.6. switch-вирази
`switch` як вираз: коротко, повертає значення, компілятор попереджає про неповне покриття. Можна перемикатися по кортежах.

```csharp
foreach (int score in new[] { 95, 78, 61, 30 })
    Console.Write($"{Grade(score)} ");
Console.WriteLine();
Console.WriteLine(Rotate((1, 0), "left"));
static char Grade(int score) => score switch  // вираз повертає значення, компілятор перевіряє повноту
{
    >= 90 => 'A',
    >= 75 => 'B',
    >= 60 => 'C',
    _     => 'F'
};
static (int X, int Y) Rotate((int X, int Y) v, string dir) => (v, dir) switch
{
    (_, "left")  => (-v.Y, v.X),            // switch по кортежу
    (_, "right") => (v.Y, -v.X),
    _ => throw new ArgumentException(dir)
};
```

**Приклад запуску:**
```
A B C F 
(0, 1)
```

### 11.7. Nullable reference types і `?.`, `??`, `??=`
`string?` явно дозволяє `null`, а компілятор попереджає про потенційний `NullReferenceException`. Оператори `?.`, `??`, `??=` роблять роботу з null короткою.

```csharp
#nullable enable
string? name = null;                      // ? — змінна може бути null
string title = "Lecture";                 // без ? — компілятор попередить при присвоєнні null
Console.WriteLine(name?.Length is null);  // ?. — якщо name == null, результат null: True
Console.WriteLine(name?.Length ?? -1);    // ?? — значення за замовчуванням: -1
name ??= "Anonymous";                     // ??= — присвоїти, лише якщо null
Console.WriteLine($"{title}: {name}");
List<int>? cache = null;
(cache ??= []).Add(7);                    // лінива ініціалізація
Console.WriteLine(cache.Count);
```

**Приклад запуску:**
```
True
-1
Lecture: Anonymous
1
```

### 11.8. Кортежі та деконструкція
`(int Min, int Max)` — легкий значущий тип для повернення кількох значень; деконструкція розкладає його у змінні.

```csharp
var (min, max) = MinMax([4, 9, 1, 7]);    // деконструкція кортежу в змінні
Console.WriteLine($"min={min}, max={max}");
int a = 1, b = 2;
(a, b) = (b, a);                          // обмін значень без тимчасової змінної
Console.WriteLine($"a={a}, b={b}");
var point = new Point3(1, 2, 3);
var (x, _, z) = point;                    // _ — ігноруємо значення
Console.WriteLine($"x={x}, z={z}");
static (int Min, int Max) MinMax(int[] values) => (values.Min(), values.Max()); // іменовані елементи
public record Point3(int X, int Y, int Z); // record автоматично має Deconstruct
```

**Приклад запуску:**
```
min=1, max=9
a=2, b=1
x=1, z=3
```

### 11.9. `init` і `required` властивості
`init` дозволяє присвоїти властивість лише під час створення об'єкта; `required` змушує задати її в ініціалізаторі (C# 11).

```csharp
var config = new ServerConfig { Host = "localhost", Port = 8080 };
// config.Port = 9090;                     // помилка: init — лише під час ініціалізації
// var bad = new ServerConfig { Port = 1 }; // помилка: required Host не задано
Console.WriteLine($"{config.Host}:{config.Port}, TLS={config.UseTls}");
public class ServerConfig
{
    public required string Host { get; init; } // required — обов'язково в ініціалізаторі
    public int Port { get; init; } = 80;        // init — лише для створення об'єкта
    public bool UseTls { get; init; }
}
```

**Приклад запуску:**
```
localhost:8080, TLS=False
```

### 11.10. Primary constructors (C# 12)
Параметри конструктора пишуться прямо після імені класу й доступні в усьому тілі — менше шаблонного коду для DI.

```csharp
var logger = new ConsoleLogger("APP");
var service = new OrderService(logger, 3);
service.Process("order-1");
public class ConsoleLogger(string prefix)          // параметри доступні в усьому тілі класу
{
    public void Log(string message) => Console.WriteLine($"[{prefix}] {message}");
}
public class OrderService(ConsoleLogger logger, int maxRetries)
{
    public int MaxRetries { get; } = maxRetries;   // ініціалізація властивості з параметра
    public void Process(string id) => logger.Log($"Обробка {id}, спроб: {MaxRetries}");
}
```

**Приклад запуску:**
```
[APP] Обробка order-1, спроб: 3
```

### 11.11. Collection expressions і spread `..` (C# 12)
Єдиний синтаксис `[1, 2, 3]` для масивів, `List<T>`, `HashSet<T>`, `Span<T>`; `..` вставляє елементи іншої колекції.

```csharp
int[] first = [1, 2, 3];                       // масив
List<int> second = [4, 5];                     // той самий синтаксис для List<T>
int[] all = [0, .. first, .. second, 6];       // .. — spread: вставити всі елементи
Console.WriteLine(string.Join(" ", all));      // 0 1 2 3 4 5 6
HashSet<string> tags = ["c#", "dotnet", "c#"]; // дублікати відкидає сам HashSet
Console.WriteLine(tags.Count);                 // 2
int[] empty = [];                              // порожня колекція
Console.WriteLine(empty.Length);
```

**Приклад запуску:**
```
0 1 2 3 4 5 6
2
0
```

### 11.12. Raw string literals `"""`
Багаторядкові рядки без екранування лапок і `\`; відступ визначається закривальними `"""`. `$$` змінює синтаксис інтерполяції на `{{ }}` — зручно для JSON.

```csharp
string name = "Andrii";
string json = $$"""
    {
      "name": "{{name}}",
      "path": "C:\temp\file.txt"
    }
    """;                                  // $$ — інтерполяція через {{ }}, а { } — звичайні символи
Console.WriteLine(json);                  // відступ до закривальних """ обрізається
string sql = """SELECT * FROM "Users" WHERE Id = 1""";  // лапки без екранування
Console.WriteLine(sql);
```

**Приклад запуску:**
```
{
  "name": "Andrii",
  "path": "C:\temp\file.txt"
}
SELECT * FROM "Users" WHERE Id = 1
```

### 11.13. async/await
`async`/`await` дозволяють чекати на I/O без блокування потоку; `Task.WhenAll` запускає кілька операцій одночасно.

```csharp
var sw = System.Diagnostics.Stopwatch.StartNew();
Task<string> a = DownloadAsync("a.txt", 300);   // обидві задачі стартують одразу
Task<string> b = DownloadAsync("b.txt", 200);
string[] results = await Task.WhenAll(a, b);    // чекаємо обидві паралельно
Console.WriteLine(string.Join(", ", results));
Console.WriteLine($"Паралельно: {sw.ElapsedMilliseconds < 450}"); // ~300 мс, а не 500
static async Task<string> DownloadAsync(string file, int ms)
{
    await Task.Delay(ms);                       // потік не блокується під час очікування
    return $"{file} ({ms} мс)";
}
```

**Приклад запуску:**
```
a.txt (300 мс), b.txt (200 мс)
Паралельно: True
```

### 11.14. Generic math: `INumber<T>` (C# 11)
Статичні абстрактні члени інтерфейсів дозволяють писати один узагальнений алгоритм для `int`, `double`, `decimal` тощо.

```csharp
using System.Numerics;
Console.WriteLine(SumAll([1, 2, 3]));             // int
Console.WriteLine(SumAll([1.5, 2.25]));           // double
Console.WriteLine(SumAll([0.1m, 0.2m]));          // decimal
static T SumAll<T>(T[] values) where T : INumber<T> // один метод для всіх числових типів
{
    T sum = T.Zero;                               // статичний абстрактний член інтерфейсу
    foreach (T v in values) sum += v;             // оператор + доступний через INumber<T>
    return sum;
}
```

**Приклад запуску:**
```
6
3.75
0.3
```

### 11.15. Ключове слово `field` (C# 14)
Напівавтоматичні властивості: у `get`/`set` можна звертатися до згенерованого компілятором поля через `field`, не оголошуючи `_name` вручну. Потрібен C# 14 (.NET 10+); на старших SDK — `<LangVersion>14</LangVersion>` або новіше в `.csproj`.

```csharp
var user = new User { Name = "  Andrii  " };
Console.WriteLine($"[{user.Name}]");        // пробіли обрізано в set
Console.WriteLine(user.Tags.Count);         // список створено при першому зверненні
user.Tags.Add("admin");
Console.WriteLine(user.Tags.Count);
public class User
{
    public string Name { get; set => field = value.Trim(); } = ""; // field — прихований backing field
    public List<string> Tags => field ??= [];                      // лінива ініціалізація без окремого поля
}
```

**Приклад запуску:**
```
[Andrii]
0
1
```

---

## 12. Підсумки
- C# має **фіксовані розміри** типів, `const` (компіляція) і `readonly` (виконання), статичний `var`.
- `switch`-вирази та патерни (`is int k and > 0`) замінюють громіздкі ланцюжки `if`.
- Масиви: `int[]`, прямокутні `int[,]`, зубчасті `int[][]`; `foreach` — зручний обхід.
- `ref`/`out`/`in` замінюють посилання та вказівники C++; пам'ять звільняє GC.
- ООП: один базовий клас + інтерфейси; `record` — рівність за значенням; `string?` — явний null.
- Винятки — `try/catch/finally`; LINQ — декларативна обробка колекцій.
- Бінарний пошук O(log n) на відсортованих даних радикально швидший за лінійний O(n); стежте за `lo + (hi - lo) / 2` і межами циклу, для частих пошуків — `HashSet<T>`.
- Стиль: `PascalCase` для типів і методів, `camelCase` для локальних, `_camelCase` для полів; правила — у `.editorconfig`, перевірка — `dotnet format`.
- `Span<T>`/`Memory<T>` дають зрізи без алокацій; `record`, `with`, `init`/`required`, primary constructors, collection expressions і `field` (C# 14) скорочують шаблонний код.
- Патерни, switch-вирази, nullable-анотації, `async/await` і generic math (`INumber<T>`) — основа виразного та безпечного сучасного C#.

---

## 13. Питання для самоперевірки
1. Чим `const` відрізняється від `readonly` і коли можна використати лише друге?
2. Що виведе програма, якщо передати `int` у метод без `ref` і змінити його всередині? А з `ref`?
3. Чим `int[,]` відрізняється від `int[][]` за будовою та `Length`?
4. Чому `p1 == p2` дає `True` для `record`, але для звичайного `class` з тими самими полями — `False`?
5. Чому бінарний пошук вимагає відсортованого масиву і скільки кроків він робить для n = 1 000 000?
6. Які правила іменування застосовуються до приватного поля, параметра методу, інтерфейсу та асинхронного методу?
7. Чому `Span<T>` не можна використати після `await`, і що використати натомість?
8. Що повернуть `a[^1]` та `a[2..5]` для масиву `[0..9]`?
9. Чим `init` відрізняється від `set`, а `required` — від звичайної властивості?
10. Як за допомогою `INumber<T>` написати один метод суми для `int`, `double` і `decimal`?
11. Що повертає `Array.BinarySearch`, якщо елемента немає, і як отримати з результату позицію для вставки?
12. Коли лінійний пошук O(n) доречніший, ніж сортування + бінарний пошук?
