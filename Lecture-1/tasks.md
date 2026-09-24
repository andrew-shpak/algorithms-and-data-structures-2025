# Лекція 1 — Прості завдання на C#

[Матеріал лекції](Examples.md)

Кожне завдання — окрема невелика програма або метод. Дані можна задати в коді. Перевірте звичайний приклад і граничні випадки з таблиць. Готові алгоритми з наступних лекцій тут не потрібні.

**Усього: 12 завдань із повними реалізаціями та очікуваним виводом.**

> **Запуск реалізацій:** C# 14 / .NET 10. Встановіть .NET 10 SDK. Створіть консольний проєкт через `dotnet new console --framework net10.0`, замініть `Program.cs` одним повним блоком `csharp` і виконайте `dotnet run`. Кожен блок незалежний; класи й методи з інших завдань копіювати не потрібно. Для порожніх результатів у виводі використовуємо `[]`; логічні значення C# друкує як `True` / `False`.

## Завдання 1. Хвилини на годиннику

Дано кількість хвилин від початку доби `minutes`, де `0 ≤ minutes < 1440`. Виведіть час у форматі `HH:mm`, завжди по дві цифри для годин і хвилин.

**Вимоги:** використайте цілочисельне ділення, остачу `%` та форматування `D2`. Не використовуйте `DateTime`.

| minutes | Результат |
|---|---|
| `75` | `01:15` |
| `0` | `00:00` |
| `1439` | `23:59` |

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

foreach (int minutes in new[] { 75, 0, 1439 })
{
    int hours = minutes / 60;
    int rest = minutes % 60;
    Console.WriteLine($"{hours:D2}:{rest:D2}");
}
```

**Очікуваний вивід:**

```text
01:15
00:00
23:59
```

</details>

## Завдання 2. Команда світлофора

Метод `Signal(string color)` повертає `СТІЙ` для `red`, `ЧЕКАЙ` для `yellow`, `ЙДИ` для `green` і `НЕВІДОМО` для будь-якого іншого рядка. Вхідні значення порівнюйте точно, з урахуванням регістру.

**Вимоги:** застосуйте `switch` або switch-вираз; метод повертає рядок, а виведення робить виклик у головній програмі.

| color | Результат |
|---|---|
| `red` | `СТІЙ` |
| `green` | `ЙДИ` |
| `yellow` | `ЧЕКАЙ` |
| `Green` або `""` | `НЕВІДОМО` |

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

foreach (string color in new[] { "red", "green", "yellow", "Green", "" })
{
    Console.WriteLine(Signal(color));
}

static string Signal(string color) => color switch
{
    "red" => "СТІЙ",
    "yellow" => "ЧЕКАЙ",
    "green" => "ЙДИ",
    _ => "НЕВІДОМО"
};
```

**Очікуваний вивід:**

```text
СТІЙ
ЙДИ
ЧЕКАЙ
НЕВІДОМО
НЕВІДОМО
```

</details>

## Завдання 3. Скільки разів стало тепліше

Масив містить температуру за послідовні дні. Порахуйте, скільки днів, починаючи з другого, були **строго теплішими за попередній день**. Рівні температури не враховуйте.

**Вимоги:** один цикл, без сортування. Не порівнюйте день із усіма попередніми днями.

| Температури | Результат |
|---|---|
| `[12, 15, 15, 10, 13]` | `2` |
| `[3, 2, 1]` | `0` |
| `[7]` | `0` |
| `[]` | `0` |

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

int[][] examples = { new[] { 12, 15, 15, 10, 13 }, new[] { 3, 2, 1 }, new[] { 7 }, Array.Empty<int>() };
foreach (int[] temperatures in examples)
{
    Console.WriteLine(CountWarmerDays(temperatures));
}

static int CountWarmerDays(int[] temperatures)
{
    int count = 0;
    for (int i = 1; i < temperatures.Length; i++)
    {
        if (temperatures[i] > temperatures[i - 1])
        {
            count++;
        }
    }
    return count;
}
```

**Очікуваний вивід:**

```text
2
0
0
0
```

</details>

## Завдання 4. Лічильник із межею

Створіть клас `LimitedCounter` з незмінною невід'ємною межею `Limit` та властивістю `Value`, початково `0`. Метод `TryIncrement()` збільшує значення на один і повертає `true`, якщо межі ще не досягнуто; інакше повертає `false` без зміни значення. Метод `Reset()` встановлює `Value = 0`.

**Вимоги:** змінювати `Value` ззовні класу не можна. Виведіть результат кожного виклику й кінцеве значення.

| Limit; виклики | Повернені значення TryIncrement | Кінцевий Value |
|---|---|---|
| `2; TryIncrement × 3` | `true, true, false` | `2` |
| `1; TryIncrement, Reset, TryIncrement` | `true, true` | `1` |
| `0; TryIncrement` | `false` | `0` |

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

var counter = new LimitedCounter(2);
Console.WriteLine($"{counter.TryIncrement()}, {counter.TryIncrement()}, {counter.TryIncrement()}; Value={counter.Value}");

var resetExample = new LimitedCounter(1);
bool first = resetExample.TryIncrement();
resetExample.Reset();
bool second = resetExample.TryIncrement();
Console.WriteLine($"{first}, {second}; Value={resetExample.Value}");

var zero = new LimitedCounter(0);
Console.WriteLine($"{zero.TryIncrement()}; Value={zero.Value}");

public sealed class LimitedCounter
{
    public int Limit { get; }
    public int Value { get; private set; }

    public LimitedCounter(int limit)
    {
        if (limit < 0)
        {
            throw new ArgumentOutOfRangeException(nameof(limit));
        }
        Limit = limit;
    }

    public bool TryIncrement()
    {
        if (Value >= Limit)
        {
            return false;
        }
        Value++;
        return true;
    }

    public void Reset() => Value = 0;
}
```

**Очікуваний вивід:**

```text
True, True, False; Value=2
True, True; Value=1
False; Value=0
```

</details>

## Завдання 5. Безпечна швидкість

Метод `LimitSpeed` обмежує швидкість діапазоном `0..90`: від’ємне значення замінює на `0`, більше за `90` — на `90`. Використайте `if`, без `Math.Clamp`. Перевірте обидві межі.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

foreach (int speed in new[] { -5, 0, 45, 90, 120 })
    Console.WriteLine(LimitSpeed(speed));

static int LimitSpeed(int speed)
{
    if (speed < 0) return 0;
    if (speed > 90) return 90;
    return speed;
}
```

**Очікуваний вивід:**

```text
0
0
45
90
90
```

</details>

## Завдання 6. Знак вимірювання

Для цілого числа поверніть `нижче нуля`, `нуль` або `вище нуля`. Використайте реляційні патерни у switch-виразі. Число не змінюйте.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

foreach (int value in new[] { -12, 0, 7 })
    Console.WriteLine(SignLabel(value));

static string SignLabel(int value) => value switch
{
    < 0 => "нижче нуля",
    0 => "нуль",
    > 0 => "вище нуля"
};
```

**Очікуваний вивід:**

```text
нижче нуля
нуль
вище нуля
```

</details>

## Завдання 7. Кожен другий сигнал

Виведіть числа від невід’ємного `start` до нуля з кроком `-2`. Якщо `start` непарне, останнім буде `1`. Використайте цикл; результат оформіть у квадратних дужках.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

foreach (int start in new[] { 6, 5, 0 })
{
    var signals = new List<int>();
    for (int value = start; value >= 0; value -= 2)
        signals.Add(value);
    Console.WriteLine($"[{string.Join(", ", signals)}]");
}
```

**Очікуваний вивід:**

```text
[6, 4, 2, 0]
[5, 3, 1]
[0]
```

</details>

## Завдання 8. Елементи на парних індексах

Поверніть елементи масиву за індексами `0, 2, 4, ...`. Парність самих значень не має значення. Початковий масив не змінюйте; порожній дає порожній результат.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

foreach (int[] values in new[] { new[] { 9, 2, 7, 4, 5 }, new[] { 8 }, Array.Empty<int>() })
{
    var result = new List<int>();
    for (int i = 0; i < values.Length; i += 2)
        result.Add(values[i]);
    Console.WriteLine($"[{string.Join(", ", result)}]");
}
```

**Очікуваний вивід:**

```text
[9, 7, 5]
[8]
[]
```

</details>

## Завдання 9. Назва без пробілів

Замініть кожен звичайний пробіл на `_`, зберігши решту символів. Використайте цикл по символах, без `Replace`. Два пробіли мають дати два підкреслення.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

foreach (string name in new[] { "red fox", "a  b", "", "cat" })
    Console.WriteLine($"[{MakeLabel(name)}]");

static string MakeLabel(string name)
{
    char[] result = name.ToCharArray();
    for (int i = 0; i < result.Length; i++)
        if (result[i] == ' ') result[i] = '_';
    return new string(result);
}
```

**Очікуваний вивід:**

```text
[red_fox]
[a__b]
[]
[cat]
```

</details>

## Завдання 10. Остання позначка

Знайдіть індекс останнього входження заданого числа в масиві. Почніть пошук з кінця; якщо збігу немає, поверніть `-1`. Готові методи пошуку не використовуйте.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

Console.WriteLine(LastPosition(new[] { 4, 7, 4, 2 }, 4));
Console.WriteLine(LastPosition(new[] { 4, 7 }, 9));
Console.WriteLine(LastPosition(Array.Empty<int>(), 4));

static int LastPosition(int[] values, int target)
{
    for (int i = values.Length - 1; i >= 0; i--)
        if (values[i] == target) return i;
    return -1;
}
```

**Очікуваний вивід:**

```text
2
-1
-1
```

</details>

## Завдання 11. Число або повідомлення

Спробуйте прочитати ціле число через `int.TryParse`. За успіху виведіть саме число; за помилки — `ERROR`. Некоректний текст не має спричиняти виняток.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

foreach (string text in new[] { "42", "-7", "cat", "", "99999999999999999999" })
{
    if (int.TryParse(text, out int value))
        Console.WriteLine(value);
    else
        Console.WriteLine("ERROR");
}
```

**Очікуваний вивід:**

```text
42
-7
ERROR
ERROR
ERROR
```

</details>

## Завдання 12. Прямокутна рамка

Створіть `record Frame` з цілими шириною та висотою у діапазоні `0..1000`. Обчислювана властивість `Perimeter` повертає `2 * (Width + Height)`. Перевірте звичайну рамку, квадрат і нульові розміри.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

foreach (var frame in new[] { new Frame(3, 5), new Frame(4, 4), new Frame(0, 0) })
    Console.WriteLine(frame.Perimeter);

public record Frame(int Width, int Height)
{
    public int Perimeter => 2 * (Width + Height);
}
```

**Очікуваний вивід:**

```text
16
16
0
```

</details>
