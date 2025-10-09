# Завдання 1: Обчислення степеня

## Мета
Реалізувати функції піднесення числа до степеня для цілих та дробових типів,
забезпечуючи коректну обробку базових випадків і повторне використання логіки.

## Вимоги
- Реалізуйте функцію `power(base, exp)` і використайте її у функції `main`, як
  показано в прикладі.
- Забезпечте коректний результат для цілих і дробових чисел, передбачте
  випадок, коли `exp == 0`.
- Додайте діалог із користувачем (зчитування, вивід результатів), як у прикладі.
- **Примітка:** завдання оцінюється в 1 бал.
- **Пам'ять:** глибока рекурсія збільшує використання стека; для великих
  показників можна перейти на ітеративний варіант.


## Приклад `main`
```cpp
#include <iostream>
using namespace std;

T power(base, exp) {
}

int main() {
    // Для цілих чисел
    int intBase;
    int intExp;
    cout << "Введіть ціле число (основа): ";
    cin >> intBase;
    cout << "Введіть степінь: ";
    cin >> intExp;
    cout << "Результат (int): " << power(intBase, intExp) << endl;

    // Для дробових чисел
    double doubleBase;
    int doubleExp;
    cout << "\nВведіть дробове число (основа): ";
    cin >> doubleBase;
    cout << "Введіть степінь: ";
    cin >> doubleExp;
    cout << "Результат (double): " << power(doubleBase, doubleExp) << endl;

    return 0;
}
```

### Приклад запуску
```text
=== Приклад 1 ===
Введіть ціле число (основа): 2
Введіть степінь: 5
Результат (int): 32

Введіть дробове число (основа): 2.5
Введіть степінь: 3
Результат (double): 15.625

=== Приклад 2 ===
Введіть ціле число (основа): 7
Введіть степінь: 0
Результат (int): 1

Введіть дробове число (основа): 2.5
Введіть степінь: 3
Результат (double): 15.625
```

# Завдання 2: Сума елементів масиву

## Мета
Реалізувати універсальний алгоритм, що обчислює суму елементів масиву для
числових типів, використовуючи рекурсивний підхід.

## Вимоги
- Напишіть функцію `arraySum(arr, n)` і викличте її у `main`, як у наведеному
  прикладі.
- Підтримайте введення масивів для типів `int` та `double`, включно з випадком `n == 0`.
- Забезпечте виведення сум для обох масивів за зразком прикладу.
- **Примітка:** завдання оцінюється в 1 бали.
- **Пам'ять:** рекурсія споживає стек пропорційно `n`; для великих масивів
  дозволено перейти на ітеративний підрахунок.

## Приклад `main`
```cpp
#include <iostream>
#include <vector>
using namespace std;

T arraySum(arr, n) {
}

int main() {
    // Для цілих чисел
    int intN;
    cout << "Введіть розмір масиву (int): ";
    cin >> intN;
    vector<int> intArr(intN);
    cout << "Введіть " << intN << " цілих чисел: ";
    for (int i = 0; i < intN; i++) {
        cin >> intArr[i];
    }
    cout << "Сума (int): " << arraySum(intArr, intN) << endl;

    // Для дробових чисел
    int doubleN;
    cout << "\nВведіть розмір масиву (double): ";
    cin >> doubleN;

    vector<double> doubleArr(doubleN);
    cout << "Введіть " << doubleN << " дробових чисел: ";
    for (int i = 0; i < doubleN; i++) {
        cin >> doubleArr[i];
    }
    cout << "Сума (double): " << arraySum(doubleArr, doubleN) << endl;

    return 0;
}
```

### Приклад запуску
```text
=== Приклад 1 ===
Введіть розмір масиву (int): 5
Введіть 5 цілих чисел: 1 2 3 4 5
Сума (int): 15

Введіть розмір масиву (double): 3
Введіть 3 дробових чисел: 1.5 2.5 3.5
Сума (double): 7.5

=== Приклад 2 ===
Введіть розмір масиву (int): 0
Сума (int): 0

Введіть розмір масиву (double): 2
Введіть 2 дробових чисел: 0.0 0.5
Сума (double): 0.5
```
# Завдання 3: Знайти мінімум

## Мета
Написати універсальну функцію, що знаходить мінімальний елемент у послідовності
для різних типів даних, використовуючи рекурсивне порівняння.

## Вимоги
- Створіть функцію `findMin(arr, n)` і використайте її в `main`, як у прикладі.
- Реалізуйте введення/виведення для `int`, `double` і `string` масивів.
- Переконайтеся, що оброблено базовий випадок `n == 1`.
- **Примітка:** завдання оцінюється в 2 бали.
- **Пам'ять:** зверніть увагу на глибину рекурсії; за потреби оптимізуйте
  обхід, щоб не перевищити ліміт стека.

## Приклад `main`
```cpp
#include <iostream>
#include <vector>
#include <string>
using namespace std;

T findMin(arr, n) {
}

int main() {
    // Для цілих чисел
    int intN;
    cout << "Введіть розмір масиву (int): ";
    cin >> intN;

    vector<int> intArr(intN);
    cout << "Введіть " << intN << " чисел: ";
    for (int i = 0; i < intN; i++) {
        cin >> intArr[i];
    }
    cout << "Мінімум (int): " << findMin(intArr, intN) << endl;

    // Для дробових чисел
    int doubleN;
    cout << "\nВведіть розмір масиву (double): ";
    cin >> doubleN;

    vector<double> doubleArr(doubleN);
    cout << "Введіть " << doubleN << " дробових чисел: ";
    for (int i = 0; i < doubleN; i++) {
        cin >> doubleArr[i];
    }
    cout << "Мінімум (double): " << findMin(doubleArr, doubleN) << endl;

    // Для рядків
    int strN;
    cout << "\nВведіть кількість рядків: ";
    cin >> strN;

    vector<string> strArr(strN);
    cout << "Введіть " << strN << " рядків: ";
    for (int i = 0; i < strN; i++) {
        cin >> strArr[i];
    }
    cout << "Мінімум (string): " << findMin(strArr, strN) << endl;

    return 0;
}
```

### Приклад запуску
```text
=== Приклад 1 ===
Введіть розмір масиву (int): 4
Введіть 4 чисел: 7 3 9 1
Мінімум (int): 1

Введіть розмір масиву (double): 3
Введіть 3 дробових чисел: 5.5 2.2 4.4
Мінімум (double): 2.2

Введіть кількість рядків: 3
Введіть 3 рядків: kiwi apple mango
Мінімум (string): apple

=== Приклад 2 ===
Введіть розмір масиву (int): 1
Введіть 1 чисел: 42
Мінімум (int): 42

Введіть розмір масиву (double): 1
Введіть 1 дробових чисел: -3.14
Мінімум (double): -3.14

Введіть кількість рядків: 2
Введіть 2 рядків: beta alpha
Мінімум (string): alpha
```

# Завдання 4: Рекурсивне сортування (Merge Sort)

## Мета
Реалізувати універсальний рекурсивний merge sort.
Повторно використовуйте одну логіку.
Працюйте з різними типами.

## Вимоги
- Створіть шаблон `merge_sort(data, left, right, comp)`.
- Реалізуйте окрему функцію злиття.
- Підтримайте сортування `int`, `double` і `string`.
- Додайте простий кастомний компаратор на лямбді.
- Обробіть випадки `left >= right` і порожній масив.
- Додайте простий діалог введення/виведення.
- **Бали:** 3.
- **Пам'ять:** один буфер злиття.
- Перевикористовуйте його на всіх рівнях.

## Приклад `main`
```cpp
#include <iostream>
#include <string>
#include <vector>
using namespace std;

template <typename T, typename Compare>
void merge_sort(vector<T> &data, int left, int right, Compare comp);

int main() {
    int n;
    cout << "Скільки цілих? ";
    cin >> n;
    vector<int> numbers(n);
    for (int &value : numbers) {
        cin >> value;
    }
    auto ascending_int = [](const int &lhs, const int &rhs) {
        // реалізуйте порівняння
    };
    merge_sort(numbers, 0, n - 1, ascending_int);
    cout << "Int: ";
    for (int value : numbers) {
        cout << value << ' ';
    }
    cout << endl;

    int m;
    cout << "Скільки рядків? ";
    cin >> m;
    vector<string> words(m);
    for (string &word : words) {
        cin >> word;
    }
    auto by_length = [](const string &lhs, const string &rhs) {
        // реалізуйте порівняння
    };
    merge_sort(words, 0, m - 1, by_length);
    cout << "String: ";
    for (const string &word : words) {
        cout << word << ' ';
    }
    cout << endl;
}
```

### Приклад запуску
```text
=== Приклад ===
Скільки цілих? 5
4 1 3 2 5
Int: 1 2 3 4 5
Скільки рядків? 4
pear plum fig apple
String: fig pear plum apple
```
