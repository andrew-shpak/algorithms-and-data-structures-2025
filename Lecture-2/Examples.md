# Лекція 2 — Масиви, стек і черга

> **Тривалість:** ≈ 180 хвилин (3 академічні пари з перервами).
> **Мова прикладів:** C# 12+ / .NET 8+ (перевірено на .NET 11 preview).
> **Формат:** кожен блок ` ```csharp ` — окрема повна консольна програма. Запуск: `dotnet new console`, вставити код у `Program.cs`, `dotnet run`. Під кожним прикладом наведено **Приклад запуску** — точний вивід програми (символ `…` позначає числа, що залежать від машини, наприклад час).

---

## Зміст

| № | Розділ | Хвилин |
|---|--------|--------|
| 1 | [Пам'ять і масиви](#1-память-і-масиви) | 30 |
| 2 | [Динамічний масив](#2-динамічний-масив) | 30 |
| — | *Перерва 1* | — |
| 3 | [Класичні задачі на масивах](#3-класичні-задачі-на-масивах) | 30 |
| 4 | [Зв'язні списки](#4-звязні-списки) | 25 |
| — | *Перерва 2* | — |
| 5 | [Стек](#5-стек) | 30 |
| 6 | [Черга](#6-черга) | 25 |
| 7 | [Підсумок](#7-підсумок) | 10 |
|   | **Разом** | **≈ 180** |

Детальніше:

1. [Пам'ять і масиви](#1-память-і-масиви)
   - 1.1 Неперервна пам'ять і доступ за O(1) · 1.2 Кеш процесора та локальність · 1.3 Значущі та посилальні типи в масивах · 1.4 Значення за замовчуванням і перевірка меж · 1.5 Клас `Array` · 1.6 `int[,]` проти `int[][]` · 1.7 `Span<T>`, `stackalloc`, `ArraySegment<T>` · 1.8 `ArrayPool<T>`
2. [Динамічний масив](#2-динамічний-масив)
   - 2.1 Ідея · 2.2 Повна реалізація `DynamicArray<T>` · 2.3 Амортизований аналіз · 2.4 `List<T>` зсередини та API · 2.5 `CollectionsMarshal.AsSpan` · 2.6 Пастки
3. [Класичні задачі на масивах](#3-класичні-задачі-на-масивах)
   - 3.1 Префіксні суми · 3.2 Два вказівники · 3.3 Ковзне вікно · 3.4 Алгоритм Кадане · 3.5 Розворот і циклічний зсув · 3.6 Злиття відсортованих масивів · 3.7 Варіанти бінарного пошуку · 3.8 Обходи матриці
4. [Зв'язні списки](#4-звязні-списки)
   - 4.1 Однозв'язний список · 4.2 Двозв'язний список із сентинелом · 4.3 `LinkedList<T>` · 4.4 Масив проти списку · 4.5 Класичні задачі на списках
5. [Стек](#5-стек)
   - 5.1 LIFO і реалізації · 5.2 `Stack<T>` · 5.3 Стек викликів і рекурсія · 5.4 Дужки та RPN · 5.5 Алгоритм сортувальної станції · 5.6 Min-stack · 5.7 Монотонний стек · 5.8 Undo/Redo
6. [Черга](#6-черга)
   - 6.1 FIFO і кільцевий буфер · 6.2 `Queue<T>` · 6.3 Дек · 6.4 BFS на сітці · 6.5 Максимум у ковзному вікні · 6.6 `PriorityQueue<TElement, TPriority>` · 6.7 Виробник/споживач
7. [Підсумок](#7-підсумок)
   - 7.1 Порівняльна таблиця · 7.2 Як обрати структуру · 7.3 Питання для самоперевірки · 7.4 Практичні завдання

---

## 1. Пам'ять і масиви

*≈ 30 хвилин*

### 1.1 Неперервна пам'ять і доступ за O(1)

**Масив** — це послідовність елементів **одного типу**, що лежать у пам'яті **одним неперервним блоком**. Розмір масиву фіксується під час створення і більше не змінюється.

Саме неперервність дає головну перевагу масиву — **доступ за індексом за O(1)**. Щоб знайти `a[i]`, середовищу виконання не треба нічого шукати: адреса обчислюється однією формулою:

```
адреса(a[i]) = адреса(a[0]) + i × розмір_елемента
```

Схема `int[] a = [10, 20, 30, 40, 50];` у пам'яті (64-бітний процес, `int` = 4 байти):

```
 змінна a (на стеку)           купа (managed heap)
 ┌──────────────┐              ┌───────────┬──────────┬──────┬──────┬──────┬──────┬──────┐
 │ посилання  ──┼─────────────►│ заголовок │ Length=5 │  10  │  20  │  30  │  40  │  50  │
 └──────────────┘              │ + тип     │          │      │      │      │      │      │
                               └───────────┴──────────┴──────┴──────┴──────┴──────┴──────┘
                                 16 байт      8 байт    [0]    [1]    [2]    [3]    [4]
                                                        +0     +4     +8     +12    +16  ← зсув від a[0]
```

Що важливо помітити:

- `int[]` у C# — це **посилальний тип**: сама змінна `a` зберігає лише посилання, а елементи живуть у керованій купі.
- Перед елементами лежить службовий **заголовок об'єкта** та поле **`Length`** — тому `a.Length` теж O(1).
- Елементи **ідуть впритул один до одного**. Це важливо для кешу процесора (див. 1.2).

| Операція над `T[]` | Складність | Пояснення |
|--------------------|-----------|-----------|
| `a[i]` (читання/запис) | O(1) | адреса за формулою |
| `a.Length` | O(1) | зберігається в заголовку |
| Пошук значення (`Array.IndexOf`) | O(n) | перебір підряд |
| Пошук у відсортованому (`Array.BinarySearch`) | O(log n) | ділення навпіл |
| Вставка / видалення в середині | O(n) | треба зсунути хвіст (і розмір фіксований!) |
| Зміна розміру (`Array.Resize`) | O(n) | створюється **новий** масив і копіюються елементи |
| Сортування (`Array.Sort`) | O(n log n) | introsort (гібрид quicksort + heapsort + insertion) |

### 1.2 Кеш процесора та локальність

Процесор читає пам'ять не байтами, а **кеш-лініями** (зазвичай по 64 байти) і тримає «гарячі» дані в кешах L1/L2/L3:

```
 ┌─────────┐   ~1 нс    ┌──────┐  ~4 нс  ┌──────┐  ~10 нс  ┌──────┐   ~100 нс   ┌──────────────┐
 │  ядро   │ ◄────────► │  L1  │ ◄─────► │  L2  │ ◄──────► │  L3  │ ◄─────────► │  RAM (ГБ)    │
 └─────────┘            │ 32КБ │         │ 1 МБ │          │ 30МБ │             └──────────────┘
                        └──────┘         └──────┘          └──────┘
```

Коли ми читаємо `a[0]`, у кеш потрапляє вся лінія — тобто ще й `a[1]…a[15]`. Наступні звернення **вже в кеші** («cache hit»). Це називається **просторова локальність**. Крім того, процесор помічає послідовний доступ і **заздалегідь підвантажує** наступні лінії (hardware prefetcher).

Зв'язний список (розділ 4) цієї переваги не має: вузли розкидані купою, і кожен перехід `node.Next` може бути «cache miss» — у 100 разів повільнішим за звернення до L1.

Порівняймо послідовний і випадковий обхід одного й того самого масиву:

```csharp
using System.Diagnostics;

const int N = 20_000_000;
int[] data = new int[N];
for (int i = 0; i < N; i++)
    data[i] = i % 100;                        // заповнюємо довільними значеннями

// Порядок обходу 1: 0, 1, 2, ... — сусідні комірки, кеш працює ідеально
int[] sequential = new int[N];
for (int i = 0; i < N; i++)
    sequential[i] = i;

// Порядок обходу 2: ті самі індекси, але перемішані — кожне звернення «стрибає» пам'яттю
int[] shuffled = (int[])sequential.Clone();
new Random(42).Shuffle(shuffled);             // Random.Shuffle з'явився в .NET 8

long sum1 = 0, sum2 = 0;
var sw = Stopwatch.StartNew();
foreach (int index in sequential)
    sum1 += data[index];
long seqMs = sw.ElapsedMilliseconds;

sw.Restart();
foreach (int index in shuffled)
    sum2 += data[index];
long rndMs = sw.ElapsedMilliseconds;

Console.WriteLine($"sums equal: {sum1 == sum2}");      // складаємо ті самі числа
Console.WriteLine($"sequential: {seqMs} ms");
Console.WriteLine($"random:     {rndMs} ms");          // зазвичай у кілька разів довше
```

**Приклад запуску:**

```
sums equal: True
sequential: … ms
random:     … ms
```

На типовому ноутбуці послідовний обхід займає ≈ 15–20 мс, а випадковий — ≈ 80–200 мс. Алгоритм і асимптотика **однакові** (O(n)), а різниця — лише у поведінці кешу.

> **Висновок:** Big-O рахує кроки, але не враховує ціну одного кроку. На практиці масив часто обганяє «теоретично кращі» структури саме завдяки локальності.

### 1.3 Значущі та посилальні типи в масивах

Масиви поводяться по-різному залежно від типу елементів:

```
 PointS[] (struct — значущий тип)          PointC[] (class — посилальний тип)
 ┌─────┬─────┬─────┬─────┐                 ┌──────┬──────┬──────┐
 │ X=1 │ Y=2 │ X=3 │ Y=4 │                 │ ref ─┼─┐    │ null │
 └─────┴─────┴─────┴─────┘                 └──────┴─┼────┴──────┘
   [0]         [1]                                  ▼
 дані лежать ПРЯМО в масиві               ┌─────────────────┐
                                           │ заголовок│X=1│Y=2│  окремий об'єкт у купі
                                           └─────────────────┘
```

- **`struct[]`** — значення зберігаються всередині масиву. Один блок пам'яті, чудова локальність, немає окремих об'єктів для GC.
- **`class[]`** — масив зберігає лише **посилання** (8 байт кожне); самі об'єкти розкидані купою. Нові елементи = `null`.
- **`object[]` із числами** — кожне `int` **упаковується** (boxing) в окремий об'єкт у купі: ≈ 24 байти замість 4 плюс посилання 8 байт.

```csharp
// 1) Масив структур: елемент — це змінна всередині масиву
var structs = new PointS[2];
structs[0].X = 5;                    // змінюємо поле прямо в масиві
PointS copy = structs[0];            // присвоєння struct — КОПІЯ значення
copy.X = 100;
Console.WriteLine($"structs[0].X = {structs[0].X}");   // 5: копія не вплинула

// 2) Масив класів: елементи — посилання, спочатку null
var classes = new PointC?[2];
Console.WriteLine($"classes[1] is null: {classes[1] is null}");
classes[0] = new PointC { X = 5 };
PointC alias = classes[0]!;          // копіюється ПОСИЛАННЯ, об'єкт той самий
alias.X = 100;
Console.WriteLine($"classes[0].X = {classes[0]!.X}");  // 100: змінили спільний об'єкт

// 3) Присвоєння масиву — теж копіювання посилання
int[] a = [1, 2, 3];
int[] b = a;                         // b і a вказують на ОДИН масив
b[0] = 99;
int[] c = (int[])a.Clone();          // Clone — новий масив (поверхнева копія)
c[1] = -1;
Console.WriteLine($"a = [{string.Join(", ", a)}], c = [{string.Join(", ", c)}]");

// 4) Clone для масиву класів копіює лише посилання («shallow copy»)
PointC?[] shallow = (PointC?[])classes.Clone();
shallow[0]!.X = 7;
Console.WriteLine($"after shallow change: classes[0].X = {classes[0]!.X}");

// 5) Упаковка: int[] проти object[]
long before = GC.GetAllocatedBytesForCurrentThread();
int[] ints = new int[1000];
for (int i = 0; i < ints.Length; i++) ints[i] = i;
long intBytes = GC.GetAllocatedBytesForCurrentThread() - before;

before = GC.GetAllocatedBytesForCurrentThread();
object[] boxes = new object[1000];
for (int i = 0; i < boxes.Length; i++) boxes[i] = i + 1000;   // кожне число → окремий об'єкт
long objBytes = GC.GetAllocatedBytesForCurrentThread() - before;

Console.WriteLine($"int[1000]    ≈ {intBytes / 1000 * 1000} bytes");
Console.WriteLine($"object[1000] uses more memory: {objBytes > 5 * intBytes}");

public struct PointS { public int X; public int Y; }
public sealed class PointC { public int X; public int Y; }
```

**Приклад запуску:**

```
structs[0].X = 5
classes[1] is null: True
classes[0].X = 100
a = [99, 2, 3], c = [99, -1, 3]
after shallow change: classes[0].X = 7
int[1000]    ≈ 4000 bytes
object[1000] uses more memory: True
```

| Масив | Байт на елемент (x64) | Об'єктів для GC | Локальність |
|-------|-----------------------|-----------------|-------------|
| `int[]` | 4 | 1 | відмінна |
| `long[]` / `double[]` | 8 | 1 | відмінна |
| `PointS[]` (2 × `int`) | 8 | 1 | відмінна |
| `PointC[]` | 8 (посилання) + ≈ 32 (об'єкт) | n + 1 | погана |
| `object[]` з `int` | 8 + 24 (boxed int) | n + 1 | погана |

### 1.4 Значення за замовчуванням, перевірка меж, індекси з кінця

- `new T[n]` **завжди обнуляє** пам'ять: `0` для чисел, `false` для `bool`, `'\0'` для `char`, `null` для посилань.
- Кожне звернення `a[i]` перевіряється: якщо `i < 0` або `i >= Length`, кидається **`IndexOutOfRangeException`**. На відміну від C++, вийти за межі й «тихо» зіпсувати пам'ять неможливо.
- JIT уміє **прибирати перевірку меж**, коли доводить, що індекс коректний (класичний цикл `for (int i = 0; i < a.Length; i++)`).
- `a[^1]` — останній елемент (`Index` з кінця), `a[1..3]` — діапазон (`Range`); для масиву діапазон **створює копію**.

```csharp
int[] numbers = new int[4];
string?[] names = new string?[3];
bool[] flags = new bool[2];
char[] letters = new char[2];

Console.WriteLine($"int[]:    {string.Join(",", numbers)}");
Console.WriteLine($"string[]: {string.Join(",", names.Select(n => n ?? "null"))}");
Console.WriteLine($"bool[]:   {string.Join(",", flags)}");
Console.WriteLine($"char[]:   {(int)letters[0]},{(int)letters[1]}");

// Перевірка меж — виняток замість пошкодження пам'яті
try
{
    numbers[4] = 1;                        // допустимі індекси: 0..3
}
catch (IndexOutOfRangeException ex)
{
    Console.WriteLine($"{ex.GetType().Name}: {ex.Message}");
}

// Індекси з кінця та діапазони
int[] primes = [2, 3, 5, 7, 11, 13];
Console.WriteLine($"primes[^1] = {primes[^1]}, primes[^2] = {primes[^2]}");
int[] slice = primes[1..4];                // елементи з індексами 1, 2, 3 — НОВИЙ масив
slice[0] = 100;                            // оригінал не змінюється
Console.WriteLine($"slice = [{string.Join(", ", slice)}], primes[1] = {primes[1]}");

// Масив нульової довжини — законний і корисний (замість null)
int[] empty = [];
Console.WriteLine($"empty.Length = {empty.Length}, same instance: {ReferenceEquals(empty, Array.Empty<int>())}");
```

**Приклад запуску:**

```
int[]:    0,0,0,0
string[]: null,null,null
bool[]:   False,False
char[]:   0,0
IndexOutOfRangeException: Index was outside the bounds of the array.
primes[^1] = 13, primes[^2] = 11
slice = [100, 5, 7], primes[1] = 3
empty.Length = 0, same instance: True
```

### 1.5 Клас `Array`: копіювання, пошук, сортування

Статичний клас `System.Array` містить базові алгоритми над масивами. Найуживаніші:

| Метод | Що робить | Складність |
|-------|-----------|-----------|
| `Array.Copy(src, srcIdx, dst, dstIdx, len)` | копіює діапазон (коректно працює з перекриттям) | O(len) |
| `Array.Resize(ref a, newSize)` | створює новий масив і копіює | O(n) |
| `Array.Fill(a, value[, start, count])` | заповнює значенням | O(n) |
| `Array.Clear(a[, start, count])` | обнуляє | O(n) |
| `Array.Reverse(a[, start, count])` | розвертає на місці | O(n) |
| `Array.IndexOf` / `LastIndexOf` | лінійний пошук | O(n) |
| `Array.Find` / `FindIndex` / `Exists` | пошук за умовою | O(n) |
| `Array.Sort(a[, comparer])` | сортування на місці | O(n log n) |
| `Array.BinarySearch(a, value)` | бінарний пошук у **відсортованому** | O(log n) |

```csharp
int[] data = [5, 3, 8, 1, 9, 2];
Print("data", data);

// Copy: копіюємо в інший масив
int[] sorted = new int[data.Length];
Array.Copy(data, sorted, data.Length);

// Sort + BinarySearch
Array.Sort(sorted);
Print("sorted", sorted);
int pos = Array.BinarySearch(sorted, 5);
int miss = Array.BinarySearch(sorted, 4);          // немає → від'ємне число
Console.WriteLine($"BinarySearch(5) = {pos}, BinarySearch(4) = {miss}, insert at {~miss}");

// Reverse на місці
Array.Reverse(sorted);
Print("reversed", sorted);

// Лінійний пошук
Console.WriteLine($"IndexOf(8) = {Array.IndexOf(data, 8)}, IndexOf(42) = {Array.IndexOf(data, 42)}");
Console.WriteLine($"FindIndex(x > 7) = {Array.FindIndex(data, x => x > 7)}, Exists(x < 0) = {Array.Exists(data, x => x < 0)}");

// Resize: насправді створює НОВИЙ масив і переприсвоює змінну (тому ref)
int[] original = data;
Array.Resize(ref data, 8);
Print("resized", data);
Console.WriteLine($"same object after Resize: {ReferenceEquals(original, data)}");

// Fill і Clear частини масиву
Array.Fill(data, 7, 6, 2);                          // з індексу 6, два елементи
Array.Clear(data, 0, 2);                            // з індексу 0, два елементи
Print("fill+clear", data);

// Copy з перекриттям: зсув усього масиву на 1 вправо (Array.Copy це підтримує)
int[] shift = [1, 2, 3, 4, 5];
Array.Copy(shift, 0, shift, 1, shift.Length - 1);
shift[0] = 0;
Print("shifted", shift);

// Сортування за ключем і зі своїм компаратором
string[] words = ["pear", "fig", "banana", "kiwi"];
Array.Sort(words, (x, y) => x.Length != y.Length ? x.Length.CompareTo(y.Length) : string.CompareOrdinal(x, y));
Console.WriteLine($"by length: {string.Join(" ", words)}");

static void Print(string label, int[] array) =>
    Console.WriteLine($"{label,-10}: {string.Join(" ", array)}");
```

**Приклад запуску:**

```
data      : 5 3 8 1 9 2
sorted    : 1 2 3 5 8 9
BinarySearch(5) = 3, BinarySearch(4) = -4, insert at 3
reversed  : 9 8 5 3 2 1
IndexOf(8) = 2, IndexOf(42) = -1
FindIndex(x > 7) = 2, Exists(x < 0) = False
resized   : 5 3 8 1 9 2 0 0
same object after Resize: False
fill+clear: 0 0 8 1 9 2 7 7
shifted   : 0 1 2 3 4
by length: fig kiwi pear banana
```

> **Чому `~miss`?** Якщо елемента немає, `BinarySearch` повертає побітове доповнення індексу першого більшого елемента: `-(index) - 1`. Оператор `~` відновлює цей індекс — саме туди треба вставити значення, щоб зберегти порядок.

### 1.6 Багатовимірні `int[,]` проти зубчастих `int[][]`

```
 int[,] grid = new int[3, 4]                  int[][] jagged = new int[3][]
 один об'єкт, один неперервний блок           масив посилань + окремі рядки в купі

 ┌───────────┬────┬────┬────┬────┐           ┌──────┐     ┌────┬────┐
 │ заголовок │ 00 │ 01 │ 02 │ 03 │ ← рядок 0 │ [0] ─┼────►│ 1  │ 2  │
 │ + розміри ├────┼────┼────┼────┤           ├──────┤     └────┴────┘
 │           │ 10 │ 11 │ 12 │ 13 │ ← рядок 1 │ [1] ─┼────►┌────┬────┬────┬────┐
 │           ├────┼────┼────┼────┤           ├──────┤     │ 3  │ 4  │ 5  │ 6  │
 │           │ 20 │ 21 │ 22 │ 23 │ ← рядок 2 │ [2] ─┼──┐  └────┴────┴────┴────┘
 └───────────┴────┴────┴────┴────┘           └──────┘  └─►┌────┐
 фізично: 00 01 02 03 10 11 ... 23                        │ 7  │
 (row-major — рядок за рядком)                            └────┘
```

| Властивість | `int[,]` | `int[][]` |
|-------------|----------|-----------|
| Будова | один блок rows × cols | масив масивів |
| Рядки різної довжини | ні | так |
| Кількість об'єктів у купі | 1 | rows + 1 |
| Доступ `m[r, c]` | множення + 2 перевірки меж (JIT оптимізує гірше) | два звичайні індекси; внутрішній цикл по `int[]` оптимізується добре |
| Швидкодія на практиці | часто **повільніша** | зазвичай **швидша** |
| Зручність | `GetLength(0/1)`, `Rank` | `m.Length`, `m[r].Length` |

**Порядок обходу важливий!** Оскільки рядки лежать у пам'яті один за одним, зовнішній цикл має йти по рядках, а внутрішній — по стовпцях. Обхід «стовпцями» стрибає через цілий рядок на кожному кроці.

Спочатку класичний приклад — прямокутна матриця і трикутник Паскаля:

```csharp
// Прямокутна матриця: один неперервний блок пам'яті rows × cols
int[,] grid = new int[3, 4];
for (int r = 0; r < grid.GetLength(0); r++)      // GetLength(0) — кількість рядків
    for (int c = 0; c < grid.GetLength(1); c++)  // GetLength(1) — кількість стовпців
        grid[r, c] = r * 10 + c;
Console.WriteLine($"grid[2,3]={grid[2, 3]} total={grid.Length} rank={grid.Rank}");

// foreach по int[,] іде рядок за рядком (row-major)
Console.WriteLine($"foreach: {string.Join(' ', grid.Cast<int>())}");

// Jagged-масив: масив масивів, рядки можуть мати різну довжину
int[][] triangle = new int[4][];
for (int r = 0; r < triangle.Length; r++)
{
    triangle[r] = new int[r + 1];                // кожен рядок — окремий масив у купі
    triangle[r][0] = triangle[r][r] = 1;         // краї трикутника Паскаля
    for (int c = 1; c < r; c++)
        triangle[r][c] = triangle[r - 1][c - 1] + triangle[r - 1][c];
}
foreach (int[] row in triangle)
    Console.WriteLine(string.Join(' ', row));
```

**Приклад запуску:**

```
grid[2,3]=23 total=12 rank=2
foreach: 0 1 2 3 10 11 12 13 20 21 22 23
1
1 1
1 2 1
1 3 3 1
```

Тепер виміряймо вплив порядку обходу та виду масиву:

```csharp
using System.Diagnostics;

const int Size = 3000;
int[,] rect = new int[Size, Size];
int[][] jagged = new int[Size][];
for (int r = 0; r < Size; r++)
{
    jagged[r] = new int[Size];
    for (int c = 0; c < Size; c++)
        rect[r, c] = jagged[r][c] = (r + c) & 7;
}

Measure("int[,]  rows first", () =>
{
    long s = 0;
    for (int r = 0; r < Size; r++)
        for (int c = 0; c < Size; c++)
            s += rect[r, c];                     // сусідні комірки пам'яті
    return s;
});

Measure("int[,]  cols first", () =>
{
    long s = 0;
    for (int c = 0; c < Size; c++)
        for (int r = 0; r < Size; r++)
            s += rect[r, c];                     // стрибок на Size комірок щоразу
    return s;
});

Measure("int[][] rows first", () =>
{
    long s = 0;
    foreach (int[] row in jagged)                // рядок — звичайний int[]
        foreach (int value in row)               // JIT прибирає перевірки меж
            s += value;
    return s;
});

static void Measure(string name, Func<long> body)
{
    var sw = Stopwatch.StartNew();
    long result = body();
    Console.WriteLine($"{name}: sum={result}, {sw.ElapsedMilliseconds} ms");
}
```

**Приклад запуску:**

```
int[,]  rows first: sum=31500000, … ms
int[,]  cols first: sum=31500000, … ms
int[][] rows first: sum=31500000, … ms
```

Типові результати: «rows first» ≈ 10–15 мс, «cols first» ≈ 30–60 мс, jagged ≈ 5–10 мс. Суми однакові, різниця — лише у доступі до пам'яті.

### 1.7 `Span<T>`, `stackalloc`, `ArraySegment<T>`

Часто потрібно працювати з **частиною** масиву, не копіюючи її. Для цього в .NET є «вікна» на пам'ять:

```
 int[] arr:   ┌────┬────┬────┬────┬────┬────┐
              │ 10 │ 20 │ 30 │ 40 │ 50 │ 60 │
              └────┴────┴────┴────┴────┴────┘
                     ▲                   ▲
 Span<int> middle = arr.AsSpan(1, 4)
 = (вказівник на arr[1], Length = 4) — жодного копіювання
```

| Тип | Де може жити | Особливості |
|-----|--------------|-------------|
| `Span<T>` / `ReadOnlySpan<T>` | тільки на стеку (`ref struct`) | вказує на масив, `stackalloc`-пам'ять, рядок, нативну пам'ять; не можна зберегти в полі класу чи використати після `await` |
| `Memory<T>` / `ReadOnlyMemory<T>` | будь-де | «довгоживучий» аналог Span; `.Span` дає Span |
| `ArraySegment<T>` | будь-де | старіший тип: масив + `Offset` + `Count` |
| `stackalloc T[n]` | стек методу | дуже швидке виділення без GC; тільки невеликі розміри (≈ до кількох КБ) |

```csharp
int[] arr = [10, 20, 30, 40, 50, 60];

// Span — «вікно» на частину масиву без копіювання
Span<int> middle = arr.AsSpan(1, 4);           // 20 30 40 50
middle[0] = 21;                                // змінює arr[1]!
middle.Reverse();                              // розвертає лише вікно
Console.WriteLine($"arr = {string.Join(" ", arr)}");

// Slice і діапазони на Span теж не копіюють
Span<int> lastTwo = arr.AsSpan()[^2..];
lastTwo.Fill(0);
Console.WriteLine($"arr = {string.Join(" ", arr)}");

// Один метод приймає і масив, і Span, і stackalloc-пам'ять
Console.WriteLine($"Sum(arr) = {Sum(arr)}");
Span<int> squares = stackalloc int[5];         // пам'ять на стеку, GC не задіяний
for (int i = 0; i < squares.Length; i++)
    squares[i] = i * i;
Console.WriteLine($"Sum(squares) = {Sum(squares)}");

// ReadOnlySpan<char> — розбір рядка без створення підрядків
ReadOnlySpan<char> date = "2025-09-15";
int year = int.Parse(date[..4]);
int month = int.Parse(date.Slice(5, 2));
int day = int.Parse(date[^2..]);
Console.WriteLine($"year={year} month={month} day={day}");

// ArraySegment — можна зберегти в полі або колекції (Span — не можна)
var segment = new ArraySegment<int>(arr, 1, 3);
Console.WriteLine($"segment: offset={segment.Offset} count={segment.Count} first={segment[0]} items={string.Join(" ", segment)}");

// Порівняння вмісту та пошук у Span — векторизовані й швидкі
ReadOnlySpan<int> a = [1, 2, 3];
ReadOnlySpan<int> b = [1, 2, 3];
Console.WriteLine($"SequenceEqual = {a.SequenceEqual(b)}, IndexOf(3) = {a.IndexOf(3)}");

static int Sum(ReadOnlySpan<int> values)
{
    int total = 0;
    foreach (int v in values)                  // foreach по Span — без виділень пам'яті
        total += v;
    return total;
}
```

**Приклад запуску:**

```
arr = 10 50 40 30 21 60
arr = 10 50 40 30 0 0
Sum(arr) = 130
Sum(squares) = 30
year=2025 month=9 day=15
segment: offset=1 count=3 first=50 items=50 40 30
SequenceEqual = True, IndexOf(3) = 2
```

### 1.8 `ArrayPool<T>`: повторне використання буферів

Якщо програма часто створює великі тимчасові масиви (наприклад, буфер для читання файлу чи мережевого пакета), GC доводиться постійно їх прибирати. **`ArrayPool<T>`** дозволяє «орендувати» масив і повертати його після використання.

```
  Rent(100)            використання             Return(buffer)
 ┌────────┐  масив ≥ 100  ┌──────────┐  той самий масив  ┌────────┐
 │  пул   │ ─────────────►│ ваш код  │ ─────────────────►│  пул   │ → наступний Rent отримає його знову
 └────────┘               └──────────┘                   └────────┘
```

Правила:

1. `Rent(n)` повертає масив **довжиною не менше n** (зазвичай округлено до степеня двійки). Працюйте лише з першими n елементами.
2. Вміст орендованого масиву **не обнулено** — там може бути сміття з минулого використання.
3. Обов'язково `Return` у `finally`. Після повернення масив **не можна** використовувати.

```csharp
using System.Buffers;

ArrayPool<byte> pool = ArrayPool<byte>.Shared;

byte[] buffer = pool.Rent(100);
try
{
    Console.WriteLine($"Rent(100) -> Length = {buffer.Length}");  // округлено до 128
    Span<byte> work = buffer.AsSpan(0, 100);   // працюємо лише з потрібною частиною
    for (int i = 0; i < work.Length; i++)
        work[i] = (byte)i;
    Console.WriteLine($"checksum = {Checksum(work)}");
}
finally
{
    pool.Return(buffer, clearArray: true);     // очищуємо, якщо там були чутливі дані
}

// Наступна оренда того самого розміру зазвичай отримує той самий масив
byte[] again = pool.Rent(120);
Console.WriteLine($"reused the same array: {ReferenceEquals(buffer, again)}, first byte = {again[0]}");
pool.Return(again);

static int Checksum(ReadOnlySpan<byte> bytes)
{
    int sum = 0;
    foreach (byte b in bytes)
        sum += b;
    return sum;
}
```

**Приклад запуску:**

```
Rent(100) -> Length = 128
checksum = 4950
reused the same array: True, first byte = 0
```

### Типові помилки (розділ 1)

1. **Думати, що `int[] b = a;` копіює масив.** Копіюється лише посилання. Для копії: `a.Clone()`, `a.ToArray()`, `a[..]` або `Array.Copy`.
2. **Вихід за межі на одиницю:** `for (int i = 0; i <= a.Length; i++)` — остання ітерація кине `IndexOutOfRangeException`. Правильно `i < a.Length`.
3. **`BinarySearch` на невідсортованому масиві** — результат не визначений (не виняток, а просто неправильна відповідь).
4. **Забути, що `Array.Resize` створює новий масив** — інші змінні, які посилались на старий масив, його і бачать.
5. **Обхід `int[,]` стовпцями** в «гарячому» циклі — повільно через кеш.
6. **`stackalloc` великого розміру** (наприклад, `stackalloc byte[1_000_000]`) — ризик `StackOverflowException`, який неможливо перехопити. Для великих буферів — `ArrayPool<T>`.
7. **Використання масиву з `ArrayPool` після `Return`** або припущення, що `Rent(n).Length == n`.
8. **`a[1..3]` на масиві вважати «вікном»** — це копія. Вікно без копіювання — `a.AsSpan(1..3)`.

### Міні-вправи (розділ 1)

**Вправа 1.1.** Не використовуючи `Array.Reverse`, розверніть частину масиву з індексу `from` до `to` включно, передавши її як `Span<int>`.

<details>
<summary>Розв'язок</summary>

```csharp
int[] values = [1, 2, 3, 4, 5, 6, 7];
ReverseInPlace(values.AsSpan(2, 4));            // індекси 2..5 → 3 4 5 6
Console.WriteLine(string.Join(" ", values));

static void ReverseInPlace(Span<int> span)
{
    int left = 0, right = span.Length - 1;
    while (left < right)
    {
        (span[left], span[right]) = (span[right], span[left]);  // обмін через кортеж
        left++;
        right--;
    }
}
```

**Приклад запуску:**

```
1 2 6 5 4 3 7
```

</details>

**Вправа 1.2.** Дано зубчастий масив оцінок студентів (у кожного різна кількість оцінок). Виведіть середню оцінку кожного студента з однією цифрою після коми.

<details>
<summary>Розв'язок</summary>

```csharp
int[][] grades =
[
    [90, 85, 77],
    [60],
    [100, 95, 98, 91],
];

for (int student = 0; student < grades.Length; student++)
{
    int[] row = grades[student];            // рядок — звичайний int[]
    double average = row.Length == 0 ? 0 : (double)row.Sum() / row.Length;
    Console.WriteLine($"student {student}: {row.Length} grades, average {average:F1}");
}
```

**Приклад запуску:**

```
student 0: 3 grades, average 84.0
student 1: 1 grades, average 60.0
student 2: 4 grades, average 96.0
```

</details>

**Вправа 1.3.** Чому `double average = row.Sum() / row.Length;` дасть неправильний результат? *Відповідь:* ділення цілих чисел відкидає дробову частину (`251 / 3 == 83`), і лише потім результат перетворюється на `double`. Потрібно привести одне з чисел до `double` до ділення.

---

## 2. Динамічний масив

*≈ 30 хвилин*

### 2.1 Ідея: масив, який уміє рости

Звичайний масив `T[]` має **фіксований розмір** і лежить у пам'яті одним неперервним блоком — тому доступ `a[i]` займає O(1).
Динамічний масив (`List<T>` у .NET, `std::vector` у C++, `ArrayList` у Java) — це обгортка над таким масивом із двома числами:

- **`Count`** — скільки елементів реально збережено;
- **`Capacity`** — скільки місця виділено.

```
 MyList<int> після Add(1), Add(2), Add(3):

 _items ──►┌─────┬─────┬─────┬─────┐
           │  1  │  2  │  3  │  0  │    Capacity = 4 (довжина буфера)
           └─────┴─────┴─────┴─────┘
             [0]   [1]   [2]   [3]
                               ▲
                          Count = 3 — наступний Add пише сюди

 Add(4): місце є → _items[3] = 4, Count = 4                       O(1)

 Add(5): Count == Capacity → ріст:
   1) new int[8]
   2) скопіювати 4 старі елементи                                  O(n)
   3) записати 5 у [4]
           ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
           │  1  │  2  │  3  │  4  │  5  │  0  │  0  │  0  │   Capacity = 8, Count = 5
           └─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘
   старий масив на 4 елементи стає сміттям і його прибере GC
```

Коли `Count == Capacity`, виділяється **новий масив удвічі більший**, старі елементи копіюються (O(n)), і лише потім додається новий.
Копіювання трапляється рідко (на розмірах 1, 2, 4, 8, ...), тому сумарно n додавань коштують O(n), а один `Add` — **амортизовано O(1)** (доведення — у 2.3).

Мінімальна версія — лише подвоєння і індексатор:

```csharp
var numbers = new DynamicArray<int>();   // старт: Capacity = 1, Count = 0
for (int i = 1; i <= 5; i++)
{
    numbers.Add(i * 10);                 // коли місця немає — масив подвоюється
    Console.WriteLine($"Add({i * 10}): Count={numbers.Count} Capacity={numbers.Capacity}");
}
Console.WriteLine($"numbers[3]={numbers[3]}"); // доступ за індексом — O(1)

// Мінімальний динамічний масив: неперервний буфер + лічильник елементів
public sealed class DynamicArray<T>
{
    private T[] _items = new T[1];       // внутрішній буфер фіксованого розміру
    public int Count { get; private set; } // скільки елементів реально зайнято
    public int Capacity => _items.Length;  // скільки місця виділено

    public void Add(T item)
    {
        if (Count == Capacity)           // буфер заповнений
        {
            var bigger = new T[Capacity * 2];         // виділяємо вдвічі більший масив
            Array.Copy(_items, bigger, Count);        // копіюємо старі елементи — O(n), але рідко
            _items = bigger;                          // старий масив забере збирач сміття
        }
        _items[Count++] = item;          // звичайний запис у кінець — O(1)
    }

    public T this[int index]
    {
        get
        {
            // перевіряємо межі саме за Count, а не за Capacity
            ArgumentOutOfRangeException.ThrowIfGreaterThanOrEqual((uint)index, (uint)Count, nameof(index));
            return _items[index];
        }
    }
}
```

**Приклад запуску:**

```
Add(10): Count=1 Capacity=1
Add(20): Count=2 Capacity=2
Add(30): Count=3 Capacity=4
Add(40): Count=4 Capacity=4
Add(50): Count=5 Capacity=8
numbers[3]=40
```

> **Чому `(uint)index`?** Від'ємне `int`, приведене до `uint`, стає величезним числом, тож одна перевірка `(uint)index >= (uint)Count` ловить і `index < 0`, і `index >= Count`. Цей трюк використовує і сам `List<T>`.

### 2.2 Будуємо самі: повна реалізація `MyList<T>`

Тепер напишемо «справжній» динамічний масив, максимально близький до `List<T>`:

| Член | Що робить | Складність |
|------|-----------|-----------|
| `Add(item)` | додає в кінець | O(1) амортизовано |
| `Insert(index, item)` | вставляє, зсуваючи хвіст праворуч | O(n − index) |
| `RemoveAt(index)` | видаляє, зсуваючи хвіст ліворуч | O(n − index) |
| `Remove(item)` | знаходить і видаляє перше входження | O(n) |
| `this[index]` | читання/запис за індексом | O(1) |
| `IndexOf`, `Contains` | лінійний пошук | O(n) |
| `EnsureCapacity(min)` | гарантує місткість, щоб уникнути багатьох ростів | O(n) при рості |
| `TrimExcess()` | зменшує буфер до `Count` | O(n) |
| `Clear()` | видаляє все (місткість лишається) | O(1) / O(n) для посилань |
| `GetEnumerator()` | `foreach` з перевіркою змін (`_version`) | O(1) на крок |

**Як працює перевірка змін під час `foreach`.** Колекція має лічильник `_version`, який збільшується при кожній модифікації. Енумератор запам'ятовує версію на старті та порівнює її на кожному `MoveNext`. Якщо версії різні — колекцію змінили під час обходу, і продовжувати небезпечно (елементи могли зсунутися), тому кидається `InvalidOperationException`.

```
 Insert(1, 99) у [10, 20, 30, 40], Count = 4, Capacity = 8

 до:    │ 10 │ 20 │ 30 │ 40 │    │ ...
                 └────┴────┘──► Array.Copy(_items, 1, _items, 2, 3)
 зсув:  │ 10 │ 20 │ 20 │ 30 │ 40 │ ...
 запис: │ 10 │ 99 │ 20 │ 30 │ 40 │ ...     Count = 5

 RemoveAt(0) у [10, 99, 20, 30, 40]
 зсув:  │ 99 │ 20 │ 30 │ 40 │ 40 │ ...     Array.Copy(_items, 1, _items, 0, 4)
 чистка:│ 99 │ 20 │ 30 │ 40 │  0 │ ...     _items[4] = default — щоб GC не тримав об'єкт
```

```csharp
using System.Collections;
using System.Runtime.CompilerServices;

// ===== Демонстрація =====
var list = new MyList<int>();
int lastCapacity = -1;
for (int i = 1; i <= 10; i++)
{
    list.Add(i);
    if (list.Capacity != lastCapacity)          // друкуємо лише моменти росту
    {
        Console.WriteLine($"Add({i}): Count={list.Count}, Capacity {lastCapacity} -> {list.Capacity}");
        lastCapacity = list.Capacity;
    }
}
Console.WriteLine($"list: {string.Join(" ", list)}");

list.Insert(0, 0);                              // вставка на початок — зсув усіх елементів
list.Insert(list.Count, 11);                    // вставка в кінець — те саме, що Add
list.RemoveAt(5);                               // видаляємо елемент з індексом 5 (значення 5)
bool removed = list.Remove(7);                  // видаляємо за значенням
list[1] = 100;                                  // запис через індексатор
Console.WriteLine($"after edits: {string.Join(" ", list)} (removed 7: {removed})");
Console.WriteLine($"IndexOf(8)={list.IndexOf(8)}, Contains(5)={list.Contains(5)}, Count={list.Count}");

// Модифікація під час foreach — енумератор помічає зміну версії
try
{
    foreach (int x in list)
    {
        if (x == 3)
            list.Add(-1);                       // змінюємо колекцію посеред обходу
    }
}
catch (InvalidOperationException ex)
{
    Console.WriteLine($"foreach: {ex.Message}");
}

// Некоректний індекс
try
{
    Console.WriteLine(list[list.Count]);
}
catch (ArgumentOutOfRangeException ex)
{
    Console.WriteLine($"indexer: {ex.ParamName} out of range");
}

// Керування місткістю
Console.WriteLine($"EnsureCapacity(100) -> {list.EnsureCapacity(100)}");
list.TrimExcess();
Console.WriteLine($"TrimExcess: Count={list.Count}, Capacity={list.Capacity}");
list.Clear();
Console.WriteLine($"Clear: Count={list.Count}, Capacity={list.Capacity}");

// Працює з будь-яким типом і з LINQ, бо реалізує IEnumerable<T>
var words = new MyList<string>(capacity: 2) { "stack", "queue", "list" };  // ініціалізатор колекції викликає Add
Console.WriteLine($"words: {string.Join(", ", words.OrderBy(w => w))}");

// ===== Реалізація =====

/// <summary>Динамічний масив: буфер T[] + кількість зайнятих комірок.</summary>
public sealed class MyList<T> : IEnumerable<T>
{
    private const int DefaultCapacity = 4;      // перший ріст одразу до 4, як у List<T>

    private T[] _items;                         // буфер; комірки [Count..Capacity) не використовуються
    private int _count;                         // кількість елементів
    private int _version;                       // збільшується при кожній зміні

    public MyList() => _items = [];             // порожній масив — без виділення пам'яті

    public MyList(int capacity)
    {
        ArgumentOutOfRangeException.ThrowIfNegative(capacity);
        _items = capacity == 0 ? [] : new T[capacity];
    }

    public int Count => _count;
    public int Capacity => _items.Length;

    public T this[int index]
    {
        get
        {
            ValidateIndex(index);
            return _items[index];
        }
        set
        {
            ValidateIndex(index);
            _items[index] = value;
            _version++;                         // заміна елемента теж вважається зміною
        }
    }

    public void Add(T item)
    {
        if (_count == _items.Length)            // місця немає
            Grow(_count + 1);
        _items[_count] = item;
        _count++;
        _version++;
    }

    public void Insert(int index, T item)
    {
        // index == Count дозволено: це вставка в кінець
        if ((uint)index > (uint)_count)
            throw new ArgumentOutOfRangeException(nameof(index));
        if (_count == _items.Length)
            Grow(_count + 1);
        if (index < _count)                     // зсуваємо хвіст на одну позицію праворуч
            Array.Copy(_items, index, _items, index + 1, _count - index);
        _items[index] = item;
        _count++;
        _version++;
    }

    public void RemoveAt(int index)
    {
        ValidateIndex(index);
        _count--;
        if (index < _count)                     // зсуваємо хвіст на одну позицію ліворуч
            Array.Copy(_items, index + 1, _items, index, _count - index);
        _items[_count] = default!;              // звільняємо посилання для GC
        _version++;
    }

    public bool Remove(T item)
    {
        int index = IndexOf(item);
        if (index < 0)
            return false;
        RemoveAt(index);
        return true;
    }

    // Array.IndexOf використовує EqualityComparer<T>.Default і шукає лише в [0, Count)
    public int IndexOf(T item) => Array.IndexOf(_items, item, 0, _count);

    public bool Contains(T item) => IndexOf(item) >= 0;

    public void Clear()
    {
        // Для int/double обнуляти не потрібно, для посилань — щоб GC міг зібрати об'єкти
        if (RuntimeHelpers.IsReferenceOrContainsReferences<T>())
            Array.Clear(_items, 0, _count);
        _count = 0;
        _version++;
    }

    public int EnsureCapacity(int capacity)
    {
        ArgumentOutOfRangeException.ThrowIfNegative(capacity);
        if (_items.Length < capacity)
            Grow(capacity);
        return _items.Length;
    }

    public void TrimExcess()
    {
        // Як List<T>: не перевиділяємо, якщо «зайвого» місця менше 10%
        int threshold = (int)(_items.Length * 0.9);
        if (_count < threshold)
            SetCapacity(_count);
    }

    private void Grow(int minCapacity)
    {
        int newCapacity = _items.Length == 0 ? DefaultCapacity : _items.Length * 2;
        if ((uint)newCapacity > (uint)Array.MaxLength)   // захист від переповнення int
            newCapacity = Array.MaxLength;
        if (newCapacity < minCapacity)
            newCapacity = minCapacity;
        SetCapacity(newCapacity);
    }

    private void SetCapacity(int capacity)
    {
        if (capacity == _items.Length)
            return;
        T[] newItems = capacity == 0 ? [] : new T[capacity];
        Array.Copy(_items, newItems, _count);   // O(n) — головна «ціна» росту
        _items = newItems;
    }

    private void ValidateIndex(int index)
    {
        if ((uint)index >= (uint)_count)        // ловить і від'ємні, і завеликі індекси
            throw new ArgumentOutOfRangeException(nameof(index));
    }

    // struct-енумератор: foreach по MyList<T> не виділяє пам'ять у купі
    public Enumerator GetEnumerator() => new(this);
    IEnumerator<T> IEnumerable<T>.GetEnumerator() => GetEnumerator();
    IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();

    public struct Enumerator : IEnumerator<T>
    {
        private readonly MyList<T> _list;
        private readonly int _version;          // версія колекції на момент старту
        private int _index;
        private T _current;

        internal Enumerator(MyList<T> list)
        {
            _list = list;
            _version = list._version;
            _index = 0;
            _current = default!;
        }

        public readonly T Current => _current;
        readonly object? IEnumerator.Current => _current;

        public bool MoveNext()
        {
            if (_version != _list._version)
                throw new InvalidOperationException("Collection was modified during enumeration.");
            if (_index < _list._count)
            {
                _current = _list._items[_index];
                _index++;
                return true;
            }
            _current = default!;
            return false;
        }

        public void Reset()
        {
            if (_version != _list._version)
                throw new InvalidOperationException("Collection was modified during enumeration.");
            _index = 0;
            _current = default!;
        }

        public readonly void Dispose() { }
    }
}
```

**Приклад запуску:**

```
Add(1): Count=1, Capacity -1 -> 4
Add(5): Count=5, Capacity 4 -> 8
Add(9): Count=9, Capacity 8 -> 16
list: 1 2 3 4 5 6 7 8 9 10
after edits: 0 100 2 3 4 6 8 9 10 11 (removed 7: True)
IndexOf(8)=6, Contains(5)=False, Count=10
foreach: Collection was modified during enumeration.
indexer: index out of range
EnsureCapacity(100) -> 100
TrimExcess: Count=11, Capacity=11
Clear: Count=0, Capacity=11
words: list, queue, stack
```

Зверніть увагу: після винятку в `foreach` елемент `-1` **вже доданий** (виняток кидається на наступному `MoveNext`), тому `Count` став 11.

### 2.3 Амортизований аналіз

**Амортизована складність** — це середня вартість однієї операції в **найгіршій послідовності** операцій. Вона не про ймовірності: гарантується, що будь-які m операцій коштують не більше m × (амортизована вартість).

#### Сумарний метод (aggregate)

Нехай ми додаємо n елементів у масив, що починається з місткості 1 і подвоюється. Копіювання відбувається, коли `Count` досягає 1, 2, 4, 8, …, 2ᵏ < n:

```
 копіювання:  1 + 2 + 4 + ... + 2ᵏ  =  2ᵏ⁺¹ − 1  <  2n
 записи:      n
 разом:       < 3n  ⇒  O(n) на n операцій  ⇒  O(1) амортизовано на одну
```

#### Метод бухгалтерського обліку (accounting)

Уявімо, що кожен `Add` «платить» **3 монети**:

- 1 монета — за запис самого елемента;
- 2 монети — кладемо «на рахунок» цього елемента.

```
 Capacity = 4, щойно відбувся ріст до 8 → у буфері 4 «старі» елементи без монет

 │ a │ b │ c │ d │ e │ f │ g │ h │     Add(e..h): кожен кладе по 2 монети = 8 монет
                   $$  $$  $$  $$

 Add(i): Count == Capacity == 8 → копіюємо 8 елементів, це коштує 8 монет.
 На рахунку рівно 8 монет (від e, f, g, h) — ріст повністю оплачений заздалегідь!
```

Після кожного подвоєння половина елементів «нова» і накопичила по 2 монети — цього вистачає, щоб скопіювати **всі** елементи. Баланс ніколи не стає від'ємним, отже кожен `Add` коштує не більше 3 монет = **O(1) амортизовано**.

#### Чому не «+k»?

Якщо збільшувати місткість на сталу величину k, копіювання відбуваються на розмірах k, 2k, 3k, …:

```
 k + 2k + 3k + ... + (n/k)·k  =  k · (n/k)(n/k + 1)/2  ≈  n² / (2k)  ⇒  O(n²)
```

Порахуймо кількість скопійованих елементів для різних стратегій:

```csharp
int[] sizes = [1_000, 10_000, 100_000, 1_000_000];

Console.WriteLine($"{"n",10} | {"x2",10} | {"x1.5",10} | {"+10",14} | {"+1000",12}");
Console.WriteLine(new string('-', 68));
foreach (int n in sizes)
{
    long doubling = CountCopies(n, cap => cap * 2);
    long oneAndHalf = CountCopies(n, cap => Math.Max(cap + 1, cap * 3 / 2));
    long plus10 = CountCopies(n, cap => cap + 10);
    long plus1000 = CountCopies(n, cap => cap + 1000);
    Console.WriteLine($"{n,10} | {doubling,10} | {oneAndHalf,10} | {plus10,14} | {plus1000,12}");
}

// Моделюємо n додавань: коли Count == Capacity, копіюємо Count елементів і ростемо
static long CountCopies(int n, Func<long, long> grow)
{
    long copies = 0;
    long capacity = 1;
    for (long count = 0; count < n; count++)
    {
        if (count == capacity)
        {
            copies += count;                   // вартість копіювання старого буфера
            capacity = grow(capacity);
        }
    }
    return copies;
}
```

**Приклад запуску:**

```
         n |         x2 |       x1.5 |            +10 |        +1000
--------------------------------------------------------------------
      1000 |       1023 |       2137 |          49600 |            1
     10000 |      16383 |      24284 |        4996000 |        45010
    100000 |     131071 |     276521 |      499960000 |      4950100
   1000000 |    1048575 |    2099753 |    49999600000 |    499501000
```

Для мільйона елементів подвоєння копіює ≈ 1 млн елементів, а «+10» — ≈ 50 **мільярдів**. Множник 1.5 економніший за пам'яттю (менше порожнього місця), але копіює трохи більше — обидва дають O(1) амортизовано.

| Стратегія росту | Сумарне копіювання | Амортизовано на `Add` | Макс. порожнього місця |
|-----------------|-------------------|-----------------------|------------------------|
| × 2 (`List<T>`, `Queue<T>`) | < 2n | O(1) | ≈ 50% |
| × 1.5 (MSVC `std::vector`, Java `ArrayList`) | < 3n | O(1) | ≈ 33% |
| + k | ≈ n²/(2k) | O(n/k) = O(n) | k |

> **Важливо:** амортизоване O(1) — це не «кожна операція O(1)». Окремий `Add` може зайняти O(n). Для систем реального часу (ігровий кадр, аудіо) такі «стрибки» бувають неприйнятними — тоді заздалегідь викликають `EnsureCapacity`.

### 2.4 `List<T>` зсередини та його API

`List<T>` влаштований майже так само, як наш `MyList<T>`: поля `_items`, `_size`, `_version`; перший ріст до 4, далі подвоєння. Переконаймося:

```csharp
var list = new List<int>();
Console.WriteLine($"new List<int>(): Capacity={list.Capacity}");
int last = list.Capacity;
for (int i = 0; i < 40; i++)
{
    list.Add(i);
    if (list.Capacity != last)
    {
        Console.WriteLine($"  after Add #{list.Count,2}: Capacity={list.Capacity}");
        last = list.Capacity;
    }
}

var presized = new List<int>(1000);            // одразу виділяємо місце — жодного росту
Console.WriteLine($"new List<int>(1000): Count={presized.Count}, Capacity={presized.Capacity}");

list.Clear();                                  // Count = 0, але буфер лишається!
Console.WriteLine($"after Clear: Count={list.Count}, Capacity={list.Capacity}");
list.TrimExcess();                             // звільняємо пам'ять
Console.WriteLine($"after TrimExcess: Capacity={list.Capacity}");
```

**Приклад запуску:**

```
new List<int>(): Capacity=0
  after Add # 1: Capacity=4
  after Add # 5: Capacity=8
  after Add # 9: Capacity=16
  after Add #17: Capacity=32
  after Add #33: Capacity=64
new List<int>(1000): Count=0, Capacity=1000
after Clear: Count=0, Capacity=64
after TrimExcess: Capacity=0
```

Основні методи `List<T>`:

| Група | Методи | Складність |
|-------|--------|-----------|
| Додавання | `Add`, `AddRange` | O(1) аморт. / O(k) |
| Вставка | `Insert`, `InsertRange` | O(n) |
| Видалення | `RemoveAt`, `Remove`, `RemoveRange`, `RemoveAll(predicate)`, `Clear` | O(n) |
| Пошук | `IndexOf`, `Contains`, `Find`, `FindIndex`, `FindAll`, `Exists`, `TrueForAll` | O(n) |
| Відсортований | `Sort`, `BinarySearch` | O(n log n) / O(log n) |
| Перетворення | `ToArray`, `ConvertAll`, `GetRange`, `Slice`, `AsReadOnly`, `Reverse` | O(n) / O(k) |
| Місткість | `Capacity`, `EnsureCapacity`, `TrimExcess` | O(n) при зміні |

```csharp
List<int> nums = [5, 1, 4, 1, 5, 9, 2, 6];

nums.AddRange([5, 3]);                          // додати кілька елементів
nums.InsertRange(0, [0, 0]);                    // вставити на початок
Console.WriteLine($"start:      {string.Join(" ", nums)}");

int removedOnes = nums.RemoveAll(x => x == 1);  // O(n) за ОДИН прохід (а не n разів Remove)
nums.RemoveRange(0, 2);                         // видалити два нулі на початку
Console.WriteLine($"cleaned:    {string.Join(" ", nums)} (removed {removedOnes} ones)");

Console.WriteLine($"Find(>5)={nums.Find(x => x > 5)}, FindIndex(>5)={nums.FindIndex(x => x > 5)}, FindLast(<5)={nums.FindLast(x => x < 5)}");
Console.WriteLine($"FindAll(odd): {string.Join(" ", nums.FindAll(x => x % 2 == 1))}");
Console.WriteLine($"Exists(==9)={nums.Exists(x => x == 9)}, TrueForAll(>0)={nums.TrueForAll(x => x > 0)}");

List<string> labels = nums.ConvertAll(x => $"#{x}");
Console.WriteLine($"ConvertAll: {string.Join(" ", labels)}");
Console.WriteLine($"GetRange(2,3): {string.Join(" ", nums.GetRange(2, 3))}");

nums.Sort();
Console.WriteLine($"sorted:     {string.Join(" ", nums)}");
nums.Reverse();
Console.WriteLine($"reversed:   {string.Join(" ", nums)}");

IReadOnlyList<int> view = nums.AsReadOnly();    // обгортка без копіювання, змінити не можна
nums[0] = 42;                                   // зміни оригіналу видно через view
Console.WriteLine($"view[0]={view[0]}, view.Count={view.Count}");
```

**Приклад запуску:**

```
start:      0 0 5 1 4 1 5 9 2 6 5 3
cleaned:    5 4 5 9 2 6 5 3 (removed 2 ones)
Find(>5)=9, FindIndex(>5)=3, FindLast(<5)=3
FindAll(odd): 5 5 9 5 3
Exists(==9)=True, TrueForAll(>0)=True
ConvertAll: #5 #4 #5 #9 #2 #6 #5 #3
GetRange(2,3): 5 9 2
sorted:     2 3 4 5 5 5 6 9
reversed:   9 6 5 5 5 4 3 2
view[0]=42, view.Count=8
```

Сортування та бінарний пошук у `List<T>`:

```csharp
List<int> scores = [42, 7, 19, 88, 3, 19];   // collection expression (C# 12)
scores.Sort();                                // сортування на місці, O(n log n)
Console.WriteLine(string.Join(' ', scores));

// BinarySearch працює лише на ВІДСОРТОВАНОМУ списку, O(log n)
int found = scores.BinarySearch(42);
Console.WriteLine($"index of 42 = {found}");
int missing = scores.BinarySearch(20);        // немає → від'ємне число
Console.WriteLine($"20 missing, insert at {~missing}"); // ~ дає позицію для вставки
scores.Insert(~missing, 20);                  // вставка в середину — O(n) через зсув
Console.WriteLine(string.Join(' ', scores));
```

**Приклад запуску:**

```
3 7 19 19 42 88
index of 42 = 4
20 missing, insert at 4
3 7 19 19 20 42 88
```

Список об'єктів: сортування за кількома ключами, пошук за умовою, а також масив байтів як буфер файлу:

```csharp
List<User> users =
[
    new("olena", 31),
    new("taras", 24),
    new("ivan", 28),
];
users.Add(new User("maria", 24));                // додавання в кінець — амортизовано O(1)

// Сортуємо за віком, а при рівному віці — за іменем
users.Sort((a, b) => a.Age != b.Age ? a.Age.CompareTo(b.Age) : string.CompareOrdinal(a.Name, b.Name));
foreach (User u in users)
    Console.WriteLine($"{u.Name} ({u.Age})");

// Лінійний пошук за умовою — O(n)
User? firstAdult = users.Find(u => u.Age > 30);
Console.WriteLine($"older than 30: {firstAdult?.Name}");

// Буфер під файл: весь вміст файлу — один масив байтів
string path = Path.Combine(Path.GetTempPath(), "lecture2.bin");
File.WriteAllBytes(path, [0x48, 0x69, 0x21]);   // записуємо 3 байти ("Hi!")
byte[] buffer = File.ReadAllBytes(path);         // читаємо назад одним викликом
Console.WriteLine($"{buffer.Length} bytes: {Convert.ToHexString(buffer)} = {System.Text.Encoding.ASCII.GetString(buffer)}");
File.Delete(path);

// record — незмінний тип із автоматичними Equals/ToString
public sealed record User(string Name, int Age);
```

**Приклад запуску:**

```
maria (24)
taras (24)
ivan (28)
olena (31)
older than 30: olena
3 bytes: 486921 = Hi!
```

**Коли використовувати `List<T>`:** потрібен доступ за індексом, більшість операцій — додавання в кінець, потрібні `Sort`/`BinarySearch`, важлива ефективність кешу (елементи поруч у пам'яті). Це **вибір за замовчуванням** для послідовності елементів.

**Не варто, коли:** часті вставки/видалення на початку або в середині (O(n)); потрібна черга FIFO (`RemoveAt(0)` — O(n), беріть `Queue<T>`); дуже великі обсяги, де копіювання при розширенні дороге (задайте `Capacity` наперед).

### 2.5 `CollectionsMarshal.AsSpan`: прямий доступ до буфера

Індексатор `List<T>` повертає **копію** елемента. Для `List<struct>` це означає, що змінити поле елемента напряму не можна:

```csharp
// НЕ КОМПІЛЮЄТЬСЯ: CS1612 — не можна змінити значення, що повертається індексатором
List<Point> points = [new Point { X = 1 }];
points[0].X = 10;
public struct Point { public int X; }
```

`CollectionsMarshal.AsSpan(list)` повертає `Span<T>` на **внутрішній буфер** списку — тоді `span[i].X = ...` змінює елемент на місці, а цикл по Span працює без перевірок версії і без копіювання структур.

```csharp
using System.Runtime.InteropServices;

List<Point> points = [new Point(1, 1), new Point(2, 2), new Point(3, 3)];

// Спосіб 1: прочитати копію, змінити, записати назад (3 операції над копією)
Point p = points[0];
p.X = 10;
points[0] = p;

// Спосіб 2: Span на внутрішній масив — змінюємо поля на місці
Span<Point> span = CollectionsMarshal.AsSpan(points);
for (int i = 0; i < span.Length; i++)
    span[i].Y *= 100;                           // жодних копій структур

// ref-змінна на елемент
ref Point last = ref span[^1];
last.X = -3;
Console.WriteLine(string.Join(" ", points));

// SetCount (.NET 8+): встановити Count без запису елементів, далі заповнити через Span
var buffer = new List<int>();
CollectionsMarshal.SetCount(buffer, 5);
Span<int> raw = CollectionsMarshal.AsSpan(buffer);
for (int i = 0; i < raw.Length; i++)
    raw[i] = i * i;
Console.WriteLine($"buffer: {string.Join(" ", buffer)} (Count={buffer.Count})");

// Небезпека: Span дивиться на СТАРИЙ буфер, якщо список виріс
Span<int> stale = CollectionsMarshal.AsSpan(buffer);
buffer.AddRange([25, 36, 49, 64]);              // Capacity була 5 → новий масив
stale[0] = 999;                                 // пишемо в старий масив — список цього не бачить
Console.WriteLine($"buffer[0] after writing via stale span: {buffer[0]}");

public struct Point(int x, int y)
{
    public int X = x;
    public int Y = y;
    public override readonly string ToString() => $"({X},{Y})";
}
```

**Приклад запуску:**

```
(10,100) (2,200) (-3,300)
buffer: 0 1 4 9 16 (Count=5)
buffer[0] after writing via stale span: 0
```

> **Правило:** поки у вас є `Span` з `CollectionsMarshal.AsSpan`, **не додавайте й не видаляйте** елементи списку.

### 2.6 Пастки: модифікація під час `foreach`, місткість, `RemoveAt(0)`

```csharp
using System.Diagnostics;

List<int> values = [1, 2, 3, 4, 5, 6, 7, 8];

// Пастка 1: видалення в foreach → InvalidOperationException
try
{
    foreach (int v in values)
        if (v % 2 == 0)
            values.Remove(v);
}
catch (InvalidOperationException ex)
{
    Console.WriteLine($"foreach+Remove: {ex.Message}");
}

// Правильно А: обхід for ЗАДОМ НАПЕРЕД — видалення не зсуває ще не переглянуті елементи
values = [1, 2, 3, 4, 5, 6, 7, 8];
for (int i = values.Count - 1; i >= 0; i--)
    if (values[i] % 2 == 0)
        values.RemoveAt(i);
Console.WriteLine($"reverse for: {string.Join(" ", values)}");

// Неправильно: for уперед пропускає елемент після видаленого
values = [2, 4, 5, 6];
for (int i = 0; i < values.Count; i++)
    if (values[i] % 2 == 0)
        values.RemoveAt(i);                     // 4 зсувається на місце 2 і не перевіряється
Console.WriteLine($"forward for (bug): {string.Join(" ", values)}");

// Правильно Б: RemoveAll — один прохід, O(n)
values = [1, 2, 3, 4, 5, 6, 7, 8];
values.RemoveAll(v => v % 2 == 0);
Console.WriteLine($"RemoveAll: {string.Join(" ", values)}");

// Пастка 2: List як черга — RemoveAt(0) зсуває ВСІ елементи, n операцій = O(n²)
const int N = 100_000;
var list = new List<int>(Enumerable.Range(0, N));
var sw = Stopwatch.StartNew();
while (list.Count > 0)
    list.RemoveAt(0);
long listMs = sw.ElapsedMilliseconds;

var queue = new Queue<int>(Enumerable.Range(0, N));
sw.Restart();
while (queue.Count > 0)
    queue.Dequeue();                            // O(1)
long queueMs = sw.ElapsedMilliseconds;
Console.WriteLine($"List.RemoveAt(0) x {N}: {listMs} ms, Queue.Dequeue x {N}: {queueMs} ms");

// Пастка 3: Capacity і Count — різні речі
var buffer = new List<int>(10);
try
{
    buffer[0] = 1;                              // Count = 0, хоча місце є!
}
catch (ArgumentOutOfRangeException)
{
    Console.WriteLine("new List<int>(10)[0] = 1 → ArgumentOutOfRangeException (Count is 0)");
}
```

**Приклад запуску:**

```
foreach+Remove: Collection was modified; enumeration operation may not execute.
reverse for: 1 3 5 7
forward for (bug): 4 5
RemoveAll: 1 3 5 7
List.RemoveAt(0) x 100000: … ms, Queue.Dequeue x 100000: … ms
new List<int>(10)[0] = 1 → ArgumentOutOfRangeException (Count is 0)
```

### Типові помилки (розділ 2)

1. **Плутати `Count` і `Capacity`.** `new List<int>(10)` не містить жодного елемента; індексатор перевіряє межі за `Count`.
2. **Змінювати список у `foreach`.** Використовуйте `RemoveAll`, обхід `for` з кінця або збирайте зміни в окремий список.
3. **`RemoveAt(0)` / `Insert(0, x)` у циклі** — квадратична складність. Для FIFO — `Queue<T>`, для двох кінців — дек (розділ 6.3).
4. **Забути, що `Clear()` не звільняє пам'ять.** Великий список, який більше не буде рости, варто `TrimExcess()`.
5. **Не задати місткість, коли розмір відомий.** `new List<T>(n)` економить log₂n перевиділень і до 2n копіювань.
6. **Змінювати поле `struct` через індексатор** — помилка компіляції CS1612 (або тиха зміна копії, якщо це локальна змінна).
7. **Зберігати `Span` з `CollectionsMarshal.AsSpan` після `Add`** — Span дивиться на старий буфер.
8. **Власна реалізація без `_items[_count] = default!` у `RemoveAt`** — «витік» пам'яті: масив тримає посилання на видалений об'єкт.

### Міні-вправи (розділ 2)

**Вправа 2.1.** Додайте до `MyList<T>` метод `AddRange(ReadOnlySpan<T> items)`, який робить **не більше одного** росту буфера. Для перевірки напишіть спрощений клас лише з `Add`, `AddRange`, `Count`, `Capacity` і лічильником перевиділень.

<details>
<summary>Розв'язок</summary>

```csharp
var list = new GrowCounterList<int>();
list.Add(1);                                     // перший ріст: 0 → 4
list.AddRange([2, 3, 4, 5, 6, 7, 8, 9, 10]);     // потрібно 10 → ОДИН ріст до 10
Console.WriteLine($"Count={list.Count}, Capacity={list.Capacity}, reallocations={list.Reallocations}");
list.AddRange([11, 12]);                         // 12 > 10 → ріст до max(20, 12) = 20
Console.WriteLine($"Count={list.Count}, Capacity={list.Capacity}, reallocations={list.Reallocations}");
Console.WriteLine(string.Join(" ", list.ToArray()));

public sealed class GrowCounterList<T>
{
    private T[] _items = [];
    private int _count;

    public int Count => _count;
    public int Capacity => _items.Length;
    public int Reallocations { get; private set; }

    public void Add(T item)
    {
        if (_count == _items.Length)
            Grow(_count + 1);
        _items[_count++] = item;
    }

    public void AddRange(ReadOnlySpan<T> items)
    {
        if (_count + items.Length > _items.Length)
            Grow(_count + items.Length);          // рівно один ріст до потрібного розміру
        items.CopyTo(_items.AsSpan(_count));     // одне копіювання блоком
        _count += items.Length;
    }

    public T[] ToArray() => _items.AsSpan(0, _count).ToArray();

    private void Grow(int min)
    {
        int newCapacity = Math.Max(_items.Length == 0 ? 4 : _items.Length * 2, min);
        Array.Resize(ref _items, newCapacity);
        Reallocations++;
    }
}
```

**Приклад запуску:**

```
Count=10, Capacity=10, reallocations=2
Count=12, Capacity=20, reallocations=3
1 2 3 4 5 6 7 8 9 10 11 12
```

</details>

**Вправа 2.2.** Скільки разів виросте `List<int>`, якщо додати 1000 елементів без задання місткості? *Відповідь:* 4 → 8 → 16 → 32 → 64 → 128 → 256 → 512 → 1024 — **9 разів** (перший — із 0 до 4), а скопійовано 4 + 8 + … + 512 = 1020 елементів.

**Вправа 2.3.** Чому `TrimExcess` не зменшує буфер, якщо заповнено понад 90%? *Відповідь:* виграш пам'яті малий, а перевиділення коштує O(n) і, найімовірніше, список скоро знову виросте.

---

> ## Перерва 1 (≈ 10 хвилин)
>
> Минуло ≈ 60 хвилин. Далі — класичні алгоритмічні прийоми на масивах.

---

## 3. Класичні задачі на масивах

*≈ 30 хвилин*

Більшість задач на масиви (на співбесідах, олімпіадах, у реальному коді) зводяться до кількох **прийомів**. Знаючи їх, ви перетворюєте «очевидний» розв'язок O(n²) на O(n) або O(n log n).

| Прийом | Типова задача | Наївно | З прийомом | Пам'ять |
|--------|---------------|--------|------------|---------|
| Префіксні суми | сума на відрізку, багато запитів | O(n) на запит | O(1) на запит (після O(n)) | O(n) |
| Два вказівники | пара з сумою в відсортованому масиві | O(n²) | O(n) | O(1) |
| Ковзне вікно | найкращий підмасив довжини k / з умовою | O(n·k) | O(n) | O(1)–O(σ) |
| Кадане | підмасив з максимальною сумою | O(n²) | O(n) | O(1) |
| Три розвороти | циклічний зсув на k | O(n·k) | O(n) | O(1) |
| Злиття | об'єднати два відсортовані масиви | O((n+m) log(n+m)) | O(n+m) | O(n+m) |
| Бінарний пошук | перша позиція ≥ x | O(n) | O(log n) | O(1) |

### 3.1 Префіксні суми

**Ідея:** один раз порахувати `prefix[i] = a[0] + … + a[i−1]` (причому `prefix[0] = 0`). Тоді сума на відрізку `[l, r]`:

```
 sum(l..r) = prefix[r + 1] − prefix[l]

 індекс:        0    1    2    3    4    5    6    7
 a:           [ 3,   1,   4,   1,   5,   9,   2,   6 ]
 prefix:  [ 0,  3,   4,   8,   9,  14,  23,  25,  31 ]
            ▲                            ▲
         prefix[2]=4                 prefix[6]=23
 sum(2..5) = 4 + 1 + 5 + 9 = 19 = prefix[6] − prefix[2] = 23 − 4
```

Розширення:
- **Кількість підмасивів із сумою k**: підмасив `(l, r]` має суму k ⇔ `prefix[r] − prefix[l] = k`. Рахуємо, скільки разів кожна префіксна сума вже траплялася, у `Dictionary` — O(n).
- **2D-префікс** для суми прямокутника: `P[r][c]` = сума верхнього лівого прямокутника; сума будь-якого прямокутника — 4 звернення (формула включень-виключень).

```csharp
int[] a = [3, 1, 4, 1, 5, 9, 2, 6];
long[] prefix = BuildPrefix(a);
Console.WriteLine($"prefix: {string.Join(" ", prefix)}");
Console.WriteLine($"sum(2..5) = {RangeSum(prefix, 2, 5)}");
Console.WriteLine($"sum(0..7) = {RangeSum(prefix, 0, 7)}");
Console.WriteLine($"sum(4..4) = {RangeSum(prefix, 4, 4)}");

int[] b = [1, 2, 3, -2, 2, 1, 1];
Console.WriteLine($"subarrays of [{string.Join(",", b)}] with sum 3: {CountSubarraysWithSum(b, 3)}");

int[,] matrix =
{
    { 1, 2, 3 },
    { 4, 5, 6 },
    { 7, 8, 9 },
};
long[,] p2 = BuildPrefix2D(matrix);
Console.WriteLine($"rect (1,1)-(2,2) sum = {RectSum(p2, 1, 1, 2, 2)}");   // 5+6+8+9
Console.WriteLine($"rect (0,0)-(2,2) sum = {RectSum(p2, 0, 0, 2, 2)}");   // уся матриця

// prefix[i] = сума перших i елементів; довжина n + 1, щоб не обробляти l = 0 окремо
static long[] BuildPrefix(int[] values)
{
    var result = new long[values.Length + 1];      // long — щоб сума не переповнилась
    for (int i = 0; i < values.Length; i++)
        result[i + 1] = result[i] + values[i];
    return result;
}

static long RangeSum(long[] prefix, int left, int right) => prefix[right + 1] - prefix[left];

// Скільки підмасивів мають суму target: O(n) часу, O(n) пам'яті
static int CountSubarraysWithSum(int[] values, int target)
{
    var seen = new Dictionary<long, int> { [0] = 1 };  // порожній префікс трапився один раз
    long running = 0;
    int count = 0;
    foreach (int v in values)
    {
        running += v;
        // скільки разів раніше була сума running − target → стільки підмасивів закінчуються тут
        if (seen.TryGetValue(running - target, out int times))
            count += times;
        seen[running] = seen.GetValueOrDefault(running) + 1;
    }
    return count;
}

// P[r + 1, c + 1] = сума прямокутника від (0,0) до (r,c)
static long[,] BuildPrefix2D(int[,] m)
{
    int rows = m.GetLength(0), cols = m.GetLength(1);
    var p = new long[rows + 1, cols + 1];
    for (int r = 0; r < rows; r++)
        for (int c = 0; c < cols; c++)
            p[r + 1, c + 1] = m[r, c] + p[r, c + 1] + p[r + 1, c] - p[r, c];
    return p;
}

static long RectSum(long[,] p, int r1, int c1, int r2, int c2) =>
    p[r2 + 1, c2 + 1] - p[r1, c2 + 1] - p[r2 + 1, c1] + p[r1, c1];
```

**Приклад запуску:**

```
prefix: 0 3 4 8 9 14 23 25 31
sum(2..5) = 19
sum(0..7) = 31
sum(4..4) = 5
subarrays of [1,2,3,-2,2,1,1] with sum 3: 5
rect (1,1)-(2,2) sum = 28
rect (0,0)-(2,2) sum = 45
```

### 3.2 Два вказівники

**Ідея:** два індекси рухаються масивом (назустріч один одному або в одному напрямку), і на кожному кроці хоча б один з них зсувається. Разом вони проходять не більше 2n кроків — **O(n)**.

```
 Пара з сумою 13 у відсортованому [1, 3, 4, 6, 8, 11]:

  L                   R
 [1, 3, 4, 6, 8, 11]      1 + 11 = 12 < 13 → L++ (потрібна більша сума)
     L                R
 [1, 3, 4, 6, 8, 11]      3 + 11 = 14 > 13 → R-- (потрібна менша сума)
     L             R
 [1, 3, 4, 6, 8, 11]      3 + 8  = 11 < 13 → L++
        L          R
 [1, 3, 4, 6, 8, 11]      4 + 8  = 12 < 13 → L++
           L       R
 [1, 3, 4, 6, 8, 11]      6 + 8  = 14 > 13 → R--
           L  R
                          6 + 6 не рахується: L == R → зупинка, пари з сумою 13 немає
```

Два режими:

- **Назустріч** (`left = 0`, `right = n − 1`): пара з сумою, паліндром, розворот, «контейнер з водою».
- **В одному напрямку** (`slow`, `fast`): видалення дублікатів, перенесення нулів у кінець, фільтрація на місці. `slow` — куди писати, `fast` — що читаємо.

```
 Видалення дублікатів з [1, 1, 2, 2, 2, 3]:
  slow = 1 (перший елемент завжди лишається)
  fast=1: a[1]=1 == a[0]      → пропуск
  fast=2: a[2]=2 != a[slow-1] → a[1]=2, slow=2
  fast=3,4: 2 == a[1]         → пропуск
  fast=5: a[5]=3 != 2         → a[2]=3, slow=3
  результат: перші 3 елементи [1, 2, 3]
```

```csharp
int[] sorted = [1, 3, 4, 6, 8, 11];
Console.WriteLine($"pair with sum 10: {FormatPair(FindPairWithSum(sorted, 10))}");
Console.WriteLine($"pair with sum 13: {FormatPair(FindPairWithSum(sorted, 13))}");

int[] withDuplicates = [1, 1, 2, 2, 2, 3, 5, 5];
int unique = RemoveDuplicatesSorted(withDuplicates);
Console.WriteLine($"unique count = {unique}: {string.Join(" ", withDuplicates[..unique])}");

int[] zeros = [0, 1, 0, 3, 12, 0, 7];
MoveZerosToEnd(zeros);
Console.WriteLine($"move zeros: {string.Join(" ", zeros)}");

Console.WriteLine($"'A man, a plan, a canal: Panama' palindrome? {IsPalindrome("A man, a plan, a canal: Panama")}");
Console.WriteLine($"'hello' palindrome? {IsPalindrome("hello")}");

int[] heights = [1, 8, 6, 2, 5, 4, 8, 3, 7];
Console.WriteLine($"max water area = {MaxArea(heights)}");

static string FormatPair((int Left, int Right)? pair) =>
    pair is { } p ? $"indices {p.Left} and {p.Right}" : "none";

// Назустріч: сума замала → лівий праворуч; завелика → правий ліворуч
static (int Left, int Right)? FindPairWithSum(int[] a, int target)
{
    int left = 0, right = a.Length - 1;
    while (left < right)
    {
        int sum = a[left] + a[right];
        if (sum == target)
            return (left, right);
        if (sum < target)
            left++;
        else
            right--;
    }
    return null;
}

// В одному напрямку: slow — довжина «чистої» частини
static int RemoveDuplicatesSorted(int[] a)
{
    if (a.Length == 0)
        return 0;
    int slow = 1;
    for (int fast = 1; fast < a.Length; fast++)
    {
        if (a[fast] != a[slow - 1])   // новий елемент — дописуємо в чисту частину
            a[slow++] = a[fast];
    }
    return slow;
}

// Ненульові елементи зберігають порядок, нулі — в кінці
static void MoveZerosToEnd(int[] a)
{
    int write = 0;
    for (int read = 0; read < a.Length; read++)
    {
        if (a[read] != 0)
        {
            (a[write], a[read]) = (a[read], a[write]);
            write++;
        }
    }
}

// Ігноруємо все, крім літер і цифр, регістр не важливий
static bool IsPalindrome(string s)
{
    int left = 0, right = s.Length - 1;
    while (left < right)
    {
        if (!char.IsLetterOrDigit(s[left])) { left++; continue; }
        if (!char.IsLetterOrDigit(s[right])) { right--; continue; }
        if (char.ToLowerInvariant(s[left]) != char.ToLowerInvariant(s[right]))
            return false;
        left++;
        right--;
    }
    return true;
}

// Площа = ширина × нижча стінка. Рухаємо нижчу стінку — лише так площа може зрости
static int MaxArea(int[] h)
{
    int left = 0, right = h.Length - 1, best = 0;
    while (left < right)
    {
        best = Math.Max(best, (right - left) * Math.Min(h[left], h[right]));
        if (h[left] < h[right])
            left++;
        else
            right--;
    }
    return best;
}
```

**Приклад запуску:**

```
pair with sum 10: indices 2 and 3
pair with sum 13: none
unique count = 4: 1 2 3 5
move zeros: 1 3 12 7 0 0 0
'A man, a plan, a canal: Panama' palindrome? True
'hello' palindrome? False
max water area = 49
```

### 3.3 Ковзне вікно

**Ідея:** підтримувати «вікно» `[left, right]` і його агрегат (суму, лічильники символів). Коли вікно зсувається на 1, агрегат оновлюється за O(1): **додаємо** елемент, що увійшов, і **віднімаємо** той, що вийшов.

```
 Максимальна сума вікна довжини k = 3 у [2, 1, 5, 1, 3, 2]:

 [2, 1, 5], 1, 3, 2      sum = 8
  2,[1, 5, 1], 3, 2      sum = 8 − 2 + 1 = 7
  2, 1,[5, 1, 3], 2      sum = 7 − 1 + 3 = 9   ← максимум
  2, 1, 5,[1, 3, 2]      sum = 9 − 5 + 2 = 6
```

Два типи вікна:

| Тип | Як рухається | Приклади |
|-----|--------------|----------|
| **Фіксоване** (довжина k) | `right++`, `left++` разом | макс. сума k елементів, середнє k останніх вимірів |
| **Змінне** | `right++` завжди; `left++` поки умова порушена | найдовший підрядок без повторів, найкоротший підмасив із сумою ≥ S |

```csharp
int[] values = [2, 1, 5, 1, 3, 2];
Console.WriteLine($"max sum of 3 consecutive = {MaxSumFixedWindow(values, 3)}");

Console.WriteLine($"longest unique substring in 'abcabcbb' = {LongestUniqueSubstring("abcabcbb")}");
Console.WriteLine($"longest unique substring in 'pwwkew'   = {LongestUniqueSubstring("pwwkew")}");

int[] positives = [2, 3, 1, 2, 4, 3];
Console.WriteLine($"shortest subarray with sum >= 7 has length {MinLengthWithSumAtLeast(positives, 7)}");
Console.WriteLine($"shortest subarray with sum >= 100 has length {MinLengthWithSumAtLeast(positives, 100)}");

double[] temperatures = [20, 22, 21, 25, 30, 28];
Console.WriteLine($"moving average (3): {string.Join(" ", MovingAverage(temperatures, 3).Select(x => x.ToString("F2")))}");

// Фіксоване вікно: O(n) замість O(n·k)
static int MaxSumFixedWindow(int[] a, int k)
{
    int windowSum = 0;
    for (int i = 0; i < k; i++)
        windowSum += a[i];                 // перше вікно
    int best = windowSum;
    for (int right = k; right < a.Length; right++)
    {
        windowSum += a[right] - a[right - k]; // увійшов a[right], вийшов a[right − k]
        best = Math.Max(best, windowSum);
    }
    return best;
}

// Змінне вікно: розширюємо праворуч, а при повторі стискаємо зліва
static int LongestUniqueSubstring(string s)
{
    var lastSeen = new Dictionary<char, int>(); // символ → остання позиція
    int left = 0, best = 0;
    for (int right = 0; right < s.Length; right++)
    {
        // якщо символ вже є всередині вікна — пересуваємо left за нього
        if (lastSeen.TryGetValue(s[right], out int previous) && previous >= left)
            left = previous + 1;
        lastSeen[s[right]] = right;
        best = Math.Max(best, right - left + 1);
    }
    return best;
}

// Працює лише для невід'ємних чисел: розширення вікна не зменшує суму
static int MinLengthWithSumAtLeast(int[] a, int target)
{
    int left = 0, sum = 0, best = int.MaxValue;
    for (int right = 0; right < a.Length; right++)
    {
        sum += a[right];
        while (sum >= target)                   // вікно задовольняє умову — пробуємо стиснути
        {
            best = Math.Min(best, right - left + 1);
            sum -= a[left++];
        }
    }
    return best == int.MaxValue ? 0 : best;
}

static double[] MovingAverage(double[] a, int k)
{
    var result = new double[a.Length - k + 1];
    double sum = 0;
    for (int i = 0; i < a.Length; i++)
    {
        sum += a[i];
        if (i >= k)
            sum -= a[i - k];
        if (i >= k - 1)
            result[i - k + 1] = sum / k;
    }
    return result;
}
```

**Приклад запуску:**

```
max sum of 3 consecutive = 9
longest unique substring in 'abcabcbb' = 3
longest unique substring in 'pwwkew'   = 3
shortest subarray with sum >= 7 has length 2
shortest subarray with sum >= 100 has length 0
moving average (3): 21.00 22.67 25.33 27.67
```

### 3.4 Алгоритм Кадане: максимальна сума підмасиву

**Задача:** знайти неперервний підмасив із максимальною сумою. Наївно — перебрати всі пари `(l, r)`: O(n²) (або O(n³) без префіксних сум).

**Ідея Кадане (динамічне програмування):** нехай `best_ending_here` — найкраща сума підмасиву, що **закінчується** в позиції i. Тоді:

```
 best_ending_here(i) = max( a[i],  best_ending_here(i−1) + a[i] )
                            ▲                ▲
                  почати заново        продовжити попередній
```

Якщо попередня сума від'ємна — вона тільки заважає, і вигідніше почати новий підмасив.

```
 a:                −2    1   −3    4   −1    2    1   −5    4
 ending_here:      −2    1   −2    4    3    5    6    1    5
 best so far:      −2    1    1    4    4    5    6    6    6
                                   └──────────────┘
                                   підмасив [4, −1, 2, 1] = 6
```

```csharp
int[] a = [-2, 1, -3, 4, -1, 2, 1, -5, 4];
var (sum, start, end) = MaxSubarray(a);
Console.WriteLine($"max sum = {sum}, indices {start}..{end}: [{string.Join(", ", a[start..(end + 1)])}]");

int[] allNegative = [-8, -3, -6, -2, -5, -4];
var neg = MaxSubarray(allNegative);
Console.WriteLine($"all negative: max sum = {neg.Sum}, indices {neg.Start}..{neg.End}");

int[] prices = [7, 1, 5, 3, 6, 4];
Console.WriteLine($"best stock profit = {MaxProfit(prices)}");

// O(n) часу, O(1) пам'яті; повертає також межі підмасиву
static (long Sum, int Start, int End) MaxSubarray(int[] a)
{
    long bestSum = a[0], endingHere = a[0];
    int bestStart = 0, bestEnd = 0, currentStart = 0;
    for (int i = 1; i < a.Length; i++)
    {
        if (endingHere < 0)             // попередній «хвіст» лише зменшує суму
        {
            endingHere = a[i];
            currentStart = i;
        }
        else
        {
            endingHere += a[i];
        }

        if (endingHere > bestSum)
        {
            bestSum = endingHere;
            bestStart = currentStart;
            bestEnd = i;
        }
    }
    return (bestSum, bestStart, bestEnd);
}

// Та сама ідея: мінімум ціни до сьогодні → найкращий прибуток при продажу сьогодні
static int MaxProfit(int[] prices)
{
    int minPrice = int.MaxValue, best = 0;
    foreach (int price in prices)
    {
        minPrice = Math.Min(minPrice, price);
        best = Math.Max(best, price - minPrice);
    }
    return best;
}
```

**Приклад запуску:**

```
max sum = 6, indices 3..6: [4, -1, 2, 1]
all negative: max sum = -2, indices 3..3
best stock profit = 5
```

### 3.5 Розворот і циклічний зсув на місці

**Розворот:** два вказівники назустріч, обмін — O(n) часу, O(1) пам'яті.

**Циклічний зсув праворуч на k** «трьома розворотами»:

```
 a = [1, 2, 3, 4, 5, 6, 7], k = 3

 1) розвернути все:         [7, 6, 5, 4, 3, 2, 1]
 2) розвернути перші k:     [5, 6, 7 | 4, 3, 2, 1]
 3) розвернути решту:       [5, 6, 7 | 1, 2, 3, 4]   готово
```

Чому це працює: зсув праворуч на k — це «перенести останні k елементів на початок». Перший розворот ставить їх на початок, але задом наперед; два наступні виправляють порядок усередині кожної частини.

```csharp
int[] a = [1, 2, 3, 4, 5, 6, 7];
Reverse(a, 0, a.Length - 1);
Console.WriteLine($"reversed:      {string.Join(" ", a)}");

a = [1, 2, 3, 4, 5, 6, 7];
RotateRight(a, 3);
Console.WriteLine($"rotate right 3: {string.Join(" ", a)}");

a = [1, 2, 3, 4, 5, 6, 7];
RotateRight(a, 10);                          // 10 % 7 = 3 — те саме
Console.WriteLine($"rotate right 10: {string.Join(" ", a)}");

a = [1, 2, 3, 4, 5, 6, 7];
RotateLeft(a, 2);
Console.WriteLine($"rotate left 2:  {string.Join(" ", a)}");

// Вбудовані засоби: Span.Reverse і копіювання через проміжний масив (O(k) пам'яті)
int[] b = [1, 2, 3, 4, 5];
b.AsSpan(1, 3).Reverse();
Console.WriteLine($"span reverse:  {string.Join(" ", b)}");

static void Reverse(int[] a, int left, int right)
{
    while (left < right)
    {
        (a[left], a[right]) = (a[right], a[left]);
        left++;
        right--;
    }
}

static void RotateRight(int[] a, int k)
{
    int n = a.Length;
    if (n == 0)
        return;
    k %= n;                                  // зсув на n — тотожний
    Reverse(a, 0, n - 1);
    Reverse(a, 0, k - 1);
    Reverse(a, k, n - 1);
}

// Зсув ліворуч на k = зсув праворуч на n − k
static void RotateLeft(int[] a, int k) => RotateRight(a, a.Length - k % a.Length);
```

**Приклад запуску:**

```
reversed:      7 6 5 4 3 2 1
rotate right 3: 5 6 7 1 2 3 4
rotate right 10: 5 6 7 1 2 3 4
rotate left 2:  3 4 5 6 7 1 2
span reverse:  1 4 3 2 5
```

### 3.6 Злиття двох відсортованих масивів

Основа сортування злиттям (Merge Sort). Два вказівники, на кожному кроці беремо менший з поточних елементів:

```
 A = [1, 4, 7]     B = [2, 3, 8, 9]
      i                 j
 1 < 2 → 1       result = [1]
 4 > 2 → 2       result = [1, 2]
 4 > 3 → 3       result = [1, 2, 3]
 4 < 8 → 4       result = [1, 2, 3, 4]
 7 < 8 → 7       result = [1, 2, 3, 4, 7]
 A закінчився → дописуємо решту B: [1, 2, 3, 4, 7, 8, 9]
```

**Варіант «на місці»:** перший масив має в кінці вільне місце під другий. Заповнюємо **з кінця** — тоді ми ніколи не затираємо ще не оброблені елементи.

```csharp
int[] a = [1, 4, 7];
int[] b = [2, 3, 8, 9];
Console.WriteLine($"merge: {string.Join(" ", Merge(a, b))}");

// nums1 має місце під nums2 (нулі в кінці)
int[] nums1 = [1, 2, 3, 0, 0, 0];
int[] nums2 = [2, 5, 6];
MergeInPlace(nums1, 3, nums2);
Console.WriteLine($"merge in place: {string.Join(" ", nums1)}");

// Перетин двох відсортованих масивів — той самий прийом
Console.WriteLine($"intersection: {string.Join(" ", Intersect([1, 2, 2, 3, 5, 8], [2, 2, 5, 7, 8]))}");

// Стабільне злиття: при рівних значеннях першим іде елемент з a
static int[] Merge(int[] a, int[] b)
{
    var result = new int[a.Length + b.Length];
    int i = 0, j = 0, k = 0;
    while (i < a.Length && j < b.Length)
        result[k++] = a[i] <= b[j] ? a[i++] : b[j++];
    while (i < a.Length)
        result[k++] = a[i++];              // залишок a
    while (j < b.Length)
        result[k++] = b[j++];              // залишок b
    return result;
}

static void MergeInPlace(int[] nums1, int m, int[] nums2)
{
    int i = m - 1, j = nums2.Length - 1, write = m + nums2.Length - 1;
    while (j >= 0)                          // коли nums2 вичерпано — решта nums1 вже на місці
    {
        if (i >= 0 && nums1[i] > nums2[j])
            nums1[write--] = nums1[i--];
        else
            nums1[write--] = nums2[j--];
    }
}

static List<int> Intersect(int[] a, int[] b)
{
    var result = new List<int>();
    int i = 0, j = 0;
    while (i < a.Length && j < b.Length)
    {
        if (a[i] < b[j]) i++;
        else if (a[i] > b[j]) j++;
        else { result.Add(a[i]); i++; j++; }
    }
    return result;
}
```

**Приклад запуску:**

```
merge: 1 2 3 4 7 8 9
merge in place: 1 2 2 3 5 6
intersection: 2 2 5 8
```

### 3.7 Варіанти бінарного пошуку

Бінарний пошук — одна з найпростіших ідей і водночас джерело найбільшої кількості помилок «на одиницю». Надійний підхід — мислити не «знайти x», а **«знайти першу позицію, де умова стає істинною»**.

```
 a = [1, 2, 4, 4, 4, 7, 9],  шукаємо 4

 індекс:    0  1  2  3  4  5  6  7(=n)
 a:         1  2  4  4  4  7  9
 a[i] >= 4: F  F  T  T  T  T  T        lower_bound(4) = 2  — перша позиція з a[i] >= 4
 a[i] >  4: F  F  F  F  F  T  T        upper_bound(4) = 5  — перша позиція з a[i] > 4

 кількість четвірок = upper − lower = 5 − 2 = 3
```

Шаблон на **напіввідкритому інтервалі** `[lo, hi)`:

```
 lo = 0, hi = n
 поки lo < hi:
     mid = lo + (hi − lo) / 2      ← не (lo + hi) / 2: захист від переповнення
     якщо умова(mid) істинна:  hi = mid      (відповідь ≤ mid)
     інакше:                    lo = mid + 1  (відповідь > mid)
 відповідь: lo  (може дорівнювати n — «такої позиції немає»)
```

| Функція | Умова | Смисл результату |
|---------|-------|------------------|
| `LowerBound(x)` | `a[mid] >= x` | перший елемент ≥ x; позиція для вставки x перед рівними |
| `UpperBound(x)` | `a[mid] > x` | перший елемент > x; позиція для вставки x після рівних |
| `Contains(x)` | `lo < n && a[lo] == x` після LowerBound | чи є x |
| Бінарний пошук за відповіддю | монотонний предикат `f(mid)` | мінімальне значення, для якого f істинний |

```csharp
int[] a = [1, 2, 4, 4, 4, 7, 9];

Console.WriteLine($"Classic(7) = {Classic(a, 7)}, Classic(5) = {Classic(a, 5)}");
Console.WriteLine($"LowerBound(4) = {LowerBound(a, 4)}, UpperBound(4) = {UpperBound(a, 4)}");
Console.WriteLine($"count of 4 = {UpperBound(a, 4) - LowerBound(a, 4)}");
Console.WriteLine($"LowerBound(5) = {LowerBound(a, 5)}  (insert position for 5)");
Console.WriteLine($"LowerBound(10) = {LowerBound(a, 10)} (n — all elements are smaller)");
Console.WriteLine($"Array.BinarySearch(4) = {Array.BinarySearch(a, 4)} (any of the equal elements!)");

// Бінарний пошук за відповіддю: ціла частина квадратного кореня
Console.WriteLine($"IntSqrt(17) = {IntSqrt(17)}, IntSqrt(1_000_000_007) = {IntSqrt(1_000_000_007)}");

// Пошук у циклічно зсунутому відсортованому масиві
int[] rotated = [15, 18, 22, 3, 6, 9, 12];
Console.WriteLine($"rotation point = {FindRotationPoint(rotated)} (minimum = {rotated[FindRotationPoint(rotated)]})");

// Класика: індекс x або −1
static int Classic(int[] a, int x)
{
    int lo = 0, hi = a.Length - 1;          // закритий інтервал [lo, hi]
    while (lo <= hi)
    {
        int mid = lo + (hi - lo) / 2;
        if (a[mid] == x) return mid;
        if (a[mid] < x) lo = mid + 1;
        else hi = mid - 1;
    }
    return -1;
}

// Перша позиція i, де a[i] >= x (напіввідкритий інтервал [lo, hi))
static int LowerBound(int[] a, int x)
{
    int lo = 0, hi = a.Length;
    while (lo < hi)
    {
        int mid = lo + (hi - lo) / 2;
        if (a[mid] >= x) hi = mid;
        else lo = mid + 1;
    }
    return lo;
}

// Перша позиція i, де a[i] > x
static int UpperBound(int[] a, int x)
{
    int lo = 0, hi = a.Length;
    while (lo < hi)
    {
        int mid = lo + (hi - lo) / 2;
        if (a[mid] > x) hi = mid;
        else lo = mid + 1;
    }
    return lo;
}

// Найбільше r, для якого r·r <= n  ⇔  (перше r, де r·r > n) − 1
static long IntSqrt(long n)
{
    long lo = 0, hi = n + 1;
    while (lo < hi)
    {
        long mid = lo + (hi - lo) / 2;
        if (mid * mid > n) hi = mid;       // монотонний предикат
        else lo = mid + 1;
    }
    return lo - 1;
}

// Перший елемент, менший за останній, — це і є мінімум (точка зсуву)
static int FindRotationPoint(int[] a)
{
    int lo = 0, hi = a.Length - 1;
    while (lo < hi)
    {
        int mid = lo + (hi - lo) / 2;
        if (a[mid] > a[hi]) lo = mid + 1;  // мінімум праворуч від mid
        else hi = mid;                      // мінімум у [lo, mid]
    }
    return lo;
}
```

**Приклад запуску:**

```
Classic(7) = 5, Classic(5) = -1
LowerBound(4) = 2, UpperBound(4) = 5
count of 4 = 3
LowerBound(5) = 5  (insert position for 5)
LowerBound(10) = 7 (n — all elements are smaller)
Array.BinarySearch(4) = 3 (any of the equal elements!)
IntSqrt(17) = 4, IntSqrt(1_000_000_007) = 31622
rotation point = 3 (minimum = 3)
```

> **Класична помилка:** `int mid = (lo + hi) / 2;` — при `lo + hi > int.MaxValue` виникає переповнення і від'ємний індекс. Саме такий баг роками жив у `Arrays.binarySearch` у Java.

### 3.8 Обходи матриці: спіраль і поворот на 90°

**Спіральний обхід** — підтримуємо чотири межі `top`, `bottom`, `left`, `right` і «обгортаємо» матрицю шарами:

```
  → → → ↓        top    = 0 → йдемо праворуч, потім top++
  ↑ → ↓ ↓        right  = 3 → йдемо вниз, потім right--
  ↑ ← ← ↓        bottom = 2 → йдемо ліворуч, потім bottom--
                 left   = 0 → йдемо вгору, потім left++
 1  2  3  4
 5  6  7  8      спіраль: 1 2 3 4 8 12 11 10 9 5 6 7
 9 10 11 12
```

**Поворот квадратної матриці на 90° за годинниковою стрілкою на місці** = транспонування + розворот кожного рядка:

```
 1 2 3      транспонування   1 4 7     розворот рядків   7 4 1
 4 5 6     ───────────────►  2 5 8    ────────────────►  8 5 2
 7 8 9      (a[r,c]↔a[c,r])  3 6 9                       9 6 3
```

```csharp
int[,] m =
{
    { 1, 2, 3, 4 },
    { 5, 6, 7, 8 },
    { 9, 10, 11, 12 },
};
Console.WriteLine($"spiral: {string.Join(" ", Spiral(m))}");

int[,] square =
{
    { 1, 2, 3 },
    { 4, 5, 6 },
    { 7, 8, 9 },
};
RotateClockwise(square);
Console.WriteLine("rotated 90°:");
Print(square);

// Діагональний обхід: усі клітинки з однаковою сумою r + c лежать на одній діагоналі
Console.WriteLine($"anti-diagonals: {string.Join(" | ", AntiDiagonals(m).Select(d => string.Join(",", d)))}");

// Пошук у матриці, відсортованій по рядках і стовпцях: старт у правому верхньому куті, O(rows + cols)
int[,] sorted =
{
    { 1, 4, 7, 11 },
    { 2, 5, 8, 12 },
    { 3, 6, 9, 16 },
};
Console.WriteLine($"search 5: {Search(sorted, 5)}, search 10: {Search(sorted, 10)}");

static List<int> Spiral(int[,] m)
{
    var result = new List<int>(m.Length);
    int top = 0, bottom = m.GetLength(0) - 1;
    int left = 0, right = m.GetLength(1) - 1;
    while (top <= bottom && left <= right)
    {
        for (int c = left; c <= right; c++) result.Add(m[top, c]);        // праворуч
        top++;
        for (int r = top; r <= bottom; r++) result.Add(m[r, right]);      // вниз
        right--;
        if (top <= bottom)                                                // ще є рядок
        {
            for (int c = right; c >= left; c--) result.Add(m[bottom, c]); // ліворуч
            bottom--;
        }
        if (left <= right)                                                // ще є стовпець
        {
            for (int r = bottom; r >= top; r--) result.Add(m[r, left]);   // вгору
            left++;
        }
    }
    return result;
}

static void RotateClockwise(int[,] m)
{
    int n = m.GetLength(0);
    for (int r = 0; r < n; r++)                  // транспонування: обмін відносно діагоналі
        for (int c = r + 1; c < n; c++)
            (m[r, c], m[c, r]) = (m[c, r], m[r, c]);
    for (int r = 0; r < n; r++)                  // розворот кожного рядка
        for (int c = 0; c < n / 2; c++)
            (m[r, c], m[r, n - 1 - c]) = (m[r, n - 1 - c], m[r, c]);
}

static List<List<int>> AntiDiagonals(int[,] m)
{
    int rows = m.GetLength(0), cols = m.GetLength(1);
    var result = new List<List<int>>();
    for (int sum = 0; sum <= rows + cols - 2; sum++)
    {
        var diagonal = new List<int>();
        for (int r = Math.Max(0, sum - cols + 1); r <= Math.Min(rows - 1, sum); r++)
            diagonal.Add(m[r, sum - r]);
        result.Add(diagonal);
    }
    return result;
}

static bool Search(int[,] m, int target)
{
    int r = 0, c = m.GetLength(1) - 1;
    while (r < m.GetLength(0) && c >= 0)
    {
        if (m[r, c] == target) return true;
        if (m[r, c] > target) c--;               // увесь стовпець нижче ще більший
        else r++;                                // увесь рядок лівіше ще менший
    }
    return false;
}

static void Print(int[,] m)
{
    for (int r = 0; r < m.GetLength(0); r++)
    {
        var row = new int[m.GetLength(1)];
        for (int c = 0; c < row.Length; c++)
            row[c] = m[r, c];
        Console.WriteLine("  " + string.Join(" ", row));
    }
}
```

**Приклад запуску:**

```
spiral: 1 2 3 4 8 12 11 10 9 5 6 7
rotated 90°:
  7 4 1
  8 5 2
  9 6 3
anti-diagonals: 1 | 2,5 | 3,6,9 | 4,7,10 | 8,11 | 12
search 5: True, search 10: False
```

### Типові помилки (розділ 3)

1. **Префіксний масив довжини n замість n + 1** — доводиться окремо обробляти `l = 0`, і легко помилитися на одиницю.
2. **Переповнення `int` у сумах** — префіксні суми та суми вікон краще тримати в `long`.
3. **Ковзне вікно зі змінною довжиною на від'ємних числах** — стискання вікна вже не гарантує зменшення суми; потрібні префіксні суми + словник.
4. **Кадане, що повертає 0 для масиву з лише від'ємних чисел** — ініціалізуйте `best = a[0]`, а не 0 (якщо порожній підмасив заборонено).
5. **`k` більше за довжину при зсуві** — не забудьте `k %= n`.
6. **Бінарний пошук:** змішування закритого `[lo, hi]` і напіввідкритого `[lo, hi)` інтервалів в одному циклі; `(lo + hi) / 2`; нескінченний цикл через `lo = mid` замість `lo = mid + 1`.
7. **Спіраль без перевірок `top <= bottom` / `left <= right`** — у неквадратних матрицях елементи дублюються.

### Міні-вправи (розділ 3)

**Вправа 3.1.** Для масиву `[1, 0, 1, 0, 1]` і цілі 2 знайдіть кількість підмасивів із сумою 2 двома способами: наївним O(n²) і префіксними сумами O(n). Порівняйте результати на кількох масивах.

<details>
<summary>Розв'язок</summary>

```csharp
int[][] tests = [[1, 0, 1, 0, 1], [1, 1, 1], [3, 4, -7, 1, 3, 3, 1, -4]];
int[] targets = [2, 2, 7];

for (int t = 0; t < tests.Length; t++)
    Console.WriteLine($"[{string.Join(",", tests[t])}] sum={targets[t]}: naive={Naive(tests[t], targets[t])}, prefix={WithPrefix(tests[t], targets[t])}");

static int Naive(int[] a, int k)
{
    int count = 0;
    for (int l = 0; l < a.Length; l++)
    {
        int sum = 0;
        for (int r = l; r < a.Length; r++)   // сума a[l..r] нарощується за O(1)
        {
            sum += a[r];
            if (sum == k) count++;
        }
    }
    return count;
}

static int WithPrefix(int[] a, int k)
{
    var seen = new Dictionary<int, int> { [0] = 1 };
    int running = 0, count = 0;
    foreach (int v in a)
    {
        running += v;
        count += seen.GetValueOrDefault(running - k);
        seen[running] = seen.GetValueOrDefault(running) + 1;
    }
    return count;
}
```

**Приклад запуску:**

```
[1,0,1,0,1] sum=2: naive=4, prefix=4
[1,1,1] sum=2: naive=2, prefix=2
[3,4,-7,1,3,3,1,-4] sum=7: naive=4, prefix=4
```

</details>

**Вправа 3.2.** Знайдіть у відсортованому масиві першу й останню позицію значення x (або `-1 -1`) за O(log n).

<details>
<summary>Розв'язок</summary>

```csharp
int[] a = [5, 7, 7, 8, 8, 10];
foreach (int x in new[] { 8, 6, 5, 10 })
{
    var (first, last) = FirstAndLast(a, x);
    Console.WriteLine($"x={x}: first={first}, last={last}");
}

static (int First, int Last) FirstAndLast(int[] a, int x)
{
    int lower = LowerBound(a, x);
    if (lower == a.Length || a[lower] != x)
        return (-1, -1);                       // значення немає
    return (lower, LowerBound(a, x + 1) - 1);   // upper_bound(x) = lower_bound(x + 1) для цілих
}

static int LowerBound(int[] a, int x)
{
    int lo = 0, hi = a.Length;
    while (lo < hi)
    {
        int mid = lo + (hi - lo) / 2;
        if (a[mid] >= x) hi = mid;
        else lo = mid + 1;
    }
    return lo;
}
```

**Приклад запуску:**

```
x=8: first=3, last=4
x=6: first=-1, last=-1
x=5: first=0, last=0
x=10: first=5, last=5
```

</details>

**Вправа 3.3.** Поверніть матрицю на 90° **проти** годинникової стрілки. *Підказка:* транспонування + розворот кожного **стовпця** (або розворот рядків, а потім транспонування).

---

## 4. Зв'язні списки

*≈ 25 хвилин*

### 4.1 Однозв'язний список

**Зв'язний список** — послідовність **вузлів**, де кожен вузол зберігає значення і посилання на наступний вузол. Вузли можуть лежати будь-де в купі — неперервний блок пам'яті не потрібен.

```
 Однозв'язний список [10, 20, 30]

  _head                                            _tail
    │                                                │
    ▼                                                ▼
 ┌──────┬──────┐     ┌──────┬──────┐     ┌──────┬──────┐
 │  10  │  ●───┼────►│  20  │  ●───┼────►│  30  │ null │
 └──────┴──────┘     └──────┴──────┘     └──────┴──────┘
  Value   Next        Value   Next        Value   Next

 AddFirst(5):   новий вузол → Next = _head; _head = новий            O(1)
 AddLast(40):   _tail.Next = новий; _tail = новий                    O(1) (якщо є _tail)
 RemoveFirst(): _head = _head.Next                                   O(1)
 RemoveLast():  треба знайти ПЕРЕДОСТАННІЙ вузол — прохід від голови O(n)
 list[i]:       i переходів від голови                               O(n)
```

| Операція | Однозв'язний (з `_tail`) | Масив / `List<T>` |
|----------|--------------------------|-------------------|
| Доступ за індексом | O(n) | O(1) |
| Додати на початок | **O(1)** | O(n) |
| Додати в кінець | O(1) | O(1) аморт. |
| Видалити перший | **O(1)** | O(n) |
| Видалити останній | O(n) | O(1) |
| Вставити після відомого вузла | **O(1)** | O(n) |
| Пошук значення | O(n) | O(n) |
| Пам'ять на елемент (`int`, x64) | ≈ 32 байти (заголовок + значення + посилання) | 4 байти |

```csharp
using System.Collections;

var list = new SinglyLinkedList<int>();
list.AddLast(20);
list.AddLast(30);
list.AddFirst(10);
list.AddLast(40);
Console.WriteLine($"list: {list} (Count={list.Count})");

Console.WriteLine($"Contains(30)={list.Contains(30)}, Contains(99)={list.Contains(99)}");
Console.WriteLine($"RemoveFirst() = {list.RemoveFirst()} → {list}");
Console.WriteLine($"Remove(30) = {list.Remove(30)} → {list}");
Console.WriteLine($"Remove(99) = {list.Remove(99)} → {list}");

list.AddLast(50);
list.AddLast(60);
list.Reverse();
Console.WriteLine($"reversed: {list}, First={list.First}, Last={list.Last}");
Console.WriteLine($"LINQ Sum = {list.Sum()}");

/// <summary>Однозв'язний список з посиланнями на голову і хвіст.</summary>
public sealed class SinglyLinkedList<T> : IEnumerable<T>
{
    // Вузол — приватна деталь реалізації
    private sealed class Node(T value)
    {
        public T Value { get; } = value;
        public Node? Next { get; set; }
    }

    private Node? _head;                 // перший вузол або null
    private Node? _tail;                 // останній вузол або null
    private int _count;

    public int Count => _count;
    public T First => _head is null ? throw new InvalidOperationException("List is empty.") : _head.Value;
    public T Last => _tail is null ? throw new InvalidOperationException("List is empty.") : _tail.Value;

    public void AddFirst(T value)
    {
        var node = new Node(value) { Next = _head };  // новий вузол вказує на стару голову
        _head = node;
        _tail ??= node;                  // якщо список був порожній, вузол — і хвіст
        _count++;
    }

    public void AddLast(T value)
    {
        var node = new Node(value);
        if (_tail is null)
            _head = node;                // порожній список
        else
            _tail.Next = node;           // підчіплюємо після хвоста
        _tail = node;
        _count++;
    }

    public T RemoveFirst()
    {
        if (_head is null)
            throw new InvalidOperationException("List is empty.");
        T value = _head.Value;
        _head = _head.Next;              // старий вузол стане сміттям
        if (_head is null)
            _tail = null;                // видалили єдиний елемент
        _count--;
        return value;
    }

    public bool Contains(T value)
    {
        var comparer = EqualityComparer<T>.Default;
        for (Node? current = _head; current is not null; current = current.Next)
            if (comparer.Equals(current.Value, value))
                return true;
        return false;
    }

    // Видалення за значенням: потрібен ПОПЕРЕДНІЙ вузол, щоб перекинути його Next
    public bool Remove(T value)
    {
        var comparer = EqualityComparer<T>.Default;
        Node? previous = null;
        for (Node? current = _head; current is not null; previous = current, current = current.Next)
        {
            if (!comparer.Equals(current.Value, value))
                continue;
            if (previous is null)
                _head = current.Next;    // видаляємо голову
            else
                previous.Next = current.Next;  // «оминаємо» вузол
            if (current == _tail)
                _tail = previous;        // видалили хвіст
            _count--;
            return true;
        }
        return false;
    }

    // Розворот на місці: O(n) часу, O(1) пам'яті
    public void Reverse()
    {
        _tail = _head;
        Node? previous = null;
        Node? current = _head;
        while (current is not null)
        {
            Node? next = current.Next;   // 1) запам'ятати наступний
            current.Next = previous;     // 2) перевернути стрілку
            previous = current;          // 3) зсунути обидва вказівники
            current = next;
        }
        _head = previous;
    }

    public IEnumerator<T> GetEnumerator()
    {
        for (Node? current = _head; current is not null; current = current.Next)
            yield return current.Value;  // ітератор-генератор: компілятор будує енумератор сам
    }

    IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();

    public override string ToString() => "[" + string.Join(" -> ", this) + "]";
}
```

**Приклад запуску:**

```
list: [10 -> 20 -> 30 -> 40] (Count=4)
Contains(30)=True, Contains(99)=False
RemoveFirst() = 10 → [20 -> 30 -> 40]
Remove(30) = True → [20 -> 40]
Remove(99) = False → [20 -> 40]
reversed: [60 -> 50 -> 40 -> 20], First=60, Last=20
LINQ Sum = 170
```

Розворот списку покроково:

```
 prev = null, curr = 10

 null   10 → 20 → 30 → null        next = 20;  10.Next = null; prev = 10; curr = 20
 null ← 10   20 → 30 → null        next = 30;  20.Next = 10;   prev = 20; curr = 30
 null ← 10 ← 20   30 → null        next = null; 30.Next = 20;  prev = 30; curr = null
 null ← 10 ← 20 ← 30               _head = prev = 30
```

### 4.2 Будуємо самі: двозв'язний список `MyLinkedList<T>` із сентинелом

У **двозв'язному** списку кожен вузол має `Next` і `Previous`. Це дає O(1) видалення **будь-якого** вузла (не треба шукати попередній) і O(1) операції з обох кінців.

**Сентинел (фіктивний вузол, «страж»)** — службовий вузол без даних, який замикає список у кільце. Завдяки йому **немає перевірок на `null`** при вставці й видаленні: у кожного реального вузла завжди є сусід.

```
 Порожній MyLinkedList:           Список [A, B, C]:

      ┌──────────┐                ┌───────────────────────────────────────────────┐
      ▼          │                ▼                                               │
  ┌────────┐     │            ┌────────┐    ┌───┐    ┌───┐    ┌───┐              │
  │sentinel│─────┘  next      │sentinel│───►│ A │───►│ B │───►│ C │──────────────┘ next
  │        │◄────┐  prev      │        │◄───│   │◄───│   │◄───│   │◄─────────────┐ prev
  └────────┘     │            └────────┘    └───┘    └───┘    └───┘              │
      │          │                │                                               │
      └──────────┘                └───────────────────────────────────────────────┘

  sentinel.next = перший (A),  sentinel.prev = останній (C)

 Вставка X між B і C (InsertBefore(C, X)) — 4 присвоєння, завжди однакові:
   X.prev = B;  X.next = C;  B.next = X;  C.prev = X

 Видалення B — 2 присвоєння:
   B.prev.next = B.next;   B.next.prev = B.prev
```

`MyLinkedList<T>` підтримує: `AddFirst`, `AddLast`, `AddAfter`, `AddBefore`, `Remove(node)` O(1), `Remove(value)`, `RemoveFirst`, `RemoveLast`, `Find`, `FindLast`, `Reverse`, `Clear` та `foreach` із перевіркою змін.

```csharp
using System.Collections;

// ===== Демонстрація =====
var list = new MyLinkedList<string>();
list.AddLast("B");
list.AddLast("D");
MyLinkedListNode<string> a = list.AddFirst("A");
MyLinkedListNode<string> d = list.Find("D")!;
list.AddBefore(d, "C");                          // O(1) — вузол уже відомий
list.AddAfter(d, "E");
Console.WriteLine($"list:     {list} (Count={list.Count})");
Console.WriteLine($"First={list.First!.Value}, Last={list.Last!.Value}, A.Next={a.Next!.Value}, A.Previous={a.Previous?.Value ?? "null"}");

list.Remove(d);                                  // O(1) видалення за вузлом
Console.WriteLine($"Remove(D node): {list}");
Console.WriteLine($"Remove(\"B\") = {list.Remove("B")}, Remove(\"Z\") = {list.Remove("Z")} → {list}");
Console.WriteLine($"RemoveFirst = {list.RemoveFirst()}, RemoveLast = {list.RemoveLast()} → {list}");

list.AddLast("F");
list.AddLast("C");
Console.WriteLine($"Find(C) has next: {list.Find("C")!.Next is not null}, FindLast(C) has next: {list.FindLast("C")!.Next is not null}");

list.Reverse();
Console.WriteLine($"reversed: {list}");

// Обхід з кінця через Previous
var backwards = new List<string>();
for (MyLinkedListNode<string>? node = list.Last; node is not null; node = node.Previous)
    backwards.Add(node.Value);
Console.WriteLine($"backwards: {string.Join(" ", backwards)}");

// Захист від помилок використання
try
{
    list.Remove(d);                              // d уже не належить списку
}
catch (InvalidOperationException ex)
{
    Console.WriteLine($"Remove(foreign node): {ex.Message}");
}

try
{
    foreach (string item in list)
        if (item == "F")
            list.AddLast("G");
}
catch (InvalidOperationException ex)
{
    Console.WriteLine($"foreach: {ex.Message}");
}

list.Clear();
Console.WriteLine($"after Clear: {list} (Count={list.Count}, First is null: {list.First is null})");

// ===== Реалізація =====

/// <summary>Вузол двозв'язного списку. Next/Previous повертають null на межах списку.</summary>
public sealed class MyLinkedListNode<T>
{
    internal MyLinkedListNode(MyLinkedList<T>? list, T value)
    {
        List = list;
        Value = value;
        NextNode = this;                          // одиночний вузол замкнений сам на себе
        PreviousNode = this;
    }

    public T Value { get; set; }
    public MyLinkedList<T>? List { get; internal set; }   // null — вузол від'єднано

    internal MyLinkedListNode<T> NextNode { get; set; }   // «сирі» посилання, можуть вказувати на сентинел
    internal MyLinkedListNode<T> PreviousNode { get; set; }

    // Назовні сентинел не показуємо: замість нього — null
    public MyLinkedListNode<T>? Next => List is null || NextNode == List.Sentinel ? null : NextNode;
    public MyLinkedListNode<T>? Previous => List is null || PreviousNode == List.Sentinel ? null : PreviousNode;
}

/// <summary>Двозв'язний список із вузлом-сентинелом.</summary>
public sealed class MyLinkedList<T> : IEnumerable<T>
{
    private int _count;
    private int _version;

    public MyLinkedList()
    {
        Sentinel = new MyLinkedListNode<T>(null, default!);  // кільце з одного вузла
    }

    internal MyLinkedListNode<T> Sentinel { get; }

    public int Count => _count;
    public MyLinkedListNode<T>? First => _count == 0 ? null : Sentinel.NextNode;
    public MyLinkedListNode<T>? Last => _count == 0 ? null : Sentinel.PreviousNode;

    public MyLinkedListNode<T> AddFirst(T value) => InsertAfter(Sentinel, value);
    public MyLinkedListNode<T> AddLast(T value) => InsertAfter(Sentinel.PreviousNode, value);

    public MyLinkedListNode<T> AddAfter(MyLinkedListNode<T> node, T value)
    {
        ValidateNode(node);
        return InsertAfter(node, value);
    }

    public MyLinkedListNode<T> AddBefore(MyLinkedListNode<T> node, T value)
    {
        ValidateNode(node);
        return InsertAfter(node.PreviousNode, value);  // «перед node» = «після його попередника»
    }

    public void Remove(MyLinkedListNode<T> node)
    {
        ValidateNode(node);
        Unlink(node);
    }

    public bool Remove(T value)
    {
        MyLinkedListNode<T>? node = Find(value);
        if (node is null)
            return false;
        Unlink(node);
        return true;
    }

    public T RemoveFirst()
    {
        MyLinkedListNode<T> node = First ?? throw new InvalidOperationException("List is empty.");
        Unlink(node);
        return node.Value;
    }

    public T RemoveLast()
    {
        MyLinkedListNode<T> node = Last ?? throw new InvalidOperationException("List is empty.");
        Unlink(node);
        return node.Value;
    }

    public MyLinkedListNode<T>? Find(T value)
    {
        var comparer = EqualityComparer<T>.Default;
        for (var node = Sentinel.NextNode; node != Sentinel; node = node.NextNode)
            if (comparer.Equals(node.Value, value))
                return node;
        return null;
    }

    public MyLinkedListNode<T>? FindLast(T value)
    {
        var comparer = EqualityComparer<T>.Default;
        for (var node = Sentinel.PreviousNode; node != Sentinel; node = node.PreviousNode)
            if (comparer.Equals(node.Value, value))
                return node;
        return null;
    }

    // Розворот: у КОЖНОМУ вузлі (включно із сентинелом) міняємо місцями next і prev
    public void Reverse()
    {
        var node = Sentinel;
        do
        {
            (node.NextNode, node.PreviousNode) = (node.PreviousNode, node.NextNode);
            node = node.PreviousNode;              // після обміну «старий next» лежить у PreviousNode
        }
        while (node != Sentinel);
        _version++;
    }

    public void Clear()
    {
        // Від'єднуємо вузли, щоб зовнішні посилання на них не вважались частиною списку
        var node = Sentinel.NextNode;
        while (node != Sentinel)
        {
            var next = node.NextNode;
            node.List = null;
            node.NextNode = node.PreviousNode = node;
            node = next;
        }
        Sentinel.NextNode = Sentinel.PreviousNode = Sentinel;
        _count = 0;
        _version++;
    }

    public IEnumerator<T> GetEnumerator()
    {
        int version = _version;
        for (var node = Sentinel.NextNode; node != Sentinel; node = node.NextNode)
        {
            yield return node.Value;
            if (version != _version)               // перевіряємо після повернення керування з тіла foreach
                throw new InvalidOperationException("Collection was modified during enumeration.");
        }
    }

    IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();

    public override string ToString() => "[" + string.Join(" <-> ", this) + "]";

    // Єдине місце, де вузол вставляється: 4 присвоєння без жодних if
    private MyLinkedListNode<T> InsertAfter(MyLinkedListNode<T> previous, T value)
    {
        var node = new MyLinkedListNode<T>(this, value);
        MyLinkedListNode<T> next = previous.NextNode;
        node.PreviousNode = previous;
        node.NextNode = next;
        previous.NextNode = node;
        next.PreviousNode = node;
        _count++;
        _version++;
        return node;
    }

    // Єдине місце, де вузол видаляється: 2 присвоєння + від'єднання
    private void Unlink(MyLinkedListNode<T> node)
    {
        node.PreviousNode.NextNode = node.NextNode;
        node.NextNode.PreviousNode = node.PreviousNode;
        node.List = null;
        node.NextNode = node.PreviousNode = node;
        _count--;
        _version++;
    }

    private void ValidateNode(MyLinkedListNode<T> node)
    {
        ArgumentNullException.ThrowIfNull(node);
        if (node.List != this)
            throw new InvalidOperationException("The node does not belong to this list.");
    }
}
```

**Приклад запуску:**

```
list:     [A <-> B <-> C <-> D <-> E] (Count=5)
First=A, Last=E, A.Next=B, A.Previous=null
Remove(D node): [A <-> B <-> C <-> E]
Remove("B") = True, Remove("Z") = False → [A <-> C <-> E]
RemoveFirst = A, RemoveLast = E → [C]
Find(C) has next: True, FindLast(C) has next: False
reversed: [C <-> F <-> C]
backwards: C F C
Remove(foreign node): The node does not belong to this list.
foreach: Collection was modified during enumeration.
after Clear: [] (Count=0, First is null: True)
```

> **Навіщо `node.List`?** Без нього виклик `list.Remove(node)` з вузлом **іншого** списку тихо зіпсував би обидва списки (і їхні `Count`). Той самий прийом використовує `LinkedList<T>` у .NET.

### 4.3 `LinkedList<T>` у .NET

`System.Collections.Generic.LinkedList<T>` — двозв'язний **кільцевий** список (роль сентинела виконує посилання `head`). Вузли — публічний клас `LinkedListNode<T>` з властивостями `Value`, `Next`, `Previous`, `List`.

| Метод | Складність |
|-------|-----------|
| `AddFirst`, `AddLast` (значення або вузол) | O(1) |
| `AddBefore(node, …)`, `AddAfter(node, …)` | O(1) |
| `Remove(node)`, `RemoveFirst`, `RemoveLast` | O(1) |
| `Remove(value)`, `Find`, `FindLast`, `Contains` | O(n) |
| `First`, `Last`, `Count` | O(1) |
| Індексатор | **немає** |

- У C++ `std::list::splice` переносить діапазон вузлів між списками за O(1). У .NET **аналога немає**: вузол знає свій список (`node.List`), тому переносимо вузли по одному — `RemoveFirst` + `AddLast(node)`, без копіювання значень, але за O(k).
- Двобічної черги (deque) у стандартній бібліотеці немає — її роль може виконувати `LinkedList<T>` (`AddFirst`/`AddLast`/`RemoveFirst`/`RemoveLast`) або власний дек на кільцевому буфері (розділ 6.3).

```csharp
var tasks = new LinkedList<string>(["read", "code", "test"]);
LinkedListNode<string> code = tasks.Find("code")!; // пошук вузла — O(n)
tasks.AddAfter(code, "review");                    // вставка за вузлом — O(1), без зсуву
tasks.AddFirst("plan");                            // початок — O(1)
tasks.Remove(tasks.First!.Next!);                  // видалення вузла "read" — O(1)
Console.WriteLine(string.Join(" -> ", tasks));

// Аналога splice немає: вузол належить одному списку, тож переносимо по одному
var done = new LinkedList<string>();
while (tasks.First is { } node && node.Value != "test")
{
    tasks.RemoveFirst();       // від'єднуємо вузол від старого списку
    done.AddLast(node);        // той самий об'єкт вузла — без копіювання значення
}
Console.WriteLine($"done: {string.Join(", ", done)} | left: {string.Join(", ", tasks)}");

// Вузол, що належить іншому списку, додати не можна
try
{
    tasks.AddLast(done.First!);
}
catch (InvalidOperationException ex)
{
    Console.WriteLine($"AddLast(foreign node): {ex.Message}");
}

// LRU-ідея: переміщення вузла в початок за O(1)
var recent = new LinkedList<int>([1, 2, 3, 4]);
LinkedListNode<int> three = recent.Find(3)!;
recent.Remove(three);
recent.AddFirst(three);                            // той самий вузол — жодних нових об'єктів
Console.WriteLine($"move 3 to front: {string.Join(" ", recent)}; node.List == recent: {three.List == recent}");
```

**Приклад запуску:**

```
plan -> code -> review -> test
done: plan, code, review | left: test
AddLast(foreign node): The LinkedList node already belongs to a LinkedList.
move 3 to front: 3 1 2 4; node.List == recent: True
```

### 4.4 Масив проти зв'язного списку

| Критерій | `T[]` / `List<T>` | `LinkedList<T>` |
|----------|-------------------|-----------------|
| Доступ `[i]` | O(1) | O(n) |
| Вставка/видалення на початку | O(n) | O(1) |
| Вставка/видалення в середині (позиція відома) | O(n) зсув | O(1) |
| Вставка/видалення в кінці | O(1) аморт. | O(1) |
| Пошук | O(n); O(log n) якщо відсортований | O(n) |
| Пам'ять | компактно (+ до 2× запасу) | ≈ 48 байт службових даних на вузол |
| Кеш процесора | відмінно | погано (вузли розкидані) |
| Навантаження на GC | 1 об'єкт | n об'єктів |
| Стабільність посилань на елемент | ні (індекси зсуваються) | так (вузол живе, поки не видалений) |

На практиці `List<T>` виграє **майже завжди**, навіть для вставок у середину невеликих колекцій (до тисяч елементів), бо `Array.Copy` — дуже швидке блочне копіювання, а обхід списку «гальмує» кеш. Зв'язний список виправданий, коли у вас **вже є посилання на вузол** (LRU-кеш, списки подій у планувальнику, «intrusive» структури).

```csharp
using System.Diagnostics;

const int N = 2_000_000;
var array = new List<int>(N);
var linked = new LinkedList<int>();
for (int i = 0; i < N; i++)
{
    array.Add(i);
    linked.AddLast(i);
}

var sw = Stopwatch.StartNew();
long sum1 = 0;
foreach (int x in array)
    sum1 += x;
long listMs = sw.ElapsedMilliseconds;

sw.Restart();
long sum2 = 0;
foreach (int x in linked)
    sum2 += x;
long linkedMs = sw.ElapsedMilliseconds;

Console.WriteLine($"sums equal: {sum1 == sum2}");
Console.WriteLine($"List<int> traversal:       {listMs} ms");
Console.WriteLine($"LinkedList<int> traversal: {linkedMs} ms");

// Приблизна пам'ять
long before = GC.GetTotalMemory(true);
var probe = new LinkedList<int>();
for (int i = 0; i < 100_000; i++)
    probe.AddLast(i);
long bytesPerNode = (GC.GetTotalMemory(true) - before) / 100_000;
bytesPerNode = (bytesPerNode + 4) / 8 * 8;                 // округлюємо до кратного 8 (вирівнювання об'єктів)
GC.KeepAlive(probe);
Console.WriteLine($"LinkedList<int> ≈ {bytesPerNode} bytes per element (List<int>: 4)");
```

**Приклад запуску:**

```
sums equal: True
List<int> traversal:       … ms
LinkedList<int> traversal: … ms
LinkedList<int> ≈ 48 bytes per element (List<int>: 4)
```

### 4.5 Класичні задачі на списках

Для алгоритмічних задач зазвичай використовують найпростіший вузол `ListNode { Val, Next }` без класу-обгортки.

**Середина списку (fast/slow).** `slow` робить 1 крок, `fast` — 2. Коли `fast` дійде до кінця, `slow` буде посередині.

```
 1 → 2 → 3 → 4 → 5
 S,F                   старт
     S   F             крок 1
         S       F     крок 2: F.Next == null → середина = 3
```

**Виявлення циклу (алгоритм Флойда, «черепаха і заєць»).** Якщо цикл є, «заєць» рано чи пізно наздожене «черепаху» всередині циклу (відстань між ними зменшується на 1 за крок). Щоб знайти **початок** циклу: поставити один вказівник на голову, другий лишити в точці зустрічі й рухати обидва по 1 кроку — вони зустрінуться саме на вході в цикл.

```
 1 → 2 → 3 → 4 → 5
         ▲       │
         └── 7 ◄─6
 Нехай до входу в цикл μ кроків, довжина циклу λ.
 У точці зустрічі черепаха пройшла μ + x, заєць — 2(μ + x) = μ + x + kλ
 ⇒ μ + x = kλ ⇒ від точки зустрічі ще μ кроків до входу в цикл.
```

**Злиття двох відсортованих списків** — як для масивів, але без додаткової пам'яті: перевішуємо посилання. Фіктивна голова (`dummy`) прибирає особливий випадок «першого вузла».

```csharp
ListNode list = ListNode.From([1, 2, 3, 4, 5]);
Console.WriteLine($"list:     {list}");
Console.WriteLine($"middle:   {Middle(list).Val}");
Console.WriteLine($"middle of [1..6]: {Middle(ListNode.From([1, 2, 3, 4, 5, 6])).Val}");

ListNode reversed = ReverseIterative(list);
Console.WriteLine($"reversed: {reversed}");
Console.WriteLine($"reversed back (recursive): {ReverseRecursive(reversed)}");

ListNode? merged = MergeSorted(ListNode.From([1, 4, 7]), ListNode.From([2, 3, 8, 9]));
Console.WriteLine($"merged:   {merged}");

ListNode? withoutSecondFromEnd = RemoveNthFromEnd(ListNode.From([10, 20, 30, 40, 50]), 2);
Console.WriteLine($"remove 2nd from end: {withoutSecondFromEnd}");

// Будуємо цикл: 1 → 2 → 3 → 4 → 5 → 6 → 7 → (назад до 3)
ListNode cyclic = ListNode.From([1, 2, 3, 4, 5, 6, 7]);
ListNode tail = cyclic, entry = cyclic;
while (tail.Next is not null) tail = tail.Next;
for (int i = 0; i < 2; i++) entry = entry.Next!;
tail.Next = entry;
Console.WriteLine($"cycle in plain list: {HasCycle(list)}; cycle in cyclic list: {HasCycle(cyclic)}");
Console.WriteLine($"cycle starts at node with value {CycleStart(cyclic)?.Val}");

// Середина: fast рухається вдвічі швидше; для парної довжини — друга з двох середин
static ListNode Middle(ListNode head)
{
    ListNode slow = head;
    ListNode? fast = head;
    while (fast?.Next is not null)
    {
        slow = slow.Next!;
        fast = fast.Next.Next;
    }
    return slow;
}

static ListNode ReverseIterative(ListNode? head)
{
    ListNode? previous = null;
    while (head is not null)
    {
        ListNode? next = head.Next;
        head.Next = previous;
        previous = head;
        head = next;
    }
    return previous!;
}

// Рекурсія: розвернути хвіст, а потім причепити голову в кінець. Глибина стеку O(n)!
static ListNode ReverseRecursive(ListNode head)
{
    if (head.Next is null)
        return head;
    ListNode newHead = ReverseRecursive(head.Next);
    head.Next.Next = head;                 // наступний вузол тепер вказує назад
    head.Next = null;
    return newHead;
}

static ListNode? MergeSorted(ListNode? a, ListNode? b)
{
    var dummy = new ListNode(0);           // фіктивна голова: не треба окремо вибирати перший вузол
    ListNode tail = dummy;
    while (a is not null && b is not null)
    {
        if (a.Val <= b.Val) { tail.Next = a; a = a.Next; }
        else { tail.Next = b; b = b.Next; }
        tail = tail.Next;
    }
    tail.Next = a ?? b;                    // решту підчіплюємо цілком — O(1)
    return dummy.Next;
}

// Два вказівники з відставанням n: коли передній дійде до кінця, задній стоїть перед цільовим вузлом
static ListNode? RemoveNthFromEnd(ListNode head, int n)
{
    var dummy = new ListNode(0) { Next = head };
    ListNode front = dummy, back = dummy;
    for (int i = 0; i <= n; i++)
        front = front.Next!;
    while (front is not null)
    {
        front = front.Next!;
        back = back.Next!;
    }
    back.Next = back.Next!.Next;
    return dummy.Next;
}

static bool HasCycle(ListNode? head)
{
    ListNode? slow = head, fast = head;
    while (fast?.Next is not null)
    {
        slow = slow!.Next;
        fast = fast.Next.Next;
        if (slow == fast)                  // порівнюємо ПОСИЛАННЯ, а не значення
            return true;
    }
    return false;
}

static ListNode? CycleStart(ListNode? head)
{
    ListNode? slow = head, fast = head;
    while (fast?.Next is not null)
    {
        slow = slow!.Next;
        fast = fast.Next.Next;
        if (slow == fast)
        {
            ListNode? pointer = head;          // один — з голови, другий — з точки зустрічі
            while (pointer != slow)
            {
                pointer = pointer!.Next;
                slow = slow!.Next;
            }
            return pointer;
        }
    }
    return null;
}

public sealed class ListNode(int val)
{
    public int Val { get; } = val;
    public ListNode? Next { get; set; }

    public static ListNode From(int[] values)
    {
        var dummy = new ListNode(0);
        ListNode tail = dummy;
        foreach (int v in values)
            tail = tail.Next = new ListNode(v);
        return dummy.Next!;
    }

    // Обмежуємо вивід, щоб не зациклитися на списку з циклом
    public override string ToString()
    {
        var parts = new List<int>();
        for (ListNode? node = this; node is not null && parts.Count < 20; node = node.Next)
            parts.Add(node.Val);
        return string.Join(" -> ", parts);
    }
}
```

**Приклад запуску:**

```
list:     1 -> 2 -> 3 -> 4 -> 5
middle:   3
middle of [1..6]: 4
reversed: 5 -> 4 -> 3 -> 2 -> 1
reversed back (recursive): 1 -> 2 -> 3 -> 4 -> 5
merged:   1 -> 2 -> 3 -> 4 -> 7 -> 8 -> 9
remove 2nd from end: 10 -> 20 -> 30 -> 50
cycle in plain list: False; cycle in cyclic list: True
cycle starts at node with value 3
```

### Типові помилки (розділ 4)

1. **Втратити решту списку при розвороті** — перезаписати `current.Next` до того, як запам'ятати наступний вузол.
2. **Забути оновити `_tail`** при видаленні останнього вузла або `_head` при видаленні першого.
3. **Не обробити порожній список і список з одного елемента** — найчастіші джерела `NullReferenceException`.
4. **Порівнювати вузли за значенням** у Флойді (`slow.Val == fast.Val`) — дублікати значень дадуть хибний «цикл».
5. **Рекурсивні алгоритми на довгих списках** — глибина рекурсії O(n) може призвести до `StackOverflowException` (мільйон вузлів).
6. **Використовувати `LinkedList<T>` «бо вставка O(1)»**, а потім шукати позицію вставки за O(n) — виграшу немає, а кеш страждає.
7. **Вставляти вузол, що вже належить іншому `LinkedList<T>`** — `InvalidOperationException`; спочатку `Remove`.

### Міні-вправи (розділ 4)

**Вправа 4.1.** Перевірте, чи є однозв'язний список паліндромом, за O(n) часу і O(1) додаткової пам'яті. *Підказка:* знайдіть середину, розверніть другу половину і порівняйте.

<details>
<summary>Розв'язок</summary>

```csharp
Console.WriteLine($"1-2-3-2-1: {IsPalindrome(Build([1, 2, 3, 2, 1]))}");
Console.WriteLine($"1-2-2-1:   {IsPalindrome(Build([1, 2, 2, 1]))}");
Console.WriteLine($"1-2-3:     {IsPalindrome(Build([1, 2, 3]))}");

static bool IsPalindrome(Node? head)
{
    // 1) середина: slow зупиниться на початку другої половини
    Node? slow = head, fast = head;
    while (fast?.Next is not null)
    {
        slow = slow!.Next;
        fast = fast.Next.Next;
    }

    // 2) розвертаємо другу половину
    Node? previous = null;
    while (slow is not null)
    {
        Node? next = slow.Next;
        slow.Next = previous;
        previous = slow;
        slow = next;
    }

    // 3) порівнюємо першу половину з розвернутою другою
    for (Node? left = head, right = previous; right is not null; left = left!.Next, right = right.Next)
        if (left!.Value != right.Value)
            return false;
    return true;
}

static Node? Build(int[] values)
{
    Node? head = null;
    for (int i = values.Length - 1; i >= 0; i--)
        head = new Node(values[i], head);       // додаємо на початок у зворотному порядку
    return head;
}

public sealed class Node(int value, Node? next)
{
    public int Value { get; } = value;
    public Node? Next { get; set; } = next;
}
```

**Приклад запуску:**

```
1-2-3-2-1: True
1-2-2-1:   True
1-2-3:     False
```

</details>

**Вправа 4.2.** Чому у двозв'язному списку `Remove(node)` — O(1), а в однозв'язному — O(n)? *Відповідь:* щоб «вирізати» вузол, треба змінити `Next` **попереднього** вузла. У двозв'язному він доступний як `node.Previous`, в однозв'язному його доводиться шукати від голови.

---

> ## Перерва 2 (≈ 10 хвилин)
>
> Минуло ≈ 125 хвилин. Далі — стек і черга: дві найважливіші «обмежені» структури.

---

## 5. Стек

*≈ 30 хвилин*

### 5.1 LIFO і дві реалізації

**Стек** — структура «останній увійшов — перший вийшов» (**LIFO**, Last In, First Out). Доступ є лише до **вершини**. Аналогія — стос тарілок: кладемо зверху і беремо зверху.

```
   Push(1)   Push(2)   Push(3)    Pop() → 3   Peek() → 2
                        ┌───┐
              ┌───┐     │ 3 │ ← top
    ┌───┐     │ 2 │     │ 2 │      ┌───┐       ┌───┐
    │ 1 │     │ 1 │     │ 1 │      │ 2 │ ←top  │ 2 │ ← top (не видаляється)
    └───┘     └───┘     └───┘      │ 1 │       │ 1 │
                                   └───┘       └───┘
```

| Операція | Опис | Складність |
|----------|------|-----------|
| `Push(x)` | покласти на вершину | O(1) (аморт. для масиву) |
| `Pop()` | зняти з вершини | O(1) |
| `Peek()` | подивитись на вершину | O(1) |
| `Count`, `IsEmpty` | розмір | O(1) |

**Реалізація на масиві** — вершина в кінці масиву (саме тому стек на масиві такий простий: кінець масиву змінюється за O(1)):

```
 _items: │ 1 │ 2 │ 3 │   │   │      _count = 3 → вершина = _items[2]
          [0] [1] [2] [3] [4]
```

**Реалізація на зв'язному списку** — вершина на **голові** списку (початок однозв'язного списку змінюється за O(1)):

```
 _top ──► [3] ──► [2] ──► [1] ──► null
```

```csharp
IStack<int>[] stacks = [new ArrayStack<int>(), new LinkedStack<int>()];
foreach (IStack<int> stack in stacks)
{
    for (int i = 1; i <= 5; i++)
        stack.Push(i * 10);
    int top = stack.Pop();
    int peek = stack.Peek();
    var rest = new List<int>();
    while (!stack.IsEmpty)
        rest.Add(stack.Pop());                  // виходять у зворотному порядку
    Console.WriteLine($"{stack.GetType().Name,-15} popped {top}, peek {peek}, rest: {string.Join(" ", rest)}");

    try
    {
        stack.Pop();
    }
    catch (InvalidOperationException ex)
    {
        Console.WriteLine($"{"",-15} Pop on empty: {ex.Message}");
    }
}

public interface IStack<T>
{
    int Count { get; }
    bool IsEmpty => Count == 0;                 // реалізація за замовчуванням в інтерфейсі
    void Push(T item);
    T Pop();
    T Peek();
}

/// <summary>Стек на динамічному масиві: вершина — останній зайнятий елемент.</summary>
public sealed class ArrayStack<T> : IStack<T>
{
    private T[] _items = new T[4];
    private int _count;

    public int Count => _count;

    public void Push(T item)
    {
        if (_count == _items.Length)
            Array.Resize(ref _items, _items.Length * 2);   // подвоєння — O(1) амортизовано
        _items[_count++] = item;
    }

    public T Pop()
    {
        if (_count == 0)
            throw new InvalidOperationException("Stack is empty.");
        T item = _items[--_count];
        _items[_count] = default!;              // не тримаємо посилання для GC
        return item;
    }

    public T Peek() => _count == 0 ? throw new InvalidOperationException("Stack is empty.") : _items[_count - 1];
}

/// <summary>Стек на однозв'язному списку: вершина — голова списку.</summary>
public sealed class LinkedStack<T> : IStack<T>
{
    private sealed record Node(T Value, Node? Next);   // незмінний вузол

    private Node? _top;
    private int _count;

    public int Count => _count;

    public void Push(T item)
    {
        _top = new Node(item, _top);            // новий вузол вказує на стару вершину
        _count++;
    }

    public T Pop()
    {
        Node top = _top ?? throw new InvalidOperationException("Stack is empty.");
        _top = top.Next;
        _count--;
        return top.Value;
    }

    public T Peek() => _top is null ? throw new InvalidOperationException("Stack is empty.") : _top.Value;
}
```

**Приклад запуску:**

```
ArrayStack`1    popped 50, peek 40, rest: 40 30 20 10
                Pop on empty: Stack is empty.
LinkedStack`1   popped 50, peek 40, rest: 40 30 20 10
                Pop on empty: Stack is empty.
```

| Реалізація | Push | Pop | Пам'ять | Кеш |
|------------|------|-----|---------|-----|
| На масиві | O(1) аморт. (інколи O(n)) | O(1) | компактно | відмінно |
| На списку | O(1) завжди (але `new` щоразу) | O(1) | +вузол на елемент | погано |

`Stack<T>` у .NET реалізований **на масиві**.

### 5.2 `Stack<T>` у .NET

| Член | Опис | Складність |
|------|------|-----------|
| `Push(item)` | на вершину | O(1) аморт. |
| `Pop()` / `TryPop(out item)` | зняти (перший кидає виняток на порожньому стеку) | O(1) |
| `Peek()` / `TryPeek(out item)` | подивитися | O(1) |
| `Count`, `Clear()`, `Contains(item)` | | O(1) / O(n) / O(n) |
| `ToArray()`, `foreach` | **від вершини до дна** | O(n) |
| `EnsureCapacity`, `TrimExcess` | як у `List<T>` | O(n) |

```csharp
var history = new Stack<string>();
history.Push("google.com");
history.Push("learn.microsoft.com");
history.Push("github.com");

Console.WriteLine($"Count={history.Count}, Peek={history.Peek()}");
Console.WriteLine($"foreach order (top → bottom): {string.Join(" | ", history)}");
Console.WriteLine($"ToArray()[0] = {history.ToArray()[0]}");
Console.WriteLine($"Contains(google.com) = {history.Contains("google.com")}");

Console.WriteLine($"back: {history.Pop()}");
Console.WriteLine($"back: {history.Pop()}");

// Try-методи — без винятків
while (history.TryPop(out string? page))
    Console.WriteLine($"TryPop: {page}");
Console.WriteLine($"TryPeek on empty: {history.TryPeek(out _)}");

try
{
    history.Pop();
}
catch (InvalidOperationException ex)
{
    Console.WriteLine($"Pop on empty: {ex.Message}");
}

// Стек із колекції: елементи кладуться по черзі, тож останній стає вершиною
var fromArray = new Stack<int>([1, 2, 3]);
Console.WriteLine($"new Stack<int>([1,2,3]).Peek() = {fromArray.Peek()}");

// Розворот послідовності — класичне застосування
string word = "stack";
var letters = new Stack<char>(word);
Console.WriteLine($"reverse of '{word}' = '{new string(letters.ToArray())}'");
```

**Приклад запуску:**

```
Count=3, Peek=github.com
foreach order (top → bottom): github.com | learn.microsoft.com | google.com
ToArray()[0] = github.com
Contains(google.com) = True
back: github.com
back: learn.microsoft.com
TryPop: google.com
TryPeek on empty: False
Pop on empty: Stack empty.
new Stack<int>([1,2,3]).Peek() = 3
reverse of 'stack' = 'kcats'
```

### 5.3 Стек викликів і рекурсія → явний стек

Кожен виклик методу створює на **стеку викликів** (call stack) **кадр** (stack frame): параметри, локальні змінні, адресу повернення. Коли метод завершується, кадр знімається — LIFO в чистому вигляді.

```
 Factorial(3)

 виклики:                                       повернення:
 ┌───────────────────┐
 │ Factorial(n=1)    │ ← вершина: return 1      ┌─► 1
 ├───────────────────┤                          │
 │ Factorial(n=2)    │   чекає: 2 * ?           ├─► 2 * 1 = 2
 ├───────────────────┤                          │
 │ Factorial(n=3)    │   чекає: 3 * ?           └─► 3 * 2 = 6
 ├───────────────────┤
 │ Main              │
 └───────────────────┘
 Розмір стеку потоку в .NET — зазвичай 1 МБ (головний потік; 8 МБ на macOS/Linux).
 Глибока рекурсія (≈ 10⁴–10⁵ кадрів) → StackOverflowException, який НЕ можна перехопити catch.
```

Будь-яку рекурсію можна переписати з **явним стеком** `Stack<T>` у купі — там мільйони елементів не проблема. Приклад: обхід дерева в глибину (DFS) і сума цифр великого «ланцюжка» вузлів.

```csharp
// Дерево:        1
//              / | \
//             2  3  4
//            / \     \
//           5   6     7
var tree = new TreeNode(1,
    new TreeNode(2, new TreeNode(5), new TreeNode(6)),
    new TreeNode(3),
    new TreeNode(4, new TreeNode(7)));

var recursive = new List<int>();
DfsRecursive(tree, recursive, depth: 0);
Console.WriteLine($"recursive DFS: {string.Join(" ", recursive)}");
Console.WriteLine($"explicit  DFS: {string.Join(" ", DfsIterative(tree))}");

// Дуже глибоке «дерево» — ланцюжок із 1 000 000 вузлів
var chain = new TreeNode(0);
TreeNode current = chain;
for (int i = 1; i < 1_000_000; i++)
{
    var next = new TreeNode(i);
    current.Children.Add(next);
    current = next;
}
Console.WriteLine($"depth of 1_000_000 chain (explicit stack): {MaxDepthIterative(chain)}");
// DfsRecursive(chain, ...) на такому ланцюжку впав би зі StackOverflowException

// Трасування стеку викликів
Console.WriteLine($"Factorial(4) = {Factorial(4, 0)}");

static void DfsRecursive(TreeNode node, List<int> output, int depth)
{
    output.Add(node.Value);
    foreach (TreeNode child in node.Children)
        DfsRecursive(child, output, depth + 1);   // кожен виклик — новий кадр
}

static List<int> DfsIterative(TreeNode root)
{
    var output = new List<int>();
    var stack = new Stack<TreeNode>();
    stack.Push(root);
    while (stack.TryPop(out TreeNode? node))
    {
        output.Add(node.Value);
        // Кладемо дітей у ЗВОРОТНОМУ порядку, щоб першим знявся лівий — як у рекурсії
        for (int i = node.Children.Count - 1; i >= 0; i--)
            stack.Push(node.Children[i]);
    }
    return output;
}

static int MaxDepthIterative(TreeNode root)
{
    int best = 0;
    var stack = new Stack<(TreeNode Node, int Depth)>();  // у стеку — «кадр» вручну: вузол + глибина
    stack.Push((root, 1));
    while (stack.TryPop(out var frame))
    {
        best = Math.Max(best, frame.Depth);
        foreach (TreeNode child in frame.Node.Children)
            stack.Push((child, frame.Depth + 1));
    }
    return best;
}

static long Factorial(int n, int depth)
{
    string indent = new(' ', depth * 2);
    Console.WriteLine($"{indent}enter Factorial({n})");
    long result = n <= 1 ? 1 : n * Factorial(n - 1, depth + 1);
    Console.WriteLine($"{indent}leave Factorial({n}) = {result}");
    return result;
}

public sealed class TreeNode(int value, params TreeNode[] children)
{
    public int Value { get; } = value;
    public List<TreeNode> Children { get; } = [.. children];
}
```

**Приклад запуску:**

```
recursive DFS: 1 2 5 6 3 4 7
explicit  DFS: 1 2 5 6 3 4 7
depth of 1_000_000 chain (explicit stack): 1000000
enter Factorial(4)
  enter Factorial(3)
    enter Factorial(2)
      enter Factorial(1)
      leave Factorial(1) = 1
    leave Factorial(2) = 2
  leave Factorial(3) = 6
leave Factorial(4) = 24
Factorial(4) = 24
```

### 5.4 Баланс дужок і постфіксний запис (RPN)

**Баланс дужок.** Відкриваюча дужка — `Push`. Закриваюча має відповідати вершині стеку — `Pop`. У кінці стек має бути порожнім.

```
 Рядок "{[()]}"

 символ:  {      [       (        )       ]      }
 стек:   {      {[      {[(      {[      {       (порожньо) → збалансовано

 Рядок "([)]"
 символ:  (      [       )  ← вершина '[' не пара для ')' → НЕ збалансовано
```

**Постфіксний запис (зворотна польська нотація, RPN):** оператор записується **після** операндів: `3 4 + 2 *` = `(3 + 4) * 2`. Дужки не потрібні, пріоритети не потрібні — обчислення одним проходом зі стеком.

```
 "3 4 + 2 *"
 токен:   3      4       +          2        *
 стек:   [3]   [3,4]    [7]       [7,2]     [14]
                       4,3 зняли         2,7 зняли
```

```csharp
Console.WriteLine($"{{[()]}} balanced? {IsBalanced("{[()]}")}");
Console.WriteLine($"([)] balanced? {IsBalanced("([)]")}");
Console.WriteLine($"((  balanced? {IsBalanced("((")}");
Console.WriteLine($"if (a[i] > 0) {{ f(x); }} balanced? {IsBalanced("if (a[i] > 0) { f(x); }")}");
Console.WriteLine($"RPN '3 4 + 2 *' = {EvalRpn("3 4 + 2 *")}");
Console.WriteLine($"RPN '5 1 2 + 4 * + 3 -' = {EvalRpn("5 1 2 + 4 * + 3 -")}");
Console.WriteLine($"RPN '10 3 /' = {EvalRpn("10 3 /")}, '2 5 -' = {EvalRpn("2 5 -")}");

// Баланс дужок: відкриваюча — push, закриваюча — має збігтися з вершиною
static bool IsBalanced(string text)
{
    var stack = new Stack<char>();
    foreach (char ch in text)
    {
        switch (ch)
        {
            case '(' or '[' or '{':
                stack.Push(ch);                    // O(1)
                break;
            case ')' or ']' or '}':
                char open = ch switch { ')' => '(', ']' => '[', _ => '{' };
                if (!stack.TryPop(out char top) || top != open)
                    return false;                  // порожній стек або чужа дужка
                break;
        }
    }
    return stack.Count == 0;                       // залишились незакриті — дисбаланс
}

// Постфіксний вираз: числа — у стек, оператор забирає два верхні
static int EvalRpn(string expression)
{
    var stack = new Stack<int>();
    foreach (string token in expression.Split(' '))
    {
        if (int.TryParse(token, out int value)) { stack.Push(value); continue; }
        int right = stack.Pop(), left = stack.Pop(); // порядок важливий для - та /
        stack.Push(token switch
        {
            "+" => left + right,
            "-" => left - right,
            "*" => left * right,
            _ => left / right,
        });
    }
    return stack.Pop();
}
```

**Приклад запуску:**

```
{[()]} balanced? True
([)] balanced? False
((  balanced? False
if (a[i] > 0) { f(x); } balanced? True
RPN '3 4 + 2 *' = 14
RPN '5 1 2 + 4 * + 3 -' = 14
RPN '10 3 /' = 3, '2 5 -' = -3
```

### 5.5 Алгоритм сортувальної станції (infix → postfix)

Як перетворити звичний **інфіксний** вираз `3 + 4 * (2 - 1)` у постфіксний? **Алгоритм сортувальної станції** (shunting-yard) Едсгера Дейкстри використовує **стек операторів**:

1. Число → одразу у вихід.
2. Оператор `op`: поки на вершині стеку оператор із **вищим** пріоритетом (або **рівним**, якщо `op` лівоасоціативний) — перекладаємо його у вихід. Потім `Push(op)`.
3. `(` → `Push`.
4. `)` → перекладаємо оператори у вихід до `(`; саму `(` викидаємо.
5. У кінці — всі оператори зі стеку у вихід.

```
 Вираз: 3 + 4 * ( 2 - 1 )

 токен │ дія                               │ вихід              │ стек операторів
 ──────┼───────────────────────────────────┼────────────────────┼────────────────
   3   │ число → вихід                      │ 3                  │
   +   │ стек порожній → push               │ 3                  │ +
   4   │ число → вихід                      │ 3 4                │ +
   *   │ '*' вищий за '+' → push            │ 3 4                │ + *
   (   │ push                               │ 3 4                │ + * (
   2   │ число → вихід                      │ 3 4 2              │ + * (
   -   │ на вершині '(' → push               │ 3 4 2              │ + * ( -
   1   │ число → вихід                      │ 3 4 2 1            │ + * ( -
   )   │ виштовхнути до '('                 │ 3 4 2 1 -          │ + *
  кінець│ вивантажити стек                  │ 3 4 2 1 - * +      │
```

| Оператор | Пріоритет | Асоціативність |
|----------|-----------|----------------|
| `^` | 3 | права (`2 ^ 3 ^ 2 = 2 ^ 9`) |
| `*`, `/` | 2 | ліва |
| `+`, `-` | 1 | ліва |

```csharp
string[] expressions =
[
    "3 + 4 * (2 - 1)",
    "(1 + 2) * (3 + 4)",
    "10 - 4 - 3",
    "2 ^ 3 ^ 2",
    "100 / (2 + 3) * 4",
];

foreach (string expression in expressions)
{
    List<string> postfix = ToPostfix(expression);
    Console.WriteLine($"{expression,-20} => {string.Join(" ", postfix),-18} = {Evaluate(postfix)}");
}

// Інфікс → постфікс (shunting-yard). Числа — цілі невід'ємні, токени можуть бути без пробілів
static List<string> ToPostfix(string expression)
{
    var output = new List<string>();
    var operators = new Stack<char>();
    int i = 0;
    while (i < expression.Length)
    {
        char ch = expression[i];
        if (char.IsWhiteSpace(ch)) { i++; continue; }

        if (char.IsDigit(ch))
        {
            int start = i;
            while (i < expression.Length && char.IsDigit(expression[i]))
                i++;                                        // багатоцифрове число
            output.Add(expression[start..i]);
            continue;
        }

        if (ch == '(')
        {
            operators.Push(ch);
        }
        else if (ch == ')')
        {
            while (operators.Peek() != '(')
                output.Add(operators.Pop().ToString());
            operators.Pop();                                // викидаємо '('
        }
        else
        {
            // Виштовхуємо «сильніші» оператори; '(' — бар'єр
            while (operators.TryPeek(out char top) && top != '(' &&
                   (Precedence(top) > Precedence(ch) || (Precedence(top) == Precedence(ch) && !IsRightAssociative(ch))))
            {
                output.Add(operators.Pop().ToString());
            }
            operators.Push(ch);
        }
        i++;
    }
    while (operators.Count > 0)
        output.Add(operators.Pop().ToString());
    return output;
}

static int Precedence(char op) => op switch
{
    '^' => 3,
    '*' or '/' => 2,
    '+' or '-' => 1,
    _ => throw new ArgumentException($"Unknown operator {op}"),
};

static bool IsRightAssociative(char op) => op == '^';

static long Evaluate(List<string> postfix)
{
    var stack = new Stack<long>();
    foreach (string token in postfix)
    {
        if (long.TryParse(token, out long number)) { stack.Push(number); continue; }
        long right = stack.Pop(), left = stack.Pop();
        stack.Push(token switch
        {
            "+" => left + right,
            "-" => left - right,
            "*" => left * right,
            "/" => left / right,
            _ => (long)Math.Pow(left, right),
        });
    }
    return stack.Pop();
}
```

**Приклад запуску:**

```
3 + 4 * (2 - 1)      => 3 4 2 1 - * +      = 7
(1 + 2) * (3 + 4)    => 1 2 + 3 4 + *      = 21
10 - 4 - 3           => 10 4 - 3 -         = 3
2 ^ 3 ^ 2            => 2 3 2 ^ ^          = 512
100 / (2 + 3) * 4    => 100 2 3 + / 4 *    = 80
```

### 5.6 Стек з мінімумом за O(1) (Min-stack)

**Задача:** стек, що підтримує `Push`, `Pop`, `Peek` і `Min` — **усі за O(1)**.

Мінімум при `Pop` може змінитися, і перераховувати його за O(n) не можна. **Ідея:** поряд з кожним значенням зберігати мінімум **на момент його додавання**. Тоді після `Pop` мінімум — це те, що записано біля нової вершини.

```
 Push(5)  Push(3)  Push(7)  Push(2)       Pop()        Pop()
                            │2 | 2│
                   │7 | 3│  │7 | 3│      │7 | 3│
          │3 | 3│  │3 | 3│  │3 | 3│      │3 | 3│      │3 | 3│
 │5 | 5│  │5 | 5│  │5 | 5│  │5 | 5│      │5 | 5│      │5 | 5│
  val min                    Min=2        Min=3        Min=3
```

Економніший варіант — другий стек, у який кладемо значення лише коли воно `<=` поточного мінімуму.

```csharp
var stack = new MinStack<int>();
foreach (int x in new[] { 5, 3, 7, 2, 2, 8 })
{
    stack.Push(x);
    Console.WriteLine($"Push({x}) → Min={stack.Min}");
}
while (stack.Count > 0)
{
    int min = stack.Min;
    Console.WriteLine($"Min={min}, Pop() = {stack.Pop()}");
}

/// <summary>Стек, що зберігає поряд з кожним значенням мінімум «під ним».</summary>
public sealed class MinStack<T> where T : IComparable<T>
{
    private readonly Stack<(T Value, T Min)> _items = new();

    public int Count => _items.Count;

    public T Min => _items.Count > 0 ? _items.Peek().Min : throw new InvalidOperationException("Stack is empty.");

    public T Peek() => _items.Peek().Value;

    public void Push(T value)
    {
        // новий мінімум = менше з (value, мінімум під ним)
        T min = _items.TryPeek(out var top) && top.Min.CompareTo(value) < 0 ? top.Min : value;
        _items.Push((value, min));
    }

    public T Pop() => _items.Pop().Value;
}
```

**Приклад запуску:**

```
Push(5) → Min=5
Push(3) → Min=3
Push(7) → Min=3
Push(2) → Min=2
Push(2) → Min=2
Push(8) → Min=2
Min=2, Pop() = 8
Min=2, Pop() = 2
Min=2, Pop() = 2
Min=3, Pop() = 7
Min=3, Pop() = 3
Min=5, Pop() = 5
```

### 5.7 Монотонний стек: наступний більший елемент

**Задача:** для кожного елемента знайти **перший більший** праворуч (або −1). Наївно — O(n²).

**Монотонний стек** зберігає індекси елементів, для яких відповідь **ще не знайдена**; значення в стеку **спадають** від дна до вершини. Коли приходить новий елемент `x`, він є відповіддю для всіх менших елементів на вершині — знімаємо їх.

```
 a = [2, 1, 2, 4, 3]

 i=0 x=2: стек порожній               push 0        стек (значення): [2]
 i=1 x=1: 1 < 2                        push 1        [2, 1]
 i=2 x=2: 2 > 1 → ans[1]=2, pop        push 2        [2, 2]      (2 не > 2 — зупинка)
 i=3 x=4: 4 > 2 → ans[2]=4; 4 > 2 → ans[0]=4
                                       push 3        [4]
 i=4 x=3: 3 < 4                        push 4        [4, 3]
 кінець: для решти відповіді немає → −1

 результат: [4, 2, 4, -1, -1]
```

Кожен індекс потрапляє в стек **один раз** і знімається **не більше одного разу** — отже, O(n) сумарно, хоча всередині `for` є `while`.

Задачі цього типу: наступний більший/менший елемент, «скільки днів до потепління», найбільший прямокутник у гістограмі, «зона видимості» будинків.

```csharp
int[] a = [2, 1, 2, 4, 3];
Console.WriteLine($"next greater of [{string.Join(", ", a)}]: [{string.Join(", ", NextGreater(a))}]");

int[] temperatures = [73, 74, 75, 71, 69, 72, 76, 73];
Console.WriteLine($"days until warmer: [{string.Join(", ", DaysUntilWarmer(temperatures))}]");

int[] histogram = [2, 1, 5, 6, 2, 3];
Console.WriteLine($"largest rectangle in histogram: {LargestRectangle(histogram)}");

// Кругова версія: після кінця масиву пошук продовжується з початку — проходимо двічі
int[] circular = [1, 2, 1];
Console.WriteLine($"next greater circular [1, 2, 1]: [{string.Join(", ", NextGreaterCircular(circular))}]");

static int[] NextGreater(int[] a)
{
    var result = new int[a.Length];
    Array.Fill(result, -1);
    var stack = new Stack<int>();                      // індекси; значення спадають від дна до вершини
    for (int i = 0; i < a.Length; i++)
    {
        while (stack.Count > 0 && a[stack.Peek()] < a[i])
            result[stack.Pop()] = a[i];                // a[i] — перший більший для вершини
        stack.Push(i);
    }
    return result;
}

static int[] DaysUntilWarmer(int[] t)
{
    var result = new int[t.Length];
    var stack = new Stack<int>();
    for (int i = 0; i < t.Length; i++)
    {
        while (stack.Count > 0 && t[stack.Peek()] < t[i])
        {
            int day = stack.Pop();
            result[day] = i - day;                     // відстань, а не значення
        }
        stack.Push(i);
    }
    return result;
}

// Для кожного стовпця: наскільки далеко вліво і вправо він може «розширитися»
static int LargestRectangle(int[] heights)
{
    int best = 0;
    var stack = new Stack<int>();                      // індекси; висоти зростають
    for (int i = 0; i <= heights.Length; i++)
    {
        int current = i == heights.Length ? 0 : heights[i];   // фіктивний 0 у кінці вивантажує стек
        while (stack.Count > 0 && heights[stack.Peek()] > current)
        {
            int height = heights[stack.Pop()];
            int left = stack.Count == 0 ? -1 : stack.Peek();   // перший нижчий ліворуч
            best = Math.Max(best, height * (i - left - 1));    // i — перший нижчий праворуч
        }
        stack.Push(i);
    }
    return best;
}

static int[] NextGreaterCircular(int[] a)
{
    int n = a.Length;
    var result = new int[n];
    Array.Fill(result, -1);
    var stack = new Stack<int>();
    for (int k = 0; k < 2 * n; k++)
    {
        int i = k % n;
        while (stack.Count > 0 && a[stack.Peek()] < a[i])
            result[stack.Pop()] = a[i];
        if (k < n)
            stack.Push(i);                             // другий прохід лише «закриває» відповіді
    }
    return result;
}
```

**Приклад запуску:**

```
next greater of [2, 1, 2, 4, 3]: [4, 2, 4, -1, -1]
days until warmer: [1, 1, 4, 2, 1, 1, 0, 0]
largest rectangle in histogram: 10
next greater circular [1, 2, 1]: [2, -1, 2]
```

### 5.8 Undo/Redo на двох стеках

Текстовий редактор зберігає **команди** у двох стеках:

- `undo` — виконані команди; `Undo()` знімає останню, відкочує її і кладе в `redo`;
- `redo` — скасовані команди; `Redo()` повторює і повертає в `undo`;
- **нова** дія очищує `redo` (гілка «майбутнього» втрачається).

```
 type "Hello"  type " world"  Undo        Undo       Redo       type "!"
 undo: [H]     [H, w]         [H]         []         [H]        [H, !]
 redo: []      []             [w]         [w, H]     [w]        []   ← очищено
 text: Hello   Hello world    Hello       ""         Hello      Hello!
```

Патерн «Команда»: кожна дія знає, як себе **виконати** і **скасувати**.

```csharp
using System.Text;

var editor = new TextEditor();
editor.Execute(new InsertCommand("Hello"));
editor.Execute(new InsertCommand(" world"));
editor.Print("typed");

editor.Undo();
editor.Print("undo");
editor.Undo();
editor.Print("undo");
editor.Undo();                                       // нічого скасовувати
editor.Print("undo (nothing)");

editor.Redo();
editor.Print("redo");

editor.Execute(new InsertCommand("!"));             // нова дія → redo очищується
editor.Print("typed '!'");
editor.Redo();
editor.Print("redo (nothing)");

editor.Execute(new DeleteLastCommand(3));
editor.Print("delete 3");
editor.Undo();
editor.Print("undo");

public interface IEditorCommand
{
    void Apply(StringBuilder text);
    void Revert(StringBuilder text);
}

public sealed class InsertCommand(string fragment) : IEditorCommand
{
    public void Apply(StringBuilder text) => text.Append(fragment);
    public void Revert(StringBuilder text) => text.Length -= fragment.Length;
}

public sealed class DeleteLastCommand(int count) : IEditorCommand
{
    private string _deleted = "";                   // запам'ятовуємо видалене, щоб повернути

    public void Apply(StringBuilder text)
    {
        int take = Math.Min(count, text.Length);
        _deleted = text.ToString(text.Length - take, take);
        text.Length -= take;
    }

    public void Revert(StringBuilder text) => text.Append(_deleted);
}

public sealed class TextEditor
{
    private readonly StringBuilder _text = new();
    private readonly Stack<IEditorCommand> _undo = new();
    private readonly Stack<IEditorCommand> _redo = new();

    public void Execute(IEditorCommand command)
    {
        command.Apply(_text);
        _undo.Push(command);
        _redo.Clear();                               // нова гілка історії
    }

    public void Undo()
    {
        if (!_undo.TryPop(out IEditorCommand? command))
            return;
        command.Revert(_text);
        _redo.Push(command);
    }

    public void Redo()
    {
        if (!_redo.TryPop(out IEditorCommand? command))
            return;
        command.Apply(_text);
        _undo.Push(command);
    }

    public void Print(string action) =>
        Console.WriteLine($"{action,-15} text=\"{_text}\" undo={_undo.Count} redo={_redo.Count}");
}
```

**Приклад запуску:**

```
typed           text="Hello world" undo=2 redo=0
undo            text="Hello" undo=1 redo=1
undo            text="" undo=0 redo=2
undo (nothing)  text="" undo=0 redo=2
redo            text="Hello" undo=1 redo=1
typed '!'       text="Hello!" undo=2 redo=0
redo (nothing)  text="Hello!" undo=2 redo=0
delete 3        text="Hel" undo=3 redo=0
undo            text="Hello!" undo=2 redo=1
```

### Типові помилки (розділ 5)

1. **`Pop` на порожньому стеку** — `InvalidOperationException`. Перевіряйте `Count` або використовуйте `TryPop`.
2. **Неправильний порядок операндів у RPN:** першим знімається **правий** операнд (`right = Pop(); left = Pop();`). Для `-` і `/` це критично.
3. **Очікувати, що `foreach` по `Stack<T>` іде від дна** — він іде від вершини; `ToArray()` теж.
4. **Забути перевірити порожність стеку в кінці** при перевірці дужок — рядок `"(("` не збалансований.
5. **Глибока рекурсія** замість явного стеку на великих вхідних даних — `StackOverflowException` завершує процес.
6. **Монотонний стек зі значеннями замість індексів** — коли потрібна відстань чи позиція, зберігайте індекси.
7. **Стек на списку, де вершина — хвіст однозв'язного списку** — `Pop` стає O(n).
8. **Не очищати `redo` при новій дії** — Redo повторить команду, яка вже не має сенсу.

### Міні-вправи (розділ 5)

**Вправа 5.1.** Спростіть Unix-шлях: `"/a/./b/../../c/"` → `"/c"`. Сегмент `..` — на рівень вгору, `.` — поточний каталог.

<details>
<summary>Розв'язок</summary>

```csharp
foreach (string path in new[] { "/a/./b/../../c/", "/home//user/docs/../photos", "/../", "/x/y/z/../../.." })
    Console.WriteLine($"{path,-28} => {Simplify(path)}");

static string Simplify(string path)
{
    var stack = new Stack<string>();
    foreach (string part in path.Split('/', StringSplitOptions.RemoveEmptyEntries))
    {
        if (part == ".")
            continue;                          // поточний каталог — нічого не робимо
        if (part == "..")
            stack.TryPop(out _);               // вгору; з кореня вище піти не можна
        else
            stack.Push(part);
    }
    // стек віддає елементи від вершини, тому розвертаємо
    return "/" + string.Join("/", stack.Reverse());
}
```

**Приклад запуску:**

```
/a/./b/../../c/              => /c
/home//user/docs/../photos   => /home/user/photos
/../                         => /
/x/y/z/../../..              => /
```

</details>

**Вправа 5.2.** Реалізуйте чергу FIFO на **двох стеках** з амортизованою складністю O(1) на операцію.

<details>
<summary>Розв'язок</summary>

```csharp
var queue = new TwoStackQueue<int>();
queue.Enqueue(1);
queue.Enqueue(2);
queue.Enqueue(3);
Console.WriteLine($"Dequeue: {queue.Dequeue()}");      // переливання: out = [3, 2, 1] → знімаємо 1
queue.Enqueue(4);                                       // потрапляє в in, out не чіпаємо
Console.WriteLine($"Dequeue: {queue.Dequeue()}, {queue.Dequeue()}, {queue.Dequeue()}");
Console.WriteLine($"Count = {queue.Count}");

// Кожен елемент максимум раз переходить з _in у _out → O(1) амортизовано
public sealed class TwoStackQueue<T>
{
    private readonly Stack<T> _in = new();     // сюди додаємо
    private readonly Stack<T> _out = new();    // звідси забираємо

    public int Count => _in.Count + _out.Count;

    public void Enqueue(T item) => _in.Push(item);

    public T Dequeue()
    {
        if (_out.Count == 0)
        {
            while (_in.Count > 0)
                _out.Push(_in.Pop());          // переливання розвертає порядок: найстаріший — на вершині
        }
        return _out.Pop();
    }
}
```

**Приклад запуску:**

```
Dequeue: 1
Dequeue: 2, 3, 4
Count = 0
```

</details>

---

## 6. Черга

*≈ 25 хвилин*

### 6.1 FIFO і кільцевий буфер: будуємо `MyQueue<T>`

**Черга** — структура «перший увійшов — перший вийшов» (**FIFO**, First In, First Out). Додаємо в **хвіст** (`Enqueue`), забираємо з **голови** (`Dequeue`). Аналогія — черга в магазині.

```
  Dequeue ◄── │ 10 │ 20 │ 30 │ 40 │ ◄── Enqueue
              head               tail
```

**Чому не `List<T>`?** `RemoveAt(0)` зсуває всі елементи — O(n). **Кільцевий буфер** (circular buffer) розв'язує це: масив фіксованої місткості, два індекси `_head` і `_tail`, які **ходять по колу** (`(i + 1) % capacity`). Елементи ніколи не зсуваються.

```
 Capacity = 5.  Enqueue 1, 2, 3, 4:

  [0]  [1]  [2]  [3]  [4]
 │ 1 │ 2 │ 3 │ 4 │   │       head = 0, tail = 4, count = 4
   ▲                   ▲
  head               tail (куди писати наступний)

 Dequeue → 1, Dequeue → 2:
 │   │   │ 3 │ 4 │   │       head = 2, tail = 4, count = 2

 Enqueue 5, 6, 7:   tail «загортається» на початок: (4 + 1) % 5 = 0
 │ 6 │ 7 │ 3 │ 4 │ 5 │       head = 2, tail = 2, count = 5 (повний: head == tail, але count == capacity)
           ▲
       head = tail

 Логічний порядок: 3, 4, 5, 6, 7  (від head по колу)

 Enqueue 8 при повному буфері → ріст:
 новий масив на 10, копіюємо ПО ПОРЯДКУ від head:  │ 3 │ 4 │ 5 │ 6 │ 7 │ 8 │   │   │   │   │
                                                   head = 0, tail = 6
```

> **Як відрізнити порожню чергу від повної**, якщо в обох випадках `head == tail`? Найпростіше — зберігати окремий лічильник `_count` (так робить `Queue<T>`). Альтернатива — завжди лишати одну комірку порожньою.

| Операція | `MyQueue<T>` / `Queue<T>` | `List<T>` як черга |
|----------|---------------------------|--------------------|
| `Enqueue` | O(1) аморт. | O(1) аморт. |
| `Dequeue` | **O(1)** | O(n) |
| `Peek` | O(1) | O(1) |
| Доступ за номером у черзі | O(1) (`(head + i) % cap`) | O(1) |

```csharp
using System.Collections;

// ===== Демонстрація =====
var queue = new MyQueue<int>(capacity: 5);
for (int i = 1; i <= 4; i++)
    queue.Enqueue(i);
queue.Dump("enqueue 1..4");

Console.WriteLine($"Dequeue: {queue.Dequeue()}, {queue.Dequeue()}");
queue.Dump("dequeue x2");

for (int i = 5; i <= 7; i++)
    queue.Enqueue(i);                              // tail загортається на початок масиву
queue.Dump("enqueue 5..7");

queue.Enqueue(8);                                  // буфер повний → ріст із «розгортанням»
queue.Dump("enqueue 8");

Console.WriteLine($"Peek={queue.Peek()}, queue[2]={queue[2]}, Contains(6)={queue.Contains(6)}");
Console.WriteLine($"foreach: {string.Join(" ", queue)}");

try
{
    foreach (int x in queue)
        if (x == 4)
            queue.Enqueue(100);
}
catch (InvalidOperationException ex)
{
    Console.WriteLine($"foreach: {ex.Message}");
}

while (queue.TryDequeue(out int item))
    Console.Write($"{item} ");
Console.WriteLine();
Console.WriteLine($"empty: Count={queue.Count}, TryPeek={queue.TryPeek(out _)}");

// ===== Реалізація =====

/// <summary>Черга FIFO на кільцевому буфері з ростом.</summary>
public sealed class MyQueue<T> : IEnumerable<T>
{
    private T[] _buffer;
    private int _head;          // індекс першого елемента
    private int _tail;          // індекс, куди запишеться наступний елемент
    private int _count;
    private int _version;

    public MyQueue(int capacity = 4)
    {
        ArgumentOutOfRangeException.ThrowIfNegativeOrZero(capacity);
        _buffer = new T[capacity];
    }

    public int Count => _count;
    public int Capacity => _buffer.Length;

    // i-й елемент від голови: зсув по колу
    public T this[int index]
    {
        get
        {
            if ((uint)index >= (uint)_count)
                throw new ArgumentOutOfRangeException(nameof(index));
            return _buffer[(_head + index) % _buffer.Length];
        }
    }

    public void Enqueue(T item)
    {
        if (_count == _buffer.Length)
            Grow();
        _buffer[_tail] = item;
        _tail = Next(_tail);                     // (tail + 1) % capacity
        _count++;
        _version++;
    }

    public T Dequeue()
    {
        if (_count == 0)
            throw new InvalidOperationException("Queue is empty.");
        T item = _buffer[_head];
        _buffer[_head] = default!;              // звільняємо посилання
        _head = Next(_head);
        _count--;
        _version++;
        return item;
    }

    public bool TryDequeue(out T item)
    {
        if (_count == 0)
        {
            item = default!;
            return false;
        }
        item = Dequeue();
        return true;
    }

    public T Peek() => _count == 0 ? throw new InvalidOperationException("Queue is empty.") : _buffer[_head];

    public bool TryPeek(out T item)
    {
        item = _count == 0 ? default! : _buffer[_head];
        return _count > 0;
    }

    public bool Contains(T item)
    {
        var comparer = EqualityComparer<T>.Default;
        for (int i = 0; i < _count; i++)
            if (comparer.Equals(this[i], item))
                return true;
        return false;
    }

    public void Dump(string label)
    {
        string cells = string.Join("|", Enumerable.Range(0, _buffer.Length).Select(i => IsOccupied(i) ? $"{_buffer[i],2}" : "  "));
        Console.WriteLine($"{label,-13} |{cells}| head={_head} tail={_tail} count={_count} cap={Capacity}");
    }

    // Копіюємо елементи ПО ПОРЯДКУ черги: від head до кінця масиву, потім від початку до tail
    private void Grow()
    {
        var newBuffer = new T[_buffer.Length * 2];
        if (_head < _tail)
        {
            Array.Copy(_buffer, _head, newBuffer, 0, _count);
        }
        else
        {
            int rightPart = _buffer.Length - _head;          // [head .. кінець)
            Array.Copy(_buffer, _head, newBuffer, 0, rightPart);
            Array.Copy(_buffer, 0, newBuffer, rightPart, _tail); // [0 .. tail)
        }
        _buffer = newBuffer;
        _head = 0;
        _tail = _count;
    }

    private int Next(int index) => index + 1 == _buffer.Length ? 0 : index + 1;  // без дорогого %

    private bool IsOccupied(int physicalIndex)
    {
        int offset = (physicalIndex - _head + _buffer.Length) % _buffer.Length;
        return offset < _count;
    }

    public IEnumerator<T> GetEnumerator()
    {
        int version = _version;
        for (int i = 0; i < _count; i++)
        {
            if (version != _version)
                throw new InvalidOperationException("Collection was modified during enumeration.");
            yield return this[i];
        }
        if (version != _version)
            throw new InvalidOperationException("Collection was modified during enumeration.");
    }

    IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();
}
```

**Приклад запуску:**

```
enqueue 1..4  | 1| 2| 3| 4|  | head=0 tail=4 count=4 cap=5
Dequeue: 1, 2
dequeue x2    |  |  | 3| 4|  | head=2 tail=4 count=2 cap=5
enqueue 5..7  | 6| 7| 3| 4| 5| head=2 tail=2 count=5 cap=5
enqueue 8     | 3| 4| 5| 6| 7| 8|  |  |  |  | head=0 tail=6 count=6 cap=10
Peek=3, queue[2]=5, Contains(6)=True
foreach: 3 4 5 6 7 8
foreach: Collection was modified during enumeration.
3 4 5 6 7 8 100 
empty: Count=0, TryPeek=False
```

### 6.2 `Queue<T>` у .NET

`Queue<T>` — саме кільцевий буфер (`_array`, `_head`, `_tail`, `_size`) з подвоєнням.

| Член | Опис | Складність |
|------|------|-----------|
| `Enqueue(item)` | у хвіст | O(1) аморт. |
| `Dequeue()` / `TryDequeue(out item)` | з голови | O(1) |
| `Peek()` / `TryPeek(out item)` | голова | O(1) |
| `Count`, `Clear()`, `Contains(item)` | | O(1) / O(n) / O(n) |
| `ToArray()`, `foreach` | від голови до хвоста | O(n) |
| `EnsureCapacity`, `TrimExcess` | | O(n) |

**Застосування:** BFS, черга завдань, планувальники процесів (round-robin), черга друку, буферизація потоків даних, обмеження швидкості запитів.

```csharp
// Черга завдань: обробляємо у порядку надходження
var jobs = new Queue<string>();
jobs.Enqueue("print report");   // у кінець — O(1)
jobs.Enqueue("send email");
jobs.Enqueue("backup");
Console.WriteLine($"next: {jobs.Peek()}, total: {jobs.Count}");
while (jobs.TryDequeue(out string? job)) // з початку — O(1)
    Console.WriteLine($"done: {job}");

// BFS: сусіди вершини ставляться в чергу → обхід «шарами»
int[][] graph = [[1, 2], [3], [3, 4], [5], [5], []];
var distance = new int[graph.Length];
Array.Fill(distance, -1);                  // -1 = ще не відвідана
var queue = new Queue<int>([0]);
distance[0] = 0;
while (queue.Count > 0)
{
    int v = queue.Dequeue();
    foreach (int next in graph[v])
    {
        if (distance[next] != -1) continue; // вже бачили
        distance[next] = distance[v] + 1;
        queue.Enqueue(next);
    }
}
Console.WriteLine($"distances: {string.Join(' ', distance)}");

// Round-robin: кожен процес отримує квант часу 3; незавершений — назад у кінець черги
var processes = new Queue<(string Name, int Remaining)>([("A", 5), ("B", 2), ("C", 7)]);
const int Quantum = 3;
int time = 0;
while (processes.TryDequeue(out var p))
{
    int run = Math.Min(Quantum, p.Remaining);
    time += run;
    if (p.Remaining > run)
        processes.Enqueue((p.Name, p.Remaining - run));
    else
        Console.WriteLine($"t={time,2}: {p.Name} finished");
}

// Останні N подій: черга обмеженої довжини
var lastThree = new Queue<int>();
foreach (int evt in new[] { 1, 2, 3, 4, 5 })
{
    lastThree.Enqueue(evt);
    if (lastThree.Count > 3)
        lastThree.Dequeue();               // викидаємо найстаріше
}
Console.WriteLine($"last three events: {string.Join(" ", lastThree)}");
```

**Приклад запуску:**

```
next: print report, total: 3
done: print report
done: send email
done: backup
distances: 0 1 1 2 2 3
t= 5: B finished
t=10: A finished
t=14: C finished
last three events: 3 4 5
```

### 6.3 Дек (двобічна черга) на кільцевому буфері

**Дек** (deque, double-ended queue) — додавання й видалення з **обох** кінців за O(1). У кільцевому буфері `_head` може рухатися **назад**: `(head − 1 + capacity) % capacity`.

```
 Capacity = 6.  PushBack(1), PushBack(2), PushFront(0), PushFront(-1):

  [0]  [1]  [2]  [3]  [4]  [5]
 │ 1 │ 2 │   │   │ -1│ 0 │        head = 4 (загорнувся назад з 0 → 5 → 4), count = 4
                   ▲
                  head             логічний порядок: -1, 0, 1, 2
```

Дек універсальний: його можна використовувати і як стек, і як чергу. У .NET вбудованого деку немає (є `LinkedList<T>`, але він повільніший).

```csharp
var deque = new Deque<int>(capacity: 4);
deque.PushBack(1);
deque.PushBack(2);
deque.PushFront(0);
deque.PushFront(-1);
Console.WriteLine($"deque: {deque} (Count={deque.Count})");

deque.PushBack(3);                        // ріст
Console.WriteLine($"after PushBack(3): {deque}, Front={deque.Front}, Back={deque.Back}");

Console.WriteLine($"PopFront={deque.PopFront()}, PopBack={deque.PopBack()} → {deque}");
Console.WriteLine($"deque[1]={deque[1]}");

// Дек як стек (обидві операції з кінця) і як черга (з різних кінців)
var palindrome = new Deque<char>();
foreach (char ch in "racecar")
    palindrome.PushBack(ch);
bool isPalindrome = true;
while (palindrome.Count > 1)
    if (palindrome.PopFront() != palindrome.PopBack())
        isPalindrome = false;
Console.WriteLine($"'racecar' palindrome via deque: {isPalindrome}");

/// <summary>Двобічна черга на кільцевому буфері.</summary>
public sealed class Deque<T>
{
    private T[] _buffer;
    private int _head;
    private int _count;

    public Deque(int capacity = 4) => _buffer = new T[Math.Max(1, capacity)];

    public int Count => _count;
    public T Front => _count > 0 ? _buffer[_head] : throw new InvalidOperationException("Deque is empty.");
    public T Back => _count > 0 ? _buffer[PhysicalIndex(_count - 1)] : throw new InvalidOperationException("Deque is empty.");

    public T this[int index] =>
        (uint)index < (uint)_count ? _buffer[PhysicalIndex(index)] : throw new ArgumentOutOfRangeException(nameof(index));

    public void PushBack(T item)
    {
        if (_count == _buffer.Length) Grow();
        _buffer[PhysicalIndex(_count)] = item;     // позиція одразу за останнім
        _count++;
    }

    public void PushFront(T item)
    {
        if (_count == _buffer.Length) Grow();
        _head = (_head - 1 + _buffer.Length) % _buffer.Length;   // крок назад по колу
        _buffer[_head] = item;
        _count++;
    }

    public T PopFront()
    {
        T item = Front;
        _buffer[_head] = default!;
        _head = (_head + 1) % _buffer.Length;
        _count--;
        return item;
    }

    public T PopBack()
    {
        T item = Back;
        _buffer[PhysicalIndex(_count - 1)] = default!;
        _count--;                                   // голова не рухається
        return item;
    }

    public override string ToString() =>
        "[" + string.Join(", ", Enumerable.Range(0, _count).Select(i => this[i])) + "]";

    private int PhysicalIndex(int logicalIndex) => (_head + logicalIndex) % _buffer.Length;

    private void Grow()
    {
        var newBuffer = new T[_buffer.Length * 2];
        for (int i = 0; i < _count; i++)
            newBuffer[i] = _buffer[PhysicalIndex(i)];   // «розгортаємо» кільце
        _buffer = newBuffer;
        _head = 0;
    }
}
```

**Приклад запуску:**

```
deque: [-1, 0, 1, 2] (Count=4)
after PushBack(3): [-1, 0, 1, 2, 3], Front=-1, Back=3
PopFront=-1, PopBack=3 → [0, 1, 2]
deque[1]=1
'racecar' palindrome via deque: True
```

### 6.4 BFS на сітці: найкоротший шлях у лабіринті

**Пошук у ширину (BFS)** обходить клітинки **шарами** — спочатку всі на відстані 1, потім 2, … Тому перша поява цілі в черзі дає **найкоротший шлях** (у незваженому графі). Черга гарантує порядок «ближчі — раніше».

```
 Лабіринт (# — стіна):          Відстані BFS від S:

  S . . # .                      0 1 2 # 8
  # # . # .                      # # 3 # 7
  . . . . .                      6 5 4 5 6
  . # # # .                      7 # # # 7
  . . . # E                      8 9 10 # 8 ← E

 Черга по кроках: [S] → [(0,1)] → [(0,2)] → [(1,2)] → [(2,2)] → [(2,1),(2,3)] → ...
```

Складність: O(rows × cols) — кожна клітинка потрапляє в чергу не більше одного разу.

```csharp
string[] maze =
[
    "S..#.",
    "##.#.",
    ".....",
    ".###.",
    "...#E",
];

var (distance, path) = ShortestPath(maze);
Console.WriteLine($"shortest distance: {distance}");
Console.WriteLine($"path: {string.Join(" ", path.Select(p => $"({p.Row},{p.Col})"))}");

// Малюємо шлях
char[][] canvas = maze.Select(row => row.ToCharArray()).ToArray();
foreach (var (r, c) in path)
    if (canvas[r][c] == '.')
        canvas[r][c] = '*';
foreach (char[] row in canvas)
    Console.WriteLine("  " + new string(row));

string[] blocked = ["S#", "#E"];
Console.WriteLine($"blocked maze distance: {ShortestPath(blocked).Distance}");

static (int Distance, List<(int Row, int Col)> Path) ShortestPath(string[] grid)
{
    int rows = grid.Length, cols = grid[0].Length;
    (int Row, int Col) start = Find(grid, 'S'), end = Find(grid, 'E');

    var dist = new int[rows, cols];
    var parent = new (int Row, int Col)[rows, cols];   // звідки прийшли — щоб відновити шлях
    for (int r = 0; r < rows; r++)
        for (int c = 0; c < cols; c++)
            dist[r, c] = -1;

    ReadOnlySpan<(int Dr, int Dc)> directions = [(-1, 0), (1, 0), (0, -1), (0, 1)];
    var queue = new Queue<(int Row, int Col)>();
    queue.Enqueue(start);
    dist[start.Row, start.Col] = 0;

    while (queue.TryDequeue(out var cell))
    {
        if (cell == end)
            break;                                        // перша поява цілі — найкоротша
        foreach (var (dr, dc) in directions)
        {
            int nr = cell.Row + dr, nc = cell.Col + dc;
            if (nr < 0 || nr >= rows || nc < 0 || nc >= cols) continue;  // за межами
            if (grid[nr][nc] == '#' || dist[nr, nc] != -1) continue;     // стіна або вже бачили
            dist[nr, nc] = dist[cell.Row, cell.Col] + 1;
            parent[nr, nc] = cell;
            queue.Enqueue((nr, nc));
        }
    }

    var path = new List<(int Row, int Col)>();
    if (dist[end.Row, end.Col] == -1)
        return (-1, path);
    for (var cell = end; cell != start; cell = parent[cell.Row, cell.Col])
        path.Add(cell);
    path.Add(start);
    path.Reverse();                                       // відновлювали від кінця
    return (dist[end.Row, end.Col], path);
}

static (int Row, int Col) Find(string[] grid, char target)
{
    for (int r = 0; r < grid.Length; r++)
    {
        int c = grid[r].IndexOf(target);
        if (c >= 0)
            return (r, c);
    }
    throw new ArgumentException($"'{target}' not found");
}
```

**Приклад запуску:**

```
shortest distance: 8
path: (0,0) (0,1) (0,2) (1,2) (2,2) (2,3) (2,4) (3,4) (4,4)
  S**#.
  ##*#.
  ..***
  .###*
  ...#E
blocked maze distance: -1
```

### 6.5 Максимум у ковзному вікні (монотонний дек)

**Задача:** для кожного вікна довжини k знайти максимум. Наївно — O(n·k). Купа — O(n log n). **Монотонний дек — O(n).**

**Ідея:** дек зберігає **індекси** кандидатів у максимум, значення в деку **спадають** від голови до хвоста.

- Новий елемент `x` приходить у хвіст: усі менші за `x` елементи з хвоста **ніколи** вже не стануть максимумом (x новіший і більший) — викидаємо їх.
- Якщо індекс у голові вийшов за межі вікна — викидаємо з голови.
- Максимум вікна — завжди в **голові**.

```
 a = [1, 3, -1, -3, 5, 3, 6, 7], k = 3        дек (значення)      max
 i=0 x=1                                       [1]
 i=1 x=3   викинули 1                          [3]
 i=2 x=-1                                      [3, -1]            3
 i=3 x=-3                                      [3, -1, -3]        3
 i=4 x=5   голова (i=1) вийшла; викинули -3,-1 [5]                5
 i=5 x=3                                       [5, 3]             5
 i=6 x=6   викинули 3, 5                       [6]                6
 i=7 x=7   викинули 6                          [7]                7
```

Для деку тут зручно використати `LinkedList<int>` (або наш `Deque<T>`).

```csharp
int[] a = [1, 3, -1, -3, 5, 3, 6, 7];
Console.WriteLine($"window max (k=3): {string.Join(" ", SlidingWindowMax(a, 3))}");
Console.WriteLine($"window max (k=1): {string.Join(" ", SlidingWindowMax(a, 1))}");
Console.WriteLine($"window max (k=8): {string.Join(" ", SlidingWindowMax(a, 8))}");

// Перевірка наївним методом на випадкових даних
var random = new Random(7);
int[] big = Enumerable.Range(0, 2000).Select(_ => random.Next(-1000, 1000)).ToArray();
bool same = SlidingWindowMax(big, 50).SequenceEqual(Naive(big, 50));
Console.WriteLine($"matches naive on 2000 random numbers: {same}");

static List<int> SlidingWindowMax(int[] a, int k)
{
    var result = new List<int>(a.Length - k + 1);
    var deque = new LinkedList<int>();              // індекси; a[...] спадають від First до Last
    for (int i = 0; i < a.Length; i++)
    {
        // 1) голова вийшла з вікна [i − k + 1, i]
        if (deque.Count > 0 && deque.First!.Value <= i - k)
            deque.RemoveFirst();
        // 2) менші за a[i] з хвоста вже ніколи не стануть максимумом
        while (deque.Count > 0 && a[deque.Last!.Value] <= a[i])
            deque.RemoveLast();
        deque.AddLast(i);
        // 3) вікно сформоване — максимум у голові
        if (i >= k - 1)
            result.Add(a[deque.First!.Value]);
    }
    return result;
}

static List<int> Naive(int[] a, int k)
{
    var result = new List<int>();
    for (int i = 0; i + k <= a.Length; i++)
        result.Add(a.AsSpan(i, k).ToArray().Max());
    return result;
}
```

**Приклад запуску:**

```
window max (k=3): 3 3 5 5 6 7
window max (k=1): 1 3 -1 -3 5 3 6 7
window max (k=8): 7
matches naive on 2000 random numbers: True
```

### 6.6 Черга з пріоритетом: `PriorityQueue<TElement, TPriority>` (анонс)

Звичайна черга видає елементи в порядку **надходження**. **Черга з пріоритетом** — у порядку **пріоритету** (у .NET — спочатку **найменший**). Всередині — **двійкова купа** (d-арна, d = 4) на масиві; детально купи розглянемо в наступних лекціях.

| Операція | Складність |
|----------|-----------|
| `Enqueue(element, priority)` | O(log n) |
| `Dequeue()` / `TryDequeue(out e, out p)` | O(log n) |
| `Peek()` | O(1) |
| `EnqueueDequeue`, `DequeueEnqueue` | O(log n) |
| `EnqueueRange` | O(n) при побудові |

> Порядок елементів з **однаковим** пріоритетом **не гарантований** (купа не стабільна). Якщо важливий FIFO серед рівних — додайте до пріоритету порядковий номер: `(priority, sequence)`.

```csharp
// Лікарня: менше число — терміновіше
var emergency = new PriorityQueue<string, int>();
emergency.Enqueue("broken arm", 3);
emergency.Enqueue("heart attack", 1);
emergency.Enqueue("cold", 5);
emergency.Enqueue("high fever", 2);

Console.WriteLine($"Peek: {emergency.Peek()} (Count={emergency.Count})");
while (emergency.TryDequeue(out string? patient, out int priority))
    Console.WriteLine($"  priority {priority}: {patient}");

// Максимум замість мінімуму — власний компаратор
var maxHeap = new PriorityQueue<string, int>(Comparer<int>.Create((x, y) => y.CompareTo(x)));
maxHeap.EnqueueRange([("bronze", 10), ("gold", 100), ("silver", 50)]);
Console.WriteLine($"max first: {maxHeap.Dequeue()}, {maxHeap.Dequeue()}, {maxHeap.Dequeue()}");

// k найбільших елементів за O(n log k): тримаємо мін-купу розміру k
int[] numbers = [7, 2, 9, 4, 11, 5, 8, 1];
var topK = new PriorityQueue<int, int>();
foreach (int x in numbers)
{
    topK.Enqueue(x, x);
    if (topK.Count > 3)
        topK.Dequeue();                       // викидаємо найменший із кандидатів
}
var largest = new List<int>();
while (topK.TryDequeue(out int value, out _))
    largest.Add(value);
Console.WriteLine($"top 3: {string.Join(" ", largest)}");

// Стабільність серед рівних: пріоритет-кортеж (рівень, порядковий номер)
var tickets = new PriorityQueue<string, (int Level, int Sequence)>();
int sequence = 0;
foreach (var (name, level) in new[] { ("T1", 2), ("T2", 1), ("T3", 2), ("T4", 1), ("T5", 2) })
    tickets.Enqueue(name, (level, sequence++));
var order = new List<string>();
while (tickets.TryDequeue(out string? ticket, out _))
    order.Add(ticket);
Console.WriteLine($"stable order: {string.Join(" ", order)}");
```

**Приклад запуску:**

```
Peek: heart attack (Count=4)
  priority 1: heart attack
  priority 2: high fever
  priority 3: broken arm
  priority 5: cold
max first: gold, silver, bronze
top 3: 8 9 11
stable order: T2 T4 T1 T3 T5
```

### 6.7 Виробник/споживач: будуємо `MyChannel<T>` і порівнюємо з `Channel<T>`

**Задача «виробник — споживач» (producer/consumer):** одні потоки створюють роботу, інші її виконують. Між ними — **потокобезпечна черга**. Якщо черга **обмежена** (bounded), швидкий виробник чекає, поки споживач звільнить місце, — це **зворотний тиск** (backpressure), який захищає від переповнення пам'яті.

```
 ┌──────────┐  WriteAsync   ┌──────────────────────────┐  ReadAsync   ┌──────────┐
 │ producer │ ────────────► │ │ 1 │ 2 │ 3 │   │  cap=3 │ ───────────► │ consumer │
 └──────────┘   чекає, якщо └──────────────────────────┘ чекає, якщо  └──────────┘
                повно                                  порожньо
                                Complete() — «більше записів не буде»:
                                читач дочитує залишок і завершується
```

Три режими будь-якого каналу:

| Ситуація | Запис | Читання |
|----------|-------|---------|
| є місце і є дані | одразу | одразу |
| буфер повний | **чекає** звільнення місця | одразу |
| буфер порожній | одразу | **чекає** появи даних (або завершення) |
| канал завершено (`Complete`) | виняток | дочитує залишок, потім «кінець» |

Спочатку побудуємо простий канал самі: `Queue<T>` + `lock` + два `SemaphoreSlim`, що рахують **вільні місця** та **доступні елементи**. `SemaphoreSlim.WaitAsync` дозволяє чекати **асинхронно** — без блокування потоку.

```csharp
using System.Collections.Concurrent;

// ===== 1. Власний канал =====
var channel = new MyChannel<int>(capacity: 2);
var log = new ConcurrentQueue<string>();            // журнал подій для детермінованого виводу

Task producer = Task.Run(async () =>
{
    for (int i = 1; i <= 5; i++)
    {
        await channel.WriteAsync(i);                 // чекає, якщо в буфері вже 2 елементи
        log.Enqueue($"wrote {i}");
    }
    channel.Complete();                              // сигнал «виробництво завершено»
});

var received = new List<int>();
Task consumer = Task.Run(async () =>
{
    while (await channel.WaitToReadAsync())          // false — канал завершено і порожній
    {
        while (channel.TryRead(out int item))
        {
            received.Add(item);
            await Task.Delay(5);                     // повільний споживач → виробник упирається в місткість
        }
    }
});

await Task.WhenAll(producer, consumer);
Console.WriteLine($"MyChannel received: {string.Join(" ", received)}");
Console.WriteLine($"max buffered at once: {channel.MaxObservedCount} (capacity 2)");
Console.WriteLine($"writes logged: {log.Count}");

try
{
    await channel.WriteAsync(99);
}
catch (InvalidOperationException ex)
{
    Console.WriteLine($"write after Complete: {ex.Message}");
}

// Кілька виробників і кілька споживачів: порядок між потоками не гарантований,
// але кожен елемент буде прочитаний рівно один раз
var shared = new MyChannel<int>(capacity: 4);
Task[] producers = Enumerable.Range(0, 3).Select(p => Task.Run(async () =>
{
    for (int i = 0; i < 100; i++)
        await shared.WriteAsync(p * 1000 + i);
})).ToArray();
var totals = new ConcurrentBag<int>();
Task[] consumers = Enumerable.Range(0, 2).Select(_ => Task.Run(async () =>
{
    while (await shared.WaitToReadAsync())
        while (shared.TryRead(out int item))
            totals.Add(item);
})).ToArray();
await Task.WhenAll(producers);
shared.Complete();
await Task.WhenAll(consumers);
Console.WriteLine($"3 producers x 100 items, 2 consumers: received {totals.Count}, distinct {totals.Distinct().Count()}, sum {totals.Sum()}");

// ===== Реалізація =====

/// <summary>Обмежений асинхронний канал: Queue&lt;T&gt; під lock + два семафори.</summary>
public sealed class MyChannel<T>
{
    private readonly Queue<T> _queue = new();
    private readonly object _sync = new();                // захищає _queue і _completed
    private readonly SemaphoreSlim _freeSlots;            // скільки ще можна записати
    private readonly SemaphoreSlim _availableItems = new(0); // скільки можна прочитати
    private readonly CancellationTokenSource _completion = new();
    private bool _completed;

    public MyChannel(int capacity)
    {
        ArgumentOutOfRangeException.ThrowIfNegativeOrZero(capacity);
        _freeSlots = new SemaphoreSlim(capacity, capacity);
    }

    public int MaxObservedCount { get; private set; }

    public async Task WriteAsync(T item, CancellationToken cancellationToken = default)
    {
        await _freeSlots.WaitAsync(cancellationToken);    // 1) займаємо вільне місце (або чекаємо)
        lock (_sync)
        {
            if (_completed)
            {
                _freeSlots.Release();
                throw new InvalidOperationException("The channel has been completed.");
            }
            _queue.Enqueue(item);                         // 2) кладемо елемент під замком
            MaxObservedCount = Math.Max(MaxObservedCount, _queue.Count);
        }
        _availableItems.Release();                        // 3) повідомляємо читачів: +1 елемент
    }

    public bool TryRead(out T item)
    {
        if (!_availableItems.Wait(0))                     // немає готових елементів — не чекаємо
        {
            item = default!;
            return false;
        }
        lock (_sync)
            item = _queue.Dequeue();
        _freeSlots.Release();                             // звільнилося місце для виробника
        return true;
    }

    // true — є що читати; false — канал завершено і всі елементи прочитані
    public async Task<bool> WaitToReadAsync()
    {
        while (true)
        {
            lock (_sync)
            {
                if (_queue.Count > 0)
                    return true;
                if (_completed)
                    return false;
            }
            try
            {
                // Чекаємо появи елемента або завершення каналу; елемент одразу «повертаємо» в семафор
                await _availableItems.WaitAsync(_completion.Token);
                _availableItems.Release();
                return true;
            }
            catch (OperationCanceledException)
            {
                // канал завершили — на наступній ітерації перевіримо, чи лишилися елементи
            }
        }
    }

    public void Complete()
    {
        lock (_sync)
            _completed = true;
        _completion.Cancel();                             // будимо всіх, хто чекає в WaitToReadAsync
    }
}
```

**Приклад запуску:**

```
MyChannel received: 1 2 3 4 5
max buffered at once: 2 (capacity 2)
writes logged: 5
write after Complete: The channel has been completed.
3 producers x 100 items, 2 consumers: received 300, distinct 300, sum 314850
```

> **Чому саме так?** `lock` захищає саму чергу від одночасного доступу (сам `Queue<T>` не потокобезпечний). Семафори ж реалізують **очікування**: `_freeSlots` — «чекати місця», `_availableItems` — «чекати даних». Порядок «спочатку семафор, потім lock» важливий: чекати, **тримаючи** lock, не можна — інакше інша сторона ніколи не зможе звільнити місце (дедлок).

Наш канал навчальний: у ньому немає підтримки скасування для читачів, політики «що робити при переповненні», оптимізацій для одного читача тощо. У реальному коді використовуйте **`System.Threading.Channels`**:

```csharp
using System.Threading.Channels;

// Обмежений канал на 2 елементи; при заповненні — чекати (зворотний тиск)
Channel<int> channel = Channel.CreateBounded<int>(new BoundedChannelOptions(2)
{
    FullMode = BoundedChannelFullMode.Wait,   // інші режими: DropOldest, DropNewest, DropWrite
    SingleReader = true,                       // підказки для оптимізацій
    SingleWriter = true,
});

Task producer = Task.Run(async () =>
{
    for (int i = 1; i <= 5; i++)
        await channel.Writer.WriteAsync(i);    // чекає, якщо буфер повний
    channel.Writer.Complete();                 // більше записів не буде
});

var received = new List<int>();
await foreach (int item in channel.Reader.ReadAllAsync())   // завершиться після Complete + спорожнення
{
    received.Add(item);
    await Task.Delay(5);
}
await producer;
Console.WriteLine($"Channel<int> received: {string.Join(" ", received)}");

bool accepted = channel.Writer.TryWrite(99);
Console.WriteLine($"TryWrite after Complete: {accepted}");

// DropOldest: буфер завжди містить останні N значень (наприклад, свіжі показники датчика)
Channel<int> latest = Channel.CreateBounded<int>(new BoundedChannelOptions(3) { FullMode = BoundedChannelFullMode.DropOldest });
for (int i = 1; i <= 6; i++)
    latest.Writer.TryWrite(i);                 // ніколи не чекає: витісняє найстаріше
latest.Writer.Complete();
var kept = new List<int>();
await foreach (int x in latest.Reader.ReadAllAsync())
    kept.Add(x);
Console.WriteLine($"DropOldest(3) kept: {string.Join(" ", kept)}");

// Необмежений канал — як ConcurrentQueue, але з асинхронним очікуванням
Channel<string> unbounded = Channel.CreateUnbounded<string>();
await unbounded.Writer.WriteAsync("a");
await unbounded.Writer.WriteAsync("b");
Console.WriteLine($"unbounded Count={unbounded.Reader.Count}, first={await unbounded.Reader.ReadAsync()}");
```

**Приклад запуску:**

```
Channel<int> received: 1 2 3 4 5
TryWrite after Complete: False
DropOldest(3) kept: 4 5 6
unbounded Count=2, first=a
```

Для випадків без асинхронного очікування є `ConcurrentQueue<T>` — неблокуюча потокобезпечна черга (`Enqueue`, `TryDequeue`, `TryPeek`):

```csharp
using System.Collections.Concurrent;

var queue = new ConcurrentQueue<int>();

// 4 потоки одночасно додають по 10 000 чисел
Parallel.For(0, 4, worker =>
{
    for (int i = 0; i < 10_000; i++)
        queue.Enqueue(worker * 10_000 + i);
});
Console.WriteLine($"ConcurrentQueue Count after parallel Enqueue: {queue.Count}");

// Кілька споживачів забирають елементи; кожен елемент дістанеться рівно одному
long sum = 0;
int taken = 0;
Parallel.For(0, 3, _ =>
{
    while (queue.TryDequeue(out int item))
    {
        Interlocked.Add(ref sum, item);        // атомарне додавання
        Interlocked.Increment(ref taken);
    }
});
Console.WriteLine($"taken={taken}, sum={sum}, empty={queue.IsEmpty}");

// Для порівняння: звичайна Queue<T> без lock при паралельному записі ламається
var unsafeQueue = new Queue<int>();
int lost = 0;
try
{
    Parallel.For(0, 4, _ =>
    {
        for (int i = 0; i < 100_000; i++)
            unsafeQueue.Enqueue(i);
    });
    lost = 400_000 - unsafeQueue.Count;
}
catch (AggregateException)
{
    lost = -1;                                 // внутрішній стан зіпсовано — виняток
}
Console.WriteLine($"plain Queue<T> without lock is reliable: {lost == 0}");
```

**Приклад запуску:**

```
ConcurrentQueue Count after parallel Enqueue: 40000
taken=40000, sum=799980000, empty=True
plain Queue<T> without lock is reliable: False
```

| Засіб | Потокобезпечний | Очікування | Обмеження розміру | Коли брати |
|-------|-----------------|------------|-------------------|------------|
| `Queue<T>` | ні | — | ні | один потік |
| `ConcurrentQueue<T>` | так (lock-free) | ні (опитування `TryDequeue`) | ні | кілька потоків, без очікування |
| `BlockingCollection<T>` | так | синхронне (блокує потік) | так | старий код, синхронні потоки |
| `Channel<T>` | так | **асинхронне** (`await`) | так (`CreateBounded`) | сучасний async-код, конвеєри обробки |

### Типові помилки (розділ 6)

1. **`List<T>.RemoveAt(0)` як черга** — O(n) на кожен `Dequeue`.
2. **Кільцевий буфер без лічильника** — неможливо відрізнити порожній стан від повного (`head == tail` в обох).
3. **Копіювання при рості «як є»** (`Array.Copy(_buffer, newBuffer, n)`) без розгортання — порядок елементів ламається, якщо `tail` загорнувся.
4. **BFS без позначки «відвідано» при додаванні в чергу** (а лише при вийманні) — клітинки потрапляють у чергу багато разів.
5. **Очікувати стабільний порядок від `PriorityQueue`** для рівних пріоритетів.
6. **Спільна `Queue<T>` між потоками без `lock`** — втрачені елементи, виняток або зіпсований стан.
7. **Чекати на семафорі чи `Monitor.Wait`, тримаючи lock не того об'єкта** — дедлок.
8. **Забути `Writer.Complete()`** — `ReadAllAsync` чекатиме вічно.

### Міні-вправи (розділ 6)

**Вправа 6.1.** Реалізуйте **лічильник запитів за останні 3000 мс**: метод `Ping(t)` отримує час `t` (зростає) і повертає кількість запитів у проміжку `[t − 3000, t]`.

<details>
<summary>Розв'язок</summary>

```csharp
var counter = new RecentCounter();
foreach (int t in new[] { 1, 100, 3001, 3002, 7000 })
    Console.WriteLine($"Ping({t}) = {counter.Ping(t)}");

public sealed class RecentCounter
{
    private readonly Queue<int> _requests = new();

    public int Ping(int t)
    {
        _requests.Enqueue(t);
        while (_requests.Peek() < t - 3000)   // найстаріші запити «випадають» з вікна
            _requests.Dequeue();
        return _requests.Count;               // кожен запит додається і видаляється один раз → O(1) аморт.
    }
}
```

**Приклад запуску:**

```
Ping(1) = 1
Ping(100) = 2
Ping(3001) = 3
Ping(3002) = 3
Ping(7000) = 1
```

</details>

**Вправа 6.2.** Знайдіть кількість «островів» (зв'язних груп `1`) у сітці, використовуючи BFS із `Queue<T>`.

<details>
<summary>Розв'язок</summary>

```csharp
int[][] grid =
[
    [1, 1, 0, 0, 0],
    [1, 1, 0, 0, 1],
    [0, 0, 1, 0, 1],
    [0, 0, 0, 1, 1],
    [1, 0, 0, 0, 0],
];
Console.WriteLine($"islands: {CountIslands(grid)}");

static int CountIslands(int[][] grid)
{
    int rows = grid.Length, cols = grid[0].Length, islands = 0;
    var visited = new bool[rows, cols];
    var queue = new Queue<(int R, int C)>();
    for (int r = 0; r < rows; r++)
    {
        for (int c = 0; c < cols; c++)
        {
            if (grid[r][c] == 0 || visited[r, c])
                continue;
            islands++;                               // новий острів — «заливаємо» його BFS
            visited[r, c] = true;
            queue.Enqueue((r, c));
            while (queue.TryDequeue(out var cell))
            {
                foreach (var (nr, nc) in new[] { (cell.R - 1, cell.C), (cell.R + 1, cell.C), (cell.R, cell.C - 1), (cell.R, cell.C + 1) })
                {
                    if (nr < 0 || nr >= rows || nc < 0 || nc >= cols) continue;
                    if (grid[nr][nc] == 0 || visited[nr, nc]) continue;
                    visited[nr, nc] = true;          // позначаємо ПРИ ДОДАВАННІ
                    queue.Enqueue((nr, nc));
                }
            }
        }
    }
    return islands;
}
```

**Приклад запуску:**

```
islands: 4
```

</details>

---

> ## Перерва 3 (≈ 5 хвилин, за потреби)
>
> Минуло ≈ 170 хвилин. Лишився підсумок і питання для самоперевірки.

---

## 7. Підсумок

*≈ 10 хвилин*

### 7.1 Порівняльна таблиця

| Структура | Доступ `[i]` | Пошук | Вставка/видал. на початку | … в кінці | … в середині (позиція відома) | Пам'ять / кеш | .NET |
|-----------|--------------|-------|---------------------------|-----------|-------------------------------|---------------|------|
| Масив `T[]` | O(1) | O(n), O(log n) відсорт. | — (розмір фіксований) | — | — | ідеально | `T[]`, `Span<T>` |
| Динамічний масив | O(1) | O(n), O(log n) відсорт. | O(n) | O(1) аморт. | O(n) | добре (до 2× запасу) | `List<T>` |
| Однозв'язний список | O(n) | O(n) | O(1) | O(1) з хвостом / O(n) видал. | O(1) після вузла | погано | — (власний) |
| Двозв'язний список | O(n) | O(n) | O(1) | O(1) | O(1) | погано, +2 посилання | `LinkedList<T>` |
| Стек | лише вершина | — | — | Push/Pop O(1) | — | добре (на масиві) | `Stack<T>` |
| Черга (кільцевий буфер) | голова (O(1) за номером у власній реалізації) | — | Dequeue O(1) | Enqueue O(1) аморт. | — | добре | `Queue<T>` |
| Дек | O(1) (кільц. буфер) | O(n) | O(1) | O(1) | O(n) | добре | — (власний / `LinkedList<T>`) |
| Черга з пріоритетом | мінімум O(1) | — | Dequeue O(log n) | Enqueue O(log n) | — | добре | `PriorityQueue<TE, TP>` |
| Потокобезпечна черга | — | — | TryDequeue O(1) | Enqueue O(1) | — | — | `ConcurrentQueue<T>`, `Channel<T>` |

| Прийом | Складність | Ключова ідея |
|--------|-----------|--------------|
| Префіксні суми | O(n) + O(1) на запит | `sum(l..r) = P[r+1] − P[l]` |
| Два вказівники | O(n) | кожен крок зсуває хоча б один вказівник |
| Ковзне вікно | O(n) | додати вхідний, відняти вихідний |
| Кадане | O(n) | продовжити або почати заново |
| Три розвороти | O(n), O(1) пам'яті | зсув = розворот усього + частин |
| Бінарний пошук | O(log n) | перша позиція, де умова стає істинною |
| fast/slow вказівники | O(n), O(1) пам'яті | середина, цикл (Флойд) |
| Монотонний стек | O(n) | кожен індекс один раз push і один раз pop |
| Монотонний дек | O(n) | максимум вікна в голові |
| BFS | O(V + E) | черга → обхід шарами → найкоротший шлях |

### 7.2 Як обрати структуру

```
 Потрібен доступ за індексом?
 ├── так ──► розмір відомий і не змінюється? ── так ──► T[] (або Span<T> для частини)
 │                                          └─ ні ───► List<T> (задайте Capacity, якщо знаєте розмір)
 └── ні
     ├── обробка «останній прийшов — перший пішов» (відкат, вкладеність, DFS)? ──► Stack<T>
     ├── обробка в порядку надходження (BFS, завдання)? ──────────────────────────► Queue<T>
     │     └── кілька потоків / async? ──► Channel<T> (з очікуванням) або ConcurrentQueue<T>
     ├── потрібні обидва кінці (ковзне вікно, 0-1 BFS)? ──────────────────────────► дек на кільцевому буфері
     ├── обробка за важливістю (планувальник, Дейкстра, top-k)? ─────────────────► PriorityQueue<TE, TP>
     └── часті вставки/видалення в середині за ГОТОВИМ вузлом (LRU)? ─────────────► LinkedList<T>
```

Практичні правила:

1. **За замовчуванням — `List<T>`**. Міняйте лише тоді, коли профілювання чи асимптотика вказують на проблему.
2. **Не використовуйте `List<T>` як чергу** — `RemoveAt(0)` O(n).
3. **`LinkedList<T>` рідко швидший** за `List<T>` на практиці через кеш; беріть його, коли тримаєте посилання на вузли.
4. **Відомий розмір — задайте місткість** (`new List<T>(n)`, `new Queue<T>(n)`).
5. **Тимчасові буфери в «гарячому» коді** — `Span<T>`, `stackalloc` (малі), `ArrayPool<T>` (великі).
6. **Глибока рекурсія** — перепишіть на явний `Stack<T>`.
7. **Між потоками** — ніколи не діліть `List<T>`/`Queue<T>` без синхронізації; беріть `Channel<T>` або `Concurrent*`.

### 7.3 Питання для самоперевірки

1. Чому доступ `a[i]` у масиві має складність O(1)? Яка формула обчислює адресу елемента?
2. Що таке просторова локальність і чому обхід `int[,]` стовпцями повільніший, ніж рядками?
3. Чим `PointS[]` (масив структур) відрізняється від `PointC[]` (масив класів) за будовою в пам'яті та поведінкою при присвоєнні елемента?
4. Які значення містить щойно створений `new string?[3]`, `new bool[2]`, `new int[4]`?
5. Чим `int[,]` відрізняється від `int[][]` за будовою в пам'яті, гнучкістю та швидкодією?
6. Чим `a[1..3]` відрізняється від `a.AsSpan(1..3)`? Чому `Span<T>` не можна зберегти в полі класу?
7. Які три правила треба пам'ятати, працюючи з `ArrayPool<T>`?
8. Чому при подвоєнні місткості n додавань коштують O(n) сумарно, а при збільшенні на сталу величину — O(n²)? Поясніть методом бухгалтерського обліку.
9. Чим `Count` відрізняється від `Capacity`? Що відбувається з `Capacity` після `Clear()`?
10. Як енумератор `List<T>` виявляє, що колекцію змінили під час `foreach`? Як правильно видалити елементи за умовою?
11. Що повертає `List<T>.BinarySearch`, якщо елемента немає, і як отримати з цього позицію для вставки?
12. Навіщо `CollectionsMarshal.AsSpan` і в чому його небезпека?
13. Як за допомогою префіксних сум за O(n) порахувати кількість підмасивів із сумою k?
14. Чим відрізняються `LowerBound` і `UpperBound`? Чому `(lo + hi) / 2` — потенційна помилка?
15. Чому алгоритм Кадане коректний? Що він поверне для масиву з лише від'ємних чисел?
16. Як розвернути масив на k позицій праворуч за O(n) часу і O(1) пам'яті?
17. Навіщо у двозв'язному списку вузол-сентинел? Які перевірки він прибирає?
18. Чому в .NET немає аналога `std::list::splice` і як перенести вузли з одного `LinkedList<T>` в інший?
19. Як алгоритм Флойда виявляє цикл у списку і як знайти вхід у цикл?
20. Чому стек на масиві зберігає вершину в кінці масиву, а стек на списку — на голові списку?
21. Як перетворити рекурсивний DFS на ітеративний? Чому це важливо для дуже глибоких структур?
22. Опишіть алгоритм сортувальної станції. Як обробляється правоасоціативний оператор `^`?
23. Як реалізувати `Min()` у стеку за O(1)?
24. Чому монотонний стек працює за O(n), хоча в циклі `for` є вкладений `while`?
25. Як у кільцевому буфері відрізнити порожню чергу від повної? Що треба зробити з елементами при рості буфера?
26. Яку структуру — стек чи чергу — використовують для BFS і для DFS без рекурсії, і чому BFS знаходить найкоротший шлях?
27. Як монотонний дек знаходить максимум у кожному вікні за O(n)?
28. Чи гарантує `PriorityQueue<TElement, TPriority>` порядок FIFO для рівних пріоритетів? Як його забезпечити?
29. Що таке зворотний тиск (backpressure) і як його забезпечує обмежений канал? Чому не можна чекати на семафорі, тримаючи `lock`?
30. Чим `Channel<T>` відрізняється від `ConcurrentQueue<T>`?

### 7.4 Практичні завдання

**Масиви та динамічний масив**

1. Додайте до `MyList<T>` методи `InsertRange(int index, ReadOnlySpan<T> items)` (один зсув хвоста) і `RemoveAll(Predicate<T>)` за O(n).
2. Реалізуйте `Sort()` для `MyList<T>` через `Array.Sort(_items, 0, _count)` та `BinarySearch(T item)`. Не забудьте збільшувати `_version`.
3. Напишіть бенчмарк: 100 000 вставок у початок `List<int>` проти `LinkedList<int>` і 100 000 вставок у середину (з пошуком позиції). Поясніть результати.
4. Стиснення масиву на місці: `[a, a, b, c, c, c]` → `[a, 2, b, c, 3]` (два вказівники).
5. «Трапінг дощової води»: скільки води затримається між стовпцями висот `[0,1,0,2,1,0,1,3,2,1,2,1]` (відповідь 6). Розв'яжіть префіксними максимумами і двома вказівниками.
6. Знайдіть мінімальну довжину підрядка, що містить усі символи рядка-шаблону (ковзне вікно + словник лічильників).
7. Бінарний пошук за відповіддю: мінімальна місткість вантажівки, щоб перевезти посилки `weights` за `days` днів.
8. Спіральне **заповнення** матриці n×n числами 1..n².

**Списки**

9. Додайте до `MyLinkedList<T>` метод `MoveToFront(node)` за O(1) і на його основі реалізуйте LRU-кеш (`Dictionary<TKey, MyLinkedListNode<...>>` + список).
10. Розбийте однозв'язний список навколо значення x: спочатку всі `< x`, потім `>= x`, зберігаючи відносний порядок.
11. Складіть два числа, записані у списках цифрами у зворотному порядку (`2→4→3` + `5→6→4` = `7→0→8`).
12. Реалізуйте сортування злиттям для однозв'язного списку за O(n log n) (середина через fast/slow + злиття).

**Стек**

13. Розширте калькулятор сортувальної станції: унарний мінус, дійсні числа, функції `max(a, b)`, змінні.
14. Декодування рядка: `"3[a2[c]]"` → `"accaccacc"` (два стеки: чисел і рядків).
15. Найдовша правильна дужкова підпослідовність у рядку з `(` і `)` (стек індексів).
16. Реалізуйте стек із `Max()` за O(1) і з операцією `PopMax()`.

**Черга**

17. Додайте до `MyQueue<T>` метод `TrimExcess()` і стратегію зменшення буфера, коли він заповнений менш ніж на 25%.
18. BFS із «0-1 вагами» на сітці (крок у клітинку `.` коштує 0, у клітинку `~` — 1) на деку: 0-ребра — у голову, 1-ребра — у хвіст.
19. «Гнила апельсинова» задача: за скільки хвилин гниль пошириться на всю сітку (BFS з кількох джерел одночасно).
20. Конвеєр із трьох стадій на `Channel<T>`: читання рядків → парсинг чисел → агрегація суми; кожна стадія — окремий `Task`, канали обмежені.
21. Додайте до `MyChannel<T>` метод `ReadAsync(CancellationToken)` і режим `DropOldest`.

---

**Ключові думки лекції:**

- Масив — неперервний блок пам'яті: O(1) доступ і найкраща локальність кешу. Усе інше будується поверх нього або порівнюється з ним.
- Динамічний масив подвоює місткість, тому `Add` — O(1) **амортизовано**; `List<T>` — вибір за замовчуванням.
- Класичні прийоми (префіксні суми, два вказівники, ковзне вікно, бінарний пошук) перетворюють O(n²) на O(n) чи O(log n).
- Зв'язні списки дають O(1) вставку/видалення за вузлом, але програють масивам на обході через кеш.
- Стек (LIFO) — рекурсія, дужки, вирази, монотонні задачі, undo/redo. Черга (FIFO) — BFS, планування, буферизація; кільцевий буфер робить обидва кінці O(1).
- Для багатопотоковості — `Channel<T>` (асинхронне очікування, зворотний тиск) або `ConcurrentQueue<T>`.
