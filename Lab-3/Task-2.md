# Задача 2: Система управління проектами

> **C# / .NET 10.** [Налаштування та запуск](../LABS.md).

## Мета
Підтримувати перелік проектів у компанії за допомогою двох операцій: додати
проект і видалити проект.

## Вимоги
- Клас `Project` з властивостями: `Name`, `Manager`, `Budget`, `LinkedList<string> Members`.
- Клас `ProjectRegistry` надає лише операції додавання і видалення.
- Довідник валідних співробітників: `HashSet<string>` для
  перевірки учасників при додаванні проекту (його можна передавати в конструктор
  або ініціалізувати іншим зручним способом).

## Необхідні методи
```csharp
public record Project(string Name, string Manager, decimal Budget, LinkedList<string> Members);

public class ProjectRegistry
{
    private readonly LinkedList<Project> _projects = new();
    private readonly HashSet<string> _companyEmployees;

    public ProjectRegistry(IEnumerable<string> companyEmployees)
    {
        _companyEmployees = new HashSet<string>(companyEmployees);
    }

    // false, якщо назва зайнята або учасник поза компанією
    public bool AddProject(Project project) => throw new NotImplementedException();

    // false, якщо проект не знайдено
    public bool RemoveProject(string projectName) => throw new NotImplementedException();
}
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
