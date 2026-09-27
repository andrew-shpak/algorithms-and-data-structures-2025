# Practise-2 — Читання CSV у C#

> **C# / .NET 10.** [Налаштування та запуск](../LABS.md).

## Читання файлу

Для першого прикладу збережіть файл `names.csv` у папці консольного проєкту:

```csv
ім'я
"Іван Петро"
"Олена Шевченко"
"Петро Іваненко"
```

`File.ReadLines` читає файл рядок за рядком, `Skip(1)` пропускає заголовок,
а `List<string>` зберігає прочитані імена. Файл шукається відносно поточної
робочої папки. Цей приклад обробляє лише показаний формат з однією колонкою.

```csharp
Console.OutputEncoding = System.Text.Encoding.UTF8;
string path = "names.csv";
if (!File.Exists(path))
{
    Console.Error.WriteLine($"Не вдалося знайти файл {path}");
    return;
}

List<string> userNames = [];
foreach (string line in File.ReadLines(path).Skip(1))
{
    if (string.IsNullOrWhiteSpace(line)) continue;
    string name = line.Trim().Trim('"');
    userNames.Add(name);
}

foreach (string name in userNames)
    Console.WriteLine(name);
```

## Колонка зі списком оцінок

У [students.csv](students.csv) чотири колонки: ім'я, вік, оцінки та курс.
Ім'я й список оцінок взято в лапки. Не розбивайте весь рядок лише за комами:
коми всередині лапок належать одній колонці.
Приклад читання саме цього формату наведено в [Lab 6](../Lab-6/README.md).

Після виділення колонки оцінок без зовнішніх лапок її можна розбити на числа:

```csharp
using System.Globalization;

string columnValue = "95,87,78";
List<double> grades = columnValue
    .Split(',', StringSplitOptions.RemoveEmptyEntries)
    .Select(value => double.Parse(value, CultureInfo.InvariantCulture))
    .ToList();

foreach (double grade in grades)
    Console.WriteLine(grade.ToString(CultureInfo.InvariantCulture));
```

## Рядки та перетворення чисел

```csharp
using System.Globalization;

string text = "algorithms";
Console.WriteLine(text[0]);     // a — перший символ непорожнього рядка
Console.WriteLine(text[^1]);    // s — останній символ
int age = int.Parse("19");
double grade = double.Parse("79.33", CultureInfo.InvariantCulture);
Console.WriteLine(age);
Console.WriteLine(grade.ToString(CultureInfo.InvariantCulture));
```

## Ініціалізація списків

```csharp
List<int> empty = [];
List<int> zeros = [0, 0, 0, 0, 0];
List<int> numbers = [1, 2, 3, 4, 5];
List<int> copy = new(numbers); // окремий список з тими самими значеннями
List<int> reserved = new(5);   // місткість 5, але Count == 0

Console.WriteLine(empty.Count);                 // 0
Console.WriteLine(string.Join(' ', zeros));     // 0 0 0 0 0
Console.WriteLine(string.Join(' ', copy));      // 1 2 3 4 5
Console.WriteLine(reserved.Count);              // 0
```

## Операції зі списком

```csharp
List<int> values = [];
Console.WriteLine(values.Count == 0); // True — список порожній
values.Add(10);
values.Add(20);
Console.WriteLine(values.Count);     // 2 — кількість елементів
Console.WriteLine(values[0]);        // 10 — доступ за індексом
```

## Середнє значення та форматування

```csharp
using System.Globalization;

List<double> grades = [80, 78, 80];
double average = grades.Count == 0 ? 0 : grades.Average();
Console.WriteLine(average.ToString("F2", CultureInfo.InvariantCulture)); // 79.33
```

Для завдання скопіюйте [students.csv](students.csv) до папки свого проєкту,
створіть тип `Student` і реалізуйте [необхідні операції](Task.md).
