# Задача 2: Система управління проектами

## Мета
Підтримувати перелік проектів у компанії за допомогою двох операцій: додати
проект і видалити проект.

## Вимоги
- Клас `Project` з полями: `name`, `manager`, `budget`, `std::list<std::string> members`.
- Клас `ProjectRegistry` надає лише операції додавання і видалення.
- Довідник валідних співробітників: `std::unordered_set<std::string>` для
  перевірки учасників при додаванні проекту (його можна передавати в конструктор
  або ініціалізувати іншим зручним способом).

## Необхідні методи
```cpp
class ProjectRegistry {
public:
    bool add_project(Project project);                    // false, якщо назва зайнята або учасник поза компанією
    bool remove_project(const std::string &project_name);  // false, якщо проект не знайдено

private:
    std::list<Project> projects_;
    std::unordered_set<std::string> company_employees_;
};
```

## Приклад взаємодії
### Вхідні дані
```
add_project name=Alpha manager=Alice budget=120000 members=[Bob,Carol]
add_project name=Alpha manager=Alice budget=80000 members=[Bob]
remove_project name=Beta
remove_project name=Alpha
```

### Очікуваний вивід
```
OK
ERROR: project already exists
ERROR: project not found
OK
```

> Учасники проекту перевіряються лише під час додавання; після видалення проекту
> жодних додаткових дій не потрібно.
