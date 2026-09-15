# Задача 4: Система замовлень в ресторані

## Мета
Створити мінімалістичний менеджер замовлень із двома операціями: додати нове
замовлення та видалити (завершити) замовлення за номером столу.

## Вимоги
- Тип `Order` з властивостями: `TableNumber`, `LinkedList<string> Dishes`,
  `Status`.
- Клас `OrderBook` виконує лише додавання та видалення замовлень.
- Набір меню: `HashSet<string>` — перелік дозволених страв,
  використовується для валідації кожного нового замовлення (ініціалізуйте його у
  зручний для вас спосіб).

## Необхідні методи
```csharp
public class OrderBook
{
    private readonly LinkedList<Order> _orders = new();
    private readonly HashSet<string> _menuItems;

    public bool AddOrder(Order order);        // false, якщо страва не з меню або стіл вже має активне замовлення
    public bool RemoveOrder(int tableNumber); // false, якщо замовлення не знайдено
}
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
> налагодження, оскільки публічний інтерфейс містить лише методи `AddOrder`/`RemoveOrder`.
