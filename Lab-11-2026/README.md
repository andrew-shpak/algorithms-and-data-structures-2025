# Lab-11-2026 — Пошук найкоротших шляхів (Дейкстра + A*)

## [IDE](https://onecompiler.com/csharp)

---

## Завдання 1: Алгоритм Дейкстри (5 балів)

### Мета

Реалізувати алгоритм Дейкстри для знаходження найкоротших шляхів від початкової вершини до всіх інших вершин у зваженому графі з невід'ємними вагами ребер. Реалізувати відновлення шляху через масив попередників.

- **Складність:** O((V + E) log V) з пріоритетною чергою

### Підказка

Алгоритм Дейкстри — жадібний алгоритм:
1. Ініціалізувати відстані: `dist[start] = 0`, для решти — `INF`
2. Використовувати `PriorityQueue<TElement, TPriority>` (min-heap) для вибору вершини з мінімальною відстанню
3. Для кожної вершини перевірити всіх сусідів і виконати **релаксацію**: якщо `dist[u] + weight(u,v) < dist[v]`, оновити `dist[v]`
4. Зберігати `parent[v] = u` при кожній релаксації для відновлення шляху

Увага: `PriorityQueue<TElement, TPriority>` у .NET за замовчуванням — min-heap (першим виходить елемент із найменшим пріоритетом). Для max-heap передайте власний `IComparer<TPriority>` або зберігайте від'ємні пріоритети. Метод оновлення пріоритету відсутній, тому додавайте вершину повторно й пропускайте застарілі записи.

### Приклад `main`

```csharp
using Graph = System.Collections.Generic.List<System.Collections.Generic.List<Edge>>;

// Граф 1: зв'язний неорієнтований
//
//    (0)---4---(1)
//     |       / |
//     7     2   3
//     |   /     |
//    (2)---1---(4)
//       \
//        2
//         \
//         (3)

Graph g1 = CreateGraph(5);
static void AddEdge(Graph g, int u, int v, int w)
{
    g[u].Add(new Edge(v, w));
    g[v].Add(new Edge(u, w));
}
AddEdge(g1, 0, 1, 4);
AddEdge(g1, 0, 2, 7);
AddEdge(g1, 1, 2, 2);
AddEdge(g1, 1, 3, 3);
AddEdge(g1, 2, 3, 2);
AddEdge(g1, 2, 4, 1);
AddEdge(g1, 3, 4, 5);

Console.WriteLine("=== Граф 1: зв'язний ===");
int[] dist1 = ShortestPaths.Dijkstra(g1, 0, out int[] parent1);
ShortestPaths.PrintResults(0, dist1, parent1);

// Граф 2: з недосяжними вершинами
Graph g2 = CreateGraph(5);
g2[0].Add(new Edge(1, 3));
g2[1].Add(new Edge(0, 3));
g2[1].Add(new Edge(2, 1));
g2[2].Add(new Edge(1, 1));
// Вершини 3, 4 — ізольована компонента
g2[3].Add(new Edge(4, 2));
g2[4].Add(new Edge(3, 2));

Console.WriteLine("\n=== Граф 2: з недосяжними вершинами ===");
int[] dist2 = ShortestPaths.Dijkstra(g2, 0, out int[] parent2);
ShortestPaths.PrintResults(0, dist2, parent2);

// Граф 3: орієнтований ланцюг
Graph g3 = CreateGraph(4);
g3[0].Add(new Edge(1, 1));
g3[1].Add(new Edge(2, 2));
g3[2].Add(new Edge(3, 3));

Console.WriteLine("\n=== Граф 3: орієнтований ланцюг ===");
int[] dist3 = ShortestPaths.Dijkstra(g3, 0, out int[] parent3);
ShortestPaths.PrintResults(0, dist3, parent3);

static Graph CreateGraph(int vertexCount) =>
    Enumerable.Range(0, vertexCount).Select(_ => new List<Edge>()).ToList();

public readonly record struct Edge(int To, int Weight);

public static class ShortestPaths
{
    public const int Inf = int.MaxValue;

    // Алгоритм Дейкстри
    // Повертає масив найкоротших відстаней від start до всіх вершин
    // parent — вихідний параметр для відновлення шляхів
    public static int[] Dijkstra(List<List<Edge>> graph, int start, out int[] parent)
    {
        // Ваш код тут
    }

    // Відновлює шлях від start до end, використовуючи масив parent
    // Повертає список вершин шляху або порожній список, якщо шлях не існує
    public static List<int> RestorePath(int start, int end, int[] parent)
    {
        // Ваш код тут
    }

    public static void PrintResults(int start, int[] dist, int[] parent)
    {
        Console.WriteLine($"\nНайкоротші відстані від вершини {start}:");
        Console.WriteLine("Вершина\tВідстань\tШлях");
        Console.WriteLine("-------\t--------\t----");

        for (int i = 0; i < dist.Length; i++)
        {
            Console.Write($"{i}\t");
            if (dist[i] == Inf)
            {
                Console.Write("INF\t\tнедосяжна");
            }
            else
            {
                Console.Write($"{dist[i]}\t\t");
                List<int> path = RestorePath(start, i, parent);
                Console.Write(string.Join(" -> ", path));
            }
            Console.WriteLine();
        }
    }
}
```

### Приклад запуску

```text
=== Граф 1: зв'язний ===

Найкоротші відстані від вершини 0:
Вершина	Відстань	Шлях
-------	--------	----
0	0		0
1	4		0 -> 1
2	6		0 -> 1 -> 2
3	7		0 -> 1 -> 3
4	7		0 -> 1 -> 2 -> 4

=== Граф 2: з недосяжними вершинами ===

Найкоротші відстані від вершини 0:
Вершина	Відстань	Шлях
-------	--------	----
0	0		0
1	3		0 -> 1
2	4		0 -> 1 -> 2
3	INF		недосяжна
4	INF		недосяжна

=== Граф 3: орієнтований ланцюг ===

Найкоротші відстані від вершини 0:
Вершина	Відстань	Шлях
-------	--------	----
0	0		0
1	1		0 -> 1
2	3		0 -> 1 -> 2
3	6		0 -> 1 -> 2 -> 3
```

---

## Завдання 2: Алгоритм A* на сітці (5 балів)

### Мета

Реалізувати алгоритм A* для пошуку найкоротшого шляху на двовимірній сітці з перешкодами. Використовувати Манхеттенську відстань як евристику. Вивести знайдений шлях та кількість відвіданих клітинок.

- **Складність:** O(V log V) у найгіршому випадку, на практиці значно швидше за Дейкстру завдяки евристиці

### Підказка

A* — розширення алгоритму Дейкстри з евристикою:
- `g(n)` — реальна вартість шляху від старту до вершини `n` (як у Дейкстри)
- `h(n)` — евристична оцінка вартості від `n` до цілі (Манхеттенська відстань: `|x1-x2| + |y1-y2|`)
- `f(n) = g(n) + h(n)` — пріоритет вершини у черзі

Ключова відмінність від Дейкстри: замість вибору вершини з мінімальним `g(n)`, A* вибирає вершину з мінімальним `f(n)`, що направляє пошук у бік цілі.

Рухи дозволені у 4 напрямках (вгору, вниз, вліво, вправо), кожен коштує 1.

### Приклад `main`

```csharp
// Сітка 1: простий шлях з перешкодами
int[,] grid1 =
{
    { 0, 0, 0, 0, 0, 0, 0 },
    { 0, 0, 0, 1, 0, 0, 0 },
    { 0, 0, 0, 1, 0, 0, 0 },
    { 0, 0, 0, 1, 0, 0, 0 },
    { 0, 0, 0, 0, 0, 0, 0 },
};

Cell start1 = new(2, 0), goal1 = new(2, 6);

Console.WriteLine("=== Сітка 1: стіна посередині ===");
List<Cell> path1 = GridSearch.AStar(grid1, start1, goal1, out int visited1);

if (path1.Count > 0)
{
    GridSearch.PrintGrid(grid1, path1, start1, goal1);
    Console.WriteLine($"Довжина шляху: {path1.Count - 1}");
    Console.WriteLine($"Відвідано клітинок: {visited1}");
}
else
{
    Console.WriteLine("Шлях не знайдено!");
}

// Сітка 2: лабіринт
int[,] grid2 =
{
    { 0, 1, 0, 0, 0 },
    { 0, 1, 0, 1, 0 },
    { 0, 0, 0, 1, 0 },
    { 0, 1, 1, 1, 0 },
    { 0, 0, 0, 0, 0 },
};

Cell start2 = new(0, 0), goal2 = new(4, 4);

Console.WriteLine("\n=== Сітка 2: лабіринт ===");
List<Cell> path2 = GridSearch.AStar(grid2, start2, goal2, out int visited2);

if (path2.Count > 0)
{
    GridSearch.PrintGrid(grid2, path2, start2, goal2);
    Console.WriteLine($"Довжина шляху: {path2.Count - 1}");
    Console.WriteLine($"Відвідано клітинок: {visited2}");
}
else
{
    Console.WriteLine("Шлях не знайдено!");
}

// Сітка 3: шлях не існує
int[,] grid3 =
{
    { 0, 0, 1 },
    { 1, 1, 1 },
    { 0, 0, 0 },
};

Cell start3 = new(0, 0), goal3 = new(2, 2);

Console.WriteLine("\n=== Сітка 3: немає шляху ===");
List<Cell> path3 = GridSearch.AStar(grid3, start3, goal3, out int visited3);

if (path3.Count > 0)
{
    GridSearch.PrintGrid(grid3, path3, start3, goal3);
    Console.WriteLine($"Довжина шляху: {path3.Count - 1}");
}
else
{
    Console.WriteLine("Шлях не знайдено!");
    Console.WriteLine($"Відвідано клітинок: {visited3}");
}

// record struct автоматично дає порівняння на рівність (==, !=, Equals, GetHashCode)
public readonly record struct Cell(int Row, int Col);

public static class GridSearch
{
    // Манхеттенська відстань
    public static int Heuristic(Cell a, Cell b)
    {
        // Ваш код тут
    }

    // Алгоритм A*
    // grid: 0 = вільна клітинка, 1 = перешкода
    // Повертає список клітинок шляху від start до goal (включно)
    // visitedCount — вихідний параметр: кількість відвіданих клітинок
    // Черга: PriorityQueue<(Cell Cell, int G), int> з пріоритетом f = g + h
    public static List<Cell> AStar(int[,] grid, Cell start, Cell goal, out int visitedCount)
    {
        // Ваш код тут
    }

    public static void PrintGrid(int[,] grid, List<Cell> path, Cell start, Cell goal)
    {
        // Копіюємо сітку для візуалізації
        int rows = grid.GetLength(0), cols = grid.GetLength(1);
        var display = new char[rows, cols];

        for (int r = 0; r < rows; r++)
            for (int c = 0; c < cols; c++)
                display[r, c] = grid[r, c] == 1 ? '#' : '.';

        // Позначаємо шлях
        foreach (Cell cell in path)
            display[cell.Row, cell.Col] = '*';

        display[start.Row, start.Col] = 'S';
        display[goal.Row, goal.Col] = 'G';

        for (int r = 0; r < rows; r++)
        {
            for (int c = 0; c < cols; c++)
                Console.Write($"{display[r, c]} ");
            Console.WriteLine();
        }
    }
}
```

### Приклад запуску

```text
=== Сітка 1: стіна посередині ===
. . * * * * *
. . * # . . *
S * * # . . G
. . . # . . .
. . . . . . .
Довжина шляху: 10
Відвідано клітинок: 23

=== Сітка 2: лабіринт ===
S # . . .
* # . # .
* . . # .
* # # # .
* * * * G
Довжина шляху: 8
Відвідано клітинок: 11

=== Сітка 3: немає шляху ===
Шлях не знайдено!
Відвідано клітинок: 2
```

---

## Сумарні бали

| Завдання | Тема | Бали |
|----------|------|------|
| Завдання 1 | Алгоритм Дейкстри | 5 |
| Завдання 2 | Алгоритм A* на сітці | 5 |

**Всього: 10 балів**
