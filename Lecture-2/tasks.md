# Лекція 2 — Прості завдання на списки, стек і чергу

[Матеріал лекції](Examples.md) · [Наступні вправи на колекції](../Lecture-3/tasks.md) · [Застосування стека й черги](../Lecture-6/tasks.md)

Кожне завдання незалежне. Почніть із готових колекцій C#: `List<T>`, `LinkedList<T>`, `Stack<T>`, `Queue<T>`. Дані можна задати прямо в програмі; меню та зчитування файлів не потрібні. Індекси починаються з нуля, `[]` означає порожню колекцію.

Кожне завдання містить повну програму: чотири початкові приклади відкриті, решта реалізацій — у розгортних блоках. Спочатку спробуйте розв’язати задачу самостійно, потім порівняйте з кодом.

| Колекція | Що тренуємо | Завдання |
|---|---|---|
| `List<T>` | Індекс, вставка, заміна, видалення | 1–2, 9–12 |
| `LinkedList<T>` | Вузол, сусіди, обидва кінці | 3–4, 13–16 |
| `Stack<T>` | LIFO: останній доданий виходить першим | 5–6, 17–20 |
| `Queue<T>` | FIFO: перший доданий виходить першим | 7–8, 21–24 |

**Усього: 24 завдання із повними реалізаціями та очікуваним виводом.**

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
