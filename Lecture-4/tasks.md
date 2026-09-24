# Лекція 4 — Прості завдання на рекурсію, generics і лямбди

[Матеріал лекції](Examples.md)

Виконуйте кожне завдання як окремий метод C#. Для рекурсії спочатку запишіть базовий випадок словами. Вхідні дані невеликі; файли й інтерактивне меню не потрібні.

> **Запуск реалізацій:** C# 12+ / .NET 8+. Створіть консольний проєкт через `dotnet new console`, замініть `Program.cs` одним повним блоком `csharp` і виконайте `dotnet run`. Кожен блок незалежний; класи й методи з інших завдань копіювати не потрібно. Для порожніх результатів у виводі використовуємо `[]`; логічні значення C# друкує як `True` / `False`.

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
