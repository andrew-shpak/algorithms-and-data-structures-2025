# Lab-11-2026 — Пошук найкоротших шляхів (Дейкстра + A*)

## [IDE](https://onecompiler.com/cpp)

---

## Завдання 1: Алгоритм Дейкстри (5 балів)

### Мета

Реалізувати алгоритм Дейкстри для знаходження найкоротших шляхів від початкової вершини до всіх інших вершин у зваженому графі з невід'ємними вагами ребер. Реалізувати відновлення шляху через масив попередників.

- **Складність:** O((V + E) log V) з пріоритетною чергою

### Підказка

Алгоритм Дейкстри — жадібний алгоритм:
1. Ініціалізувати відстані: `dist[start] = 0`, для решти — `INF`
2. Використовувати `priority_queue` (min-heap) для вибору вершини з мінімальною відстанню
3. Для кожної вершини перевірити всіх сусідів і виконати **релаксацію**: якщо `dist[u] + weight(u,v) < dist[v]`, оновити `dist[v]`
4. Зберігати `parent[v] = u` при кожній релаксації для відновлення шляху

Увага: `priority_queue` у C++ за замовчуванням — max-heap. Для min-heap використовуйте `greater<>` або зберігайте від'ємні відстані.

### Приклад `main`

```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <limits>
#include <algorithm>
using namespace std;

const int INF = numeric_limits<int>::max();

struct Edge {
    int to, weight;
    Edge(int t, int w) : to(t), weight(w) {}
};

using Graph = vector<vector<Edge>>;

// Алгоритм Дейкстри
// Повертає вектор найкоротших відстаней від start до всіх вершин
// parent — вихідний параметр для відновлення шляхів
vector<int> dijkstra(const Graph& graph, int start, vector<int>& parent) {
    // Ваш код тут
}

// Відновлює шлях від start до end, використовуючи масив parent
// Повертає вектор вершин шляху або порожній вектор, якщо шлях не існує
vector<int> restore_path(int start, int end, const vector<int>& parent) {
    // Ваш код тут
}

void print_results(int start, const vector<int>& dist, const vector<int>& parent) {
    cout << "\nНайкоротші відстані від вершини " << start << ":\n";
    cout << "Вершина\tВідстань\tШлях\n";
    cout << "-------\t--------\t----\n";

    for (int i = 0; i < (int)dist.size(); i++) {
        cout << i << "\t";
        if (dist[i] == INF) {
            cout << "INF\t\tнедосяжна";
        } else {
            cout << dist[i] << "\t\t";
            vector<int> path = restore_path(start, i, parent);
            for (int j = 0; j < (int)path.size(); j++) {
                if (j > 0) cout << " -> ";
                cout << path[j];
            }
        }
        cout << endl;
    }
}

int main() {
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

    Graph g1(5);
    auto add_edge = [](Graph& g, int u, int v, int w) {
        g[u].push_back(Edge(v, w));
        g[v].push_back(Edge(u, w));
    };
    add_edge(g1, 0, 1, 4);
    add_edge(g1, 0, 2, 7);
    add_edge(g1, 1, 2, 2);
    add_edge(g1, 1, 3, 3);
    add_edge(g1, 2, 3, 2);
    add_edge(g1, 2, 4, 1);
    add_edge(g1, 3, 4, 5);

    cout << "=== Граф 1: зв'язний ===" << endl;
    vector<int> parent1;
    vector<int> dist1 = dijkstra(g1, 0, parent1);
    print_results(0, dist1, parent1);

    // Граф 2: з недосяжними вершинами
    Graph g2(5);
    g2[0].push_back(Edge(1, 3));
    g2[1].push_back(Edge(0, 3));
    g2[1].push_back(Edge(2, 1));
    g2[2].push_back(Edge(1, 1));
    // Вершини 3, 4 — ізольована компонента
    g2[3].push_back(Edge(4, 2));
    g2[4].push_back(Edge(3, 2));

    cout << "\n=== Граф 2: з недосяжними вершинами ===" << endl;
    vector<int> parent2;
    vector<int> dist2 = dijkstra(g2, 0, parent2);
    print_results(0, dist2, parent2);

    // Граф 3: орієнтований ланцюг
    Graph g3(4);
    g3[0].push_back(Edge(1, 1));
    g3[1].push_back(Edge(2, 2));
    g3[2].push_back(Edge(3, 3));

    cout << "\n=== Граф 3: орієнтований ланцюг ===" << endl;
    vector<int> parent3;
    vector<int> dist3 = dijkstra(g3, 0, parent3);
    print_results(0, dist3, parent3);

    return 0;
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

```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <cmath>
#include <algorithm>
using namespace std;

struct Cell {
    int row, col;
    Cell(int r = 0, int c = 0) : row(r), col(c) {}
    bool operator==(const Cell& o) const { return row == o.row && col == o.col; }
    bool operator!=(const Cell& o) const { return !(*this == o); }
};

struct State {
    int f, g;
    Cell cell;
    State(int f, int g, Cell c) : f(f), g(g), cell(c) {}
    bool operator>(const State& o) const { return f > o.f; }
};

// Манхеттенська відстань
int heuristic(Cell a, Cell b) {
    // Ваш код тут
}

// Алгоритм A*
// grid: 0 = вільна клітинка, 1 = перешкода
// Повертає вектор клітинок шляху від start до goal (включно)
// visited_count — вихідний параметр: кількість відвіданих клітинок
vector<Cell> astar(const vector<vector<int>>& grid, Cell start, Cell goal, int& visited_count) {
    // Ваш код тут
}

void print_grid(const vector<vector<int>>& grid, const vector<Cell>& path, Cell start, Cell goal) {
    // Копіюємо сітку для візуалізації
    int rows = grid.size(), cols = grid[0].size();
    vector<vector<char>> display(rows, vector<char>(cols));

    for (int r = 0; r < rows; r++)
        for (int c = 0; c < cols; c++)
            display[r][c] = grid[r][c] == 1 ? '#' : '.';

    // Позначаємо шлях
    for (const Cell& c : path)
        display[c.row][c.col] = '*';

    display[start.row][start.col] = 'S';
    display[goal.row][goal.col] = 'G';

    for (int r = 0; r < rows; r++) {
        for (int c = 0; c < cols; c++)
            cout << display[r][c] << ' ';
        cout << endl;
    }
}

int main() {
    // Сітка 1: простий шлях з перешкодами
    vector<vector<int>> grid1 = {
        {0, 0, 0, 0, 0, 0, 0},
        {0, 0, 0, 1, 0, 0, 0},
        {0, 0, 0, 1, 0, 0, 0},
        {0, 0, 0, 1, 0, 0, 0},
        {0, 0, 0, 0, 0, 0, 0}
    };

    Cell start1(2, 0), goal1(2, 6);
    int visited1 = 0;

    cout << "=== Сітка 1: стіна посередині ===" << endl;
    vector<Cell> path1 = astar(grid1, start1, goal1, visited1);

    if (!path1.empty()) {
        print_grid(grid1, path1, start1, goal1);
        cout << "Довжина шляху: " << path1.size() - 1 << endl;
        cout << "Відвідано клітинок: " << visited1 << endl;
    } else {
        cout << "Шлях не знайдено!" << endl;
    }

    // Сітка 2: лабіринт
    vector<vector<int>> grid2 = {
        {0, 1, 0, 0, 0},
        {0, 1, 0, 1, 0},
        {0, 0, 0, 1, 0},
        {0, 1, 1, 1, 0},
        {0, 0, 0, 0, 0}
    };

    Cell start2(0, 0), goal2(4, 4);
    int visited2 = 0;

    cout << "\n=== Сітка 2: лабіринт ===" << endl;
    vector<Cell> path2 = astar(grid2, start2, goal2, visited2);

    if (!path2.empty()) {
        print_grid(grid2, path2, start2, goal2);
        cout << "Довжина шляху: " << path2.size() - 1 << endl;
        cout << "Відвідано клітинок: " << visited2 << endl;
    } else {
        cout << "Шлях не знайдено!" << endl;
    }

    // Сітка 3: шлях не існує
    vector<vector<int>> grid3 = {
        {0, 0, 1},
        {1, 1, 1},
        {0, 0, 0}
    };

    Cell start3(0, 0), goal3(2, 2);
    int visited3 = 0;

    cout << "\n=== Сітка 3: немає шляху ===" << endl;
    vector<Cell> path3 = astar(grid3, start3, goal3, visited3);

    if (!path3.empty()) {
        print_grid(grid3, path3, start3, goal3);
        cout << "Довжина шляху: " << path3.size() - 1 << endl;
    } else {
        cout << "Шлях не знайдено!" << endl;
        cout << "Відвідано клітинок: " << visited3 << endl;
    }

    return 0;
}
```

### Приклад запуску

```text
=== Сітка 1: стіна посередині ===
. . * * * . .
. . * # * . .
S * * # . * .
. . . # . * .
. . . . . * G
Довжина шляху: 10
Відвідано клітинок: 18

=== Сітка 2: лабіринт ===
S # . . .
* # . # .
* * * # .
. # # # *
. . . * G
Довжина шляху: 10
Відвідано клітинок: 12

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
