# Лекція 1 — Нагадування: можливості C#

---

## Зміст
1. [Вступ: курс і мова C#](#1-вступ-курс-і-мова-c)
2. [Мінімальна програма](#2-мінімальна-програма)
3. [Типи даних, const, readonly, var](#3-типи-даних-const-readonly-var)
4. [Умови, switch і pattern matching](#4-умови-switch-і-pattern-matching)
   - [if/else, класичний switch, switch-вирази](#41-if--else-і-тернарний-оператор)
   - [Види патернів: базові, property, позиційні, list](#44-базові-патерни-константа-тип-var-_-реляційні-логічні)
   - [Практичні приклади, порядок гілок, типові помилки, вправи](#48-практичні-приклади)
5. [Цикли та масиви](#5-цикли-та-масиви)
6. [Методи: ref, out, in, необов'язкові параметри](#6-методи-ref-out-in-необовязкові-параметри)
7. [Класи, record, struct, властивості, nullable](#7-класи-record-struct-властивості-nullable)
8. [Винятки та LINQ](#8-винятки-та-linq)
9. [Алгоритми: O(n) проти O(log n)](#9-алгоритми-on-проти-olog-n)
10. [Форматування та стиль коду](#10-форматування-та-стиль-коду)
11. [Робота з файлами](#11-робота-з-файлами)
12. [Власні колекції: мінімальні реалізації](#12-власні-колекції-мінімальні-реалізації)
13. [15 сучасних можливостей C#](#13-15-сучасних-можливостей-c)
14. [Підсумки](#14-підсумки)
15. [Питання для самоперевірки](#15-питання-для-самоперевірки)

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
Розгалуження в C# схожі на C++, але суворіші: умова завжди має тип `bool`, «провалювання» між `case` заборонене, а **pattern matching** (патерни) дозволяє перевіряти не лише значення, а й **форму** даних — тип, властивості, елементи списку. Компілятор перевіряє патерни на повноту та недосяжні гілки.

### 4.1. `if` / `else` і тернарний оператор
- Умова — лише `bool`: `if (count)` не компілюється (CS0029), пишіть `if (count != 0)`.
- `&&` і `||` обчислюються **з коротким замиканням**: права частина не виконується, якщо результат уже відомий.
- `умова ? a : b` — **вираз** (повертає значення), обидві гілки мають зводитися до спільного типу.

```csharp
int temperature = 23;
if (temperature < 0)
{
    Console.WriteLine("Мороз");
}
else if (temperature < 20)                                  // else if — лише ланцюжок вкладених if
{
    Console.WriteLine("Прохолодно");
}
else
{
    Console.WriteLine("Тепло");
}
int age = 17;
string status = age >= 18 ? "повнолітній" : "неповнолітній"; // тернарний оператор — це вираз, а не оператор
Console.WriteLine(status);
string? nickname = null;
Console.WriteLine(nickname is null ? "без ніку" : nickname.ToUpper()); // у гілці «інакше» компілятор знає, що nickname не null
int x = 5, y = 0;
if (y != 0 && x / y > 1)                                    // && — коротке замикання: ділення на 0 не виконується
{
    Console.WriteLine("не виконається");
}
Console.WriteLine("&& зупинився на першій хибній умові");
// if (age) { }                                             // CS0029: int не перетворюється на bool, на відміну від C++
if (int.TryParse("abc", out int n))                         // TryParse не кидає виняток: bool + результат через out
{
    Console.WriteLine(n);
}
else
{
    Console.WriteLine("Не число");
}
int count = 3;
Console.WriteLine($"{count} {(count == 1 ? "файл" : "файли")}"); // тернарний у інтерполяції — у дужках
```

**Приклад запуску:**
```
Тепло
неповнолітній
без ніку
&& зупинився на першій хибній умові
Не число
3 файли
```

### 4.2. Класичний `switch`-оператор
- Кожна секція має завершуватися `break`, `return`, `continue`, `throw` або `goto` — інакше помилка **CS0163**.
- Кілька міток поспіль без коду між ними (`case A: case B:`) — дозволено, це єдиний «законний» fall-through.
- `goto case X` / `goto default` — явний перехід до іншої секції.
- `case` приймає будь-який патерн, а `when` додає довільну умову (guard).

```csharp
// Калькулятор: рядок замість Console.ReadLine(), щоб приклад був відтворюваним
string input = "3 + 5";
string[] parts = input.Split(' ');                          // ["3", "+", "5"]
double a = double.Parse(parts[0]);
char op = parts[1][0];
double b = double.Parse(parts[2]);
switch (op)
{
    case '+':
        Console.WriteLine(a + b);
        break;                                              // break обов'язковий: «провалювання» заборонене
    case '-':
        Console.WriteLine(a - b);
        break;
    case '/' when b == 0:                                   // when — додаткова умова (guard)
        Console.WriteLine("Помилка: ділення на 0");
        break;
    case '/':
        Console.WriteLine(a / b);
        break;
    default:                                                // default може стояти будь-де, але зазвичай — останнім
        Console.WriteLine("Невідомий оператор");
        break;
}
// Кілька міток на одну секцію: порожні case «склеюються» — це єдиний дозволений fall-through
foreach (DayOfWeek day in new[] { DayOfWeek.Saturday, DayOfWeek.Wednesday })
{
    switch (day)
    {
        case DayOfWeek.Saturday:
        case DayOfWeek.Sunday:
            Console.WriteLine($"{day}: вихідний");
            break;
        default:
            Console.WriteLine($"{day}: робочий");
            break;
    }
}
// goto case — явний перехід до іншої мітки (свідомий «fall-through»)
foreach (string level in new[] { "critical", "warning" })
{
    switch (level)
    {
        case "critical":
            Console.Write("[SMS] ");
            goto case "error";
        case "error":
            Console.Write("[email] ");
            goto case "warning";
        case "warning":
            Console.WriteLine("[log]");
            break;
    }
}
// Патерни в case: тип + змінна + when
object?[] values = [42, -7, "hi", null];
foreach (object? v in values)
{
    switch (v)
    {
        case int number when number > 0:                    // змінна number видима лише в цій секції
            Console.WriteLine($"додатне {number}");
            break;
        case int number:                                    // те саме ім'я в іншій секції — дозволено
            Console.WriteLine($"недодатне {number}");
            break;
        case string s:
            Console.WriteLine($"рядок \"{s}\"");
            continue;                                       // секцію можна завершити й continue/return/throw
        case null:
            Console.WriteLine("null");
            break;
    }
}
```

**Приклад запуску:**
```
8
Saturday: вихідний
Wednesday: робочий
[SMS] [email] [log]
[log]
додатне 42
недодатне -7
рядок "hi"
null
```

Типова помилка з C++ — забутий `break`:

```csharp
// НЕ КОМПІЛЮЄТЬСЯ: CS0163 Control cannot fall through from one case label ('case 1:') to another
int x = 1;
switch (x)
{
    case 1:
        Console.WriteLine("один");
    case 2:
        Console.WriteLine("два");
        break;
}
```

### 4.3. `switch`-вирази
`значення switch { патерн => результат, ... }` — вираз, що **повертає** значення. Порівняно з оператором: немає `case`/`break`, гілки розділяються комами, `_` замінює `default`, у гілці можна кинути виняток через throw-вираз.

- Якщо гілки покривають не всі значення — попередження **CS8509** (або **CS8846**, якщо «дірку» може закрити лише `when`), а під час виконання — `SwitchExpressionException`.
- Для `enum` потрібна гілка `_` навіть якщо перелічені всі імена: `(DayOfWeek)42` — теж валідне значення (CS8524).

```csharp
using System.Runtime.CompilerServices;                     // SwitchExpressionException
Console.WriteLine($"{Sign(7)} {Sign(-2)} {Sign(0)}");
Console.WriteLine($"{DayKind(DayOfWeek.Sunday)}, {DayKind(DayOfWeek.Monday)}");
Console.WriteLine(ParseColor("GREEN"));
try
{
    Console.WriteLine(ParseColor("purple"));
}
catch (ArgumentException e)
{
    Console.WriteLine(e.Message);
}
// Неповний switch-вираз: попередження CS8509, а під час виконання — SwitchExpressionException
int code = 3;
try
{
#pragma warning disable CS8509                              // вимикаємо лише для демонстрації
    string word = code switch
    {
        1 => "один",
        2 => "два"
    };
#pragma warning restore CS8509
    Console.WriteLine(word);
}
catch (SwitchExpressionException e)
{
    Console.WriteLine($"{e.GetType().Name}: {e.UnmatchedValue}");
}

static string Sign(int x) => x switch                       // switch-вираз: значення => результат
{
    > 0 => "Positive",                                      // гілки розділяються комами
    < 0 => "Negative",
    _ => "Zero"                                             // _ (discard) — будь-яке інше значення
};
static string DayKind(DayOfWeek day) => day switch
{
    DayOfWeek.Saturday or DayOfWeek.Sunday => "вихідний",   // or замість кількох case-міток
    _ => "робочий"
};
static ConsoleColor ParseColor(string name) => name.ToLowerInvariant() switch
{
    "red" => ConsoleColor.Red,
    "green" => ConsoleColor.Green,
    _ => throw new ArgumentException($"Невідомий колір: {name}", nameof(name)) // throw-вираз у гілці
};
```

**Приклад запуску:**
```
Positive Negative Zero
вихідний, робочий
Green
Невідомий колір: purple (Parameter 'name')
SwitchExpressionException: 3
```

### 4.4. Базові патерни: константа, тип, `var`, `_`, реляційні, логічні
| Патерн | Приклад | Збігається, якщо |
|--------|---------|------------------|
| Константний | `null`, `0`, `"red"`, `Color.Red` | значення дорівнює константі |
| Тип | `int`, `string` | значення має цей тип (і не `null`) |
| Declaration | `int n`, `string s` | тип збігся → значення в нову змінну |
| `var` | `var x` | **завжди** (навіть `null`), захоплює значення |
| Discard | `_` | завжди, значення не потрібне |
| Реляційний | `> 0`, `<= 'z'` | порівняння з **константою** |
| Логічні | `and`, `or`, `not` | комбінація патернів (`not` > `and` > `or` за пріоритетом) |
| Дужки | `(> 0 and < 10) or 100` | групування |

`x is null` / `x is not null` — рекомендована перевірка на `null`: на відміну від `==`, її не можна «зламати» перевантаженим оператором.

```csharp
object?[] things = [null, 0, 7, 42, 13, "", "привіт", 'Q', '9', true, 2.5m];
foreach (object? t in things)
{
    Console.WriteLine($"{t ?? "null",-7} → {Classify(t)}");
}
object value = 42;
if (value is int k and >= 10 and <= 99)                     // тип + реляційні + логічний and
{
    Console.WriteLine($"Двозначне ціле: {k}");
}
Console.WriteLine(IsVowel('е') ? "голосна" : "приголосна");
Console.WriteLine(Twice("5"));
Console.WriteLine(Twice(5));

static string Classify(object? o) => o switch
{
    null => "null",                                         // константний патерн
    0 => "нуль",                                            // константа: o is int і дорівнює 0
    int n and (> 0 and < 10) => $"цифра {n}",               // declaration + логічний + дужки
    int n when n % 2 == 0 => $"парне int {n}",              // declaration + when
    int => "непарне int",                                   // type-патерн без змінної
    string { Length: 0 } => "порожній рядок",               // тип + property-патерн
    string s => $"рядок довжини {s.Length}",
    char c and (>= 'a' and <= 'z' or >= 'A' and <= 'Z') => $"латинська літера {c}",
    bool => "bool",
    var other => $"щось інше: {other.GetType().Name}"       // var-патерн: збігається завжди (і з null)
};
static bool IsVowel(char c) => c is 'а' or 'е' or 'є' or 'и' or 'і' or 'ї' or 'о' or 'у' or 'ю' or 'я'; // коротше за 10 порівнянь через ||
static int Twice(object o)
{
    if (o is not int number)                                // not: змінна призначена, коли умова хибна
    {
        return -1;
    }
    return number * 2;                                      // тут number гарантовано ініціалізований
}
```

**Приклад запуску:**
```
null    → null
0       → нуль
7       → цифра 7
42      → парне int 42
13      → непарне int
        → порожній рядок
привіт  → рядок довжини 6
Q       → латинська літера Q
9       → щось інше: Char
True    → bool
2.5     → щось інше: Decimal
Двозначне ціле: 42
голосна
-1
10
```

### 4.5. Property-патерни
`{ Властивість: патерн, ... }` перевіряє властивості об'єкта (і неявно — що він не `null`). Патерни вкладаються: `{ Address: { City: "Київ" } }` або коротше — **extended property pattern** `{ Address.City: "Київ" }` (C# 10). Порожній `{ }` означає «будь-що, крім `null`».

```csharp
Person[] people =
[
    new("Олена", 19, new Address("Київ", "Хрещатик")),
    new("Андрій", 34, new Address("Львів", "Ринок")),
    new("Ірина", 70, null),
    new("Тарас", 15, new Address("Київ", "Лесі Українки")),
    new("Марко", 65, new Address("Одеса", "Дерибасівська"))
];
foreach (Person p in people)
{
    Console.WriteLine($"{p.Name}: {Describe(p)}");
}
if (people[1].Address is { } address)                       // { } — «не null» + змінна
{
    Console.WriteLine($"Адреса Андрія: {address.City}, {address.Street}");
}
string word = "патерн";
Console.WriteLine(word is { Length: > 3 and < 10 } ? "середнє слово" : "інше");

static string Describe(Person p) => p switch
{
    { Address: null } => "адреса невідома",
    { Age: < 18, Address.City: "Київ" } => "неповнолітній киянин",   // extended property pattern (C# 10)
    { Address: { City: "Київ" } } => "киянин",                     // вкладена форма того самого
    { Age: >= 18 and < 60, Address.City: var city } => $"працездатний, місто {city}", // var захоплює значення
    { Name: [var initial, ..], Age: >= 60 } => $"пенсіонер, ініціал {initial}", // property + list-патерн
    _ => "інше"
};
public record Address(string City, string Street);
public record Person(string Name, int Age, Address? Address);
```

**Приклад запуску:**
```
Олена: киянин
Андрій: працездатний, місто Львів
Ірина: адреса невідома
Тарас: неповнолітній киянин
Марко: пенсіонер, ініціал М
Адреса Андрія: Львів, Ринок
середнє слово
```

### 4.6. Позиційні та кортежні патерни
**Позиційний** патерн `(a, b)` викликає `Deconstruct`: `record` генерує його автоматично, для звичайного класу — пишемо самі. **Кортежний** патерн перемикається одразу за кількома значеннями: `(a, b) switch { ... }` — ідеально для таблиць рішень і скінченних автоматів.

```csharp
Point[] points = [new(0, 0), new(5, 0), new(2, 3), new(-1, 4), new(-2, -2), new(3, -1)];
Console.WriteLine(string.Join("; ", points.Select(Quadrant)));
Fraction[] fractions = [new(0, 5), new(6, 3), new(1, 0), new(3, 4)];
Console.WriteLine(string.Join("; ", fractions.Select(DescribeFraction)));
// Кортежні патерни: камінь-ножиці-папір
Console.WriteLine(Play(Move.Rock, Move.Scissors));
Console.WriteLine(Play(Move.Rock, Move.Paper));
Console.WriteLine(Play(Move.Paper, Move.Paper));
// Скінченний автомат світлофора: (стан, подія) → новий стан
Light light = Light.Red;
Console.Write(light);
foreach (Signal signal in new[] { Signal.Timer, Signal.Timer, Signal.Timer, Signal.Emergency, Signal.Timer })
{
    light = Next(light, signal);
    Console.Write($" -{signal}-> {light}");
}
Console.WriteLine();

static string Quadrant(Point p) => p switch
{
    (0, 0) => "початок",                                    // позиційний патерн: викликає Deconstruct
    (_, 0) or (0, _) => "на осі",                           // _ — будь-яка координата
    ( > 0, > 0) => "I",
    ( < 0, > 0) => "II",
    ( < 0, < 0) => "III",
    _ => "IV"
};
static string DescribeFraction(Fraction f) => f switch
{
    (_, 0) => "некоректний дріб",
    (0, _) => "нуль",
    var (n, d) when n % d == 0 => $"ціле {n / d}",          // var (n, d) — деконструкція у змінні
    (var n, var d) => $"{n}/{d}"
};
static string Play(Move first, Move second) => (first, second) switch
{
    var (a, b) when a == b => "нічия",
    (Move.Rock, Move.Scissors) or (Move.Scissors, Move.Paper) or (Move.Paper, Move.Rock) => "виграв перший",
    _ => "виграв другий"
};
static Light Next(Light current, Signal signal) => (current, signal) switch
{
    (_, Signal.Emergency) => Light.Red,                     // аварія — з будь-якого стану
    (Light.Red, Signal.Timer) => Light.RedYellow,
    (Light.RedYellow, Signal.Timer) => Light.Green,
    (Light.Green, Signal.Timer) => Light.Yellow,
    (Light.Yellow, Signal.Timer) => Light.Red,
    _ => throw new ArgumentOutOfRangeException(nameof(current)) // enum може містити й неназвані значення
};
public readonly record struct Point(int X, int Y);           // record сам генерує Deconstruct(out int X, out int Y)
public sealed class Fraction(int numerator, int denominator)
{
    public void Deconstruct(out int numerator1, out int denominator1) // звичайний клас: Deconstruct пишемо вручну
    {
        numerator1 = numerator;
        denominator1 = denominator;
    }
}
public enum Move { Rock, Paper, Scissors }
public enum Light { Red, RedYellow, Green, Yellow }
public enum Signal { Timer, Emergency }
```

**Приклад запуску:**
```
початок; на осі; I; II; III; IV
нуль; ціле 2; некоректний дріб; 3/4
виграв перший
виграв другий
нічия
Red -Timer-> RedYellow -Timer-> Green -Timer-> Yellow -Emergency-> Red -Timer-> RedYellow
```

### 4.7. List-патерни (C# 11)
`[p1, p2, ..]` перевіряє довжину та елементи будь-якого типу з `Length`/`Count` та індексатором: масивів, `List<T>`, `string`, `Span<T>`. `..` — «нуль або більше елементів» (не більше одного на патерн); `.. var middle` — **slice**-патерн, що захоплює зріз (потрібен індексатор з `Range`).

```csharp
int[][] arrays = [[], [7], [1, 2], [1, 2, 3, 4], [9, 0, 9], [5, 6, 7, 8]];
foreach (int[] arr in arrays)
{
    Console.WriteLine($"[{string.Join(", ", arr)}] → {Describe(arr)}");
}
List<int> list = [3, 1, 4];
Console.WriteLine(list is [_, _, _] ? "List з 3 елементів" : "інше"); // List<T> теж підтримує list-патерни
// Рядки та span: елементи — char
foreach (string file in new[] { "Program.cs", "README.md", "\"quoted\"", "x" })
{
    Console.WriteLine($"{file} → {Kind(file)}");
}
ReadOnlySpan<char> span = "2025-09-15".AsSpan();
if (span is [var c1, var c2, var c3, var c4, '-', ..])       // span: без алокацій
{
    Console.WriteLine($"Рік: {c1}{c2}{c3}{c4}");
}

static string Describe(int[] a) => a switch
{
    [] => "порожній",
    [var single] => $"один елемент {single}",
    [1, 2] => "рівно [1, 2]",
    [1, 2, ..] => "починається з 1, 2",                     // .. — нуль або більше елементів
    [var first, .., var last] when first == last => $"однакові краї {first}",
    [var first, .. var middle, var last] => $"перший {first}, середина [{string.Join(", ", middle)}], останній {last}" // slice-патерн
};
static string Kind(string s) => s switch
{
    [.., '.', 'c', 's'] => "C# файл",
    [.., '.', 'm', 'd'] => "Markdown",
    ['"', .. var inner, '"'] => $"у лапках: {inner}",       // slice рядка — теж string
    _ => "невідомо"
};
```

**Приклад запуску:**
```
[] → порожній
[7] → один елемент 7
[1, 2] → рівно [1, 2]
[1, 2, 3, 4] → починається з 1, 2
[9, 0, 9] → однакові краї 9
[5, 6, 7, 8] → перший 5, середина [6, 7], останній 8
List з 3 елементів
Program.cs → C# файл
README.md → Markdown
"quoted" → у лапках: quoted
x → невідомо
Рік: 2025
```

### 4.8. Практичні приклади
Оцінки за діапазонами, площі фігур над ієрархією `record`, HTTP-статуси, FizzBuzz через кортеж та обробка `null`:

```csharp
// 1. Оцінка за діапазонами балів
Console.WriteLine(string.Join(" ", new[] { 100, 91, 75, 64, 12 }.Select(Grade)));
try
{
    Grade(101);
}
catch (ArgumentOutOfRangeException e)
{
    Console.WriteLine(e.ParamName);
}
// 2. Площі фігур: type- і позиційні патерни над ієрархією record
Shape[] shapes = [new Circle(1), new Rectangle(2, 3), new Rectangle(4, 4), new Triangle(3, 4, 5)];
foreach (Shape shape in shapes)
{
    Console.WriteLine($"{Name(shape)}: {Area(shape):F2}");
}
// 3. HTTP-статуси
Console.WriteLine(string.Join(" | ", new[] { 200, 204, 301, 404, 418, 503, 102, 999 }.Select(Http)));
// 4. FizzBuzz через кортеж
Console.WriteLine(string.Join(" ", Enumerable.Range(1, 15).Select(FizzBuzz)));
// 5. Обробка null
foreach (string? name in new[] { null, "", "   ", "Олена", "Максиміліанна" })
{
    Console.WriteLine(Greet(name));
}

static string Grade(int score) => score switch
{
    < 0 or > 100 => throw new ArgumentOutOfRangeException(nameof(score)), // спершу — некоректні значення
    >= 90 => "A",
    >= 75 => "B",
    >= 60 => "C",
    _ => "F"
};
static double Area(Shape shape) => shape switch
{
    Circle c => Math.PI * c.Radius * c.Radius,              // type-патерн зі змінною
    Rectangle(var w, var h) => w * h,                       // позиційний: record має Deconstruct
    Triangle(var a, var b, var c) => Heron(a, b, c),
    _ => throw new ArgumentException($"Невідома фігура {shape}")
};
static string Name(Shape shape) => shape switch
{
    Circle => "коло",
    Rectangle(var w, var h) when w == h => "квадрат",       // той самий тип, але уточнення через when
    Rectangle => "прямокутник",
    Triangle(var a, var b, var c) when a * a + b * b == c * c => "прямокутний трикутник",
    Triangle => "трикутник",
    _ => "?"
};
static double Heron(double a, double b, double c)
{
    double p = (a + b + c) / 2;
    return Math.Sqrt(p * (p - a) * (p - b) * (p - c));
}
static string Http(int code) => code switch
{
    200 or 201 or 204 => $"{code} OK",
    >= 200 and < 300 => $"{code} успіх",
    >= 300 and < 400 => $"{code} перенаправлення",
    404 => $"{code} не знайдено",
    >= 400 and < 500 => $"{code} помилка клієнта",
    >= 500 and < 600 => $"{code} помилка сервера",
    >= 100 and < 200 => $"{code} інформаційний",
    _ => $"{code} некоректний"
};
static string FizzBuzz(int i) => (i % 3, i % 5) switch
{
    (0, 0) => "FizzBuzz",
    (0, _) => "Fizz",
    (_, 0) => "Buzz",
    _ => i.ToString()
};
static string Greet(string? name) => name switch
{
    null => "Привіт, незнайомцю",                           // null — перша гілка, далі name не null
    { Length: 0 } => "Порожнє ім'я",
    _ when string.IsNullOrWhiteSpace(name) => "Лише пробіли",
    { Length: > 10 } => $"Привіт, {name[..10]}…",
    _ => $"Привіт, {name}"
};
public abstract record Shape;
public sealed record Circle(double Radius) : Shape;
public sealed record Rectangle(double Width, double Height) : Shape;
public sealed record Triangle(double A, double B, double C) : Shape;
```

**Приклад запуску:**
```
A A B C F
score
коло: 3.14
прямокутник: 6.00
квадрат: 16.00
прямокутний трикутник: 6.00
200 OK | 204 OK | 301 перенаправлення | 404 не знайдено | 418 помилка клієнта | 503 помилка сервера | 102 інформаційний | 999 некоректний
1 2 Fizz 4 Buzz Fizz 7 8 Fizz Buzz 11 Fizz 13 14 FizzBuzz
Привіт, незнайомцю
Порожнє ім'я
Лише пробіли
Привіт, Олена
Привіт, Максиміліа…
```

Розбір текстових команд через list-патерни та простий обчислювач виразів над деревом `record` (рекурсивні позиційні патерни):

```csharp
// Розбір команд: Split + list-патерни
string[] commands = ["", "help", "add milk", "add eggs 12", "add eggs -1", "del a b c", "move box to kitchen", "jump"];
foreach (string line in commands)
{
    Console.WriteLine($"'{line}' → {Execute(line)}");
}
// Обчислювач виразів: (2 + 3) * 4 + 0
Expr expr = new Add(new Mul(new Add(new Num(2), new Num(3)), new Num(4)), new Num(0));
Console.WriteLine($"{Show(expr)} = {Eval(expr)}");
Expr simple = Simplify(expr);
Console.WriteLine($"Спрощено: {Show(simple)} = {Eval(simple)}");
Console.WriteLine(Show(Simplify(new Mul(new Num(1), new Mul(new Num(7), new Num(0))))));

static string Execute(string line) => line.Split(' ', StringSplitOptions.RemoveEmptyEntries) switch
{
    [] => "порожня команда",
    ["help"] => "команди: add, del, move",
    ["add", var item] => $"додано {item}",
    ["add", var item, var qty] when int.TryParse(qty, out int n) && n > 0 => $"додано {item} × {n}",
    ["add", ..] => "некоректна кількість",
    ["del", .. var items] when items.Length > 0 => $"видалено: {string.Join(", ", items)}",
    ["move", var what, "to", var where] => $"{what} переміщено в {where}",
    [var command, ..] => $"невідома команда '{command}'"
};
static double Eval(Expr e) => e switch
{
    Num(var value) => value,
    Add(var left, var right) => Eval(left) + Eval(right),   // рекурсія по дереву
    Mul(var left, var right) => Eval(left) * Eval(right),
    _ => throw new NotSupportedException(e.GetType().Name)
};
static Expr Simplify(Expr e) => e switch
{
    Add(var l, var r) => (Simplify(l), Simplify(r)) switch
    {
        (Num(0), var x) => x,                               // 0 + x = x
        (var x, Num(0)) => x,                               // x + 0 = x (var не можна оголошувати всередині or)
        var (x, y) => new Add(x, y)
    },
    Mul(var l, var r) => (Simplify(l), Simplify(r)) switch
    {
        (Num(0), _) or (_, Num(0)) => new Num(0),            // x * 0 = 0
        (Num(1), var x) => x,                               // 1 * x = x
        (var x, Num(1)) => x,                               // x * 1 = x
        var (x, y) => new Mul(x, y)
    },
    _ => e
};
static string Show(Expr e) => e switch
{
    Num(var v) => v.ToString(),
    Add(var l, var r) => $"({Show(l)} + {Show(r)})",
    Mul(var l, var r) => $"{Show(l)} * {Show(r)}",
    _ => "?"
};
public abstract record Expr;
public sealed record Num(double Value) : Expr;
public sealed record Add(Expr Left, Expr Right) : Expr;
public sealed record Mul(Expr Left, Expr Right) : Expr;
```

**Приклад запуску:**
```
'' → порожня команда
'help' → команди: add, del, move
'add milk' → додано milk
'add eggs 12' → додано eggs × 12
'add eggs -1' → некоректна кількість
'del a b c' → видалено: a, b, c
'move box to kitchen' → box переміщено в kitchen
'jump' → невідома команда 'jump'
((2 + 3) * 4 + 0) = 20
Спрощено: (2 + 3) * 4 = 20
0
```

### 4.9. Порядок гілок, `when` і продуктивність
- **Гілки перевіряються зверху вниз**, перша збіжна перемагає. Тому конкретніші патерни — вище, загальніші — нижче.
- Якщо гілку повністю «накриває» попередня (subsumption) — **помилка** CS8510 у switch-виразі або CS8120 у switch-операторі:

```csharp
// НЕ КОМПІЛЮЄТЬСЯ: CS8510 The pattern is unreachable. It has already been handled by a previous arm
Console.WriteLine(Grade(95));
static string Grade(int score) => score switch
{
    >= 60 => "C",
    >= 90 => "A",
    _ => "F"
};
```

```csharp
// НЕ КОМПІЛЮЄТЬСЯ: CS8120 The switch case is unreachable. It has already been handled by a previous case
object o = 5;
switch (o)
{
    case int:
        Console.WriteLine("int");
        break;
    case int n when n > 0:
        Console.WriteLine("додатне");
        break;
}
```

- **`when` vs патерни.** Патерни компілятор «розуміє»: перевіряє повноту й недосяжність. Умова `when` для нього — чорна скринька. Нижче логічно покрито все, але компілятор видає попередження CS8846 (приклад компілюється):

```csharp
// Компілюється з попередженням CS8846: компілятор не аналізує умови when
Console.WriteLine(Sign(-3));
static string Sign(int x) => x switch
{
    int n when n >= 0 => "невід'ємне",
    int n when n < 0 => "від'ємне"
};
```

Краще: `>= 0 => "невід'ємне", _ => "від'ємне"`. Правило: якщо умову можна виразити патерном (`> 0`, `{ Length: 0 }`, `null`) — пишіть патерн; `when` — для зв'язків між змінними (`a == b`) і викликів методів.
- **Продуктивність.** Компілятор перетворює весь `switch` на **дерево рішень** (decision tree): кожна перевірка (тип, довжина, властивість) виконується максимум один раз, а щільні цілі константи — у таблицю переходів (`switch` IL). Рядкові константи порівнюються через обчислений хеш або довжину + символ. Тому великий `switch` зазвичай не повільніший за ручний ланцюжок `if`, а часто швидший.
- **Pattern matching чи поліморфізм?**

| Обирайте `switch` з патернами | Обирайте `virtual`/`abstract` методи |
|-------------------------------|--------------------------------------|
| Набір типів **закритий** і стабільний (дерево виразів, повідомлення протоколу) | Нові типи з'являються часто (плагіни, фігури від користувачів) |
| Операцій **багато** і вони додаються (Eval, Show, Simplify) | Операцій мало, а поведінка — частина типу |
| Типи — «дані» (`record`) без поведінки | Об'єкти з інкапсульованим станом |
| Рішення залежить від **кількох** об'єктів одразу (`(a, b) switch`) | Рішення залежить від одного об'єкта |

Додаючи новий тип до ієрархії з патернами, доведеться оновити **всі** `switch`; додаючи нову операцію до поліморфної ієрархії — **всі** класи. Для `IShape` з §7 поліморфізм природніший, для `Expr` з §4.8 — патерни.

### 4.10. Типові помилки
- **Неправильний порядок діапазонів:** `>= 60` перед `>= 90` — гілка `A` недосяжна (CS8510). Від вужчого до ширшого.
- **`_` або `var x` не останньою гілкою** — усе нижче стає недосяжним.
- **Змінна в `or`/`not`:** `(Num(1), var x) or (var x, Num(1))` — CS8780; розбийте на дві гілки.
- **Реляційний патерн з неконстантою:** `> limit` не компілюється — порівняння зі змінною лише через `when x > limit`.
- **`x == null` замість `x is null`:** `==` може бути перевантажений; `is null` — завжди перевірка посилання.
- **Ігнорування CS8509:** неповний switch-вираз компілюється, але падає `SwitchExpressionException` на першому непередбаченому значенні. Додавайте `_ => throw ...` явно.
- **`..` двічі в list-патерні** (`[.., 1, ..]`) — не дозволено; для пошуку всередині використовуйте `Contains`.
- **Очікування fall-through як у C++** — кожна непорожня секція `case` завершується `break`/`return`/`goto case`.

### 4.11. Міні-вправи
**Вправа 1.** Напишіть `IsLeap(int year)` одним switch-виразом по кортежу `(year % 4, year % 100, year % 400)`. Перевірте на 2024, 2025, 1900, 2000.

<details>
<summary>Розв'язок</summary>

```csharp
foreach (int year in new[] { 2024, 2025, 1900, 2000 })
{
    Console.WriteLine($"{year}: {(IsLeap(year) ? "високосний" : "звичайний")}");
}
static bool IsLeap(int year) => (year % 4, year % 100, year % 400) switch
{
    (_, _, 0) => true,                                      // ділиться на 400
    (_, 0, _) => false,                                     // ділиться на 100, але не на 400
    (0, _, _) => true,                                      // ділиться на 4
    _ => false
};
```

**Приклад запуску:**
```
2024: високосний
2025: звичайний
1900: звичайний
2000: високосний
```

</details>

**Вправа 2.** Класифікуйте трикутник за сторонами `(a, b, c)`: «некоректні сторони» (≤ 0), «не існує» (порушено нерівність трикутника), «рівносторонній», «рівнобедрений», «різносторонній». Зверніть увагу на порядок гілок.

<details>
<summary>Розв'язок</summary>

```csharp
(double, double, double)[] triangles = [(3, 3, 3), (5, 5, 8), (3, 4, 5), (1, 2, 10), (0, 4, 4)];
foreach (var (a, b, c) in triangles)
{
    Console.WriteLine($"({a}, {b}, {c}) → {Classify(a, b, c)}");
}
static string Classify(double a, double b, double c) => (a, b, c) switch
{
    _ when a <= 0 || b <= 0 || c <= 0 => "некоректні сторони",
    _ when a + b <= c || a + c <= b || b + c <= a => "не існує",
    var (x, y, z) when x == y && y == z => "рівносторонній",
    var (x, y, z) when x == y || y == z || x == z => "рівнобедрений",
    _ => "різносторонній"
};
```

**Приклад запуску:**
```
(3, 3, 3) → рівносторонній
(5, 5, 8) → рівнобедрений
(3, 4, 5) → різносторонній
(1, 2, 10) → не існує
(0, 4, 4) → некоректні сторони
```

</details>

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

## 11. Робота з файлами
Простір імен `System.IO` (підключений неявно): `Path` — робота зі шляхами, `File`/`Directory` — операції над файлами й папками, `Stream*` — потокове читання. Кожен приклад створює власні файли в `Path.GetTempPath()`, тому запускається окремо.

### 11.1. Шляхи: `Path` і поточна папка
Шляхи **ніколи не склеюють вручну** через `+ "/"` — роздільник різний у Windows (`\`) і macOS/Linux (`/`). Відносний шлях рахується від **поточної робочої папки** процесу: для `dotnet run` це папка проєкту, а при запуску `.exe` з іншого місця — зовсім інша папка. Для файлів, що постачаються разом із програмою, використовуйте `AppContext.BaseDirectory`.

```csharp
string dataDir = Path.Combine(Path.GetTempPath(), "lec1-demo", "data"); // Combine сам ставить / (macOS, Linux) або \ (Windows)
string csvPath = Path.Combine(dataDir, "students.csv");
Console.WriteLine($"Ім'я файлу: {Path.GetFileName(csvPath)}");                    // students.csv
Console.WriteLine($"Без розширення: {Path.GetFileNameWithoutExtension(csvPath)}"); // students
Console.WriteLine($"Розширення: {Path.GetExtension(csvPath)}");                    // .csv — разом із крапкою
Console.WriteLine($"Папка: {Path.GetFileName(Path.GetDirectoryName(csvPath))}");   // data
Console.WriteLine($"Нове розширення: {Path.GetFileName(Path.ChangeExtension(csvPath, ".json"))}");
Console.WriteLine($"Абсолютний? {Path.IsPathRooted(csvPath)} / {Path.IsPathRooted("data/input.txt")}");
// AppContext.BaseDirectory — папка зі зібраною програмою (…/bin/Debug/net10.0/), не залежить від місця запуску
// Environment.CurrentDirectory — робоча папка процесу; для `dotnet run` це папка проєкту (де .csproj)
Console.WriteLine($"BaseDirectory всередині bin: {AppContext.BaseDirectory.Contains("bin")}");
string relative = Path.GetFullPath("data/input.txt");                             // відносні шляхи резолвляться від CurrentDirectory
Console.WriteLine($"Відносний шлях — від поточної папки: {relative.StartsWith(Environment.CurrentDirectory)}");
// Надійно для файлів, які копіюються разом із програмою (<CopyToOutputDirectory> у .csproj):
string shipped = Path.Combine(AppContext.BaseDirectory, "data", "input.txt");
Console.WriteLine($"Файл поруч із .dll: {Path.GetFileName(shipped)}");
```

**Приклад запуску:**
```
Ім'я файлу: students.csv
Без розширення: students
Розширення: .csv
Папка: data
Нове розширення: students.json
Абсолютний? True / False
BaseDirectory всередині bin: True
Відносний шлях — від поточної папки: True
Файл поруч із .dll: input.txt
```

### 11.2. Файли й папки: існування, створення, видалення, перелік
`File`/`Directory` — статичні методи для операцій зі шляхами; `FileInfo`/`DirectoryInfo` — об'єкти з метаданими (розмір, дата).

```csharp
string root = Path.Combine(Path.GetTempPath(), "lec1-demo", "dirs");
if (Directory.Exists(root)) Directory.Delete(root, recursive: true); // recursive: true — разом із вмістом
Directory.CreateDirectory(Path.Combine(root, "sub"));  // створює всі проміжні папки; якщо вже є — не помилка
File.WriteAllText(Path.Combine(root, "a.txt"), "A");
File.WriteAllText(Path.Combine(root, "b.csv"), "B");
File.WriteAllText(Path.Combine(root, "sub", "c.txt"), "C");
Console.WriteLine($"a.txt існує: {File.Exists(Path.Combine(root, "a.txt"))}, x.txt існує: {File.Exists(Path.Combine(root, "x.txt"))}");
Console.WriteLine($"sub — папка: {Directory.Exists(Path.Combine(root, "sub"))}, sub — файл: {File.Exists(Path.Combine(root, "sub"))}");
// EnumerateFiles повертає шляхи ліниво (по одному), GetFiles — одразу весь масив; порядок не гарантовано → Order()
foreach (string f in Directory.EnumerateFiles(root, "*.txt").Order())                          // лише верхній рівень
    Console.WriteLine($"  верхній рівень: {Path.GetFileName(f)}");
foreach (string f in Directory.EnumerateFiles(root, "*.txt", SearchOption.AllDirectories).Order()) // з підпапками
    Console.WriteLine($"  рекурсивно: {Path.GetRelativePath(root, f)}");
File.Copy(Path.Combine(root, "a.txt"), Path.Combine(root, "a-copy.txt"), overwrite: true);
File.Move(Path.Combine(root, "b.csv"), Path.Combine(root, "sub", "b.csv"));   // переміщення = перейменування
File.Delete(Path.Combine(root, "a.txt"));                                     // неіснуючий файл — не помилка
var info = new FileInfo(Path.Combine(root, "a-copy.txt"));                    // FileInfo — метадані файлу
Console.WriteLine($"{info.Name}: {info.Length} байт");
Console.WriteLine($"Файлів усього: {Directory.EnumerateFiles(root, "*", SearchOption.AllDirectories).Count()}");
```

**Приклад запуску:**
```
a.txt існує: True, x.txt існує: False
sub — папка: True, sub — файл: False
  верхній рівень: a.txt
  рекурсивно: a.txt
  рекурсивно: sub/c.txt
a-copy.txt: 1 байт
Файлів усього: 3
```

### 11.3. Увесь файл одразу: `ReadAllText`, `WriteAllLines`, кодування
Найпростіший спосіб для **невеликих** файлів. За замовчуванням .NET пише в UTF-8 (без BOM); `Encoding.UTF8` явно додає BOM — так Блокнот у Windows точно розпізнає кирилицю.

```csharp
using System.Text;
string dir = Path.Combine(Path.GetTempPath(), "lec1-demo");
Directory.CreateDirectory(dir);
string path = Path.Combine(dir, "notes.txt");
File.WriteAllText(path, "Привіт, файли!\n", Encoding.UTF8);          // створює або ПЕРЕЗАПИСУЄ файл
File.AppendAllText(path, "Другий рядок\n", Encoding.UTF8);           // дописує в кінець (створює, якщо немає)
string text = File.ReadAllText(path, Encoding.UTF8);                 // весь файл — один string
Console.Write(text);
Console.WriteLine($"Символів: {text.Length}, байт на диску: {new FileInfo(path).Length}"); // кирилиця в UTF-8 — 2 байти, + 3 байти BOM
string[] lines = ["Олена;95", "Андрій;87", "Ірина;78"];
string listPath = Path.Combine(dir, "scores.txt");
File.WriteAllLines(listPath, lines);                                 // кожен елемент — окремий рядок; UTF-8 за замовчуванням
string[] back = File.ReadAllLines(listPath);                         // масив рядків без символів \n
Console.WriteLine($"Рядків: {back.Length}, перший: {back[0]}");
// Помилка кодування: ASCII не має кирилиці — кожен символ замінюється на '?'
File.WriteAllText(Path.Combine(dir, "ascii.txt"), "Привіт", Encoding.ASCII);
Console.WriteLine($"ASCII: {File.ReadAllText(Path.Combine(dir, "ascii.txt"))}");
```

**Приклад запуску:**
```
Привіт, файли!
Другий рядок
Символів: 28, байт на диску: 53
Рядків: 3, перший: Олена;95
ASCII: ??????
```

### 11.4. Великі файли рядок за рядком: `ReadLines`, `StreamReader`, `StreamWriter`
`File.ReadAllLines` завантажує **весь** файл у пам'ять (гігабайтний лог — гігабайти RAM), а `File.ReadLines` читає **ліниво**. Потоки (`Stream*`) тримають відкритий дескриптор файлу та буфер — їх **обов'язково** звільняють через `using`: інакше файл лишається заблокованим, а останні записані дані можуть не потрапити на диск.

```csharp
string path = Path.Combine(Path.GetTempPath(), "lec1-demo", "big.log");
Directory.CreateDirectory(Path.GetDirectoryName(path)!);
// StreamWriter: пише в буфер у пам'яті і скидає на диск частинами
using (var writer = new StreamWriter(path))                   // using-блок: Dispose() викличеться навіть при винятку
{
    for (int i = 1; i <= 100_000; i++)
        writer.WriteLine(i % 1000 == 0 ? $"{i};ERROR;disk full" : $"{i};INFO;ok");
}                                                             // тут буфер скинуто, файл закрито — інакше кінець файлу може загубитись
// File.ReadLines — ЛІНИВО: у пам'яті одночасно лише поточний рядок; ReadAllLines завантажив би всі 100 000
int errors = File.ReadLines(path).Count(line => line.Contains(";ERROR;"));
string firstError = File.ReadLines(path).First(line => line.Contains("ERROR")); // First зупиняє читання на 1000-му рядку
Console.WriteLine($"Помилок: {errors}, перша: {firstError}");
// StreamReader вручну: ReadLine() повертає null в кінці файлу
using var reader = new StreamReader(path);                    // using var: Dispose() наприкінці області видимості (методу)
int count = 0;
string? line;
while ((line = reader.ReadLine()) is not null)
{
    count++;
}
Console.WriteLine($"Прочитано рядків: {count}");
```

**Приклад запуску:**
```
Помилок: 100, перша: 1000;ERROR;disk full
Прочитано рядків: 100000
```

### 11.5. CSV: розбір і запис
CSV — текст, де колонки розділені комою. Правила надійного розбору: пропустити заголовок, перевірити кількість колонок, парсити числа через `TryParse` з `CultureInfo.InvariantCulture` і **пропускати** погані рядки, а не падати. `Split(',')` не впорається з полями в лапках (`"Шевченко, Тарас"`) — у реальних проєктах беріть бібліотеку **CsvHelper**.

```csharp
using System.Globalization;
string dir = Path.Combine(Path.GetTempPath(), "lec1-demo");
Directory.CreateDirectory(dir);
string input = Path.Combine(dir, "grades.csv");
File.WriteAllLines(input,
[
    "Name,Age,Score",          // заголовок
    "Olena,19,95.5",
    "Andrii,twenty,87.0",      // поганий вік
    "Iryna,20,78.25",
    "",                        // порожній рядок
    "Taras,21"                 // бракує колонки
]);
var students = new List<Student>();
int lineNo = 1;
foreach (string line in File.ReadLines(input).Skip(1))             // Skip(1) — пропускаємо заголовок
{
    lineNo++;
    if (string.IsNullOrWhiteSpace(line)) continue;
    string[] cols = line.Split(',');
    if (cols.Length != 3)
    {
        Console.WriteLine($"Рядок {lineNo}: очікували 3 колонки, отримали {cols.Length}");
        continue;
    }
    // TryParse замість Parse: поганий рядок не "валить" програму; InvariantCulture — завжди крапка як роздільник
    if (!int.TryParse(cols[1], NumberStyles.Integer, CultureInfo.InvariantCulture, out int age) ||
        !double.TryParse(cols[2], NumberStyles.Float, CultureInfo.InvariantCulture, out double score))
    {
        Console.WriteLine($"Рядок {lineNo}: некоректні числа — пропускаємо");
        continue;
    }
    students.Add(new Student(cols[0].Trim(), age, score));
}
Console.WriteLine($"Завантажено: {students.Count}, середній бал: {students.Average(s => s.Score).ToString("F2", CultureInfo.InvariantCulture)}");
// Запис CSV: число форматуємо з InvariantCulture, інакше в uk-UA вийде "95,5" і зламає колонки
string output = Path.Combine(dir, "top.csv");
IEnumerable<string> rows = students
    .Where(s => s.Score >= 80)
    .Select(s => string.Join(",", s.Name, s.Age, s.Score.ToString(CultureInfo.InvariantCulture)));
File.WriteAllLines(output, ["Name,Age,Score", .. rows]);
Console.Write(File.ReadAllText(output));
// Чому культура важлива: у uk-UA десятковий роздільник — кома
var uk = new CultureInfo("uk-UA");
Console.WriteLine($"uk-UA: {95.5.ToString(uk)}; \"95.5\" як uk-UA розпізнано: {double.TryParse("95.5", NumberStyles.Float, uk, out _)}");
public record Student(string Name, int Age, double Score);
```

**Приклад запуску:**
```
Рядок 3: некоректні числа — пропускаємо
Рядок 6: очікували 3 колонки, отримали 2
Завантажено: 2, середній бал: 86.88
Name,Age,Score
Olena,19,95.5
uk-UA: 95,5; "95.5" як uk-UA розпізнано: False
```

### 11.6. JSON: `System.Text.Json`
`JsonSerializer.Serialize` перетворює об'єкт (зручно — `record`) у JSON, `Deserialize<T>` — назад. Вбудовано в .NET, додаткові пакети не потрібні.

```csharp
using System.Text.Encodings.Web;
using System.Text.Json;
using System.Text.Unicode;
string path = Path.Combine(Path.GetTempPath(), "lec1-demo", "course.json");
Directory.CreateDirectory(Path.GetDirectoryName(path)!);
var course = new Course("Алгоритми", 2025, [new Lecture(1, "Нагадування C#"), new Lecture(2, "Складність")]);
var options = new JsonSerializerOptions
{
    WriteIndented = true,                                     // гарне форматування з відступами
    PropertyNamingPolicy = JsonNamingPolicy.CamelCase,        // Title → "title"
    Encoder = JavaScriptEncoder.Create(UnicodeRanges.All)     // кирилиця як є, а не А...
};
string json = JsonSerializer.Serialize(course, options);      // об'єкт → рядок JSON
File.WriteAllText(path, json);
Console.WriteLine(File.ReadAllText(path));
Course? loaded = JsonSerializer.Deserialize<Course>(File.ReadAllText(path), options); // JSON → об'єкт; null для "null"
Console.WriteLine($"Лекцій: {loaded?.Lectures.Count}, друга: {loaded?.Lectures[1].Topic}");
// record порівнює за значенням, але List<T> — за посиланням, тому порівнюємо вміст
Console.WriteLine($"Round-trip без втрат: {loaded!.Title == course.Title && loaded.Lectures.SequenceEqual(course.Lectures)}");
public record Lecture(int Number, string Topic);
public record Course(string Title, int Year, List<Lecture> Lectures);
```

**Приклад запуску:**
```
{
  "title": "Алгоритми",
  "year": 2025,
  "lectures": [
    {
      "number": 1,
      "topic": "Нагадування C#"
    },
    {
      "number": 2,
      "topic": "Складність"
    }
  ]
}
Лекцій: 2, друга: Складність
Round-trip без втрат: True
```

### 11.7. Бінарні файли: `ReadAllBytes`, `BinaryWriter`/`BinaryReader`
Бінарний формат компактний і швидкий, але не читається людиною: читати треба **в тому самому порядку й тих самих типах**, у яких писали.

```csharp
string dir = Path.Combine(Path.GetTempPath(), "lec1-demo");
Directory.CreateDirectory(dir);
string rawPath = Path.Combine(dir, "raw.bin");
byte[] bytes = [0xCA, 0xFE, 0xBA, 0xBE];
File.WriteAllBytes(rawPath, bytes);                             // масив байтів як є, без кодування
Console.WriteLine($"Байти: {Convert.ToHexString(File.ReadAllBytes(rawPath))}");
string binPath = Path.Combine(dir, "points.bin");
using (var writer = new BinaryWriter(File.Create(binPath)))    // BinaryWriter закриє і FileStream
{
    writer.Write(3);                                            // int — 4 байти: кількість точок
    for (int i = 0; i < 3; i++)
    {
        writer.Write(i * 1.5);                                  // double — 8 байт
        writer.Write($"P{i}");                                  // string — довжина + UTF-8 байти
    }
}
Console.WriteLine($"Розмір файлу: {new FileInfo(binPath).Length} байт");
using (var reader = new BinaryReader(File.OpenRead(binPath)))
{
    int n = reader.ReadInt32();                                 // читати СТРОГО в тому ж порядку і тих же типах
    for (int i = 0; i < n; i++)
            {
        double x = reader.ReadDouble();                         // спершу double, потім string — як писали
        string name = reader.ReadString();
        Console.WriteLine($"{name} = {x}");
    }
}
```

**Приклад запуску:**
```
Байти: CAFEBABE
Розмір файлу: 37 байт
P0 = 0
P1 = 1.5
P2 = 3
```

### 11.8. Асинхронна робота з файлами
Методи з суфіксом `Async` не блокують потік під час дискових операцій — важливо для веб-серверів і UI. Під капотом ті самі операції, просто з `await`.

```csharp
string dir = Path.Combine(Path.GetTempPath(), "lec1-demo");
Directory.CreateDirectory(dir);
string path = Path.Combine(dir, "async.txt");
await File.WriteAllLinesAsync(path, ["перший", "другий", "третій"]); // потік не блокується, поки ОС пише на диск
string text = await File.ReadAllTextAsync(path);
Console.WriteLine($"Рядків: {text.Split('\n', StringSplitOptions.RemoveEmptyEntries).Length}");
// Кілька файлів одночасно
string[] names = ["x.txt", "y.txt", "z.txt"];
await Task.WhenAll(names.Select(n => File.WriteAllTextAsync(Path.Combine(dir, n), n.ToUpper())));
string[] contents = await Task.WhenAll(names.Select(n => File.ReadAllTextAsync(Path.Combine(dir, n))));
Console.WriteLine(string.Join(" ", contents));
// Асинхронне читання рядок за рядком великого файлу
int total = 0;
await foreach (string line in File.ReadLinesAsync(path))             // IAsyncEnumerable<string> (.NET 7+)
    total += line.Length;
Console.WriteLine($"Сума довжин: {total}");
```

**Приклад запуску:**
```
Рядків: 3
X.TXT Y.TXT Z.TXT
Сума довжин: 18
```

### 11.9. Обробка помилок
Файл можуть видалити, заблокувати чи заборонити до нього доступ **між** перевіркою `File.Exists` і читанням — тому надійний код ловить винятки. `FileNotFoundException` і `DirectoryNotFoundException` успадковуються від `IOException`, отже ловимо їх **раніше**.

```csharp
string dir = Path.Combine(Path.GetTempPath(), "lec1-demo");
Directory.CreateDirectory(dir);
Console.WriteLine(TryRead(Path.Combine(dir, "missing.txt")));
Console.WriteLine(TryRead(Path.Combine(dir, "no-such-dir", "file.txt")));
Console.WriteLine(TryRead(dir));                                  // папка замість файлу
string locked = Path.Combine(dir, "locked.txt");
using (var stream = new FileStream(locked, FileMode.Create, FileAccess.Write, FileShare.None)) // ексклюзивний доступ
{
    Console.WriteLine(TryRead(locked));                           // файл зайнятий іншим потоком
}
File.WriteAllText(locked, "тепер доступний");
Console.WriteLine(TryRead(locked));

static string TryRead(string path)
{
    try
    {
        return $"OK: {File.ReadAllText(path)}";
    }
    catch (FileNotFoundException e)                               // конкретніші типи — першими
    {
        return $"Файл не знайдено: {Path.GetFileName(e.FileName)}";
    }
    catch (DirectoryNotFoundException)
    {
        return "Папку не знайдено";
    }
    catch (UnauthorizedAccessException)                           // немає прав або шлях — це папка
    {
        return "Доступ заборонено (немає прав або це папка)";
    }
    catch (IOException e)                                         // базовий для File/DirectoryNotFound — тому останній
    {
        return $"Помилка вводу-виводу: {e.GetType().Name}";
    }
}
```

**Приклад запуску:**
```
Файл не знайдено: missing.txt
Папку не знайдено
Доступ заборонено (немає прав або це папка)
Помилка вводу-виводу: IOException
OK: тепер доступний
```

### 11.10. Типові помилки
- **Забутий `Dispose`:** `new StreamWriter(path)` без `using` — дані лишаються в буфері й не потрапляють у файл, а сам файл заблокований для інших процесів. Завжди `using` / `using var`.
- **Десятковий роздільник залежить від культури:** `double.Parse("95.5")` на комп'ютері з українською локаллю кидає `FormatException` (там роздільник — кома), а `ToString()` пише `95,5` і ламає CSV. Для файлів — завжди `CultureInfo.InvariantCulture`.
- **Кодування:** запис кирилиці в `Encoding.ASCII` дає `??????`; читання файлу з Windows-1251 як UTF-8 — «кракозябри». Явно вказуйте `Encoding.UTF8` і знайте, у якому кодуванні прийшли дані.
- **Завантаження величезних файлів у пам'ять:** `ReadAllText`/`ReadAllLines` на файлі в кілька ГБ — `OutOfMemoryException` або повільна робота. Для великих файлів — `File.ReadLines` чи `StreamReader`.
- **Відносні шляхи:** `"data/input.txt"` працює з `dotnet run`, але «зникає» при запуску з іншої папки — використовуйте `AppContext.BaseDirectory` або шлях з аргументів.
- **Ручне склеювання шляхів** (`dir + "\\" + name`) — ламається на іншій ОС; використовуйте `Path.Combine`.

---

## 12. Власні колекції: мінімальні реалізації
Щоб розуміти, що відбувається «під капотом» стандартних колекцій, корисно один раз написати їх самому. Нижче — навмисно мінімальні версії (без видалення, перевірок версії ітератора тощо). **Детально структури даних розбираються в Лекції 2**, а в робочому коді використовуйте готові `List<T>`, `LinkedList<T>`, `Queue<T>` і `System.Threading.Channels` — вони протестовані, оптимізовані та потокобезпечні там, де це заявлено.

### 12.1. `MyList<T>`: динамічний масив
Масив, який подвоюється при заповненні; `yield return` робить колекцію придатною для `foreach` і LINQ.

```csharp
using System.Collections;
var list = new MyList<int>();
for (int i = 1; i <= 5; i++) list.Add(i * 10);        // ємність росте: 2 → 4 → 8
list[0] = 7;                                          // індексатор із перевіркою меж
Console.WriteLine($"Count={list.Count}, Capacity={list.Capacity}");
Console.WriteLine(string.Join(" ", list));            // foreach/string.Join працюють через IEnumerable<T>
Console.WriteLine($"Сума через LINQ: {list.Sum()}");
public sealed class MyList<T> : IEnumerable<T>
{
    private T[] _items = new T[2];                    // внутрішній масив фіксованого розміру
    public int Count { get; private set; }
    public int Capacity => _items.Length;
    public void Add(T item)
    {
        if (Count == _items.Length)
            Array.Resize(ref _items, _items.Length * 2); // подвоєння → Add за амортизоване O(1)
        _items[Count++] = item;
    }
    public T this[int index]                          // індексатор: list[i]
    {
        get => (uint)index < (uint)Count ? _items[index] : throw new ArgumentOutOfRangeException(nameof(index));
        set => _items[(uint)index < (uint)Count ? index : throw new ArgumentOutOfRangeException(nameof(index))] = value;
    }
    public IEnumerator<T> GetEnumerator()
    {
        for (int i = 0; i < Count; i++)
            yield return _items[i];                   // yield: компілятор сам генерує клас-ітератор
    }
    IEnumerator IEnumerable.GetEnumerator() => GetEnumerator(); // негенерична версія для сумісності
}
```

**Приклад запуску:**
```
Count=5, Capacity=8
7 20 30 40 50
Сума через LINQ: 147
```

### 12.2. `MyLinkedList<T>`: однозв'язний список
Кожен вузол зберігає значення й посилання на наступний; вставка на початок і в кінець (з хвостом) — O(1), доступ за індексом — O(n).

```csharp
using System.Collections;
var names = new MyLinkedList<string>();
names.AddLast("Olena");
names.AddLast("Andrii");
names.AddFirst("Iryna");                              // O(1) — лише перепризначення Head
foreach (string name in names) Console.Write($"{name} -> ");
Console.WriteLine($"null (Count={names.Count})");
public sealed class MyLinkedList<T> : IEnumerable<T>
{
    private sealed class Node(T value)                // вузол: значення + посилання на наступний
    {
        public T Value { get; } = value;
        public Node? Next { get; set; }
    }
    private Node? _head;
    private Node? _tail;                              // хвіст → AddLast за O(1), а не O(n)
    public int Count { get; private set; }
    public void AddFirst(T value)
    {
        var node = new Node(value) { Next = _head };
        _head = node;
        _tail ??= node;                               // перший елемент — і голова, і хвіст
        Count++;
    }
    public void AddLast(T value)
    {
        var node = new Node(value);
        if (_tail is null) _head = node;
        else _tail.Next = node;
        _tail = node;
        Count++;
    }
    public IEnumerator<T> GetEnumerator()
    {
        for (Node? current = _head; current is not null; current = current.Next)
            yield return current.Value;
    }
    IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();
}
```

**Приклад запуску:**
```
Iryna -> Olena -> Andrii -> null (Count=3)
```

### 12.3. `MyQueue<T>`: черга на кільцевому буфері
FIFO-черга фіксованої ємності: індекси голови й хвоста рухаються по колу, тому `Enqueue`/`Dequeue` — O(1) без зсуву елементів.

```csharp
var queue = new MyQueue<string>(capacity: 3);
queue.Enqueue("A"); queue.Enqueue("B"); queue.Enqueue("C");
Console.WriteLine($"Dequeue: {queue.Dequeue()}");     // FIFO: першим прийшов — першим вийшов
queue.Enqueue("D");                                   // голова зсунулась — D стає в звільнену клітинку 0
while (queue.Count > 0) Console.Write($"{queue.Dequeue()} ");
Console.WriteLine();
try { queue.Dequeue(); }
catch (InvalidOperationException e) { Console.WriteLine(e.Message); }
public sealed class MyQueue<T>(int capacity)
{
    private readonly T[] _buffer = new T[capacity];   // кільцевий буфер: індекси "загортаються" по модулю
    private int _head;                                // звідки забирати
    private int _tail;                                // куди класти
    public int Count { get; private set; }
    public void Enqueue(T item)
    {
        if (Count == _buffer.Length) throw new InvalidOperationException("Черга переповнена");
        _buffer[_tail] = item;
        _tail = (_tail + 1) % _buffer.Length;         // після останньої клітинки — знову 0
        Count++;
    }
    public T Dequeue()
    {
        if (Count == 0) throw new InvalidOperationException("Черга порожня");
        T item = _buffer[_head];
        _buffer[_head] = default!;                    // не тримаємо посилання — GC зможе прибрати об'єкт
        _head = (_head + 1) % _buffer.Length;
        Count--;
        return item;
    }
}
```

**Приклад запуску:**
```
Dequeue: A
B C D 
Черга порожня
```

### 12.4. `MyChannel<T>`: асинхронна черга виробник/споживач
Канал — потокобезпечна черга, з якої споживач асинхронно **чекає** на дані. `lock` захищає `Queue<T>`, а `SemaphoreSlim` рахує доступні елементи.

```csharp
var channel = new MyChannel<int>();
Task producer = Task.Run(async () =>
{
    for (int i = 1; i <= 5; i++)
    {
        await channel.WriteAsync(i * i);              // виробник кладе дані
        await Task.Delay(10);                         // імітація роботи
    }
});
int sum = 0;
for (int i = 0; i < 5; i++)
{
    int value = await channel.ReadAsync();           // споживач чекає, поки з'являться дані, не блокуючи потік
    Console.WriteLine($"Отримано: {value}");
    sum += value;
}
await producer;
Console.WriteLine($"Сума: {sum}");
public sealed class MyChannel<T>
{
    private readonly Queue<T> _queue = new();
    private readonly object _sync = new();            // Queue<T> не потокобезпечна → доступ лише під lock
    private readonly SemaphoreSlim _available = new(0); // лічильник доступних елементів
    public Task WriteAsync(T item)
    {
        lock (_sync) _queue.Enqueue(item);
        _available.Release();                         // +1: будимо одного споживача
        return Task.CompletedTask;
    }
    public async Task<T> ReadAsync(CancellationToken token = default)
    {
        await _available.WaitAsync(token);            // -1: асинхронно чекаємо, якщо елементів 0
        lock (_sync) return _queue.Dequeue();         // await усередині lock заборонено — тому окремо
    }
}
```

**Приклад запуску:**
```
Отримано: 1
Отримано: 4
Отримано: 9
Отримано: 16
Отримано: 25
Сума: 55
```

### 12.5. Як використовувати `System.Threading.Channels`
**Канал** — потокобезпечна асинхронна «труба» між виробниками (`ChannelWriter<T>`) і споживачами (`ChannelReader<T>`); готова промислова заміна `MyChannel<T>`.

| Що | API | Поведінка |
|----|-----|-----------|
| Створення | `Channel.CreateUnbounded<T>()` | без обмеження — запис завжди миттєвий, пам'ять може рости |
| | `Channel.CreateBounded<T>(new BoundedChannelOptions(capacity) { FullMode, SingleReader, SingleWriter })` | обмежена ємність; `SingleReader`/`SingleWriter` — підказки для оптимізації |
| `FullMode` | `Wait` / `DropOldest` / `DropNewest` / `DropWrite` | чекати місця / викинути найстаріший / найновіший у каналі / новий, що записується |
| Writer | `WriteAsync`, `TryWrite`, `Complete()` | запис (чекає при `Wait`), спроба без очікування, «даних більше не буде» |
| Reader | `ReadAsync`, `TryRead`, `WaitToReadAsync`, `ReadAllAsync()`, `Completion` | читання; `await foreach` над `ReadAllAsync()` завершується після `Complete()` |

**Приклад 1 — простий виробник і споживач:**

```csharp
using System.Threading.Channels;
Channel<string> channel = Channel.CreateUnbounded<string>(); // без обмеження розміру
Task producer = Task.Run(async () =>
{
    foreach (string job in new[] { "парсинг", "обчислення", "звіт" })
    {
        await channel.Writer.WriteAsync(job);                // для unbounded завершується одразу
        await Task.Delay(10);
    }
    channel.Writer.Complete();                               // сигнал "більше даних не буде" — ОБОВ'ЯЗКОВО
});
await foreach (string job in channel.Reader.ReadAllAsync()) // завершується після Complete() і вичитки всього
    Console.WriteLine($"Обробляю: {job}");
await producer;
await channel.Reader.Completion;                             // Task, що завершується, коли канал закрито й порожній
Console.WriteLine("Канал закрито");
```

**Приклад запуску:**
```
Обробляю: парсинг
Обробляю: обчислення
Обробляю: звіт
Канал закрито
```

**Приклад 2 — кілька виробників, один споживач:** `Complete()` викликаємо лише після `Task.WhenAll` усіх виробників.

```csharp
using System.Threading.Channels;
var channel = Channel.CreateUnbounded<string>(new UnboundedChannelOptions { SingleReader = true }); // підказка для оптимізації
Task[] producers = Enumerable.Range(1, 3).Select(id => Task.Run(async () =>
{
    for (int i = 1; i <= 2; i++)
    {
        await channel.Writer.WriteAsync($"P{id}-{i}");       // кілька потоків пишуть одночасно — це безпечно
        await Task.Delay(5);
    }
})).ToArray();
Task consumer = Task.Run(async () =>
{
    var received = new List<string>();
    await foreach (string item in channel.Reader.ReadAllAsync())
        received.Add(item);
    received.Sort();                                         // порядок між виробниками не гарантовано → сортуємо
    Console.WriteLine($"Отримано {received.Count}: {string.Join(" ", received)}");
});
await Task.WhenAll(producers);                               // чекаємо ВСІХ виробників...
channel.Writer.Complete();                                   // ...і лише тоді закриваємо канал
await consumer;
```

**Приклад запуску:**
```
Отримано 6: P1-1 P1-2 P2-1 P2-2 P3-1 P3-2
```

**Приклад 3 — конвеєр із трьох етапів:** кожен етап читає з одного каналу і пише в наступний.

```csharp
using System.Threading.Channels;
// Конвеєр: генерація → піднесення до квадрату → вивід; етапи працюють паралельно
Channel<int> numbers = Channel.CreateBounded<int>(2);
Channel<(int N, int Square)> squares = Channel.CreateBounded<(int, int)>(2);
Task generate = Task.Run(async () =>
{
    for (int i = 1; i <= 5; i++) await numbers.Writer.WriteAsync(i);
    numbers.Writer.Complete();
});
Task square = Task.Run(async () =>
{
    await foreach (int n in numbers.Reader.ReadAllAsync())   // один читач зберігає порядок
        await squares.Writer.WriteAsync((n, n * n));
    squares.Writer.Complete();                               // закриття передається далі по конвеєру
});
Task print = Task.Run(async () =>
{
    await foreach (var (n, sq) in squares.Reader.ReadAllAsync())
        Console.WriteLine($"{n}² = {sq}");
});
await Task.WhenAll(generate, square, print);
Console.WriteLine("Конвеєр завершено");
```

**Приклад запуску:**
```
1² = 1
2² = 4
3² = 9
4² = 16
5² = 25
Конвеєр завершено
```

**Приклад 4 — bounded-канал і backpressure:** повний канал пригальмовує виробника або відкидає дані.

```csharp
using System.Threading.Channels;
// FullMode = Wait: коли канал повний, WriteAsync чекає, а TryWrite повертає false — це backpressure
var bounded = Channel.CreateBounded<int>(new BoundedChannelOptions(2)
{
    FullMode = BoundedChannelFullMode.Wait,
    SingleReader = true,
    SingleWriter = true
});
Console.WriteLine($"TryWrite 1: {bounded.Writer.TryWrite(1)}, 2: {bounded.Writer.TryWrite(2)}, 3: {bounded.Writer.TryWrite(3)}");
ValueTask pending = bounded.Writer.WriteAsync(3);            // виробник "пригальмовано"
Console.WriteLine($"WriteAsync(3) завершено: {pending.IsCompleted}");
bounded.Reader.TryRead(out int first);                       // споживач звільнив місце
await pending;                                               // тепер запис пройшов
Console.WriteLine($"Прочитано {first}, WriteAsync(3) завершено після читання");
// DropOldest: запис ніколи не чекає — найстаріший елемент викидається (напр., останні показники датчика)
var latest = Channel.CreateBounded<int>(new BoundedChannelOptions(3) { FullMode = BoundedChannelFullMode.DropOldest });
for (int i = 1; i <= 6; i++) latest.Writer.TryWrite(i);
latest.Writer.Complete();
Console.WriteLine($"DropOldest залишив: {string.Join(" ", await latest.Reader.ReadAllAsync().ToListAsync())}");
// DropWrite: викидається НОВИЙ елемент, що не влазить
var keepFirst = Channel.CreateBounded<int>(new BoundedChannelOptions(3) { FullMode = BoundedChannelFullMode.DropWrite });
for (int i = 1; i <= 6; i++) keepFirst.Writer.TryWrite(i);
keepFirst.Writer.Complete();
var kept = new List<int>();
while (await keepFirst.Reader.WaitToReadAsync())             // класичний цикл: чекаємо даних, потім вичитуємо все
    while (keepFirst.Reader.TryRead(out int x)) kept.Add(x);
Console.WriteLine($"DropWrite залишив: {string.Join(" ", kept)}");
```

**Приклад запуску:**
```
TryWrite 1: True, 2: True, 3: False
WriteAsync(3) завершено: False
Прочитано 1, WriteAsync(3) завершено після читання
DropOldest залишив: 4 5 6
DropWrite залишив: 1 2 3
```

**Приклад 5 — скасування і запис у закритий канал:**

```csharp
using System.Threading.Channels;
var channel = Channel.CreateUnbounded<int>();
using var cts = new CancellationTokenSource(TimeSpan.FromMilliseconds(100)); // скасування через 100 мс
try
{
    int value = await channel.Reader.ReadAsync(cts.Token);  // даних немає і не буде — без токена чекали б вічно
    Console.WriteLine(value);
}
catch (OperationCanceledException)
{
    Console.WriteLine("Читання скасовано за тайм-аутом");
}
// Помилка: запис після Complete()
channel.Writer.Complete();
Console.WriteLine($"TryWrite після Complete: {channel.Writer.TryWrite(1)}"); // false, без винятку
try
{
    await channel.Writer.WriteAsync(1);
}
catch (ChannelClosedException)
{
    Console.WriteLine("WriteAsync після Complete: ChannelClosedException");
}
Console.WriteLine($"Reader.Completion завершено: {channel.Reader.Completion.IsCompleted}");
```

**Приклад запуску:**
```
Читання скасовано за тайм-аутом
TryWrite після Complete: False
WriteAsync після Complete: ChannelClosedException
Reader.Completion завершено: True
```

**Типові помилки з каналами:**
- **Забутий `Complete()`** — `await foreach (… ReadAllAsync())` ніколи не завершиться, програма «зависне».
- **`Complete()` зарано** (до завершення всіх виробників) — інші виробники отримають `ChannelClosedException`.
- **Unbounded-канал при повільному споживачі** — черга росте без меж, аж до `OutOfMemoryException`; використовуйте `CreateBounded` з `FullMode = Wait`.
- **Запис після `Complete()`** — `WriteAsync` кидає `ChannelClosedException`, `TryWrite` мовчки повертає `false` (дані губляться).
- **Нескінченне очікування** `ReadAsync` без `CancellationToken` — передавайте токен для тайм-аутів і зупинки сервісу.

---

## 13. 15 сучасних можливостей C#
Можливості C# 7–12, які постійно трапляються в сучасному коді та в прикладах курсу. Кожен приклад — окрема програма з top-level statements.

### 13.1. Span<T> і ReadOnlySpan<T>
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

### 13.2. Memory<T>
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

### 13.3. Індекси та діапазони (`^`, `..`)
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

### 13.4. record і `with`
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

### 13.5. Pattern matching: property, list, relational
Патерни перевіряють **форму** даних: тип (`int n`), порівняння (`> 0`), логічні комбінації (`and`/`or`/`not`), властивості (`{ Address.City: "Київ" }`), позиції (`(0, _)`), списки (`[1, .., var last]`). Докладно, з прикладами й типовими помилками, — у [§4.4–4.7](#44-базові-патерни-константа-тип-var-_-реляційні-логічні).

### 13.6. switch-вирази
`x switch { патерн => значення, _ => ... }` — вираз, що повертає результат; компілятор попереджає про неповне покриття (CS8509) і забороняє недосяжні гілки (CS8510). Перемикання за кортежами `(a, b) switch` зручне для таблиць рішень. Докладно — у [§4.3](#43-switch-вирази), практичні приклади — у [§4.8](#48-практичні-приклади).

### 13.7. Nullable reference types і `?.`, `??`, `??=`
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

### 13.8. Кортежі та деконструкція
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

### 13.9. `init` і `required` властивості
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

### 13.10. Primary constructors (C# 12)
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

### 13.11. Collection expressions і spread `..` (C# 12)
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

### 13.12. Raw string literals `"""`
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

### 13.13. async/await
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

### 13.14. Generic math: `INumber<T>` (C# 11)
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

### 13.15. Ключове слово `field` (C# 14)
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

## 14. Підсумки
- C# має **фіксовані розміри** типів, `const` (компіляція) і `readonly` (виконання), статичний `var`.
- Класичний `switch` не має fall-through (CS0163); `switch`-вирази й патерни — константні, типові, реляційні, логічні, property, позиційні, кортежні та list — замінюють ланцюжки `if`; гілки йдуть від вужчих до ширших (CS8510), неповнота — CS8509; для закритих ієрархій даних патерни, для розширюваних — поліморфізм.
- Масиви: `int[]`, прямокутні `int[,]`, зубчасті `int[][]`; `foreach` — зручний обхід.
- `ref`/`out`/`in` замінюють посилання та вказівники C++; пам'ять звільняє GC.
- ООП: один базовий клас + інтерфейси; `record` — рівність за значенням; `string?` — явний null.
- Винятки — `try/catch/finally`; LINQ — декларативна обробка колекцій.
- Бінарний пошук O(log n) на відсортованих даних радикально швидший за лінійний O(n); стежте за `lo + (hi - lo) / 2` і межами циклу, для частих пошуків — `HashSet<T>`.
- Стиль: `PascalCase` для типів і методів, `camelCase` для локальних, `_camelCase` для полів; правила — у `.editorconfig`, перевірка — `dotnet format`.
- Файли: шляхи — через `Path.Combine`; малі файли — `File.ReadAllText`/`WriteAllLines`, великі — ліниво `File.ReadLines` або `StreamReader` з `using`; числа в CSV — з `CultureInfo.InvariantCulture`; JSON — `System.Text.Json`; помилки I/O — `try/catch` від конкретних винятків до `IOException`.
- Власні `MyList<T>`, `MyLinkedList<T>`, `MyQueue<T>`, `MyChannel<T>` показують механіку колекцій; у робочому коді — `List<T>`, `LinkedList<T>`, `Queue<T>` і `System.Threading.Channels` (не забувайте `Writer.Complete()` і bounded-канали для backpressure).
- `Span<T>`/`Memory<T>` дають зрізи без алокацій; `record`, `with`, `init`/`required`, primary constructors, collection expressions і `field` (C# 14) скорочують шаблонний код.
- Патерни, switch-вирази, nullable-анотації, `async/await` і generic math (`INumber<T>`) — основа виразного та безпечного сучасного C#.

---

## 15. Питання для самоперевірки
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
13. Чим `File.ReadLines` відрізняється від `File.ReadAllLines` і що обрати для лог-файлу розміром 5 ГБ?
14. Чому `double.Parse("95.5")` може впасти на комп'ютері з українською локаллю і як це виправити?
15. Що станеться, якщо не викликати `Dispose` для `StreamWriter`, і в якому порядку ставити `catch` для `FileNotFoundException` та `IOException`?
16. Чому `MyList<T>` подвоює масив, а не збільшує його на 1, і яка амортизована складність `Add`?
17. Що станеться зі споживачем `await foreach (var x in reader.ReadAllAsync())`, якщо виробник не викличе `Complete()`?
18. Чим відрізняються `BoundedChannelFullMode.Wait`, `DropOldest` і `DropWrite`?
19. Чим switch-вираз відрізняється від switch-оператора і що станеться під час виконання, якщо жодна гілка не збіглася?
20. Чому `score switch { >= 60 => "C", >= 90 => "A", _ => "F" }` не компілюється і як це виправити?
21. Чим відрізняються патерни `{ Address.City: "Київ" }`, `(0, _)` та `[var first, .., var last]` і що кожен вимагає від типу?
22. Коли для ієрархії типів краще обрати `switch` з патернами, а коли — `virtual`-методи?
