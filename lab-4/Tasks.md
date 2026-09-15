# Завдання 1: Обчислення степеня

## Мета
Реалізувати функції піднесення числа до степеня для цілих та дробових типів,
забезпечуючи коректну обробку базових випадків і повторне використання логіки.

## Вимоги
- Реалізуйте узагальнений метод `Power(baseValue, exp)` і використайте його в `Program.cs`, як
  показано в прикладі.
- Забезпечте коректний результат для цілих і дробових чисел, передбачте
  випадок, коли `exp == 0`.
- Додайте діалог із користувачем (зчитування, вивід результатів), як у прикладі.
- **Примітка:** завдання оцінюється в 1 бал.
- **Пам'ять:** глибока рекурсія збільшує використання стека; для великих
  показників можна перейти на ітеративний варіант.


## Приклад `Program.cs`
```csharp
using System.Globalization;
using System.Numerics;

static T Power<T>(T baseValue, int exp) where T : INumber<T>
{
    throw new NotImplementedException();  // реалізуйте
}

// Для цілих чисел
Console.Write("Введіть ціле число (основа): ");
int intBase = int.Parse(Console.ReadLine()!);
Console.Write("Введіть степінь: ");
int intExp = int.Parse(Console.ReadLine()!);
Console.WriteLine($"Результат (int): {Power(intBase, intExp)}");

// Для дробових чисел
Console.Write("\nВведіть дробове число (основа): ");
double doubleBase = double.Parse(Console.ReadLine()!, CultureInfo.InvariantCulture);
Console.Write("Введіть степінь: ");
int doubleExp = int.Parse(Console.ReadLine()!);
Console.WriteLine($"Результат (double): {Power(doubleBase, doubleExp).ToString(CultureInfo.InvariantCulture)}");
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
- Напишіть узагальнений метод `ArraySum(arr, n)` і викличте його в `Program.cs`, як у наведеному
  прикладі.
- Підтримайте введення масивів для типів `int` та `double`, включно з випадком `n == 0`.
- Забезпечте виведення сум для обох масивів за зразком прикладу.
- **Примітка:** завдання оцінюється в 1 бали.
- **Пам'ять:** рекурсія споживає стек пропорційно `n`; для великих масивів
  дозволено перейти на ітеративний підрахунок.

## Приклад `Program.cs`
```csharp
using System.Globalization;
using System.Numerics;

static T ArraySum<T>(List<T> arr, int n) where T : INumber<T>
{
    throw new NotImplementedException();  // реалізуйте
}

static List<T> ReadValues<T>(int count) where T : IParsable<T> =>
    count == 0
        ? []
        : Console.ReadLine()!
            .Split(' ', StringSplitOptions.RemoveEmptyEntries)
            .Take(count)
            .Select(token => T.Parse(token, CultureInfo.InvariantCulture))
            .ToList();

// Для цілих чисел
Console.Write("Введіть розмір масиву (int): ");
int intN = int.Parse(Console.ReadLine()!);
if (intN > 0) Console.Write($"Введіть {intN} цілих чисел: ");
List<int> intArr = ReadValues<int>(intN);
Console.WriteLine($"Сума (int): {ArraySum(intArr, intN)}");

// Для дробових чисел
Console.Write("\nВведіть розмір масиву (double): ");
int doubleN = int.Parse(Console.ReadLine()!);
if (doubleN > 0) Console.Write($"Введіть {doubleN} дробових чисел: ");
List<double> doubleArr = ReadValues<double>(doubleN);
Console.WriteLine($"Сума (double): {ArraySum(doubleArr, doubleN).ToString(CultureInfo.InvariantCulture)}");
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
- Створіть узагальнений метод `FindMin(arr, n)` і використайте його в `Program.cs`, як у прикладі.
- Реалізуйте введення/виведення для `int`, `double` і `string` масивів (`List<T>`).
- Переконайтеся, що оброблено базовий випадок `n == 1`.
- **Примітка:** завдання оцінюється в 2 бали.
- **Пам'ять:** зверніть увагу на глибину рекурсії; за потреби оптимізуйте
  обхід, щоб не перевищити ліміт стека.

## Приклад `Program.cs`
```csharp
using System.Globalization;

static T FindMin<T>(List<T> arr, int n) where T : IComparable<T>
{
    throw new NotImplementedException();  // реалізуйте
}

static List<T> ReadValues<T>(int count) where T : IParsable<T> =>
    Console.ReadLine()!
        .Split(' ', StringSplitOptions.RemoveEmptyEntries)
        .Take(count)
        .Select(token => T.Parse(token, CultureInfo.InvariantCulture))
        .ToList();

// Для цілих чисел
Console.Write("Введіть розмір масиву (int): ");
int intN = int.Parse(Console.ReadLine()!);
Console.Write($"Введіть {intN} чисел: ");
List<int> intArr = ReadValues<int>(intN);
Console.WriteLine($"Мінімум (int): {FindMin(intArr, intN)}");

// Для дробових чисел
Console.Write("\nВведіть розмір масиву (double): ");
int doubleN = int.Parse(Console.ReadLine()!);
Console.Write($"Введіть {doubleN} дробових чисел: ");
List<double> doubleArr = ReadValues<double>(doubleN);
Console.WriteLine($"Мінімум (double): {FindMin(doubleArr, doubleN).ToString(CultureInfo.InvariantCulture)}");

// Для рядків
Console.Write("\nВведіть кількість рядків: ");
int strN = int.Parse(Console.ReadLine()!);
Console.Write($"Введіть {strN} рядків: ");
List<string> strArr = ReadValues<string>(strN);
Console.WriteLine($"Мінімум (string): {FindMin(strArr, strN)}");
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
- Створіть узагальнений метод `MergeSort<T>(data, left, right, comp)`, де `comp` — `Func<T, T, bool>`.
- Реалізуйте окремий метод злиття.
- Підтримайте сортування `int`, `double` і `string`.
- Додайте простий кастомний компаратор у вигляді лямбда-виразу.
- Обробіть випадки `left >= right` і порожній масив.
- Додайте простий діалог введення/виведення.
- **Бали:** 4.
- **Пам'ять:** один буфер злиття.
- Перевикористовуйте його на всіх рівнях.

## Приклад `Program.cs`
```csharp
static void MergeSort<T>(List<T> data, int left, int right, Func<T, T, bool> comp)
{
    throw new NotImplementedException();  // реалізуйте (разом з окремим методом злиття)
}

Console.Write("Скільки цілих? ");
int n = int.Parse(Console.ReadLine()!);
List<int> numbers = Console.ReadLine()!
    .Split(' ', StringSplitOptions.RemoveEmptyEntries)
    .Take(n)
    .Select(int.Parse)
    .ToList();
Func<int, int, bool> ascendingInt = (lhs, rhs) =>
{
    throw new NotImplementedException();  // реалізуйте порівняння
};
MergeSort(numbers, 0, n - 1, ascendingInt);
Console.WriteLine($"Int: {string.Join(' ', numbers)}");

Console.Write("Скільки рядків? ");
int m = int.Parse(Console.ReadLine()!);
List<string> words = Console.ReadLine()!
    .Split(' ', StringSplitOptions.RemoveEmptyEntries)
    .Take(m)
    .ToList();
Func<string, string, bool> byLength = (lhs, rhs) =>
{
    throw new NotImplementedException();  // реалізуйте порівняння
};
MergeSort(words, 0, m - 1, byLength);
Console.WriteLine($"String: {string.Join(' ', words)}");
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
