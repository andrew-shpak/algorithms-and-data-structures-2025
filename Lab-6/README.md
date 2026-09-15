# Lab 6 - Робота з CSV файлами

## Приклад розв'язання (на основі Practise-2)

```csharp
using System.Globalization; // CultureInfo.InvariantCulture

List<Student> students = ReadCsv("students.csv");

if (students.Count == 0)
{
    Console.WriteLine("Помилка: файл не знайдено");
    return 1;
}

DisplayRecords(students);
DisplayAverages(students);
DisplayTop(students, 3, true);
DisplayTop(students, 3, false);

return 0;

// Метод для читання CSV файлу та парсингу даних студентів
static List<Student> ReadCsv(string fileName)
{
    var students = new List<Student>();
    if (!File.Exists(fileName))
    {
        return students; // Повертаємо порожній список при помилці
    }

    // Читаємо кожен рядок файлу, пропускаючи заголовок
    foreach (string line in File.ReadLines(fileName).Skip(1))
    {
        if (string.IsNullOrWhiteSpace(line))
        {
            continue; // Пропускаємо порожні рядки
        }

        // Рядок виду: "Ім'я",вік,"оцінка,оцінка,...",курс
        // Розбиття за лапками: ["", ім'я, ",вік,", оцінки, ",курс"]
        string[] parts = line.Split('"');

        string name = parts[1];                       // Ім'я (в лапках)
        int age = int.Parse(parts[2].Trim(','));      // Вік
        List<double> grades = parts[3]                // Оцінки (в лапках, через кому)
            .Split(',')
            .Select(g => double.Parse(g, CultureInfo.InvariantCulture))
            .ToList();
        int course = int.Parse(parts[4].Trim(','));   // Курс

        students.Add(new Student(name, age, grades, course));
    }
    return students;
}

// Метод для виведення всіх записів студентів
static void DisplayRecords(List<Student> students)
{
    Console.WriteLine("\n=== ВСІ ЗАПИСИ СТУДЕНТІВ ===");
    foreach (var s in students)
    {
        Console.WriteLine($"Ім'я: {s.Name}, Вік: {s.Age}, Курс: {s.Course}");
    }
}

// Метод для виведення середніх балів всіх студентів
static void DisplayAverages(List<Student> students)
{
    Console.WriteLine("\n=== СЕРЕДНІ БАЛИ ===");
    foreach (var s in students)
    {
        Console.WriteLine($"{s.Name}. Середній бал: {s.Average.ToString("F1", CultureInfo.InvariantCulture)}");
    }
}

// Метод для виведення топ N студентів (найвищі або найнижчі бали)
static void DisplayTop(List<Student> students, int n, bool highest)
{
    // Сортування копії списку за середнім балом (List<T>.Sort з компаратором)
    var sorted = new List<Student>(students);
    sorted.Sort((a, b) =>
    {
        int byAverage = highest
            ? b.Average.CompareTo(a.Average)  // За балом (спадання)
            : a.Average.CompareTo(b.Average); // За балом (зростання)
        return byAverage != 0 ? byAverage : string.CompareOrdinal(a.Name, b.Name); // За ім'ям при однакових балах
    });

    Console.WriteLine($"\n=== ТОП {n}{(highest ? " НАЙВИЩІ" : " НАЙНИЖЧІ")} БАЛИ ===");
    // Math.Min, щоб не вийти за межі списку, якщо студентів менше ніж n
    for (int i = 0; i < Math.Min(n, sorted.Count); i++)
    {
        Console.WriteLine($"{i + 1}. {sorted[i].Name} - {sorted[i].Average.ToString("F1", CultureInfo.InvariantCulture)}");
    }
}

// Запис для зберігання даних студента
record Student(string Name, int Age, List<double> Grades, int Course)
{
    // Властивість для обчислення середнього балу
    public double Average => Grades.Average();
}
```