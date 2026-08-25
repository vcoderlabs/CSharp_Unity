# Chương 1 — Core operators

## 1. Mục tiêu học

- Viết truy vấn LINQ với **Where**, **Select**, **SelectMany**
- Sắp xếp bằng **OrderBy** / **ThenBy** (và bản Descending)
- Nhóm bằng **GroupBy**, nối bằng **Join** / **GroupJoin**
- Tóm tắt chuỗi bằng **Aggregate**
- Phân biệt **method syntax** và **query syntax** (.NET 8+)

## 2. Điều kiện tiên quyết

- Level 4: `List<T>`, `IEnumerable<T>`, foreach
- Level 5: generic method / `Func<T, TResult>`
- Level 7: lambda, `Func`, `Predicate`

## 3. Khái niệm

**LINQ** (*Language Integrated Query*) là bộ API extension methods trên `IEnumerable<T>` (và `IQueryable<T>` ở chương 5) để **lọc, chiếu, sắp xếp, nhóm, nối** dữ liệu theo phong cách khai báo.

Hai cách viết cùng ý nghĩa:

```csharp
// Method syntax (phổ biến trong code thực tế)
var q1 = numbers.Where(n => n > 0).Select(n => n * 2);

// Query syntax (gần SQL)
var q2 =
    from n in numbers
    where n > 0
    select n * 2;
```

| Toán tử | Vai trò ngắn |
|---------|----------------|
| `Where` | Lọc phần tử thỏa điều kiện |
| `Select` | Chiếu / map 1 → 1 |
| `SelectMany` | Làm phẳng 1 → nhiều |
| `OrderBy` / `ThenBy` | Sắp xếp chính / phụ |
| `GroupBy` | Nhóm theo khóa |
| `Join` | Inner join theo key |
| `GroupJoin` | Left outer group join |
| `Aggregate` | Gấp (fold) thành một giá trị |

## 4. Mô hình tư duy

```text
Nguồn (IEnumerable<T>)
    │  Where     → còn lại các phần tử “đúng điều kiện”
    │  Select    → mỗi phần tử thành hình dạng mới (cùng số lượng)
    │  SelectMany→ mỗi phần tử “bung” thành nhiều → một dãy phẳng
    │  OrderBy   → thứ tự
    │  GroupBy   → các “túi” IGrouping<TKey, TElement>
    │  Join      → ghép hai dãy theo khóa khớp
    │  Aggregate → một kết quả cuối (sum tự viết, concat string, …)
    ▼
Vẫn là IEnumerable<...> cho đến khi materialize / enumerate
```

## 5. Cú pháp

```csharp
using System.Linq;

IEnumerable<T> Where<T>(this IEnumerable<T> source, Func<T, bool> predicate);
IEnumerable<TResult> Select<T, TResult>(this IEnumerable<T> source, Func<T, TResult> selector);
IEnumerable<TResult> SelectMany<T, TResult>(
    this IEnumerable<T> source, Func<T, IEnumerable<TResult>> selector);

IOrderedEnumerable<T> OrderBy<T, TKey>(this IEnumerable<T> source, Func<T, TKey> keySelector);
IOrderedEnumerable<T> ThenBy<T, TKey>(this IOrderedEnumerable<T> source, Func<T, TKey> keySelector);

IEnumerable<IGrouping<TKey, T>> GroupBy<T, TKey>(
    this IEnumerable<T> source, Func<T, TKey> keySelector);

IEnumerable<TResult> Join<TOuter, TInner, TKey, TResult>(
    this IEnumerable<TOuter> outer,
    IEnumerable<TInner> inner,
    Func<TOuter, TKey> outerKey,
    Func<TInner, TKey> innerKey,
    Func<TOuter, TInner, TResult> resultSelector);

TAccumulate Aggregate<T, TAccumulate>(
    this IEnumerable<T> source,
    TAccumulate seed,
    Func<TAccumulate, T, TAccumulate> func);
```

Query syntax tương ứng: `where`, `select`, `from` lồng nhau (= SelectMany), `orderby`, `group … by`, `join … on … equals`.

## 6. Ví dụ

### Cơ bản

```csharp
var scores = new[] { 90, 55, 72, 40, 88 };

var passed = scores
    .Where(s => s >= 50)
    .OrderByDescending(s => s)
    .Select(s => $"Đạt: {s}");

foreach (var line in passed)
    Console.WriteLine(line);
```

### Trung cấp — SelectMany + GroupBy

```csharp
record Player(string Name, string Class, int Level);
record Party(string Name, List<Player> Members);

var parties = new List<Party>
{
    new("Alpha", new() { new("Ada", "Mage", 10), new("Bob", "Tank", 8) }),
    new("Beta",  new() { new("Cara", "Mage", 12), new("Dan", "Healer", 9) }),
};

// Tất cả player từ mọi party
var allPlayers = parties.SelectMany(p => p.Members);

// Nhóm theo class
var byClass = allPlayers.GroupBy(p => p.Class);
foreach (var g in byClass)
{
    Console.WriteLine($"{g.Key}: {string.Join(", ", g.Select(x => x.Name))}");
}
```

### Nâng cao — Join + Aggregate

```csharp
record Order(int Id, int CustomerId, decimal Total);
record Customer(int Id, string Name);

var customers = new[]
{
    new Customer(1, "An"),
    new Customer(2, "Bình"),
};

var orders = new[]
{
    new Order(100, 1, 50m),
    new Order(101, 1, 30m),
    new Order(102, 2, 80m),
};

var rows =
    from c in customers
    join o in orders on c.Id equals o.CustomerId
    select new { c.Name, o.Total };

foreach (var r in rows)
    Console.WriteLine($"{r.Name}: {r.Total}");

// GroupJoin: mỗi customer + danh sách order (kể cả không có order)
var left =
    customers.GroupJoin(
        orders,
        c => c.Id,
        o => o.CustomerId,
        (c, ords) => new { c.Name, Sum = ords.Sum(o => o.Total) });

// Aggregate: nối tên
string names = customers.Aggregate("", (acc, c) =>
    acc.Length == 0 ? c.Name : acc + ", " + c.Name);
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| `NullReferenceException` trong lambda | Phần tử null | `Where(x => x is not null)` rồi `!` hoặc filter trước |
| `SelectMany` trả “list of lists” | Dùng `Select` thay vì `SelectMany` | Đổi sang `SelectMany` |
| `OrderBy` rồi `OrderBy` lại | OrderBy thứ hai **thay** thứ tự, không phải phụ | Dùng `ThenBy` sau `OrderBy` |
| Join không ra dòng | Key không khớp / khác kiểu | Kiểm tra key, `StringComparer` nếu string |
| Aggregate trên dãy rỗng không seed | Throw | Dùng overload có `seed` hoặc kiểm tra `Any()` |

## 8. Gỡ lỗi

1. Tách pipeline: gán từng bước `var step1 = …; var step2 = step1.…` rồi `ToList()` tạm để xem.
2. Trong debugger, tránh expand `IEnumerable` nhiều lần nếu nguồn có side-effect (xem chương 5).
3. Với Join: in ra các key hai bên trước khi join.
4. `GroupBy`: nhớ mỗi nhóm là `IGrouping<TKey, T>` — duyệt `foreach (var g in groups)`.

## 9. Best practices

- Prefer **method syntax** trong codebase lớn; query syntax hữu ích khi join/group dài.
- Giữ lambda ngắn; logic phức tạp → method riêng `bool IsEligible(Player p)`.
- `Select` chỉ đổi hình dạng; lọc ở `Where` — đừng nhồi điều kiện vào Select.
- `OrderBy` ổn định tương đối trong LINQ to Objects; vẫn nên `ThenBy` khi cần tie-break rõ.
- Tránh Aggregate cho Sum/Count có sẵn — dùng `Sum`, `Count` (chương 2) rõ nghĩa hơn.

## 10. Bài tập

Cho model:

```csharp
record Product(string Name, string Category, decimal Price);
```

**Bài 1** — Lọc sản phẩm `Price >= 100`, chọn chỉ `Name`.

**Bài 2** — Sắp xếp theo `Category` tăng, rồi `Price` giảm.

**Bài 3** — `GroupBy` Category; in số lượng mỗi nhóm.

**Bài 4** — Có `orders: List<List<int>>` (mỗi đơn là list productId) — dùng `SelectMany` lấy mọi id.

## 11. Gợi ý

- Bài 1: `Where` → `Select(p => p.Name)`.
- Bài 2: `OrderBy(p => p.Category).ThenByDescending(p => p.Price)`.
- Bài 3: `GroupBy(p => p.Category)` rồi `g.Count()`.
- Bài 4: `orders.SelectMany(o => o)`.

## 12. Đáp án

**Bài 1:**

```csharp
var names = products
    .Where(p => p.Price >= 100m)
    .Select(p => p.Name);
```

**Bài 2:**

```csharp
var sorted = products
    .OrderBy(p => p.Category)
    .ThenByDescending(p => p.Price);
```

**Bài 3:**

```csharp
foreach (var g in products.GroupBy(p => p.Category))
    Console.WriteLine($"{g.Key}: {g.Count()}");
```

**Bài 4:**

```csharp
IEnumerable<int> allIds = orders.SelectMany(order => order);
```

## 13. Đáp án thay thế

Bài 1–3 bằng query syntax:

```csharp
var names =
    from p in products
    where p.Price >= 100m
    select p.Name;

var sorted =
    from p in products
    orderby p.Category, p.Price descending
    select p;
```

Bài 3 có thể `Select(g => new { g.Key, Count = g.Count() })` rồi materialize.

## 14. Thử thách

Viết `GroupJoin` giữa `Category(string Name)` và `Product` theo tên category; với category không có product vẫn hiện `Count = 0`. Thêm `Aggregate` tính tổng giá mọi product (seed `0m`).

## 15. Ứng dụng thực tế

- Báo cáo doanh số: filter → group theo ngày/region → aggregate
- API: map DTO bằng `Select`
- Flatten nested JSON/DTO bằng `SelectMany`
- Join in-memory khi chưa dùng SQL/EF

## 16. Liên hệ Unity

- Lọc `FindObjectsByType<Enemy>()` rồi `Where(e => e.IsAlive)`
- Inventory: `items.GroupBy(i => i.Type)`
- Quest rewards: `SelectMany(q => q.Rewards)`
- Cẩn trọng LINQ mỗi `Update()` — allocate / enumerate tốn GC; cache kết quả khi có thể

## 17. Kiểm tra kiến thức

1. `Select` và `SelectMany` khác nhau chỗ nào?  
   **Đáp án:** Select: 1→1; SelectMany: 1→nhiều rồi làm phẳng thành một dãy.

2. Sau `OrderBy`, muốn sắp phụ thì dùng gì?  
   **Đáp án:** `ThenBy` / `ThenByDescending` (không gọi `OrderBy` lần nữa nếu muốn giữ khóa chính).

3. `GroupBy` trả về kiểu gì (ý niệm)?  
   **Đáp án:** Các nhóm `IGrouping<TKey, TElement>`.

4. `Join` trong LINQ to Objects gần với join SQL nào?  
   **Đáp án:** Inner join.

5. Method syntax và query syntax có khác semantics không?  
   **Đáp án:** Không — cùng mô hình; compiler chuyển query syntax thành gọi method.
