# Лекція 7 — Прості вправи на будову хеш-таблиці

[Матеріал лекції](Examples.md)

Працюємо з маленькими таблицями на C#. Спочатку намалюйте комірки на папері, потім відтворіть кроки програмою. Усі ключі — невід'ємні цілі числа; хеш-функція в цих вправах задана явно.

> **Запуск реалізацій:** C# 14 / .NET 10. Встановіть .NET 10 SDK. Створіть консольний проєкт через `dotnet new console --framework net10.0`, замініть `Program.cs` одним повним блоком `csharp` і виконайте `dotnet run`. Кожен блок незалежний; класи й методи з інших завдань копіювати не потрібно. Для порожніх результатів у виводі використовуємо `[]`; логічні значення C# друкує як `True` / `False`.

## Завдання 1. Ключі по кошиках

Для таблиці з `m > 0` кошиків використайте `h(key) = key % m`. Зберігайте колізії в окремих `List<int>`; новий ключ додавайте в кінець відповідного кошика. Усі вхідні ключі різні.

**Вимоги:** виведіть кошики за індексами `0..m-1`, включно з порожніми. Готовий `Dictionary` не використовуйте.

| Ключі; m | Кошики після вставок |
|---|---|
| `[2, 7, 12, 4]; 5` | `0: []; 1: []; 2: [2, 7, 12]; 3: []; 4: [4]` |
| `[0, 1, 2]; 3` | `0: [0]; 1: [1]; 2: [2]` |
| `[]; 2` | `0: []; 1: []` |

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

PrintBuckets(new[] { 2, 7, 12, 4 }, 5);
PrintBuckets(new[] { 0, 1, 2 }, 3);
PrintBuckets(Array.Empty<int>(), 2);

static void PrintBuckets(int[] keys, int m)
{
    var buckets = new List<int>[m];
    for (int i = 0; i < m; i++)
    {
        buckets[i] = new List<int>();
    }
    foreach (int key in keys)
    {
        buckets[key % m].Add(key);
    }
    var descriptions = new List<string>();
    for (int i = 0; i < m; i++)
    {
        descriptions.Add($"{i}: [{string.Join(", ", buckets[i])}]");
    }
    Console.WriteLine(string.Join("; ", descriptions));
}
```

**Очікуваний вивід:**

```text
0: []; 1: []; 2: [2, 7, 12]; 3: []; 4: [4]
0: [0]; 1: [1]; 2: [2]
0: []; 1: []
```

</details>

## Завдання 2. Знайти вільну комірку

У таблиці з п'яти комірок вставляйте різні ключі методом лінійного пробування. Почніть із `key % 5`, а зайняті комірки оминайте кроком `(index + 1) % 5`. `null` позначає вільну комірку.

**Вимоги:** для кожного ключа поверніть індекс вставки або `FULL`, якщо всі комірки зайняті. Перевіряйте не більше п'яти комірок за одну вставку.

| Послідовність вставок у порожню таблицю | Індекси або FULL |
|---|---|
| `[4, 9, 14]` | `[4, 0, 1]` |
| `[0, 5, 10, 15, 20, 25]` | `[0, 1, 2, 3, 4, FULL]` |

Для першого прикладу кінцевий масив: `[9, 14, null, null, 4]`. Перевірте, що ключ `0` не плутається з порожньою коміркою.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

foreach (int[] keys in new[] { new[] { 4, 9, 14 }, new[] { 0, 5, 10, 15, 20, 25 } })
{
    int?[] table = new int?[5];
    var positions = new List<string>();
    foreach (int key in keys)
    {
        int index = Insert(table, key);
        positions.Add(index == -1 ? "FULL" : index.ToString());
    }
    Console.WriteLine(string.Join(", ", positions));
    var cells = new List<string>();
    foreach (int? value in table)
    {
        cells.Add(value?.ToString() ?? "null");
    }
    Console.WriteLine($"[{string.Join(", ", cells)}]");
}

static int Insert(int?[] table, int key)
{
    int index = key % table.Length;
    for (int checkedCells = 0; checkedCells < table.Length; checkedCells++)
    {
        if (table[index] is null)
        {
            table[index] = key;
            return index;
        }
        index = (index + 1) % table.Length;
    }
    return -1;
}
```

**Очікуваний вивід:**

```text
4, 0, 1
[9, 14, null, null, 4]
0, 1, 2, 3, 4, FULL
[0, 5, 10, 15, 20]
```

</details>

## Завдання 3. Пошук через видалену комірку

Дано таблицю лінійного пробування розміру `5`: `[10, DELETED, 20, EMPTY, EMPTY]`. Хеш-функція — `key % 5`. `DELETED` означає, що тут раніше був ключ; `EMPTY` — що вставок сюди ще не було.

**Вимоги:** напишіть пошук, який проходить крізь `DELETED`, але зупиняється на `EMPTY`. Якщо збігу немає після п'яти перевірок, поверніть `-1`. Стани зберігайте окремо від значень ключів.

| Шуканий ключ | Індекс | Перевірені комірки |
|---|---|---|
| `10` | `0` | `0` |
| `20` | `2` | `0, 1, 2` |
| `15` | `-1` | `0, 1, 2, 3` |

**Самостійно:** для таблиці лише з `DELETED` пошук будь-якого ключа має завершитися після п'яти перевірок, а не зациклитися.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

int[] values = { 10, 0, 20, 0, 0 };
CellState[] states = { CellState.Occupied, CellState.Deleted, CellState.Occupied, CellState.Empty, CellState.Empty };
foreach (int key in new[] { 10, 20, 15 })
{
    var checkedCells = new List<int>();
    int index = Search(values, states, key, checkedCells);
    Console.WriteLine($"{key}: {index}; checked=[{string.Join(", ", checkedCells)}]");
}
Array.Fill(states, CellState.Deleted);
var allDeleted = new List<int>();
Console.WriteLine($"DELETED: {Search(values, states, 15, allDeleted)}; checked={allDeleted.Count}");

static int Search(int[] values, CellState[] states, int key, List<int> checkedCells)
{
    int index = key % values.Length;
    for (int attempt = 0; attempt < values.Length; attempt++)
    {
        checkedCells.Add(index);
        if (states[index] == CellState.Empty)
        {
            return -1;
        }
        if (states[index] == CellState.Occupied && values[index] == key)
        {
            return index;
        }
        index = (index + 1) % values.Length;
    }
    return -1;
}

public enum CellState { Empty, Occupied, Deleted }
```

**Очікуваний вивід:**

```text
10: 0; checked=[0]
20: 2; checked=[0, 1, 2]
15: -1; checked=[0, 1, 2, 3]
DELETED: -1; checked=5
```

</details>

## Завдання 4. Нові адреси після збільшення таблиці

Ключі `[6, 11, 16]` зберігалися в таблиці розміру `5`. Збільште кількість кошиків до `7`: обчисліть **новий** індекс кожного ключа та заново розкладіть ключі методом ланцюжків.

**Вимоги:** обійдіть старі кошики та перерахуйте `key % 7`. Простого копіювання старих кошиків недостатньо.

| Ключ | Старий індекс (% 5) | Новий індекс (% 7) |
|---|---|---|
| `6` | `1` | `6` |
| `11` | `1` | `4` |
| `16` | `1` | `2` |

Очікувані нові кошики: `0: []; 1: []; 2: [16]; 3: []; 4: [11]; 5: []; 6: [6]`. Для порожньої старої таблиці всі сім нових кошиків мають бути порожніми.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

var oldBuckets = new List<int>[5];
for (int i = 0; i < oldBuckets.Length; i++)
{
    oldBuckets[i] = new List<int>();
}
foreach (int key in new[] { 6, 11, 16 })
{
    oldBuckets[key % oldBuckets.Length].Add(key);
}
Print(Rehash(oldBuckets, 7));
foreach (var bucket in oldBuckets)
{
    bucket.Clear();
}
Print(Rehash(oldBuckets, 7));

static List<int>[] Rehash(List<int>[] oldBuckets, int newSize)
{
    var result = new List<int>[newSize];
    for (int i = 0; i < newSize; i++)
    {
        result[i] = new List<int>();
    }
    foreach (var bucket in oldBuckets)
    {
        foreach (int key in bucket)
        {
            result[key % newSize].Add(key);
        }
    }
    return result;
}

static void Print(List<int>[] buckets)
{
    var descriptions = new List<string>();
    for (int i = 0; i < buckets.Length; i++)
    {
        descriptions.Add($"{i}: [{string.Join(", ", buckets[i])}]");
    }
    Console.WriteLine(string.Join("; ", descriptions));
}
```

**Очікуваний вивід:**

```text
0: []; 1: []; 2: [16]; 3: []; 4: [11]; 5: []; 6: [6]
0: []; 1: []; 2: []; 3: []; 4: []; 5: []; 6: []
```

</details>
