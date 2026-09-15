# Lab 3
## Приклад для задачі 1
```csharp
// LinkedList<T> — контейнер для зберігання книжок із швидкими вставками/видаленнями
// HashSet<string> — індекс унікальних ISBN

public record Book(string Title, string Author, string Isbn, int Year);

public class Library
{
    private readonly LinkedList<Book> _books = new();         // основне сховище книжок
    private readonly HashSet<string> _isbnIndex = new();      // використовується для контролю унікальних ISBN

    // Додаємо книгу лише якщо ISBN ще не зареєстрований у системі
    public bool AddBook(Book book)
    {
        if (!_isbnIndex.Add(book.Isbn))
        {
            return false;  // ISBN дублюється — операція провалюється
        }
        _books.AddLast(book);  // фактичне додавання до списку
        return true;
    }

    // Видаляємо книгу за ISBN; повертаємо false, якщо збіг не знайдено
    public bool RemoveBook(string isbn)
    {
        for (var node = _books.First; node is not null; node = node.Next)
        {
            if (node.Value.Isbn == isbn)
            {
                _isbnIndex.Remove(isbn);  // підтримка індексу унікальності
                _books.Remove(node);      // видаляємо сам запис
                return true;
            }
        }
        return false;  // ISBN не знайдено
    }
}
```
