# Задача 3: Менеджер контактів

## Мета
Забезпечити мінімальний реєстр контактів із можливістю додати контакт і
видалити контакт за ім'ям.

## Вимоги
- Клас `Contact` з полями: `name`, `phone`, `email`, `std::list<std::string> tags`.
- Клас `ContactBook` обмежений операціями додавання й видалення.
- Унікальні імена контролюються внутрішнім індексом `std::unordered_set<std::string>`.

## Необхідні методи
```cpp
class ContactBook {
public:
    bool add_contact(Contact contact);            // false, якщо ім'я вже зайняте
    bool remove_contact(const std::string &name); // false, якщо контакт не знайдено

private:
    std::list<Contact> contacts_;
    std::unordered_set<std::string> name_index_;
};
```

## Приклад взаємодії
### Вхідні дані
```
add_contact name=Alice phone=+380501112233 email=alice@example.com tags=[friend,work]
add_contact name=Alice phone=+380631234567 email=alice@work.com tags=[team]
remove_contact name=Bob
remove_contact name=Alice
```

### Очікуваний вивід
```
OK
ERROR: contact already exists
ERROR: contact not found
OK
```

> Після видалення контакту ім'я звільняється та може бути використане повторно.
