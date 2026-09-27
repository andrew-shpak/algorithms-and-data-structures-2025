# Задача 1: Система управління бібліотекою

> **C# / .NET 10.** [Налаштування та запуск](../LABS.md).

## Мета
Розробити мінімальний модуль обліку книжок із двома операціями: додати книгу та
видалити книгу за ISBN.

## Вимоги
- Клас `Book` з властивостями: `Title`, `Author`, `Isbn`, `Year`.
- Клас `Library`, що виконує лише операції додавання й видалення.
- Сховище книжок: `LinkedList<Book>`.
- Контроль унікальності ISBN: `HashSet<string>`.

## Необхідні методи
```csharp
public record Book(string Title, string Author, string Isbn, int Year);

public class Library
{
    private readonly LinkedList<Book> _books = new();
    private readonly HashSet<string> _isbnIndex = new();

    // false, якщо ISBN вже існує
    public bool AddBook(Book book) => throw new NotImplementedException();

    // false, якщо ISBN не знайдено
    public bool RemoveBook(string isbn) => throw new NotImplementedException();
}
```

## Приклад взаємодії
### Вхідні дані
```
add_book title="Clean Code" author="Robert C. Martin" isbn=9780132350884 year=2008
add_book title="Duplicate" author="Someone" isbn=9780132350884 year=2020
remove_book isbn=9780321334879
remove_book isbn=9780132350884
```

### Очікуваний вивід
```
OK
ERROR: ISBN already exists
ERROR: book not found
OK
```
