# Chương 5 — Deferred execution

## 1. Mục tiêu học

- Phân biệt **deferred** vs **immediate** execution
- Hiểu `IEnumerable<T>` (LINQ to Objects) vs `IQueryable<T>` (provider: EF Core, …)
- Nhận diện và tránh **multiple enumeration**
- Biết toán tử nào kích hoạt thực thi ngay

## 2. Điều kiện tiên quyết

- Chương 1–4: operators + ToList
- Level 4: `IEnumerable`, foreach, iterator ý niệm
- (Tuỳ chọn) nghe qua ORM / EF — không bắt buộc có database

## 3. Khái niệm

### Deferred (trì hoãn)

`Where`, `Select`, `SelectMany`, `OrderBy`, `GroupBy`, `Join`, `Distinct`, `Skip`, `Take`, `Chunk`… trả về object query. **Chưa duyệt nguồn** cho đến khi:

- `foreach`
- `ToList` / `ToArray` / `ToDictionary` / …
- `Count`, `Any`, `First`, `Sum`, … (element/aggregate)

### Immediate (ngay)

Các toán tử trả giá trị đơn hoặc collection cụ thể — chạy ngay khi gọi.

### IEnumerable vs IQueryable

| | `IEnumerable<T>` | `IQueryable<T>` |
|--|------------------|-----------------|
| Biểu diễn | Delegates (`Func`) trên object | **Expression trees** (cây biểu thức) |
| Chạy đâu | Trong process .NET | Provider dịch (SQL, …) |
| LINQ | LINQ to Objects | LINQ to Entities / khác |
| Rủi ro | Multiple enum = chạy lại RAM | Multiple enum = **nhiều round-trip DB** |

Bạn chưa cần EF để hiểu: **cùng cú pháp LINQ**, mô hình thực thi khác nhau.

## 4. Mô hình tư duy

```text
var q = list.Where(x => Expensive(x)).Select(...);
        ↑ chỉ “ghi công thức”

foreach (var x in q) { }   // lần 1: chạy Where/Select
foreach (var x in q) { }   // lần 2: chạy LẠI từ đầu

var snap = q.ToList();     // chạy 1 lần
foreach (var x in snap)    // chỉ duyệt list — không chạy lại Where
```

## 5. Cú pháp

Không có API mới bắt buộc — tập trung hành vi:

```csharp
IEnumerable<int> q = numbers.Where(n =>
{
    Console.WriteLine($"filter {n}");
    return n % 2 == 0;
});

Console.WriteLine("chưa chạy filter");
foreach (var n in q) { }          // in filter…
var list = q.ToList();            // filter chạy lại!

// IQueryable (minh họa chữ ký — cần provider thật để chạy SQL)
// IQueryable<Player> players = db.Players.Where(p => p.Level > 10);
```

## 6. Ví dụ

### Cơ bản — chứng minh deferred

```csharp
var data = new List<int> { 1, 2, 3 };

IEnumerable<int> q = data.Select(n =>
{
    Console.WriteLine($"map {n}");
    return n * 2;
});

Console.WriteLine("after define");
foreach (var x in q)
    Console.WriteLine($"got {x}");
// after define → map 1, got 2, map 2, got 4, …
```

### Trung cấp — multiple enumeration bug

```csharp
IEnumerable<int> Evens(IEnumerable<int> src) => src.Where(n => n % 2 == 0);

void Report(IEnumerable<int> src)
{
    var q = Evens(src);
    Console.WriteLine($"Count = {q.Count()}"); // duyệt 1
    Console.WriteLine($"Sum   = {q.Sum()}");   // duyệt 2
    foreach (var n in q)                       // duyệt 3
        Console.WriteLine(n);
}

// Sửa: materialize một lần
void ReportFixed(IEnumerable<int> src)
{
    var evens = Evens(src).ToList();
    Console.WriteLine($"Count = {evens.Count}");
    Console.WriteLine($"Sum   = {evens.Sum()}");
    foreach (var n in evens)
        Console.WriteLine(n);
}
```

### Nâng cao — nguồn đổi giữa các lần duyệt + IQueryable ý niệm

```csharp
var live = new List<int> { 1, 2, 3 };
var q = live.Where(n => n > 1);

live.Add(4);
Console.WriteLine(string.Join(',', q)); // 2,3,4 — thấy phần tử mới!

live.Clear();
Console.WriteLine(q.Any()); // false — query gắn nguồn hiện tại

// IQueryable: mỗi lần enumerate có thể tạo SQL mới
// var query = db.Orders.Where(o => o.Total > 100);
// query.Count();  // SELECT COUNT...
// query.ToList(); // SELECT * ...  ← 2 round-trips nếu không cache
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Chậm gấp đôi/ba | Count + foreach cùng query | `ToList` một lần |
| Kết quả khác nhau giữa 2 lần duyệt | Nguồn mutable / I/O | Snapshot `ToList` |
| N+1 query (EF) | Enumerate + query lồng | Include / projection / materialize |
| `Where` sau `ToList` trên IQueryable | Kéo hết về RAM rồi lọc | Lọc trên `IQueryable` trước |
| Side-effect trong Select chạy “lúc lạ” | Deferred | Đừng side-effect trong projection |

## 8. Gỡ lỗi

1. Thêm log trong lambda để đếm số lần chạy.
2. ReSharper/Rider/analyzer: cảnh báo *possible multiple enumeration*.
3. Với EF: bật sensitive logging SQL — thấy bao nhiêu câu lệnh.
4. Tách: `var list = query.ToList();` rồi chỉ dùng `list`.

## 9. Best practices

- Method trả `IEnumerable` deferred: document “có thể enumerate nhiều lần = chạy lại”.
- Public API thường materialize (`IReadOnlyList`) trừ khi streaming cố ý.
- Một query → tối đa **một** lần thực thi nặng; tái sử dụng collection.
- `IQueryable`: giữ filter/projection trên server; `AsEnumerable()` chỉ khi cần logic .NET không dịch được.
- Tránh side-effect (I/O, random, mutate) trong `Select`/`Where`.

## 10. Bài tập

**Bài 1** — Viết demo in ra thứ tự: định nghĩa query → “before” → foreach (có log trong Where).

**Bài 2** — Hàm nhận `IEnumerable<int>`, in `Any()` và `Count()` — chỉ ra bị multiple enumeration; sửa bằng `ToList`.

**Bài 3** — List thay đổi sau khi tạo query; giải thích output.

**Bài 4** — Liệt kê 5 toán tử immediate và 5 toán tử deferred (tự ghi bảng).

## 11. Gợi ý

- Bài 1: bám ví dụ mục 6 cơ bản.
- Bài 2: `var list = source.ToList();` rồi Any/Count trên list (Count property).
- Bài 3: deferred gắn nguồn sống.
- Bài 4: Immediate ≈ element/aggregate/To*; Deferred ≈ Where/Select/OrderBy/…

## 12. Đáp án

**Bài 1–2:** xem ví dụ mục 6.

**Bài 3:**

```csharp
var list = new List<string> { "a" };
var q = list.Where(x => true);
list.Add("b");
// foreach q → a, b
```

**Bài 4 (mẫu):**

- Deferred: Where, Select, SelectMany, OrderBy, GroupBy, Join, Distinct, Skip, Take  
- Immediate: ToList, Count, Any, First, Sum, Average, Min, Max, ToDictionary  

## 13. Đáp án thay thế

Dùng `Lazy<T>` hoặc field cache:

```csharp
sealed class CachedQuery<T>
{
    private readonly Func<IEnumerable<T>> _factory;
    private List<T>? _cache;
    public CachedQuery(Func<IEnumerable<T>> factory) => _factory = factory;
    public IReadOnlyList<T> Materialize() => _cache ??= _factory().ToList();
}
```

## 14. Thử thách

Viết analyzer-style helper `EnumerateOnce<T>(IEnumerable<T> source)` trả `IEnumerable<T>` ném exception nếu bị enumerate lần 2 (dùng flag trong iterator) — để test chỗ gọi vô tình duyệt kép.

## 15. Ứng dụng thực tế

- API layer: quyết định trả query hay list
- EF Core performance
- Streaming file lớn: giữ deferred + foreach một lần
- Báo cáo: snapshot tại thời điểm export

## 16. Liên hệ Unity

- Query component mỗi frame + Count + foreach = nhân chi phí
- Cache `ToList` khi scene load / interval
- Tránh LINQ deferred giữ reference collection bị Clear mỗi frame
- Addressables/async: materialize kết quả load một lần

## 17. Kiểm tra kiến thức

1. `Where` có chạy filter ngay khi gọi không?  
   **Đáp án:** Không — deferred đến khi enumerate/materialize.

2. Multiple enumeration nghĩa là gì?  
   **Đáp án:** Duyệt cùng một query nhiều lần → thực thi lại nhiều lần.

3. `IQueryable` khác `IEnumerable` ở điểm cốt lõi?  
   **Đáp án:** Expression tree + provider dịch (thường SQL); không phải chỉ Func in-memory.

4. Cách sửa điển hình khi cần Count và foreach?  
   **Đáp án:** `ToList()` một lần rồi dùng list.

5. Mutate list gốc sau khi tạo query deferred — kết quả foreach?  
   **Đáp án:** Thấy trạng thái **hiện tại** của nguồn (không phải snapshot trừ khi đã ToList).
