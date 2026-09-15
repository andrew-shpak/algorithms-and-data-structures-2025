# Лекція 3 — .NET колекції

---

## Зміст

1. [Вступ: колекції в .NET](#1-вступ-колекції-в-net)
2. [Послідовні колекції: List, LinkedList, Queue](#2-послідовні-колекції-list-linkedlist-queue)
3. [Множини та словники: SortedSet, HashSet, SortedDictionary, Dictionary](#3-множини-та-словники-sortedset-hashset-sorteddictionary-dictionary)
4. [Перелічення: IEnumerable, foreach, LINQ](#4-перелічення-ienumerable-foreach-linq)
5. [Приклади коду](#5-приклади-коду)
6. [Як обрати колекцію](#6-як-обрати-колекцію)
7. [Підсумки](#7-підсумки)
8. [Питання для самоперевірки](#8-питання-для-самоперевірки)

---

## 1. Вступ: колекції в .NET

Простір імен **`System.Collections.Generic`** містить узагальнені (generic) колекції, а **`System.Linq`** — алгоритми над ними:

- **Колекції** — узагальнені структури даних (`List<T>`, `LinkedList<T>`, `Dictionary<TKey,TValue>`, ...)
- **Перелічувачі** — універсальний спосіб обходу через інтерфейс `IEnumerable<T>` і цикл `foreach`
- **Алгоритми** — методи LINQ (`OrderBy`, `Where`, `Sum`, ...) та вбудовані методи (`Sort`, `IndexOf`, ...)

Головна ідея: метод LINQ не знає, з якою колекцією працює, — він бачить лише `IEnumerable<T>`.

---

## 2. Послідовні колекції: List, LinkedList, Queue

| Колекція | Внутрішня будова | Сильні сторони | Слабкі сторони |
|----------|------------------|----------------|----------------|
| `T[]` | Неперервний масив фіксованого розміру | Доступ за індексом O(1), мінімум накладних витрат | Розмір не змінюється |
| `List<T>` | Неперервний динамічний масив | Доступ за індексом O(1), кеш-дружній | Вставка в середину O(n) |
| `LinkedList<T>` | Двозв'язний список вузлів | Вставка/видалення за вузлом O(1), обидва кінці O(1) | Немає індексатора, погана локальність |
| `Queue<T>` / `Stack<T>` | Кільцевий буфер / масив | `Enqueue`/`Dequeue`, `Push`/`Pop` за O(1) | Доступ лише до одного кінця |

**Важливо про `List<T>`:** коли `Count == Capacity`, список виділяє новий масив (удвічі більший) і копіює елементи. Тому `Add` — **амортизовано** O(1). Двобічної черги (deque) у стандартній бібліотеці немає — її роль виконує `LinkedList<T>`.

---

## 3. Множини та словники: SortedSet, HashSet, SortedDictionary, Dictionary

| Колекція | Будова | Порядок | Пошук |
|----------|--------|---------|-------|
| `SortedSet<T>` / `SortedDictionary<TKey,TValue>` | Збалансоване дерево (червоно-чорне) | Відсортовані за ключем | O(log n) |
| `HashSet<T>` / `Dictionary<TKey,TValue>` | Хеш-таблиця | Без гарантованого порядку | O(1) у середньому, O(n) у найгіршому |

- Множина зберігає **унікальні ключі**, словник — пари **ключ → значення**.
- Для дублікатів можна використати `Dictionary<TKey, List<TValue>>` або `ILookup` з LINQ.
- Відсортовані версії потрібні, коли важливий порядок або запити на діапазон (`GetViewBetween`, `Min`, `Max`).

---

## 4. Перелічення: IEnumerable, foreach, LINQ

Перелічувач (`IEnumerator<T>`) поводиться як курсор: `MoveNext()` переходить до наступного елемента, `Current` повертає поточний. Цикл `foreach` робить це автоматично.

| Інтерфейс | Можливості | Приклад колекції |
|-----------|------------|------------------|
| `IEnumerable<T>` | лише обхід уперед (`foreach`, LINQ) | усі колекції |
| `ICollection<T>` | + `Count`, `Add`, `Remove`, `Contains` | `LinkedList<T>`, `HashSet<T>`, `SortedSet<T>` |
| `IList<T>` | + індексатор `[i]`, `Insert`, `RemoveAt` | `T[]`, `List<T>` |

Змінювати колекцію під час `foreach` **не можна** — наступний `MoveNext()` кине `InvalidOperationException`.

---

## 5. Приклади коду

Кожен приклад — окрема консольна програма з top-level statements. Запуск: `dotnet new console`, вставити код у `Program.cs`, `dotnet run`.

### Приклад 1 — `List<T>`: розмір і місткість

```csharp
var list = new List<int>(4);      // порожній список: Count = 0, але місце під 4 елементи вже зарезервоване
for (int i = 1; i <= 5; i++)
{
    list.Add(i * 10);             // додаємо в кінець; на 5-му елементі місткість буде збільшено
}
Console.WriteLine($"Count={list.Count} Capacity>=5? {list.Capacity >= 5}"); // Count — реальні елементи, Capacity — місце
Console.WriteLine($"list[2]={list[2]}");                 // доступ за індексом за O(1), з перевіркою меж
list.Insert(1, 15);               // вставка в середину: усі елементи праворуч зсуваються — O(n)
list.RemoveAt(list.Count - 1);    // видалення з кінця — O(1)
Console.WriteLine(string.Join(' ', list));               // string.Join обходить список через IEnumerable<T>
```

### Приклад 2 — `LinkedList<T>`: вставка та видалення за вузлом

```csharp
var lst = new LinkedList<int>([1, 2, 4, 5]);   // двозв'язний список з виразу колекції (C# 12)
LinkedListNode<int> node = lst.First!.Next!.Next!; // крокуємо до 3-го вузла (O(k), бо індексатора немає)
lst.AddBefore(node, 3);                        // вставка перед вузлом за O(1): лише перев'язуємо посилання
lst.AddFirst(0);                               // додавання на початок — O(1)

LinkedListNode<int>? current = lst.First;      // видаляємо всі парні елементи вручну
while (current is not null)
{
    LinkedListNode<int>? next = current.Next;  // запам'ятовуємо наступний ДО видалення поточного
    if (current.Value % 2 == 0)
    {
        lst.Remove(current);                   // видалення вузла за O(1)
    }
    current = next;
}
Console.WriteLine(string.Join(' ', lst));      // решта вузлів не зачеплені — посилання на них валідні
```

### Приклад 3 — двобічна черга на `LinkedList<T>` та `Queue<T>`

```csharp
var deque = new LinkedList<int>();   // LinkedList як двобічна черга (deque)
deque.AddLast(2);                    // [2]
deque.AddLast(3);                    // [2, 3]
deque.AddFirst(1);                   // [1, 2, 3] — на відміну від List<T>.Insert(0, ...), це O(1)
deque.AddFirst(0);                   // [0, 1, 2, 3]
Console.WriteLine($"front={deque.First!.Value} back={deque.Last!.Value}"); // перший і останній елементи
deque.RemoveFirst();                 // [1, 2, 3] — видалення з початку за O(1)
deque.RemoveLast();                  // [1, 2]
Console.WriteLine($"size={deque.Count}");

var queue = new Queue<int>();        // звичайна черга FIFO на кільцевому буфері
queue.Enqueue(10);                   // додаємо в хвіст — O(1)
queue.Enqueue(20);
Console.WriteLine($"dequeue={queue.Dequeue()} peek={queue.Peek()}"); // забираємо голову, потім дивимось нову
```

### Приклад 4 — `SortedSet<T>` проти `HashSet<T>`

```csharp
var sorted = new SortedSet<int> { 5, 1, 4, 1, 3 }; // дублікат 1 відкидається, елементи впорядковані
var hashed = new HashSet<int> { 5, 1, 4, 1, 3 };   // теж унікальні, але порядок не гарантується

Console.WriteLine(string.Join(' ', sorted));        // завжди виведе 1 3 4 5

bool inserted = sorted.Add(4);                      // Add повертає false, якщо такий ключ уже є
Console.WriteLine($"inserted 4 again? {inserted}");

int lowerBound = sorted.GetViewBetween(2, int.MaxValue).Min; // перший елемент >= 2 за O(log n) — лише в SortedSet
Console.WriteLine($"lower bound(2)={lowerBound}");

Console.WriteLine($"hashed contains 3? {hashed.Contains(3)} size={hashed.Count}"); // Contains в середньому O(1)
```

### Приклад 5 — `Dictionary` та `SortedDictionary`: частоти слів

```csharp
string text = "to be or not to be";
var freq = new Dictionary<string, int>();          // хеш-таблиця: слово -> кількість

foreach (string word in text.Split(' '))           // розбиваємо рядок на слова
{
    freq[word] = freq.GetValueOrDefault(word) + 1; // читання індексатором кинуло б KeyNotFoundException — беремо 0 за замовчуванням
}

var sortedFreq = new SortedDictionary<string, int>(freq, StringComparer.Ordinal); // копія з алфавітним порядком ключів
foreach (var (w, c) in sortedFreq)                 // деконструкція KeyValuePair на ключ і значення
{
    Console.WriteLine($"{w}: {c}");
}

bool found = freq.TryGetValue("xyz", out int count); // TryGetValue не кидає виняток і нічого не додає
Console.WriteLine($"found xyz? {found} (count={count})");
```

### Приклад 6 — перелічення та LINQ

```csharp
List<int> numbers = [7, 2, 9, 4, 1];               // вираз колекції (C# 12)
numbers.Sort();                                    // сортування на місці, O(n log n)
Console.WriteLine($"index of 4 = {numbers.IndexOf(4)}"); // лінійний пошук; повертає -1, якщо не знайдено

int sum = numbers.Sum();                           // LINQ: сума елементів будь-якого IEnumerable<int>
Console.WriteLine($"sum = {sum}");

for (int i = numbers.Count - 1; i >= 0; i--)       // обхід з кінця через індекси
{
    Console.Write($"{numbers[i]} ");
}
Console.WriteLine();

// Видаляти в foreach не можна (InvalidOperationException) — використовуємо RemoveAll з предикатом
numbers.RemoveAll(x => x % 2 == 1);                // один прохід, O(n)
Console.WriteLine(string.Join(' ', numbers.Select(x => x * 10))); // Select — лінива проєкція кожного елемента
```

### Приклад запуску

```
# Приклад 1
Count=5 Capacity>=5? True
list[2]=30
10 15 20 30 40
# Приклад 2
1 3 5
# Приклад 3
front=0 back=3
size=2
dequeue=10 peek=20
# Приклад 4
1 3 4 5
inserted 4 again? False
lower bound(2)=3
hashed contains 3? True size=4
# Приклад 5
be: 2
not: 1
or: 1
to: 2
found xyz? False (count=0)
# Приклад 6
index of 4 = 2
sum = 23
9 7 4 2 1 
20 40
```

---

## 6. Як обрати колекцію

| Операція | `List<T>` | `Queue<T>` | `LinkedList<T>` | `SortedSet`/`SortedDictionary` | `HashSet`/`Dictionary` |
|----------|-----------|------------|-----------------|--------------------------------|------------------------|
| Доступ за індексом | O(1) | — | — | — | — |
| Вставка в кінець | O(1)* | O(1)* | O(1) | — | — |
| Вставка на початок | O(n) | — | O(1) | — | — |
| Вставка в середину | O(n) | — | O(1)** | O(log n) | O(1) avg |
| Пошук за значенням/ключем | O(n) | O(n) | O(n) | O(log n) | O(1) avg, O(n) worst |
| Видалення | O(n) | O(1) голова | O(1)** | O(log n) | O(1) avg |
| Впорядкованість | ні | ні | ні | так | ні |

\* амортизовано; \*\* якщо вже маємо посилання на вузол `LinkedListNode<T>`.

**Практичні правила:**

1. За замовчуванням — `List<T>` (або масив, якщо розмір відомий). Він найшвидший у більшості реальних задач завдяки кешу.
2. Черга FIFO — `Queue<T>`; вставки з обох кінців (ковзне вікно) — `LinkedList<T>`.
3. Часті вставки/видалення в середині за готовими вузлами — `LinkedList<T>`.
4. Швидкий пошук «чи є елемент» без порядку — `HashSet<T>` / `Dictionary<TKey,TValue>`.
5. Потрібен відсортований порядок або запити на діапазон — `SortedSet<T>` / `SortedDictionary<TKey,TValue>`.

---

## 7. Підсумки

- .NET розділяє **колекції**, **перелічення** (`IEnumerable<T>`) та **алгоритми** (LINQ); інтерфейс `IEnumerable<T>` з'єднує їх між собою.
- `List<T>` — неперервна пам'ять і O(1) доступ; `LinkedList<T>` — O(1) вставки за вузлом та з обох кінців; `Queue<T>` — O(1) FIFO.
- `SortedSet`/`SortedDictionary` — дерево, O(log n), впорядковані; `HashSet`/`Dictionary` — хеш-таблиця, O(1) у середньому.
- Модифікація колекції під час `foreach` **інвалідує перелічувач** — використовуйте `RemoveAll`, обхід за індексами або копію.
- Вибір колекції — це вибір між складністю операцій, порядком і використанням пам'яті.

---

## 8. Питання для самоперевірки

1. Чому `Add` у `List<T>` має амортизовану складність O(1), а не строго O(1)?
2. Чим відрізняється читання `freq[key]` від `freq.TryGetValue(key, out var v)` у `Dictionary<TKey,TValue>`?
3. Коли `SortedSet<T>` кращий за `HashSet<T>`, незважаючи на гіршу асимптотику пошуку?
4. Чому `LinkedList<T>` не має методу `Sort` і що використати натомість?
5. Що станеться, якщо видалити елемент зі `List<T>` всередині `foreach`, і як правильно видаляти елементи під час обходу?
