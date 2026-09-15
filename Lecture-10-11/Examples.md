# Лекції 10–11 — Графи: від термінології до SCC і мостів (C#)

> Розрахунок: **~195 хвилин** (дві пари) з трьома перервами.
> Усі приклади — повні програми на C# (top-level statements, `Nullable` увімкнено).
> Кожен приклад можна запустити як file-based app: `dotnet run Example.cs`.
> Позначення: `n = |V|` — кількість вершин, `m = |E|` — кількість ребер.

---

## Зміст

### Частина I — Основи (≈ 50 хв)

0. [Вступ і план](#0-вступ-і-план) — 5 хв
1. [Термінологія та приклади з життя](#1-термінологія-та-приклади-з-життя) — 10 хв
2. [Представлення графа в пам'яті](#2-представлення-графа-в-памяті) — 15 хв
3. [Обхід у ширину (BFS)](#3-обхід-у-ширину-bfs) — 20 хв

**☕ Перерва 1 (після ~50 хв)**

### Частина II — DFS, топологія, найкоротші шляхи (≈ 55 хв)

4. [Обхід у глибину (DFS)](#4-обхід-у-глибину-dfs) — 20 хв
5. [Топологічне сортування](#5-топологічне-сортування) — 12 хв
6. [Алгоритм Дейкстри](#6-алгоритм-дейкстри) — 15 хв
7. [Беллман–Форд і від'ємні цикли](#7-беллманфорд-і-відємні-цикли) — 8 хв

**☕ Перерва 2 (після ~105 хв)**

### Частина III — Усі пари, DAG, A*, остовні дерева (≈ 45 хв)

8. [Флойд–Воршелл](#8-флойдворшелл) — 8 хв
9. [Найкоротший і найдовший шлях у DAG](#9-найкоротший-і-найдовший-шлях-у-dag) — 5 хв
10. [A* на сітці](#10-a-на-сітці) — 8 хв
11. [Мінімальне остовне дерево: Union-Find, Краскал, Прім](#11-мінімальне-остовне-дерево-union-find-краскал-прім) — 17 хв
12. [Застосування Union-Find](#12-застосування-union-find) — 7 хв

**☕ Перерва 3 (після ~150 хв)**

### Частина IV — Структура графа та практика (≈ 45 хв)

13. [Компоненти сильної зв'язності (Косарайю, Тарʼян)](#13-компоненти-сильної-звязності-косарайю-таряна) — 10 хв
14. [Мости та точки зчленування](#14-мости-та-точки-зчленування) — 10 хв
15. [Як обрати алгоритм](#15-як-обрати-алгоритм) — 5 хв
16. [Практичні задачі](#16-практичні-задачі) — 15 хв
17. [Питання для самоперевірки та задачі](#17-питання-для-самоперевірки-та-задачі) — 5 хв

**Разом: 5 + 10 + 15 + 20 + 20 + 12 + 15 + 8 + 8 + 5 + 8 + 17 + 7 + 10 + 10 + 5 + 15 + 5 = 195 хв**

---

## 0. Вступ і план

Графи — це «мова» для опису **зв'язків**. Коли в задачі є об'єкти та відношення
між ними («хто з ким дружить», «що від чого залежить», «звідки куди можна доїхати»),
майже завжди під капотом лежить граф.

Що ми вміємо після цієї лекції:

- обирати **представлення** графа під задачу та оцінювати пам'ять;
- писати **BFS** і **DFS** «з закритими очима» та знати, що вони дають «безкоштовно»
  (відстані, компоненти, цикли, часові мітки);
- розв'язувати задачі **залежностей** (топологічне сортування);
- шукати **найкоротші шляхи**: Дейкстра, Беллман–Форд, Флойд–Воршелл, DAG, A*;
- будувати **мінімальне остовне дерево** (Краскал, Прім) і користуватися **Union-Find**;
- знаходити **компоненти сильної зв'язності**, **мости** і **точки зчленування**.

### Наш «наскрізний» граф

Щоб не губитися, майже всі приклади працюють з **одним і тим самим** маленьким графом
із 6 вершин. Запам'ятайте його — далі він з'являтиметься постійно.

Неорієнтований граф G (6 вершин, 7 ребер):

```text
      0 ───── 1 ───── 3
      │       │       │
      │       │       │
      2 ───── 4 ───── 5

Ребра: 0-1, 0-2, 1-3, 1-4, 2-4, 3-5, 4-5
```

Зважена версія (ваги на ребрах — «хвилини в дорозі»):

```text
      0 ──7── 1 ──2── 3
      │       │       │
      2       1       3
      │       │       │
      2 ──3── 4 ──6── 5

(0,1,7) (0,2,2) (1,3,2) (1,4,1) (2,4,3) (3,5,3) (4,5,6)
```

Орієнтована ациклічна версія (DAG) — для топологічного сортування:

```text
      0 ────► 1 ────► 3
      │       │       │
      ▼       ▼       ▼
      2 ────► 4 ────► 5

0→1, 0→2, 1→3, 1→4, 2→4, 3→5, 4→5
```

---

## 1. Термінологія та приклади з життя

### 1.1 Основні визначення

**Граф** `G = (V, E)` — пара множин: **вершини** (vertices, nodes) `V` та **ребра** (edges) `E`.

| Термін | Значення | Приклад на графі G |
|---|---|---|
| **Вершина** | Об'єкт | `0`, `1`, …, `5` |
| **Ребро** | Зв'язок між двома вершинами | `1-4` |
| **Суміжні** (сусіди) | Вершини, з'єднані ребром | `1` і `3` |
| **Неорієнтований** граф | Ребро `u-v` = `v-u` | дружба у Facebook |
| **Орієнтований** граф (орграф) | Ребро `u→v` має напрям | підписка в Instagram |
| **Зважений** граф | Кожне ребро має вагу `w(u,v)` | відстань у км |
| **Степінь** `deg(v)` | Кількість ребер при вершині | `deg(1) = 3` |
| **Вхідний / вихідний степінь** | Для орграфа: `in(v)`, `out(v)` | у DAG: `in(4) = 2`, `out(4) = 1` |
| **Шлях** | Послідовність вершин, сусідні з'єднані ребрами | `0 → 2 → 4 → 5` |
| **Простий шлях** | Шлях без повторення вершин | `0 → 1 → 3` |
| **Цикл** | Шлях, що повертається в початок | `0 → 1 → 4 → 2 → 0` |
| **Зв'язний** граф | Між будь-якими двома вершинами є шлях | G — зв'язний |
| **Компонента зв'язності** | Максимальна зв'язна частина | у G одна компонента |
| **Дерево** | Зв'язний граф без циклів (`m = n − 1`) | остовне дерево G |
| **DAG** | Directed Acyclic Graph — орграф без циклів | залежності пакетів |
| **Петля** | Ребро `v-v` | «сам собі друг» |
| **Мультиграф** | Дозволено кілька ребер між тією самою парою | кілька рейсів між містами |
| **Розріджений** граф | `m ≈ O(n)` | дороги, соцмережі |
| **Щільний** граф | `m ≈ O(n²)` | «кожен з кожним» |

**Лема про рукостискання.** У неорієнтованому графі `Σ deg(v) = 2m`, бо кожне ребро
додає 1 до степеня обох кінців. Для G: `2 + 3 + 2 + 2 + 3 + 2 = 14 = 2 · 7`. ✔

**Скільки може бути ребер?** Простий неорієнтований граф: `m ≤ n(n−1)/2`.
Орієнтований: `m ≤ n(n−1)`. Тому `O(m)` завжди `⊆ O(n²)`, і
`O(log m) = O(log n)` — це знадобиться в аналізі Дейкстри та Краскала.

### 1.2 Приклади з життя

| Предметна область | Вершини | Ребра | Тип | Типова задача |
|---|---|---|---|---|
| Карти, навігація | перехрестя | дороги | зважений, орієнтований | найкоротший шлях (Дейкстра, A*) |
| Соцмережі | люди | дружба / підписка | неор. / орієнт. | «друзі друзів» (BFS), спільноти |
| Збірка ПЗ, NuGet | пакети | «залежить від» | DAG | порядок збірки (топосорт) |
| Інтернет | роутери | канали | зважений | маршрутизація (OSPF = Дейкстра, RIP = Беллман–Форд) |
| Електромережі | підстанції | лінії | зважений | дешева мережа (MST), критичні лінії (мости) |
| Ігри | клітинки карти | переходи | сітка | пошук шляху (A*), заливка (flood fill) |
| Компілятори | блоки коду | переходи керування | орієнт. | мертвий код (досяжність), цикли (SCC) |
| Валюта | валюти | курси (`-log r`) | зважений | арбітраж = від'ємний цикл (Беллман–Форд) |

### 1.3 Сітка — теж граф

Двовимірна карта `char[,]` — це граф, у якому клітинка `(r, c)` має до 4 сусідів.
Ми **не будуємо** списки суміжності явно — сусідів генеруємо «на льоту»:

```text
  ┌───┬───┬───┐
  │   │ ↑ │   │     сусіди (r, c): (r-1, c), (r+1, c), (r, c-1), (r, c+1)
  ├───┼───┼───┤
  │ ← │ ● │ → │     n = rows · cols вершин, m ≈ 2 · rows · cols ребер
  ├───┼───┼───┤
  │   │ ↓ │   │
  └───┴───┴───┘
```

### Міні-вправа 1

Скільки ребер має повний граф на 5 вершинах `K5`? Чи може неорієнтований граф мати
рівно одну вершину непарного степеня?

<details>
<summary>Розв'язок</summary>

`K5`: `5 · 4 / 2 = 10` ребер. Одна вершина непарного степеня неможлива: сума степенів
`2m` парна, отже кількість вершин непарного степеня завжди **парна**.
</details>

---

## 2. Представлення графа в пам'яті

### 2.1 Три класичні способи

```text
Граф G:                 Матриця суміжності        Список суміжності     Список ребер
                          0 1 2 3 4 5
  0 ─ 1 ─ 3            0 [0 1 1 0 0 0]            0: 1, 2               (0,1)
  │   │   │            1 [1 0 0 1 1 0]            1: 0, 3, 4            (0,2)
  2 ─ 4 ─ 5            2 [1 0 0 0 1 0]            2: 0, 4               (1,3)
                       3 [0 1 0 0 0 1]            3: 1, 5               (1,4)
                       4 [0 1 1 0 0 1]            4: 1, 2, 5            (2,4)
                       5 [0 0 0 1 1 0]            5: 3, 4               (3,5)
                                                                        (4,5)
```

| Операція | Матриця `bool[n,n]` | Список `List<int>[]` | Список ребер | `HashSet<int>[]` |
|---|---|---|---|---|
| Пам'ять | `O(n²)` | `O(n + m)` | `O(m)` | `O(n + m)` (більша константа) |
| Чи є ребро `u-v`? | `O(1)` | `O(deg u)` | `O(m)` | `O(1)` в середньому |
| Перебрати сусідів `u` | `O(n)` | `O(deg u)` | `O(m)` | `O(deg u)` |
| Перебрати всі ребра | `O(n²)` | `O(n + m)` | `O(m)` | `O(n + m)` |
| Додати ребро | `O(1)` | `O(1)` амортиз. | `O(1)` | `O(1)` в середньому |
| Видалити ребро | `O(1)` | `O(deg u)` | `O(m)` | `O(1)` в середньому |
| Додати вершину | `O(n²)` (перевиділення) | `O(1)` амортиз. | `O(1)` | `O(1)` |
| Де добре | Флойд–Воршелл, щільні графи, `n ≤ ~5000` | **майже завжди**: BFS, DFS, Дейкстра | Краскал, Беллман–Форд | часті перевірки ребра |

**Оцінка пам'яті на практиці.** Для `n = 100 000`: матриця `bool` — `10¹⁰` байт ≈ **10 ГБ** 🙅.
Список суміжності на `m = 500 000` ребер — кілька мегабайт.

### 2.2 Матриця, список, список ребер

```csharp
// Приклад 2.1: три представлення одного графа G
const int N = 6;
// Список ребер — «сирі» дані, з яких будуємо все інше
(int U, int V)[] edges = [(0, 1), (0, 2), (1, 3), (1, 4), (2, 4), (3, 5), (4, 5)];

// 1) Матриця суміжності: matrix[u, v] == true, якщо є ребро u-v
var matrix = new bool[N, N];

// 2) Список суміжності: adjacency[u] — список сусідів u
var adjacency = new List<int>[N];
for (int i = 0; i < N; i++)
{
    adjacency[i] = []; // УВАГА: кожен список треба створити, інакше NullReferenceException
}

foreach (var (u, v) in edges)
{
    // Граф неорієнтований — ребро записуємо в обидва боки
    matrix[u, v] = true;
    matrix[v, u] = true;
    adjacency[u].Add(v);
    adjacency[v].Add(u);
}

Console.WriteLine("Матриця суміжності:");
for (int u = 0; u < N; u++)
{
    var row = new List<string>();
    for (int v = 0; v < N; v++)
    {
        row.Add(matrix[u, v] ? "1" : "0");
    }
    Console.WriteLine($"  {u}: {string.Join(' ', row)}");
}

Console.WriteLine("Список суміжності:");
for (int u = 0; u < N; u++)
{
    Console.WriteLine($"  {u}: [{string.Join(", ", adjacency[u])}], deg = {adjacency[u].Count}");
}

// Перевірка ребра: матриця — O(1), список — O(deg)
Console.WriteLine($"Ребро 1-4? матриця: {matrix[1, 4]}, список: {adjacency[1].Contains(4)}");
Console.WriteLine($"Ребро 0-5? матриця: {matrix[0, 5]}, список: {adjacency[0].Contains(5)}");

// Лема про рукостискання: сума степенів = 2m
int degreeSum = adjacency.Sum(list => list.Count);
Console.WriteLine($"Сума степенів = {degreeSum}, 2m = {2 * edges.Length}");
```

**Приклад запуску:**

```text
Матриця суміжності:
  0: 0 1 1 0 0 0
  1: 1 0 0 1 1 0
  2: 1 0 0 0 1 0
  3: 0 1 0 0 0 1
  4: 0 1 1 0 0 1
  5: 0 0 0 1 1 0
Список суміжності:
  0: [1, 2], deg = 2
  1: [0, 3, 4], deg = 3
  2: [0, 4], deg = 2
  3: [1, 5], deg = 2
  4: [1, 2, 5], deg = 3
  5: [3, 4], deg = 2
Ребро 1-4? матриця: True, список: True
Ребро 0-5? матриця: False, список: False
Сума степенів = 14, 2m = 14
```

### 2.3 Іменовані вершини: `Dictionary<string, List<string>>`

Коли вершини — рядки (міста, пакети, користувачі), є два шляхи:

1. Словник `Dictionary<string, List<string>>` — просто й наочно.
2. **Стиснення індексів**: `Dictionary<string, int>` «ім'я → номер» + звичайний `List<int>[]`.
   Швидше (масиви замість хешів) і дає змогу застосувати будь-який алгоритм «на числах».

```csharp
// Приклад 2.2: граф з іменованими вершинами та перетворення імен в індекси
var friends = new Dictionary<string, List<string>>();

void AddFriendship(string a, string b)
{
    // CollectionsMarshal був би швидшим, але TryAdd — читабельніше для лекції
    friends.TryAdd(a, []);
    friends.TryAdd(b, []);
    friends[a].Add(b);
    friends[b].Add(a);
}

AddFriendship("Анна", "Богдан");
AddFriendship("Анна", "Віра");
AddFriendship("Богдан", "Галина");
AddFriendship("Віра", "Галина");
AddFriendship("Дмитро", "Олена"); // окрема «компанія»

foreach (var (person, list) in friends)
{
    Console.WriteLine($"{person,-7} → {string.Join(", ", list)}");
}

// Стиснення: ім'я → індекс, індекс → ім'я
var indexOf = new Dictionary<string, int>();
var names = new List<string>();
foreach (string name in friends.Keys)
{
    indexOf[name] = names.Count;
    names.Add(name);
}

var adjacency = new List<int>[names.Count];
for (int i = 0; i < names.Count; i++)
{
    adjacency[i] = friends[names[i]].Select(friend => indexOf[friend]).ToList();
}

Console.WriteLine("Після стиснення індексів:");
for (int i = 0; i < names.Count; i++)
{
    Console.WriteLine($"  {i} ({names[i]}): [{string.Join(", ", adjacency[i])}]");
}
```

> `Dictionary<TKey, TValue>` у .NET перебирає ключі в порядку вставки, **поки не було
> видалень**. Покладатися на це в продакшені не варто, але для детермінованого виводу
> в лекції — підходить.

**Приклад запуску:**

```text
Анна    → Богдан, Віра
Богдан  → Анна, Галина
Віра    → Анна, Галина
Галина  → Богдан, Віра
Дмитро  → Олена
Олена   → Дмитро
Після стиснення індексів:
  0 (Анна): [1, 2]
  1 (Богдан): [0, 3]
  2 (Віра): [0, 3]
  3 (Галина): [1, 2]
  4 (Дмитро): [5]
  5 (Олена): [4]
```

### 2.4 Зважені ребра: `record struct`

`readonly record struct` дає незмінний, компактний (без алокацій у купі) тип із
готовими `Equals`, `GetHashCode` і `ToString` — ідеально для ребра.

```csharp
// Приклад 2.3: зважений граф G на record struct
Edge[] edges =
[
    new(0, 1, 7), new(0, 2, 2), new(1, 3, 2), new(1, 4, 1),
    new(2, 4, 3), new(3, 5, 3), new(4, 5, 6),
];

const int N = 6;
// Для зваженого списку суміжності зберігаємо пару (сусід, вага)
var adjacency = new List<(int To, int Weight)>[N];
for (int i = 0; i < N; i++)
{
    adjacency[i] = [];
}

foreach (var edge in edges)
{
    adjacency[edge.From].Add((edge.To, edge.Weight));
    adjacency[edge.To].Add((edge.From, edge.Weight)); // неорієнтований
}

Console.WriteLine($"Перше ребро: {edges[0]}");
Console.WriteLine($"Сумарна вага всіх ребер: {edges.Sum(e => e.Weight)}");

// Сортування ребер за вагою — перший крок алгоритму Краскала
var sorted = edges.OrderBy(e => e.Weight).ThenBy(e => e.From).ToArray();
Console.WriteLine($"Відсортовані: {string.Join(" ", sorted.Select(e => $"{e.From}-{e.To}:{e.Weight}"))}");

for (int u = 0; u < N; u++)
{
    Console.WriteLine($"  {u}: {string.Join(", ", adjacency[u].Select(p => $"{p.To}(w={p.Weight})"))}");
}

// Ребро як значення: однакові поля — однакові ребра
Console.WriteLine($"new Edge(1,4,1) == edges[3]: {new Edge(1, 4, 1) == edges[3]}");

/// <summary>Зважене ребро. Незмінне та без алокацій у купі.</summary>
readonly record struct Edge(int From, int To, int Weight);
```

**Приклад запуску:**

```text
Перше ребро: Edge { From = 0, To = 1, Weight = 7 }
Сумарна вага всіх ребер: 24
Відсортовані: 1-4:1 0-2:2 1-3:2 2-4:3 3-5:3 4-5:6 0-1:7
  0: 1(w=7), 2(w=2)
  1: 0(w=7), 3(w=2), 4(w=1)
  2: 0(w=2), 4(w=3)
  3: 1(w=2), 5(w=3)
  4: 1(w=1), 2(w=3), 5(w=6)
  5: 3(w=3), 4(w=6)
new Edge(1,4,1) == edges[3]: True
```

### 2.5 Багаторазовий клас `Graph`

Щоб не повторювати ініціалізацію, зберемо простий клас. Далі в лекції для
самодостатності прикладів ми часто будуватимемо `List<int>[]` прямо в коді,
але в реальному проєкті краще мати такий тип.

```csharp
// Приклад 2.4: багаторазовий клас Graph
var graph = new Graph(6, isDirected: false);
foreach (var (u, v) in new[] { (0, 1), (0, 2), (1, 3), (1, 4), (2, 4), (3, 5), (4, 5) })
{
    graph.AddEdge(u, v);
}

Console.WriteLine(graph);
Console.WriteLine($"Вершин: {graph.VertexCount}, ребер: {graph.EdgeCount}");
Console.WriteLine($"Сусіди 4: {string.Join(", ", graph.Neighbors(4))}");
Console.WriteLine($"Є ребро 3-5: {graph.HasEdge(3, 5)}; є ребро 5-3: {graph.HasEdge(5, 3)}");

var directed = new Graph(3, isDirected: true);
directed.AddEdge(0, 1);
directed.AddEdge(1, 2);
Console.WriteLine($"Орграф: є 0→1: {directed.HasEdge(0, 1)}, є 1→0: {directed.HasEdge(1, 0)}");

try
{
    graph.AddEdge(0, 42);
}
catch (ArgumentOutOfRangeException ex)
{
    Console.WriteLine($"Помилка: {ex.ParamName} поза межами");
}

/// <summary>
/// Простий граф на вершинах 0..n-1 зі списками суміжності.
/// Підтримує орієнтований і неорієнтований режими, ваги зберігаються разом із сусідом.
/// </summary>
sealed class Graph
{
    private readonly List<(int To, int Weight)>[] _adjacency;

    public Graph(int vertexCount, bool isDirected)
    {
        ArgumentOutOfRangeException.ThrowIfNegative(vertexCount);
        IsDirected = isDirected;
        _adjacency = new List<(int To, int Weight)>[vertexCount];
        for (int i = 0; i < vertexCount; i++)
        {
            _adjacency[i] = [];
        }
    }

    public bool IsDirected { get; }

    public int VertexCount => _adjacency.Length;

    /// <summary>Кількість логічних ребер (неорієнтоване ребро рахуємо один раз).</summary>
    public int EdgeCount { get; private set; }

    public void AddEdge(int from, int to, int weight = 1)
    {
        ValidateVertex(from);
        ValidateVertex(to);
        _adjacency[from].Add((to, weight));
        if (!IsDirected)
        {
            _adjacency[to].Add((from, weight));
        }
        EdgeCount++;
    }

    public IEnumerable<int> Neighbors(int vertex)
    {
        ValidateVertex(vertex);
        foreach (var (to, _) in _adjacency[vertex])
        {
            yield return to;
        }
    }

    public IReadOnlyList<(int To, int Weight)> WeightedNeighbors(int vertex)
    {
        ValidateVertex(vertex);
        return _adjacency[vertex];
    }

    public bool HasEdge(int from, int to) => Neighbors(from).Contains(to);

    public override string ToString()
    {
        var lines = new List<string>();
        for (int v = 0; v < VertexCount; v++)
        {
            lines.Add($"{v}: [{string.Join(", ", Neighbors(v))}]");
        }
        return string.Join(Environment.NewLine, lines);
    }

    private void ValidateVertex(int vertex, [System.Runtime.CompilerServices.CallerArgumentExpression(nameof(vertex))] string? name = null)
    {
        if ((uint)vertex >= (uint)_adjacency.Length)
        {
            throw new ArgumentOutOfRangeException(name, vertex, "Вершина поза межами графа.");
        }
    }
}
```

**Приклад запуску:**

```text
0: [1, 2]
1: [0, 3, 4]
2: [0, 4]
3: [1, 5]
4: [1, 2, 5]
5: [3, 4]
Вершин: 6, ребер: 7
Сусіди 4: 1, 2, 5
Є ребро 3-5: True; є ребро 5-3: True
Орграф: є 0→1: True, є 1→0: False
Помилка: to поза межами
```

### Типові помилки (представлення)

1. **Забули ініціалізувати `adjacency[i] = []`** → `NullReferenceException` на першому `Add`.
2. **Неорієнтоване ребро додали в один бік** → BFS «не бачить» половину графа.
3. **Матриця на 10⁵ вершин** → `OutOfMemoryException`. Рахуйте `n²` байт заздалегідь.
4. **Нумерація з 1 у вхідних даних** (`1..n`) → або масив `n + 1`, або `u - 1` при читанні.
5. **Паралельні ребра й петлі** — у мультиграфі `Contains` не скаже, скільки ребер; у задачах
   на MST/мости паралельні ребра змінюють відповідь.

### Міні-вправа 2

Напишіть функцію, яка перетворює матрицю суміжності `bool[,]` на список ребер
неорієнтованого графа (кожне ребро — один раз).

<details>
<summary>Розв'язок</summary>

```csharp
// Приклад 2.5 (розв'язок): матриця → список ребер
bool[,] matrix =
{
    { false, true,  true,  false },
    { true,  false, false, true  },
    { true,  false, false, true  },
    { false, true,  true,  false },
};

List<(int U, int V)> ToEdgeList(bool[,] m)
{
    var result = new List<(int U, int V)>();
    int n = m.GetLength(0);
    for (int u = 0; u < n; u++)
    {
        // Починаємо з v = u + 1: беремо лише верхній трикутник, щоб не дублювати ребра
        for (int v = u + 1; v < n; v++)
        {
            if (m[u, v])
            {
                result.Add((u, v));
            }
        }
    }
    return result;
}

Console.WriteLine(string.Join(" ", ToEdgeList(matrix)));
```

```text
(0, 1) (0, 2) (1, 3) (2, 3)
```
</details>

---

## 3. Обхід у ширину (BFS)

### 3.1 Ідея: хвиля від каменя у воді

BFS (Breadth-First Search) відвідує вершини **шарами**: спочатку сама `s`, потім усі
на відстані 1, потім 2 і т. д. Інструмент — **черга** (FIFO).

```text
Старт = 0                         Шари BFS:

      0 ───── 1 ───── 3           шар 0: {0}
      │       │       │           шар 1: {1, 2}
      │       │       │           шар 2: {3, 4}
      2 ───── 4 ───── 5           шар 3: {5}

Черга по кроках:
  [0]  → дістали 0, додали 1, 2          → [1, 2]
  [1, 2] → дістали 1, додали 3, 4        → [2, 3, 4]
  [2, 3, 4] → дістали 2 (4 вже відкрита) → [3, 4]
  [3, 4] → дістали 3, додали 5           → [4, 5]
  [4, 5] → дістали 4 (5 вже відкрита)    → [5]
  [5] → дістали 5                        → []
```

**Ключова властивість.** У **незваженому** графі BFS знаходить **найкоротші** відстані
(у кількості ребер). Чому: вершина з відстанню `d + 1` може потрапити в чергу лише
від вершини з відстанню `d`, а черга обробляє всі вершини шару `d` раніше за шар `d + 1`.

**Складність:** `O(n + m)` часу (кожна вершина в черзі один раз, кожне ребро переглядається
1 раз в орграфі, 2 рази в неорієнтованому), `O(n)` пам'яті.

### 3.2 BFS: відстані, батьки, відновлення шляху

```csharp
// Приклад 3.1: BFS з відстанями, масивом батьків і відновленням шляху
int[][] adjacency = BuildSampleGraph();
var (distance, parent) = Bfs(adjacency, source: 0);

for (int v = 0; v < adjacency.Length; v++)
{
    Console.WriteLine($"dist[{v}] = {distance[v]}, parent[{v}] = {parent[v]}");
}

Console.WriteLine($"Шлях 0 → 5: {string.Join(" → ", RestorePath(parent, 5))}");
Console.WriteLine($"Шлях 0 → 4: {string.Join(" → ", RestorePath(parent, 4))}");

static (int[] Distance, int[] Parent) Bfs(int[][] adjacency, int source)
{
    int n = adjacency.Length;
    var distance = new int[n];
    var parent = new int[n];
    Array.Fill(distance, -1); // -1 = «ще не відкрита»; одночасно замінює масив visited
    Array.Fill(parent, -1);

    var queue = new Queue<int>();
    distance[source] = 0;
    queue.Enqueue(source);

    while (queue.Count > 0)
    {
        int u = queue.Dequeue();
        foreach (int v in adjacency[u])
        {
            if (distance[v] != -1)
            {
                continue; // вже відкрита раніше — її відстань вже найкоротша
            }
            // ВАЖЛИВО: позначаємо при ДОДАВАННІ в чергу, а не при вийманні,
            // інакше одна вершина може потрапити в чергу кілька разів
            distance[v] = distance[u] + 1;
            parent[v] = u;
            queue.Enqueue(v);
        }
    }

    return (distance, parent);
}

static List<int> RestorePath(int[] parent, int target)
{
    var path = new List<int>();
    // Йдемо від цілі до старту по батьках; у старту parent = -1
    for (int v = target; v != -1; v = parent[v])
    {
        path.Add(v);
    }
    path.Reverse(); // зібрали задом наперед — розвертаємо
    return path;
}

static int[][] BuildSampleGraph()
{
    (int U, int V)[] edges = [(0, 1), (0, 2), (1, 3), (1, 4), (2, 4), (3, 5), (4, 5)];
    var lists = Enumerable.Range(0, 6).Select(_ => new List<int>()).ToArray();
    foreach (var (u, v) in edges)
    {
        lists[u].Add(v);
        lists[v].Add(u);
    }
    return lists.Select(list => list.ToArray()).ToArray();
}
```

**Приклад запуску:**

```text
dist[0] = 0, parent[0] = -1
dist[1] = 1, parent[1] = 0
dist[2] = 1, parent[2] = 0
dist[3] = 2, parent[3] = 1
dist[4] = 2, parent[4] = 1
dist[5] = 3, parent[5] = 3
Шлях 0 → 5: 0 → 1 → 3 → 5
Шлях 0 → 4: 0 → 1 → 4
```

> Якщо цільова вершина недосяжна (`distance[target] == -1`), `RestorePath` поверне
> лише `[target]`. У реальному коді спочатку перевіряйте досяжність.

### 3.3 BFS по шарах

Іноді потрібні саме **шари** (рівні дерева, «друзі другого кола»). Трюк: на початку
кожної ітерації зовнішнього циклу в черзі лежить рівно один шар — запам'ятовуємо `queue.Count`.

```csharp
// Приклад 3.2: BFS пошарово
(int U, int V)[] edges = [(0, 1), (0, 2), (1, 3), (1, 4), (2, 4), (3, 5), (4, 5)];
var adjacency = Enumerable.Range(0, 6).Select(_ => new List<int>()).ToArray();
foreach (var (u, v) in edges)
{
    adjacency[u].Add(v);
    adjacency[v].Add(u);
}

var visited = new bool[adjacency.Length];
var queue = new Queue<int>();
queue.Enqueue(0);
visited[0] = true;
int level = 0;

while (queue.Count > 0)
{
    int layerSize = queue.Count; // фіксуємо розмір ПОТОЧНОГО шару
    var layer = new List<int>(layerSize);
    for (int i = 0; i < layerSize; i++)
    {
        int u = queue.Dequeue();
        layer.Add(u);
        foreach (int v in adjacency[u])
        {
            if (!visited[v])
            {
                visited[v] = true;
                queue.Enqueue(v); // потрапить у НАСТУПНИЙ шар
            }
        }
    }
    Console.WriteLine($"Шар {level}: {{{string.Join(", ", layer)}}}");
    level++;
}
```

**Приклад запуску:**

```text
Шар 0: {0}
Шар 1: {1, 2}
Шар 2: {3, 4}
Шар 3: {5}
```

### 3.4 BFS на сітці (лабіринт)

```text
Лабіринт (S — старт, E — вихід, # — стіна):

  S . . # .
  # # . # .
  . . . . .
  . # # # .
  . . . # E
```

```csharp
// Приклад 3.3: найкоротший шлях у лабіринті BFS-ом
string[] maze =
[
    "S..#.",
    "##.#.",
    ".....",
    ".###.",
    "...#E",
];

int rows = maze.Length;
int cols = maze[0].Length;
// Чотири напрямки: вгору, вниз, вліво, вправо
(int Dr, int Dc)[] directions = [(-1, 0), (1, 0), (0, -1), (0, 1)];

var start = Find('S');
var exit = Find('E');

var distance = new int[rows, cols];
var parent = new (int R, int C)[rows, cols];
for (int r = 0; r < rows; r++)
{
    for (int c = 0; c < cols; c++)
    {
        distance[r, c] = -1;
    }
}

var queue = new Queue<(int R, int C)>();
distance[start.R, start.C] = 0;
queue.Enqueue(start);

while (queue.Count > 0)
{
    var (r, c) = queue.Dequeue();
    if ((r, c) == exit)
    {
        break; // BFS: перший раз дійшли до виходу — це вже найкоротше
    }
    foreach (var (dr, dc) in directions)
    {
        int nr = r + dr;
        int nc = c + dc;
        // Перевірки: у межах, не стіна, ще не відвідана
        if (nr < 0 || nr >= rows || nc < 0 || nc >= cols) continue;
        if (maze[nr][nc] == '#' || distance[nr, nc] != -1) continue;

        distance[nr, nc] = distance[r, c] + 1;
        parent[nr, nc] = (r, c);
        queue.Enqueue((nr, nc));
    }
}

Console.WriteLine($"Довжина шляху: {distance[exit.R, exit.C]}");

// Малюємо шлях зірочками
var canvas = maze.Select(line => line.ToCharArray()).ToArray();
for (var cell = parent[exit.R, exit.C]; cell != start; cell = parent[cell.R, cell.C])
{
    canvas[cell.R][cell.C] = '*';
}
foreach (var line in canvas)
{
    Console.WriteLine(string.Join(' ', line));
}

(int R, int C) Find(char symbol)
{
    for (int r = 0; r < rows; r++)
    {
        int c = maze[r].IndexOf(symbol);
        if (c >= 0)
        {
            return (r, c);
        }
    }
    throw new InvalidOperationException($"Символ {symbol} не знайдено");
}
```

**Приклад запуску:**

```text
Довжина шляху: 8
S * * # .
# # * # .
. . * * *
. # # # *
. . . # E
```

### 3.5 Multi-source BFS: «гниючі апельсини», найближча лікарня

Якщо стартів **кілька**, кладемо в чергу **всі одразу** з відстанню 0. Це еквівалентно
фіктивній «супервершині», з'єднаній з усіма стартами. Складність та сама — `O(n + m)`,
а не `k · O(n + m)`.

```csharp
// Приклад 3.4: відстань кожної клітинки до найближчої лікарні (H)
string[] city =
[
    "H....",
    ".##..",
    "....H",
    "..#..",
];

int rows = city.Length;
int cols = city[0].Length;
var distance = new int[rows, cols];
var queue = new Queue<(int R, int C)>();

for (int r = 0; r < rows; r++)
{
    for (int c = 0; c < cols; c++)
    {
        distance[r, c] = -1;
        if (city[r][c] == 'H')
        {
            distance[r, c] = 0;   // УСІ джерела стартують одночасно
            queue.Enqueue((r, c));
        }
    }
}

(int Dr, int Dc)[] directions = [(-1, 0), (1, 0), (0, -1), (0, 1)];
while (queue.Count > 0)
{
    var (r, c) = queue.Dequeue();
    foreach (var (dr, dc) in directions)
    {
        int nr = r + dr, nc = c + dc;
        if (nr < 0 || nr >= rows || nc < 0 || nc >= cols) continue;
        if (city[nr][nc] == '#' || distance[nr, nc] != -1) continue;
        distance[nr, nc] = distance[r, c] + 1;
        queue.Enqueue((nr, nc));
    }
}

for (int r = 0; r < rows; r++)
{
    var cells = new List<string>();
    for (int c = 0; c < cols; c++)
    {
        cells.Add(city[r][c] == '#' ? "#" : distance[r, c].ToString());
    }
    Console.WriteLine(string.Join(' ', cells));
}
```

**Приклад запуску:**

```text
0 1 2 3 2
1 # # 2 1
2 3 2 1 0
3 4 # 2 1
```

### 3.6 Перевірка на двочастковість (bipartite)

Граф **двочастковий**, якщо вершини можна розфарбувати у 2 кольори так, щоб кожне
ребро з'єднувало різні кольори. Еквівалентно: **немає циклів непарної довжини**.

BFS фарбує шар `d` у колір `d % 2`. Конфлікт — ребро між однаковими кольорами.

```text
G (цикл 0-1-4-2-0 довжини 4, цикл 1-3-5-4-1 довжини 4):    Трикутник:

  0(A) ── 1(B) ── 3(A)                                        0 ── 1
   │       │       │                                           ╲  ╱
  2(B) ── 4(A) ── 5(B)       → двочастковий                     2     → ні
```

```csharp
// Приклад 3.5: перевірка двочастковості BFS-ом (враховує незв'язні графи)
var sample = BuildUndirected(6, [(0, 1), (0, 2), (1, 3), (1, 4), (2, 4), (3, 5), (4, 5)]);
var triangle = BuildUndirected(3, [(0, 1), (1, 2), (2, 0)]);
var forest = BuildUndirected(5, [(0, 1), (2, 3), (3, 4)]); // дві компоненти

Report("G", sample);
Report("Трикутник", triangle);
Report("Ліс", forest);

static void Report(string name, List<int>[] adjacency)
{
    int[]? colors = TryColor(adjacency);
    Console.WriteLine(colors is null
        ? $"{name}: не двочастковий"
        : $"{name}: двочастковий, кольори = [{string.Join(", ", colors)}]");
}

static int[]? TryColor(List<int>[] adjacency)
{
    int n = adjacency.Length;
    var color = new int[n];
    Array.Fill(color, -1);

    // Зовнішній цикл: граф може бути незв'язним — запускаємо BFS з кожної нефарбованої вершини
    for (int start = 0; start < n; start++)
    {
        if (color[start] != -1) continue;
        color[start] = 0;
        var queue = new Queue<int>([start]);
        while (queue.Count > 0)
        {
            int u = queue.Dequeue();
            foreach (int v in adjacency[u])
            {
                if (color[v] == -1)
                {
                    color[v] = 1 - color[u]; // протилежний колір
                    queue.Enqueue(v);
                }
                else if (color[v] == color[u])
                {
                    return null; // ребро між однаковими кольорами → непарний цикл
                }
            }
        }
    }
    return color;
}

static List<int>[] BuildUndirected(int n, (int U, int V)[] edges)
{
    var adjacency = Enumerable.Range(0, n).Select(_ => new List<int>()).ToArray();
    foreach (var (u, v) in edges)
    {
        adjacency[u].Add(v);
        adjacency[v].Add(u);
    }
    return adjacency;
}
```

**Приклад запуску:**

```text
G: двочастковий, кольори = [0, 1, 1, 0, 0, 1]
Трикутник: не двочастковий
Ліс: двочастковий, кольори = [0, 1, 0, 1, 0]
```

### 3.7 0-1 BFS: ваги лише 0 або 1

Якщо кожне ребро важить **0 або 1**, Дейкстра не потрібна. Використовуємо **дек**
(`LinkedList<int>` у .NET — стандартного `Deque<T>` немає):

- ребро ваги **0** → кладемо сусіда **на початок** (він у тому самому «шарі»);
- ребро ваги **1** → **в кінець**.

Складність `O(n + m)` замість `O((n + m) log n)`.

Класична задача: «мінімальна кількість стін, які треба зламати», «мінімум розворотів
ребер, щоб дістатися з 0 до n−1».

```text
Орграф, треба дійти з 0 до 5. Ребро за напрямком — вага 0,
проти напрямку (якщо «розвернути» ребро) — вага 1.

  0 ──► 1 ◄── 3
  │           ▲
  ▼           │
  2 ──► 4 ◄── 5
```

```csharp
// Приклад 3.6: 0-1 BFS — мінімум розворотів ребер, щоб дійти 0 → 5
const int N = 6;
(int From, int To)[] directedEdges = [(0, 1), (3, 1), (0, 2), (5, 3), (2, 4), (5, 4)];

var adjacency = Enumerable.Range(0, N).Select(_ => new List<(int To, int Weight)>()).ToArray();
foreach (var (from, to) in directedEdges)
{
    adjacency[from].Add((to, 0)); // рух за напрямком — безкоштовно
    adjacency[to].Add((from, 1)); // рух проти напрямку — треба розвернути ребро
}

var distance = new int[N];
Array.Fill(distance, int.MaxValue);
distance[0] = 0;

var deque = new LinkedList<int>();
deque.AddFirst(0);

while (deque.Count > 0)
{
    int u = deque.First!.Value;
    deque.RemoveFirst();
    foreach (var (v, w) in adjacency[u])
    {
        if (distance[u] + w < distance[v])
        {
            distance[v] = distance[u] + w;
            if (w == 0)
            {
                deque.AddFirst(v); // та сама відстань — обробити якомога раніше
            }
            else
            {
                deque.AddLast(v);
            }
        }
    }
}

Console.WriteLine($"Відстані (розвороти): [{string.Join(", ", distance)}]");
Console.WriteLine($"Мінімум розворотів 0 → 5: {distance[5]}");
```

**Приклад запуску:**

```text
Відстані (розвороти): [0, 0, 0, 1, 0, 1]
Мінімум розворотів 0 → 5: 1
```

> На відміну від звичайного BFS, у 0-1 BFS вершина **може** потрапити в дек кілька разів
> (коли її відстань покращилась). Тому перевіряємо `distance[u] + w < distance[v]`,
> а не `visited`.

### Типові помилки (BFS)

1. **`visited` ставлять при вийманні з черги** → вершина додається в чергу багато разів,
   на щільних графах час вибухає до `O(m)` елементів у черзі та повторної роботи.
2. **BFS для зважених графів** → відповідь неправильна; потрібна Дейкстра (або 0-1 BFS).
3. **Використання `Stack<T>` замість `Queue<T>`** → вийде не BFS, відстані некоректні.
4. **Сітка: переплутані `rows`/`cols`** або перевірка меж після звернення до масиву.
5. **Незв'язний граф**: один запуск BFS обходить лише компоненту старту.

### Міні-вправа 3

У соцмережі (граф G) знайдіть усіх «друзів другого кола» вершини 0 — тих, до кого
відстань рівно 2.

<details>
<summary>Розв'язок</summary>

Запускаємо BFS з 0 і беремо вершини з `distance == 2`. Для G це `{3, 4}`: вершина 4 —
сусідка і 1, і 2, але відстань до неї все одно 2. Можна зупинити BFS, щойно
`distance[u] == 2` для вийнятої вершини, — далі нічого корисного не буде.
</details>

---

## ☕ Перерва 1 (≈ 50 хв від початку)

---

## 4. Обхід у глибину (DFS)

### 4.1 Ідея: лабіринт з ниткою Аріадни

DFS (Depth-First Search) іде **якомога глибше**, а коли далі нікуди — **повертається**
(backtracking) до останньої вершини, де лишились невідвідані сусіди. Інструмент —
**стек** (явний `Stack<T>` або стек викликів при рекурсії).

```text
Старт = 0, сусідів перебираємо за зростанням:

      0 ───── 1 ───── 3          Порядок входу:  0, 1, 3, 5, 4, 2
      │       │       │
      │       │       │          Дерево DFS:  0
      2 ───── 4 ───── 5                       └─ 1
                                                 └─ 3
                                                    └─ 5
                                                       └─ 4
                                                          └─ 2
```

DFS **не** дає найкоротших відстаней, зате дає **структуру**: часи входу/виходу,
дерево обходу, класифікацію ребер. На цьому побудовані топосорт, пошук циклів, SCC,
мости й точки зчленування.

**Складність:** `O(n + m)` часу, `O(n)` пам'яті (глибина рекурсії до `n`!).

### 4.2 Рекурсивний DFS із часами входу та виходу

`tin[v]` — момент, коли ми **увійшли** у `v` (пофарбували в сірий),
`tout[v]` — коли **вийшли** (усі нащадки оброблені, вершина стала чорною).

**Лема про дужки.** Для будь-яких `u`, `v` інтервали `[tin, tout]` або вкладені
(один — предок іншого в дереві DFS), або не перетинаються. Звідси перевірка
«`u` — предок `v`» за `O(1)`: `tin[u] <= tin[v] && tout[v] <= tout[u]`.

```csharp
// Приклад 4.1: рекурсивний DFS з часами входу/виходу та деревом обходу
var adjacency = BuildUndirected(6, [(0, 1), (0, 2), (1, 3), (1, 4), (2, 4), (3, 5), (4, 5)]);
int n = adjacency.Length;
var tin = new int[n];
var tout = new int[n];
var parent = new int[n];
var visited = new bool[n];
Array.Fill(parent, -1);
int timer = 0;
var order = new List<int>();

Dfs(0, depth: 0);

Console.WriteLine($"Порядок входу: {string.Join(", ", order)}");
for (int v = 0; v < n; v++)
{
    Console.WriteLine($"v={v}: tin={tin[v],2}, tout={tout[v],2}, parent={parent[v],2}");
}

bool IsAncestor(int u, int v) => tin[u] <= tin[v] && tout[v] <= tout[u];
Console.WriteLine($"1 предок 4? {IsAncestor(1, 4)}; 2 предок 3? {IsAncestor(2, 3)}");

void Dfs(int u, int depth)
{
    visited[u] = true;
    tin[u] = timer++;
    order.Add(u);
    Console.WriteLine($"{new string(' ', depth * 2)}→ вхід у {u}");
    foreach (int v in adjacency[u])
    {
        if (!visited[v])
        {
            parent[v] = u;
            Dfs(v, depth + 1);
        }
    }
    tout[u] = timer++;
    Console.WriteLine($"{new string(' ', depth * 2)}← вихід з {u}");
}

static List<int>[] BuildUndirected(int n, (int U, int V)[] edges)
{
    var adjacency = Enumerable.Range(0, n).Select(_ => new List<int>()).ToArray();
    foreach (var (u, v) in edges)
    {
        adjacency[u].Add(v);
        adjacency[v].Add(u);
    }
    // Сортуємо сусідів — щоб порядок обходу був передбачуваним
    foreach (var list in adjacency)
    {
        list.Sort();
    }
    return adjacency;
}
```

**Приклад запуску:**

```text
→ вхід у 0
  → вхід у 1
    → вхід у 3
      → вхід у 5
        → вхід у 4
          → вхід у 2
          ← вихід з 2
        ← вихід з 4
      ← вихід з 5
    ← вихід з 3
  ← вихід з 1
← вихід з 0
Порядок входу: 0, 1, 3, 5, 4, 2
v=0: tin= 0, tout=11, parent=-1
v=1: tin= 1, tout=10, parent= 0
v=2: tin= 5, tout= 6, parent= 4
v=3: tin= 2, tout= 9, parent= 1
v=4: tin= 4, tout= 7, parent= 5
v=5: tin= 3, tout= 8, parent= 3
1 предок 4? True; 2 предок 3? False
```

### 4.3 Ітеративний DFS

Рекурсія в C# на графі з 10⁵–10⁶ вершин (наприклад, «ланцюжок») призводить до
`StackOverflowException`, який **неможливо перехопити** — процес просто падає.
Рішення — явний стек.

Наївний варіант «поклав усіх сусідів у стек» дає порядок, **схожий** на DFS, але
неправильні часи виходу. Правильна імітація рекурсії — зберігати в стеку пару
**(вершина, індекс наступного сусіда)**.

```csharp
// Приклад 4.2: ітеративний DFS, що точно імітує рекурсію (однакові tin/tout)
(int U, int V)[] edges = [(0, 1), (0, 2), (1, 3), (1, 4), (2, 4), (3, 5), (4, 5)];
const int N = 6;
var adjacency = Enumerable.Range(0, N).Select(_ => new List<int>()).ToArray();
foreach (var (u, v) in edges)
{
    adjacency[u].Add(v);
    adjacency[v].Add(u);
}
foreach (var list in adjacency) list.Sort();

var tin = new int[N];
var tout = new int[N];
var visited = new bool[N];
int timer = 0;

// Стек кадрів: (вершина, скільки сусідів уже переглянуто)
var stack = new Stack<(int Vertex, int NextIndex)>();
visited[0] = true;
tin[0] = timer++;
stack.Push((0, 0));

while (stack.Count > 0)
{
    var (u, index) = stack.Pop();
    if (index < adjacency[u].Count)
    {
        // Повертаємо кадр назад з просунутим індексом — «продовжимо цикл foreach пізніше»
        stack.Push((u, index + 1));
        int v = adjacency[u][index];
        if (!visited[v])
        {
            visited[v] = true;
            tin[v] = timer++;
            stack.Push((v, 0)); // «рекурсивний виклик»
        }
    }
    else
    {
        tout[u] = timer++; // усі сусіди оброблені — «повернення з функції»
    }
}

for (int v = 0; v < N; v++)
{
    Console.WriteLine($"v={v}: tin={tin[v],2}, tout={tout[v],2}");
}
```

**Приклад запуску:**

```text
v=0: tin= 0, tout=11
v=1: tin= 1, tout=10
v=2: tin= 5, tout= 6
v=3: tin= 2, tout= 9
v=4: tin= 4, tout= 7
v=5: tin= 3, tout= 8
```

> Порівняйте з прикладом 4.1: часи **ідентичні**. Це і є перевірка, що ітеративна версія
> коректна.

### 4.4 Класифікація ребер (орієнтований граф)

Під час DFS кожне ребро `u → v` орграфа належить до одного з 4 типів.
Використовуємо **три кольори**: білий (0) — не відвідана, сірий (1) — у стеку рекурсії,
чорний (2) — завершена.

| Тип | Умова в момент перегляду `u → v` | Сенс |
|---|---|---|
| **Tree** (деревне) | `v` біла | ребро дерева DFS |
| **Back** (зворотне) | `v` сіра | веде до предка → **цикл!** |
| **Forward** (пряме) | `v` чорна і `tin[u] < tin[v]` | до нащадка, але не деревне |
| **Cross** (поперечне) | `v` чорна і `tin[u] > tin[v]` | між гілками / деревами |

У **неорієнтованому** графі бувають лише tree та back ребра.

```text
Орграф для класифікації:

   0 ──► 1 ──► 2
   │     ▲     │
   │     └─────┘  (2 → 1: back, цикл 1→2→1)
   ▼
   3 ──► 2        (3 → 2: cross)
   0 ──► 2        (0 → 2: forward)
```

```csharp
// Приклад 4.3: класифікація ребер орграфа за кольорами та часом входу
const int N = 4;
(int From, int To)[] edges = [(0, 1), (0, 2), (0, 3), (1, 2), (2, 1), (3, 2)];
var adjacency = Enumerable.Range(0, N).Select(_ => new List<int>()).ToArray();
foreach (var (from, to) in edges)
{
    adjacency[from].Add(to);
}

var color = new int[N]; // 0 — білий, 1 — сірий, 2 — чорний
var tin = new int[N];
int timer = 0;

for (int v = 0; v < N; v++)
{
    if (color[v] == 0) Dfs(v);
}

void Dfs(int u)
{
    color[u] = 1;
    tin[u] = timer++;
    foreach (int v in adjacency[u])
    {
        string kind = color[v] switch
        {
            0 => "tree",
            1 => "back (цикл!)",
            _ => tin[u] < tin[v] ? "forward" : "cross",
        };
        Console.WriteLine($"{u} → {v}: {kind}");
        if (color[v] == 0)
        {
            Dfs(v);
        }
    }
    color[u] = 2;
}
```

**Приклад запуску:**

```text
0 → 1: tree
1 → 2: tree
2 → 1: back (цикл!)
0 → 2: forward
0 → 3: tree
3 → 2: cross
```

### 4.5 Компоненти зв'язності

Запускаємо DFS (або BFS) з кожної ще не відвіданої вершини — кожен запуск відкриває
рівно одну компоненту. Сумарно `O(n + m)`.

```text
Граф із трьома компонентами:

  0 ── 1      3 ── 4      6
  │  ╱        │
  2           5
```

```csharp
// Приклад 4.4: компоненти зв'язності (ітеративний DFS зі стеком)
const int N = 7;
(int U, int V)[] edges = [(0, 1), (1, 2), (2, 0), (3, 4), (3, 5)];
var adjacency = Enumerable.Range(0, N).Select(_ => new List<int>()).ToArray();
foreach (var (u, v) in edges)
{
    adjacency[u].Add(v);
    adjacency[v].Add(u);
}

var component = new int[N];
Array.Fill(component, -1);
int count = 0;

for (int start = 0; start < N; start++)
{
    if (component[start] != -1) continue;

    // Новий запуск = нова компонента
    var stack = new Stack<int>();
    stack.Push(start);
    component[start] = count;
    while (stack.Count > 0)
    {
        int u = stack.Pop();
        foreach (int v in adjacency[u])
        {
            if (component[v] == -1)
            {
                component[v] = count; // позначаємо при додаванні, як у BFS
                stack.Push(v);
            }
        }
    }
    count++;
}

Console.WriteLine($"Кількість компонент: {count}");
foreach (var group in Enumerable.Range(0, N).GroupBy(v => component[v]))
{
    Console.WriteLine($"  Компонента {group.Key}: {{{string.Join(", ", group)}}}");
}
```

**Приклад запуску:**

```text
Кількість компонент: 3
  Компонента 0: {0, 1, 2}
  Компонента 1: {3, 4, 5}
  Компонента 2: {6}
```

> Для порядку «для кожної компоненти» не важливо, як саме ми обходимо: BFS, «стековий
> псевдо-DFS» чи рекурсія дадуть ту саму множину вершин.

### 4.6 Пошук циклу: неорієнтований граф (через батька)

У неорієнтованому графі ребро до **вже відвіданої** вершини, яка **не є батьком**, —
це цикл. Батька виключаємо, бо ребро `u-parent` ми щойно пройшли в інший бік.

```csharp
// Приклад 4.5: цикл у неорієнтованому графі + відновлення самого циклу
var sample = BuildUndirected(6, [(0, 1), (0, 2), (1, 3), (1, 4), (2, 4), (3, 5), (4, 5)]);
var tree = BuildUndirected(5, [(0, 1), (0, 2), (1, 3), (1, 4)]);

PrintCycle("G", sample);
PrintCycle("Дерево", tree);

static void PrintCycle(string name, List<int>[] adjacency)
{
    int n = adjacency.Length;
    var parent = new int[n];
    var visited = new bool[n];
    Array.Fill(parent, -1);
    List<int>? cycle = null;

    for (int s = 0; s < n && cycle is null; s++)
    {
        if (!visited[s]) Dfs(s, -1);
    }

    Console.WriteLine(cycle is null
        ? $"{name}: циклів немає"
        : $"{name}: цикл {string.Join(" - ", cycle)}");

    void Dfs(int u, int from)
    {
        visited[u] = true;
        parent[u] = from;
        foreach (int v in adjacency[u])
        {
            if (cycle is not null) return;
            if (v == from) continue; // ребро назад до батька — не цикл
            if (visited[v])
            {
                // Знайшли зворотне ребро u - v: піднімаємось від u по батьках до v
                cycle = [];
                for (int x = u; x != v; x = parent[x]) cycle.Add(x);
                cycle.Add(v);
                return;
            }
            Dfs(v, u);
        }
    }
}

static List<int>[] BuildUndirected(int n, (int U, int V)[] edges)
{
    var adjacency = Enumerable.Range(0, n).Select(_ => new List<int>()).ToArray();
    foreach (var (u, v) in edges)
    {
        adjacency[u].Add(v);
        adjacency[v].Add(u);
    }
    return adjacency;
}
```

**Приклад запуску:**

```text
G: цикл 4 - 5 - 3 - 1
Дерево: циклів немає
```

> ⚠️ Правило «пропусти батька» ламається в **мультиграфі**: два паралельні ребра `u-v`
> утворюють цикл довжини 2. Тоді пропускають не вершину-батька, а **ідентифікатор ребра**.

### 4.7 Пошук циклу: орієнтований граф (три кольори)

В орграфі «відвідана вершина» ще не означає цикл (див. cross-ребра). Цикл є тоді й лише
тоді, коли DFS знаходить **back-ребро** — ребро до **сірої** вершини.

```text
   0 ──► 1 ──► 2          без циклу: 0→1→2, 0→2
   └───────────▲

   0 ──► 1 ──► 2          з циклом: 1→2→3→1
         ▲     │
         └─ 3 ◄┘
```

```csharp
// Приклад 4.6: цикл в орграфі — білий/сірий/чорний
Console.WriteLine(FindCycle(3, [(0, 1), (1, 2), (0, 2)]) is { } c1 ? $"Цикл: {c1}" : "Ациклічний");
Console.WriteLine(FindCycle(4, [(0, 1), (1, 2), (2, 3), (3, 1)]) is { } c2 ? $"Цикл: {c2}" : "Ациклічний");

static string? FindCycle(int n, (int From, int To)[] edges)
{
    var adjacency = Enumerable.Range(0, n).Select(_ => new List<int>()).ToArray();
    foreach (var (from, to) in edges) adjacency[from].Add(to);

    var color = new VertexColor[n];
    var parent = new int[n];
    string? found = null;

    for (int s = 0; s < n && found is null; s++)
    {
        if (color[s] == VertexColor.White) Dfs(s);
    }
    return found;

    void Dfs(int u)
    {
        color[u] = VertexColor.Gray; // «зараз у стеку рекурсії»
        foreach (int v in adjacency[u])
        {
            if (found is not null) return;
            if (color[v] == VertexColor.White)
            {
                parent[v] = u;
                Dfs(v);
            }
            else if (color[v] == VertexColor.Gray)
            {
                // Back-ребро u → v: цикл v → ... → u → v
                var cycle = new List<int> { v };
                for (int x = u; x != v; x = parent[x]) cycle.Add(x);
                cycle.Add(v);
                cycle.Reverse(1, cycle.Count - 2); // впорядкувати середину в напрямку ребер
                found = string.Join(" → ", cycle);
                return;
            }
            // Чорна вершина: вже повністю досліджена, циклу через неї немає
        }
        color[u] = VertexColor.Black;
    }
}

enum VertexColor { White, Gray, Black }
```

**Приклад запуску:**

```text
Ациклічний
Цикл: 1 → 2 → 3 → 1
```

### 4.8 Flood fill: кількість островів

Класика співбесід (LeetCode 200). Сітка `'1'` — суша, `'0'` — вода. Кожен запуск
заливки з нової клітинки суші — новий острів. Позначаємо відвідане, **змінюючи
саму сітку** (або окремим `bool[,]`, якщо вхід змінювати не можна).

```text
  1 1 0 0 0        A A . . .      A — 4 клітинки
  1 1 0 0 1   →    A A . . B      B — 1 клітинка (діагональ НЕ з'єднує!)
  0 0 1 0 0        . . C . .      C — 1 клітинка
  0 0 0 1 1        . . . D D      D — 2 клітинки
```

```csharp
// Приклад 4.7: кількість островів і розмір кожного (ітеративна заливка)
char[][] grid =
[
    "11000".ToCharArray(),
    "11001".ToCharArray(),
    "00100".ToCharArray(),
    "00011".ToCharArray(),
];

int rows = grid.Length, cols = grid[0].Length;
var sizes = new List<int>();

for (int r = 0; r < rows; r++)
{
    for (int c = 0; c < cols; c++)
    {
        if (grid[r][c] == '1')
        {
            sizes.Add(Fill(r, c));
        }
    }
}

Console.WriteLine($"Островів: {sizes.Count}");
Console.WriteLine($"Розміри: {string.Join(", ", sizes)}");
Console.WriteLine("Сітка після заливки (# — залите):");
foreach (var row in grid) Console.WriteLine(new string(row));

int Fill(int startR, int startC)
{
    int size = 0;
    var stack = new Stack<(int R, int C)>();
    stack.Push((startR, startC));
    grid[startR][startC] = '#'; // позначаємо ОДРАЗУ, щоб не додати двічі
    while (stack.Count > 0)
    {
        var (r, c) = stack.Pop();
        size++;
        foreach (var (nr, nc) in new[] { (r - 1, c), (r + 1, c), (r, c - 1), (r, c + 1) })
        {
            if (nr >= 0 && nr < rows && nc >= 0 && nc < cols && grid[nr][nc] == '1')
            {
                grid[nr][nc] = '#';
                stack.Push((nr, nc));
            }
        }
    }
    return size;
}
```

**Приклад запуску:**

```text
Островів: 4
Розміри: 4, 1, 1, 2
Сітка після заливки (# — залите):
##000
##00#
00#00
000##
```

### BFS vs DFS — коротке порівняння

| Критерій | BFS | DFS |
|---|---|---|
| Структура даних | черга | стек / рекурсія |
| Найкоротший шлях (незважений) | ✅ | ❌ |
| Пам'ять | ширина графа (може бути `O(n)`) | глибина (може бути `O(n)`) |
| Часи входу/виходу, класифікація ребер | ❌ | ✅ |
| Топосорт, SCC, мости | Кан (BFS-подібний) для топосорту | ✅ все |
| Компоненти, двочастковість, flood fill | ✅ | ✅ |

### Типові помилки (DFS)

1. **Рекурсія на великому графі** → `StackOverflowException` (не ловиться!). Для `n > ~10⁴`
   у «ланцюжкових» графах — ітеративний варіант або окремий `Thread` з великим стеком.
2. **Цикл в орграфі через `visited`** замість кольорів → хибні цикли на cross/forward ребрах.
3. **Цикл у неорієнтованому графі без виключення батька** → кожне ребро «цикл».
4. **Забули зовнішній цикл по всіх вершинах** → оброблено лише одну компоненту.
5. **Наївний стековий DFS для tout** → часи виходу неправильні (див. 4.3).

### Міні-вправа 4

Чи можна за допомогою часів `tin`/`tout` за `O(1)` відповісти, чи лежить вершина `v`
у піддереві `u` в дереві DFS? А чи існує **шлях** `u → v` в орграфі?

<details>
<summary>Розв'язок</summary>

Піддерево — так: `tin[u] <= tin[v] && tout[v] <= tout[u]`. Досяжність у довільному
орграфі — **ні**: `v` може бути досяжною через cross-ребро, не лежачи в піддереві `u`.
Для загальної досяжності потрібна транзитивне замикання (Флойд–Воршелл на `bool`, `O(n³)`)
або BFS з кожної вершини.
</details>

---

## 5. Топологічне сортування

### 5.1 Постановка

**Топологічний порядок** DAG — лінійне впорядкування вершин, у якому для кожного ребра
`u → v` вершина `u` стоїть **раніше** за `v`. Існує **тоді й лише тоді**, коли граф
ациклічний. Порядок, як правило, не єдиний.

```text
DAG:                          Можливі порядки:
  0 ────► 1 ────► 3             0, 1, 2, 3, 4, 5
  │       │       │             0, 2, 1, 4, 3, 5
  ▼       ▼       ▼             0, 1, 3, 2, 4, 5
  2 ────► 4 ────► 5             ... (але 4 ніколи не раніше за 2)
```

Застосування: порядок збірки проєктів у `.sln`, міграції БД, розклад курсів,
обчислення формул в Excel, порядок ініціалізації DI-контейнера.

### 5.2 Алгоритм Кана (BFS за вхідними степенями)

1. Порахувати `inDegree[v]`.
2. Покласти в чергу всі вершини з `inDegree == 0` (від них ніхто не залежить).
3. Дістаємо `u`, додаємо у відповідь, «видаляємо» його ребра: `inDegree[v]--`;
   якщо стало 0 — у чергу.
4. Якщо у відповіді менше за `n` вершин — **є цикл**.

```text
inDegree: 0:0  1:1  2:1  3:1  4:2  5:2

Крок  Черга     Взяли  Оновлення inDegree            Відповідь
 1    [0]        0     1:0 → в чергу, 2:0 → в чергу   0
 2    [1,2]      1     3:0 → в чергу, 4:1             0 1
 3    [2,3]      2     4:0 → в чергу                  0 1 2
 4    [3,4]      3     5:1                            0 1 2 3
 5    [4]        4     5:0 → в чергу                  0 1 2 3 4
 6    [5]        5     —                              0 1 2 3 4 5
```

```csharp
// Приклад 5.1: топологічне сортування — алгоритм Кана з виявленням циклу
var dag = new (int From, int To)[] { (0, 1), (0, 2), (1, 3), (1, 4), (2, 4), (3, 5), (4, 5) };
var cyclic = new (int From, int To)[] { (0, 1), (1, 2), (2, 3), (3, 1) };

Print("DAG", 6, dag);
Print("З циклом", 4, cyclic);

static void Print(string name, int n, (int From, int To)[] edges)
{
    List<int>? order = KahnSort(n, edges);
    Console.WriteLine(order is null
        ? $"{name}: топологічного порядку не існує (цикл)"
        : $"{name}: {string.Join(" ", order)}");
}

static List<int>? KahnSort(int n, (int From, int To)[] edges)
{
    var adjacency = Enumerable.Range(0, n).Select(_ => new List<int>()).ToArray();
    var inDegree = new int[n];
    foreach (var (from, to) in edges)
    {
        adjacency[from].Add(to);
        inDegree[to]++;
    }

    var queue = new Queue<int>();
    for (int v = 0; v < n; v++)
    {
        if (inDegree[v] == 0) queue.Enqueue(v);
    }

    var order = new List<int>(n);
    while (queue.Count > 0)
    {
        int u = queue.Dequeue();
        order.Add(u);
        foreach (int v in adjacency[u])
        {
            // «Видаляємо» ребро u → v
            if (--inDegree[v] == 0)
            {
                queue.Enqueue(v);
            }
        }
    }

    // Вершини циклу ніколи не отримають inDegree == 0 і не потраплять у відповідь
    return order.Count == n ? order : null;
}
```

**Приклад запуску:**

```text
DAG: 0 1 2 3 4 5
З циклом: топологічного порядку не існує (цикл)
```

> **Лексикографічно найменший** порядок: замініть `Queue<int>` на
> `PriorityQueue<int, int>` (пріоритет = номер вершини). Складність стане `O(m + n log n)`.

### 5.3 Топосорт через DFS

Вершину записуємо у список **у момент виходу** (`tout`). Коли ми виходимо з `u`, усі
вершини, досяжні з `u`, уже записані. Отже, **розвернутий** порядок виходу — топологічний.

```csharp
// Приклад 5.2: топосорт через DFS (post-order + reverse) з виявленням циклу
const int N = 6;
(int From, int To)[] edges = [(0, 1), (0, 2), (1, 3), (1, 4), (2, 4), (3, 5), (4, 5)];
var adjacency = Enumerable.Range(0, N).Select(_ => new List<int>()).ToArray();
foreach (var (from, to) in edges) adjacency[from].Add(to);

var state = new int[N]; // 0 — білий, 1 — сірий, 2 — чорний
var postOrder = new List<int>(N);
bool hasCycle = false;

for (int v = 0; v < N; v++)
{
    if (state[v] == 0) Dfs(v);
}

if (hasCycle)
{
    Console.WriteLine("Цикл — порядку немає");
}
else
{
    Console.WriteLine($"Порядок виходу:     {string.Join(" ", postOrder)}");
    postOrder.Reverse();
    Console.WriteLine($"Топологічний порядок: {string.Join(" ", postOrder)}");
}

void Dfs(int u)
{
    state[u] = 1;
    foreach (int v in adjacency[u])
    {
        if (state[v] == 1) { hasCycle = true; return; } // back-ребро
        if (state[v] == 0) Dfs(v);
    }
    state[u] = 2;
    postOrder.Add(u); // записуємо ПІСЛЯ всіх нащадків
}
```

**Приклад запуску:**

```text
Порядок виходу:     5 3 4 1 2 0
Топологічний порядок: 0 2 1 4 3 5
```

### 5.4 Розклад курсів (Course Schedule II)

Дано `numCourses` і пари `[course, prerequisite]` — «щоб узяти `course`, треба спершу
`prerequisite`». Повернути порядок проходження або повідомити, що це неможливо.

⚠️ Уважно з напрямком ребра: `prerequisite → course`.

```csharp
// Приклад 5.3: розклад курсів з назвами — Кан + пояснення, які курси «застрягли»
string[] courses = ["Програмування", "Дискретна математика", "Алгоритми", "Бази даних", "Компілятори"];
(string Course, string Requires)[] prerequisites =
[
    ("Алгоритми", "Програмування"),
    ("Алгоритми", "Дискретна математика"),
    ("Бази даних", "Програмування"),
    ("Компілятори", "Алгоритми"),
];

Plan(courses, prerequisites);
Console.WriteLine();
// Додаємо абсурдну вимогу: Програмування вимагає Компіляторів → цикл
Plan(courses, [.. prerequisites, ("Програмування", "Компілятори")]);

static void Plan(string[] courses, (string Course, string Requires)[] prerequisites)
{
    var index = courses.Select((name, i) => (name, i)).ToDictionary(p => p.name, p => p.i);
    int n = courses.Length;
    var adjacency = Enumerable.Range(0, n).Select(_ => new List<int>()).ToArray();
    var inDegree = new int[n];
    foreach (var (course, requires) in prerequisites)
    {
        adjacency[index[requires]].Add(index[course]); // спершу requires, потім course
        inDegree[index[course]]++;
    }

    var queue = new Queue<int>(Enumerable.Range(0, n).Where(v => inDegree[v] == 0));
    var semester = new List<string>();
    while (queue.Count > 0)
    {
        int u = queue.Dequeue();
        semester.Add(courses[u]);
        foreach (int v in adjacency[u])
        {
            if (--inDegree[v] == 0) queue.Enqueue(v);
        }
    }

    if (semester.Count == n)
    {
        Console.WriteLine("План: " + string.Join(" → ", semester));
    }
    else
    {
        var stuck = Enumerable.Range(0, n).Where(v => inDegree[v] > 0).Select(v => courses[v]);
        Console.WriteLine("Неможливо! Застрягли курси: " + string.Join(", ", stuck));
    }
}
```

**Приклад запуску:**

```text
План: Програмування → Дискретна математика → Бази даних → Алгоритми → Компілятори

Неможливо! Застрягли курси: Програмування, Алгоритми, Бази даних, Компілятори
```

> Застряглі курси — це вершини **циклу та все, що від циклу залежить**.

### Типові помилки (топосорт)

1. **Ребро в протилежний бік** (`course → prerequisite`) — отримаєте розвернутий порядок.
2. **DFS-топосорт без `Reverse()`** — порядок виходу навпаки.
3. **DFS-топосорт без перевірки сірих вершин** — на циклічному графі поверне «якийсь» порядок
   і не скаже про помилку.
4. **Кан без перевірки `order.Count == n`** — мовчки загубите вершини циклу.

### Міні-вправа 5

Як за допомогою алгоритму Кана знайти мінімальну кількість **семестрів**, якщо в семестр
можна брати скільки завгодно курсів, для яких виконані передумови?

<details>
<summary>Розв'язок</summary>

Обробляти чергу **пошарово** (як у прикладі 3.2): один шар = один семестр. Відповідь —
кількість шарів. Для прикладу 5.3: шар 1 {Програмування, Дискретна математика},
шар 2 {Алгоритми, Бази даних}, шар 3 {Компілятори} → 3 семестри. Це також довжина
найдовшого шляху в DAG (у вершинах).
</details>

---

## 6. Алгоритм Дейкстри

### 6.1 Ідея

Дано зважений граф з **невід'ємними** вагами і старт `s`. Потрібні найкоротші відстані
до всіх вершин.

Дейкстра — «BFS із пріоритетною чергою»: щоразу **фіксуємо** невідвідану вершину з
**найменшою** поточною відстанню. Чому це правильно: будь-який інший шлях до неї
проходить через вершину з більшою або рівною відстанню, а ваги не від'ємні — отже,
коротшим він стати не може.

**Релаксація** ребра `u → v` вагою `w`:
```text
if dist[u] + w < dist[v]:  dist[v] = dist[u] + w;  parent[v] = u
```

### 6.2 Трасування на зваженому G

```text
      0 ──7── 1 ──2── 3
      │       │       │
      2       1       3
      │       │       │
      2 ──3── 4 ──6── 5

Крок  Фіксуємо (dist)  Релаксації                          dist: 0  1  2  3  4  5
 0    —                                                           0  ∞  ∞  ∞  ∞  ∞
 1    0 (0)            1: 7, 2: 2                                 0  7  2  ∞  ∞  ∞
 2    2 (2)            4: 2+3=5                                   0  7  2  ∞  5  ∞
 3    4 (5)            1: 5+1=6 < 7 ✔, 5: 5+6=11                  0  6  2  ∞  5  11
 4    1 (6)            3: 6+2=8                                   0  6  2  8  5  11
 5    3 (8)            5: 8+3=11 (не краще)                       0  6  2  8  5  11
 6    5 (11)           —                                          0  6  2  8  5  11
```

Зверніть увагу на крок 3: пряме ребро `0-1` вагою 7 програє обхідному шляху `0-2-4-1` вагою 6.

### 6.3 Реалізація з `PriorityQueue<int, int>` і «лінивим видаленням»

`PriorityQueue<TElement, TPriority>` у .NET — це мін-купа, але **без операції
DecreaseKey**. Тому замість «зменшити пріоритет» ми просто **додаємо новий запис**, а
застарілі записи **пропускаємо** при вийманні (`if (d > dist[u]) continue`).
Це називається *lazy deletion*. У купі може бути до `O(m)` записів, тому складність
`O((n + m) log m) = O((n + m) log n)`.

```csharp
// Приклад 6.1: Дейкстра з PriorityQueue, лінивим видаленням і відновленням шляху
const int N = 6;
(int U, int V, int W)[] edges = [(0, 1, 7), (0, 2, 2), (1, 3, 2), (1, 4, 1), (2, 4, 3), (3, 5, 3), (4, 5, 6)];
var adjacency = Enumerable.Range(0, N).Select(_ => new List<(int To, int Weight)>()).ToArray();
foreach (var (u, v, w) in edges)
{
    adjacency[u].Add((v, w));
    adjacency[v].Add((u, w));
}

var (dist, parent) = Dijkstra(adjacency, source: 0);

for (int v = 0; v < N; v++)
{
    string path = string.Join(" → ", RestorePath(parent, v));
    Console.WriteLine($"до {v}: dist = {dist[v],2}, шлях {path}");
}

static (long[] Dist, int[] Parent) Dijkstra(List<(int To, int Weight)>[] adjacency, int source)
{
    int n = adjacency.Length;
    var dist = new long[n];          // long — щоб dist[u] + w не переповнилось
    var parent = new int[n];
    Array.Fill(dist, long.MaxValue);
    Array.Fill(parent, -1);

    var heap = new PriorityQueue<int, long>();
    dist[source] = 0;
    heap.Enqueue(source, 0);
    int popped = 0, skipped = 0;

    while (heap.TryDequeue(out int u, out long d))
    {
        popped++;
        if (d > dist[u])
        {
            skipped++;
            continue; // застарілий запис: ми вже знайшли кращу відстань до u
        }
        foreach (var (v, w) in adjacency[u])
        {
            long candidate = d + w;
            if (candidate < dist[v])
            {
                dist[v] = candidate;
                parent[v] = u;
                heap.Enqueue(v, candidate); // замість DecreaseKey — новий запис
            }
        }
    }

    Console.WriteLine($"Діставали з купи {popped} разів, з них застарілих: {skipped}");
    return (dist, parent);
}

static List<int> RestorePath(int[] parent, int target)
{
    var path = new List<int>();
    for (int v = target; v != -1; v = parent[v]) path.Add(v);
    path.Reverse();
    return path;
}
```

**Приклад запуску:**

```text
Діставали з купи 7 разів, з них застарілих: 1
до 0: dist =  0, шлях 0
до 1: dist =  6, шлях 0 → 2 → 4 → 1
до 2: dist =  2, шлях 0 → 2
до 3: dist =  8, шлях 0 → 2 → 4 → 1 → 3
до 4: dist =  5, шлях 0 → 2 → 4
до 5: dist = 11, шлях 0 → 2 → 4 → 5
```

> Застарілий запис тут — `(1, 7)`: його додали на кроці 1, а на кроці 3 до купи потрапив
> кращий `(1, 6)`.

### 6.4 Чому від'ємні ребра ламають Дейкстру

```text
   0 ──2──► 2 ──1──► 3        Правильно: dist[2] = 5 + (-4) = 1, dist[3] = 2.
   │        ▲
   5       -4                 Дейкстра фіксує 2 з dist = 2 (бо 2 < 5) і рахує dist[3] = 3.
   │        │                 Потім фіксує 1 (dist = 5) і бачить 5 - 4 = 1 < 2,
   ▼        │                 але 2 вже «зафіксована» — оновлення втрачено,
   1 ───────┘                 а неправильне dist[3] = 3 вже пораховано від неї.
```

Інваріант «зафіксована вершина має остаточну відстань» спирається на те, що продовження
шляху **не зменшує** довжину. Одне від'ємне ребро його руйнує.

```csharp
// Приклад 6.2: «класична» Дейкстра (з visited) помиляється на від'ємному ребрі
const int N = 4;
(int From, int To, int W)[] edges = [(0, 2, 2), (0, 1, 5), (1, 2, -4), (2, 3, 1)];
var adjacency = Enumerable.Range(0, N).Select(_ => new List<(int To, int W)>()).ToArray();
foreach (var (from, to, w) in edges) adjacency[from].Add((to, w));

var dist = new int[N];
Array.Fill(dist, int.MaxValue);
var done = new bool[N];
dist[0] = 0;
var heap = new PriorityQueue<int, int>();
heap.Enqueue(0, 0);

while (heap.TryDequeue(out int u, out _))
{
    if (done[u]) continue;
    done[u] = true; // «фіксуємо» — більше не оновлюємо
    foreach (var (v, w) in adjacency[u])
    {
        if (!done[v] && dist[u] + w < dist[v])
        {
            dist[v] = dist[u] + w;
            heap.Enqueue(v, dist[v]);
        }
    }
}

Console.WriteLine($"Дейкстра:   dist = [{string.Join(", ", dist)}]");
Console.WriteLine("Правильно:  dist = [0, 5, 1, 2]  (див. Беллман–Форд нижче)");
```

**Приклад запуску:**

```text
Дейкстра:   dist = [0, 5, 2, 3]
Правильно:  dist = [0, 5, 1, 2]  (див. Беллман–Форд нижче)
```

> «А якщо додати до всіх ваг константу, щоб вони стали додатні?» — **Не працює**: шлях з
> більшою кількістю ребер отримує більшу надбавку, і найкоротший шлях змінюється.
> Коректне перезважування — **алгоритм Джонсона** (потенціали з Беллмана–Форда).

### Типові помилки (Дейкстра)

1. **Немає перевірки `d > dist[u]`** → алгоритм правильний, але повільний (повторна обробка).
2. **`int.MaxValue + w`** → переповнення, від'ємна «відстань». Використовуйте `long` або
   перевіряйте `dist[u] != INF`.
3. **Від'ємні ваги** → неправильна відповідь без жодних попереджень.
4. **Зупинка на першому `Enqueue` цілі** замість першого `Dequeue` → не найкоротший шлях.
   Ранній вихід дозволено лише при **вийманні** цілі з купи.
5. **`PriorityQueue` плутають із max-heap** → у .NET це **min**-heap за замовчуванням.

### Міні-вправа 6

Як змінити Дейкстру, щоб знайти шлях з **максимальною надійністю**, якщо кожне ребро має
ймовірність безвідмовної роботи `p ∈ (0, 1]`, а надійність шляху — добуток ймовірностей?

<details>
<summary>Розв'язок</summary>

Варіант 1: максимізувати добуток — max-heap (пріоритет `-p`), релаксація
`rel[u] * p > rel[v]`. Добуток не зростає при подовженні шляху (`p ≤ 1`), тож жадібність
коректна. Варіант 2: ваги `-log p ≥ 0` — і звичайна Дейкстра на сумах.
</details>

---

## 7. Беллман–Форд і від'ємні цикли

### 7.1 Ідея

Найкоротший простий шлях має не більше `n − 1` ребер. Тому достатньо `n − 1` разів
прорелаксувати **всі** ребра: після `k`-ї ітерації правильні всі відстані, чиї
найкоротші шляхи мають ≤ `k` ребер.

Якщо на `n`-й ітерації щось **ще покращилося** — у графі є **від'ємний цикл**,
досяжний зі старту (найкоротшого шляху не існує — можна крутитися нескінченно).

**Складність:** `O(n · m)` часу, `O(n)` пам'яті. Працює зі **списком ребер**.

```text
Граф з від'ємним ребром (з 6.4):          Граф з від'ємним циклом 1 → 2 → 3 → 1:

   0 ──2──► 2 ──1──► 3                       0 ──1──► 1 ──1──► 2
   │        ▲                                         ▲        │
   5       -4                                        -3        1
   ▼        │                                         │        ▼
   1 ───────┘                                         └─────── 3
                                             сума циклу: 1 + 1 - 3 = -1 < 0
```

```csharp
// Приклад 7.1: Беллман–Форд — від'ємні ваги, ранній вихід і відновлення від'ємного циклу
Run("Від'ємне ребро", 4, [(0, 2, 2), (0, 1, 5), (1, 2, -4), (2, 3, 1)]);
Run("Від'ємний цикл", 4, [(0, 1, 1), (1, 2, 1), (2, 3, 1), (3, 1, -3)]);

static void Run(string name, int n, (int From, int To, int W)[] edges)
{
    const long Inf = long.MaxValue;
    var dist = new long[n];
    var parent = new int[n];
    Array.Fill(dist, Inf);
    Array.Fill(parent, -1);
    dist[0] = 0;

    int lastUpdated = -1;
    for (int iteration = 1; iteration <= n; iteration++)
    {
        lastUpdated = -1;
        foreach (var (u, v, w) in edges)
        {
            // dist[u] == Inf: з недосяжної вершини не релаксуємо (інакше Inf + w переповниться)
            if (dist[u] != Inf && dist[u] + w < dist[v])
            {
                dist[v] = dist[u] + w;
                parent[v] = u;
                lastUpdated = v;
            }
        }
        if (lastUpdated == -1)
        {
            Console.WriteLine($"{name}: стабілізувалось після ітерації {iteration}");
            break; // нічого не змінилось — далі теж не зміниться
        }
    }

    if (lastUpdated == -1)
    {
        Console.WriteLine($"  dist = [{string.Join(", ", dist)}]");
        return;
    }

    // Оновлення на n-й ітерації → від'ємний цикл.
    // lastUpdated може бути не НА циклі, а «за» ним; n кроків по parent гарантовано заводять у цикл.
    int x = lastUpdated;
    for (int i = 0; i < n; i++) x = parent[x];

    var cycle = new List<int> { x };
    for (int v = parent[x]; v != x; v = parent[v]) cycle.Add(v);
    cycle.Add(x);
    cycle.Reverse();
    Console.WriteLine($"{name}: знайдено від'ємний цикл {string.Join(" → ", cycle)}");
}
```

**Приклад запуску:**

```text
Від'ємне ребро: стабілізувалось після ітерації 2
  dist = [0, 5, 1, 2]
Від'ємний цикл: знайдено від'ємний цикл 3 → 1 → 2 → 3
```

### 7.2 Дейкстра vs Беллман–Форд

| | Дейкстра | Беллман–Форд |
|---|---|---|
| Від'ємні ребра | ❌ | ✅ |
| Від'ємні цикли | ❌ | ✅ виявляє |
| Складність | `O((n + m) log n)` | `O(n · m)` |
| Представлення | список суміжності | список ребер |
| Обмеження «≤ k ребер» | незручно | природно (`k` ітерацій з копією масиву) — див. задачу 16.4 |

### Типові помилки (Беллман–Форд)

1. **Релаксація з `dist[u] == Inf`** → переповнення `long.MaxValue + w`.
2. **Відновлення циклу прямо від `lastUpdated`** → може бути «хвіст» поза циклом, потрібно
   спершу зробити `n` кроків по `parent`.
3. **Для неорієнтованого графа з від'ємним ребром** → кожне таке ребро саме по собі від'ємний
   цикл `u → v → u`.

---

## ☕ Перерва 2 (≈ 105 хв від початку)

---

## 8. Флойд–Воршелл

### 8.1 Ідея: динамічне програмування за «дозволеними проміжними вершинами»

`d[k][i][j]` — найкоротший шлях `i → j`, у якому проміжними можуть бути лише вершини
`0..k`. Переходи:

```text
d[k][i][j] = min( d[k-1][i][j],                     // не йдемо через k
                  d[k-1][i][k] + d[k-1][k][j] )     // йдемо через k
```

Вимір `k` можна викинути і рахувати «на місці» в одній матриці `n × n`.
**Порядок циклів важливий: `k` — зовнішній!**

- Час `O(n³)`, пам'ять `O(n²)`. На практиці `n ≤ ~400–500`.
- Від'ємні ребра — ✅. Від'ємний цикл: після алгоритму `d[i][i] < 0`.
- Відповідь одразу для **всіх пар**.

```text
Орграф (4 вершини), таблиця ваг ребер (∞ — ребра немає):

         до 0  до 1  до 2  до 3
  з 0:     0     3     ∞     7
  з 1:     8     0     2     ∞
  з 2:     5     ∞     0     1
  з 3:     2     ∞     ∞     0

Ребра: 0→1:3, 0→3:7, 1→0:8, 1→2:2, 2→0:5, 2→3:1, 3→0:2
```

### 8.2 Реалізація з відновленням шляху

Для шляхів зберігаємо `next[i, j]` — **першу** вершину після `i` на найкоротшому шляху
`i → j`. Коли покращуємо через `k`, `next[i, j] = next[i, k]`.

```csharp
// Приклад 8.1: Флойд–Воршелл з матрицею next для відновлення шляхів
const int N = 4;
const int Inf = int.MaxValue / 2; // «половина» — щоб Inf + Inf не переповнилось
(int From, int To, int W)[] edges = [(0, 1, 3), (0, 3, 7), (1, 0, 8), (1, 2, 2), (2, 0, 5), (2, 3, 1), (3, 0, 2)];

var dist = new int[N, N];
var next = new int[N, N];
for (int i = 0; i < N; i++)
{
    for (int j = 0; j < N; j++)
    {
        dist[i, j] = i == j ? 0 : Inf;
        next[i, j] = -1;
    }
    next[i, i] = i;
}
foreach (var (from, to, w) in edges)
{
    dist[from, to] = Math.Min(dist[from, to], w); // Math.Min — на випадок паралельних ребер
    next[from, to] = to;
}

for (int k = 0; k < N; k++)          // k — ЗОВНІШНІЙ цикл
{
    for (int i = 0; i < N; i++)
    {
        if (dist[i, k] == Inf) continue; // невелика оптимізація
        for (int j = 0; j < N; j++)
        {
            if (dist[i, k] + dist[k, j] < dist[i, j])
            {
                dist[i, j] = dist[i, k] + dist[k, j];
                next[i, j] = next[i, k];
            }
        }
    }
}

Console.WriteLine("Матриця відстаней:");
Console.WriteLine("     " + string.Join("", Enumerable.Range(0, N).Select(j => $"{j,4}")));
for (int i = 0; i < N; i++)
{
    var row = Enumerable.Range(0, N).Select(j => dist[i, j] >= Inf ? "   ∞" : $"{dist[i, j],4}");
    Console.WriteLine($"  {i}: {string.Join("", row)}");
}

bool hasNegativeCycle = Enumerable.Range(0, N).Any(i => dist[i, i] < 0);
Console.WriteLine($"Від'ємний цикл: {hasNegativeCycle}");

foreach (var (from, to) in new[] { (1, 3), (3, 2), (0, 2) })
{
    Console.WriteLine($"Шлях {from} → {to} (dist {dist[from, to]}): {string.Join(" → ", Path(from, to))}");
}

List<int> Path(int from, int to)
{
    if (next[from, to] == -1) return [];
    var path = new List<int> { from };
    while (from != to)
    {
        from = next[from, to];
        path.Add(from);
    }
    return path;
}
```

**Приклад запуску:**

```text
Матриця відстаней:
        0   1   2   3
  0:    0   3   5   6
  1:    5   0   2   3
  2:    3   6   0   1
  3:    2   5   7   0
Від'ємний цикл: False
Шлях 1 → 3 (dist 3): 1 → 2 → 3
Шлях 3 → 2 (dist 7): 3 → 0 → 1 → 2
Шлях 0 → 2 (dist 5): 0 → 1 → 2
```

### Типові помилки (Флойд–Воршелл)

1. **`k` усередині** (`for i for j for k`) → алгоритм неправильний.
2. **`int.MaxValue` як нескінченність** → `Inf + Inf` переповнюється. Беріть `int.MaxValue / 2`.
3. **Не врахували паралельні ребра** — при читанні беріть `Math.Min`.
4. **Використали для `n = 10⁴`** → `10¹²` операцій. Для розріджених графів краще `n` разів Дейкстра.

### Міні-вправа 8

Як за допомогою Флойда–Воршелла порахувати **транзитивне замикання** (чи досяжна `j` з `i`)?

<details>
<summary>Розв'язок</summary>

Матриця `bool reach[i, j]`, ініціалізація ребрами та `reach[i, i] = true`, а перехід —
`reach[i, j] |= reach[i, k] && reach[k, j]`. Складність та сама `O(n³)`, а з
`System.Collections.BitArray` або `ulong`-масками рядків — у ~64 рази швидше.
</details>

---

## 9. Найкоротший і найдовший шлях у DAG

У DAG немає циклів, тому достатньо **один раз** пройти вершини в топологічному порядку
й прорелаксувати їхні вихідні ребра. Коли ми обробляємо `u`, усі шляхи в `u` вже враховані.

- Час `O(n + m)` — швидше за Дейкстру.
- **Від'ємні ваги — дозволені.**
- **Найдовший шлях** (в загальному графі NP-складна задача!) у DAG розв'язується тим самим
  алгоритмом із `max` замість `min`. Застосування — **критичний шлях** проєкту (CPM):
  мінімальний час завершення всіх задач із залежностями.

```text
Зважений DAG (ваги — тривалість задачі в днях):

      0 ──3──► 1 ──4──► 3
      │        │        │
      2        1        2
      ▼        ▼        ▼
      2 ──6──► 4 ──5──► 5
```

```csharp
// Приклад 9.1: найкоротший та найдовший (критичний) шлях у DAG за O(n + m)
const int N = 6;
(int From, int To, int W)[] edges = [(0, 1, 3), (0, 2, 2), (1, 3, 4), (1, 4, 1), (2, 4, 6), (3, 5, 2), (4, 5, 5)];
var adjacency = Enumerable.Range(0, N).Select(_ => new List<(int To, int W)>()).ToArray();
var inDegree = new int[N];
foreach (var (from, to, w) in edges)
{
    adjacency[from].Add((to, w));
    inDegree[to]++;
}

// 1) Топологічний порядок (Кан)
var order = new List<int>();
var queue = new Queue<int>(Enumerable.Range(0, N).Where(v => inDegree[v] == 0));
while (queue.Count > 0)
{
    int u = queue.Dequeue();
    order.Add(u);
    foreach (var (v, _) in adjacency[u])
    {
        if (--inDegree[v] == 0) queue.Enqueue(v);
    }
}
Console.WriteLine($"Топологічний порядок: {string.Join(" ", order)}");

// 2) Одна функція для обох задач: better(a, b) визначає, чи a «краще» за b
Solve("Найкоротший", (candidate, current) => candidate < current, long.MaxValue);
Solve("Найдовший  ", (candidate, current) => candidate > current, long.MinValue);

void Solve(string title, Func<long, long, bool> better, long initial)
{
    var dist = new long[N];
    var parent = new int[N];
    Array.Fill(dist, initial);
    Array.Fill(parent, -1);
    dist[0] = 0;

    foreach (int u in order)
    {
        if (dist[u] == initial) continue; // недосяжна зі старту
        foreach (var (v, w) in adjacency[u])
        {
            if (better(dist[u] + w, dist[v]))
            {
                dist[v] = dist[u] + w;
                parent[v] = u;
            }
        }
    }

    var path = new List<int>();
    for (int v = N - 1; v != -1; v = parent[v]) path.Add(v);
    path.Reverse();
    Console.WriteLine($"{title} 0 → 5: {dist[N - 1]} днів, шлях {string.Join(" → ", path)}");
}
```

**Приклад запуску:**

```text
Топологічний порядок: 0 1 2 3 4 5
Найкоротший 0 → 5: 9 днів, шлях 0 → 1 → 3 → 5
Найдовший   0 → 5: 13 днів, шлях 0 → 2 → 4 → 5
```

> Найдовший шлях = 13 днів — це **мінімальний** час виконання проєкту, навіть якщо паралельно
> працює скільки завгодно людей. Затримка будь-якої задачі на критичному шляху зсуває дедлайн.

---

## 10. A* на сітці

### 10.1 Ідея

A* — це Дейкстра, яка **знає напрямок до цілі**. Пріоритет у купі:

```text
f(v) = g(v) + h(v)
       │      └── евристика: оцінка відстані від v до цілі
       └── фактична відстань від старту до v
```

- Якщо `h` **допустима** (ніколи не переоцінює справжню відстань), A* знаходить
  **оптимальний** шлях.
- Якщо `h` ще й **монотонна** (`h(u) ≤ w(u,v) + h(v)`), кожну вершину достатньо
  обробити один раз — як у Дейкстрі.
- `h ≡ 0` → звичайна Дейкстра. Чим точніша `h`, тим менше вершин розкриваємо.

| Рух на сітці | Евристика |
|---|---|
| 4 напрямки, крок = 1 | Манхеттенська: `|Δr| + |Δc|` |
| 8 напрямків, діагональ = 1 | Чебишева: `max(|Δr|, |Δc|)` |
| 8 напрямків, діагональ = √2 | Октильна |
| Довільний рух | Евклідова |

### 10.2 Реалізація та порівняння з Дейкстрою

```csharp
// Приклад 10.1: A* з манхеттенською евристикою vs Дейкстра (h = 0) на тій самій сітці
string[] map =
[
    "S.........",
    ".########.",
    "..........",
    ".########.",
    ".........E",
];

var (aStarLength, aStarExpanded, aStarPath) = Search(map, useHeuristic: true);
var (dijkstraLength, dijkstraExpanded, _) = Search(map, useHeuristic: false);

Console.WriteLine($"A*:       довжина {aStarLength}, розкрито вершин {aStarExpanded}");
Console.WriteLine($"Дейкстра: довжина {dijkstraLength}, розкрито вершин {dijkstraExpanded}");

var canvas = map.Select(row => row.ToCharArray()).ToArray();
foreach (var (r, c) in aStarPath)
{
    if (canvas[r][c] == '.') canvas[r][c] = '*';
}
foreach (var row in canvas) Console.WriteLine(new string(row));

static (int Length, int Expanded, List<(int R, int C)> Path) Search(string[] map, bool useHeuristic)
{
    int rows = map.Length, cols = map[0].Length;
    (int R, int C) start = (0, 0), goal = (rows - 1, cols - 1);

    int Heuristic((int R, int C) cell) =>
        useHeuristic ? Math.Abs(cell.R - goal.R) + Math.Abs(cell.C - goal.C) : 0;

    var g = new int[rows, cols];
    var parent = new (int R, int C)[rows, cols];
    for (int r = 0; r < rows; r++)
        for (int c = 0; c < cols; c++)
            g[r, c] = int.MaxValue;

    // Пріоритет — кортеж (f, h): при рівних f перевага клітинці, ближчій до цілі.
    // Це класичний tie-break, який помітно зменшує кількість розкритих вершин.
    var open = new PriorityQueue<(int R, int C), (int F, int H)>();
    g[start.R, start.C] = 0;
    open.Enqueue(start, (Heuristic(start), Heuristic(start)));
    var closed = new bool[rows, cols];
    int expanded = 0;

    while (open.TryDequeue(out var cell, out _))
    {
        if (closed[cell.R, cell.C]) continue; // лінива видалення, як у Дейкстрі
        closed[cell.R, cell.C] = true;
        expanded++;
        if (cell == goal) break; // при монотонній h вийняли ціль — шлях оптимальний

        foreach (var (dr, dc) in new[] { (0, 1), (1, 0), (0, -1), (-1, 0) })
        {
            (int R, int C) next = (cell.R + dr, cell.C + dc);
            if (next.R < 0 || next.R >= rows || next.C < 0 || next.C >= cols) continue;
            if (map[next.R][next.C] == '#' || closed[next.R, next.C]) continue;

            int tentative = g[cell.R, cell.C] + 1;
            if (tentative < g[next.R, next.C])
            {
                g[next.R, next.C] = tentative;
                parent[next.R, next.C] = cell;
                int h = Heuristic(next);
                open.Enqueue(next, (tentative + h, h));
            }
        }
    }

    var path = new List<(int R, int C)>();
    for (var cell = goal; cell != start; cell = parent[cell.R, cell.C]) path.Add(cell);
    path.Add(start);
    path.Reverse();
    return (g[goal.R, goal.C], expanded, path);
}
```

**Приклад запуску:**

```text
A*:       довжина 13, розкрито вершин 14
Дейкстра: довжина 13, розкрито вершин 34
S*********
.########*
.........*
.########*
.........E
```

### Типові помилки (A*)

1. **Недопустима евристика** (наприклад, Евклідова × 2 чи Манхеттенська при 8 напрямках) → шлях
   не оптимальний.
2. **Пріоритет `h` замість `g + h`** → це «жадібний best-first», швидкий, але не оптимальний.
3. **Немає `closed`/перевірки застарілих записів** → повторне розкриття вершин.

---

## 11. Мінімальне остовне дерево: Union-Find, Краскал, Прім

### 11.1 Постановка

**Остовне дерево** зв'язного неорієнтованого графа — підмножина з `n − 1` ребер, що
з'єднує всі вершини без циклів. **Мінімальне** (MST) — з найменшою сумою ваг.

Застосування: найдешевша мережа кабелів/труб, кластеризація (видалити `k − 1` найважчих
ребер MST → `k` кластерів), наближений розв'язок TSP (2-наближення).

**Властивість розрізу (cut property).** Для будь-якого розбиття вершин на дві частини
найлегше ребро, що перетинає розріз, належить деякому MST. На ній стоять і Краскал, і Прім.

```text
Зважений G та його MST (подвійні лінії):

      0 ──7── 1 ══2══ 3             MST: 1-4 (1), 0-2 (2), 1-3 (2), 2-4 (3), 3-5 (3)
      ║       ║       ║             Вага: 1 + 2 + 2 + 3 + 3 = 11
      2       1       3
      ║       ║       ║             Не взяли: 4-5 (6) — утворює цикл 4-1-3-5
      2 ══3══ 4 ──6── 5                       0-1 (7) — утворює цикл 0-2-4-1
```

### 11.2 Union-Find (Disjoint Set Union, DSU)

Структура для **множин, що не перетинаються**, з двома операціями:

- `Find(x)` — «представник» (корінь) множини, де лежить `x`;
- `Union(a, b)` — об'єднати множини `a` і `b`.

Дві оптимізації:

- **Стиснення шляху** (path compression): після `Find` всі вершини шляху вказують прямо на корінь.
- **Об'єднання за рангом** (union by rank): менше дерево підвішуємо під більше.

Разом — `O(α(n))` амортизовано на операцію, де `α` — обернена функція Акермана
(`α(n) ≤ 4` для будь-якого `n`, меншого за кількість атомів у Всесвіті).

```text
Без стиснення:               Після Find(4) зі стисненням:
      0                            0
      │                        ╱ ╱ │ ╲
      1                       1  2  3  4
      │
      2
      │
      3
      │
      4
```

```csharp
// Приклад 11.1: Union-Find зі стисненням шляху та об'єднанням за рангом
var dsu = new DisjointSetUnion(8);
Console.WriteLine($"Множин спочатку: {dsu.SetCount}");

foreach (var (a, b) in new[] { (0, 1), (2, 3), (1, 3), (4, 5), (6, 7), (5, 7), (0, 2) })
{
    bool merged = dsu.Union(a, b);
    Console.WriteLine($"Union({a}, {b}) → {(merged ? "об'єднали" : "вже разом")}, множин: {dsu.SetCount}");
}

Console.WriteLine($"Connected(0, 3): {dsu.Connected(0, 3)}");
Console.WriteLine($"Connected(3, 4): {dsu.Connected(3, 4)}");
Console.WriteLine($"Розмір множини з 6: {dsu.SizeOf(6)}");

/// <summary>Система неперетинних множин: стиснення шляху + об'єднання за рангом.</summary>
sealed class DisjointSetUnion
{
    private readonly int[] _parent;
    private readonly int[] _rank;
    private readonly int[] _size;

    public DisjointSetUnion(int count)
    {
        _parent = new int[count];
        _rank = new int[count];
        _size = new int[count];
        for (int i = 0; i < count; i++)
        {
            _parent[i] = i; // спочатку кожен елемент — окрема множина і сам собі корінь
            _size[i] = 1;
        }
        SetCount = count;
    }

    public int SetCount { get; private set; }

    public int Find(int x)
    {
        // Ітеративно: 1) знайти корінь; 2) перевісити всіх на шляху прямо на корінь
        int root = x;
        while (_parent[root] != root) root = _parent[root];
        while (_parent[x] != root)
        {
            int next = _parent[x];
            _parent[x] = root;
            x = next;
        }
        return root;
    }

    /// <summary>Повертає false, якщо a і b вже в одній множині (у графі це означає цикл).</summary>
    public bool Union(int a, int b)
    {
        int rootA = Find(a);
        int rootB = Find(b);
        if (rootA == rootB) return false;

        // Нижче дерево підвішуємо під вище — висота росте лише при рівних рангах
        if (_rank[rootA] < _rank[rootB]) (rootA, rootB) = (rootB, rootA);
        _parent[rootB] = rootA;
        _size[rootA] += _size[rootB];
        if (_rank[rootA] == _rank[rootB]) _rank[rootA]++;
        SetCount--;
        return true;
    }

    public bool Connected(int a, int b) => Find(a) == Find(b);

    public int SizeOf(int x) => _size[Find(x)];
}
```

**Приклад запуску:**

```text
Множин спочатку: 8
Union(0, 1) → об'єднали, множин: 7
Union(2, 3) → об'єднали, множин: 6
Union(1, 3) → об'єднали, множин: 5
Union(4, 5) → об'єднали, множин: 4
Union(6, 7) → об'єднали, множин: 3
Union(5, 7) → об'єднали, множин: 2
Union(0, 2) → вже разом, множин: 2
Connected(0, 3): True
Connected(3, 4): False
Розмір множини з 6: 4
```

### 11.3 Алгоритм Краскала

1. Відсортувати ребра за вагою.
2. Для кожного ребра `(u, v)`: якщо `u` і `v` у **різних** множинах DSU — взяти ребро і
   об'єднати множини; інакше ребро утворило б цикл — пропустити.
3. Зупинитися, коли взяли `n − 1` ребро.

Складність `O(m log m)` (сортування) + `O(m · α(n))`. Добре для **розріджених** графів
та коли ребра вже дані списком.

```csharp
// Приклад 11.2: Краскал на зваженому G
const int N = 6;
WeightedEdge[] edges =
[
    new(0, 1, 7), new(0, 2, 2), new(1, 3, 2), new(1, 4, 1),
    new(2, 4, 3), new(3, 5, 3), new(4, 5, 6),
];

var parent = Enumerable.Range(0, N).ToArray();
var rank = new int[N];

var mst = new List<WeightedEdge>();
long total = 0;
// Стабільне сортування: за вагою, при рівних — за From, To (детермінований результат)
foreach (var edge in edges.OrderBy(e => e.Weight).ThenBy(e => e.From).ThenBy(e => e.To))
{
    if (Union(edge.From, edge.To))
    {
        mst.Add(edge);
        total += edge.Weight;
        Console.WriteLine($"  беремо    {edge.From}-{edge.To} (w={edge.Weight})");
        if (mst.Count == N - 1) break; // дерево вже повне
    }
    else
    {
        Console.WriteLine($"  пропускаємо {edge.From}-{edge.To} (w={edge.Weight}) — цикл");
    }
}

Console.WriteLine(mst.Count == N - 1
    ? $"MST вага = {total}, ребер = {mst.Count}"
    : "Граф незв'язний — остовного дерева немає (є лише остовний ліс)");

int Find(int x) => parent[x] == x ? x : parent[x] = Find(parent[x]); // рекурсивне стиснення

bool Union(int a, int b)
{
    a = Find(a);
    b = Find(b);
    if (a == b) return false;
    if (rank[a] < rank[b]) (a, b) = (b, a);
    parent[b] = a;
    if (rank[a] == rank[b]) rank[a]++;
    return true;
}

readonly record struct WeightedEdge(int From, int To, int Weight);
```

**Приклад запуску:**

```text
  беремо    1-4 (w=1)
  беремо    0-2 (w=2)
  беремо    1-3 (w=2)
  беремо    2-4 (w=3)
  беремо    3-5 (w=3)
MST вага = 11, ребер = 5
```

> Зверніть увагу: `4-5 (6)` і `0-1 (7)` навіть не розглядались — ми зупинились, щойно набрали `n − 1 = 5` ребер.

### 11.4 Алгоритм Пріма

«Дейкстра для MST»: вирощуємо **одне** дерево від стартової вершини, щоразу додаючи
**найлегше ребро**, що виходить з дерева назовні. Пріоритет у купі — **вага ребра**
(а не відстань від старту, як у Дейкстрі!).

Складність з купою та лінивим видаленням: `O(m log n)`. Для **щільних** графів (`m ≈ n²`)
є варіант на масиві за `O(n²)` — він швидший за Краскала.

```csharp
// Приклад 11.3: Прім з PriorityQueue (ліниве видалення)
const int N = 6;
(int U, int V, int W)[] edges = [(0, 1, 7), (0, 2, 2), (1, 3, 2), (1, 4, 1), (2, 4, 3), (3, 5, 3), (4, 5, 6)];
var adjacency = Enumerable.Range(0, N).Select(_ => new List<(int To, int W)>()).ToArray();
foreach (var (u, v, w) in edges)
{
    adjacency[u].Add((v, w));
    adjacency[v].Add((u, w));
}

var inTree = new bool[N];
// Елемент — (куди, звідки), пріоритет — вага ребра
var heap = new PriorityQueue<(int To, int From), int>();
heap.Enqueue((0, -1), 0); // стартуємо з вершини 0 «фіктивним» ребром ваги 0
long total = 0;
var taken = new List<string>();

while (heap.TryDequeue(out var item, out int weight))
{
    if (inTree[item.To]) continue; // вершина вже в дереві — застарілий запис
    inTree[item.To] = true;
    total += weight;
    if (item.From != -1)
    {
        taken.Add($"{item.From}-{item.To}(w={weight})");
    }

    foreach (var (next, w) in adjacency[item.To])
    {
        if (!inTree[next])
        {
            heap.Enqueue((next, item.To), w); // пріоритет — ВАГА РЕБРА, не відстань
        }
    }
}

Console.WriteLine($"Порядок додавання ребер: {string.Join(", ", taken)}");
Console.WriteLine($"MST вага = {total}");
Console.WriteLine($"Усі вершини в дереві: {inTree.All(x => x)}");
```

**Приклад запуску:**

```text
Порядок додавання ребер: 0-2(w=2), 2-4(w=3), 4-1(w=1), 1-3(w=2), 3-5(w=3)
MST вага = 11
Усі вершини в дереві: True
```

### 11.5 Краскал vs Прім

| | Краскал | Прім (купа) | Прім (масив) |
|---|---|---|---|
| Складність | `O(m log m)` | `O(m log n)` | `O(n²)` |
| Найкраще для | розріджених, ребра списком | розріджених, список суміжності | щільних графів |
| Структура | DSU | `PriorityQueue` | масив `minEdge[]` |
| Незв'язний граф | природно дає остовний ліс | лише компоненту старту | лише компоненту старту |

### Типові помилки (MST)

1. **Прім з пріоритетом `dist[u] + w`** (як у Дейкстрі) → це дерево найкоротших шляхів, а не MST.
   Для G воно має вагу 14 (ребра 0-2, 2-4, 4-1, 1-3, 4-5), а MST — 11.
2. **DSU без стиснення шляху й рангу** → `O(n)` на `Find`, `O(m · n)` загалом.
3. **MST для орієнтованого графа** → це інша задача (arborescence, алгоритм Чу–Лю/Едмондса).
4. **Не перевірили зв'язність** (`mst.Count == n − 1`).

### Міні-вправа 11

Потрібно розбити 6 міст зі зваженого G на **2 кластери** так, щоб мінімальна відстань
між кластерами була максимальною. Як?

<details>
<summary>Розв'язок</summary>

Запустити Краскала і зупинитись, коли кількість множин у DSU стане `k = 2` (тобто взяти
`n − k = 4` ребра). Для G: беремо 1-4, 0-2, 1-3, 2-4 → кластери `{0, 1, 2, 3, 4}` та `{5}`.
Наступне ребро MST (3-5, вага 3) — і є відстань між кластерами.
</details>

---

## 12. Застосування Union-Find

### 12.1 Динамічна зв'язність (онлайн-запити)

Ребра **надходять поступово** (прокладають нові дороги), і між ними приходять запити
«чи зв'язані `a` і `b`?». BFS на кожен запит — `O(q · (n + m))`. DSU — майже `O(q)`.

> Обмеження: DSU вміє лише **додавати** ребра. Видалення — складніша задача
> (офлайн-алгоритми, «DSU з відкатами» + дерево відрізків за часом).

```csharp
// Приклад 12.1: дороги будуються поступово — відповідаємо на запити про зв'язність
string[] log =
[
    "build 0 1",
    "ask 0 3",
    "build 2 3",
    "build 1 2",
    "ask 0 3",
    "ask 4 5",
    "build 4 5",
    "ask 4 5",
    "count",
];

int n = 6;
var parent = Enumerable.Range(0, n).ToArray();
var size = Enumerable.Repeat(1, n).ToArray();
int components = n;

foreach (string line in log)
{
    string[] parts = line.Split(' ');
    switch (parts[0])
    {
        case "build":
            Union(int.Parse(parts[1]), int.Parse(parts[2]));
            break;
        case "ask":
            int a = int.Parse(parts[1]), b = int.Parse(parts[2]);
            Console.WriteLine($"{line,-10} → {(Find(a) == Find(b) ? "так" : "ні")}");
            break;
        case "count":
            Console.WriteLine($"{line,-10} → {components} компонент(и)");
            break;
    }
}

int Find(int x)
{
    while (parent[x] != x)
    {
        parent[x] = parent[parent[x]]; // «ділення шляху навпіл» — простий варіант стиснення
        x = parent[x];
    }
    return x;
}

void Union(int a, int b)
{
    a = Find(a);
    b = Find(b);
    if (a == b) return;
    if (size[a] < size[b]) (a, b) = (b, a); // об'єднання за розміром — альтернатива рангу
    parent[b] = a;
    size[a] += size[b];
    components--;
}
```

**Приклад запуску:**

```text
ask 0 3    → ні
ask 0 3    → так
ask 4 5    → ні
ask 4 5    → так
count      → 2 компонент(и)
```

### 12.2 Надлишкове ребро (Redundant Connection)

Дерево з `n` вершин отримало **одне зайве** ребро → з'явився рівно один цикл. Знайти
ребро, видалення якого знову робить граф деревом (якщо варіантів кілька — останнє у вхідних даних).

Рішення: додаємо ребра по черзі в DSU. Перше ребро, чиї кінці **вже** в одній множині, — відповідь.

```text
  1 ─── 2           Ребра: [1,2] [1,3] [2,3] [3,4] [1,5]
  │   ╱              [2,3]: 2 і 3 вже зв'язані через 1 → зайве
  │ ╱
  3 ─── 4
  1 ─── 5
```

```csharp
// Приклад 12.2: надлишкове ребро + кількість «провінцій» (компонент) через DSU
int[][] edges = [[1, 2], [1, 3], [2, 3], [3, 4], [1, 5]];
Console.WriteLine($"Надлишкове ребро: [{string.Join(", ", FindRedundant(edges))}]");

int[][] isConnected =
[
    [1, 1, 0, 0],
    [1, 1, 0, 0],
    [0, 0, 1, 1],
    [0, 0, 1, 1],
];
Console.WriteLine($"Провінцій: {CountProvinces(isConnected)}");

static int[] FindRedundant(int[][] edges)
{
    // Вершини нумеруються з 1 — масив на n + 1
    var parent = Enumerable.Range(0, edges.Length + 1).ToArray();
    int Find(int x) => parent[x] == x ? x : parent[x] = Find(parent[x]);

    foreach (int[] edge in edges)
    {
        int a = Find(edge[0]), b = Find(edge[1]);
        if (a == b) return edge; // кінці вже зв'язані — ребро замикає цикл
        parent[a] = b;
    }
    return [];
}

static int CountProvinces(int[][] matrix)
{
    int n = matrix.Length;
    var parent = Enumerable.Range(0, n).ToArray();
    int Find(int x) => parent[x] == x ? x : parent[x] = Find(parent[x]);
    int count = n;
    for (int i = 0; i < n; i++)
    {
        for (int j = i + 1; j < n; j++)
        {
            if (matrix[i][j] == 1 && Find(i) != Find(j))
            {
                parent[Find(i)] = Find(j);
                count--; // кожне успішне об'єднання зменшує кількість компонент на 1
            }
        }
    }
    return count;
}
```

**Приклад запуску:**

```text
Надлишкове ребро: [2, 3]
Провінцій: 2
```

### Інші застосування DSU

| Задача | Ідея |
|---|---|
| Краскал | цикл ↔ `Find(u) == Find(v)` |
| Перколяція (протікання) | фіктивні вершини «верх» і «низ», чи з'єднані |
| Об'єднання акаунтів за email | email → акаунт, union акаунтів зі спільним email |
| «Острови II» (клітинки додаються) | кожна нова суша — union із сусідами-сушею |
| Еквівалентність змінних `a == b`, `a != c` | спершу union усіх `==`, потім перевірка `!=` |
| Офлайн LCA (Тарʼян) | DFS + DSU |

---

## ☕ Перерва 3 (≈ 150 хв від початку)

---

## 13. Компоненти сильної зв'язності (Косарайю, Тарʼян)

### 13.1 Визначення

В орграфі вершини `u` і `v` **сильно зв'язні**, якщо є шлях `u → v` **і** шлях `v → u`.
Компонента сильної зв'язності (SCC) — максимальна множина попарно сильно зв'язних вершин.

Якщо стиснути кожну SCC в одну вершину, отримаємо **граф конденсації** — він завжди **DAG**.

```text
Орграф (8 вершин):                          Конденсація (DAG):

   0 ──► 1          6 ◄──► 7                   {6,7}
   ▲     │          │                            │
   │     ▼          ▼                            ▼
   └──── 2 ──► 3 ──► 4          {0,1,2} ──► {3,4,5}
               ▲     │
               │     ▼
               └──── 5

Ребра: 0→1, 1→2, 2→0, 2→3, 3→4, 4→5, 5→3, 6→4, 6→7, 7→6
SCC: {0,1,2}, {3,4,5}, {6,7}
```

Застосування: циклічні залежності між модулями, 2-SAT, аналіз соцмереж («хто кого
читає взаємно»), спрощення графа до DAG перед DP.

### 13.2 Алгоритм Косарайю (два проходи DFS)

1. DFS по графу `G`, записати вершини у порядку **виходу**.
2. Побудувати **транспонований** граф `Gᵀ` (усі ребра розвернуті).
3. Йти вершинами у порядку **спадання часу виходу** і запускати DFS на `Gᵀ`; кожен запуск
   збирає рівно одну SCC.

Чому працює: вершина з найбільшим `tout` лежить у «витоковій» SCC конденсації.
У `Gᵀ` з неї не вийти в інші SCC — тож DFS збере рівно її.

```csharp
// Приклад 13.1: Косарайю — два проходи DFS і граф конденсації
const int N = 8;
(int From, int To)[] edges = [(0, 1), (1, 2), (2, 0), (2, 3), (3, 4), (4, 5), (5, 3), (6, 4), (6, 7), (7, 6)];

var graph = Enumerable.Range(0, N).Select(_ => new List<int>()).ToArray();
var reversed = Enumerable.Range(0, N).Select(_ => new List<int>()).ToArray();
foreach (var (from, to) in edges)
{
    graph[from].Add(to);
    reversed[to].Add(from); // транспонований граф
}

// Прохід 1: порядок виходу на G
var visited = new bool[N];
var exitOrder = new List<int>();
for (int v = 0; v < N; v++)
{
    if (!visited[v]) FirstPass(v);
}
Console.WriteLine($"Порядок виходу: {string.Join(" ", exitOrder)}");

// Прохід 2: у зворотному порядку виходу, DFS на Gᵀ
var component = new int[N];
Array.Fill(component, -1);
int count = 0;
for (int i = exitOrder.Count - 1; i >= 0; i--)
{
    int v = exitOrder[i];
    if (component[v] == -1)
    {
        SecondPass(v, count);
        count++;
    }
}

for (int c = 0; c < count; c++)
{
    var members = Enumerable.Range(0, N).Where(v => component[v] == c);
    Console.WriteLine($"SCC {c}: {{{string.Join(", ", members)}}}");
}

// Граф конденсації: ребра між різними SCC (без дублікатів)
var condensation = new SortedSet<(int, int)>();
foreach (var (from, to) in edges)
{
    if (component[from] != component[to])
    {
        condensation.Add((component[from], component[to]));
    }
}
Console.WriteLine($"Ребра конденсації: {string.Join(", ", condensation.Select(e => $"{e.Item1}→{e.Item2}"))}");

void FirstPass(int u)
{
    visited[u] = true;
    foreach (int v in graph[u])
    {
        if (!visited[v]) FirstPass(v);
    }
    exitOrder.Add(u);
}

void SecondPass(int u, int id)
{
    component[u] = id;
    foreach (int v in reversed[u])
    {
        if (component[v] == -1) SecondPass(v, id);
    }
}
```

**Приклад запуску:**

```text
Порядок виходу: 5 4 3 2 1 0 7 6
SCC 0: {6, 7}
SCC 1: {0, 1, 2}
SCC 2: {3, 4, 5}
Ребра конденсації: 0→2, 1→2
```

> Косарайю нумерує SCC у **топологічному порядку** конденсації: ребра конденсації завжди
> йдуть від меншого номера до більшого.

### 13.3 Алгоритм Тарʼяна (один прохід, low-link)

Один DFS і стек вершин. Для кожної вершини:

- `tin[v]` — час входу;
- `low[v]` — найменший `tin`, досяжний з піддерева `v` через **не більше одне** ребро до вершини,
  що **ще лежить у стеку**.

Якщо після обробки всіх сусідів `low[v] == tin[v]`, то `v` — «корінь» SCC: знімаємо зі
стеку вершини до `v` включно.

```csharp
// Приклад 13.2: Тарʼян — SCC за один DFS
const int N = 8;
(int From, int To)[] edges = [(0, 1), (1, 2), (2, 0), (2, 3), (3, 4), (4, 5), (5, 3), (6, 4), (6, 7), (7, 6)];
var graph = Enumerable.Range(0, N).Select(_ => new List<int>()).ToArray();
foreach (var (from, to) in edges) graph[from].Add(to);

var tin = new int[N];
var low = new int[N];
Array.Fill(tin, -1);
var onStack = new bool[N];
var stack = new Stack<int>();
int timer = 0;
var components = new List<List<int>>();

for (int v = 0; v < N; v++)
{
    if (tin[v] == -1) Dfs(v);
}

for (int i = 0; i < components.Count; i++)
{
    Console.WriteLine($"SCC (знайдена {i + 1}-ю): {{{string.Join(", ", components[i].Order())}}}");
}
Console.WriteLine($"low = [{string.Join(", ", low)}]");

void Dfs(int u)
{
    tin[u] = low[u] = timer++;
    stack.Push(u);
    onStack[u] = true;

    foreach (int v in graph[u])
    {
        if (tin[v] == -1)
        {
            Dfs(v);
            low[u] = Math.Min(low[u], low[v]); // нащадок дістався вище — і ми теж
        }
        else if (onStack[v])
        {
            low[u] = Math.Min(low[u], tin[v]); // ребро в поточну (незакриту) SCC
        }
        // інакше v у вже закритій SCC — ігноруємо (cross-ребро в іншу компоненту)
    }

    if (low[u] == tin[u])
    {
        var scc = new List<int>();
        int w;
        do
        {
            w = stack.Pop();
            onStack[w] = false;
            scc.Add(w);
        } while (w != u);
        components.Add(scc);
    }
}
```

**Приклад запуску:**

```text
SCC (знайдена 1-ю): {3, 4, 5}
SCC (знайдена 2-ю): {0, 1, 2}
SCC (знайдена 3-ю): {6, 7}
low = [0, 0, 0, 3, 3, 3, 6, 6]
```

> Тарʼян знаходить SCC у **зворотному** топологічному порядку: першою закривається
> «стокова» компонента `{3, 4, 5}`.

### Типові помилки (SCC)

1. **Косарайю: другий прохід по `G`, а не `Gᵀ`** → SCC «зливаються».
2. **Косарайю: другий прохід у порядку зростання `tout`** → неправильно.
3. **Тарʼян: `low[u] = min(low[u], low[v])` для вершини не в стеку** → різні SCC склеюються.
4. **Рекурсія на великих графах** → ітеративна версія (як у 4.3).

---

## 14. Мости та точки зчленування

### 14.1 Визначення

У **неорієнтованому** графі:

- **Міст** — ребро, видалення якого збільшує кількість компонент зв'язності.
- **Точка зчленування** (articulation point, cut vertex) — вершина, видалення якої
  (з усіма ребрами) збільшує кількість компонент.

Застосування: критичні лінії електромережі, «вузькі місця» в комп'ютерній мережі,
надійність транспорту.

```text
G + «хвіст» 5-6, 6-7:

      0 ───── 1 ───── 3
      │       │       │
      │       │       │
      2 ───── 4 ───── 5 ═════ 6 ═════ 7
                        міст     міст

Мости: 5-6, 6-7   (кожне інше ребро лежить на циклі)
Точки зчленування: 5, 6
```

### 14.2 Low-link для неорієнтованого графа

`low[v]` = мінімальний `tin`, досяжний з піддерева `v` у дереві DFS, використавши
**не більше одного** зворотного ребра.

- Ребро дерева `u — v` (`v` — дитина) — **міст**, якщо `low[v] > tin[u]`:
  з піддерева `v` не можна піднятись до `u` або вище в обхід цього ребра.
- `u` (не корінь) — **точка зчленування**, якщо є дитина `v` з `low[v] >= tin[u]`.
- Корінь DFS — точка зчленування, якщо має **≥ 2 дітей** у дереві DFS.

```csharp
// Приклад 14.1: мости та точки зчленування за один DFS (low-link)
const int N = 8;
(int U, int V)[] edges = [(0, 1), (0, 2), (1, 3), (1, 4), (2, 4), (3, 5), (4, 5), (5, 6), (6, 7)];

// Зберігаємо id ребра — щоб коректно працювати з паралельними ребрами
var adjacency = Enumerable.Range(0, N).Select(_ => new List<(int To, int EdgeId)>()).ToArray();
for (int id = 0; id < edges.Length; id++)
{
    var (u, v) = edges[id];
    adjacency[u].Add((v, id));
    adjacency[v].Add((u, id));
}

var tin = new int[N];
var low = new int[N];
Array.Fill(tin, -1);
int timer = 0;
var bridges = new List<(int, int)>();
var articulation = new SortedSet<int>();

for (int v = 0; v < N; v++)
{
    if (tin[v] == -1) Dfs(v, parentEdge: -1);
}

Console.WriteLine($"tin = [{string.Join(", ", tin)}]");
Console.WriteLine($"low = [{string.Join(", ", low)}]");
Console.WriteLine($"Мости: {string.Join(", ", bridges.Select(b => $"{b.Item1}-{b.Item2}"))}");
Console.WriteLine($"Точки зчленування: {string.Join(", ", articulation)}");

void Dfs(int u, int parentEdge)
{
    tin[u] = low[u] = timer++;
    int children = 0;

    foreach (var (v, edgeId) in adjacency[u])
    {
        if (edgeId == parentEdge) continue; // не повертаємось тим самим ребром (саме ребром, не вершиною!)

        if (tin[v] != -1)
        {
            low[u] = Math.Min(low[u], tin[v]); // зворотне ребро
            continue;
        }

        Dfs(v, edgeId);
        children++;
        low[u] = Math.Min(low[u], low[v]);

        if (low[v] > tin[u])
        {
            bridges.Add((u, v)); // з піддерева v не піднятися до u в обхід ребра u-v
        }
        if (parentEdge != -1 && low[v] >= tin[u])
        {
            articulation.Add(u); // без u піддерево v відірветься від предків u
        }
    }

    if (parentEdge == -1 && children >= 2)
    {
        articulation.Add(u); // окреме правило для кореня DFS
    }
}
```

**Приклад запуску:**

```text
tin = [0, 1, 5, 2, 4, 3, 6, 7]
low = [0, 0, 0, 0, 0, 0, 6, 7]
Мости: 6-7, 5-6
Точки зчленування: 5, 6
```

### Типові помилки (мости)

1. **Міст: `low[v] >= tin[u]`** (як для точок зчленування) → кожне ребро дерева до вершини
   на циклі «стане» мостом. Для мостів — **строго** `>`.
2. **Пропуск батьківської вершини замість батьківського ребра** → паралельні ребра `u-v`
   хибно вважаються мостом.
3. **Забули правило для кореня** → корінь з двома піддеревами не потрапляє в точки зчленування
   (або навпаки — корінь-листок потрапляє).

### Міні-вправа 14

Скільки мостів у дереві з `n` вершин? А точок зчленування?

<details>
<summary>Розв'язок</summary>

У дереві **кожне** з `n − 1` ребер — міст (циклів немає). Точки зчленування — усі вершини
зі степенем ≥ 2, тобто всі, крім листків (для `n ≥ 3`).
</details>

---

## 15. Як обрати алгоритм

| Задача | Умови | Алгоритм | Складність |
|---|---|---|---|
| Обхід, досяжність | будь-який граф | BFS / DFS | `O(n + m)` |
| Найкоротший шлях | ваги відсутні (всі = 1) | BFS | `O(n + m)` |
| Найкоротший шлях | ваги 0 або 1 | 0-1 BFS (дек) | `O(n + m)` |
| Найкоротший шлях від одного джерела | ваги ≥ 0 | Дейкстра + `PriorityQueue` | `O((n + m) log n)` |
| Найкоротший шлях до однієї цілі | ваги ≥ 0, є евристика | A* | залежить від `h`; гірший випадок як Дейкстра |
| Найкоротший шлях | від'ємні ваги | Беллман–Форд | `O(n · m)` |
| Від'ємний цикл | — | Беллман–Форд (n-та ітерація) / Флойд (`d[i][i] < 0`) | `O(n · m)` / `O(n³)` |
| Найкоротший шлях з ≤ k ребер | — | Беллман–Форд на k ітерацій | `O(k · m)` |
| Усі пари | `n ≤ ~500` | Флойд–Воршелл | `O(n³)` |
| Усі пари | розріджений, ваги ≥ 0 | n × Дейкстра | `O(n (n + m) log n)` |
| Найкоротший / найдовший шлях | DAG | топосорт + релаксація | `O(n + m)` |
| Порядок залежностей, цикл в DAG | орграф | Кан / DFS | `O(n + m)` |
| Цикл у неорієнтованому графі | — | DFS з батьком / DSU | `O(n + m)` |
| Цикл в орграфі | — | DFS з трьома кольорами / Кан | `O(n + m)` |
| Компоненти зв'язності | статичний граф | BFS / DFS | `O(n + m)` |
| Компоненти зв'язності | ребра додаються | Union-Find | `O(α(n))` на операцію |
| Двочастковість | — | BFS/DFS розфарбування | `O(n + m)` |
| MST | розріджений | Краскал + DSU | `O(m log m)` |
| MST | щільний | Прім на масиві | `O(n²)` |
| SCC, конденсація | орграф | Косарайю / Тарʼян | `O(n + m)` |
| Мости, точки зчленування | неорієнтований | Тарʼян (low-link) | `O(n + m)` |
| Flood fill, острови | сітка | BFS / DFS / DSU | `O(rows · cols)` |

**Швидкий алгоритм вибору:**

```text
Є ваги? ── ні ──► BFS
   │
   так ─► лише 0/1? ── так ──► 0-1 BFS
   │
   ні ─► граф DAG? ── так ──► топосорт + релаксація
   │
   ні ─► є від'ємні? ── так ──► усі пари і n мале? ── так ──► Флойд–Воршелл
   │                                   │
   │                                   ні ──► Беллман–Форд
   ні ─► одна ціль і є евристика? ── так ──► A*
   │
   ні ──► Дейкстра
```

---

## 16. Практичні задачі

### 16.1 Word Ladder (BFS на неявному графі)

Перетворити `beginWord` на `endWord`, змінюючи **одну літеру за крок**; кожне проміжне
слово має бути в словнику. Знайти мінімальну кількість слів у ланцюжку.

Вершини — слова, ребра — «відрізняються однією літерою». Будувати граф явно за `O(N² · L)`
дорого, тому використовуємо **шаблони**: `hot` → `*ot`, `h*t`, `ho*`. Слова з однаковим
шаблоном — сусіди.

```csharp
// Приклад 16.1: Word Ladder — BFS через шаблони з «*»
string begin = "hit";
string end = "cog";
string[] dictionary = ["hot", "dot", "dog", "lot", "log", "cog"];

var (length, ladder) = Solve(begin, end, dictionary);
Console.WriteLine(length == 0
    ? "Ланцюжка не існує"
    : $"Довжина: {length}, ланцюжок: {string.Join(" → ", ladder)}");

var (noLength, _) = Solve("hit", "cog", ["hot", "dot", "dog", "lot", "log"]);
Console.WriteLine($"Без 'cog' у словнику: {noLength}");

static (int Length, List<string> Ladder) Solve(string begin, string end, string[] words)
{
    var wordSet = new HashSet<string>(words);
    if (!wordSet.Contains(end)) return (0, []);

    // Шаблон → слова, що йому відповідають
    var buckets = new Dictionary<string, List<string>>();
    foreach (string word in wordSet.Append(begin))
    {
        foreach (string pattern in Patterns(word))
        {
            if (!buckets.TryGetValue(pattern, out var list))
            {
                buckets[pattern] = list = [];
            }
            list.Add(word);
        }
    }

    var parent = new Dictionary<string, string?> { [begin] = null };
    var queue = new Queue<string>([begin]);
    while (queue.Count > 0)
    {
        string word = queue.Dequeue();
        if (word == end)
        {
            var ladder = new List<string>();
            for (string? w = end; w is not null; w = parent[w]) ladder.Add(w);
            ladder.Reverse();
            return (ladder.Count, ladder);
        }
        foreach (string pattern in Patterns(word))
        {
            foreach (string next in buckets[pattern])
            {
                if (parent.TryAdd(next, word)) // TryAdd = «якщо ще не відвідане — відвідати»
                {
                    queue.Enqueue(next);
                }
            }
            buckets[pattern].Clear(); // шаблон використано — більше не перевіряємо (оптимізація)
        }
    }
    return (0, []);
}

static IEnumerable<string> Patterns(string word)
{
    for (int i = 0; i < word.Length; i++)
    {
        yield return string.Concat(word.AsSpan(0, i), "*", word.AsSpan(i + 1));
    }
}
```

**Приклад запуску:**

```text
Довжина: 5, ланцюжок: hit → hot → dot → dog → cog
Без 'cog' у словнику: 0
```

### 16.2 Clone Graph (DFS + словник «оригінал → копія»)

Глибоко скопіювати зв'язний неорієнтований граф, заданий вузлом з посиланнями на сусідів.
Ключ — `Dictionary<Node, Node>`: він водночас і `visited`, і спосіб **не створити копію
двічі** (інакше цикли дадуть нескінченну рекурсію).

```csharp
// Приклад 16.2: глибоке копіювання графа з циклами
// Граф-квадрат: 1 - 2 - 3 - 4 - 1
var nodes = Enumerable.Range(1, 4).Select(i => new Node(i)).ToArray();
void Link(int a, int b)
{
    nodes[a - 1].Neighbors.Add(nodes[b - 1]);
    nodes[b - 1].Neighbors.Add(nodes[a - 1]);
}
Link(1, 2);
Link(2, 3);
Link(3, 4);
Link(4, 1);

Node copy = Clone(nodes[0]);

Console.WriteLine($"Оригінал: {Describe(nodes[0])}");
Console.WriteLine($"Копія:    {Describe(copy)}");
Console.WriteLine($"Той самий об'єкт? {ReferenceEquals(copy, nodes[0])}");
Console.WriteLine($"Сусід копії — з копії? {!nodes.Contains(copy.Neighbors[0])}");

static Node Clone(Node start)
{
    var copies = new Dictionary<Node, Node>(ReferenceEqualityComparer.Instance);
    return CloneNode(start);

    Node CloneNode(Node original)
    {
        if (copies.TryGetValue(original, out Node? existing))
        {
            return existing; // вже скопійований (або в процесі) — повертаємо ту саму копію
        }
        var clone = new Node(original.Value);
        copies[original] = clone; // ВАЖЛИВО: реєструємо ДО обходу сусідів, інакше цикл → нескінченна рекурсія
        foreach (Node neighbor in original.Neighbors)
        {
            clone.Neighbors.Add(CloneNode(neighbor));
        }
        return clone;
    }
}

static string Describe(Node start)
{
    var seen = new HashSet<Node>(ReferenceEqualityComparer.Instance) { start };
    var queue = new Queue<Node>([start]);
    var parts = new List<string>();
    while (queue.Count > 0)
    {
        Node node = queue.Dequeue();
        parts.Add($"{node.Value}:[{string.Join(",", node.Neighbors.Select(x => x.Value))}]");
        foreach (Node next in node.Neighbors.Where(seen.Add))
        {
            queue.Enqueue(next);
        }
    }
    return string.Join(" ", parts);
}

sealed class Node(int value)
{
    public int Value { get; } = value;

    public List<Node> Neighbors { get; } = [];
}
```

**Приклад запуску:**

```text
Оригінал: 1:[2,4] 2:[1,3] 4:[3,1] 3:[2,4]
Копія:    1:[2,4] 2:[1,3] 4:[3,1] 3:[2,4]
Той самий об'єкт? False
Сусід копії — з копії? True
```

### 16.3 Network Delay Time (Дейкстра)

Мережа з `n` вузлів, орієнтовані зважені ребра `(u, v, time)`. Сигнал відправлено з вузла
`k`. За який час сигнал отримають **усі** вузли? Якщо хтось недосяжний — `-1`.

Відповідь — `max(dist)` після Дейкстри.

```csharp
// Приклад 16.3: Network Delay Time — максимум серед найкоротших відстаней
Console.WriteLine(NetworkDelay([[2, 1, 1], [2, 3, 1], [3, 4, 1]], n: 4, k: 2));
Console.WriteLine(NetworkDelay([[1, 2, 1]], n: 2, k: 2));
Console.WriteLine(NetworkDelay([[1, 2, 4], [1, 3, 1], [3, 2, 1], [2, 4, 1], [3, 4, 5]], n: 4, k: 1));

static int NetworkDelay(int[][] times, int n, int k)
{
    // Вузли 1..n → масив на n + 1
    var adjacency = Enumerable.Range(0, n + 1).Select(_ => new List<(int To, int W)>()).ToArray();
    foreach (int[] t in times) adjacency[t[0]].Add((t[1], t[2]));

    var dist = Enumerable.Repeat(int.MaxValue, n + 1).ToArray();
    dist[k] = 0;
    var heap = new PriorityQueue<int, int>();
    heap.Enqueue(k, 0);

    while (heap.TryDequeue(out int u, out int d))
    {
        if (d > dist[u]) continue;
        foreach (var (v, w) in adjacency[u])
        {
            if (d + w < dist[v])
            {
                dist[v] = d + w;
                heap.Enqueue(v, dist[v]);
            }
        }
    }

    int answer = dist.Skip(1).Max(); // пропускаємо фіктивний вузол 0
    return answer == int.MaxValue ? -1 : answer;
}
```

**Приклад запуску:**

```text
2
-1
3
```

### 16.4 Cheapest Flights Within K Stops (Беллман–Форд з обмеженням)

Знайти найдешевший маршрут `src → dst`, що має **не більше `k` пересадок** (тобто ≤ `k + 1`
рейсів). Звичайна Дейкстра тут ламається: найдешевший шлях до проміжного міста може
«витратити» забагато пересадок.

Рішення: `k + 1` ітерація Беллмана–Форда, де кожна ітерація читає відстані з
**копії попередньої** — тоді за ітерацію шлях подовжується **рівно на одне** ребро.

```text
   0 ──100──► 1 ──100──► 2 ──200──► 3
   │          │                     ▲
   │          └────────600──────────┤
   └──────────────────500───────────┘

Рейси: 0→1:100, 1→2:100, 2→3:200, 1→3:600, 0→3:500
k = 0: лише прямий 0→3 = 500
k = 1: 0→1→3 = 700 — гірше, тож 500
k = 2: 0→1→2→3 = 400
```

```csharp
// Приклад 16.4: найдешевші рейси з ≤ k пересадок
int[][] flights = [[0, 1, 100], [1, 2, 100], [2, 3, 200], [1, 3, 600], [0, 3, 500]];
for (int k = 0; k <= 2; k++)
{
    Console.WriteLine($"k = {k}: {CheapestPrice(4, flights, 0, 3, k)}");
}

// Демонстрація, чому потрібна копія масиву
Console.WriteLine($"Без копії, k = 0: {CheapestPriceBuggy(4, flights, 0, 3, 0)} (неправильно!)");

static int CheapestPrice(int n, int[][] flights, int src, int dst, int k)
{
    const int Inf = int.MaxValue;
    var price = Enumerable.Repeat(Inf, n).ToArray();
    price[src] = 0;

    for (int i = 0; i <= k; i++) // k пересадок = k + 1 рейс = k + 1 ітерація
    {
        var next = (int[])price.Clone(); // читаємо зі СТАРОГО, пишемо в НОВИЙ
        foreach (int[] f in flights)
        {
            int from = f[0], to = f[1], cost = f[2];
            if (price[from] != Inf && price[from] + cost < next[to])
            {
                next[to] = price[from] + cost;
            }
        }
        price = next;
    }
    return price[dst] == Inf ? -1 : price[dst];
}

static int CheapestPriceBuggy(int n, int[][] flights, int src, int dst, int k)
{
    var price = Enumerable.Repeat(int.MaxValue, n).ToArray();
    price[src] = 0;
    for (int i = 0; i <= k; i++)
    {
        foreach (int[] f in flights)
        {
            // Помилка: оновлення цієї ж ітерації одразу використовуються далі,
            // тож за одну ітерацію шлях може подовжитись на кілька ребер
            if (price[f[0]] != int.MaxValue && price[f[0]] + f[2] < price[f[1]])
            {
                price[f[1]] = price[f[0]] + f[2];
            }
        }
    }
    return price[dst] == int.MaxValue ? -1 : price[dst];
}
```

**Приклад запуску:**

```text
k = 0: 500
k = 1: 500
k = 2: 400
Без копії, k = 0: 400 (неправильно!)
```

### 16.5 Порядок збірки проєктів (dependency resolution)

Система збірки (MSBuild, Gradle, Bazel) має набір проєктів із залежностями. Потрібно:

1. видати **порядок збірки** (детермінований — за алфавітом серед доступних);
2. згрупувати в **паралельні хвилі** (проєкти однієї хвилі можна збирати одночасно);
3. при циклі — **показати сам цикл**, щоб розробник міг його розірвати.

```text
Проєкт          Залежить від
Api             Application, Infrastructure
Application     Domain
Infrastructure  Application, Shared
Domain          Shared
Shared          —
Tests           Domain

Shared ◄── Domain ◄── Application ◄── Infrastructure ◄── Api
  ▲          ▲                             │
  │          └── Tests                     │
  └────────────────────────────────────────┘
```

```csharp
// Приклад 16.5: порядок збірки з хвилями паралелізму та звітом про цикл
var projects = new Dictionary<string, string[]>
{
    ["Api"] = ["Application", "Infrastructure"],
    ["Application"] = ["Domain"],
    ["Infrastructure"] = ["Application", "Shared"],
    ["Domain"] = ["Shared"],
    ["Shared"] = [],
    ["Tests"] = ["Domain"],
};

var resolver = new BuildResolver(projects);
Console.WriteLine(resolver.Describe());

Console.WriteLine();
projects["Shared"] = ["Api"]; // хтось додав «зручне» посилання Shared → Api
Console.WriteLine(new BuildResolver(projects).Describe());

/// <summary>Визначає порядок збірки проєктів: Кан з лексикографічним вибором і хвилями.</summary>
sealed class BuildResolver
{
    private readonly Dictionary<string, string[]> _dependencies;

    public BuildResolver(Dictionary<string, string[]> dependencies)
    {
        _dependencies = dependencies;
    }

    public string Describe()
    {
        // Граф «залежність → залежний»: спершу збираємо те, від чого залежать
        var dependents = _dependencies.Keys.ToDictionary(p => p, _ => new List<string>());
        var remaining = _dependencies.ToDictionary(p => p.Key, p => p.Value.Length);
        foreach (var (project, deps) in _dependencies)
        {
            foreach (string dep in deps)
            {
                if (!dependents.ContainsKey(dep))
                {
                    return $"Помилка: {project} посилається на невідомий проєкт {dep}";
                }
                dependents[dep].Add(project);
            }
        }

        // Хвиля = усі проєкти, у яких на цей момент немає незібраних залежностей
        var wave = new SortedSet<string>(remaining.Where(p => p.Value == 0).Select(p => p.Key), StringComparer.Ordinal);
        var lines = new List<string>();
        int built = 0;
        while (wave.Count > 0)
        {
            lines.Add($"Хвиля {lines.Count + 1}: {string.Join(", ", wave)}");
            built += wave.Count;
            var nextWave = new SortedSet<string>(StringComparer.Ordinal);
            foreach (string project in wave)
            {
                foreach (string dependent in dependents[project])
                {
                    if (--remaining[dependent] == 0) nextWave.Add(dependent);
                }
            }
            wave = nextWave;
        }

        if (lines.Count == 0)
        {
            lines.Add("Жоден проєкт не можна зібрати першим");
        }

        if (built == _dependencies.Count)
        {
            return string.Join(Environment.NewLine, lines);
        }

        return $"{string.Join(Environment.NewLine, lines)}{Environment.NewLine}Цикл залежностей: {FindCycle(remaining)}";
    }

    private string FindCycle(Dictionary<string, int> remaining)
    {
        // Незібрані проєкти містять цикл. Йдемо по незібраних залежностях, доки не повторимось.
        string current = remaining.Where(p => p.Value > 0).Select(p => p.Key).Order(StringComparer.Ordinal).First();
        var path = new List<string>();
        var position = new Dictionary<string, int>();
        while (!position.ContainsKey(current))
        {
            position[current] = path.Count;
            path.Add(current);
            current = _dependencies[current].Where(d => remaining[d] > 0).Order(StringComparer.Ordinal).First();
        }
        var cycle = path.Skip(position[current]).Append(current);
        return string.Join(" → ", cycle);
    }
}
```

**Приклад запуску:**

```text
Хвиля 1: Shared
Хвиля 2: Domain
Хвиля 3: Application, Tests
Хвиля 4: Infrastructure
Хвиля 5: Api

Жоден проєкт не можна зібрати першим
Цикл залежностей: Api → Application → Domain → Shared → Api
```

> У кожного незібраного проєкту є хоча б одна незібрана залежність (інакше він потрапив би
> в хвилю), тож «ходіння» по незібраних залежностях ніколи не застрягне і рано чи пізно
> повториться — це і є цикл.

---

## 17. Питання для самоперевірки та задачі

### 17.1 Питання

1. Чому `Σ deg(v) = 2m` і що з цього випливає про кількість вершин непарного степеня?
2. Коли матриця суміжності краща за список? Скільки пам'яті займе `bool[n, n]` для `n = 50 000`?
3. Чому у BFS вершину треба позначати відвіданою **при додаванні** в чергу?
4. Чому BFS дає найкоротші шляхи в незваженому графі, а DFS — ні?
5. Як multi-source BFS зводиться до звичайного BFS?
6. Чим 0-1 BFS відрізняється від звичайного BFS і чому вершина може потрапити в дек двічі?
7. Навіщо в DFS для орграфа три кольори, а не `bool visited`?
8. Що таке back-, forward- і cross-ребра? Які з них можливі в неорієнтованому графі?
9. Чому розвернутий порядок виходу DFS — топологічний?
10. Як алгоритм Кана виявляє цикл?
11. Що таке «ліниве видалення» в Дейкстрі і яка через нього складність?
12. Наведіть граф із від'ємним ребром (без від'ємних циклів), на якому Дейкстра помиляється.
13. Чому Беллману–Форду достатньо `n − 1` ітерацій? Як знайти сам від'ємний цикл?
14. Чому в Флойді–Воршеллі цикл по `k` має бути зовнішнім?
15. Чому найдовший шлях у DAG простий, а в загальному графі — NP-складний?
16. Яка умова на евристику гарантує оптимальність A*? Чи допустима Манхеттенська відстань при 8 напрямках руху?
17. Що дають стиснення шляху та об'єднання за рангом окремо і разом?
18. Чим відрізняються пріоритети в Прімі та в Дейкстрі?
19. Чому граф конденсації завжди ациклічний?
20. Чому для мостів умова `low[v] > tin[u]`, а для точок зчленування `low[v] >= tin[u]`?
21. У задачі «Cheapest Flights Within K Stops» навіщо копіювати масив цін на кожній ітерації?

<details>
<summary>Короткі відповіді</summary>

1. Кожне ребро додає 1 до степеня двох вершин; кількість вершин непарного степеня парна.
2. Для щільних графів, частих перевірок ребра, Флойда; `50 000² = 2.5·10⁹` байт ≈ 2.3 ГБ.
3. Інакше одна вершина потрапить у чергу від кожного сусіда — зайва пам'ять і час.
4. BFS обробляє вершини за зростанням відстані (шарами); DFS іде вглиб першим-ліпшим шляхом.
5. Фіктивна вершина з ребрами ваги 0 (або 1) до всіх джерел; на практиці — усі джерела в чергу на старті.
6. Дек: ребра ваги 0 — на початок; вершина повторно додається, коли її відстань покращилась.
7. «Відвідана» вершина може бути вже завершеною (cross/forward-ребро) — це не цикл; цикл — лише ребро до сірої.
8. Back — до предка, forward — до нащадка не деревним ребром, cross — між гілками. Неорієнтований: лише tree і back.
9. При виході з `u` усі досяжні з `u` вершини вже вийшли → стоять у списку раніше → після розвороту — пізніше за `u`.
10. Вершини циклу ніколи не отримують `inDegree = 0`; у відповіді менше `n` вершин.
11. Не зменшуємо ключ, а додаємо дублікат і пропускаємо застарілі; `O((n + m) log n)`.
12. Приклад 6.2: `0→2:2, 0→1:5, 1→2:−4, 2→3:1`.
13. Простий найкоротший шлях має ≤ `n − 1` ребер. Оновлення на `n`-й ітерації → `n` кроків по `parent` → обхід циклу.
14. Інваріант: після ітерації `k` враховано всі шляхи через проміжні `0..k`; інакше рядок/стовпець `k` ще не готовий.
15. У DAG немає циклів — топологічний порядок дає DP; у загальному графі це узагальнення гамільтонового шляху.
16. Допустимість (не переоцінює); для однократної обробки — монотонність. При 8 напрямках Манхеттенська переоцінює → ні.
17. Кожна окремо дає `O(log n)` амортизовано, разом — `O(α(n))`.
18. Прім — вага одного ребра до дерева; Дейкстра — сумарна відстань від старту.
19. Цикл між SCC означав би, що вони взаємно досяжні, тобто мали б бути однією SCC.
20. Для мосту піддерево `v` не повинно досягати навіть `u` в обхід ребра; для вершини `u` досить не досягати предків `u`.
21. Щоб за одну ітерацію шлях подовжувався рівно на одне ребро і не порушувалось обмеження `k`.
</details>

### 17.2 Задачі для практики

| # | Задача | Ключова ідея | Складність |
|---|---|---|---|
| 1 | Rotting Oranges | multi-source BFS по шарах | ⭐ |
| 2 | Flood Fill | DFS/BFS на сітці | ⭐ |
| 3 | Find if Path Exists in Graph | BFS / DSU | ⭐ |
| 4 | Number of Provinces | компоненти / DSU | ⭐⭐ |
| 5 | Is Graph Bipartite? | розфарбування BFS | ⭐⭐ |
| 6 | Course Schedule I / II | Кан | ⭐⭐ |
| 7 | Shortest Path in Binary Matrix | BFS, 8 напрямків | ⭐⭐ |
| 8 | Minimum Cost to Make at Least One Valid Path in a Grid | 0-1 BFS | ⭐⭐⭐ |
| 9 | Path With Minimum Effort | Дейкстра з `max` / бінпошук + BFS | ⭐⭐ |
| 10 | Min Cost to Connect All Points | Прім на масиві `O(n²)` | ⭐⭐ |
| 11 | Accounts Merge | DSU над email | ⭐⭐ |
| 12 | Find Eventual Safe States | кольори / Кан на `Gᵀ` | ⭐⭐ |
| 13 | Critical Connections in a Network | мости (low-link) | ⭐⭐⭐ |
| 14 | Parallel Courses III | найдовший шлях у DAG | ⭐⭐⭐ |
| 15 | Alien Dictionary | побудова графа з порядку слів + топосорт | ⭐⭐⭐ |
| 16 | Swim in Rising Water | Дейкстра з `max` / DSU за часом | ⭐⭐⭐ |
| 17 | Arbitrage (власна) | Беллман–Форд на `−log(rate)` | ⭐⭐⭐ |

### 17.3 Завдання для самостійної роботи

1. Розширте клас `Graph` із прикладу 2.4 методами `Bfs`, `Dfs`, `TopologicalSort` і `Dijkstra`,
   що повертають незмінні результати (`record`). Покрийте їх тестами на графі G.
2. Реалізуйте ітеративну версію Тарʼяна для мостів і перевірте її на «ланцюжку» з 10⁶ вершин
   (рекурсивна версія має впасти).
3. Порівняйте на випадковому графі (`n = 10⁵`, `m = 5·10⁵`) час Дейкстри з `PriorityQueue`
   і з `SortedSet<(long, int)>` (зі справжнім DecreaseKey через `Remove`/`Add`).
4. Реалізуйте A* для 8 напрямків з октильною евристикою та порівняйте кількість розкритих
   вершин з Дейкстрою на карті 200 × 200 з випадковими стінами.
5. Напишіть утиліту, що читає `.csproj`-файли (`<ProjectReference>`) у каталозі та друкує
   хвилі збірки й циклічні посилання (на основі прикладу 16.5).

### Міні-вправа 17 (фінальна)

Дано карту міста (зважений неорієнтований граф), перелік пожежних станцій і запити
«за скільки хвилин приїде найближча машина до вершини `v`». Яким алгоритмом відповісти
на **всі** запити одразу?

<details>
<summary>Розв'язок</summary>

**Multi-source Дейкстра**: покласти всі станції в купу з відстанню 0 і запустити Дейкстру
один раз. `dist[v]` — час від найближчої станції. Складність `O((n + m) log n)` незалежно
від кількості станцій — та сама ідея, що й multi-source BFS із розділу 3.5.
</details>

---

## Підсумок

- Граф = вершини + ребра; майже завжди зберігаємо **списком суміжності**.
- **BFS** — шари та найкоротші шляхи без ваг; **DFS** — структура: часи, цикли, топосорт, SCC, мости.
- Найкоротші шляхи: **BFS → 0-1 BFS → DAG-DP → Дейкстра → A* → Беллман–Форд → Флойд–Воршелл**
  у порядку зростання загальності (і зазвичай вартості).
- **MST**: Краскал + Union-Find або Прім + купа; **Union-Find** корисний далеко за межами MST.
- Перш ніж писати код, визначте: орієнтований? зважений? від'ємні ваги? DAG? сітка? — і
  скористайтеся таблицею з розділу 15.
