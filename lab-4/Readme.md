# Lab 4

## Проєкт «Сума натуральних чисел»

У цьому завданні реалізовано рекурсивний алгоритм, що обчислює суму всіх
натуральних чисел від 1 до `N`. Використання шаблонної функції `sumN`
дозволяє працювати як з типом `int`, так і з довшими цілими значеннями без
дублювання логіки.

### Приклад `main`

```cpp
#include <iostream>
using namespace std;

template <typename T>
T sumN(T n) {
    if (n == 0) return 0;  // Базовий випадок
    return n + sumN(n - 1);
}

int main() {
    // Для цілих чисел
    int intN;
    cout << "Введіть ціле число N: ";
    cin >> intN;
    cout << "Сума (int): " << sumN(intN) << endl;

    // Для довгих цілих
    long long longN;
    cout << "\nВведіть велике число N: ";
    cin >> longN;
    cout << "Сума (long long): " << sumN(longN) << endl;

    return 0;
}
```

### Приклад запуску

```text
Введіть ціле число N: 5
Сума (int): 15

Введіть велике число N: 1000
Сума (long long): 500500
```

> **Порада:** рекурсивні виклики зменшують значення `n`, доки не буде
> досягнуто базового випадку `n == 0`. Переконайтеся, що вхідні значення не
> призводять до переповнення типу.

### Примітка про лямбда-функції

Лямбда-вислови (`[](...) { ... }`) у C++ дають змогу оголошувати невеликі
функції безпосередньо в місці виклику. Це зручно, коли компаратор або інша
допоміжна логіка потрібні один раз, але не варто засмічувати код окремими
глобальними функціями. У деяких завданнях цієї лабораторної ми свідомо
використовуємо звичайні функції-порівнювачі замість лямбд, аби показати, що
передавання поведінки через вказівники на функції працює однаково й без
додаткового синтаксису. Якщо ви впевнено почуваєтесь із лямбдами, їх можна
підставити в ті самі місця:

```cpp
auto int_less = [](const int &lhs, const int &rhs) {
    return lhs < rhs;
};
merge_sort(data, 0, n - 1, int_less);
```

Найважливіше — зберігати сигнатуру `bool(const T &, const T &)`. Ось ще кілька
простих прикладів лямбда-функцій:

```cpp
// 1. Миттєвий розрахунок
cout << [](int a, int b) { return a + b; }(4, 6) << endl;  // 10

// 2. Обхід контейнера
vector<int> nums{1, 2, 3};
for_each(nums.begin(), nums.end(), [](int value) { cout << value << ' '; });

// 3. Захоплення змінних
int total = 0;
auto add_to_total = [&total](int value) { total += value; };
add_to_total(5);
add_to_total(7);
cout << total << endl;  // 12

// 4. Порівняння рядків без урахування регістру
auto case_insensitive = [](const string &a, const string &b) {
    string lower_a = a, lower_b = b;
    transform(lower_a.begin(), lower_a.end(), lower_a.begin(), ::tolower);
    transform(lower_b.begin(), lower_b.end(), lower_b.begin(), ::tolower);
    return lower_a < lower_b;
};
merge_sort(words, 0, n_words - 1, case_insensitive);
```

### Лямбда всередині функцій і методів

Лямбда можна оголошувати безпосередньо
в тілі функції чи методу.
Захоплення `[&]`, `[=]` або `[this]`
визначає, які змінні будуть доступні
у тiлi лямбди. Приклад нижче показує
локальні обчислення без виносу
допоміжних функцій у простір імен класу:

```cpp
class GradesBook {
public:
    double average() const {
        auto accumulate_weighted = [this]() {
            double total = 0.0;
            for (const auto &grade : grades_) {
                total += grade.value * grade.weight;
            }
            return total;
        };

        auto total_weight = [](const vector<Grade> &grades) {
            double sum = 0.0;
            for (const auto &grade : grades) {
                sum += grade.weight;
            }
            return sum;
        };

        double weight = total_weight(grades_);
        return weight == 0.0 ? 0.0 : accumulate_weighted() / weight;
    }

private:
    struct Grade {
        double value;
        double weight;
    };
    vector<Grade> grades_;
};
```

Лямбда `accumulate_weighted` використовує `[this]`,
тому має доступ до полів класу.
`total_weight` не потребує доступу до стану об'єкта,
тож передаємо вектор оцінок аргументом.
Такий підхід робить код компактним
і тримає допоміжну логіку поруч із місцем використання.
