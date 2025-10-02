# Lab 3
## Приклад для задачі 1
```cpp
#include <list>           // контейнер для зберігання книжок із швидкими вставками/видаленнями
#include <string>         // текстові поля книжки
#include <unordered_set>  // індекс унікальних ISBN

struct Book {
    std::string title;
    std::string author;
    std::string isbn;
    int year;
};

class Library {
public:
    // Додаємо книгу лише якщо ISBN ще не зареєстрований у системі
    bool add_book(const Book &book) {
        auto inserted = isbn_index_.insert(book.isbn);
        if (!inserted.second) {
            return false;  // ISBN дублюється — операція провалюється
        }
        books_.push_back(book);  // фактичне додавання до списку
        return true;
    }

    // Видаляємо книгу за ISBN; повертаємо false, якщо збіг не знайдено
    bool remove_book(const std::string &isbn) {
        for (auto it = books_.begin(); it != books_.end(); ++it) {
            if (it->isbn == isbn) {
                isbn_index_.erase(it->isbn);  // підтримка індексу унікальності
                books_.erase(it);             // видаляємо сам запис
                return true;
            }
        }
        return false;  // ISBN не знайдено
    }

private:
    std::list<Book> books_;                  // основне сховище книжок
    std::unordered_set<std::string> isbn_index_;  // використовується для контролю унікальних ISBN
};
```
