# Задача 4: Система замовлень в ресторані

## Мета
Створити мінімалістичний менеджер замовлень із двома операціями: додати нове
замовлення та видалити (завершити) замовлення за номером столу.

## Вимоги
- Структура `Order` з полями: `table_number`, `std::list<std::string> dishes`,
  `status`.
- Клас `OrderBook` виконує лише додавання та видалення замовлень.
- Набір меню: `std::unordered_set<std::string>` — перелік дозволених страв,
  використовується для валідації кожного нового замовлення (ініціалізуйте його у
  зручний для вас спосіб).

## Необхідні методи
```cpp
class OrderBook {
public:
    bool add_order(Order order);         // false, якщо страва не з меню або стіл вже має активне замовлення
    bool remove_order(int table_number); // false, якщо замовлення не знайдено

private:
    std::list<Order> orders_;
    std::unordered_set<std::string> menu_items_;
};
```

## Приклад взаємодії
### Вхідні дані
```
add_order table=1 dishes=[pizza,salad] status=new
add_order table=2 dishes=[pizza,pasta] status=new
add_order table=3 dishes=[sushi] status=new
remove_order table=2
remove_order table=4
```

### Очікуваний вивід
```
OK
OK
ERROR: dish not in menu
OK
ERROR: order not found
```

> Для перевірки актуального списку замовлень використовуйте власні інструменти
> налагодження, оскільки публічний інтерфейс містить лише методи add/remove.
