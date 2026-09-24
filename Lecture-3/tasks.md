# Лекція 3 — Прості завдання на колекції .NET

[Матеріал лекції](Examples.md) · [Базові приклади списку, стека й черги](../Lecture-2/tasks.md)

Кожне завдання незалежне; мова — C#. Дані задавайте в коді, без файлів і меню. Тут обираємо колекцію під конкретну операцію. `[]` означає порожню колекцію.

> **Запуск реалізацій:** C# 14 / .NET 10. Встановіть .NET 10 SDK. Створіть консольний проєкт через `dotnet new console --framework net10.0`, замініть `Program.cs` одним повним блоком `csharp` і виконайте `dotnet run`. Кожен блок незалежний; класи й методи з інших завдань копіювати не потрібно. Для порожніх результатів у виводі використовуємо `[]`; логічні значення C# друкує як `True` / `False`.

## Завдання 1. Переплести дві смужки — `List<string>`

Створіть новий список: перший елемент першої смужки, перший елемент другої, другий елемент першої, другий елемент другої тощо. Коли одна смужка закінчиться, допишіть решту іншої.

**Вимоги:** початкові списки не змінюйте, використовуйте індекси та `Add`. Не сортуйте елементи.

| Перша смужка | Друга смужка | Результат |
|---|---|---|
| `[A, B, C]` | `[1, 2]` | `[A, 1, B, 2, C]` |
| `[A]` | `[1, 2, 3]` | `[A, 1, 2, 3]` |
| `[]` | `[1, 2]` | `[1, 2]` |
| `[]` | `[]` | `[]` |

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

Console.WriteLine(string.Join(", ", Weave(new() { "A", "B", "C" }, new() { "1", "2" })));
Console.WriteLine(string.Join(", ", Weave(new() { "A" }, new() { "1", "2", "3" })));
Console.WriteLine(string.Join(", ", Weave(new(), new() { "1", "2" })));
Console.WriteLine($"Count={Weave(new(), new()).Count}");

static List<string> Weave(List<string> first, List<string> second)
{
    var result = new List<string>();
    for (int i = 0; i < first.Count || i < second.Count; i++)
    {
        if (i < first.Count)
        {
            result.Add(first[i]);
        }
        if (i < second.Count)
        {
            result.Add(second[i]);
        }
    }
    return result;
}
```

**Очікуваний вивід:**

```text
A, 1, B, 2, C
A, 1, 2, 3
1, 2
Count=0
```

</details>

## Завдання 2. Перечепити вагон — `LinkedList<string>`

Є список вагонів з унікальними назвами. Знайдіть заданий вагон і перенесіть **той самий вузол** у кінець. Якщо такого вагона немає або він уже останній, нічого не змінюйте.

**Вимоги:** використайте `Find`, `Remove(node)`, `AddLast(node)`. Спочатку від'єднайте вузол: вузол, який ще належить списку, не можна додати повторно.

| Вагони; назва | Результат |
|---|---|
| `[A, B, C, D]; B` | `[A, C, D, B]` |
| `[A, B]; B` | `[A, B]` |
| `[A]; X` | `[A]` |
| `[]; A` | `[]` |

**Питання:** яка частина операції потребує пошуку O(n), а які кроки виконуються за O(1)?

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

(string[] Wagons, string Name)[] examples =
{
    (new[] { "A", "B", "C", "D" }, "B"),
    (new[] { "A", "B" }, "B"),
    (new[] { "A" }, "X"),
    (Array.Empty<string>(), "A")
};
foreach (var example in examples)
{
    var train = new LinkedList<string>(example.Wagons);
    MoveToEnd(train, example.Name);
    Console.WriteLine($"[{string.Join(", ", train)}]");
}

static void MoveToEnd(LinkedList<string> train, string name)
{
    var node = train.Find(name);
    if (node is null || node == train.Last)
    {
        return;
    }
    train.Remove(node);
    train.AddLast(node);
}
```

**Очікуваний вивід:**

```text
[A, C, D, B]
[A, B]
[A]
[]
```

</details>

## Завдання 3. Розшифрувати позначення — `Dictionary<string, string>`

Створіть словник `cm → сантиметр`, `m → метр`, `kg → кілограм`. Для кожного коду поверніть назву або `невідома одиниця`, якщо ключа немає. Порівняння ключів має враховувати регістр.

**Вимоги:** використайте `TryGetValue`; невідомий код не повинен спричиняти виняток чи додаватися до словника.

| Коди | Результат у тому самому порядку |
|---|---|
| `[m, kg, cm]` | `[метр, кілограм, сантиметр]` |
| `[km, M, m]` | `[невідома одиниця, невідома одиниця, метр]` |
| `[]` | `[]` |

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

var units = new Dictionary<string, string>(StringComparer.Ordinal)
{
    ["cm"] = "сантиметр",
    ["m"] = "метр",
    ["kg"] = "кілограм"
};
string[][] examples =
{
    new[] { "m", "kg", "cm" },
    new[] { "km", "M", "m" },
    Array.Empty<string>()
};
foreach (string[] codes in examples)
{
    var names = new List<string>();
    foreach (string code in codes)
    {
        names.Add(units.TryGetValue(code, out string? name) ? name : "невідома одиниця");
    }
    Console.WriteLine($"[{string.Join(", ", names)}]");
}
Console.WriteLine($"Ключів: {units.Count}");
```

**Очікуваний вивід:**

```text
[метр, кілограм, сантиметр]
[невідома одиниця, невідома одиниця, метр]
[]
Ключів: 3
```

</details>

## Завдання 4. Алфавіт робота — `HashSet<char>`

Робот розуміє лише команди `L`, `R`, `U`, `D`. Знайдіть **індекс першого недозволеного символу** у рядку або поверніть `-1`, якщо всі символи дозволені. Пробіли й малі літери недозволені.

**Вимоги:** множина зберігає дозволені символи; перевірку виконуйте через `Contains`. Сам рядок переглядайте зліва направо й завершуйте пошук після першої помилки.

| Команди | Результат |
|---|---|
| `LURD` | `-1` |
| `LRXU` | `2` |
| `lR` | `0` |
| `L R` | `1` |
| `""` | `-1` |

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

foreach (string commands in new[] { "LURD", "LRXU", "lR", "L R", "" })
{
    Console.WriteLine(FirstInvalid(commands));
}

static int FirstInvalid(string commands)
{
    var allowed = new HashSet<char> { 'L', 'R', 'U', 'D' };
    for (int i = 0; i < commands.Length; i++)
    {
        if (!allowed.Contains(commands[i]))
        {
            return i;
        }
    }
    return -1;
}
```

**Очікуваний вивід:**

```text
-1
2
0
1
-1
```

</details>
