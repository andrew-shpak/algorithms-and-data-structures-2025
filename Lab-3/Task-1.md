# Задача 1: Система управління бібліотекою

## Мета
Розробити мінімальний модуль обліку книжок із двома операціями: додати книгу та
видалити книгу за ISBN.

## Вимоги
- Клас `Book` з полями: `title`, `author`, `isbn`, `year`.
- Клас `Library`, що виконує лише операції додавання й видалення.
- Сховище книжок: `std::list<Book>`.
- Контроль унікальності ISBN: `std::unordered_set<std::string>`.

## Необхідні методи
```cpp
class Library {
public:
    bool add_book(const Book &book);          // false, якщо ISBN вже існує
    bool remove_book(const std::string &isbn); // false, якщо ISBN не знайдено

private:
    std::list<Book> books_;
    std::unordered_set<std::string> isbn_index_;
};
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