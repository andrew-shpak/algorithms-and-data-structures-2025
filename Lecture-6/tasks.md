# Лекція 6 — Невеликі задачі на стек і чергу

[Матеріал лекції](Examples.md) · [Перші кроки з Push, Pop, Enqueue, Dequeue](../Lecture-2/tasks.md)

Кожну задачу виконуйте окремо на C#. Дані задавайте в коді. Тут практикуємо обробку послідовностей; для всіх задач достатньо звичайних `Stack<T>` і `Queue<T>`. Два приклади відкриті, решта реалізацій — у розгортних блоках.

**Усього: 12 завдань із повними реалізаціями та очікуваним виводом.**

> **Запуск реалізацій:** C# 14 / .NET 10. Встановіть .NET 10 SDK. Створіть консольний проєкт через `dotnet new console --framework net10.0`, замініть `Program.cs` одним повним блоком `csharp` і виконайте `dotnet run`. Кожен блок незалежний; класи й методи з інших завдань копіювати не потрібно. Для порожніх результатів у виводі використовуємо `[]`; логічні значення C# друкує як `True` / `False`.

## Завдання 1. Пари однакових жетонів — `Stack<char>`

Жетони — великі латинські літери. Читайте рядок зліва направо. Якщо новий жетон збігається з вершиною стека, приберіть вершину й не додавайте новий жетон: пара зникає. Інакше покладіть новий жетон у стек. Виведіть залишок у початковому напрямку читання.

| Вхід | Результат |
|---|---|
| `ABBA` | `""` |
| `ABBAC` | `C` |
| `ABCA` | `ABCA` |
| `AAA` | `A` |
| `""` | `""` |

**Трасування для `ABBAC`** (стек від дна до вершини): `[] → [A] → [A, B] → [A] → [] → [C]`.

**Розібраний приклад:**

```csharp
using System;
using System.Collections.Generic;

string input = "ABBAC";
var tokens = new Stack<char>();
foreach (char token in input)
{
    if (tokens.TryPeek(out char top) && top == token)
    {
        tokens.Pop();
    }
    else
    {
        tokens.Push(token);
    }
}

char[] result = tokens.ToArray(); // Від вершини до дна.
Array.Reverse(result);           // Повертаємо порядок читання.
Console.WriteLine(new string(result));
```

**Очікуваний вивід:**

```text
C
```

**Самостійно:** після кожного символу виводьте `Count`. Для `ABBAC` очікуємо `1, 2, 1, 0, 1`. Поясніть, чому дві літери `A` у `ABCA` не зникають.

## Завдання 2. Двійковий запис числа — `Stack<int>`

Для невід'ємного цілого `n` знаходьте остачу від ділення на `2`, кладіть її в стек і діліть `n` на `2` націло. Зніміть цифри зі стека, щоб отримати двійковий запис від старшої цифри до молодшої.

**Вимоги:** обробіть `0` окремо. Не використовуйте готове перетворення числа в іншу систему числення.

| n | Результат |
|---|---|
| `13` | `1101` |
| `8` | `1000` |
| `1` | `1` |
| `0` | `0` |

**Перевірка вручну:** для `13` остачі надходять як `1, 0, 1, 1`, а виходять як `1, 1, 0, 1`.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

foreach (int number in new[] { 13, 8, 1, 0 })
{
    Console.WriteLine(ToBinary(number));
}

static string ToBinary(int n)
{
    if (n == 0)
    {
        return "0";
    }
    var bits = new Stack<int>();
    while (n > 0)
    {
        bits.Push(n % 2);
        n /= 2;
    }
    string result = "";
    while (bits.TryPop(out int bit))
    {
        result += bit;
    }
    return result;
}
```

**Очікуваний вивід:**

```text
1101
1000
1
0
```

</details>

## Завдання 3. Пакування листів пачками — `Queue<string>`

У черзі лежать листи в порядку надходження. Розкладіть **всі** листи в пачки по `size` штук. Остання пачка може бути неповною; порожніх пачок не створюйте. `size ≥ 1`.

| Черга; size | Пачки |
|---|---|
| `[A, B, C, D, E]; 2` | `[A, B]`, `[C, D]`, `[E]` |
| `[A, B]; 1` | `[A]`, `[B]` |
| `[A, B]; 5` | `[A, B]` |
| `[]; 2` | Немає пачок |

**Розібраний приклад:**

```csharp
using System;
using System.Collections.Generic;

var letters = new Queue<string>(new[] { "A", "B", "C", "D", "E" });
int size = 2;
while (letters.Count > 0)
{
    var batch = new List<string>();
    while (batch.Count < size && letters.Count > 0)
    {
        batch.Add(letters.Dequeue());
    }
    Console.WriteLine(string.Join(" ", batch));
}
```

**Очікуваний вивід:**

```text
A B
C D
E
```

**Самостійно:** додайте нумерацію пачок від `1` та кількість листів у кожній: для прикладу це `2, 2, 1`. Після пакування черга має бути порожня.

## Завдання 4. Ходи в настільній грі — `Queue<string>`

Гравці стоять у черзі. За один хід візьміть гравця з голови, запишіть його ім'я й поверніть у хвіст. Виконайте рівно `turns` ходів (`turns ≥ 0`). Порожня черга означає, що ходів немає.

**Вимоги:** застосовуйте `Dequeue` та `Enqueue`. Обмежуйте кількість ітерацій числом ходів: цикл лише з умовою `Count > 0` тут не завершиться.

| Початкова черга; turns | Ходи | Черга після гри |
|---|---|---|
| `[Аня, Богдан, Віра]; 5` | `[Аня, Богдан, Віра, Аня, Богдан]` | `[Віра, Аня, Богдан]` |
| `[Аня]; 3` | `[Аня, Аня, Аня]` | `[Аня]` |
| `[Аня, Віра]; 0` | `[]` | `[Аня, Віра]` |
| `[]; 4` | `[]` | `[]` |

**Самостійно:** переконайтеся, що кількість гравців після гри не змінилася, навіть якщо ходів було більше, ніж гравців.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

(string[] Players, int Turns)[] examples =
{
    (new[] { "Аня", "Богдан", "Віра" }, 5),
    (new[] { "Аня" }, 3),
    (new[] { "Аня", "Віра" }, 0),
    (Array.Empty<string>(), 4)
};
foreach (var example in examples)
{
    var players = new Queue<string>(example.Players);
    var moves = new List<string>();
    for (int turn = 0; turn < example.Turns && players.Count > 0; turn++)
    {
        string player = players.Dequeue();
        moves.Add(player);
        players.Enqueue(player);
    }
    Console.WriteLine($"Ходи: [{string.Join(", ", moves)}]; черга: [{string.Join(", ", players)}]");
}
```

**Очікуваний вивід:**

```text
Ходи: [Аня, Богдан, Віра, Аня, Богдан]; черга: [Віра, Аня, Богдан]
Ходи: [Аня, Аня, Аня]; черга: [Аня]
Ходи: []; черга: [Аня, Віра]
Ходи: []; черга: []
```

</details>

## Завдання 5. Клавіша стирання — Stack

Символ `#` стирає останню введену малу латинську літеру. Якщо стирати нічого, пропустіть команду. Побудуйте остаточний текст через стек; перед виведенням відновіть напрямок читання.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

foreach (string input in new[] { "ab#c", "##a", "abc###", "" })
{
    var letters = new Stack<char>();
    foreach (char symbol in input)
    {
        if (symbol == '#') letters.TryPop(out _);
        else letters.Push(symbol);
    }
    char[] result = letters.ToArray();
    Array.Reverse(result);
    Console.WriteLine($"[{new string(result)}]");
}
```

**Очікуваний вивід:**

```text
[ac]
[a]
[]
[]
```

</details>

## Завдання 6. Відкат доданих балів — Stack

Операція `Add(x)` додає бали до поточного підсумку та запам’ятовує попередній підсумок. `Undo()` відновлює його; якщо історія порожня, нічого не змінює. Після кожної операції виведіть підсумок. Для прикладу бали невеликі.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

int total = 0;
var history = new Stack<int>();
Add(5);
Add(3);
Undo();
Undo();
Undo();

void Add(int points)
{
    history.Push(total);
    total += points;
    Console.WriteLine(total);
}

void Undo()
{
    if (history.TryPop(out int previous)) total = previous;
    Console.WriteLine(total);
}
```

**Очікуваний вивід:**

```text
5
8
5
0
0
```

</details>

## Завдання 7. Постфіксні дії без дужок — Stack

Обчисліть коректний постфіксний вираз із цілих чисел та операторів `+`, `-`, `*`, розділених пробілами. Для оператора спочатку зніміть правий операнд, потім лівий. Гарантується один підсумковий результат і відсутність переповнення.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

foreach (string expression in new[] { "2 3 + 4 *", "8 3 -", "7" })
    Console.WriteLine(Evaluate(expression));

static int Evaluate(string expression)
{
    var values = new Stack<int>();
    foreach (string token in expression.Split(' ', StringSplitOptions.RemoveEmptyEntries))
    {
        if (int.TryParse(token, out int number)) values.Push(number);
        else
        {
            int right = values.Pop();
            int left = values.Pop();
            values.Push(token switch
            {
                "+" => left + right,
                "-" => left - right,
                "*" => left * right,
                _ => throw new ArgumentException("Невідомий оператор")
            });
        }
    }
    return values.Pop();
}
```

**Очікуваний вивід:**

```text
20
5
7
```

</details>

## Завдання 8. Прибрати пошкоджені елементи зі стека

Видаліть зі стека всі елементи, рівні `bad`, зберігши порядок інших від вершини до дна. Використайте один допоміжний стек. Вхідний масив задає значення від дна до вершини.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

foreach (int[] values in new[] { new[] { 1, 9, 2, 9, 3 }, new[] { 9, 9 }, Array.Empty<int>() })
{
    var stack = new Stack<int>(values);
    RemoveBad(stack, 9);
    Console.WriteLine($"[{string.Join(", ", stack)}]");
}

static void RemoveBad(Stack<int> stack, int bad)
{
    var buffer = new Stack<int>();
    while (stack.TryPop(out int value))
        if (value != bad) buffer.Push(value);
    while (buffer.TryPop(out int value)) stack.Push(value);
}
```

**Очікуваний вивід:**

```text
[3, 2, 1]
[]
[]
```

</details>

## Завдання 9. Одна сторінка за чергу

Черга містить документи з додатною кількістю сторінок. За хід надрукуйте одну сторінку першого документа. Якщо сторінки залишилися, поверніть документ у хвіст; завершений документ більше не додавайте. Виведіть назву на кожному ході.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

var jobs = new Queue<(string Name, int Pages)>();
jobs.Enqueue(("A", 2));
jobs.Enqueue(("B", 1));
jobs.Enqueue(("C", 2));
while (jobs.TryDequeue(out var job))
{
    Console.WriteLine(job.Name);
    if (job.Pages > 1) jobs.Enqueue((job.Name, job.Pages - 1));
}
Console.WriteLine($"Залишок: {jobs.Count}");
```

**Очікуваний вивід:**

```text
A
B
C
A
C
Залишок: 0
```

</details>

## Завдання 10. Дві смуги обслуговування

Почергово обслуговуйте одну термінову та одну звичайну заявку, починаючи з термінової. Якщо одна черга порожня, беріть з іншої. Усередині кожної черги зберігайте FIFO; термінові заявки не можуть безмежно витісняти звичайні.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

Console.WriteLine(string.Join(", ", Serve(new[] { "U1", "U2", "U3" }, new[] { "N1", "N2" })));
Console.WriteLine(string.Join(", ", Serve(Array.Empty<string>(), new[] { "N1", "N2" })));
Console.WriteLine(Serve(Array.Empty<string>(), Array.Empty<string>()).Count);

static List<string> Serve(string[] urgentItems, string[] normalItems)
{
    var urgent = new Queue<string>(urgentItems);
    var normal = new Queue<string>(normalItems);
    var result = new List<string>();
    while (urgent.Count > 0 || normal.Count > 0)
    {
        if (urgent.TryDequeue(out string? item)) result.Add(item);
        if (normal.TryDequeue(out item)) result.Add(item);
    }
    return result;
}
```

**Очікуваний вивід:**

```text
U1, N1, U2, N2, U3
N1, N2
0
```

</details>

## Завдання 11. Три останні події — дек

Зберігайте лише три останні події через `LinkedList<string>` як двобічну чергу. Нову додавайте в кінець, найстарішу за потреби видаляйте з початку. Після кожного додавання покажіть поточну історію від старої до нової.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

var history = new LinkedList<string>();
foreach (string item in new[] { "A", "B", "C", "D", "E" })
{
    history.AddLast(item);
    if (history.Count > 3) history.RemoveFirst();
    Console.WriteLine(string.Join(", ", history));
}
```

**Очікуваний вивід:**

```text
A
A, B
A, B, C
B, C, D
C, D, E
```

</details>

## Завдання 12. Чи можливий порядок вивантаження

Елементи з різними номерами надходять у заданому порядку. Кожен можна покласти у стек, а знімати дозволено лише вершину. Перевірте, чи можна отримати задану перестановку на виході. Обидва масиви містять ті самі номери без повторів.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

Console.WriteLine(CanUnload(new[] { 1, 2, 3 }, new[] { 2, 1, 3 }));
Console.WriteLine(CanUnload(new[] { 1, 2, 3 }, new[] { 3, 1, 2 }));
Console.WriteLine(CanUnload(Array.Empty<int>(), Array.Empty<int>()));

static bool CanUnload(int[] incoming, int[] outgoing)
{
    var stack = new Stack<int>();
    int next = 0;
    foreach (int value in incoming)
    {
        stack.Push(value);
        while (next < outgoing.Length && stack.TryPeek(out int top) && top == outgoing[next])
        {
            stack.Pop();
            next++;
        }
    }
    return next == outgoing.Length;
}
```

**Очікуваний вивід:**

```text
True
False
True
```

</details>
