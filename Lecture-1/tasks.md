# Лекція 1 — Прості завдання на C#

[Матеріал лекції](Examples.md)

Кожне завдання — окрема невелика програма або метод. Дані можна задати в коді. Перевірте звичайний приклад і граничні випадки з таблиць. Готові алгоритми з наступних лекцій тут не потрібні.

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
