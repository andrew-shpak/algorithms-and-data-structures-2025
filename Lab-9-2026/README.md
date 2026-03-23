# Lab-9-2026 — Збалансовані дерева пошуку (AVL)

## [IDE](https://onecompiler.com/cpp)

---

## Завдання 1: AVL-дерево з перевіркою збалансованості

### Мета

Реалізувати AVL-дерево з автоматичним балансуванням при вставці та видаленні елементів. Створити функцію перевірки AVL-властивості (різниця висот лівого та правого піддерев не перевищує 1 для кожного вузла).

- **Складність:**
  - Вставка: O(log n) завдяки збалансованості
  - Видалення: O(log n) з перебалансуванням
  - Перевірка збалансованості: O(n)

### Підказка

AVL-балансування потребує чотирьох типів обертань:
- **Ліве обертання** — коли правий нащадок важчий (Right-Right випадок)
- **Праве обертання** — коли лівий нащадок важчий (Left-Left випадок)
- **Ліве-праве обертання** — спочатку ліве обертання лівого нащадка, потім праве обертання вузла (Left-Right випадок)
- **Право-ліве обертання** — спочатку праве обертання правого нащадка, потім ліве обертання вузла (Right-Left випадок)

Баланс-фактор вузла = висота(лівий) - висота(правий). Якщо |баланс-фактор| > 1, потрібне обертання.

Видалення з AVL = BST-видалення + балансування на зворотному шляху рекурсії. Три випадки видалення: листок, один нащадок, два нащадки (заміна на inorder-наступника). Після видалення може знадобитися балансування кількох вузлів на шляху до кореня.

### Приклад `main`

```cpp
#include <iostream>
#include <cmath>
using namespace std;

struct Node {
    int key;
    Node* left;
    Node* right;
    int height;

    Node(int k) : key(k), left(nullptr), right(nullptr), height(1) {}
};

class AVLTree {
private:
    Node* root;

    // Отримати висоту вузла
    int get_height(Node* node) {
        // Ваш код тут
    }

    // Обчислити баланс-фактор вузла
    int get_balance(Node* node) {
        // Ваш код тут
    }

    // Праве обертання
    Node* rotate_right(Node* y) {
        // Ваш код тут
    }

    // Ліве обертання
    Node* rotate_left(Node* x) {
        // Ваш код тут
    }

    // Вставка з балансуванням
    Node* insert_helper(Node* node, int key) {
        // Ваш код тут
    }

    // Перевірка збалансованості
    bool is_balanced_helper(Node* node, int& height) {
        // Ваш код тут
    }

    // Знайти вузол з мінімальним ключем
    Node* find_min(Node* node) {
        // Ваш код тут
    }

    // Видалення з перебалансуванням
    Node* delete_helper(Node* node, int key) {
        // Ваш код тут
    }

    // Інфіксний обхід
    void inorder_helper(Node* node) {
        if (!node) return;
        inorder_helper(node->left);
        cout << node->key << " ";
        inorder_helper(node->right);
    }

    // Вивід дерева
    void print_helper(Node* node, string indent, bool last) {
        // Ваш код тут
    }

public:
    AVLTree() : root(nullptr) {}

    void insert(int key) {
        root = insert_helper(root, key);
    }

    void remove(int key) {
        root = delete_helper(root, key);
    }

    void inorder() {
        inorder_helper(root);
    }

    bool is_balanced() {
        int height = 0;
        return is_balanced_helper(root, height);
    }

    int height() {
        return get_height(root);
    }

    void print_tree() {
        // Ваш код тут
    }
};

int main() {
    AVLTree tree;

    cout << "=== ВСТАВКА ЕЛЕМЕНТІВ ===" << endl;
    int values[] = {10, 20, 30, 40, 50, 25};

    for (int val : values) {
        tree.insert(val);
        cout << "Вставлено: " << val << endl;
        cout << "Висота дерева: " << tree.height() << endl;
        cout << "Збалансоване: " << (tree.is_balanced() ? "Так" : "Ні") << endl;
        cout << endl;
    }

    cout << "=== СТРУКТУРА ДЕРЕВА ===" << endl;
    tree.print_tree();

    cout << "\n=== ФІНАЛЬНА ПЕРЕВІРКА ===" << endl;
    cout << "Загальна висота: " << tree.height() << endl;
    cout << "Дерево збалансоване: " << (tree.is_balanced() ? "Так" : "Ні") << endl;

    cout << "\n=== ВИДАЛЕННЯ ЕЛЕМЕНТІВ ===" << endl;
    cout << "Inorder до видалень: ";
    tree.inorder();
    cout << endl;

    // Видалення листка
    tree.remove(50);
    cout << "Після видалення 50 (листок): ";
    tree.inorder();
    cout << "| Збалансоване: " << (tree.is_balanced() ? "Так" : "Ні") << endl;

    // Видалення вузла з одним нащадком
    tree.remove(40);
    cout << "Після видалення 40 (один нащадок): ";
    tree.inorder();
    cout << "| Збалансоване: " << (tree.is_balanced() ? "Так" : "Ні") << endl;

    // Видалення вузла з двома нащадками
    tree.remove(25);
    cout << "Після видалення 25 (два нащадки): ";
    tree.inorder();
    cout << "| Збалансоване: " << (tree.is_balanced() ? "Так" : "Ні") << endl;

    return 0;
}
```

### Приклад запуску

```text
=== ВСТАВКА ЕЛЕМЕНТІВ ===
Вставлено: 10
Висота дерева: 1
Збалансоване: Так

Вставлено: 20
Висота дерева: 2
Збалансоване: Так

Вставлено: 30
Висота дерева: 2
Збалансоване: Так

Вставлено: 40
Висота дерева: 3
Збалансоване: Так

Вставлено: 50
Висота дерева: 3
Збалансоване: Так

Вставлено: 25
Висота дерева: 3
Збалансоване: Так

=== СТРУКТУРА ДЕРЕВА ===
        ┌── 50
    ┌── 40
    │   └── 30
┌── 25
│   └── 20
10

=== ФІНАЛЬНА ПЕРЕВІРКА ===
Загальна висота: 3
Дерево збалансоване: Так

=== ВИДАЛЕННЯ ЕЛЕМЕНТІВ ===
Inorder до видалень: 10 20 25 30 40 50
Після видалення 50 (листок): 10 20 25 30 40 | Збалансоване: Так
Після видалення 40 (один нащадок): 10 20 25 30 | Збалансоване: Так
Після видалення 25 (два нащадки): 10 20 30 | Збалансоване: Так
```

---

## Завдання 2: K-й найменший елемент (Order Statistics Tree)

### Мета

Розширити AVL-дерево з Завдання 1, додавши до кожного вузла поле `size` — кількість вузлів у його піддереві (включаючи сам вузол). Використовуючи це поле, реалізувати ефективний пошук k-го найменшого елемента за O(log n).

- **Складність:** O(log n) для пошуку k-го елемента

### Підказка

Для кожного вузла `size = 1 + size(лівий) + size(правий)`. Щоб знайти k-й найменший:
- Нехай `left_size` = кількість вузлів у лівому піддереві
- Якщо `k == left_size + 1` — поточний вузол і є відповідь
- Якщо `k <= left_size` — шукаємо в лівому піддереві
- Якщо `k > left_size + 1` — шукаємо в правому піддереві з `k = k - left_size - 1`

Не забудьте оновлювати `size` після вставки та обертань.

### Приклад `main`

```cpp
#include <iostream>
using namespace std;

struct Node {
    int key;
    Node* left;
    Node* right;
    int height;
    int size;  // кількість вузлів у піддереві

    Node(int k) : key(k), left(nullptr), right(nullptr), height(1), size(1) {}
};

class OrderStatTree {
private:
    Node* root;

    int get_height(Node* node) {
        return node ? node->height : 0;
    }

    int get_size(Node* node) {
        return node ? node->size : 0;
    }

    // Оновити висоту та розмір вузла
    void update(Node* node) {
        // Ваш код тут
    }

    int get_balance(Node* node) {
        return node ? get_height(node->left) - get_height(node->right) : 0;
    }

    // Праве обертання (не забудьте оновити size!)
    Node* rotate_right(Node* y) {
        // Ваш код тут
    }

    // Ліве обертання (не забудьте оновити size!)
    Node* rotate_left(Node* x) {
        // Ваш код тут
    }

    Node* insert_helper(Node* node, int key) {
        // Ваш код тут (аналогічно до Завдання 1, але з оновленням size)
    }

    // Пошук k-го найменшого елемента
    int kth_smallest_helper(Node* node, int k) {
        // Ваш код тут
    }

public:
    OrderStatTree() : root(nullptr) {}

    void insert(int key) {
        root = insert_helper(root, key);
    }

    // Повертає k-й найменший елемент (1-індексація)
    int kth_smallest(int k) {
        if (k < 1 || k > get_size(root)) {
            cout << "k = " << k << " поза межами [1, " << get_size(root) << "]" << endl;
            return -1;
        }
        return kth_smallest_helper(root, k);
    }

    int size() {
        return get_size(root);
    }
};

int main() {
    OrderStatTree tree;

    cout << "=== ПОБУДОВА ДЕРЕВА ===" << endl;
    for (int v : {50, 30, 70, 20, 40, 60, 80, 10, 25, 35})
        tree.insert(v);

    cout << "Кількість елементів: " << tree.size() << endl;
    // Відсортований порядок: 10 20 25 30 35 40 50 60 70 80

    cout << "\n=== ПОШУК K-ГО НАЙМЕНШОГО ===" << endl;
    for (int k = 1; k <= tree.size(); ++k) {
        cout << k << "-й найменший: " << tree.kth_smallest(k) << endl;
    }

    cout << "\n=== ГРАНИЧНІ ВИПАДКИ ===" << endl;
    tree.kth_smallest(0);   // поза межами
    tree.kth_smallest(11);  // поза межами

    return 0;
}
```

### Приклад запуску

```text
=== ПОБУДОВА ДЕРЕВА ===
Кількість елементів: 10

=== ПОШУК K-ГО НАЙМЕНШОГО ===
1-й найменший: 10
2-й найменший: 20
3-й найменший: 25
4-й найменший: 30
5-й найменший: 35
6-й найменший: 40
7-й найменший: 50
8-й найменший: 60
9-й найменший: 70
10-й найменший: 80

=== ГРАНИЧНІ ВИПАДКИ ===
k = 0 поза межами [1, 10]
k = 11 поза межами [1, 10]
```

---

## Завдання

| # | Тема |
|---|------|
| Завдання 1 | AVL-дерево з перевіркою збалансованості |
| Завдання 2 | K-й найменший елемент (Order Statistics Tree) |
