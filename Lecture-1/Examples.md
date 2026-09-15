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
10. [Підсумки](#10-підсумки)
11. [Питання для самоперевірки](#11-питання-для-самоперевірки)

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

---

## 10. Підсумки
- C# має **фіксовані розміри** типів, `const` (компіляція) і `readonly` (виконання), статичний `var`.
- `switch`-вирази та патерни (`is int k and > 0`) замінюють громіздкі ланцюжки `if`.
- Масиви: `int[]`, прямокутні `int[,]`, зубчасті `int[][]`; `foreach` — зручний обхід.
- `ref`/`out`/`in` замінюють посилання та вказівники C++; пам'ять звільняє GC.
- ООП: один базовий клас + інтерфейси; `record` — рівність за значенням; `string?` — явний null.
- Винятки — `try/catch/finally`; LINQ — декларативна обробка колекцій.
- Бінарний пошук O(log n) на відсортованих даних радикально швидший за лінійний O(n).

---

## 11. Питання для самоперевірки
1. Чим `const` відрізняється від `readonly` і коли можна використати лише друге?
2. Що виведе програма, якщо передати `int` у метод без `ref` і змінити його всередині? А з `ref`?
3. Чим `int[,]` відрізняється від `int[][]` за будовою та `Length`?
4. Чому `p1 == p2` дає `True` для `record`, але для звичайного `class` з тими самими полями — `False`?
5. Чому бінарний пошук вимагає відсортованого масиву і скільки кроків він робить для n = 1 000 000?
