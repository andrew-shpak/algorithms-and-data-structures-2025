# Lab 6 - Робота з CSV файлами

## Приклад розв'язання (на основі Practise-2)

```cpp
#include <iostream>   // cout
#include <fstream>    // ifstream
#include <sstream>    // stringstream
#include <vector>     // vector
#include <algorithm>  // sort, min
#include <iomanip>    // fixed, setprecision
using namespace std;

// Структура для зберігання даних студента
struct Student {
    string name;               // Ім'я студента
    int age;                   // Вік студента
    vector<double> grades;     // Оцінки студента
    int course;                // Курс навчання

    // Метод для обчислення середнього балу
    double getAverage() const {
        double sum = 0;
        for (double g : grades) {
            sum += g;
        }
        return sum / grades.size();
    }
};

// Функція для читання CSV файлу та парсингу даних студентів
vector<Student> readCSV(const string& filename) {
    vector<Student> students;
    ifstream file(filename);
    if (!file.is_open()) {
        return students;  // Повертаємо порожній вектор при помилці
    }

    string line;
    getline(file, line);  // Пропускаємо заголовок

    // Читаємо кожен рядок файлу
    while (getline(file, line)) {
        if (line.empty()) {
            continue;  // Пропускаємо порожні рядки
        }

        Student s;
        stringstream ss(line);
        string field;

        // Парсинг імені (в лапках)
        getline(ss, field, '"');
        getline(ss, s.name, '"');
        getline(ss, field, ',');

        // Парсинг віку
        getline(ss, field, ',');
        s.age = stoi(field);

        // Парсинг оцінок (в лапках, через кому)
        getline(ss, field, '"');
        getline(ss, field, '"');
        stringstream gradeStream(field);
        string grade;
        while (getline(gradeStream, grade, ',')) {
            s.grades.push_back(stod(grade));
        }

        // Парсинг курсу
        getline(ss, field, ',');
        getline(ss, field);
        s.course = stoi(field);

        students.push_back(s);
    }
    return students;
}

// Функція для виведення всіх записів студентів
void displayRecords(const vector<Student>& students) {
    cout << "\n=== ВСІ ЗАПИСИ СТУДЕНТІВ ===\n";
    for (const auto& s : students) {
        cout << "Ім'я: " << s.name << ", Вік: " << s.age
             << ", Курс: " << s.course << "\n";
    }
}

// Функція для виведення середніх балів всіх студентів
void displayAverages(const vector<Student>& students) {
    cout << "\n=== СЕРЕДНІ БАЛИ ===\n";
    for (const auto& s : students) {
        cout << s.name << ". Середній бал: "
             << fixed << setprecision(1) << s.getAverage() << "\n";
    }
}

// Функція для виведення топ N студентів (найвищі або найнижчі бали)
void displayTop(vector<Student> students, int n, bool highest) {
    // Сортування студентів за середнім балом використовуючи std::sort
    sort(students.begin(), students.end(), [highest](const Student& a, const Student& b) {
        double avgA = a.getAverage(), avgB = b.getAverage();
        if (avgA != avgB) {
            return highest ? avgA > avgB : avgA < avgB;  // За балом
        }
        return a.name < b.name;  // За ім'ям при однакових балах
    });

    cout << "\n=== ТОП " << n << (highest ? " НАЙВИЩІ" : " НАЙНИЖЧІ") << " БАЛИ ===\n";
    // Каст (int) потрібен для сумісності типів: n має тип int, students.size() повертає size_t
    for (int i = 0; i < min(n, (int)students.size()); i++) {
        cout << (i + 1) << ". " << students[i].name << " - "
             << fixed << setprecision(1) << students[i].getAverage() << "\n";
    }
}

int main() {
    vector<Student> students = readCSV("students.csv");

    if (students.empty()) {
        cout << "Помилка: файл не знайдено\n";
        return 1;
    }

    displayRecords(students);
    displayAverages(students);
    displayTop(students, 3, true);
    displayTop(students, 3, false);

    return 0;
}
```