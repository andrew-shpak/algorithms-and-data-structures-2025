# Лекція 3 — Прості завдання на колекції .NET

[Матеріал лекції](Examples.md) · [Базові приклади списку, стека й черги](../Lecture-2/tasks.md)

Кожне завдання незалежне; мова — C#. Дані задавайте в коді, без файлів і меню. Тут обираємо колекцію під конкретну операцію. `[]` означає порожню колекцію.

**Усього: 12 завдань із повними реалізаціями та очікуваним виводом.**

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

## Завдання 5. Зворотний словник кольорів

Перетворіть словник `назва → код` на `код → назва`. У цій задачі і назви, і коди унікальні. Початковий словник не змінюйте; невідомий код позначте `NONE`.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

var colors = new Dictionary<string, int> { ["red"] = 10, ["blue"] = 20 };
var names = new Dictionary<int, string>();
foreach (var pair in colors) names.Add(pair.Value, pair.Key);
foreach (int code in new[] { 20, 10, 30 })
    Console.WriteLine(names.TryGetValue(code, out string? name) ? name : "NONE");
Console.WriteLine($"Original={colors.Count}");
```

**Очікуваний вивід:**

```text
blue
red
NONE
Original=2
```

</details>

## Завдання 6. Налаштування з перевизначеннями

Створіть новий словник налаштувань: спочатку базові значення, потім користувацькі. Якщо ключ повторюється, користувацьке значення перемагає. Обидва вхідні словники залишаються без змін.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

var defaults = new Dictionary<string, string> { ["theme"] = "light", ["size"] = "medium" };
var custom = new Dictionary<string, string> { ["theme"] = "dark", ["sound"] = "off" };
var result = new Dictionary<string, string>(defaults);
foreach (var pair in custom) result[pair.Key] = pair.Value;
foreach (string key in new[] { "theme", "size", "sound" })
    Console.WriteLine($"{key}={result[key]}");
Console.WriteLine($"Default theme={defaults["theme"]}");
```

**Очікуваний вивід:**

```text
theme=dark
size=medium
sound=off
Default theme=light
```

</details>

## Завдання 7. Перший повторний номер

Знайдіть перше число, яке повторно зустрічається під час читання масиву зліва направо. Використайте результат `HashSet.Add`: `false` означає, що значення вже було. Якщо повторів немає, поверніть `null`.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

foreach (int[] values in new[] { new[] { 4, 2, 7, 2, 4 }, new[] { 1, 2 }, Array.Empty<int>() })
    Console.WriteLine(FirstRepeat(values)?.ToString() ?? "NONE");

static int? FirstRepeat(int[] values)
{
    var seen = new HashSet<int>();
    foreach (int value in values)
        if (!seen.Add(value)) return value;
    return null;
}
```

**Очікуваний вивід:**

```text
2
NONE
NONE
```

</details>

## Завдання 8. Попередній доступний розмір — SortedSet

У впорядкованій множині знайдіть найбільший розмір, строго менший за запитаний. Для малої множини достатньо прямого перебору до першого завеликого значення. Якщо підхожого немає, поверніть `null`.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

var sizes = new SortedSet<int> { 36, 38, 40, 42 };
foreach (int wanted in new[] { 41, 38, 36 })
    Console.WriteLine(PreviousSize(sizes, wanted)?.ToString() ?? "NONE");

static int? PreviousSize(SortedSet<int> sizes, int wanted)
{
    int? result = null;
    foreach (int size in sizes)
    {
        if (size >= wanted) break;
        result = size;
    }
    return result;
}
```

**Очікуваний вивід:**

```text
40
36
NONE
```

</details>

## Завдання 9. Пари ключів і підписів

Два масиви однакової довжини містять унікальні номери шафок і відповідні підписи. Побудуйте словник за індексами. Продемонструйте також порожні масиви.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

var labels = BuildLabels(new[] { 12, 7, 25 }, new[] { "A", "B", "C" });
Console.WriteLine(labels[7]);
Console.WriteLine(labels[25]);
Console.WriteLine(BuildLabels(Array.Empty<int>(), Array.Empty<string>()).Count);

static Dictionary<int, string> BuildLabels(int[] keys, string[] values)
{
    var result = new Dictionary<int, string>();
    for (int i = 0; i < keys.Length; i++) result.Add(keys[i], values[i]);
    return result;
}
```

**Очікуваний вивід:**

```text
B
C
0
```

</details>

## Завдання 10. Пріоритет і порядок надходження

Обробіть повідомлення за зростанням числового пріоритету. Для рівних пріоритетів збережіть порядок додавання. Передайте в `PriorityQueue` пару `(priority, sequence)` як ключ пріоритету.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

var queue = new PriorityQueue<string, (int Priority, int Sequence)>();
(string Text, int Priority)[] messages = { ("A", 2), ("B", 1), ("C", 1), ("D", 3) };
for (int i = 0; i < messages.Length; i++)
    queue.Enqueue(messages[i].Text, (messages[i].Priority, i));
while (queue.TryDequeue(out string? text, out _)) Console.WriteLine(text);
Console.WriteLine($"Count={queue.Count}");
```

**Очікуваний вивід:**

```text
B
C
A
D
Count=0
```

</details>

## Завдання 11. Знімок списку для читання

Порівняйте обгортку `AsReadOnly()` над початковим списком і таку саму обгортку над його копією. Змініть початковий список. З’ясуйте, яка з двох колекцій показує нове значення, а яка зберігає старе.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

var original = new List<int> { 10, 20 };
var liveView = original.AsReadOnly();
var snapshot = new List<int>(original).AsReadOnly();
original[0] = 99;
original.Add(30);
Console.WriteLine($"View: {string.Join(", ", liveView)}");
Console.WriteLine($"Snapshot: {string.Join(", ", snapshot)}");
```

**Очікуваний вивід:**

```text
View: 99, 20, 30
Snapshot: 10, 20
```

</details>

## Завдання 12. Перемикачі ламп — HashSet

Кожен номер у послідовності натискань змінює стан відповідної лампи. Увімкнену вимикаємо, вимкнену вмикаємо. Спочатку всі вимкнені. Поверніть номери увімкнених ламп за зростанням.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

foreach (int[] presses in new[] { new[] { 1, 2, 1, 3, 2, 2 }, new[] { 5, 5 }, Array.Empty<int>() })
{
    var on = new HashSet<int>();
    foreach (int lamp in presses)
        if (!on.Add(lamp)) on.Remove(lamp);
    var result = new List<int>(on);
    result.Sort();
    Console.WriteLine($"[{string.Join(", ", result)}]");
}
```

**Очікуваний вивід:**

```text
[2, 3]
[]
[]
```

</details>
