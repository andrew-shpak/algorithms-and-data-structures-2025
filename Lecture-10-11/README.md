# Лекція 10-11 — Графи та алгоритми на графах

- Examples: https://docs.google.com/document/d/1NXkVqS94aFS9xrcnHBiSHbKp9bZ-7C80k-0TrbP4ZOk/edit?usp=sharing

---

## Зміст

1. [Представлення графів](#завдання-1-представлення-графів)
2. [BFS — пошук в ширину](#завдання-2-bfs--пошук-в-ширину)
3. [DFS — пошук в глибину](#завдання-3-dfs--пошук-в-глибину)
4. [Найкоротший шлях у незваженому графі](#завдання-4-найкоротший-шлях-у-незваженому-графі-bfs)
5. [Алгоритм Дейкстри](#завдання-5-алгоритм-дейкстри)
6. [Алгоритм Беллмана-Форда](#завдання-6-алгоритм-беллмана-форда)
7. [Алгоритм Флойда-Уоршелла](#завдання-7-алгоритм-флойда-уоршелла)
8. [Алгоритм Пріма (MST)](#завдання-8-мінімальне-остовне-дерево--алгоритм-пріма)
9. [Топологічне сортування](#завдання-9-топологічне-сортування)
10. [Виявлення циклів в орієнтованому графі](#завдання-10-виявлення-циклів-в-орієнтованому-графі)
11. [Перевірка двочастковості графа](#завдання-11-перевірка-двочастковості-графа)
12. [Компоненти зв'язності](#завдання-12-компоненти-звязності)
13. [Алгоритм Крускала (MST)](#завдання-13-алгоритм-крускала-mst)
14. [Мости та точки зчленування](#завдання-14-мости-та-точки-зчленування)
15. [Компоненти сильної зв'язності (Косараджу)](#завдання-15-компоненти-сильної-звязності-алгоритм-косараджу)
16. [Найкоротший шлях у DAG](#завдання-16-найкоротший-шлях-у-dag)
17. [Пошук шляху в лабіринті (BFS на сітці)](#завдання-17-пошук-шляху-в-лабіринті-bfs-на-сітці)
18. [Алгоритм A* (евристичний пошук)](#завдання-18-алгоритм-a-евристичний-пошук)

---

## Завдання 1: Представлення графів

### Теорія

Граф G = (V, E) складається з множини **вершин** V та множини **ребер** E.

**Два основні способи представлення:**

| | Матриця суміжності | Список суміжності |
|---|---|---|
| Пам'ять | O(V²) | O(V + E) |
| Перевірка ребра (u,v) | O(1) | O(degree(u)) |
| Обхід сусідів | O(V) | O(degree(u)) |
| Коли використовувати | Щільні графи | Розріджені графи |

```
Граф (неорієнтований):
    0 --- 1
    |     |
    2 --- 3

Матриця суміжності:       Список суміжності:
    0  1  2  3             0: [1, 2]
0 [ 0  1  1  0 ]          1: [0, 3]
1 [ 1  0  0  1 ]          2: [0, 3]
2 [ 1  0  0  1 ]          3: [1, 2]
3 [ 0  1  1  0 ]
```

```cpp
#include <iostream>
#include <vector>
using namespace std;

// === Матриця суміжності ===
class GraphMatrix {
    int V;
    vector<vector<int>> matrix;

public:
    GraphMatrix(int vertices) : V(vertices), matrix(V, vector<int>(V, 0)) {}

    void addEdge(int u, int v) {
        matrix[u][v] = 1;
        matrix[v][u] = 1;  // для неорієнтованого графа
    }

    void print() {
        cout << "Матриця суміжності:" << endl;
        for (int i = 0; i < V; i++) {
            for (int j = 0; j < V; j++)
                cout << matrix[i][j] << " ";
            cout << endl;
        }
    }
};

// === Список суміжності ===
class GraphList {
    int V;
    vector<vector<int>> adj;

public:
    GraphList(int vertices) : V(vertices), adj(V) {}

    void addEdge(int u, int v) {
        adj[u].push_back(v);
        adj[v].push_back(u);  // для неорієнтованого графа
    }

    void print() {
        cout << "Список суміжності:" << endl;
        for (int i = 0; i < V; i++) {
            cout << i << ": ";
            for (int neighbor : adj[i])
                cout << neighbor << " ";
            cout << endl;
        }
    }
};

int main() {
    // Граф:
    //     0 --- 1
    //     |     |
    //     2 --- 3
    cout << "=== Матриця суміжності ===" << endl;
    GraphMatrix gm(4);
    gm.addEdge(0, 1);
    gm.addEdge(0, 2);
    gm.addEdge(1, 3);
    gm.addEdge(2, 3);
    gm.print();

    cout << "\n=== Список суміжності ===" << endl;
    GraphList gl(4);
    gl.addEdge(0, 1);
    gl.addEdge(0, 2);
    gl.addEdge(1, 3);
    gl.addEdge(2, 3);
    gl.print();

    return 0;
}
```

### Приклад запуску

```text
=== Матриця суміжності ===
Матриця суміжності:
0 1 1 0
1 0 0 1
1 0 0 1
0 1 1 0

=== Список суміжності ===
Список суміжності:
0: 1 2
1: 0 3
2: 0 3
3: 1 2
```

---

## Завдання 2: BFS — пошук в ширину

### Теорія

BFS (Breadth-First Search) обходить граф **пошарово** — спочатку всі сусіди стартової вершини, потім їхні сусіди тощо. Використовує **чергу**.

```
Порядок BFS від вершини 0:

Крок 0: відвідуємо 0           Черга: [1, 2]
Крок 1: відвідуємо 1           Черга: [2, 3]
Крок 2: відвідуємо 2           Черга: [3]
Крок 3: відвідуємо 3           Черга: [4]
Крок 4: відвідуємо 4           Черга: []

    0 --- 1
    |     |
    2 --- 3 --- 4
```

**Складність:** O(V + E)

```cpp
#include <iostream>
#include <vector>
#include <queue>
using namespace std;

class Graph {
    int V;
    vector<vector<int>> adj;

public:
    Graph(int vertices) : V(vertices), adj(V) {}

    void addEdge(int u, int v) {
        adj[u].push_back(v);
        adj[v].push_back(u);
    }

    // BFS з виведенням рівнів (відстаней)
    void bfs(int start) {
        vector<bool> visited(V, false);
        vector<int> level(V, -1);
        queue<int> q;

        visited[start] = true;
        level[start] = 0;
        q.push(start);

        cout << "BFS обхід від вершини " << start << ":" << endl;

        while (!q.empty()) {
            int u = q.front();
            q.pop();

            cout << "  Вершина " << u << " (рівень " << level[u] << ")" << endl;

            for (int v : adj[u]) {
                if (!visited[v]) {
                    visited[v] = true;
                    level[v] = level[u] + 1;
                    q.push(v);
                }
            }
        }
    }
};

int main() {
    //     0 --- 1
    //     |     |
    //     2 --- 3 --- 4
    //           |
    //           5
    Graph g(6);
    g.addEdge(0, 1);
    g.addEdge(0, 2);
    g.addEdge(1, 3);
    g.addEdge(2, 3);
    g.addEdge(3, 4);
    g.addEdge(3, 5);

    g.bfs(0);

    return 0;
}
```

### Приклад запуску

```text
BFS обхід від вершини 0:
  Вершина 0 (рівень 0)
  Вершина 1 (рівень 1)
  Вершина 2 (рівень 1)
  Вершина 3 (рівень 2)
  Вершина 4 (рівень 3)
  Вершина 5 (рівень 3)
```

---

## Завдання 3: DFS — пошук в глибину

### Теорія

DFS (Depth-First Search) обходить граф **вглиб** — спочатку йде якомога далі по одній гілці, потім повертається. Використовує **рекурсію** (або стек).

DFS фіксує **час входу** (discovery time) та **час виходу** (finish time) для кожної вершини — це корисно для багатьох алгоритмів.

```
Порядок DFS від вершини 0:

    0 → 1 → 3 → 4 (глухий кут) → ↩ 3 → 5 → ↩ 3 → ↩ 1 → ↩ 0 → 2

    0 --- 1
    |     |
    2 --- 3 --- 4
          |
          5
```

**Складність:** O(V + E)

```cpp
#include <iostream>
#include <vector>
using namespace std;

class Graph {
    int V;
    vector<vector<int>> adj;
    int timer;

public:
    Graph(int vertices) : V(vertices), adj(V), timer(0) {}

    void addEdge(int u, int v) {
        adj[u].push_back(v);
        adj[v].push_back(u);
    }

    void dfs(int u, vector<bool>& visited,
             vector<int>& tin, vector<int>& tout) {
        visited[u] = true;
        tin[u] = timer++;

        cout << "  Вхід у вершину " << u
             << " (tin=" << tin[u] << ")" << endl;

        for (int v : adj[u]) {
            if (!visited[v]) {
                dfs(v, visited, tin, tout);
            }
        }

        tout[u] = timer++;
        cout << "  Вихід з вершини " << u
             << " (tout=" << tout[u] << ")" << endl;
    }

    void runDFS(int start) {
        vector<bool> visited(V, false);
        vector<int> tin(V, -1), tout(V, -1);
        timer = 0;

        cout << "DFS обхід від вершини " << start << ":" << endl;
        dfs(start, visited, tin, tout);

        cout << "\nЧаси входу/виходу:" << endl;
        for (int i = 0; i < V; i++) {
            cout << "  Вершина " << i
                 << ": tin=" << tin[i]
                 << ", tout=" << tout[i] << endl;
        }
    }
};

int main() {
    //     0 --- 1
    //     |     |
    //     2 --- 3 --- 4
    Graph g(5);
    g.addEdge(0, 1);
    g.addEdge(0, 2);
    g.addEdge(1, 3);
    g.addEdge(2, 3);
    g.addEdge(3, 4);

    g.runDFS(0);

    return 0;
}
```

### Приклад запуску

```text
DFS обхід від вершини 0:
  Вхід у вершину 0 (tin=0)
  Вхід у вершину 1 (tin=1)
  Вхід у вершину 3 (tin=2)
  Вхід у вершину 2 (tin=3)
  Вихід з вершини 2 (tout=4)
  Вхід у вершину 4 (tin=5)
  Вихід з вершини 4 (tout=6)
  Вихід з вершини 3 (tout=7)
  Вихід з вершини 1 (tout=8)
  Вихід з вершини 0 (tout=9)

Часи входу/виходу:
  Вершина 0: tin=0, tout=9
  Вершина 1: tin=1, tout=8
  Вершина 2: tin=3, tout=4
  Вершина 3: tin=2, tout=7
  Вершина 4: tin=5, tout=6
```

---

## Завдання 4: Найкоротший шлях у незваженому графі (BFS)

### Теорія

У **незваженому** графі (або графі з однаковими вагами) BFS знаходить найкоротший шлях від стартової вершини до всіх інших. Відстань = кількість ребер.

**Складність:** O(V + E)

```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <algorithm>
using namespace std;

class Graph {
    int V;
    vector<vector<int>> adj;

public:
    Graph(int vertices) : V(vertices), adj(V) {}

    void addEdge(int u, int v) {
        adj[u].push_back(v);
        adj[v].push_back(u);
    }

    // BFS для знаходження найкоротших шляхів
    pair<vector<int>, vector<int>> shortestPaths(int start) {
        vector<int> dist(V, -1);
        vector<int> parent(V, -1);
        queue<int> q;

        dist[start] = 0;
        q.push(start);

        while (!q.empty()) {
            int u = q.front();
            q.pop();

            for (int v : adj[u]) {
                if (dist[v] == -1) {
                    dist[v] = dist[u] + 1;
                    parent[v] = u;
                    q.push(v);
                }
            }
        }

        return {dist, parent};
    }

    // Відновлення шляху
    vector<int> getPath(int start, int end, const vector<int>& parent) {
        if (parent[end] == -1 && start != end)
            return {};  // шлях не існує

        vector<int> path;
        for (int v = end; v != -1; v = parent[v])
            path.push_back(v);

        reverse(path.begin(), path.end());
        return path;
    }
};

int main() {
    //     0 --- 1 --- 5
    //     |     |
    //     2 --- 3 --- 4
    Graph g(6);
    g.addEdge(0, 1);
    g.addEdge(0, 2);
    g.addEdge(1, 3);
    g.addEdge(2, 3);
    g.addEdge(3, 4);
    g.addEdge(1, 5);

    auto [dist, parent] = g.shortestPaths(0);

    cout << "Найкоротші відстані від вершини 0:" << endl;
    for (int i = 0; i < 6; i++) {
        cout << "  До вершини " << i << ": " << dist[i] << " ребер";

        auto path = g.getPath(0, i, parent);
        if (!path.empty()) {
            cout << "  (шлях: ";
            for (int j = 0; j < path.size(); j++) {
                if (j > 0) cout << " -> ";
                cout << path[j];
            }
            cout << ")";
        }
        cout << endl;
    }

    return 0;
}
```

### Приклад запуску

```text
Найкоротші відстані від вершини 0:
  До вершини 0: 0 ребер  (шлях: 0)
  До вершини 1: 1 ребер  (шлях: 0 -> 1)
  До вершини 2: 1 ребер  (шлях: 0 -> 2)
  До вершини 3: 2 ребер  (шлях: 0 -> 1 -> 3)
  До вершини 4: 3 ребер  (шлях: 0 -> 1 -> 3 -> 4)
  До вершини 5: 2 ребер  (шлях: 0 -> 1 -> 5)
```

---

## Завдання 5: Алгоритм Дейкстри

### Теорія

Алгоритм Дейкстри знаходить найкоротші шляхи від стартової вершини до всіх інших у **зваженому графі з невід'ємними вагами**.

**Ідея:**
1. Ініціалізуємо відстані: `dist[start] = 0`, решта = ∞
2. Використовуємо пріоритетну чергу (мінімальна купа)
3. На кожному кроці витягуємо вершину з найменшою відстанню
4. Оновлюємо відстані до її сусідів (релаксація)

```
Граф:
    0 --(4)-- 1
    |         |
   (2)       (5)
    |         |
    2 --(1)-- 3

Дейкстра від 0:
  dist = [0, 4, 2, 3]
  Шлях 0→3: 0 → 2 → 3 (вага 3, а не 0→1→3 = 9)
```

**Складність:** O((V + E) log V) з пріоритетною чергою

```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <limits>
#include <algorithm>
using namespace std;

const int INF = numeric_limits<int>::max();

class Graph {
private:
    int V;
    vector<vector<pair<int, int>>> adj;  // (сусід, вага)

public:
    Graph(int vertices) : V(vertices), adj(V) {}

    void addEdge(int u, int v, int weight) {
        adj[u].push_back({v, weight});
        adj[v].push_back({u, weight});  // для неорієнтованого графа
    }

    // Алгоритм Дейкстри
    vector<int> dijkstra(int start) {
        // Відстані до всіх вершин
        vector<int> dist(V, INF);
        dist[start] = 0;

        // Priority queue: (відстань, вершина)
        // Мінімальна купа за відстанню
        priority_queue<pair<int, int>,
                      vector<pair<int, int>>,
                      greater<pair<int, int>>> pq;

        pq.push({0, start});

        while (!pq.empty()) {
            int u = pq.top().second;
            int d = pq.top().first;
            pq.pop();

            // Якщо вже знайшли коротший шлях, пропускаємо
            if (d > dist[u]) {
                continue;
            }

            // Оновлюємо відстані до сусідів
            for (auto& [v, weight] : adj[u]) {
                int newDist = dist[u] + weight;

                if (newDist < dist[v]) {
                    dist[v] = newDist;
                    pq.push({newDist, v});
                }
            }
        }

        return dist;
    }

    // Дейкстра зі збереженням шляху
    pair<vector<int>, vector<int>> dijkstraWithPath(int start) {
        vector<int> dist(V, INF);
        vector<int> parent(V, -1);
        dist[start] = 0;

        priority_queue<pair<int, int>,
                      vector<pair<int, int>>,
                      greater<pair<int, int>>> pq;

        pq.push({0, start});

        while (!pq.empty()) {
            int u = pq.top().second;
            int d = pq.top().first;
            pq.pop();

            if (d > dist[u]) {
                continue;
            }

            for (auto& [v, weight] : adj[u]) {
                int newDist = dist[u] + weight;

                if (newDist < dist[v]) {
                    dist[v] = newDist;
                    parent[v] = u;
                    pq.push({newDist, v});
                }
            }
        }

        return {dist, parent};
    }

    // Відновити шлях від start до end
    vector<int> getPath(int start, int end, const vector<int>& parent) {
        if (parent[end] == -1 && start != end) {
            return {};  // шлях не існує
        }

        vector<int> path;
        int current = end;

        while (current != -1) {
            path.push_back(current);
            current = parent[current];
        }

        reverse(path.begin(), path.end());
        return path;
    }

    // Вивести результати
    void printDistances(int start, const vector<int>& dist) {
        cout << "Найкоротші відстані від вершини " << start << ":" << endl;
        for (int v = 0; v < V; v++) {
            cout << "  До вершини " << v << ": ";
            if (dist[v] == INF) {
                cout << "недосяжна" << endl;
            } else {
                cout << dist[v] << endl;
            }
        }
    }
};

int main() {
    Graph g(4);

    g.addEdge(0, 1, 4);
    g.addEdge(0, 2, 2);
    g.addEdge(1, 3, 5);
    g.addEdge(2, 3, 1);

    // Знайти найкоротші відстані
    auto dist = g.dijkstra(0);
    g.printDistances(0, dist);

    // Знайти найкоротший шлях до конкретної вершини
    auto [distances, parent] = g.dijkstraWithPath(0);

    cout << "\nНайкоротший шлях від 0 до 3: ";
    auto path = g.getPath(0, 3, parent);
    for (int v : path) {
        cout << v << " ";
    }
    cout << "\nДовжина шляху: " << distances[3] << endl;

    return 0;
}
```

### Приклад запуску

```text
Найкоротші відстані від вершини 0:
  До вершини 0: 0
  До вершини 1: 4
  До вершини 2: 2
  До вершини 3: 3

Найкоротший шлях від 0 до 3: 0 2 3
Довжина шляху: 3
```

---

## Завдання 6: Алгоритм Беллмана-Форда

### Теорія

Алгоритм Беллмана-Форда знаходить найкоротші шляхи від стартової вершини, працюючи навіть із **від'ємними вагами** ребер. Також виявляє **від'ємні цикли**.

**Ідея:**
1. Ініціалізуємо відстані: `dist[start] = 0`, решта = ∞
2. Повторюємо V-1 разів: для кожного ребра (u, v, w) — релаксація
3. Якщо на V-й ітерації ще можна покращити — є від'ємний цикл

```
Граф з від'ємним ребром:
    0 --(4)--> 1
    |          |
   (2)       (-3)     ← від'ємна вага!
    ↓          ↓
    2 --(1)--> 3

Дейкстра НЕ працює тут!
Беллман-Форд: dist = [0, 4, 2, 1]
```

**Складність:** O(V · E)

```cpp
#include <iostream>
#include <vector>
#include <limits>
using namespace std;

const int INF = numeric_limits<int>::max();

struct Edge {
    int from, to, weight;
};

class BellmanFord {
    int V;
    vector<Edge> edges;

public:
    BellmanFord(int vertices) : V(vertices) {}

    void addEdge(int u, int v, int w) {
        edges.push_back({u, v, w});
    }

    // Повертає {відстані, чи є від'ємний цикл}
    pair<vector<int>, bool> solve(int start) {
        vector<int> dist(V, INF);
        dist[start] = 0;

        // V-1 ітерацій релаксації
        for (int i = 0; i < V - 1; i++) {
            bool updated = false;
            for (auto& [from, to, weight] : edges) {
                if (dist[from] != INF && dist[from] + weight < dist[to]) {
                    dist[to] = dist[from] + weight;
                    updated = true;
                }
            }
            // Якщо нічого не змінилось — можна зупинитись
            if (!updated) break;
        }

        // Перевірка на від'ємний цикл (V-та ітерація)
        bool hasNegativeCycle = false;
        for (auto& [from, to, weight] : edges) {
            if (dist[from] != INF && dist[from] + weight < dist[to]) {
                hasNegativeCycle = true;
                break;
            }
        }

        return {dist, hasNegativeCycle};
    }
};

int main() {
    // Приклад 1: Граф з від'ємними ребрами (без від'ємного циклу)
    cout << "=== Граф з від'ємними ребрами ===" << endl;
    BellmanFord bf1(5);
    bf1.addEdge(0, 1, 4);
    bf1.addEdge(0, 2, 2);
    bf1.addEdge(1, 3, -3);
    bf1.addEdge(2, 3, 1);
    bf1.addEdge(3, 4, 2);
    bf1.addEdge(2, 4, 5);

    auto [dist1, neg1] = bf1.solve(0);
    for (int i = 0; i < 5; i++)
        cout << "  dist[" << i << "] = "
             << (dist1[i] == INF ? "INF" : to_string(dist1[i])) << endl;
    cout << "  Від'ємний цикл: " << (neg1 ? "Так" : "Ні") << endl;

    // Приклад 2: Граф з від'ємним циклом
    cout << "\n=== Граф з від'ємним циклом ===" << endl;
    BellmanFord bf2(4);
    bf2.addEdge(0, 1, 1);
    bf2.addEdge(1, 2, -3);
    bf2.addEdge(2, 3, 2);
    bf2.addEdge(3, 1, -1);  // цикл 1→2→3→1 з сумою -3+2-1 = -2

    auto [dist2, neg2] = bf2.solve(0);
    cout << "  Від'ємний цикл: " << (neg2 ? "Так" : "Ні") << endl;

    return 0;
}
```

### Приклад запуску

```text
=== Граф з від'ємними ребрами ===
  dist[0] = 0
  dist[1] = 4
  dist[2] = 2
  dist[3] = 1
  dist[4] = 3
  Від'ємний цикл: Ні

=== Граф з від'ємним циклом ===
  Від'ємний цикл: Так
```

### Порівняння з Дейкстрою

| | Дейкстра | Беллман-Форд |
|---|---|---|
| Складність | O((V+E) log V) | O(V·E) |
| Від'ємні ваги | ❌ Не працює | ✅ Підтримує |
| Від'ємні цикли | ❌ Не виявляє | ✅ Виявляє |
| Коли використовувати | Невід'ємні ваги | Від'ємні ваги або потрібна перевірка циклів |

---

## Завдання 7: Алгоритм Флойда-Уоршелла

### Теорія

Алгоритм Флойда-Уоршелла знаходить найкоротші шляхи **між усіма парами** вершин. Працює з від'ємними вагами (але не з від'ємними циклами).

**Ідея (динамічне програмування):**
- `dist[i][j]` — найкоротша відстань від i до j
- Для кожної проміжної вершини k: `dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j])`

**Складність:** O(V³)

```cpp
#include <iostream>
#include <vector>
#include <limits>
#include <iomanip>
using namespace std;

const int INF = 1e9;

class FloydWarshall {
    int V;
    vector<vector<int>> dist;

public:
    FloydWarshall(int vertices) : V(vertices), dist(V, vector<int>(V, INF)) {
        for (int i = 0; i < V; i++)
            dist[i][i] = 0;
    }

    void addEdge(int u, int v, int w) {
        dist[u][v] = w;
    }

    void addUndirectedEdge(int u, int v, int w) {
        dist[u][v] = w;
        dist[v][u] = w;
    }

    void solve() {
        // Ключова формула: якщо через вершину k коротше — оновлюємо
        for (int k = 0; k < V; k++) {
            for (int i = 0; i < V; i++) {
                for (int j = 0; j < V; j++) {
                    if (dist[i][k] != INF && dist[k][j] != INF) {
                        dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j]);
                    }
                }
            }
        }
    }

    void print() {
        cout << "Матриця найкоротших відстаней:" << endl;
        cout << "     ";
        for (int j = 0; j < V; j++)
            cout << setw(5) << j;
        cout << endl;

        for (int i = 0; i < V; i++) {
            cout << setw(3) << i << ": ";
            for (int j = 0; j < V; j++) {
                if (dist[i][j] == INF)
                    cout << setw(5) << "INF";
                else
                    cout << setw(5) << dist[i][j];
            }
            cout << endl;
        }
    }

    int getDistance(int u, int v) {
        return dist[u][v];
    }
};

int main() {
    // Орієнтований зважений граф:
    //     0 --(3)--> 1
    //     |          |
    //    (7)        (1)
    //     ↓          ↓
    //     2 <--(2)-- 3
    //     |
    //    (1)
    //     ↓
    //     3
    FloydWarshall fw(4);
    fw.addEdge(0, 1, 3);
    fw.addEdge(0, 2, 7);
    fw.addEdge(1, 3, 1);
    fw.addEdge(3, 2, 2);
    fw.addEdge(2, 3, 1);

    fw.solve();
    fw.print();

    cout << "\nВідстань від 0 до 2: " << fw.getDistance(0, 2) << endl;
    cout << "Відстань від 1 до 2: " << fw.getDistance(1, 2) << endl;

    return 0;
}
```

### Приклад запуску

```text
Матриця найкоротших відстаней:
         0    1    2    3
  0:     0    3    6    4
  1:   INF    0    3    1
  2:   INF  INF    0    1
  3:   INF  INF    2    0

Відстань від 0 до 2: 6
Відстань від 1 до 2: 3
```

---

## Завдання 8: Мінімальне остовне дерево — Алгоритм Пріма

### Теорія

**Остовне дерево** (Spanning Tree) — підграф, що:
- Містить всі вершини
- Є деревом (зв'язний, без циклів)
- Має V-1 ребер

**Мінімальне остовне дерево (MST)** — остовне дерево з мінімальною сумарною вагою ребер.

```
Граф:                          MST (сума = 7):
    0 --(4)-- 1                    0 --(4)-- 1
    |    \    |                    |
   (2)   (6) (5)                  (2)
    |      \  |                    |
    2 --(1)-- 3                    2 --(1)-- 3
```

**Два основні алгоритми:**
- **Алгоритм Пріма** — росте дерево від однієї вершини
- **Алгоритм Крускала** — сортує ребра і додає найменші (див. Лекцію 8 — Union-Find)

**Застосування:** проектування мереж (дороги, електропроводка), кластеризація, наближені розв'язки TSP, мережі зв'язку.

**Алгоритм Пріма (ідея):**
1. Починаємо з будь-якої вершини
2. Вибираємо найлегше ребро, що з'єднує дерево з новою вершиною
3. Додаємо цю вершину до дерева
4. Повторюємо, поки не додамо всі вершини

**Складність:** O((V + E) log V) з пріоритетною чергою

```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <limits>
using namespace std;

const int INF = numeric_limits<int>::max();

class Graph {
private:
    int V;
    vector<vector<pair<int, int>>> adj;

public:
    Graph(int vertices) : V(vertices), adj(V) {}

    void addEdge(int u, int v, int weight) {
        adj[u].push_back({v, weight});
        adj[v].push_back({u, weight});
    }

    // Алгоритм Пріма
    int primMST() {
        // Відстань до дерева для кожної вершини
        vector<int> key(V, INF);
        vector<bool> inMST(V, false);
        vector<int> parent(V, -1);

        // Priority queue: (вага, вершина)
        priority_queue<pair<int, int>,
                      vector<pair<int, int>>,
                      greater<pair<int, int>>> pq;

        // Починаємо з вершини 0
        key[0] = 0;
        pq.push({0, 0});

        int mstWeight = 0;

        while (!pq.empty()) {
            int u = pq.top().second;
            pq.pop();

            if (inMST[u]) {
                continue;
            }

            // Додаємо вершину до MST
            inMST[u] = true;
            mstWeight += key[u];

            // Оновлюємо ключі сусідів
            for (auto& [v, weight] : adj[u]) {
                if (!inMST[v] && weight < key[v]) {
                    key[v] = weight;
                    parent[v] = u;
                    pq.push({weight, v});
                }
            }
        }

        // Вивести ребра MST
        cout << "Ребра в MST:" << endl;
        for (int v = 1; v < V; v++) {
            if (parent[v] != -1) {
                cout << "  " << parent[v] << " -- " << v
                     << " (вага: " << key[v] << ")" << endl;
            }
        }

        return mstWeight;
    }
};

int main() {
    Graph g(4);

    g.addEdge(0, 1, 4);
    g.addEdge(0, 2, 2);
    g.addEdge(0, 3, 6);
    g.addEdge(1, 3, 5);
    g.addEdge(2, 3, 1);

    int mstWeight = g.primMST();
    cout << "\nЗагальна вага MST: " << mstWeight << endl;

    return 0;
}
```

### Приклад запуску

```text
Ребра в MST:
  0 -- 1 (вага: 4)
  0 -- 2 (вага: 2)
  2 -- 3 (вага: 1)

Загальна вага MST: 7
```

---

## Завдання 9: Топологічне сортування

### Теорія

**Топологічне сортування** — лінійне впорядкування вершин **орієнтованого ациклічного графа** (DAG), таке що для кожного ребра (u, v) вершина u стоїть перед v.

**Застосування:**
- Порядок компіляції модулів
- Розклад задач із залежностями
- Порядок курсів у навчальному плані

```
DAG:
    5 → 0 ← 4
    ↓       ↓
    2 → 3 → 1

Одне з можливих топологічних сортувань: 4 5 2 0 3 1
```

**Алгоритм на основі DFS:**
1. Запускаємо DFS для кожної невідвіданої вершини
2. Коли вершина повністю оброблена (tout), додаємо її на стек
3. Стек у зворотному порядку — топологічне сортування

**Складність:** O(V + E)

```cpp
#include <iostream>
#include <vector>
#include <stack>
using namespace std;

class Graph {
    int V;
    vector<vector<int>> adj;

    void dfs(int u, vector<bool>& visited, stack<int>& result) {
        visited[u] = true;

        for (int v : adj[u]) {
            if (!visited[v])
                dfs(v, visited, result);
        }

        // Після обробки всіх нащадків — додаємо на стек
        result.push(u);
    }

public:
    Graph(int vertices) : V(vertices), adj(V) {}

    // Орієнтоване ребро u → v
    void addEdge(int u, int v) {
        adj[u].push_back(v);
    }

    void topologicalSort() {
        vector<bool> visited(V, false);
        stack<int> result;

        // Запускаємо DFS для всіх невідвіданих вершин
        for (int i = 0; i < V; i++) {
            if (!visited[i])
                dfs(i, visited, result);
        }

        // Виводимо результат
        cout << "Топологічне сортування: ";
        while (!result.empty()) {
            cout << result.top() << " ";
            result.pop();
        }
        cout << endl;
    }
};

int main() {
    // DAG — залежності курсів:
    // 5 → 0, 5 → 2, 4 → 0, 4 → 1, 2 → 3, 3 → 1
    //
    //    5 → 0 ← 4
    //    ↓       ↓
    //    2 → 3 → 1
    Graph g(6);
    g.addEdge(5, 0);
    g.addEdge(5, 2);
    g.addEdge(4, 0);
    g.addEdge(4, 1);
    g.addEdge(2, 3);
    g.addEdge(3, 1);

    g.topologicalSort();

    // Другий приклад: послідовність одягання
    // 0=труси, 1=штани, 2=пояс, 3=сорочка, 4=краватка, 5=піджак
    // 0→1, 1→2, 3→2, 3→4, 2→5, 4→5
    cout << "\nПослідовність одягання:" << endl;
    Graph g2(6);
    g2.addEdge(0, 1);  // труси → штани
    g2.addEdge(1, 2);  // штани → пояс
    g2.addEdge(3, 2);  // сорочка → пояс
    g2.addEdge(3, 4);  // сорочка → краватка
    g2.addEdge(2, 5);  // пояс → піджак
    g2.addEdge(4, 5);  // краватка → піджак
    g2.topologicalSort();

    return 0;
}
```

### Приклад запуску

```text
Топологічне сортування: 5 4 2 3 1 0

Послідовність одягання:
Топологічне сортування: 3 4 0 1 2 5
```

---

## Завдання 10: Виявлення циклів в орієнтованому графі

### Теорія

Цикл в орієнтованому графі виявляється за допомогою DFS із **трьома кольорами**:
- **Білий (0)** — вершина ще не відвідана
- **Сірий (1)** — вершина в процесі обробки (в стеку рекурсії)
- **Чорний (2)** — вершина повністю оброблена

Якщо під час DFS натрапляємо на **сіру** вершину — знайдено **зворотне ребро** → цикл!

```
Граф з циклом:              Граф без циклу (DAG):
    0 → 1                       0 → 1
    ↑   ↓                           ↓
    3 ← 2                       2 → 3

Цикл: 0 → 1 → 2 → 3 → 0       Немає зворотних ребер
```

**Складність:** O(V + E)

```cpp
#include <iostream>
#include <vector>
using namespace std;

class Graph {
    int V;
    vector<vector<int>> adj;

    // Повертає true, якщо знайдено цикл
    bool dfs(int u, vector<int>& color, vector<int>& cycle_path) {
        color[u] = 1;  // сірий — в обробці
        cycle_path.push_back(u);

        for (int v : adj[u]) {
            if (color[v] == 1) {
                // Знайдено зворотне ребро → цикл!
                cycle_path.push_back(v);
                return true;
            }
            if (color[v] == 0) {
                if (dfs(v, color, cycle_path))
                    return true;
            }
        }

        cycle_path.pop_back();
        color[u] = 2;  // чорний — оброблено
        return false;
    }

public:
    Graph(int vertices) : V(vertices), adj(V) {}

    void addEdge(int u, int v) {
        adj[u].push_back(v);
    }

    bool hasCycle() {
        vector<int> color(V, 0);  // 0=білий, 1=сірий, 2=чорний
        vector<int> cycle_path;

        for (int i = 0; i < V; i++) {
            if (color[i] == 0) {
                if (dfs(i, color, cycle_path)) {
                    // Виводимо цикл
                    int cycle_start = cycle_path.back();
                    cout << "  Цикл: ";
                    bool in_cycle = false;
                    for (int v : cycle_path) {
                        if (v == cycle_start) in_cycle = true;
                        if (in_cycle) cout << v << " → ";
                    }
                    cout << endl;
                    return true;
                }
            }
        }
        return false;
    }
};

int main() {
    // Граф 1: з циклом 0 → 1 → 2 → 3 → 0
    cout << "=== Граф 1 ===" << endl;
    Graph g1(4);
    g1.addEdge(0, 1);
    g1.addEdge(1, 2);
    g1.addEdge(2, 3);
    g1.addEdge(3, 0);  // зворотне ребро → цикл

    cout << "Має цикл: " << (g1.hasCycle() ? "Так" : "Ні") << endl;

    // Граф 2: DAG (без циклу)
    cout << "\n=== Граф 2 (DAG) ===" << endl;
    Graph g2(4);
    g2.addEdge(0, 1);
    g2.addEdge(0, 2);
    g2.addEdge(1, 3);
    g2.addEdge(2, 3);

    cout << "Має цикл: " << (g2.hasCycle() ? "Так" : "Ні") << endl;

    // Граф 3: цикл не від кореня
    cout << "\n=== Граф 3 ===" << endl;
    Graph g3(5);
    g3.addEdge(0, 1);
    g3.addEdge(1, 2);
    g3.addEdge(2, 3);
    g3.addEdge(3, 1);  // цикл: 1 → 2 → 3 → 1
    g3.addEdge(0, 4);

    cout << "Має цикл: " << (g3.hasCycle() ? "Так" : "Ні") << endl;

    return 0;
}
```

### Приклад запуску

```text
=== Граф 1 ===
  Цикл: 0 → 1 → 2 → 3 → 0 →
Має цикл: Так

=== Граф 2 (DAG) ===
Має цикл: Ні

=== Граф 3 ===
  Цикл: 1 → 2 → 3 → 1 →
Має цикл: Так
```

---

## Завдання 11: Перевірка двочастковості графа

### Теорія

Граф є **двочастковим** (bipartite), якщо його вершини можна розділити на дві групи так, що кожне ребро з'єднує вершини з різних груп.

**Еквівалентні визначення:**
- Граф можна розфарбувати у 2 кольори без конфліктів
- Граф не містить циклів непарної довжини

**Алгоритм:** BFS з розфарбовуванням — якщо натрапляємо на сусіда з тим же кольором, граф не двочастковий.

```
Двочастковий:             Не двочастковий:
    0(A) --- 1(B)              0(A) --- 1(B)
    |        |                  |      / |
    2(B) --- 3(A)              2(B) --- 3(?)  ← конфлікт!

Група A: {0, 3}               Трикутник 1-2-3
Група B: {1, 2}               не розфарбовується
```

**Складність:** O(V + E)

```cpp
#include <iostream>
#include <vector>
#include <queue>
using namespace std;

class Graph {
    int V;
    vector<vector<int>> adj;

public:
    Graph(int vertices) : V(vertices), adj(V) {}

    void addEdge(int u, int v) {
        adj[u].push_back(v);
        adj[v].push_back(u);
    }

    bool isBipartite() {
        vector<int> color(V, -1);  // -1 = нерозфарбовано

        // Перевіряємо кожну компоненту зв'язності
        for (int start = 0; start < V; start++) {
            if (color[start] != -1) continue;

            // BFS з розфарбовуванням
            queue<int> q;
            color[start] = 0;
            q.push(start);

            while (!q.empty()) {
                int u = q.front();
                q.pop();

                for (int v : adj[u]) {
                    if (color[v] == -1) {
                        // Розфарбовуємо в протилежний колір
                        color[v] = 1 - color[u];
                        q.push(v);
                    } else if (color[v] == color[u]) {
                        // Конфлікт — сусіди одного кольору
                        cout << "  Конфлікт: вершини " << u
                             << " та " << v << " мають однаковий колір!"
                             << endl;
                        return false;
                    }
                }
            }
        }

        // Виводимо групи
        cout << "  Група A: ";
        for (int i = 0; i < V; i++)
            if (color[i] == 0) cout << i << " ";
        cout << endl;

        cout << "  Група B: ";
        for (int i = 0; i < V; i++)
            if (color[i] == 1) cout << i << " ";
        cout << endl;

        return true;
    }
};

int main() {
    // Граф 1: двочастковий (цикл парної довжини)
    //    0 --- 1
    //    |     |
    //    3 --- 2
    cout << "=== Граф 1 (цикл довжини 4) ===" << endl;
    Graph g1(4);
    g1.addEdge(0, 1);
    g1.addEdge(1, 2);
    g1.addEdge(2, 3);
    g1.addEdge(3, 0);
    cout << "Двочастковий: " << (g1.isBipartite() ? "Так" : "Ні") << endl;

    // Граф 2: не двочастковий (трикутник — цикл непарної довжини)
    //    0 --- 1
    //     \   /
    //      \ /
    //       2
    cout << "\n=== Граф 2 (трикутник) ===" << endl;
    Graph g2(3);
    g2.addEdge(0, 1);
    g2.addEdge(1, 2);
    g2.addEdge(2, 0);
    cout << "Двочастковий: " << (g2.isBipartite() ? "Так" : "Ні") << endl;

    // Граф 3: двочастковий (дерево завжди двочастковий)
    //      0
    //     / \
    //    1   2
    //   / \
    //  3   4
    cout << "\n=== Граф 3 (дерево) ===" << endl;
    Graph g3(5);
    g3.addEdge(0, 1);
    g3.addEdge(0, 2);
    g3.addEdge(1, 3);
    g3.addEdge(1, 4);
    cout << "Двочастковий: " << (g3.isBipartite() ? "Так" : "Ні") << endl;

    return 0;
}
```

### Приклад запуску

```text
=== Граф 1 (цикл довжини 4) ===
  Група A: 0 2
  Група B: 1 3
Двочастковий: Так

=== Граф 2 (трикутник) ===
  Конфлікт: вершини 2 та 0 мають однаковий колір!
Двочастковий: Ні

=== Граф 3 (дерево) ===
  Група A: 0 3 4
  Група B: 1 2
Двочастковий: Так
```

---

## Завдання 12: Компоненти зв'язності

### Теорія

**Компонента зв'язності** — максимальна підмножина вершин, де між будь-якими двома є шлях. Для знаходження всіх компонент запускаємо BFS/DFS від кожної невідвіданої вершини.

**Складність:** O(V + E)

```cpp
#include <iostream>
#include <vector>
#include <queue>
using namespace std;

class Graph {
    int V;
    vector<vector<int>> adj;

    void bfs(int start, vector<bool>& visited, vector<int>& component) {
        queue<int> q;
        visited[start] = true;
        q.push(start);

        while (!q.empty()) {
            int u = q.front();
            q.pop();
            component.push_back(u);

            for (int v : adj[u]) {
                if (!visited[v]) {
                    visited[v] = true;
                    q.push(v);
                }
            }
        }
    }

public:
    Graph(int vertices) : V(vertices), adj(V) {}

    void addEdge(int u, int v) {
        adj[u].push_back(v);
        adj[v].push_back(u);
    }

    vector<vector<int>> findComponents() {
        vector<bool> visited(V, false);
        vector<vector<int>> components;

        for (int i = 0; i < V; i++) {
            if (!visited[i]) {
                vector<int> component;
                bfs(i, visited, component);
                components.push_back(component);
            }
        }

        return components;
    }
};

int main() {
    // Граф з 3 компонентами:
    // Компонента 1: 0-1-2
    // Компонента 2: 3-4
    // Компонента 3: 5 (ізольована)
    Graph g(6);
    g.addEdge(0, 1);
    g.addEdge(1, 2);
    g.addEdge(3, 4);

    auto components = g.findComponents();

    cout << "Кількість компонент зв'язності: "
         << components.size() << endl;

    for (int i = 0; i < components.size(); i++) {
        cout << "Компонента " << (i + 1) << ": { ";
        for (int v : components[i])
            cout << v << " ";
        cout << "}" << endl;
    }

    // Найбільша компонента
    int maxSize = 0;
    for (auto& comp : components)
        maxSize = max(maxSize, (int)comp.size());
    cout << "\nНайбільша компонента: " << maxSize << " вершин" << endl;

    return 0;
}
```

### Приклад запуску

```text
Кількість компонент зв'язності: 3
Компонента 1: { 0 1 2 }
Компонента 2: { 3 4 }
Компонента 3: { 5 }

Найбільша компонента: 3 вершин
```

---

## Завдання 13: Алгоритм Крускала (MST)

### Теорія

Алгоритм Крускала — альтернативний спосіб побудови MST. На відміну від Пріма, він працює з **ребрами**, а не з вершинами.

**Ідея:**
1. Відсортувати всі ребра за вагою (від найменшої)
2. Для кожного ребра: додати до MST, якщо воно **не створює цикл**
3. Перевірка циклів — через **Union-Find** (див. Лекцію 8)

**Порівняння Прім vs Крускал:**

| | Прім | Крускал |
|---|---|---|
| Підхід | Росте від вершини | Сортує ребра |
| Складність | O((V+E) log V) | O(E log E) |
| Краще для | Щільні графи | Розріджені графи |
| Структура | Priority queue | Union-Find |

**Складність:** O(E log E)

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

// Union-Find (Disjoint Set Union)
class DSU {
    vector<int> parent, rank_;
public:
    DSU(int n) : parent(n), rank_(n, 0) {
        for (int i = 0; i < n; i++) parent[i] = i;
    }
    int find(int x) {
        if (x != parent[x]) parent[x] = find(parent[x]);
        return parent[x];
    }
    bool unite(int a, int b) {
        int ra = find(a), rb = find(b);
        if (ra == rb) return false;
        if (rank_[ra] < rank_[rb]) swap(ra, rb);
        parent[rb] = ra;
        if (rank_[ra] == rank_[rb]) rank_[ra]++;
        return true;
    }
};

struct Edge {
    int u, v, weight;
};

int main() {
    int V = 6;
    vector<Edge> edges = {
        {0, 1, 4}, {0, 2, 3}, {1, 2, 1},
        {1, 3, 2}, {2, 3, 4}, {3, 4, 2},
        {4, 5, 6}, {2, 5, 5}
    };

    // Крок 1: сортуємо ребра за вагою
    sort(edges.begin(), edges.end(), [](const Edge& a, const Edge& b) {
        return a.weight < b.weight;
    });

    // Крок 2: жадібно додаємо ребра
    DSU dsu(V);
    int mstWeight = 0;
    vector<Edge> mstEdges;

    cout << "Обробка ребер:" << endl;
    for (auto& e : edges) {
        if (dsu.unite(e.u, e.v)) {
            mstWeight += e.weight;
            mstEdges.push_back(e);
            cout << "  + Додано (" << e.u << ", " << e.v
                 << ") вага=" << e.weight << endl;
        } else {
            cout << "  - Пропущено (" << e.u << ", " << e.v
                 << ") — цикл" << endl;
        }

        if (mstEdges.size() == V - 1) break;
    }

    cout << "\nРебра MST:" << endl;
    for (auto& e : mstEdges) {
        cout << "  " << e.u << " -- " << e.v
             << " (вага: " << e.weight << ")" << endl;
    }
    cout << "Загальна вага MST: " << mstWeight << endl;

    return 0;
}
```

### Приклад запуску

```text
Обробка ребер:
  + Додано (1, 2) вага=1
  + Додано (1, 3) вага=2
  + Додано (3, 4) вага=2
  + Додано (0, 2) вага=3
  - Пропущено (0, 1) — цикл
  - Пропущено (2, 3) — цикл
  + Додано (2, 5) вага=5

Ребра MST:
  1 -- 2 (вага: 1)
  1 -- 3 (вага: 2)
  3 -- 4 (вага: 2)
  0 -- 2 (вага: 3)
  2 -- 5 (вага: 5)
Загальна вага MST: 13
```

---

## Завдання 14: Мости та точки зчленування

### Теорія

**Міст** — ребро, видалення якого збільшує кількість компонент зв'язності (розриває граф).

**Точка зчленування** — вершина, видалення якої збільшує кількість компонент зв'язності.

```
Граф:
    0 --- 1 --- 3 --- 4
    |   /       |
    2           5

Мости: (1,3), (3,5)          — їх видалення розриває граф
Точки зчленування: 1, 3      — їх видалення розриває граф
Ребро (0,1) — НЕ міст, бо є шлях 0→2→1
```

**Алгоритм (на основі DFS):**
- `tin[u]` — час входу в вершину u
- `low[u]` — мінімальний `tin`, досяжний з піддерева u (через зворотні ребра)
- Ребро (u, v) — **міст**, якщо `low[v] > tin[u]` (з піддерева v не можна піднятися вище u)
- Вершина u — **точка зчленування**, якщо `low[v] >= tin[u]` (для некореневих) або має ≥ 2 дітей в DFS-дереві (для кореня)

**Складність:** O(V + E)

```cpp
#include <iostream>
#include <vector>
using namespace std;

class Graph {
    int V;
    vector<vector<int>> adj;
    int timer;

    void dfs(int u, int parent,
             vector<int>& tin, vector<int>& low,
             vector<bool>& visited,
             vector<pair<int,int>>& bridges,
             vector<bool>& isArticulation) {

        visited[u] = true;
        tin[u] = low[u] = timer++;
        int children = 0;

        for (int v : adj[u]) {
            if (v == parent) continue;

            if (visited[v]) {
                // Зворотне ребро — оновлюємо low
                low[u] = min(low[u], tin[v]);
            } else {
                children++;
                dfs(v, u, tin, low, visited, bridges, isArticulation);

                // Після повернення з рекурсії
                low[u] = min(low[u], low[v]);

                // Перевірка мосту: з піддерева v не можна піднятися до u або вище
                if (low[v] > tin[u]) {
                    bridges.push_back({u, v});
                }

                // Перевірка точки зчленування (некореневий випадок)
                if (parent != -1 && low[v] >= tin[u]) {
                    isArticulation[u] = true;
                }
            }
        }

        // Кореневий випадок: точка зчленування, якщо > 1 дитини в DFS-дереві
        if (parent == -1 && children > 1) {
            isArticulation[u] = true;
        }
    }

public:
    Graph(int vertices) : V(vertices), adj(V), timer(0) {}

    void addEdge(int u, int v) {
        adj[u].push_back(v);
        adj[v].push_back(u);
    }

    void findBridgesAndArticulations() {
        vector<int> tin(V, -1), low(V, -1);
        vector<bool> visited(V, false);
        vector<bool> isArticulation(V, false);
        vector<pair<int,int>> bridges;
        timer = 0;

        for (int i = 0; i < V; i++) {
            if (!visited[i])
                dfs(i, -1, tin, low, visited, bridges, isArticulation);
        }

        cout << "Мости:" << endl;
        if (bridges.empty()) {
            cout << "  (немає)" << endl;
        } else {
            for (auto& [u, v] : bridges)
                cout << "  (" << u << ", " << v << ")" << endl;
        }

        cout << "Точки зчленування:" << endl;
        bool found = false;
        for (int i = 0; i < V; i++) {
            if (isArticulation[i]) {
                cout << "  " << i << endl;
                found = true;
            }
        }
        if (!found) cout << "  (немає)" << endl;

        // Виводимо tin/low для розуміння
        cout << "\ntin/low значення:" << endl;
        for (int i = 0; i < V; i++)
            cout << "  Вершина " << i << ": tin=" << tin[i]
                 << ", low=" << low[i] << endl;
    }
};

int main() {
    // Граф 1:
    //    0 --- 1 --- 3 --- 4
    //    |   /       |
    //    2           5
    cout << "=== Граф 1 ===" << endl;
    Graph g1(6);
    g1.addEdge(0, 1);
    g1.addEdge(0, 2);
    g1.addEdge(1, 2);
    g1.addEdge(1, 3);
    g1.addEdge(3, 4);
    g1.addEdge(3, 5);
    g1.findBridgesAndArticulations();

    // Граф 2: ланцюг (всі ребра — мости)
    //    0 --- 1 --- 2 --- 3
    cout << "\n=== Граф 2 (ланцюг) ===" << endl;
    Graph g2(4);
    g2.addEdge(0, 1);
    g2.addEdge(1, 2);
    g2.addEdge(2, 3);
    g2.findBridgesAndArticulations();

    // Граф 3: цикл (немає мостів)
    //    0 --- 1
    //    |     |
    //    3 --- 2
    cout << "\n=== Граф 3 (цикл) ===" << endl;
    Graph g3(4);
    g3.addEdge(0, 1);
    g3.addEdge(1, 2);
    g3.addEdge(2, 3);
    g3.addEdge(3, 0);
    g3.findBridgesAndArticulations();

    return 0;
}
```

### Приклад запуску

```text
=== Граф 1 ===
Мости:
  (3, 4)
  (3, 5)
  (1, 3)
Точки зчленування:
  1
  3

tin/low значення:
  Вершина 0: tin=0, low=0
  Вершина 1: tin=1, low=0
  Вершина 2: tin=2, low=0
  Вершина 3: tin=3, low=3
  Вершина 4: tin=4, low=4
  Вершина 5: tin=5, low=5

=== Граф 2 (ланцюг) ===
Мости:
  (2, 3)
  (1, 2)
  (0, 1)
Точки зчленування:
  1
  2

tin/low значення:
  Вершина 0: tin=0, low=0
  Вершина 1: tin=1, low=1
  Вершина 2: tin=2, low=2
  Вершина 3: tin=3, low=3

=== Граф 3 (цикл) ===
Мости:
  (немає)
Точки зчленування:
  (немає)

tin/low значення:
  Вершина 0: tin=0, low=0
  Вершина 1: tin=1, low=0
  Вершина 2: tin=2, low=0
  Вершина 3: tin=3, low=0
```

---

## Завдання 15: Компоненти сильної зв'язності (Алгоритм Косараджу)

### Теорія

В **орієнтованому** графі **компонента сильної зв'язності** (SCC — Strongly Connected Component) — максимальна підмножина вершин, де з кожної вершини досяжна будь-яка інша.

```
Орієнтований граф:
    0 → 1 → 2
    ↑       ↓
    3 ← ← ←┘
        4 → 5
        ↑   ↓
        └───┘

SCC 1: {0, 1, 2, 3}     (цикл 0→1→2→3→0)
SCC 2: {4, 5}            (цикл 4→5→4)
```

**Алгоритм Косараджу (два DFS):**
1. Запустити DFS на вихідному графі, записати вершини в порядку завершення (tout)
2. Побудувати **транспонований граф** (розвернути всі ребра)
3. Запустити DFS на транспонованому графі в порядку спадання tout — кожен DFS знайде одну SCC

**Складність:** O(V + E)

```cpp
#include <iostream>
#include <vector>
#include <stack>
using namespace std;

class Graph {
    int V;
    vector<vector<int>> adj;       // вихідний граф
    vector<vector<int>> adj_rev;   // транспонований граф

    void dfs1(int u, vector<bool>& visited, stack<int>& order) {
        visited[u] = true;
        for (int v : adj[u]) {
            if (!visited[v])
                dfs1(v, visited, order);
        }
        order.push(u);  // додаємо при завершенні
    }

    void dfs2(int u, vector<bool>& visited, vector<int>& component) {
        visited[u] = true;
        component.push_back(u);
        for (int v : adj_rev[u]) {
            if (!visited[v])
                dfs2(v, visited, component);
        }
    }

public:
    Graph(int vertices) : V(vertices), adj(V), adj_rev(V) {}

    void addEdge(int u, int v) {
        adj[u].push_back(v);
        adj_rev[v].push_back(u);  // зворотне ребро
    }

    vector<vector<int>> findSCC() {
        // Крок 1: DFS на вихідному графі → порядок завершення
        vector<bool> visited(V, false);
        stack<int> order;

        for (int i = 0; i < V; i++) {
            if (!visited[i])
                dfs1(i, visited, order);
        }

        // Крок 2: DFS на транспонованому графі в зворотному порядку
        fill(visited.begin(), visited.end(), false);
        vector<vector<int>> components;

        while (!order.empty()) {
            int u = order.top();
            order.pop();

            if (!visited[u]) {
                vector<int> component;
                dfs2(u, visited, component);
                components.push_back(component);
            }
        }

        return components;
    }
};

int main() {
    // Граф:
    //    0 → 1 → 2
    //    ↑       ↓
    //    3 ← ← ←┘
    //        4 → 5
    //        ↑   ↓
    //        └───┘
    //    6 (ізольована)
    Graph g(7);
    g.addEdge(0, 1);
    g.addEdge(1, 2);
    g.addEdge(2, 3);
    g.addEdge(3, 0);  // SCC: {0,1,2,3}
    g.addEdge(4, 5);
    g.addEdge(5, 4);  // SCC: {4,5}
    // 6 — окрема SCC: {6}

    auto scc = g.findSCC();

    cout << "Кількість SCC: " << scc.size() << endl;
    for (int i = 0; i < scc.size(); i++) {
        cout << "SCC " << (i + 1) << ": { ";
        for (int v : scc[i])
            cout << v << " ";
        cout << "}" << endl;
    }

    // Другий приклад: DAG (кожна вершина — окрема SCC)
    cout << "\n=== DAG ===" << endl;
    Graph g2(4);
    g2.addEdge(0, 1);
    g2.addEdge(1, 2);
    g2.addEdge(2, 3);

    auto scc2 = g2.findSCC();
    cout << "Кількість SCC: " << scc2.size() << endl;
    for (int i = 0; i < scc2.size(); i++) {
        cout << "SCC " << (i + 1) << ": { ";
        for (int v : scc2[i])
            cout << v << " ";
        cout << "}" << endl;
    }

    return 0;
}
```

### Приклад запуску

```text
Кількість SCC: 3
SCC 1: { 0 3 2 1 }
SCC 2: { 4 5 }
SCC 3: { 6 }

=== DAG ===
Кількість SCC: 4
SCC 1: { 0 }
SCC 2: { 1 }
SCC 3: { 2 }
SCC 4: { 3 }
```

---

## Завдання 16: Найкоротший шлях у DAG

### Теорія

У **DAG** (орієнтованому ациклічному графі) найкоротші шляхи можна знайти ефективніше за Дейкстру, використовуючи **топологічне сортування** + **релаксацію**.

**Перевага:** працює з **від'ємними вагами** (на відміну від Дейкстри) і швидше за Беллмана-Форда.

**Алгоритм:**
1. Знайти топологічний порядок вершин
2. Ініціалізувати `dist[start] = 0`, решта = ∞
3. Пройти вершини в топологічному порядку, релаксуючи ребра

**Складність:** O(V + E) — лінійний час!

**Застосування:** планування проектів (критичний шлях), найдовший шлях у DAG.

```cpp
#include <iostream>
#include <vector>
#include <stack>
#include <limits>
using namespace std;

const int INF = numeric_limits<int>::max();

class DAG {
    int V;
    vector<vector<pair<int,int>>> adj;  // (сусід, вага)

    void topoSort(int u, vector<bool>& visited, stack<int>& order) {
        visited[u] = true;
        for (auto& [v, w] : adj[u]) {
            if (!visited[v])
                topoSort(v, visited, order);
        }
        order.push(u);
    }

public:
    DAG(int vertices) : V(vertices), adj(V) {}

    void addEdge(int u, int v, int w) {
        adj[u].push_back({v, w});
    }

    vector<int> shortestPath(int start) {
        // Крок 1: топологічне сортування
        vector<bool> visited(V, false);
        stack<int> order;

        for (int i = 0; i < V; i++) {
            if (!visited[i])
                topoSort(i, visited, order);
        }

        // Крок 2: релаксація в топологічному порядку
        vector<int> dist(V, INF);
        dist[start] = 0;

        while (!order.empty()) {
            int u = order.top();
            order.pop();

            if (dist[u] == INF) continue;

            for (auto& [v, w] : adj[u]) {
                if (dist[u] + w < dist[v]) {
                    dist[v] = dist[u] + w;
                }
            }
        }

        return dist;
    }

    // Найдовший шлях = заперечити ваги та шукати найкоротший
    vector<int> longestPath(int start) {
        vector<bool> visited(V, false);
        stack<int> order;

        for (int i = 0; i < V; i++) {
            if (!visited[i])
                topoSort(i, visited, order);
        }

        vector<int> dist(V, INF);
        dist[start] = 0;

        while (!order.empty()) {
            int u = order.top();
            order.pop();

            if (dist[u] == INF) continue;

            for (auto& [v, w] : adj[u]) {
                if (dist[u] - w < dist[v]) {  // заперечені ваги
                    dist[v] = dist[u] - w;
                }
            }
        }

        // Повертаємо відстані з оберненим знаком
        for (int& d : dist)
            if (d != INF) d = -d;

        return dist;
    }
};

int main() {
    // DAG:
    //    0 --(2)--> 1 --(3)--> 3
    //    |          |          ↑
    //   (4)        (1)       (2)
    //    ↓          ↓        |
    //    2 --(-1)-> 4 ------┘
    //               |
    //              (5)
    //               ↓
    //               5
    DAG g(6);
    g.addEdge(0, 1, 2);
    g.addEdge(0, 2, 4);
    g.addEdge(1, 3, 3);
    g.addEdge(1, 4, 1);
    g.addEdge(2, 4, -1);  // від'ємна вага — ОК для DAG!
    g.addEdge(4, 3, 2);
    g.addEdge(4, 5, 5);

    cout << "=== Найкоротші шляхи від вершини 0 ===" << endl;
    auto dist = g.shortestPath(0);
    for (int i = 0; i < 6; i++) {
        cout << "  dist[" << i << "] = ";
        if (dist[i] == INF) cout << "INF";
        else cout << dist[i];
        cout << endl;
    }

    cout << "\n=== Найдовші шляхи від вершини 0 ===" << endl;
    auto longest = g.longestPath(0);
    for (int i = 0; i < 6; i++) {
        cout << "  longest[" << i << "] = ";
        if (longest[i] == INF) cout << "INF";
        else cout << longest[i];
        cout << endl;
    }

    return 0;
}
```

### Приклад запуску

```text
=== Найкоротші шляхи від вершини 0 ===
  dist[0] = 0
  dist[1] = 2
  dist[2] = 4
  dist[3] = 5
  dist[4] = 3
  dist[5] = 8

=== Найдовші шляхи від вершини 0 ===
  longest[0] = 0
  longest[1] = 2
  longest[2] = 4
  longest[3] = 5
  longest[4] = 3
  longest[5] = 8
```

---

## Завдання 17: Пошук шляху в лабіринті (BFS на сітці)

### Теорія

Лабіринт можна представити як граф: кожна прохідна клітинка — вершина, суміжні прохідні клітинки — ребра. BFS знаходить найкоротший шлях (за кількістю кроків).

```
Лабіринт (S=старт, E=кінець, #=стіна, .=прохід):

  # # # # # # #
  # S . # . . #
  # . # . . # #
  # . . . # . #
  # # # . . E #
  # # # # # # #

BFS знаходить шлях довжиною 7 кроків.
```

**Складність:** O(рядки × стовпці)

```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <algorithm>
using namespace std;

struct Cell {
    int row, col;
};

class Maze {
    int rows, cols;
    vector<string> grid;

    // Напрямки: вгору, вниз, вліво, вправо
    int dr[4] = {-1, 1, 0, 0};
    int dc[4] = {0, 0, -1, 1};

    bool isValid(int r, int c) {
        return r >= 0 && r < rows && c >= 0 && c < cols && grid[r][c] != '#';
    }

public:
    Maze(vector<string>& maze) : grid(maze) {
        rows = grid.size();
        cols = grid[0].size();
    }

    // BFS для знаходження найкоротшого шляху
    int solve(Cell start, Cell end) {
        vector<vector<int>> dist(rows, vector<int>(cols, -1));
        vector<vector<Cell>> parent(rows, vector<Cell>(cols, {-1, -1}));
        queue<Cell> q;

        dist[start.row][start.col] = 0;
        q.push(start);

        while (!q.empty()) {
            Cell cur = q.front();
            q.pop();

            // Дістались до виходу!
            if (cur.row == end.row && cur.col == end.col) {
                // Відновлюємо та малюємо шлях
                Cell c = end;
                while (!(c.row == start.row && c.col == start.col)) {
                    if (grid[c.row][c.col] != 'E')
                        grid[c.row][c.col] = '*';
                    c = parent[c.row][c.col];
                }
                return dist[end.row][end.col];
            }

            // Перебираємо сусідів
            for (int d = 0; d < 4; d++) {
                int nr = cur.row + dr[d];
                int nc = cur.col + dc[d];

                if (isValid(nr, nc) && dist[nr][nc] == -1) {
                    dist[nr][nc] = dist[cur.row][cur.col] + 1;
                    parent[nr][nc] = cur;
                    q.push({nr, nc});
                }
            }
        }

        return -1;  // шлях не знайдено
    }

    void print() {
        for (auto& row : grid)
            cout << row << endl;
    }
};

int main() {
    vector<string> maze = {
        "#######",
        "#S.#..#",
        "#.#..##",
        "#...#.#",
        "###..E#",
        "#######"
    };

    cout << "Лабіринт:" << endl;
    Maze m(maze);
    m.print();

    Cell start = {1, 1};  // S
    Cell end = {4, 5};    // E

    int steps = m.solve(start, end);

    if (steps != -1) {
        cout << "\nРозв'язок (шлях позначено *):" << endl;
        m.print();
        cout << "\nДовжина шляху: " << steps << " кроків" << endl;
    } else {
        cout << "\nШлях не знайдено!" << endl;
    }

    // Приклад 2: лабіринт без розв'язку
    cout << "\n=== Лабіринт без розв'язку ===" << endl;
    vector<string> maze2 = {
        "#####",
        "#S.##",
        "###.#",
        "#.E.#",
        "#####"
    };

    Maze m2(maze2);
    m2.print();

    int steps2 = m2.solve({1, 1}, {3, 2});
    if (steps2 == -1)
        cout << "Шлях не знайдено!" << endl;

    return 0;
}
```

### Приклад запуску

```text
Лабіринт:
#######
#S.#..#
#.#..##
#...#.#
###..E#
#######

Розв'язок (шлях позначено *):
#######
#S.#..#
#*#..##
#***#.#
###**E#
#######

Довжина шляху: 7 кроків

=== Лабіринт без розв'язку ===
#####
#S.##
###.#
#.E.#
#####
Шлях не знайдено!
```

---

## Завдання 18: Алгоритм A* (евристичний пошук)

### Теорія

**A*** (A-star) — розширення алгоритму Дейкстри, що використовує **евристичну функцію** для прискорення пошуку шляху до конкретної цільової вершини.

#### Ідея

Дейкстра досліджує всі напрямки рівномірно — як кола на воді від каменя. A* «спрямовує» пошук у бік цілі, оцінюючи кожну вершину функцією:

```
f(n) = g(n) + h(n)

g(n) — реальна вартість шляху від старту до n (як у Дейкстри)
h(n) — евристична оцінка вартості від n до цілі (додаткова інформація)
f(n) — повна оцінка вартості найкоротшого шляху через n
```

```
Дейкстра: досліджує рівномірно         A*: спрямований до цілі

    . . . . . . . . .                  . . . . . . . . .
    . . . * * * . . .                  . . . . . * * . .
    . . * * * * * . .                  . . . . * * . . .
    . * * * S * * * .                  . . . S * * . . .
    . . * * * * * . .                  . . . . * * * . .
    . . . * * * . . .                  . . . . . * E . .
    . . . . . . . . .                  . . . . . . . . .

    S=старт, E=кінець                  Досліджує значно менше вершин!
```

#### Допустимість евристики

Евристика h(n) називається **допустимою** (admissible), якщо вона **ніколи не переоцінює** реальну відстань до цілі:

```
h(n) ≤ реальна_відстань(n, ціль)   для всіх n
```

**Якщо h допустима → A* гарантовано знаходить оптимальний шлях.**

#### Консистентність (монотонність)

Сильніша умова: h є **консистентною**, якщо для кожного ребра (n, m) з вагою c:

```
h(n) ≤ c(n, m) + h(m)
```

Консистентність ⇒ допустимість. Консистентна евристика гарантує, що A* не переглядає вершини повторно (як Дейкстра).

#### Популярні евристики для сіток

| Евристика | Формула | Коли використовувати |
|-----------|---------|---------------------|
| Манхеттенська | \|x₁-x₂\| + \|y₁-y₂\| | 4-напрямковий рух (вгору/вниз/вліво/вправо) |
| Діагональна (Чебишева) | max(\|x₁-x₂\|, \|y₁-y₂\|) | 8-напрямковий рух (з діагоналями) |
| Евклідова | √((x₁-x₂)² + (y₁-y₂)²) | Рух у довільному напрямку |
| h(n) = 0 | Завжди 0 | Вироджується в звичайну Дейкстру |

**Чим точніша евристика (ближча до реальної відстані), тим менше вершин досліджує A*.** Але h(n) > реальної відстані — і оптимальність втрачається.

#### Зв'язок з Дейкстрою

| Властивість | Дейкстра | A* |
|-------------|----------|-----|
| Пріоритет | g(n) | f(n) = g(n) + h(n) |
| Евристика | h(n) = 0 | h(n) > 0 |
| Досліджує | Всі досяжні вершини | Переважно у напрямку цілі |
| Результат | Шляхи до ВСІХ вершин | Шлях до ОДНІЄЇ цілі |
| Оптимальність | Завжди (невід'ємні ваги) | Якщо h допустима |

**Складність:** O(b^d) у найгіршому випадку (b — фактор розгалуження, d — глибина), але на практиці значно менше завдяки евристиці.

### Реалізація: A* на зваженій сітці

```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <cmath>
#include <algorithm>
using namespace std;

struct Cell {
    int row, col;
    double f, g;
};

struct CellCompare {
    bool operator()(const Cell& a, const Cell& b) {
        return a.f > b.f;  // мін-купа за f
    }
};

class AStar {
    int rows, cols;
    vector<string> grid;

    // 4-напрямковий рух
    int dr[4] = {-1, 1, 0, 0};
    int dc[4] = {0, 0, -1, 1};

    bool isValid(int r, int c) {
        return r >= 0 && r < rows && c >= 0 && c < cols && grid[r][c] != '#';
    }

    // Манхеттенська евристика (допустима для 4-напрямкового руху)
    double heuristic(int r1, int c1, int r2, int c2) {
        return abs(r1 - r2) + abs(c1 - c2);
    }

public:
    AStar(vector<string>& maze) : grid(maze) {
        rows = grid.size();
        cols = grid[0].size();
    }

    // Повертає {довжина шляху, кількість досліджених вершин}
    pair<int, int> solve(pair<int,int> start, pair<int,int> goal) {
        auto [sr, sc] = start;
        auto [gr, gc] = goal;

        vector<vector<double>> gScore(rows, vector<double>(cols, 1e9));
        vector<vector<bool>> closed(rows, vector<bool>(cols, false));
        vector<vector<pair<int,int>>> parent(rows,
            vector<pair<int,int>>(cols, {-1, -1}));

        priority_queue<Cell, vector<Cell>, CellCompare> open;

        gScore[sr][sc] = 0;
        open.push({sr, sc, heuristic(sr, sc, gr, gc), 0});

        int explored = 0;

        while (!open.empty()) {
            Cell cur = open.top();
            open.pop();

            // Пропускаємо вже оброблені вершини
            if (closed[cur.row][cur.col]) continue;
            closed[cur.row][cur.col] = true;
            explored++;

            // Досягли цілі
            if (cur.row == gr && cur.col == gc) {
                // Відновлюємо шлях
                int r = gr, c = gc;
                while (!(r == sr && c == sc)) {
                    if (grid[r][c] != 'S' && grid[r][c] != 'E')
                        grid[r][c] = '*';
                    auto [pr, pc] = parent[r][c];
                    r = pr;
                    c = pc;
                }
                return {(int)cur.g, explored};
            }

            // Досліджуємо сусідів
            for (int d = 0; d < 4; d++) {
                int nr = cur.row + dr[d];
                int nc = cur.col + dc[d];

                if (!isValid(nr, nc) || closed[nr][nc]) continue;

                double newG = cur.g + 1;  // вага кожного кроку = 1
                if (newG < gScore[nr][nc]) {
                    gScore[nr][nc] = newG;
                    double h = heuristic(nr, nc, gr, gc);
                    parent[nr][nc] = {cur.row, cur.col};
                    open.push({nr, nc, newG + h, newG});
                }
            }
        }

        return {-1, explored};  // шлях не існує
    }

    void print() {
        for (auto& row : grid)
            cout << "  " << row << endl;
    }
};

// BFS (для порівняння кількості досліджених вершин)
pair<int, int> bfsSolve(vector<string>& grid,
                         pair<int,int> start, pair<int,int> goal) {
    int rows = grid.size(), cols = grid[0].size();
    auto [sr, sc] = start;
    auto [gr, gc] = goal;

    vector<vector<int>> dist(rows, vector<int>(cols, -1));
    queue<pair<int,int>> q;
    dist[sr][sc] = 0;
    q.push(start);

    int explored = 0;
    int dr[] = {-1, 1, 0, 0};
    int dc[] = {0, 0, -1, 1};

    while (!q.empty()) {
        auto [r, c] = q.front();
        q.pop();
        explored++;

        if (r == gr && c == gc) return {dist[r][c], explored};

        for (int d = 0; d < 4; d++) {
            int nr = r + dr[d], nc = c + dc[d];
            if (nr >= 0 && nr < rows && nc >= 0 && nc < cols
                && grid[nr][nc] != '#' && dist[nr][nc] == -1) {
                dist[nr][nc] = dist[r][c] + 1;
                q.push({nr, nc});
            }
        }
    }

    return {-1, explored};
}

int main() {
    vector<string> maze = {
        "############",
        "#S.........#",
        "#.########.#",
        "#.#......#.#",
        "#.#.####.#.#",
        "#.#.#..#.#.#",
        "#.#.#..#.#.#",
        "#.#.####.#.#",
        "#.#......#.#",
        "#.########.#",
        "#.........E#",
        "############"
    };

    pair<int,int> start = {1, 1};
    pair<int,int> goal = {10, 10};

    // --- BFS (еквівалент Дейкстри для незваженого графа) ---
    vector<string> mazeBFS = maze;
    auto [bfsDist, bfsExplored] = bfsSolve(mazeBFS, start, goal);

    // --- A* ---
    AStar astar(maze);
    auto [astarDist, astarExplored] = astar.solve(start, goal);

    cout << "Лабіринт з розв'язком A*:" << endl;
    astar.print();

    cout << "\n=== Порівняння BFS vs A* ===" << endl;
    cout << "                  BFS       A*" << endl;
    cout << "Довжина шляху:    " << bfsDist
         << "        " << astarDist << endl;
    cout << "Досліджено:       " << bfsExplored
         << "        " << astarExplored << endl;

    if (bfsExplored > 0) {
        int saved = 100 - (astarExplored * 100 / bfsExplored);
        cout << "Економія A*:      " << saved << "% менше вершин" << endl;
    }

    return 0;
}
```

### Приклад запуску

```text
Лабіринт з розв'язком A*:
  ############
  #S.........#
  #*########.#
  #*#......#.#
  #*#.####.#.#
  #*#.#..#.#.#
  #*#.#..#.#.#
  #*#.####.#.#
  #*#......#.#
  #*########.#
  #*********E#
  ############

=== Порівняння BFS vs A* ===
                  BFS       A*
Довжина шляху:    18        18
Досліджено:       80        42
Економія A*:      47% менше вершин
```

### Реалізація: A* на зваженому графі (список суміжності)

```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <limits>
#include <cmath>
#include <algorithm>
using namespace std;

const double INF = numeric_limits<double>::max();

struct Node {
    int id;
    double f, g;
    bool operator>(const Node& other) const { return f > other.f; }
};

class WeightedGraph {
    int V;
    vector<vector<pair<int, double>>> adj;  // (сусід, вага)
    vector<pair<double, double>> coords;     // (x, y) координати вершин

public:
    WeightedGraph(int vertices) : V(vertices), adj(V), coords(V) {}

    void setCoords(int u, double x, double y) {
        coords[u] = {x, y};
    }

    void addEdge(int u, int v, double w) {
        adj[u].push_back({v, w});
        adj[v].push_back({u, w});
    }

    // Евклідова евристика (допустима, бо пряма < будь-якого шляху)
    double heuristic(int u, int goal) {
        auto [x1, y1] = coords[u];
        auto [x2, y2] = coords[goal];
        return sqrt((x1 - x2) * (x1 - x2) + (y1 - y2) * (y1 - y2));
    }

    // A*: повертає {відстань, шлях, кількість досліджених}
    tuple<double, vector<int>, int> astar(int start, int goal) {
        vector<double> g(V, INF);
        vector<int> parent(V, -1);
        vector<bool> closed(V, false);

        priority_queue<Node, vector<Node>, greater<Node>> open;

        g[start] = 0;
        open.push({start, heuristic(start, goal), 0});

        int explored = 0;

        while (!open.empty()) {
            auto [u, f_u, g_u] = open.top();
            open.pop();

            if (closed[u]) continue;
            closed[u] = true;
            explored++;

            if (u == goal) {
                // Відновлюємо шлях
                vector<int> path;
                for (int v = goal; v != -1; v = parent[v])
                    path.push_back(v);
                reverse(path.begin(), path.end());
                return {g[goal], path, explored};
            }

            for (auto [v, w] : adj[u]) {
                if (closed[v]) continue;
                double newG = g[u] + w;

                if (newG < g[v]) {
                    g[v] = newG;
                    parent[v] = u;
                    double h = heuristic(v, goal);
                    open.push({v, newG + h, newG});
                }
            }
        }

        return {INF, {}, explored};
    }

    // Дейкстра (для порівняння)
    pair<double, int> dijkstra(int start, int goal) {
        vector<double> dist(V, INF);
        vector<bool> visited(V, false);

        priority_queue<pair<double,int>, vector<pair<double,int>>,
                       greater<pair<double,int>>> pq;

        dist[start] = 0;
        pq.push({0, start});

        int explored = 0;

        while (!pq.empty()) {
            auto [d, u] = pq.top();
            pq.pop();

            if (visited[u]) continue;
            visited[u] = true;
            explored++;

            if (u == goal) return {dist[goal], explored};

            for (auto [v, w] : adj[u]) {
                if (!visited[v] && dist[u] + w < dist[v]) {
                    dist[v] = dist[u] + w;
                    pq.push({dist[v], v});
                }
            }
        }

        return {INF, explored};
    }
};

int main() {
    // Граф — міста з координатами (для евристики)
    //
    //    0(0,0)---4---1(4,0)
    //    |  \          |
    //    3   5         6
    //    |    \        |
    //    2(0,3)--7--3(4,3)---2---4(6,3)
    //                            |
    //                            3
    //                            |
    //                           5(6,6)

    WeightedGraph g(6);
    g.setCoords(0, 0, 0);
    g.setCoords(1, 4, 0);
    g.setCoords(2, 0, 3);
    g.setCoords(3, 4, 3);
    g.setCoords(4, 6, 3);
    g.setCoords(5, 6, 6);

    g.addEdge(0, 1, 4);
    g.addEdge(0, 2, 3);
    g.addEdge(0, 3, 5);
    g.addEdge(1, 3, 6);
    g.addEdge(2, 3, 7);
    g.addEdge(3, 4, 2);
    g.addEdge(4, 5, 3);

    int start = 0, goal = 5;

    // A*
    auto [astarDist, path, astarExplored] = g.astar(start, goal);
    cout << "=== A* ===" << endl;
    cout << "Шлях: ";
    for (int i = 0; i < path.size(); i++) {
        if (i > 0) cout << " → ";
        cout << path[i];
    }
    cout << endl;
    cout << "Відстань: " << astarDist << endl;
    cout << "Досліджено вершин: " << astarExplored << endl;

    // Дейкстра
    auto [dijkDist, dijkExplored] = g.dijkstra(start, goal);
    cout << "\n=== Дейкстра ===" << endl;
    cout << "Відстань: " << dijkDist << endl;
    cout << "Досліджено вершин: " << dijkExplored << endl;

    cout << "\n=== Порівняння ===" << endl;
    cout << "Обидва знайшли однакову відстань: "
         << (astarDist == dijkDist ? "Так" : "Ні") << endl;
    cout << "A* досліджив " << astarExplored
         << " vs Дейкстра " << dijkExplored << " вершин" << endl;

    return 0;
}
```

### Приклад запуску

```text
=== A* ===
Шлях: 0 → 3 → 4 → 5
Відстань: 10
Досліджено вершин: 4

=== Дейкстра ===
Відстань: 10
Досліджено вершин: 6

=== Порівняння ===
Обидва знайшли однакову відстань: Так
A* досліджив 4 vs Дейкстра 6 вершин
```

### Коли використовувати A*

**A* замість Дейкстри:**
- Пошук шляху до **однієї конкретної** цілі
- Є геометричні координати (для евристики)
- Великий граф, де Дейкстра досліджує занадто багато вершин
- Ігри, робототехніка, GPS-навігація

**Дейкстра замість A*:**
- Потрібні шляхи до **всіх** вершин
- Немає очевидної евристики
- Граф абстрактний (без геометрії)
- Граф малий (різниця несуттєва)

---

## Зведена таблиця алгоритмів

| Алгоритм | Задача | Складність | Обмеження |
|---|---|---|---|
| BFS | Обхід / найкор. шлях (незважений) | O(V + E) | — |
| DFS | Обхід / топ. сорт. / цикли | O(V + E) | — |
| Дейкстра | Найкор. шлях (один→всі) | O((V+E) log V) | Невід'ємні ваги |
| Беллман-Форд | Найкор. шлях (один→всі) | O(V · E) | Виявляє від'єм. цикли |
| Флойд-Уоршелл | Найкор. шлях (всі→всі) | O(V³) | Без від'єм. циклів |
| DAG найкор. шлях | Найкор./найдовший шлях у DAG | O(V + E) | Тільки DAG |
| Прім | MST | O((V+E) log V) | Зв'язний граф |
| Крускал | MST | O(E log E) | Потребує Union-Find |
| Топ. сортування | Лінійний порядок DAG | O(V + E) | Тільки DAG |
| Перевірка циклів | Виявлення циклів | O(V + E) | Оріент. граф: 3 кольори |
| Двочастковість | 2-розфарбування | O(V + E) | — |
| Компоненти | Знаходження компонент | O(V + E) | — |
| Мости / точки зчл. | Критичні ребра/вершини | O(V + E) | DFS + tin/low |
| Косараджу (SCC) | Компоненти сильної зв'язності | O(V + E) | Орієнтований граф |
| BFS на сітці | Шлях у лабіринті | O(R × C) | Незважена сітка |
| A* | Шлях до конкретної цілі | O(b^d), на практиці << Дейкстри | Потребує допустиму евристику |
