# Chương 5 — Interface Collection

## 1. Mục tiêu học

- Hiểu vai trò `IEnumerable<T>`, `ICollection<T>`, `IList<T>`, `IReadOnlyCollection<T>` / `IReadOnlyList<T>`
- Viết method nhận interface thay vì concrete type khi phù hợp
- Biết hierarchy và khả năng từng interface
- Tránh side-effect khi expose collection mutable

## 2. Điều kiện tiên quyết

- Level 2: interface, polymorphism
- Chương 1–4: các collection concrete

## 3. Khái niệm

Các collection .NET implement nhiều interface. Nhìn từ “ít khả năng → nhiều khả năng”:

```text
IEnumerable<T>          — chỉ duyệt (foreach)
    ↑
IReadOnlyCollection<T>  — duyệt + Count
    ↑
IReadOnlyList<T>        — + indexer get
```

Phía mutable:

```text
IEnumerable<T>
    ↑
ICollection<T>          — Count, Add, Remove, Clear, Contains
    ↑
IList<T>                — + indexer get/set, Insert, RemoveAt
```

| Interface | Bạn làm được |
|-----------|----------------|
| `IEnumerable<T>` | `foreach`, LINQ (sau) |
| `ICollection<T>` | Đếm, thêm/xóa (nếu không read-only) |
| `IList<T>` | Truy cập / sửa theo index |
| `IReadOnlyCollection<T>` | `Count` + duyệt, không Add qua interface này |
| `IReadOnlyList<T>` | + `this[int]` chỉ đọc |

> `IReadOnlyList<T>` **không** đảm bảo object bên dưới bất biến — caller vẫn có thể cast về `List<T>` và sửa. Muốn thật sự đóng băng: trả về bản copy hoặc dùng collection bất biến.

## 4. Mô hình tư duy

```text
API công khai nên "xin ít nhất đủ dùng":

Chỉ cần duyệt?     → IEnumerable<T>
Cần Count?         → IReadOnlyCollection<T>
Cần list[i]?       → IReadOnlyList<T>
Cần sửa collection?→ ICollection<T> / IList<T> (hiếm khi public)

Caller có thể truyền: Array, List, Queue (IEnumerable), ...
```

## 5. Cú pháp

```csharp
using System.Collections.Generic;

static int Sum(IEnumerable<int> source)
{
    int s = 0;
    foreach (int x in source) s += x;
    return s;
}

static void AddIfMissing(ICollection<string> setLike, string item)
{
    if (!setLike.Contains(item))
        setLike.Add(item);
}

static int FirstOrDefaultIndex(IList<string> list, string value)
{
    for (int i = 0; i < list.Count; i++)
        if (list[i] == value) return i;
    return -1;
}

static IReadOnlyList<int> Snapshot(List<int> live)
    => live.ToArray(); // copy — caller không sửa được list gốc qua return
```

## 6. Ví dụ

### Cơ bản

Method nhận mọi nguồn duyệt được:

```csharp
static void PrintAll(IEnumerable<string> items)
{
    foreach (var item in items)
        Console.WriteLine(item);
}

// Gọi được với:
PrintAll(new[] { "a", "b" });
PrintAll(new List<string> { "c" });
PrintAll(new HashSet<string> { "d" });
```

### Trung cấp

Trả về read-only view cho encapsulation:

```csharp
class Inventory
{
    private readonly List<string> _items = new();

    public IReadOnlyList<string> Items => _items;

    public void Add(string item) => _items.Add(item);
}

// Bên ngoài: có thể đọc Items[i], Count; không Add qua property
// (nhưng vẫn cast được — xem mục 3)
```

### Nâng cao

Phân nhánh tối ưu khi biết concrete type (micro-optimization — biết là đủ):

```csharp
static bool ContainsFast(IEnumerable<int> source, int value)
{
    if (source is HashSet<int> set)
        return set.Contains(value);
    if (source is ICollection<int> col)
        return col.Contains(value);
    foreach (int x in source)
        if (x == value) return true;
    return false;
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Không gọi được `Add` trên `IEnumerable` | Interface chỉ duyệt | Đổi sang `ICollection` hoặc nhận `List` |
| `NotSupportedException` trên Add | Mảng cast thành `IList` — fixed size | Không Add vào array qua IList |
| Tưởng `IReadOnlyList` bất biến tuyệt đối | Chỉ là contract bề mặt | Copy nếu cần đảm bảo |
| Method chỉ nhận `List<T>` | Caller có array bị buộc `ToList()` | Nới thành `IEnumerable`/`IReadOnlyList` |

## 8. Gỡ lỗi

1. Xem signature method — đang yêu cầu concrete type quá chặt?
2. Exception `NotSupportedException`: in `collection.GetType()` — có thể là array hoặc `ReadOnlyCollection`.
3. Side-effect lạ: ai đó cast `IReadOnlyList` về `List` — tìm chỗ cast.

## 9. Best practices

- **Parameter:** xin interface hẹp nhất đủ dùng (`IEnumerable` nếu chỉ foreach).
- **Return:** cân nhắc `IReadOnlyList<T>` / `IReadOnlyCollection<T>` thay vì `List<T>` public.
- Đừng return field `List` mutable nếu không muốn caller sửa nội bộ — copy hoặc wrapper.
- Tránh `IList` trên API public trừ khi API thật sự cho phép Insert/indexer set.
- Unity: nhiều API Unity trả array; viết helper nhận `IEnumerable` để dễ test.

## 10. Bài tập

**Bài 1 — CountGreaterThan**  
Nhận `IEnumerable<int>` và `threshold`, trả về số phần tử &gt; threshold.

**Bài 2 — Fill**  
Nhận `IList<int>` và `value`, gán mọi phần tử = value.

**Bài 3 — ToReadOnlyCopy**  
Nhận `IEnumerable<string>`, trả `IReadOnlyList<string>` là **bản sao** độc lập.

**Bài 4 — UniquePreserveOrder**  
Nhận `IEnumerable<T>`, trả `List<T>` unique giữ thứ tự xuất hiện đầu (dùng HashSet phụ). Generic method OK nếu đã biết sơ generics; không thì làm cho `string`.

## 11. Gợi ý

- Bài 1: foreach + đếm.
- Bài 2: `for (i=0; i<list.Count; i++) list[i]=value`.
- Bài 3: `source.ToList()` hoặc copy thủ công rồi trả về — kiểu trả về `IReadOnlyList`.
- Bài 4: HashSet `seen`; chỉ Add vào result khi `seen.Add(x)` true.

## 12. Đáp án

**Bài 1** — Chỉ cần duyệt nên dùng IEnumerable:

```csharp
static int CountGreaterThan(IEnumerable<int> source, int threshold)
{
    int count = 0;
    foreach (int x in source)
        if (x > threshold) count++;
    return count;
}
```

**Bài 2** — Cần indexer set → IList:

```csharp
static void Fill(IList<int> list, int value)
{
    for (int i = 0; i < list.Count; i++)
        list[i] = value;
}
```

**Bài 3** — Materialize bản sao:

```csharp
static IReadOnlyList<string> ToReadOnlyCopy(IEnumerable<string> source)
{
    return new List<string>(source); // List implement IReadOnlyList
}
```

**Bài 4** — Unique giữ thứ tự:

```csharp
static List<string> UniquePreserveOrder(IEnumerable<string> source)
{
    var seen = new HashSet<string>();
    var result = new List<string>();
    foreach (string s in source)
    {
        if (seen.Add(s))
            result.Add(s);
    }
    return result;
}
```

## 13. Đáp án thay thế

Bài 3 trả về mảng: `return source.ToArray();` — `T[]` cũng là `IReadOnlyList<T>`.

## 14. Thử thách

Viết `ReadOnlyView<T>` wrapper tự tạo: nhận `IList<T>`, implement `IReadOnlyList<T>`, **không** cho cast ngược dễ dàng (không expose list gốc). So sánh với `list.AsReadOnly()`.

## 15. Ứng dụng thực tế

- Thư viện dùng chung: API ổn định dù bên trong đổi List → array
- Domain model: aggregate expose `IReadOnlyCollection` children
- Testability: truyền fake enumerable dễ hơn mock List cụ thể

## 16. Liên hệ Unity

- Nhiều API Unity dùng mảng (`RaycastHit[]`, `Collider[]`)
- Viết utility `static void ApplyDamage(IEnumerable<IDamageable> targets)` — nhận list hoặc array
- ScriptableObject database: expose `IReadOnlyList<ItemDef>` thay vì List public trên Inspector logic

## 17. Kiểm tra kiến thức

1. Interface nào chỉ đảm bảo duyệt được?  
   **Đáp án:** `IEnumerable<T>`.

2. `IList<T>` thêm gì so với `ICollection<T>`?  
   **Đáp án:** Indexer, Insert, RemoveAt (truy cập theo vị trí).

3. Vì sao parameter nên là `IEnumerable` thay vì `List`?  
   **Đáp án:** Caller linh hoạt hơn (array, set, yield…); giảm phụ thuộc.

4. `IReadOnlyList` có chặn sửa tuyệt đối không?  
   **Đáp án:** Không — có thể cast về kiểu mutable gốc.

5. `Count` có trên interface nào (read-only)?  
   **Đáp án:** `IReadOnlyCollection<T>` (và `ICollection<T>`).
