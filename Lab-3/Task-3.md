# Задача 3: Менеджер контактів

## Мета
Забезпечити мінімальний реєстр контактів із можливістю додати контакт і
видалити контакт за ім'ям.

## Вимоги
- Клас `Contact` з властивостями: `Name`, `Phone`, `Email`, `LinkedList<string> Tags`.
- Клас `ContactBook` обмежений операціями додавання й видалення.
- Унікальні імена контролюються внутрішнім індексом `HashSet<string>`.

## Необхідні методи
```csharp
public class ContactBook
{
    private readonly LinkedList<Contact> _contacts = new();
    private readonly HashSet<string> _nameIndex = new();

    public bool AddContact(Contact contact); // false, якщо ім'я вже зайняте
    public bool RemoveContact(string name);  // false, якщо контакт не знайдено
}
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
