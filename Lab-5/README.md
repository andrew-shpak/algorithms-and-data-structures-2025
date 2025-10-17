# Lab 5

## Завдання 1: Швидке сортування (Quick Sort)

### Мета
Реалізувати алгоритм швидкого сортування для цілих чисел. Застосувати рекурсивний підхід із розбиттям масиву навколо опорного елемента.

### Вимоги
- Створіть функцію `quick_sort(data, left, right)` для сортування цілих чисел.
- Обробіть базовий випадок `left >= right`.
- Продемонструйте роботу на прикладі масиву цілих чисел.
- **Бали:** 3.
- **Складність:** середня O(n log n), найгірша O(n²) при поганому виборі опорного елемента.

### Приклад `main`

```cpp
#include <iostream>
#include <vector>
using namespace std;

void quick_sort(vector<int> &arr, int left, int right) {

}

int main() {
    vector<int> arr = {5, 2, 9, 1, 7, 3};

    cout << "Початковий масив: ";
    for (int value : arr) {
        cout << value << ' ';
    }
    cout << endl;

    quick_sort(arr, 0, arr.size() - 1);

    cout << "Відсортований масив: ";
    for (int value : arr) {
        cout << value << ' ';
    }
    cout << endl;

    return 0;
}
```

### Приклад запуску

```text
Початковий масив: 5 2 9 1 7 3
Відсортований масив: 1 2 3 5 7 9
```
 

---

## Завдання 2: Сортування вставками (Insertion Sort)

### Мета
Реалізувати простий і ефективний алгоритм сортування вставками для цілих чисел. Цей алгоритм добре працює на невеликих або майже відсортованих масивах.

### Вимоги
- Створіть функцію `insertion_sort(arr)` для сортування цілих чисел.
- Реалізуйте алгоритм вставками зі зсувом елементів.
- Продемонструйте роботу на прикладі масиву цілих чисел.
- **Бали:** 2.
- **Складність:** O(n²) в середньому випадку, O(n) для майже відсортованих масивів.
- **Переваги:** стабільне сортування, працює "на місці", ефективне для малих даних.

### Приклад `main`

```cpp
#include <iostream>
#include <vector>
using namespace std;

void insertion_sort(vector<int> &arr) {
}

int main() {
    vector<int> arr = {5, 2, 9, 1, 3};

    cout << "Початковий масив: ";
    for (int value : arr) {
        cout << value << ' ';
    }
    cout << endl;

    insertion_sort(arr);

    cout << "Відсортований масив: ";
    for (int value : arr) {
        cout << value << ' ';
    }
    cout << endl;

    return 0;
}
```

### Приклад запуску

```text
Початковий масив: 5 2 9 1 3
Відсортований масив: 1 2 3 5 9
``` 

---

## Завдання 3: Перевірка збалансованих дужок (Balanced Parentheses)

### Мета
Реалізувати функцію для перевірки збалансованості дужок `()` у рядку. Навчитись використовувати структуру даних stack для розв'язання практичних задач.

### Вимоги
- Створіть функцію `is_balanced(string s)` що повертає `bool`.
- **Бали:** 2.
- **Складність:** O(n) час, O(n) пам'ять для стека.

### Приклад `main`

```cpp
#include <iostream>
#include <string>
#include <stack>
using namespace std;

bool is_balanced(string s) {
}

int main() {
    // Тестові випадки з очікуваними результатами
    vector<pair<string, bool>> test_cases = {
        {"()", true},           // збалансовано
        {"(())", true},         // вкладені дужки
        {"()()", true},         // послідовні пари
        {"((()))", true},       // множинні вкладення
        {"(()", false},         // не вистачає закриваючої
        {")(", false},          // неправильний порядок
        {"())", false},         // зайва закриваюча
        {"(((", false}          // тільки відкриваючі
    };

    cout << "Перевірка збалансованості дужок:\n" << endl;

    for (const auto &test : test_cases) {
        string input = test.first;
        bool expected = test.second;
        bool result = is_balanced(input);

        cout << "\"" << input << "\" -> " << (result ? "true" : "false");
        cout << " (очікується: " << (expected ? "true" : "false") << ")";

        if (result == expected) {
            cout << " ✓" << endl;
        } else {
            cout << " ✗" << endl;
        }
    }

    return 0;
}
```

### Приклад запуску

```text
Перевірка збалансованості дужок:

"()" -> true (очікується: true) ✓
"(())" -> true (очікується: true) ✓
"()()" -> true (очікується: true) ✓
"((()))" -> true (очікується: true) ✓
"(()" -> false (очікується: false) ✓
")(" -> false (очікується: false) ✓
"())" -> false (очікується: false) ✓
"(((" -> false (очікується: false) ✓
```

---
