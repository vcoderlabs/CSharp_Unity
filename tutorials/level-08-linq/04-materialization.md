# Chương 4 — Materialization

## 1. Mục tiêu học

- Hiểu **materialization**: biến truy vấn trì hoãn thành cấu trúc dữ liệu cụ thể
- Dùng đúng **ToList**, **ToArray**, **ToDictionary**, **ToHashSet**, **ToLookup**
- Biết khi nào **nên** và **không nên** materialize sớm
- Xử lý trùng key khi `ToDictionary`

## 2. Điều kiện tiên quyết

- Chương 1–3: pipeline LINQ, Skip/Take
- Level 4: `List`, `Dictionary`, `HashSet`

## 3. Khái niệm

Hầu hết toán tử LINQ trả về `IEnumerable<T>` **chưa chạy** (deferred). **Materialization** buộc thực thi và **lưu kết quả** vào collection:

| Method | Kết quả | Ghi chú |
|--------|---------|---------|
| `ToList()` | `List<T>` | Linh hoạt, hay dùng nhất |
| `ToArray()` | `T[]` | Cố định kích thước |
| `ToDictionary(key)` | `Dictionary<TKey, T>` | Key phải unique |
| `ToDictionary(key, val)` | `Dictionary<TKey, TValue>` | Chiếu value |
| `ToHashSet()` | `HashSet<T>` | Unique + lookup O(1) |
| `ToLookup(key)` | `ILookup<TKey, T>` | Cho phép trùng key (giống GroupBy materialize) |

**Không materialize** nếu bạn chỉ `foreach` một lần và không cần tái sử dụng / index / Count nhiều lần.

## 4. Mô hình tư duy

```text
query = source.Where(...).Select(...)   ← công thức (chưa chạy)

list = query.ToList()   ← chạy 1 lần, giữ snapshot trong RAM
arr  = query.ToArray()  ← tương tự, mảng
dict = query.ToDictionary(x => x.Id)  ← chạy + index theo key

Sau ToList: sửa source gốc ≠ đổi list (đã copy phần tử / reference)
Với object: list giữ reference — mutate property vẫn thấy đổi
```

## 5. Cú pháp

```csharp
List<T> ToList<T>(this IEnumerable<T> source);
T[] ToArray<T>(this IEnumerable<T> source);

Dictionary<TKey, TSource> ToDictionary<TSource, TKey>(
    this IEnumerable<TSource> source,
    Func<TSource, TKey> keySelector) where TKey : notnull;

Dictionary<TKey, TElement> ToDictionary<TSource, TKey, TElement>(
    this IEnumerable<TSource> source,
    Func<TSource, TKey> keySelector,
    Func<TSource, TElement> elementSelector) where TKey : notnull;

HashSet<T> ToHashSet<T>(this IEnumerable<T> source);
ILookup<TKey, T> ToLookup<T, TKey>(this IEnumerable<T> source, Func<T, TKey> keySelector);
```

## 6. Ví dụ

### Cơ bản

```csharp
var q = Enumerable.Range(1, 5).Where(n => n % 2 == 1).Select(n => n * 10);

List<int> list = q.ToList();   // 10, 30, 50
int[] arr = q.ToArray();
```

### Trung cấp — ToDictionary

```csharp
record Product(int Id, string Name, decimal Price);

var products = new[]
{
    new Product(1, "Sword", 100m),
    new Product(2, "Shield", 80m),
};

Dictionary<int, Product> byId = products.ToDictionary(p => p.Id);
Dictionary<int, string> names = products.ToDictionary(p => p.Id, p => p.Name);

Console.WriteLine(byId[1].Name); // Sword
```

### Nâng cao — ToLookup vs GroupBy + snapshot

```csharp
record Log(string Level, string Message);

var logs = new List<Log>
{
    new("Info", "start"),
    new("Error", "boom"),
    new("Info", "ok"),
};

ILookup<string, Log> lookup = logs.ToLookup(l => l.Level);
foreach (var err in lookup["Error"])
    Console.WriteLine(err.Message);

HashSet<string> levels = logs.Select(l => l.Level).ToHashSet();

// Tránh ToDictionary khi trùng key:
// products.ToDictionary(p => p.Name); // throw nếu trùng Name
// → dùng GroupBy hoặc ToLookup, hoặc DistinctBy trước
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| `ArgumentException`: same key | `ToDictionary` trùng key | `GroupBy`/`ToLookup`/`DistinctBy` |
| Materialize quá sớm | Mất lợi ích composition / tốn RAM | Để deferred đến cuối pipeline |
| Materialize quá muộn | Enumerate nhiều lần (chương 5) | `ToList` một lần rồi dùng lại |
| Nghĩ `ToList` deep copy | Chỉ copy sequence; object vẫn shared | Clone thủ công nếu cần |
| `ToDictionary` key null | `TKey : notnull` | Không dùng null làm key |

## 8. Gỡ lỗi

1. Exception trùng key: `GroupBy(key).Where(g => g.Count() > 1)` để tìm key lỗi.
2. So sánh trước/sau: `query.Count()` vs `list.Count` — Count() có thể duyệt lại nguồn.
3. Profiler/log: đặt `Console.WriteLine` trong selector tạm để xem ToList chạy bao nhiêu lần map.
4. Kiểm tra bạn có gọi `ToList()` bên trong vòng lặp không — anti-pattern.

## 9. Best practices

- Cuối API public thường trả `IReadOnlyList<T>` từ `ToList()` — ổn định cho caller.
- Lookup theo id: `ToDictionary` một lần.
- Cho phép trùng key: `ToLookup` hoặc `Dictionary<TKey, List<T>>` tự xây.
- Prefer `ToHashSet` khi cần `Contains` nhiều lần sau filter.
- Đừng `ToList()` giữa mỗi Where/Select — chỉ cuối chuỗi (trừ khi cần break deferred cố ý).

## 10. Bài tập

```csharp
record Emp(int Id, string Dept, string Name);
var emps = new List<Emp>
{
    new(1, "IT", "An"),
    new(2, "HR", "Bình"),
    new(3, "IT", "Chi"),
};
```

**Bài 1** — Materialize tên nhân viên IT thành `List<string>`.

**Bài 2** — `ToDictionary` theo `Id`.

**Bài 3** — `ToLookup` theo `Dept`; in số người IT.

**Bài 4** — Tạo `HashSet<string>` các phòng ban.

## 11. Gợi ý

- Bài 1: `Where` → `Select` → `ToList`.
- Bài 2: `ToDictionary(e => e.Id)`.
- Bài 3: `ToLookup(e => e.Dept)` rồi `lookup["IT"].Count()`.
- Bài 4: `Select(e => e.Dept).ToHashSet()`.

## 12. Đáp án

```csharp
List<string> itNames = emps
    .Where(e => e.Dept == "IT")
    .Select(e => e.Name)
    .ToList();

Dictionary<int, Emp> byId = emps.ToDictionary(e => e.Id);

ILookup<string, Emp> byDept = emps.ToLookup(e => e.Dept);
int itCount = byDept["IT"].Count();

HashSet<string> depts = emps.Select(e => e.Dept).ToHashSet();
```

## 13. Đáp án thay thế

Dictionary nhóm thủ công:

```csharp
var map = emps
    .GroupBy(e => e.Dept)
    .ToDictionary(g => g.Key, g => g.ToList());
```

`ToArray` thay `ToList` khi API yêu cầu mảng cố định.

## 14. Thử thách

Viết extension `ToDictionarySafe<T, TKey>` bỏ qua hoặc ghi đè khi trùng key (chọn policy `skip` / `overwrite`), không throw. So sánh với `DistinctBy` + `ToDictionary`.

## 15. Ứng dụng thực tế

- Cache kết quả query trong memory
- Bind UI grid từ `List`
- Index entity theo Id
- Export cố định snapshot tại thời điểm T

## 16. Liên hệ Unity

- Cache `FindObjectsByType` → `ToList()` một lần khi load scene
- Dictionary theo instance id / net id
- Tránh `ToList` trong `Update`
- ScriptableObject database: materialize lúc init

## 17. Kiểm tra kiến thức

1. `ToList()` làm gì với deferred query?  
   **Đáp án:** Thực thi ngay và lưu kết quả vào `List<T>`.

2. `ToDictionary` khi trùng key?  
   **Đáp án:** Ném `ArgumentException`.

3. `ToLookup` khác `ToDictionary` chỗ nào?  
   **Đáp án:** Lookup cho phép nhiều phần tử cùng key.

4. Khi nào nên materialize?  
   **Đáp án:** Cần tái sử dụng kết quả, index, tránh multiple enumeration, hoặc trả về khỏi method.

5. `ToHashSet` hữu ích khi nào?  
   **Đáp án:** Cần tập unique và kiểm tra membership nhanh.
