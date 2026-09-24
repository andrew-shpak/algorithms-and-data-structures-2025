# Лекція 2 — Прості завдання на списки, стек і чергу

[Матеріал лекції](Examples.md) · [Наступні вправи на колекції](../Lecture-3/tasks.md) · [Застосування стека й черги](../Lecture-6/tasks.md)

Кожне завдання незалежне. Почніть із готових колекцій C#: `List<T>`, `LinkedList<T>`, `Stack<T>`, `Queue<T>`. Дані можна задати прямо в програмі; меню та зчитування файлів не потрібні. Індекси починаються з нуля, `[]` означає порожню колекцію.

Кожне завдання містить повну програму: чотири початкові приклади відкриті, решта реалізацій — у розгортних блоках. Спочатку спробуйте розв’язати задачу самостійно, потім порівняйте з кодом.

| Колекція | Що тренуємо | Завдання |
|---|---|---|
| `List<T>` | Індекс, вставка, заміна, видалення | 1–2 |
| `LinkedList<T>` | Вузол, сусіди, обидва кінці | 3–4 |
| `Stack<T>` | LIFO: останній доданий виходить першим | 5–6 |
| `Queue<T>` | FIFO: перший доданий виходить першим | 7–8 |

> **Запуск реалізацій:** C# 14 / .NET 10. Встановіть .NET 10 SDK. Створіть консольний проєкт через `dotnet new console --framework net10.0`, замініть `Program.cs` одним повним блоком `csharp` і виконайте `dotnet run`. Кожен блок незалежний; класи й методи з інших завдань копіювати не потрібно. Для порожніх результатів у виводі використовуємо `[]`; логічні значення C# друкує як `True` / `False`.

## Завдання 1. Кольори в пеналі — `List<string>`

Є список `["синій", "зелений"]`. Додайте `"жовтий"` у кінець, вставте `"червоний"` за індексом `1`, замініть перший колір на `"чорний"` і видаліть `"зелений"` за значенням. Виведіть результат і кількість елементів.

**Розібраний приклад:**

```csharp
using System;
using System.Collections.Generic;

var colors = new List<string> { "синій", "зелений" };
colors.Add("жовтий");
colors.Insert(1, "червоний");
colors[0] = "чорний";
colors.Remove("зелений");

Console.WriteLine(string.Join(", ", colors));
Console.WriteLine($"Кількість: {colors.Count}");
```

**Очікуваний вивід:**

```text
чорний, червоний, жовтий
Кількість: 3
```

**Самостійно:** перед видаленням ще раз додайте `"зелений"`. Перевірте, що `Remove` видаляє лише перший збіг: результат — `["чорний", "червоний", "жовтий", "зелений"]`. Видалення відсутнього кольору має залишати список без змін.

## Завдання 2. Прибрати порожні підписи — `List<string>`

Видаліть усі елементи, які дорівнюють `""`. Збережіть порядок решти. Рядок із пробілом `" "` у цій задачі не вважається порожнім.

**Вимоги:** змінюйте початковий список через `RemoveAt`, рухаючись індексами від кінця до початку. Не використовуйте `RemoveAll` або LINQ.

| Початковий список | Результат |
|---|---|
| `["літо", "", "", "осінь", ""]` | `["літо", "осінь"]` |
| `["", ""]` | `[]` |
| `[" ", "зима"]` | `[" ", "зима"]` |
| `[]` | `[]` |

**Питання:** який елемент можна пропустити, якщо після видалення рухатися вперед і збільшувати індекс?

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

string[][] examples =
{
    new[] { "літо", "", "", "осінь", "" },
    new[] { "", "" },
    new[] { " ", "зима" },
    Array.Empty<string>()
};
foreach (string[] example in examples)
{
    var labels = new List<string>(example);
    for (int i = labels.Count - 1; i >= 0; i--)
    {
        if (labels[i] == "")
        {
            labels.RemoveAt(i);
        }
    }
    Console.WriteLine($"[{string.Join(", ", labels)}]");
}
```

**Очікуваний вивід:**

```text
[літо, осінь]
[]
[ , зима]
[]
```

</details>

## Завдання 3. Додати зупинку — `LinkedList<string>`

Побудуйте маршрут `Дім → Парк → Школа`. Знайдіть вузол `Парк` і вставте після нього `Музей`. Потім видаліть першу зупинку.

**Розібраний приклад:**

```csharp
using System;
using System.Collections.Generic;

var route = new LinkedList<string>(new[] { "Дім", "Парк", "Школа" });
var park = route.Find("Парк");
if (park is not null)
{
    route.AddAfter(park, "Музей");
}
if (route.Count > 0)
{
    route.RemoveFirst();
}

Console.WriteLine(string.Join(" -> ", route));
```

**Очікуваний вивід:**

```text
Парк -> Музей -> Школа
```

**Самостійно:** повторіть із маршрутом `["Дім", "Школа"]`. Якщо `Парк` відсутній, пропустіть вставку; після видалення першої зупинки має залишитися `["Школа"]`. Для порожнього маршруту результат — `[]`.

**Зверніть увагу:** `Find` шукає вузол за O(n), а `AddAfter` за вже відомим вузлом виконується за O(1).

## Завдання 4. Стрічка з двох кінців — `LinkedList<int>`

Почніть із порожнього списку. Команда `left x` додає число на початок, `right x` — у кінець. Після всіх команд виведіть стрічку від `First` до `Last`, переходячи через `Next`.

**Вимоги:** використайте `AddFirst` і `AddLast`; не звертайтеся до списку за індексом.

| Команди | Результат |
|---|---|
| `right 4; left 2; right 7; left 1` | `[1, 2, 4, 7]` |
| `left 5; left 5` | `[5, 5]` |
| Немає команд | `[]` |

**Самостійно:** виведіть ту саму стрічку від `Last` через `Previous`. Для першого прикладу: `[7, 4, 2, 1]`.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

(string Side, int Value)[][] examples =
{
    new[] { ("right", 4), ("left", 2), ("right", 7), ("left", 1) },
    new[] { ("left", 5), ("left", 5) },
    Array.Empty<(string, int)>()
};
foreach (var commands in examples)
{
    var ribbon = new LinkedList<int>();
    foreach (var command in commands)
    {
        if (command.Side == "left")
        {
            ribbon.AddFirst(command.Value);
        }
        else
        {
            ribbon.AddLast(command.Value);
        }
    }

    var forward = new List<int>();
    for (var node = ribbon.First; node is not null; node = node.Next)
    {
        forward.Add(node.Value);
    }
    var backward = new List<int>();
    for (var node = ribbon.Last; node is not null; node = node.Previous)
    {
        backward.Add(node.Value);
    }
    Console.WriteLine($"[{string.Join(", ", forward)}] / [{string.Join(", ", backward)}]");
}
```

**Очікуваний вивід:**

```text
[1, 2, 4, 7] / [7, 4, 2, 1]
[5, 5] / [5, 5]
[] / []
```

</details>

## Завдання 5. Стос тарілок — `Stack<string>`

Покладіть у стек синю, білу й червону тарілки саме в такому порядку. Подивіться на верхню тарілку через `Peek`, а потім зніміть усі через `Pop`.

**Розібраний приклад:**

```csharp
using System;
using System.Collections.Generic;

var plates = new Stack<string>();
plates.Push("синя");
plates.Push("біла");
plates.Push("червона");

if (plates.Count > 0)
{
    Console.WriteLine($"Зверху: {plates.Peek()}");
}
Console.WriteLine($"До зняття: {plates.Count}");
while (plates.Count > 0)
{
    Console.WriteLine($"Зняли: {plates.Pop()}");
}
Console.WriteLine($"Залишилось: {plates.Count}");
```

**Очікуваний вивід:**

```text
Зверху: червона
До зняття: 3
Зняли: червона
Зняли: біла
Зняли: синя
Залишилось: 0
```

**Самостійно:** запустіть без додавання тарілок. Вивід: `До зняття: 0`, потім `Залишилось: 0`. Поясніть, чому `Peek` не зменшує `Count`.

## Завдання 6. Слово навпаки — `Stack<char>`

Додайте символи слова у стек зліва направо. Знімайте їх по одному й будуйте новий рядок. Для вправи використовуйте лише малі латинські літери; `Reverse` не застосовуйте.

| Слово | Результат |
|---|---|
| `code` | `edoc` |
| `level` | `level` |
| `a` | `a` |
| `""` | `""` |

**Перевірка:** після побудови результату стек має бути порожнім. Перед `Pop` перевіряйте `Count` або використовуйте `TryPop`.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

foreach (string word in new[] { "code", "level", "a", "" })
{
    var letters = new Stack<char>();
    foreach (char letter in word)
    {
        letters.Push(letter);
    }
    string result = "";
    while (letters.TryPop(out char letter))
    {
        result += letter;
    }
    Console.WriteLine($"[{result}]; Count={letters.Count}");
}
```

**Очікуваний вивід:**

```text
[edoc]; Count=0
[level]; Count=0
[a]; Count=0
[]; Count=0
```

</details>

## Завдання 7. Черга до сканера — `Queue<string>`

До сканера надійшли `лист-A` і `лист-B`. Обробіть один лист, додайте `лист-C`, подивіться, хто наступний, і обробіть решту.

**Розібраний приклад:**

```csharp
using System;
using System.Collections.Generic;

var scans = new Queue<string>();
scans.Enqueue("лист-A");
scans.Enqueue("лист-B");
Console.WriteLine($"Скануємо: {scans.Dequeue()}");
scans.Enqueue("лист-C");
Console.WriteLine($"Наступний: {scans.Peek()}");
while (scans.Count > 0)
{
    Console.WriteLine($"Скануємо: {scans.Dequeue()}");
}
Console.WriteLine($"Залишилось: {scans.Count}");
```

**Очікуваний вивід:**

```text
Скануємо: лист-A
Наступний: лист-B
Скануємо: лист-B
Скануємо: лист-C
Залишилось: 0
```

**Самостійно:** після спорожнення спробуйте `TryDequeue`. Якщо він повертає `false`, виведіть `Черга порожня`. Не викликайте `Dequeue` або `Peek` для порожньої черги.

## Завдання 8. Видати не більше k наліпок — `Queue<int>`

Черга містить номери наліпок у порядку видачі. Видайте перші `k` наліпок, але зупиніться, якщо черга закінчилася. Поверніть список виданих номерів і покажіть залишок черги. `k ≥ 0`.

| Черга від голови до хвоста; k | Видано | Залишок |
|---|---|---|
| `[11, 12, 13, 14]; 2` | `[11, 12]` | `[13, 14]` |
| `[11, 12]; 5` | `[11, 12]` | `[]` |
| `[11, 12]; 0` | `[]` | `[11, 12]` |
| `[]; 3` | `[]` | `[]` |

**Вимоги:** використайте `Dequeue` з перевіркою кількості або `TryDequeue`. Не замінюйте чергу списком із `RemoveAt(0)`.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

(int[] Numbers, int K)[] examples =
{
    (new[] { 11, 12, 13, 14 }, 2),
    (new[] { 11, 12 }, 5),
    (new[] { 11, 12 }, 0),
    (Array.Empty<int>(), 3)
};
foreach (var example in examples)
{
    var queue = new Queue<int>(example.Numbers);
    List<int> issued = TakeStickers(queue, example.K);
    Console.WriteLine($"Видано: [{string.Join(", ", issued)}]; залишок: [{string.Join(", ", queue)}]");
}

static List<int> TakeStickers(Queue<int> queue, int k)
{
    var issued = new List<int>();
    while (issued.Count < k && queue.TryDequeue(out int number))
    {
        issued.Add(number);
    }
    return issued;
}
```

**Очікуваний вивід:**

```text
Видано: [11, 12]; залишок: [13, 14]
Видано: [11, 12]; залишок: []
Видано: []; залишок: [11, 12]
Видано: []; залишок: []
```

</details>
