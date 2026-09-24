# Лекція 4 — Прості завдання на рекурсію, generics і лямбди

[Матеріал лекції](Examples.md)

Виконуйте кожне завдання як окремий метод C#. Для рекурсії спочатку запишіть базовий випадок словами. Вхідні дані невеликі; файли й інтерактивне меню не потрібні.

**Усього: 12 завдань із повними реалізаціями та очікуваним виводом.**

> **Запуск реалізацій:** C# 14 / .NET 10. Встановіть .NET 10 SDK. Створіть консольний проєкт через `dotnet new console --framework net10.0`, замініть `Program.cs` одним повним блоком `csharp` і виконайте `dotnet run`. Кожен блок незалежний; класи й методи з інших завдань копіювати не потрібно. Для порожніх результатів у виводі використовуємо `[]`; логічні значення C# друкує як `True` / `False`.

## Завдання 1. Рекурсивне відлуння

Метод `Echo(string text, int count)` повертає рядок із `count` повторень `text` без роздільників, де `0 ≤ count ≤ 20`.

**Вимоги:** використайте рекурсію без циклів. Для `count = 0` поверніть порожній рядок; кожен наступний виклик має зменшувати `count`.

| text; count | Результат |
|---|---|
| `"ха"; 3` | `хахаха` |
| `"!"; 1` | `!` |
| `"ха"; 0` | `""` |
| `""; 4` | `""` |

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

Console.WriteLine($"[{Echo("ха", 3)}]");
Console.WriteLine($"[{Echo("!", 1)}]");
Console.WriteLine($"[{Echo("ха", 0)}]");
Console.WriteLine($"[{Echo("", 4)}]");

static string Echo(string text, int count)
{
    if (count == 0)
    {
        return "";
    }
    return text + Echo(text, count - 1);
}
```

**Очікуваний вивід:**

```text
[хахаха]
[!]
[]
[]
```

</details>

## Завдання 2. Кількість цифр

Рекурсивний метод `DigitCount(int n)` визначає кількість десяткових цифр невід'ємного числа. Число `0` має одну цифру.

**Вимоги:** не перетворюйте число на рядок і не використовуйте логарифми. Скорочуйте число цілочисельним діленням на `10`.

| n | Результат |
|---|---|
| `5070` | `4` |
| `10` | `2` |
| `7` | `1` |
| `0` | `1` |

**Самоперевірка:** намалюйте виклики для `5070` та підпишіть значення, яке повертає кожен із них.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

foreach (int number in new[] { 5070, 10, 7, 0 })
{
    Console.WriteLine(DigitCount(number));
}

static int DigitCount(int n)
{
    if (n < 10)
    {
        return 1;
    }
    return 1 + DigitCount(n / 10);
}
```

**Очікуваний вивід:**

```text
4
2
1
1
```

</details>

## Завдання 3. Поміняти місцями пару будь-якого типу

Напишіть узагальнений метод `SwapPair<T>((T First, T Second) pair)`, який повертає нову пару `(Second, First)`. Викличте той самий метод для цілих чисел і рядків.

**Вимоги:** не використовуйте `object`, приведення типів або окремі перевантаження для `int` і `string`.

| Вхідна пара | Результат |
|---|---|
| `(3, 8)` | `(8, 3)` |
| `("ліво", "право")` | `("право", "ліво")` |
| `(5, 5)` | `(5, 5)` |

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

Console.WriteLine(SwapPair((3, 8)));
Console.WriteLine(SwapPair(("ліво", "право")));
Console.WriteLine(SwapPair((5, 5)));

static (T First, T Second) SwapPair<T>((T First, T Second) pair)
{
    return (pair.Second, pair.First);
}
```

**Очікуваний вивід:**

```text
(8, 3)
(право, ліво)
(5, 5)
```

</details>

## Завдання 4. Одна функція — різні перетворення

Напишіть `Transform(int[] values, Func<int, int> operation)`, що створює новий масив і застосовує `operation` до кожного числа. Значення за модулем не перевищують `100`.

**Вимоги:** пройдіть масив циклом, не змінюючи початкових даних. Передайте лямбди `x => x + 1`, `x => x * x`, `x => -x`.

| Масив; перетворення | Результат |
|---|---|
| `[2, -3, 0]; x => x + 1` | `[3, -2, 1]` |
| `[2, -3, 0]; x => x * x` | `[4, 9, 0]` |
| `[2, -3, 0]; x => -x` | `[-2, 3, 0]` |
| `[]; x => x + 1` | `[]` |

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

int[] values = { 2, -3, 0 };
Console.WriteLine(string.Join(", ", Transform(values, x => x + 1)));
Console.WriteLine(string.Join(", ", Transform(values, x => x * x)));
Console.WriteLine(string.Join(", ", Transform(values, x => -x)));
Console.WriteLine($"Length={Transform(Array.Empty<int>(), x => x + 1).Length}");
Console.WriteLine($"Початковий: {string.Join(", ", values)}");

static int[] Transform(int[] values, Func<int, int> operation)
{
    int[] result = new int[values.Length];
    for (int i = 0; i < values.Length; i++)
    {
        result[i] = operation(values[i]);
    }
    return result;
}
```

**Очікуваний вивід:**

```text
3, -2, 1
4, 9, 0
-2, 3, 0
Length=0
Початковий: 2, -3, 0
```

</details>

## Завдання 5. Сходинки зі зірочок

Рекурсивно виведіть рядки з `1, 2, ..., n` зірочками, де `0 ≤ n ≤ 10`. Спочатку виконайте рекурсивний виклик для `n - 1`, потім надрукуйте поточний рядок. Для нуля нічого не друкуйте.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

PrintSteps(3);
PrintSteps(0);
Console.WriteLine("END");

static void PrintSteps(int n)
{
    if (n == 0) return;
    PrintSteps(n - 1);
    Console.WriteLine(new string('*', n));
}
```

**Очікуваний вивід:**

```text
*
**
***
END
```

</details>

## Завдання 6. Рекурсивний пошук символу

Перевірте, чи є заданий символ у рядку, починаючи з індексу `0`. На кожному кроці переходьте на один символ далі. Не використовуйте цикли або `Contains`; довжина рядка не перевищує `100`.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

Console.WriteLine(ContainsChar("planet", 'n', 0));
Console.WriteLine(ContainsChar("planet", 'x', 0));
Console.WriteLine(ContainsChar("", 'a', 0));

static bool ContainsChar(string text, char target, int index)
{
    if (index == text.Length) return false;
    if (text[index] == target) return true;
    return ContainsChar(text, target, index + 1);
}
```

**Очікуваний вивід:**

```text
True
False
False
```

</details>

## Завдання 7. Ділення через віднімання

Рекурсивно обчисліть цілу частку `a / b`, де `0 ≤ a ≤ 100`, `b > 0`. Якщо `a < b`, поверніть `0`; інакше відніміть `b` і додайте один до результату наступного виклику. Оператори `/` та `%` не використовуйте.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

Console.WriteLine(Quotient(17, 5));
Console.WriteLine(Quotient(4, 7));
Console.WriteLine(Quotient(0, 3));
Console.WriteLine(Quotient(9, 3));

static int Quotient(int a, int b)
{
    if (a < b) return 0;
    return 1 + Quotient(a - b, b);
}
```

**Очікуваний вивід:**

```text
3
0
0
3
```

</details>

## Завдання 8. Двійкові слова без сусідніх одиниць

Згенеруйте всі рядки довжини `n`, складені з `0` та `1`, без підрядка `11`. `0 ≤ n ≤ 8`. Рекурсивно пробуйте `0` перед `1`; для `n = 0` існує один порожній рядок. Показуйте кожен рядок у квадратних дужках.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

foreach (string word in Generate(3, "")) Console.WriteLine($"[{word}]");
foreach (string word in Generate(0, "")) Console.WriteLine($"[{word}]");

static List<string> Generate(int remaining, string prefix)
{
    if (remaining == 0) return new List<string> { prefix };
    var result = Generate(remaining - 1, prefix + "0");
    if (prefix.Length == 0 || prefix[^1] != '1')
        result.AddRange(Generate(remaining - 1, prefix + "1"));
    return result;
}
```

**Очікуваний вивід:**

```text
[000]
[001]
[010]
[100]
[101]
[]
```

</details>

## Завдання 9. Коробка будь-якого типу

Створіть клас `Box<T>` із властивістю `Value`. Метод `Exchange` записує нове значення й повертає попереднє. Перевірте одну коробку з числом і другу з рядком; приведення до `object` не потрібне.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

var number = new Box<int>(5);
Console.WriteLine($"old={number.Exchange(9)}, new={number.Value}");
var word = new Box<string>("day");
Console.WriteLine($"old={word.Exchange("night")}, new={word.Value}");

public sealed class Box<T>(T value)
{
    public T Value { get; private set; } = value;
    public T Exchange(T next)
    {
        T previous = Value;
        Value = next;
        return previous;
    }
}
```

**Очікуваний вивід:**

```text
old=5, new=9
old=day, new=night
```

</details>

## Завдання 10. Останній елемент або запасне значення

Напишіть `LastOr<T>` для масиву довільного типу. Якщо масив непорожній, поверніть останній елемент; інакше — передане запасне значення. Використайте один узагальнений метод для чисел і рядків.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

Console.WriteLine(LastOr(new[] { 2, 5, 8 }, -1));
Console.WriteLine(LastOr(Array.Empty<int>(), -1));
Console.WriteLine(LastOr(new[] { "east", "west" }, "none"));
Console.WriteLine(LastOr(Array.Empty<string>(), "none"));

static T LastOr<T>(T[] values, T fallback)
{
    return values.Length == 0 ? fallback : values[^1];
}
```

**Очікуваний вивід:**

```text
8
-1
west
none
```

</details>

## Завдання 11. Замикання з накопиченим добутком

Фабрика повертає `Func<int, int>` із власним накопиченим добутком, початково `1`. Кожен виклик множить його на аргумент і повертає нове значення. Дві функції від різних викликів фабрики повинні мати незалежний стан. Використовуйте невеликі числа.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

var first = MakeMultiplier();
var second = MakeMultiplier();
Console.WriteLine(first(2));
Console.WriteLine(first(3));
Console.WriteLine(second(5));
Console.WriteLine(first(0));

static Func<int, int> MakeMultiplier()
{
    int product = 1;
    return value =>
    {
        product *= value;
        return product;
    };
}
```

**Очікуваний вивід:**

```text
2
6
5
0
```

</details>

## Завдання 12. Порядок двох перетворень

Напишіть `Compose(first, second)`, що повертає функцію `x => second(first(x))`. Порівняйте «додати 2, потім помножити на 3» і зворотний порядок. Також перевірте композицію з тотожним перетворенням.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

var addThenMultiply = Compose(x => x + 2, x => x * 3);
var multiplyThenAdd = Compose(x => x * 3, x => x + 2);
Console.WriteLine(addThenMultiply(4));
Console.WriteLine(multiplyThenAdd(4));
Console.WriteLine(Compose(x => x, x => -x)(4));

static Func<int, int> Compose(Func<int, int> first, Func<int, int> second)
{
    return x => second(first(x));
}
```

**Очікуваний вивід:**

```text
18
14
-4
```

</details>
