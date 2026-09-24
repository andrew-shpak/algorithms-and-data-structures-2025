# Лекція 2 — Прості завдання на списки, стек і чергу

[Матеріал лекції](Examples.md) · [Наступні вправи на колекції](../Lecture-3/tasks.md) · [Застосування стека й черги](../Lecture-6/tasks.md)

Кожне завдання незалежне. Почніть із готових колекцій C#: `List<T>`, `LinkedList<T>`, `Stack<T>`, `Queue<T>`. Дані можна задати прямо в програмі; меню та зчитування файлів не потрібні. Індекси починаються з нуля, `[]` означає порожню колекцію.

Кожне завдання містить повну програму: чотири початкові приклади відкриті, решта реалізацій — у розгортних блоках. Спочатку спробуйте розв’язати задачу самостійно, потім порівняйте з кодом.

| Колекція | Що тренуємо | Завдання |
|---|---|---|
| `List<T>` | Індекс, вставка, заміна, видалення | 1–2, 9–12 |
| `LinkedList<T>` | Вузол, сусіди, обидва кінці | 3–4, 13–16 |
| `Stack<T>` | LIFO: останній доданий виходить першим | 5–6, 17–20 |
| `Queue<T>` | FIFO: перший доданий виходить першим | 7–8, 21–25 |
| `ConcurrentQueue<T>` | Спільна FIFO-черга для кількох завдань | 26–27 |
| `ConcurrentStack<T>` | LIFO та пакетне вилучення | 28 |
| `ConcurrentBag<T>` | Повтори, невизначений порядок, спільні результати | 29–30 |

**Усього: 30 завдань із повними реалізаціями та очікуваним виводом.**

**Звичайні й конкурентні версії.** `Queue<T>` та `Stack<T>` підходять для послідовної роботи. Для спільного додавання й вилучення з кількох потоків є `ConcurrentQueue<T>` і `ConcurrentStack<T>` у просторі імен `System.Collections.Concurrent`. [Потокобезпечні колекції .NET](https://learn.microsoft.com/en-us/dotnet/standard/collections/thread-safe/).

`ConcurrentBag<T>` зберігає повтори, але не надає індексів і не гарантує порядок. Це окрема неупорядкована колекція; вона не замінює список, якщо потрібні позиції елементів. [Опис ConcurrentBag](https://learn.microsoft.com/en-us/dotnet/api/system.collections.concurrent.concurrentbag-1?view=net-10.0).

У нових прикладах `Task.Run` запускає роботу, а `await Task.WhenAll` очікує її завершення. Для вилучення одразу викликайте `TryDequeue`, `TryPop` чи `TryTake`: перевірка `Count` або `IsEmpty` перед окремою дією не резервує елемент. Якщо виробник ще додає дані, порожня колекція не означає кінець роботи — приклад із явним завершенням є в [Лекції 3](../Lecture-3/tasks.md#завдання-18-виробник-ще-працює--blockingcollection-над-concurrentqueue).

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

## Завдання 9. Замінити всі маркери — List

У списку чисел замініть кожен маркер `-1` на `0`, залишивши інші числа на своїх місцях. Змінюйте початковий список через індексатор. Перевірте повторні маркери та порожній список.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

foreach (int[] source in new[] { new[] { -1, 4, -1, 8 }, new[] { 2, 3 }, Array.Empty<int>() })
{
    var values = new List<int>(source);
    for (int i = 0; i < values.Count; i++)
        if (values[i] == -1) values[i] = 0;
    Console.WriteLine($"[{string.Join(", ", values)}]");
}
```

**Очікуваний вивід:**

```text
[0, 4, 0, 8]
[2, 3]
[]
```

</details>

## Завдання 10. Копія фрагмента — List

Скопіюйте `count` елементів від індексу `start` через `GetRange`. Межі коректні: `0 ≤ start ≤ Count`, `0 ≤ count ≤ Count - start`. Зміна копії не повинна змінювати оригінал.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

var source = new List<int> { 10, 20, 30, 40 };
var part = source.GetRange(1, 2);
part[0] = 99;
Console.WriteLine($"Копія: [{string.Join(", ", part)}]");
Console.WriteLine($"Оригінал: [{string.Join(", ", source)}]");
Console.WriteLine($"Порожній фрагмент: {source.GetRange(source.Count, 0).Count}");
```

**Очікуваний вивід:**

```text
Копія: [99, 30]
Оригінал: [10, 20, 30, 40]
Порожній фрагмент: 0
```

</details>

## Завдання 11. Пересунути картку ліворуч — List

Поміняйте картку за індексом `index` з попередньою. Для першої картки, від’ємного індексу або індексу поза списком поверніть `false` без змін. Інакше поверніть `true`.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

var cards = new List<string> { "A", "B", "C" };
Console.WriteLine(MoveLeft(cards, 2));
Console.WriteLine(string.Join(", ", cards));
Console.WriteLine(MoveLeft(cards, 0));
Console.WriteLine(MoveLeft(new List<string>(), 0));

static bool MoveLeft(List<string> cards, int index)
{
    if (index <= 0 || index >= cards.Count) return false;
    (cards[index - 1], cards[index]) = (cards[index], cards[index - 1]);
    return true;
}
```

**Очікуваний вивід:**

```text
True
A, C, B
False
False
```

</details>

## Завдання 12. Позиції чисел, кратних трьом — List

Поверніть список індексів елементів, що діляться на `3` без остачі. Нуль і від’ємні кратні також враховуються. Самі значення не копіюйте в результат.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

foreach (int[] values in new[] { new[] { 5, 6, 0, -3, 8 }, new[] { 1, 2 }, Array.Empty<int>() })
{
    var positions = new List<int>();
    for (int i = 0; i < values.Length; i++)
        if (values[i] % 3 == 0) positions.Add(i);
    Console.WriteLine($"[{string.Join(", ", positions)}]");
}
```

**Очікуваний вивід:**

```text
[1, 2, 3]
[]
[]
```

</details>

## Завдання 13. Від’єднати хвіст — LinkedList

Перенесіть у новий список усі вузли, починаючи з першого вузла зі значенням `marker`. Збережіть їх порядок і самі об’єкти вузлів. Якщо маркера немає, новий список порожній. Перед від’єднанням запам’ятовуйте `Next`.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

foreach (int marker in new[] { 3, 9 })
{
    var source = new LinkedList<int>(new[] { 1, 2, 3, 4 });
    var tail = new LinkedList<int>();
    var node = source.Find(marker);
    while (node is not null)
    {
        var next = node.Next;
        source.Remove(node);
        tail.AddLast(node);
        node = next;
    }
    Console.WriteLine($"[{string.Join(", ", source)}] / [{string.Join(", ", tail)}]");
}
```

**Очікуваний вивід:**

```text
[1, 2] / [3, 4]
[1, 2, 3, 4] / []
```

</details>

## Завдання 14. З’єднати ланцюжки без нових вузлів — LinkedList

Перенесіть усі вузли другого списку в кінець першого. Порядок збережіть; другий список після операції має бути порожнім. Списки — різні об’єкти.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

var first = new LinkedList<int>(new[] { 1, 2 });
var second = new LinkedList<int>(new[] { 3, 4 });
AppendNodes(first, second);
Console.WriteLine(string.Join(", ", first));
Console.WriteLine($"Другий: {second.Count}");
AppendNodes(first, second); // Порожній список нічого не додає.
Console.WriteLine($"Перший: {first.Count}");

static void AppendNodes(LinkedList<int> first, LinkedList<int> second)
{
    while (second.First is not null)
    {
        var node = second.First;
        second.Remove(node);
        first.AddLast(node);
    }
}
```

**Очікуваний вивід:**

```text
1, 2, 3, 4
Другий: 0
Перший: 4
```

</details>

## Завдання 15. Курсор по колу — LinkedList

Виведіть рівно `steps ≥ 0` значень, почавши з першого вузла. Після останнього повертайтеся до першого. Порожній список не дає жодного значення; обмежуйте цикл кількістю кроків.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

Console.WriteLine($"[{string.Join(", ", Walk(new[] { "A", "B", "C" }, 5))}]");
Console.WriteLine($"[{string.Join(", ", Walk(new[] { "X" }, 3))}]");
Console.WriteLine($"[{string.Join(", ", Walk(Array.Empty<string>(), 4))}]");

static List<string> Walk(string[] values, int steps)
{
    var ring = new LinkedList<string>(values);
    var result = new List<string>();
    var current = ring.First;
    for (int i = 0; i < steps && current is not null; i++)
    {
        result.Add(current.Value);
        current = current.Next ?? ring.First;
    }
    return result;
}
```

**Очікуваний вивід:**

```text
[A, B, C, A, B]
[X, X, X]
[]
```

</details>

## Завдання 16. Прибрати кожну другу намистину — LinkedList

Видаліть вузли, які спочатку стояли на позиціях `2, 4, 6, ...` при нумерації від одиниці. Пересувайтеся через `Next`, без індексатора. Після видалення не втрачайте наступний вузол.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

foreach (int[] values in new[] { new[] { 1, 2, 3, 4, 5 }, new[] { 7 }, Array.Empty<int>() })
{
    var beads = new LinkedList<int>(values);
    var current = beads.First;
    bool remove = false;
    while (current is not null)
    {
        var next = current.Next;
        if (remove) beads.Remove(current);
        remove = !remove;
        current = next;
    }
    Console.WriteLine($"[{string.Join(", ", beads)}]");
}
```

**Очікуваний вивід:**

```text
[1, 3, 5]
[7]
[]
```

</details>

## Завдання 17. Прибрати завеликі коробки з вершини — Stack

Знімайте коробки, поки вага верхньої більша за `limit`. Щойно верхня коробка підходить, зупиніться: коробки під нею не перевіряйте. Вхідний масив задає стек від дна до вершини.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

foreach (int[] weights in new[] { new[] { 9, 3, 8, 7 }, new[] { 6, 8 }, Array.Empty<int>() })
{
    var boxes = new Stack<int>(weights);
    while (boxes.TryPeek(out int top) && top > 5)
        boxes.Pop();
    Console.WriteLine($"Зверху вниз: [{string.Join(", ", boxes)}]");
}
```

**Очікуваний вивід:**

```text
Зверху вниз: [3, 9]
Зверху вниз: []
Зверху вниз: []
```

</details>

## Завдання 18. Незалежна копія стека

Скопіюйте стек так, щоб порядок від вершини до дна зберігся. Зміни копії не повинні впливати на оригінал. `ToArray` повертає елементи від вершини, тому перед конструктором нового стека розверніть масив.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

var original = new Stack<int>(new[] { 1, 2, 3 });
var copy = CopyStack(original);
copy.Push(4);
Console.WriteLine(string.Join(", ", original));
Console.WriteLine(string.Join(", ", copy));
Console.WriteLine(CopyStack(new Stack<int>()).Count);

static Stack<int> CopyStack(Stack<int> source)
{
    int[] values = source.ToArray();
    Array.Reverse(values);
    return new Stack<int>(values);
}
```

**Очікуваний вивід:**

```text
3, 2, 1
4, 3, 2, 1
0
```

</details>

## Завдання 19. Підняти нижню картку — Stack

Перенесіть нижній елемент стека на вершину, зберігши порядок решти. Дозволений один допоміжний стек. Порожній стек та стек з одним елементом не змінюються. Вхідні масиви записані від дна до вершини.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

foreach (int[] values in new[] { new[] { 1, 2, 3 }, new[] { 7 }, Array.Empty<int>() })
{
    var cards = new Stack<int>(values);
    LiftBottom(cards);
    Console.WriteLine($"[{string.Join(", ", cards)}]");
}

static void LiftBottom(Stack<int> cards)
{
    if (cards.Count < 2) return;
    var buffer = new Stack<int>();
    while (cards.Count > 1) buffer.Push(cards.Pop());
    int bottom = cards.Pop();
    while (buffer.TryPop(out int value)) cards.Push(value);
    cards.Push(bottom);
}
```

**Очікуваний вивід:**

```text
[1, 3, 2]
[7]
[]
```

</details>

## Завдання 20. Побачити n-ту картку згори — Stack

Поверніть елемент на позиції `position` від вершини, нумерація від одиниці. Якщо позиція некоректна, поверніть `null`. Стек не змінюйте; для цієї невеликої вправи дозволено `ToArray`.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

var cards = new Stack<int>(new[] { 10, 20, 30 });
foreach (int position in new[] { 1, 3, 4, 0 })
    Console.WriteLine(PeekAt(cards, position)?.ToString() ?? "NONE");
Console.WriteLine($"Count={cards.Count}");

static int? PeekAt(Stack<int> cards, int position)
{
    if (position < 1 || position > cards.Count) return null;
    return cards.ToArray()[position - 1];
}
```

**Очікуваний вивід:**

```text
30
10
NONE
NONE
Count=3
```

</details>

## Завдання 21. Подвоїти кожну заявку — Queue

З черги `[A, B]` отримайте `[A, A, B, B]`, змінюючи ту саму чергу. Запам’ятайте початковий розмір і обробіть тільки стільки елементів: нові копії повторно не обробляйте.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

foreach (string[] values in new[] { new[] { "A", "B" }, new[] { "X" }, Array.Empty<string>() })
{
    var queue = new Queue<string>(values);
    int originalCount = queue.Count;
    for (int i = 0; i < originalCount; i++)
    {
        string item = queue.Dequeue();
        queue.Enqueue(item);
        queue.Enqueue(item);
    }
    Console.WriteLine($"[{string.Join(", ", queue)}]");
}
```

**Очікуваний вивід:**

```text
[A, A, B, B]
[X, X]
[]
```

</details>

## Завдання 22. Скасувати один талон — Queue

Видаліть лише перший талон із заданим номером. Збережіть порядок решти та поверніть, чи був талон знайдений. Перегляньте рівно початкову кількість елементів черги.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

var tickets = new Queue<int>(new[] { 2, 5, 2, 8 });
Console.WriteLine(Cancel(tickets, 2));
Console.WriteLine(string.Join(", ", tickets));
Console.WriteLine(Cancel(tickets, 9));
Console.WriteLine(Cancel(new Queue<int>(), 2));

static bool Cancel(Queue<int> tickets, int target)
{
    bool removed = false;
    int count = tickets.Count;
    for (int i = 0; i < count; i++)
    {
        int ticket = tickets.Dequeue();
        if (!removed && ticket == target) removed = true;
        else tickets.Enqueue(ticket);
    }
    return removed;
}
```

**Очікуваний вивід:**

```text
True
5, 2, 8
False
False
```

</details>

## Завдання 23. Розвернути початок черги

Розверніть лише перші `k` елементів черги; решта мають залишитися в початковому порядку. `0 ≤ k ≤ Count`. Використайте допоміжний стек, а потім поверніть незмінений хвіст за розвернутий початок.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

foreach (int k in new[] { 3, 0, 5 })
{
    var queue = new Queue<int>(new[] { 1, 2, 3, 4, 5 });
    ReversePrefix(queue, k);
    Console.WriteLine(string.Join(", ", queue));
}

static void ReversePrefix(Queue<int> queue, int k)
{
    int untouched = queue.Count - k;
    var stack = new Stack<int>();
    for (int i = 0; i < k; i++) stack.Push(queue.Dequeue());
    while (stack.TryPop(out int value)) queue.Enqueue(value);
    for (int i = 0; i < untouched; i++) queue.Enqueue(queue.Dequeue());
}
```

**Очікуваний вивід:**

```text
3, 2, 1, 4, 5
1, 2, 3, 4, 5
5, 4, 3, 2, 1
```

</details>

## Завдання 24. Об’єднати два потоки часу — Queue

Дві черги містять час подій у неспадному порядку. Перенесіть події в одну впорядковану чергу, щоразу порівнюючи голови. Для однакового часу беріть із першої. Початкові черги після обробки спорожнюються.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;

var first = new Queue<int>(new[] { 1, 4, 7 });
var second = new Queue<int>(new[] { 2, 4, 8 });
Console.WriteLine(string.Join(", ", MergeEvents(first, second)));
Console.WriteLine($"Залишок: {first.Count + second.Count}");
Console.WriteLine(MergeEvents(new Queue<int>(), new Queue<int>()).Count);

static Queue<int> MergeEvents(Queue<int> first, Queue<int> second)
{
    var result = new Queue<int>();
    while (first.Count > 0 || second.Count > 0)
    {
        if (second.Count == 0 || (first.Count > 0 && first.Peek() <= second.Peek()))
            result.Enqueue(first.Dequeue());
        else result.Enqueue(second.Dequeue());
    }
    return result;
}
```

**Очікуваний вивід:**

```text
1, 2, 4, 4, 7, 8
Залишок: 0
0
```

</details>

## Завдання 25. Буфер на два елементи — Queue

Звичайна черга приймає не більше двох значень. Напишіть `TryEnqueue`, що повертає `false`, коли буфер повний. Обробіть один елемент і повторіть вставку. Усе виконується в одному потоці.

`new Queue<int>(2)` задає початкову місткість, а не жорсткий ліміт; обмеження перевіряє наш метод. Перевірка `Count` і вставка тут придатні лише для послідовного виконання. [Конструктор Queue у .NET 10](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.queue-1.-ctor?view=net-10.0).

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;
using System.Collections.Concurrent;
using System.Threading;
using System.Threading.Tasks;

var buffer = new Queue<int>(2);
Console.WriteLine(TryEnqueue(buffer, 10, 2));
Console.WriteLine(TryEnqueue(buffer, 20, 2));
Console.WriteLine(TryEnqueue(buffer, 30, 2));
Console.WriteLine($"Оброблено: {buffer.Dequeue()}");
Console.WriteLine(TryEnqueue(buffer, 30, 2));
Console.WriteLine($"Залишок: {string.Join(", ", buffer)}");

static bool TryEnqueue(Queue<int> queue, int value, int limit)
{
    if (queue.Count >= limit) return false;
    queue.Enqueue(value);
    return true;
}
```

**Очікуваний вивід:**

```text
True
True
False
Оброблено: 10
True
Залишок: 20, 30
```

</details>

## Завдання 26. Два джерела подій — ConcurrentQueue

Два виробники додають події `A1, A2` та `B1, B2` у спільну чергу. Дочекайтеся обох через `Task.WhenAll`, потім заберіть усі події. Перевірте, що порядок кожного виробника збережений: `A1` раніше за `A2`, а `B1` раніше за `B2`.

Загальне чергування `A` і `B` залежить від планувальника. Порядок перевіряйте **до** сортування; сортування потрібне тільки для стабільного друку складу результату.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;
using System.Collections.Concurrent;
using System.Threading;
using System.Threading.Tasks;

var events = new ConcurrentQueue<string>();
await Task.WhenAll(
    Task.Run(() => Produce("A")),
    Task.Run(() => Produce("B")));

var received = new List<string>();
while (events.TryDequeue(out string? item)) received.Add(item);
bool localOrder = received.IndexOf("A1") < received.IndexOf("A2")
    && received.IndexOf("B1") < received.IndexOf("B2");
received.Sort(StringComparer.Ordinal);
Console.WriteLine($"Отримано: {string.Join(", ", received)}");
Console.WriteLine($"FIFO кожного виробника: {localOrder}");
Console.WriteLine($"Порожня: {events.IsEmpty}");

void Produce(string source)
{
    events.Enqueue(source + "1");
    events.Enqueue(source + "2");
}
```

**Очікуваний вивід:**

```text
Отримано: A1, A2, B1, B2
FIFO кожного виробника: True
Порожня: True
```

</details>

## Завдання 27. Двоє працівників розбирають готову чергу

Черга `ConcurrentQueue<int>` уже містить номери `1..6`; нові номери більше не надходять. Два завдання забирають їх через `TryDequeue`. Кожне складає результат у **власний** `List<int>`, а головна програма об’єднує списки лише після `Task.WhenAll`.

Розподіл роботи між працівниками не визначений: один може забрати навіть усі елементи. Перевіряємо загальний набір і порожню чергу, а не однакове навантаження. Тут `false` від `TryDequeue` завершує цикл, бо виробники вже не працюють.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;
using System.Collections.Concurrent;
using System.Threading;
using System.Threading.Tasks;

var queue = new ConcurrentQueue<int>(new[] { 1, 2, 3, 4, 5, 6 });
List<int>[] batches = await Task.WhenAll(Task.Run(Consume), Task.Run(Consume));
var processed = new List<int>();
foreach (var batch in batches) processed.AddRange(batch);
processed.Sort();
Console.WriteLine($"Оброблено: {string.Join(", ", processed)}");
Console.WriteLine($"Кількість: {processed.Count}");
Console.WriteLine($"Порожня: {queue.IsEmpty}");

List<int> Consume()
{
    var local = new List<int>();
    while (queue.TryDequeue(out int item)) local.Add(item);
    return local;
}
```

**Очікуваний вивід:**

```text
Оброблено: 1, 2, 3, 4, 5, 6
Кількість: 6
Порожня: True
```

</details>

## Завдання 28. Пачка карток — ConcurrentStack

Ознайомтеся з пакетними операціями `ConcurrentStack<T>`. Додайте `A, B, C` через `PushRange`, а потім заберіть не більше двох карток через `TryPopRange`. Використовуйте фактичну кількість повернених елементів, а не довжину буфера.

Цей перший приклад API виконується послідовно, тому порядок LIFO можна показати точно. Приклад зі спільним стеком і кількома завданнями є в [Лекції 3](../Lecture-3/tasks.md).

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;
using System.Collections.Concurrent;
using System.Threading;
using System.Threading.Tasks;

var cards = new ConcurrentStack<string>();
cards.PushRange(new[] { "A", "B", "C" });
string[] buffer = new string[2];
int count = cards.TryPopRange(buffer);
Console.WriteLine($"Пачка: {string.Join(", ", buffer, 0, count)}");
Console.WriteLine($"Верхівка: {(cards.TryPeek(out string? top) ? top : "NONE")}");
Console.WriteLine($"Знято: {(cards.TryPop(out string? last) ? last : "NONE")}");
Console.WriteLine($"Порожня пачка: {cards.TryPopRange(buffer)}");
```

**Очікуваний вивід:**

```text
Пачка: C, B
Верхівка: A
Знято: A
Порожня пачка: 0
```

</details>

## Завдання 29. Жетони з повторами — ConcurrentBag

Додайте в `ConcurrentBag<int>` жетони `5, 5, 9`. Переконайтеся, що `TryPeek` не видаляє елемент, потім заберіть усі жетони через `TryTake`. Повторні значення мають зберегтися.

Bag не гарантує FIFO або LIFO. Сортуйте лише отриманий результат для друку; не очікуйте конкретного значення від `TryPeek`. [Контракт ConcurrentBag у .NET 10](https://learn.microsoft.com/en-us/dotnet/api/system.collections.concurrent.concurrentbag-1?view=net-10.0).

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;
using System.Collections.Concurrent;
using System.Threading;
using System.Threading.Tasks;

var tokens = new ConcurrentBag<int>();
tokens.Add(5);
tokens.Add(5);
tokens.Add(9);
Console.WriteLine($"Є жетон: {tokens.TryPeek(out _)}");
Console.WriteLine($"До вилучення: {tokens.Count}");
var taken = new List<int>();
while (tokens.TryTake(out int token)) taken.Add(token);
taken.Sort();
Console.WriteLine($"Отримано: {string.Join(", ", taken)}");
Console.WriteLine($"Ще один: {tokens.TryTake(out _)}");
```

**Очікуваний вивід:**

```text
Є жетон: True
До вилучення: 3
Отримано: 5, 5, 9
Ще один: False
```

</details>

## Завдання 30. Незалежні результати вимірювань — ConcurrentBag

Для кожного слова запустіть окреме завдання, яке додає пару `(слово, довжина)` у спільний `ConcurrentBag`. Після завершення всіх завдань виведіть звіт за назвою слова. Для порожнього вхідного масиву звіт містить нуль записів.

Локальна змінна `word` усередині циклу дає кожному завданню власне слово. Сортування масиву-знімка впорядковує звіт; сам bag залишається неупорядкованим.

<details>
<summary>Повна реалізація на C#</summary>

```csharp
using System;
using System.Collections.Generic;
using System.Collections.Concurrent;
using System.Threading;
using System.Threading.Tasks;

await Report(new[] { "sun", "rain", "snow" });
await Report(Array.Empty<string>());

static async Task Report(string[] words)
{
    var results = new ConcurrentBag<(string Word, int Length)>();
    var workers = new Task[words.Length];
    for (int i = 0; i < words.Length; i++)
    {
        string word = words[i];
        workers[i] = Task.Run(() => results.Add((word, word.Length)));
    }
    await Task.WhenAll(workers);
    var snapshot = results.ToArray();
    Array.Sort(snapshot, (a, b) => StringComparer.Ordinal.Compare(a.Word, b.Word));
    foreach (var item in snapshot) Console.WriteLine($"{item.Word}: {item.Length}");
    Console.WriteLine($"Записів: {snapshot.Length}");
}
```

**Очікуваний вивід:**

```text
rain: 4
snow: 4
sun: 3
Записів: 3
Записів: 0
```

</details>
