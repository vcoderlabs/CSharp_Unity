# Chương 1 — Generic cơ bản

## 1. Mục tiêu học

- Hiểu generics giải quyết vấn đề gì (tái sử dụng + an toàn kiểu)
- Viết **generic method**, **generic class**, **generic interface**
- Phân biệt type parameter `T` với kiểu cụ thể khi dùng
- Tránh lạm dụng `object` và cast

## 2. Điều kiện tiên quyết

- Level 2: class, interface, method
- Level 4: đã dùng `List<T>`, `Dictionary<TKey,TValue>` như người tiêu dùng

## 3. Khái niệm

**Generics** cho phép tham số hóa kiểu:

```csharp
List<int> numbers = new();
List<string> names = new();
```

Cùng một định nghĩa `List<T>`, nhiều kiểu phần tử — compiler kiểm tra bạn không `Add` nhầm kiểu.

### Vì sao không dùng object?

```csharp
ArrayList legacy = new(); // non-generic
legacy.Add(1);
legacy.Add("oops");
int x = (int)legacy[0]; // cast, boxing với value type
```

Generics: không cast tay, tránh boxing không cần thiết với `List<int>`.

| Thành phần | Ví dụ |
|------------|--------|
| Generic method | `Swap<T>(ref T a, ref T b)` |
| Generic class | `Box<T> { public T Value; }` |
| Generic interface | `IRepository<T>` |
| Nhiều type param | `Dictionary<TKey, TValue>` |

## 4. Mô hình tư duy

```text
Định nghĩa:  class Box<T>     ← T = “chỗ trống cho một kiểu”
Sử dụng:     Box<int>          ← T được thay bằng int
             Box<Player>       ← T = Player

Compiler sinh IL generic; JIT có thể specialize cho value type.
Bạn viết một lần — dùng nhiều kiểu.
```

## 5. Cú pháp

```csharp
// Generic method
static void Swap<T>(ref T a, ref T b)
{
    T tmp = a;
    a = b;
    b = tmp;
}

// Generic class
class Box<T>
{
    public T Value { get; set; }
    public Box(T value) => Value = value;
}

// Generic interface
interface IStore<T>
{
    void Add(T item);
    T? Find(Predicate<T> match);
}

// Nhiều tham số
class Pair<TFirst, TSecond>
{
    public TFirst First { get; set; }
    public TSecond Second { get; set; }
}
```

Gọi method: `Swap(ref x, ref y)` — compiler thường **suy luận** `T`; hoặc `Swap<int>(ref x, ref y)`.

## 6. Ví dụ

### Cơ bản

```csharp
var intBox = new Box<int>(42);
var nameBox = new Box<string>("Ada");
Console.WriteLine(intBox.Value);
Console.WriteLine(nameBox.Value);

int a = 1, b = 2;
Swap(ref a, ref b);
Console.WriteLine($"{a}, {b}"); // 2, 1
```

### Trung cấp

Generic method tìm max (cần comparable — tạm dùng `IComparable<T>`, chi tiết constraints ở chương 2):

```csharp
static T Max<T>(T a, T b) where T : IComparable<T>
    => a.CompareTo(b) >= 0 ? a : b;
```

### Nâng cao

Interface + class repository tối giản:

```csharp
interface IRepository<T>
{
    void Add(T entity);
    IReadOnlyList<T> GetAll();
}

class ListRepository<T> : IRepository<T>
{
    private readonly List<T> _items = new();
    public void Add(T entity) => _items.Add(entity);
    public IReadOnlyList<T> GetAll() => _items;
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Không tạo được `new T()` | T chưa constraint `new()` | Thêm `where T : new()` (chương 2) hoặc factory |
| `T` không có property `.Name` | Compiler không biết T là gì | Constraint hoặc truyền `Func<T,...>` |
| Nhầm `List` non-generic | Tài liệu cũ | Luôn `List<T>` |
| Khai báo `class Foo<t>` chữ thường | Convention | Type param Pascal: `T`, `TKey`, `TValue` |

## 8. Gỡ lỗi

1. Đọc lỗi CS0310/CS0452 — thường liên quan constraint thiếu.
2. Hover kiểu trong IDE: xem `T` đã suy luận thành gì.
3. Viết test với `int` và `string` — nếu một bên fail, generic chưa đủ tổng quát hoặc thiếu constraint.

## 9. Best practices

- Đặt tên: `T`, `TKey`, `TValue`, `TResult`, `TEntity` — rõ vai trò.
- Prefer generic thay `object` khi kiểu biết lúc biên dịch.
- Đừng generic hóa sớm mọi class — chỉ khi thật sự tái sử dụng đa kiểu.
- Generic method trên class non-generic cũng rất hữu ích (helper).
- Public API: cân nhắc interface generic để dễ fake/test.

## 10. Bài tập

**Bài 1** — Viết `static T Identity<T>(T value)` trả về đúng value.

**Bài 2** — Class `Result<T>` có `bool Success`, `T? Value`, `string? Error`; factory `Ok(T)` / `Fail(string)`.

**Bài 3** — Method `static List<T> Repeat<T>(T item, int count)`.

**Bài 4** — Interface `IStack<T>` với `Push`, `Pop`, `TryPeek`; class `ArrayStack<T>` implement bằng `List<T>`.

## 11. Gợi ý

- Bài 1: một dòng `return value;`.
- Bài 2: private ctor + static methods.
- Bài 3: vòng for `Add(item)`.
- Bài 4: Pop lấy `list[^1]` rồi `RemoveAt`.

## 12. Đáp án

**Bài 1** — Method generic đơn giản nhất:

```csharp
static T Identity<T>(T value) => value;
```

**Bài 2** — Bọc kết quả thành công/thất bại:

```csharp
sealed class Result<T>
{
    public bool Success { get; }
    public T? Value { get; }
    public string? Error { get; }

    private Result(bool success, T? value, string? error)
    {
        Success = success;
        Value = value;
        Error = error;
    }

    public static Result<T> Ok(T value) => new(true, value, null);
    public static Result<T> Fail(string error) => new(false, default, error);
}
```

**Bài 3** — Lặp phần tử:

```csharp
static List<T> Repeat<T>(T item, int count)
{
    var list = new List<T>(count);
    for (int i = 0; i < count; i++)
        list.Add(item);
    return list;
}
```

**Bài 4** — Stack trên List:

```csharp
interface IStack<T>
{
    void Push(T item);
    T Pop();
    bool TryPeek(out T? item);
    int Count { get; }
}

class ArrayStack<T> : IStack<T>
{
    private readonly List<T> _items = new();
    public int Count => _items.Count;

    public void Push(T item) => _items.Add(item);

    public T Pop()
    {
        if (_items.Count == 0) throw new InvalidOperationException("empty");
        T item = _items[^1];
        _items.RemoveAt(_items.Count - 1);
        return item;
    }

    public bool TryPeek(out T? item)
    {
        if (_items.Count == 0)
        {
            item = default;
            return false;
        }
        item = _items[^1];
        return true;
    }
}
```

## 13. Đáp án thay thế

Bài 4 dùng mảng + `count` thủ công (giống `Stack<T>` nội bộ) thay vì `List<T>` — tốt cho học memory. Bài 2 có thể dùng `record Result<T>(bool Success, T? Value, string? Error)`.

## 14. Thử thách

Viết `Map<TIn, TOut>(IEnumerable<TIn> source, Func<TIn, TOut> selector)` trả `List<TOut>` — tự implement ý tưởng Select (không dùng LINQ).

## 15. Ứng dụng thực tế

- Mọi collection BCL
- Repository / Unit of Work generic trong backend
- Serializer `JsonSerializer.Deserialize<T>`
- Pipeline xử lý `IProcessor<TInput, TOutput>`

## 16. Liên hệ Unity

- `GetComponent<T>()`, `FindObjectOfType<T>()` — generic method
- Object pool `ObjectPool<T>` / tự viết `Pool<T> where T : Component`
- ScriptableObject database `Database<T>` load theo type
- Tránh `List<object>` chứa lẫn Component — mất type safety

## 17. Kiểm tra kiến thức

1. Generics giúp gì so với `object`?  
   **Đáp án:** An toàn kiểu lúc biên dịch; giảm cast/boxing không cần thiết.

2. `List<T>` — `T` gọi là gì?  
   **Đáp án:** Type parameter (tham số kiểu).

3. Generic method khai báo ở đâu?  
   **Đáp án:** Sau tên method: `void Foo<T>(...)`.

4. Có dùng nhiều type parameter không?  
   **Đáp án:** Có, ví dụ `Dictionary<TKey,TValue>`.

5. Compiler suy luận `T` khi gọi `Swap(ref a, ref b)` dựa vào đâu?  
   **Đáp án:** Kiểu của đối số `a`, `b`.
