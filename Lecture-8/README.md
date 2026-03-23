# Лекція 8 — Union-Find (Disjoint Set Union)

---

## Зміст

1. [Вступ та мотивація](#1-вступ-та-мотивація)
2. [Формальне визначення](#2-формальне-визначення)
3. [Наївні підходи](#3-наївні-підходи)
4. [Оптимізації: ранг і стиснення шляхів](#4-оптимізації-ранг-і-стиснення-шляхів)
5. [Повна реалізація на C++](#5-повна-реалізація-на-c)
6. [Просунуті техніки](#6-просунуті-техніки)
7. [Застосування](#7-застосування)
8. [Розбір задач](#8-розбір-задач)
9. [Підсумки](#9-підсумки)

---

## 1. Вступ та мотивація

### Задача про зв'язність

Уявіть, що маємо **n** об'єктів (вершини графа, комп'ютери в мережі, люди в соціальній мережі). Нам потрібно ефективно виконувати дві операції:

- **Union(a, b)** — об'єднати множини, до яких належать елементи `a` та `b`
- **Find(a)** — визначити, до якої множини належить елемент `a`

Додатково нас часто цікавить:
- **Connected(a, b)** — чи належать `a` та `b` до однієї множини? (тобто `Find(a) == Find(b)`)

### Приклади з реального життя

| Домен | Елементи | Union | Find / Connected |
|-------|----------|-------|-----------------|
| Соціальна мережа | Користувачі | Додати дружбу | Чи у спільній групі? |
| Комп'ютерна мережа | Комп'ютери | З'єднати кабелем | Чи є зв'язок між машинами? |
| Електрична схема | Контакти | З'єднати провідником | Чи є електричний зв'язок? |
| Обробка зображень | Пікселі | Об'єднати в регіон | Чи належать пікселі до одного об'єкта? |

### Чому не простий масив чи граф?

- **Масив належності** — Union за O(n), бо треба оновити всі елементи однієї множини
- **Граф (BFS/DFS)** — Connected за O(n) в найгіршому випадку
- **Union-Find** — обидві операції за **майже O(1)** (амортизовано)

---

## 2. Формальне визначення

### Абстрактний тип даних

**Система непересічних множин** (Disjoint Set Union, DSU) — це абстрактний тип даних, що підтримує колекцію **непересічних** (disjoint) динамічних множин S = {S₁, S₂, ..., Sₖ}.

**Непересічність** означає, що кожен елемент належить рівно одній множині:

```
Sᵢ ∩ Sⱼ = ∅  для всіх i ≠ j
S₁ ∪ S₂ ∪ ... ∪ Sₖ = U  (універсальна множина)
```

**Представник** (representative) — виділений елемент кожної множини, що однозначно її ідентифікує. Будь-який елемент множини може бути її представником, але представник фіксований, поки множина не змінюється.

### Інтерфейс операцій

| Операція | Опис | Передумова |
|----------|------|------------|
| **MakeSet(x)** | Створює нову множину {x} з єдиним елементом x | x не належить жодній існуючій множині |
| **Find(x)** | Повертає представника множини, що містить x | x належить якійсь множині |
| **Union(x, y)** | Об'єднує множини, що містять x та y, в одну | x та y належать різним множинам |

**Інваріанти:**
- Після `MakeSet(x)`: `Find(x) = x`
- Після `Union(x, y)`: `Find(x) = Find(y)` для всіх елементів обох множин
- `Find(x) = Find(y)` тоді і тільки тоді, коли x та y в одній множині

### Представлення через ліс кореневих дерев

Кожну множину представляємо як **кореневе дерево**, де:
- Кожен вузол вказує на свого **батька**
- **Корінь** вказує сам на себе (`parent[root] = root`)
- Корінь дерева є **представником** множини

```
Множина {0, 1, 2, 3, 4}         Множина {5, 6, 7}

        2 (корінь/представник)           6 (корінь/представник)
       / \                              / \
      0   3                            5   7
          |
          1
          |
          4

Find(4) → 1 → 3 → 2           Find(5) → 6
Представник = 2                 Представник = 6
```

### Відношення еквівалентності

DSU моделює **відношення еквівалентності** — бінарне відношення R, що задовольняє три властивості:

1. **Рефлексивність:** a R a (кожен елемент еквівалентний самому собі)
2. **Симетричність:** якщо a R b, то b R a
3. **Транзитивність:** якщо a R b і b R c, то a R c

Кожна множина в DSU — це **клас еквівалентності**. Операція `Union` — це злиття двох класів еквівалентності.

**Приклад:** «бути в одній мережі» — відношення еквівалентності:
- Комп'ютер з'єднаний сам із собою (рефлексивність)
- Якщо A з'єднаний з B, то B з'єднаний з A (симетричність)
- Якщо A з'єднаний з B, а B з C, то A з'єднаний з C (транзитивність)

---

## 3. Наївні підходи

### 3.1 Quick-Find

**Ідея:** зберігаємо масив `id[]`, де `id[i]` — ідентифікатор множини, до якої належить елемент `i`.

```
Початковий стан (кожен елемент — окрема множина):
Елемент: 0  1  2  3  4
id:      0  1  2  3  4

Після Union(1, 2):
Елемент: 0  1  2  3  4
id:      0  2  2  3  4

Після Union(3, 4):
Елемент: 0  1  2  3  4
id:      0  2  2  4  4

Після Union(1, 3):  ← потрібно змінити id всіх елементів множини 1 (тобто 1 і 2)
Елемент: 0  1  2  3  4
id:      0  4  4  4  4
```

```cpp
class QuickFind {
    vector<int> id;

public:
    QuickFind(int n) : id(n) {
        // Кожен елемент — представник своєї множини
        for (int i = 0; i < n; i++)
            id[i] = i;
    }

    // O(1) — просто порівнюємо ідентифікатори
    int find(int x) {
        return id[x];
    }

    bool connected(int a, int b) {
        return find(a) == find(b);
    }

    // O(n) — потрібно пройти весь масив
    void unite(int a, int b) {
        int id_a = find(a);
        int id_b = find(b);
        if (id_a == id_b) return;

        // Замінюємо всі входження id_a на id_b
        for (int i = 0; i < id.size(); i++) {
            if (id[i] == id_a)
                id[i] = id_b;
        }
    }
};
```

**Складність:**

| Операція | Час |
|----------|-----|
| Find | O(1) |
| Union | O(n) |
| Connected | O(1) |

**Проблема:** якщо є m операцій Union над n елементами, загальна складність — **O(m·n)**, що неприйнятно для великих даних.

### 3.2 Quick-Union

**Ідея:** замість плоского масиву будуємо **ліс** (набір дерев). Кожен елемент зберігає посилання на свого батька. Корінь дерева — представник множини.

```
Початковий стан (кожен — корінь):
parent: [0, 1, 2, 3, 4]

Дерева:  0   1   2   3   4

Після Union(1, 2):  (1 вказує на 2)
parent: [0, 2, 2, 3, 4]

Дерева:  0   2   3   4
             |
             1

Після Union(3, 4):  (3 вказує на 4)
parent: [0, 2, 2, 4, 4]

Дерева:  0   2   4
             |   |
             1   3

Після Union(1, 4):  (корінь 1 = 2, корінь 4 = 4 → 2 вказує на 4)
parent: [0, 2, 4, 4, 4]

Дерева:  0     4
              / \
             2   3
             |
             1
```

```cpp
class QuickUnion {
    vector<int> parent;

public:
    QuickUnion(int n) : parent(n) {
        for (int i = 0; i < n; i++)
            parent[i] = i;
    }

    // O(n) у найгіршому випадку — дерево може виродитись у ланцюг
    int find(int x) {
        while (x != parent[x])
            x = parent[x];
        return x;
    }

    bool connected(int a, int b) {
        return find(a) == find(b);
    }

    // O(n) у найгіршому випадку (через find)
    void unite(int a, int b) {
        int root_a = find(a);
        int root_b = find(b);
        if (root_a == root_b) return;
        parent[root_a] = root_b;
    }
};
```

**Проблема виродження:**

```
Union(0,1), Union(1,2), Union(2,3), Union(3,4)

Дерево вироджується у ланцюг:
4 → 3 → 2 → 1 → 0

Find(0) потребує O(n) кроків!
```

**Складність:**

| Операція | Середній | Найгірший |
|----------|----------|-----------|
| Find | O(log n)* | O(n) |
| Union | O(log n)* | O(n) |

*\* середній випадок за умови випадкових об'єднань*

---

## 4. Оптимізації: ранг і стиснення шляхів

### 4.1 Union by Rank (об'єднання за рангом)

**Ідея:** при об'єднанні двох дерев завжди приєднуємо **менше** дерево до **більшого**. Це запобігає виродженню дерева у ланцюг.

**Ранг** (rank) — верхня межа висоти дерева.

```
Без оптимізації (ланцюг):        З Union by Rank (збалансоване):

    4                                  2
    |                                 / \
    3                                1   3
    |                               |     |
    2                               0     4
    |
    1
    |
    0

Висота: 4                          Висота: 2
```

**Правило:** корінь дерева з меншим рангом стає нащадком кореня з більшим рангом. Якщо ранги рівні — вибираємо довільно та збільшуємо ранг на 1.

```cpp
void unite(int a, int b) {
    int root_a = find(a);
    int root_b = find(b);
    if (root_a == root_b) return;

    // Приєднуємо менше дерево до більшого
    if (rank[root_a] < rank[root_b]) {
        parent[root_a] = root_b;
    } else if (rank[root_a] > rank[root_b]) {
        parent[root_b] = root_a;
    } else {
        parent[root_b] = root_a;
        rank[root_a]++;
    }
}
```

### 4.2 Доведення: висота ≤ log₂(n) при Union by Rank

**Лема 1:** Дерево з рангом r містить **щонайменше 2ʳ вузлів**.

**Доведення** (індукцією за кількістю операцій Union):

- **Базовий випадок:** після `MakeSet` ранг = 0, вузлів = 1 = 2⁰ ✓
- **Індуктивний крок:** ранг збільшується лише коли об'єднуємо два дерева з **однаковим** рангом r. Кожне з них має ≥ 2ʳ вузлів (за припущенням індукції). Результат має ранг r+1 і ≥ 2ʳ + 2ʳ = 2ʳ⁺¹ вузлів ✓

**Наслідок:** якщо дерево має n вузлів і ранг r, то n ≥ 2ʳ, звідки:

```
r ≤ log₂(n)
```

Оскільки ранг є верхньою межею висоти, `Find` виконується за **O(log n)**.

**Лема 2:** кількість вузлів з рангом r не перевищує n / 2ʳ.

**Доведення:** кожен вузол з рангом r є (або був) коренем піддерева з ≥ 2ʳ вузлами. Ці піддерева не перетинаються. Тому вузлів з рангом r не більше n / 2ʳ.

```
Ранг:    0      1      2      3
Макс.    n      n/2    n/4    n/8   ...  вузлів кожного рангу
         ────── ────── ────── ──────
Сума:    n + n/2 + n/4 + ... ≤ 2n
```

### 4.3 Path Compression (стиснення шляхів)

**Ідея:** під час виконання `find(x)` перенаправляємо всі вузли на шляху безпосередньо до кореня. Наступні виклики `find()` для цих вузлів будуть за O(1).

```
До стиснення:              Після find(0):

    4                          4
    |                        / | \ \
    3                       0  1  2  3
    |
    2
    |
    1
    |
    0
```

```cpp
// Рекурсивна версія
int find(int x) {
    if (x != parent[x])
        parent[x] = find(parent[x]);  // стиснення: вказуємо прямо на корінь
    return parent[x];
}

// Ітеративна версія (два проходи)
int find(int x) {
    // Перший прохід: знаходимо корінь
    int root = x;
    while (root != parent[root])
        root = parent[root];

    // Другий прохід: стискаємо шлях
    while (x != root) {
        int next = parent[x];
        parent[x] = root;
        x = next;
    }
    return root;
}
```

### 4.4 Альтернативні стратегії стиснення

Окрім повного стиснення (Path Compression), існують дві легші стратегії, які простіші в реалізації та мають ту ж амортизовану складність:

#### Path Splitting (розщеплення шляху)

Кожен вузол на шляху починає вказувати на свого **діда** (батька батька). Один прохід, без рекурсії.

```
До:              Після find(0) з Path Splitting:

    5                5
    |               / \
    4              3   4
    |              |
    3              2
    |              |
    2              0
    |
    1
    |
    0

Кожен вузол перестрибнув на рівень вище.
```

```cpp
int find(int x) {
    while (x != parent[x]) {
        int next = parent[x];
        parent[x] = parent[next];  // вказуємо на діда
        x = next;
    }
    return x;
}
```

#### Path Halving (половинне стиснення)

Кожен **другий** вузол на шляху починає вказувати на діда. Один прохід, одна змінна.

```cpp
int find(int x) {
    while (x != parent[x]) {
        parent[x] = parent[parent[x]];  // вказуємо на діда
        x = parent[x];                  // перестрибуємо
    }
    return x;
}
```

**Порівняння стратегій стиснення:**

| Стратегія | Повне стиснення | Path Splitting | Path Halving |
|-----------|-----------------|----------------|--------------|
| Амортизована складність | O(α(n)) | O(α(n)) | O(α(n)) |
| Кількість проходів | 2 (або рекурсія) | 1 | 1 |
| Ефект стиснення | Максимальний | Помірний | Помірний |
| Реалізація | Рекурсія або 2 цикли | 1 цикл | 1 цикл |

На практиці всі три дають однакову асимптотику. Path Splitting та Path Halving часто використовуються в ітеративних реалізаціях, де рекурсія небажана (наприклад, через глибину стеку).

### 4.5 Комбінація оптимізацій та аналіз складності

При одночасному використанні **Union by Rank** та **Path Compression** амортизована складність кожної операції становить:

$$O(\alpha(n))$$

де **α(n)** — обернена функція Аккермана.

#### Функція Аккермана

Функція Аккермана A(m, n) — одна з найшвидше зростаючих обчислюваних функцій:

```
A(0, n) = n + 1
A(m, 0) = A(m-1, 1)
A(m, n) = A(m-1, A(m, n-1))

Приклади:
A(0, n) = n + 1                    (лінійний ріст)
A(1, n) = n + 2                    (лінійний ріст)
A(2, n) = 2n + 3                   (лінійний ріст)
A(3, n) = 2^(n+3) - 3              (експоненціальний ріст)
A(4, n) = 2^2^2^...^2 - 3          (вежа з n+3 двійок!)
A(4, 2) = 2^(2^(2^(2^65536))) - 3  (число з ~10^19728 цифрами)
```

**Обернена функція Аккермана** α(n) = min{k : A(k, k) ≥ n}:

| n | α(n) | Коментар |
|---|------|----------|
| 1 | 0 | |
| 2–3 | 1 | |
| 4–7 | 2 | |
| 8–2047 | 3 | |
| 2048 – A(4,4) | 4 | A(4,4) — число з ~10^19728 цифрами |
| Будь-яке практичне n | ≤ 4 | α(n) ≤ 4 для n < кількість атомів у Всесвіті |

#### Інтуїція за амортизованим аналізом

Чому комбінація працює так добре?

**Union by Rank** гарантує, що дерева не глибокі (висота ≤ log n). Це обмежує початкову вартість find.

**Path Compression** «оплачує» поточну дорогу тим, що скорочує шляхи для **всіх майбутніх** операцій. Довга операція find означає, що багато вузлів було стиснуто — і наступні операції будуть дешевшими.

```
Уявімо послідовність з m операцій find на n елементах:

Без оптимізацій:       O(m · n)     — кожен find може бути O(n)
Union by Rank:         O(m · log n) — кожен find O(log n)
Path Compression:      O(m · log n) — амортизовано O(log n)
Обидві разом:          O(m · α(n))  — амортизовано "майже O(1)"

Стиснення шляхів не просто зменшує log n до α(n) —
воно радикально змінює структуру дерева після серії операцій.
```

**Ключовий момент:** після серії операцій зі стисненням, дерева стають **майже плоскими** — переважна більшість вузлів вказує безпосередньо на корінь або на вузол, дуже близький до кореня.

> **Теорема (Tarjan, 1975):** Послідовність m операцій Union/Find на n елементах з використанням Union by Rank та Path Compression виконується за O(m · α(n)) часу.

> **Нижня межа (Fredman & Saks, 1989):** Ω(m · α(n)) — це оптимально. Жодна структура даних не може виконати m операцій Union/Find швидше за цю межу в моделі з клітинковими зондами (cell probe model).

---

## 5. Повна реалізація на C++

### 5.1 Базова реалізація

```cpp
#include <iostream>
#include <vector>
using namespace std;

class DSU {
    vector<int> parent;
    vector<int> rank_;
    int components;  // кількість компонент зв'язності

public:
    DSU(int n) : parent(n), rank_(n, 0), components(n) {
        for (int i = 0; i < n; i++)
            parent[i] = i;
    }

    // Знайти представника множини з стисненням шляхів
    int find(int x) {
        if (x != parent[x])
            parent[x] = find(parent[x]);
        return parent[x];
    }

    // Об'єднати множини за рангом
    bool unite(int a, int b) {
        int ra = find(a);
        int rb = find(b);
        if (ra == rb) return false;  // вже в одній множині

        // Приєднуємо менше дерево до більшого
        if (rank_[ra] < rank_[rb]) swap(ra, rb);
        parent[rb] = ra;
        if (rank_[ra] == rank_[rb]) rank_[ra]++;

        components--;
        return true;  // об'єднання відбулось
    }

    bool connected(int a, int b) {
        return find(a) == find(b);
    }

    int count_components() {
        return components;
    }
};
```

### 5.2 Альтернатива: Union by Size (об'єднання за розміром)

Замість рангу можна зберігати **розмір** кожної компоненти. Це корисно, коли потрібно знати розмір множини.

```cpp
class DSU_Size {
    vector<int> parent;
    vector<int> size_;

public:
    DSU_Size(int n) : parent(n), size_(n, 1) {
        for (int i = 0; i < n; i++)
            parent[i] = i;
    }

    int find(int x) {
        if (x != parent[x])
            parent[x] = find(parent[x]);
        return parent[x];
    }

    bool unite(int a, int b) {
        int ra = find(a);
        int rb = find(b);
        if (ra == rb) return false;

        // Приєднуємо менше дерево до більшого
        if (size_[ra] < size_[rb]) swap(ra, rb);
        parent[rb] = ra;
        size_[ra] += size_[rb];

        return true;
    }

    bool connected(int a, int b) {
        return find(a) == find(b);
    }

    // Повертає розмір множини, до якої належить x
    int component_size(int x) {
        return size_[find(x)];
    }
};
```

### 5.3 Приклад роботи

```cpp
int main() {
    DSU dsu(7);
    // Елементи: 0, 1, 2, 3, 4, 5, 6

    cout << "Початкова кількість компонент: " << dsu.count_components() << endl;
    // Виведе: 7

    // Створюємо зв'язки
    dsu.unite(0, 1);
    dsu.unite(2, 3);
    dsu.unite(4, 5);
    cout << "Після 3 об'єднань: " << dsu.count_components() << " компонент" << endl;
    // Виведе: 4   (компоненти: {0,1}, {2,3}, {4,5}, {6})

    cout << "0 і 1 зв'язані? " << (dsu.connected(0, 1) ? "Так" : "Ні") << endl;
    // Виведе: Так
    cout << "0 і 2 зв'язані? " << (dsu.connected(0, 2) ? "Так" : "Ні") << endl;
    // Виведе: Ні

    dsu.unite(1, 3);  // об'єднуємо {0,1} та {2,3}
    cout << "0 і 2 зв'язані? " << (dsu.connected(0, 2) ? "Так" : "Ні") << endl;
    // Виведе: Так

    cout << "Фінальна кількість компонент: " << dsu.count_components() << endl;
    // Виведе: 3   (компоненти: {0,1,2,3}, {4,5}, {6})

    return 0;
}
```

**Вивід:**

```text
Початкова кількість компонент: 7
Після 3 об'єднань: 4 компонент
0 і 1 зв'язані? Так
0 і 2 зв'язані? Ні
0 і 2 зв'язані? Так
Фінальна кількість компонент: 3
```

---

## 6. Просунуті техніки

### 6.1 Weighted DSU (DSU з вагами)

У базовому DSU ми лише знаємо, чи два елементи в одній множині. **Weighted DSU** додатково зберігає **вагу** (або відстань) кожного елемента відносно кореня його дерева. Це дозволяє відповідати на запити виду: «яка різниця/відстань між елементами a та b?»

**Задача-приклад:** є n змінних x₀, x₁, ..., xₙ₋₁. Послідовно надходять факти виду «xₐ - xᵦ = w». Потрібно визначати, чи новий факт суперечить попереднім, і відповідати на запити «чому дорівнює xₐ - xᵦ?»

**Ідея:** зберігаємо `weight[x]` — значення x відносно кореня його дерева. Тоді `xₐ - xᵦ = weight[a] - weight[b]` (якщо a та b в одному дереві).

```
Після факту "x1 - x0 = 5":

    0 (корінь)          weight[0] = 0
    |                   weight[1] = 5
    1                   x1 - x0 = weight[1] - weight[0] = 5

Після факту "x2 - x1 = 3":

    0 (корінь)          weight[0] = 0
    |                   weight[1] = 5
    1                   weight[2] = 8  (5 + 3)
    |
    2                   x2 - x0 = weight[2] - weight[0] = 8
```

```cpp
#include <iostream>
#include <vector>
using namespace std;

class WeightedDSU {
    vector<int> parent, rank_;
    vector<long long> weight;  // weight[x] = значення x відносно кореня

public:
    WeightedDSU(int n) : parent(n), rank_(n, 0), weight(n, 0) {
        for (int i = 0; i < n; i++) parent[i] = i;
    }

    // Повертає {корінь, вага x відносно кореня}
    pair<int, long long> find(int x) {
        if (x == parent[x]) return {x, 0};

        auto [root, w] = find(parent[x]);
        parent[x] = root;
        weight[x] += w;  // накопичуємо вагу при стисненні
        return {root, weight[x]};
    }

    // Додати факт: x_a - x_b = w
    // Повертає false, якщо суперечить існуючим фактам
    bool unite(int a, int b, long long w) {
        auto [ra, wa] = find(a);  // wa = a відносно кореня ra
        auto [rb, wb] = find(b);  // wb = b відносно кореня rb

        if (ra == rb) {
            // Вже в одній множині — перевіряємо несуперечливість
            // a - b = wa - wb (за існуючими фактами)
            return (wa - wb) == w;
        }

        // Об'єднуємо: потрібно weight[ra] відносно rb
        // a - b = w → (ra + wa) - (rb + wb) = w → ra - rb = w - wa + wb
        if (rank_[ra] < rank_[rb]) {
            parent[ra] = rb;
            weight[ra] = w - wa + wb;
        } else if (rank_[ra] > rank_[rb]) {
            parent[rb] = ra;
            weight[rb] = -w + wa - wb;
        } else {
            parent[rb] = ra;
            weight[rb] = -w + wa - wb;
            rank_[ra]++;
        }

        return true;
    }

    // Запит: x_a - x_b = ?
    // Повертає {чи можливо відповісти, різницю}
    pair<bool, long long> query(int a, int b) {
        auto [ra, wa] = find(a);
        auto [rb, wb] = find(b);

        if (ra != rb) return {false, 0};  // не пов'язані
        return {true, wa - wb};
    }
};

int main() {
    WeightedDSU dsu(5);

    // Факт: x1 - x0 = 5
    cout << "x1 - x0 = 5: "
         << (dsu.unite(1, 0, 5) ? "OK" : "Суперечність!") << endl;

    // Факт: x2 - x1 = 3
    cout << "x2 - x1 = 3: "
         << (dsu.unite(2, 1, 3) ? "OK" : "Суперечність!") << endl;

    // Запит: x2 - x0 = ?
    auto [ok1, diff1] = dsu.query(2, 0);
    if (ok1) cout << "x2 - x0 = " << diff1 << endl;  // 8

    // Факт: x3 - x0 = 2
    dsu.unite(3, 0, 2);

    // Запит: x2 - x3 = ?
    auto [ok2, diff2] = dsu.query(2, 3);
    if (ok2) cout << "x2 - x3 = " << diff2 << endl;  // 6

    // Суперечливий факт: x2 - x0 = 10 (вже знаємо, що = 8)
    cout << "x2 - x0 = 10: "
         << (dsu.unite(2, 0, 10) ? "OK" : "Суперечність!") << endl;

    return 0;
}
```

**Вивід:**

```text
x1 - x0 = 5: OK
x2 - x1 = 3: OK
x2 - x0 = 8
x2 - x3 = 6
x2 - x0 = 10: Суперечність!
```

### 6.2 DSU з відкатом (Rollback DSU)

Звичайний DSU **незворотний** — після стиснення шляху не можна повернути попередній стан. **Rollback DSU** дозволяє скасовувати операції, що корисно для offline-алгоритмів та задач на дереві відрізків.

**Обмеження:** не можна використовувати Path Compression (бо вона змінює структуру незворотно). Використовуємо **лише Union by Rank/Size**.

**Ідея:** зберігаємо **історію змін** у стеку. Операція `rollback()` відновлює попередній стан.

```cpp
#include <iostream>
#include <vector>
#include <stack>
using namespace std;

class RollbackDSU {
    vector<int> parent, rank_;
    int comp;

    // Стек для відкату: зберігаємо {вершина, старий_батько, старий_ранг}
    struct Change {
        int vertex, old_parent, old_rank;
        int old_comp;
    };
    stack<Change> history;

public:
    RollbackDSU(int n) : parent(n), rank_(n, 0), comp(n) {
        for (int i = 0; i < n; i++) parent[i] = i;
    }

    // Find БЕЗ стиснення шляху (інакше відкат неможливий)
    int find(int x) {
        while (x != parent[x])
            x = parent[x];
        return x;
    }

    // Union зі збереженням історії
    bool unite(int a, int b) {
        int ra = find(a), rb = find(b);
        if (ra == rb) return false;

        // Зберігаємо стан перед зміною
        if (rank_[ra] < rank_[rb]) swap(ra, rb);
        history.push({rb, parent[rb], rank_[ra], comp});

        parent[rb] = ra;
        if (rank_[ra] == rank_[rb]) rank_[ra]++;
        comp--;

        return true;
    }

    // Скасувати останню операцію unite
    void rollback() {
        if (history.empty()) return;

        auto [vertex, old_parent, old_rank, old_comp] = history.top();
        history.pop();

        parent[vertex] = old_parent;
        rank_[find(vertex)] = old_rank;  // відновлюємо ранг батька
        comp = old_comp;
    }

    bool connected(int a, int b) {
        return find(a) == find(b);
    }

    int components() { return comp; }

    // Зберегти поточний "checkpoint" (кількість операцій)
    int save() { return history.size(); }

    // Відкатити до збереженого checkpoint
    void rollback_to(int checkpoint) {
        while ((int)history.size() > checkpoint)
            rollback();
    }
};

int main() {
    RollbackDSU dsu(5);

    int checkpoint = dsu.save();  // зберігаємо стан

    dsu.unite(0, 1);
    dsu.unite(2, 3);
    dsu.unite(0, 2);

    cout << "Після об'єднань: " << dsu.components() << " компонент" << endl;
    cout << "0 і 3 зв'язані? " << (dsu.connected(0, 3) ? "Так" : "Ні") << endl;

    // Відкатуємо до checkpoint (скасовуємо всі 3 unite)
    dsu.rollback_to(checkpoint);

    cout << "\nПісля відкату: " << dsu.components() << " компонент" << endl;
    cout << "0 і 3 зв'язані? " << (dsu.connected(0, 3) ? "Так" : "Ні") << endl;

    return 0;
}
```

**Вивід:**

```text
Після об'єднань: 2 компонент
0 і 3 зв'язані? Так

Після відкату: 5 компонент
0 і 3 зв'язані? Ні
```

**Складність Rollback DSU:**

| Операція | Час |
|----------|-----|
| Find | O(log n) — без стиснення |
| Union | O(log n) |
| Rollback | O(1) |

### 6.3 DSU з перерахуванням елементів множини

Іноді потрібно не тільки перевірити належність, а й **перерахувати всі елементи** певної множини. Для цього зберігаємо зв'язний список елементів у кожній компоненті.

**Оптимізація (small-to-large / маленький до великого):** при об'єднанні переносимо елементи **меншої** множини до **більшої**. Кожен елемент переноситься не більше O(log n) разів (бо після кожного перенесення розмір його множини щонайменше подвоюється).

```cpp
#include <iostream>
#include <vector>
#include <list>
using namespace std;

class DSU_Enumerate {
    vector<int> parent, size_;
    vector<list<int>> members;  // список елементів кожної компоненти

public:
    DSU_Enumerate(int n) : parent(n), size_(n, 1), members(n) {
        for (int i = 0; i < n; i++) {
            parent[i] = i;
            members[i].push_back(i);
        }
    }

    int find(int x) {
        if (x != parent[x]) parent[x] = find(parent[x]);
        return parent[x];
    }

    void unite(int a, int b) {
        int ra = find(a), rb = find(b);
        if (ra == rb) return;

        // Менша множина переноситься до більшої
        if (size_[ra] < size_[rb]) swap(ra, rb);

        // Переносимо елементи rb → ra
        for (int elem : members[rb])
            members[ra].push_back(elem);

        parent[rb] = ra;
        size_[ra] += size_[rb];
        members[rb].clear();
    }

    // Повертає всі елементи множини, що містить x
    const list<int>& get_members(int x) {
        return members[find(x)];
    }
};

int main() {
    DSU_Enumerate dsu(6);

    dsu.unite(0, 1);
    dsu.unite(2, 3);
    dsu.unite(0, 2);

    cout << "Множина елемента 3: { ";
    for (int elem : dsu.get_members(3))
        cout << elem << " ";
    cout << "}" << endl;

    return 0;
}
```

**Вивід:**

```text
Множина елемента 3: { 0 1 2 3 }
```

**Складність:** O(n log n) сумарно для всіх переносів (кожен елемент переноситься ≤ log n разів).

---

## 7. Застосування

### 7.1 Визначення циклів у неорієнтованому графі

Якщо при додаванні ребра (u, v) вершини u та v вже знаходяться в одній компоненті — це означає, що ребро створює **цикл**.

```cpp
#include <iostream>
#include <vector>
using namespace std;

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

int main() {
    int n = 5;  // вершини: 0..4
    vector<pair<int,int>> edges = {
        {0, 1}, {1, 2}, {2, 3}, {3, 4}, {4, 1}  // останнє ребро утворює цикл
    };

    DSU dsu(n);
    for (auto& [u, v] : edges) {
        if (!dsu.unite(u, v)) {
            cout << "Ребро (" << u << ", " << v << ") утворює цикл!" << endl;
        } else {
            cout << "Ребро (" << u << ", " << v << ") додано" << endl;
        }
    }

    return 0;
}
```

**Вивід:**

```text
Ребро (0, 1) додано
Ребро (1, 2) додано
Ребро (2, 3) додано
Ребро (3, 4) додано
Ребро (4, 1) утворює цикл!
```

### 7.2 Алгоритм Крускала (мінімальне кістякове дерево)

Алгоритм Крускала будує **мінімальне кістякове дерево** (MST — Minimum Spanning Tree) жадібним методом:

1. Відсортувати всі ребра за вагою
2. Для кожного ребра (від найлегшого): додати до MST, якщо воно не створює цикл

Union-Find ідеально підходить для кроку 2 — перевірка циклу за O(α(n)).

```
Граф:
    0 ---4--- 1
    |       / |
    2     3   6
    |   /     |
    3 ---5--- 4

Ребра (вага): (0,1,4), (0,3,2), (1,3,3), (1,4,6), (3,4,5)
Відсортовані: (0,3,2), (1,3,3), (0,1,4), (3,4,5), (1,4,6)
```

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

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
    int n = 5;  // вершини
    vector<Edge> edges = {
        {0, 1, 4}, {0, 3, 2}, {1, 3, 3},
        {1, 4, 6}, {3, 4, 5}, {0, 2, 1},
        {2, 3, 8}
    };

    // Крок 1: сортуємо ребра за вагою
    sort(edges.begin(), edges.end(), [](const Edge& a, const Edge& b) {
        return a.weight < b.weight;
    });

    // Крок 2: жадібно додаємо ребра
    DSU dsu(n);
    int mst_weight = 0;
    vector<Edge> mst_edges;

    for (auto& e : edges) {
        if (dsu.unite(e.u, e.v)) {
            mst_weight += e.weight;
            mst_edges.push_back(e);
            cout << "Додано ребро (" << e.u << ", " << e.v
                 << ") вага = " << e.weight << endl;
        } else {
            cout << "Пропущено ребро (" << e.u << ", " << e.v
                 << ") — створює цикл" << endl;
        }

        // MST має рівно n-1 ребер
        if (mst_edges.size() == n - 1) break;
    }

    cout << "\nВага MST: " << mst_weight << endl;
    return 0;
}
```

**Вивід:**

```text
Додано ребро (0, 2) вага = 1
Додано ребро (0, 3) вага = 2
Додано ребро (1, 3) вага = 3
Додано ребро (3, 4) вага = 5

Вага MST: 11
```

**Складність алгоритму Крускала:**
- Сортування ребер: O(E log E)
- Перевірка та об'єднання: O(E · α(V))
- **Загалом: O(E log E)**

### 7.3 Кількість зв'язних компонент

Класична задача: дано n вершин і m ребер, знайти кількість зв'язних компонент.

```cpp
int main() {
    int n = 8;  // вершини: 0..7
    vector<pair<int,int>> edges = {
        {0, 1}, {1, 2},    // компонента 1: {0, 1, 2}
        {3, 4},             // компонента 2: {3, 4}
        {5, 6}, {6, 7}     // компонента 3: {5, 6, 7}
    };

    DSU dsu(n);
    for (auto& [u, v] : edges)
        dsu.unite(u, v);

    cout << "Кількість компонент: " << dsu.count_components() << endl;
    // Виведе: 3
    return 0;
}
```

### 7.4 Задача: мінімальна кількість ребер для зв'язності

Дано n вершин та ребра. Скільки ребер потрібно додати, щоб граф став зв'язним?

**Відповідь:** `count_components() - 1`, бо кожне нове ребро може з'єднати дві компоненти.

```cpp
int min_edges_to_connect(int n, vector<pair<int,int>>& edges) {
    DSU dsu(n);
    for (auto& [u, v] : edges)
        dsu.unite(u, v);
    return dsu.count_components() - 1;
}
```

### 7.5 Задача: найбільша компонента

```cpp
int largest_component(int n, vector<pair<int,int>>& edges) {
    DSU_Size dsu(n);
    for (auto& [u, v] : edges)
        dsu.unite(u, v);

    int max_size = 0;
    for (int i = 0; i < n; i++)
        max_size = max(max_size, dsu.component_size(i));

    return max_size;
}
```

---

## 8. Розбір задач

### Задача 1: «Друзі друзів»

> Є n людей (0..n-1). Дано список пар друзів. Два людини належать до одного кола спілкування, якщо вони друзі, або мають спільного друга (транзитивно). Знайти кількість кіл спілкування та розмір найбільшого.

```cpp
#include <iostream>
#include <vector>
using namespace std;

class DSU {
    vector<int> parent, size_;
    int comp;
public:
    DSU(int n) : parent(n), size_(n, 1), comp(n) {
        for (int i = 0; i < n; i++) parent[i] = i;
    }
    int find(int x) {
        if (x != parent[x]) parent[x] = find(parent[x]);
        return parent[x];
    }
    bool unite(int a, int b) {
        int ra = find(a), rb = find(b);
        if (ra == rb) return false;
        if (size_[ra] < size_[rb]) swap(ra, rb);
        parent[rb] = ra;
        size_[ra] += size_[rb];
        comp--;
        return true;
    }
    int components() { return comp; }
    int component_size(int x) { return size_[find(x)]; }
};

int main() {
    int n = 8;
    vector<pair<int,int>> friendships = {
        {0, 1}, {1, 2},        // коло 1: {0, 1, 2}
        {3, 4}, {4, 5}, {3, 5},// коло 2: {3, 4, 5}
        {6, 7}                 // коло 3: {6, 7}
    };

    DSU dsu(n);
    for (auto& [a, b] : friendships)
        dsu.unite(a, b);

    cout << "Кількість кіл спілкування: " << dsu.components() << endl;

    int max_circle = 0;
    for (int i = 0; i < n; i++)
        max_circle = max(max_circle, dsu.component_size(i));

    cout << "Найбільше коло: " << max_circle << " людей" << endl;

    return 0;
}
```

**Вивід:**

```text
Кількість кіл спілкування: 3
Найбільше коло: 3 людей
```

### Задача 2: «Острови на сітці»

> Дано матрицю m×n з '1' (суша) та '0' (вода). Порахувати кількість островів. Острів — це група з'єднаних по горизонталі/вертикалі клітинок суші.

```cpp
#include <iostream>
#include <vector>
using namespace std;

class DSU {
    vector<int> parent, rank_;
    int comp;
public:
    DSU(int n) : parent(n), rank_(n, 0), comp(0) {
        for (int i = 0; i < n; i++) parent[i] = i;
    }
    void add_component() { comp++; }
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
        comp--;
        return true;
    }
    int components() { return comp; }
};

int count_islands(vector<vector<char>>& grid) {
    int m = grid.size(), n = grid[0].size();
    DSU dsu(m * n);

    // Додаємо кожну клітинку суші як окрему компоненту
    for (int i = 0; i < m; i++)
        for (int j = 0; j < n; j++)
            if (grid[i][j] == '1')
                dsu.add_component();

    // Об'єднуємо сусідні клітинки суші
    int dx[] = {0, 1};  // право та вниз
    int dy[] = {1, 0};
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) {
            if (grid[i][j] == '0') continue;
            for (int d = 0; d < 2; d++) {
                int ni = i + dx[d], nj = j + dy[d];
                if (ni < m && nj < n && grid[ni][nj] == '1')
                    dsu.unite(i * n + j, ni * n + nj);
            }
        }
    }

    return dsu.components();
}

int main() {
    vector<vector<char>> grid = {
        {'1', '1', '0', '0', '0'},
        {'1', '1', '0', '0', '0'},
        {'0', '0', '1', '0', '0'},
        {'0', '0', '0', '1', '1'}
    };

    cout << "Кількість островів: " << count_islands(grid) << endl;
    // Виведе: 3
    return 0;
}
```

**Вивід:**

```text
Кількість островів: 3
```

---

## 9. Підсумки

### Порівняння підходів

| Підхід | Find | Union | Пам'ять |
|--------|------|-------|---------|
| Quick-Find | O(1) | O(n) | O(n) |
| Quick-Union | O(n) | O(n) | O(n) |
| Union by Rank | O(log n) | O(log n) | O(n) |
| Path Compression | O(log n)* | O(log n)* | O(n) |
| Rank + Compression | **O(α(n))** | **O(α(n))** | O(n) |

*\* амортизовано*

### Коли використовувати Union-Find?

**Використовуйте DSU, коли:**
- Потрібно динамічно об'єднувати множини та перевіряти належність
- Будуєте MST (алгоритм Крускала)
- Шукаєте зв'язні компоненти
- Потрібно визначити цикли в неорієнтованому графі
- Задача про еквівалентність / транзитивне замикання

**Не використовуйте DSU, коли:**
- Потрібно розділяти множини (DSU не підтримує ефективне розділення)
- Потрібно перераховувати всі елементи множини
- Краще підходить BFS/DFS (наприклад, пошук найкоротшого шляху)

### Ключові тези

1. **Union-Find** — структура даних для роботи з непересічними множинами
2. Дві ключові оптимізації: **Union by Rank** та **Path Compression**
3. Амортизована складність — **O(α(n)) ≈ O(1)** на практиці
4. Основне застосування: зв'язність, цикли, MST (Крускал)
5. Реалізація проста (~20 рядків), але ефективність вражаюча

---

## Додаткові матеріали

- [Візуалізація Union-Find](https://visualgo.net/en/ufds)
- [CP-Algorithms: Disjoint Set Union](https://cp-algorithms.com/data_structures/disjoint_set_union.html)
- Cormen et al., "Introduction to Algorithms" — Chapter 21: Data Structures for Disjoint Sets
