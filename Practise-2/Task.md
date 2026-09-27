## [IDE](https://onecompiler.com/csharp)

> **C# / .NET 10.** [Налаштування та запуск](../LABS.md).

# Tasks
- Read a CSV file and display its records (2 points)
	- Use [students.csv](students.csv) and create a C# class or record `Student` with `Name`, `Age`, `Grades` (`List<double>`), and `Course`.
- Calculate and display average student grades (1 point)
```text
	Іван Петро. Середній бал: 20.2
	Олена Шевченко. Середній бал: 25.3
```
- Find and display the top 3 students with the highest and lowest average grades (1 point)
	- Use `List<T>.Sort` or LINQ `OrderBy` / `OrderByDescending` and `Take(3)`.
```text
	1. Іван Петро - 25.3
	2. Олена Шевченко - 22.3
	3. Петро Іваненко - 20.4
```
