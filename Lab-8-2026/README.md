# Lab-8-2026 — Бінарні дерева

## [IDE](https://onecompiler.com/cpp)

---

## Завдання 1: Обходи дерева

### Мета

Реалізувати три основні рекурсивні обходи бінарного дерева пошуку: інфіксний (inorder), префіксний (preorder) та постфіксний (postorder).

- **Бали:** 1
- **Складність:** O(n) для кожного обходу, де n — кількість вузлів

### Приклад `main`

```cpp
#include <iostream>
using namespace std;

struct Node {
    int data;
    Node* left;
    Node* right;
    Node(int val) : data(val), left(nullptr), right(nullptr) {}
};

Node* insert(Node* root, int val) {
    // Ваш код тут
}

void inorder(Node* root) {
    // Ваш код тут
}

void preorder(Node* root) {
    // Ваш код тут
}

void postorder(Node* root) {
    // Ваш код тут
}

int main() {
    Node* root = nullptr;
    for (int v : {50, 30, 70, 20, 40, 60, 80})
        root = insert(root, v);

    cout << "Інфіксний обхід (Inorder): ";
    inorder(root);
    cout << endl;

    cout << "Префіксний обхід (Preorder): ";
    preorder(root);
    cout << endl;

    cout << "Постфіксний обхід (Postorder): ";
    postorder(root);
    cout << endl;

    return 0;
}
```

### Приклад запуску

```text
Інфіксний обхід (Inorder): 20 30 40 50 60 70 80
Префіксний обхід (Preorder): 50 30 20 40 70 60 80
Постфіксний обхід (Postorder): 20 40 30 60 80 70 50
```

---

## Завдання 2: Діаметр бінарного дерева

### Мета

Обчислити діаметр бінарного дерева — найдовший шлях між будь-якими двома вузлами дерева, виміряний у кількості ребер. Цей шлях може не проходити через корінь.

- **Бали:** 1
- **Складність:** O(n), де n — кількість вузлів

### Підказка

Для кожного вузла найдовший шлях, що проходить через нього, дорівнює сумі висот його лівого та правого піддерев (+ 2 ребра до кожного нащадка). Діаметр дерева — максимум серед усіх таких шляхів. Задачу можна розв'язати за один прохід, обчислюючи висоту та діаметр одночасно.

### Приклад `main`

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

struct Node {
    int data;
    Node* left;
    Node* right;
    Node(int val) : data(val), left(nullptr), right(nullptr) {}
};

Node* insert(Node* root, int val) {
    // Ваш код тут
}

// Повертає діаметр дерева
// height — вихідний параметр для висоти піддерева
int diameter(Node* root, int& height) {
    // Ваш код тут
}

int main() {
    // Дерево 1 (збалансоване):
    //         50
    //        /  \
    //      30    70
    //     / \   / \
    //    20  40 60  80
    Node* root1 = nullptr;
    for (int v : {50, 30, 70, 20, 40, 60, 80})
        root1 = insert(root1, v);

    int h = 0;
    cout << "Дерево 1 (збалансоване):" << endl;
    cout << "Діаметр: " << diameter(root1, h) << endl;
    cout << endl;

    // Дерево 2 (діаметр НЕ проходить через корінь):
    //         50
    //        /
    //      30
    //     / \
    //    20   40
    //   /      \
    //  10       45
    Node* root2 = nullptr;
    for (int v : {50, 30, 20, 10, 40, 45})
        root2 = insert(root2, v);

    h = 0;
    cout << "Дерево 2 (діаметр не через корінь):" << endl;
    cout << "Діаметр: " << diameter(root2, h) << endl;

    return 0;
}
```

### Приклад запуску

```text
Дерево 1 (збалансоване):
Діаметр: 4

Дерево 2 (діаметр не через корінь):
Діаметр: 4
```

---

## Завдання 3: Перевірка коректності BST

### Мета

Реалізувати функцію перевірки, чи є бінарне дерево коректним деревом пошуку (BST). Наївна перевірка (порівняння вузла лише з прямими нащадками) є недостатньою — потрібно використовувати діапазонний підхід із мінімальними та максимальними допустимими значеннями.

- **Бали:** 2
- **Складність:** O(n), де n — кількість вузлів

### Приклад `main`

```cpp
#include <iostream>
#include <climits>
using namespace std;

struct Node {
    int data;
    Node* left;
    Node* right;
    Node(int val) : data(val), left(nullptr), right(nullptr) {}
};

// Перевірка BST з допустимим діапазоном [min_val, max_val]
bool is_valid_bst(Node* root, long long min_val, long long max_val) {
    // Ваш код тут
}

bool is_valid_bst(Node* root) {
    return is_valid_bst(root, LLONG_MIN, LLONG_MAX);
}

int main() {
    // Дерево 1: коректне BST
    //       50
    //      /  \
    //    30    70
    //   / \   / \
    //  20  40 60  80
    Node* t1 = new Node(50);
    t1->left = new Node(30);
    t1->right = new Node(70);
    t1->left->left = new Node(20);
    t1->left->right = new Node(40);
    t1->right->left = new Node(60);
    t1->right->right = new Node(80);

    cout << "Дерево 1: "
         << (is_valid_bst(t1) ? "Коректне BST" : "Не BST") << endl;

    // Дерево 2: НЕ коректне BST
    //       50
    //      /  \
    //    30    70
    //   / \
    //  20  60  <-- 60 > 50, але знаходиться в лівому піддереві кореня
    Node* t2 = new Node(50);
    t2->left = new Node(30);
    t2->right = new Node(70);
    t2->left->left = new Node(20);
    t2->left->right = new Node(60);

    cout << "Дерево 2: "
         << (is_valid_bst(t2) ? "Коректне BST" : "Не BST") << endl;

    // Дерево 3: НЕ коректне BST
    //       50
    //      /  \
    //    30    70
    //         / \
    //        40  80  <-- 40 < 50, але знаходиться в правому піддереві кореня
    Node* t3 = new Node(50);
    t3->left = new Node(30);
    t3->right = new Node(70);
    t3->right->left = new Node(40);
    t3->right->right = new Node(80);

    cout << "Дерево 3: "
         << (is_valid_bst(t3) ? "Коректне BST" : "Не BST") << endl;

    return 0;
}
```

### Приклад запуску

```text
Дерево 1: Коректне BST
Дерево 2: Не BST
Дерево 3: Не BST
```

---

## Завдання 4: Видалення вузла з BST

### Мета

Реалізувати операцію видалення вузла з бінарного дерева пошуку з урахуванням трьох випадків:

1. **Вузол-листок** — просто видалити
2. **Вузол з одним нащадком** — замінити вузол його нащадком
3. **Вузол з двома нащадками** — знайти inorder-наступника (найменший елемент у правому піддереві), скопіювати його значення та видалити наступника

- **Бали:** 2
- **Складність:** O(h), де h — висота дерева

### Приклад `main`

```cpp
#include <iostream>
using namespace std;

struct Node {
    int data;
    Node* left;
    Node* right;
    Node(int val) : data(val), left(nullptr), right(nullptr) {}
};

Node* insert(Node* root, int val) {
    // Ваш код тут
}

void inorder(Node* root) {
    // Ваш код тут
}

// Знаходить вузол з мінімальним значенням у дереві
Node* find_min(Node* root) {
    // Ваш код тут
}

// Видаляє вузол зі значенням key з дерева
Node* delete_node(Node* root, int key) {
    // Ваш код тут
}

int main() {
    Node* root = nullptr;
    for (int v : {50, 30, 70, 20, 40, 60, 80})
        root = insert(root, v);

    //         50
    //        /  \
    //      30    70
    //     / \   / \
    //    20  40 60  80

    cout << "Початкове дерево: ";
    inorder(root);
    cout << endl;

    // Випадок 1: видалення листка (20)
    root = delete_node(root, 20);
    cout << "Після видалення 20 (листок): ";
    inorder(root);
    cout << endl;

    // Випадок 2: видалення вузла з одним нащадком (30 → має лише 40)
    root = delete_node(root, 30);
    cout << "Після видалення 30 (один нащадок): ";
    inorder(root);
    cout << endl;

    // Випадок 3: видалення вузла з двома нащадками (50 → наступник 60)
    root = delete_node(root, 50);
    cout << "Після видалення 50 (два нащадки): ";
    inorder(root);
    cout << endl;

    return 0;
}
```

### Приклад запуску

```text
Початкове дерево: 20 30 40 50 60 70 80
Після видалення 20 (листок): 30 40 50 60 70 80
Після видалення 30 (один нащадок): 40 50 60 70 80
Після видалення 50 (два нащадки): 40 60 70 80
```

---

## Завдання 5: Серіалізація та десеріалізація BST

### Мета

Реалізувати функції для серіалізації бінарного дерева пошуку у рядок та його відновлення (десеріалізації) з рядка. Використовувати префіксний обхід (preorder) із маркером `#` для позначення відсутніх вузлів.

- **Бали:** 2
- **Складність:** O(n) для обох операцій, де n — кількість вузлів

### Підказка

Серіалізація: обхід дерева у префіксному порядку, записуючи значення вузлів через пробіл. Для `nullptr` записуйте `#`. Десеріалізація: зчитування токенів по одному з потоку та рекурсивна побудова дерева у тому ж порядку.

### Приклад `main`

```cpp
#include <iostream>
#include <sstream>
#include <string>
using namespace std;

struct Node {
    int data;
    Node* left;
    Node* right;
    Node(int val) : data(val), left(nullptr), right(nullptr) {}
};

Node* insert(Node* root, int val) {
    // Ваш код тут
}

void inorder(Node* root) {
    // Ваш код тут
}

// Серіалізує дерево у рядок (префіксний обхід, "#" для nullptr)
string serialize(Node* root) {
    // Ваш код тут
}

// Відновлює дерево з рядка
Node* deserialize(const string& data) {
    // Ваш код тут
}

int main() {
    Node* root = nullptr;
    for (int v : {50, 30, 70, 20, 40, 60, 80})
        root = insert(root, v);

    //         50
    //        /  \
    //      30    70
    //     / \   / \
    //    20  40 60  80

    cout << "Оригінальне дерево (inorder): ";
    inorder(root);
    cout << endl;

    string data = serialize(root);
    cout << "Серіалізовано: " << data << endl;

    Node* restored = deserialize(data);
    cout << "Відновлене дерево (inorder): ";
    inorder(restored);
    cout << endl;

    // Тест з порожнім деревом
    string empty_data = serialize(nullptr);
    cout << "Порожнє дерево серіалізовано: \"" << empty_data << "\"" << endl;

    Node* empty_restored = deserialize(empty_data);
    cout << "Відновлене порожнє дерево (inorder): ";
    inorder(empty_restored);
    cout << "(порожньо)" << endl;

    return 0;
}
```

### Приклад запуску

```text
Оригінальне дерево (inorder): 20 30 40 50 60 70 80
Серіалізовано: 50 30 20 # # 40 # # 70 60 # # 80 # #
Відновлене дерево (inorder): 20 30 40 50 60 70 80
Порожнє дерево серіалізовано: "#"
Відновлене порожнє дерево (inorder): (порожньо)
```

---

## Сумарні бали

| Завдання | Тема | Бали |
|----------|------|------|
| Завдання 1 | Обходи дерева | 1 |
| Завдання 2 | Діаметр бінарного дерева | 1 |
| Завдання 3 | Перевірка коректності BST | 2 |
| Завдання 4 | Видалення вузла з BST | 2 |
| Завдання 5 | Серіалізація та десеріалізація BST | 2 |

**Всього: 8 балів**
