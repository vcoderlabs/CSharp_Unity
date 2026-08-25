# Chương 2 — Generic Constraints

## 1. Mục tiêu học

- Dùng mệnh đề `where` để giới hạn type parameter
- Nắm `class`, `struct`, `new()`, ràng buộc interface/base class, `notnull`, `unmanaged`
- Hiểu tại sao không constraint thì không gọi được member cụ thể trên `T`
- Kết hợp nhiều ràng buộc đúng thứ tự

## 2. Điều kiện tiên quyết

- Chương 1: generic class/method/interface
- Level 2: interface, abstract class, constructor

## 3. Khái niệm

**Constraint** = điều kiện `T` phải thỏa để dùng được API trên `T`:

```csharp
void PrintName<T>(T item) where T : INamed
{
    Console.WriteLine(item.Name); // OK vì INamed có Name
}
```

Không có constraint, `T` chỉ như “kiểu bí ẩn”: chỉ làm được thao tác hợp lệ với mọi kiểu (gán, so với null nếu reference…).

### Các constraint phổ biến

| Constraint | Ý nghĩa |
|------------|---------|
| `where T : class` | Reference type |
| `where T : struct` | Non-nullable value type |
| `where T : new()` | Có public parameterless ctor |
| `where T : BaseClass` | Kế thừa BaseClass |
| `where T : IInterface` | Implement interface |
| `where T : U` | T phải derive từ / tương thích U (type param khác) |
| `where T : notnull` | Không cho `T` nullable reference / `Nullable<T>` (ngữ cảnh NRT) |
| `where T : unmanaged` | Unmanaged value type (unsafe/interop) |

Thứ tự: `class`/`struct`/`notnull`/`unmanaged` trước → base → interfaces → `new()` cuối.

## 4. Mô hình tư duy

```text
Không where:   T  ≈  “hộp đen”
where T : IComparable<T>:  mở được CompareTo
where T : new():           gọi được new T()
where T : class:           gán null được (tùy NRT); không phải struct
where T : struct:          không null; hay dùng với tránh boxing pattern
```

## 5. Cú pháp

```csharp
class Factory<T> where T : class, new()
{
    public T Create() => new T();
}

class Repo<TEntity, TKey>
    where TEntity : class, IEntity<TKey>
    where TKey : notnull
{
    private readonly Dictionary<TKey, TEntity> _map = new();
    public void Add(TEntity e) => _map[e.Id] = e;
}

static TResult Pipe<T, TResult>(T input, Func<T, TResult> f) => f(input);
// không cần constraint nếu chỉ truyền Func
```

## 6. Ví dụ

### Cơ bản

```csharp
interface IHasId { int Id { get; } }

static int GetId<T>(T item) where T : IHasId => item.Id;

static T Create<T>() where T : new() => new T();
```

### Trung cấp

So sánh max với `IComparable<T>`:

```csharp
static T Max<T>(IEnumerable<T> items) where T : IComparable<T>
{
    using var e = items.GetEnumerator();
    if (!e.MoveNext()) throw new InvalidOperationException("empty");
    T best = e.Current;
    while (e.MoveNext())
        if (e.Current.CompareTo(best) > 0)
            best = e.Current;
    return best;
}
```

### Nâng cao

`class` + interface + `new()`:

```csharp
abstract class Entity
{
    public Guid Id { get; protected set; } = Guid.NewGuid();
}

interface IAuditable
{
    DateTime UpdatedAt { get; set; }
}

class EntityFactory<T> where T : Entity, IAuditable, new()
{
    public T CreateFresh()
    {
        var e = new T();
        e.UpdatedAt = DateTime.UtcNow;
        return e;
    }
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| CS0304 Cannot create instance of T | Thiếu `new()` | Thêm constraint hoặc truyền `Func<T>` |
| CS0453 struct vs class conflict | Vừa `class` vừa `struct` | Chỉ chọn một |
| `new()` với abstract / không có ctor trống | Constraint không thỏa | Bỏ `new()`, dùng factory delegate |
| Gọi `T.Name` không constraint | T không biết member | Interface constraint hoặc selector `Func<T,string>` |
| `where T : struct, class` | Mâu thuẫn | Không hợp lệ |

## 8. Gỡ lỗi

1. Đọc thông báo “type argument must be…” — thiếu đúng constraint nào.
2. Thử instantiate với kiểu cụ thể: `Create<MyClass>()` — MyClass có public ctor không tham số?
3. Nếu `new()` quá chật: đổi API thành `Create(Func<T> factory)`.

## 9. Best practices

- Constraint **vừa đủ** — đừng `class` nếu không cần.
- Prefer `Func<T>` / abstract factory hơn `new()` khi DI / test.
- `IComparable<T>` / `IEquatable<T>` thay vì so sánh ad-hoc.
- Nhiều type param: mỗi cái một dòng `where` cho dễ đọc.
- Document giả định: “TEntity phải có Id không trùng”.

## 10. Bài tập

**Bài 1** — `static T[] CreateArray<T>(int size) where T : new()` — mỗi phần tử `new T()`.

**Bài 2** — `static bool IsNullOrDefault<T>(T value) where T : class` — true nếu null.

**Bài 3** — Interface `IEntity<TKey>`; class `Repository<TEntity,TKey>` với `where TEntity : IEntity<TKey>` và Dictionary nội bộ; method `Add` / `Get`.

**Bài 4** — `static void EnsureNotNull<T>(T value, string name) where T : class` — ném `ArgumentNullException` nếu null.

## 11. Gợi ý

- Bài 1: `var a = new T[size]; for ... a[i]=new T();`
- Bài 2: `return value is null;`
- Bài 3: `TKey` từ `entity.Id`; `Get` dùng `TryGetValue`.
- Bài 4: `ArgumentNullException.ThrowIfNull(value, name);` (.NET) hoặc if throw.

## 12. Đáp án

**Bài 1** — Cần `new()` để khởi tạo từng phần tử:

```csharp
static T[] CreateArray<T>(int size) where T : new()
{
    var arr = new T[size];
    for (int i = 0; i < size; i++)
        arr[i] = new T();
    return arr;
}
```

**Bài 2** — Chỉ reference type mới null theo nghĩa class:

```csharp
static bool IsNullOrDefault<T>(T value) where T : class
    => value is null;
```

**Bài 3** — Repository với khóa từ entity:

```csharp
interface IEntity<TKey>
{
    TKey Id { get; }
}

class Repository<TEntity, TKey> where TEntity : IEntity<TKey> where TKey : notnull
{
    private readonly Dictionary<TKey, TEntity> _data = new();

    public void Add(TEntity entity) => _data[entity.Id] = entity;

    public bool TryGet(TKey id, out TEntity? entity) => _data.TryGetValue(id, out entity);
}
```

**Bài 4** — Guard null:

```csharp
static void EnsureNotNull<T>(T value, string name) where T : class
{
    ArgumentNullException.ThrowIfNull(value, name);
}
```

## 13. Đáp án thay thế

Bài 1 không `new()`: nhận `Func<T> factory` và gọi `factory()` — linh hoạt hơn cho class không có ctor trống.

```csharp
static T[] CreateArray<T>(int size, Func<T> factory)
{
    var arr = new T[size];
    for (int i = 0; i < size; i++) arr[i] = factory();
    return arr;
}
```

## 14. Thử thách

Viết `MinBy<TSource, TKey>(IEnumerable<TSource> source, Func<TSource, TKey> keySelector) where TKey : IComparable<TKey>` trả phần tử có key nhỏ nhất.

## 15. Ứng dụng thực tế

- EF Core / repository pattern: `where T : class`
- Unity `where T : Component` trên extension Get/Add
- Math helpers `where T : INumber<T>` (.NET 7+)
- Serialization `where T : new()` (các lib cũ)

## 16. Liên hệ Unity

```csharp
public static T GetOrAdd<T>(this GameObject go) where T : Component
{
    var c = go.GetComponent<T>();
    return c != null ? c : go.AddComponent<T>();
}
```

- `where T : ScriptableObject` cho loader
- `where T : class` trên pool nếu T là MonoBehaviour reference
- Tránh constraint thừa khiến API khó gọi với struct (ví dụ damage number)

## 17. Kiểm tra kiến thức

1. `where T : new()` yêu cầu gì?  
   **Đáp án:** Public parameterless constructor.

2. Có ghi đồng thời `class` và `struct` không?  
   **Đáp án:** Không — mâu thuẫn.

3. Vì sao cần `where T : IComparable<T>` cho Max?  
   **Đáp án:** Để gọi `CompareTo` an toàn lúc biên dịch.

4. `new()` thường đặt vị trí nào trong danh sách constraint?  
   **Đáp án:** Cuối cùng.

5. Cách nào linh hoạt hơn `new()` khi tạo T?  
   **Đáp án:** Truyền `Func<T>` / factory từ DI.
