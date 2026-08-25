# Chương 2 — Element operators

## 1. Mục tiêu học

- Kiểm tra tồn tại / toàn bộ: **Any**, **All**, **Contains**
- Lấy phần tử: **First**, **FirstOrDefault**, **Single**, **SingleOrDefault**, **Last**, **LastOrDefault**
- Tóm tắt số: **Count**, **Sum**, **Average**, **Min**, **Max**
- Biết toán tử nào **immediate** (chạy ngay) và khi nào throw

## 2. Điều kiện tiên quyết

- Chương 1: Where, Select, pipeline LINQ
- Hiểu `default` của reference type (`null`) và value type (`0`, `false`, …)

## 3. Khái niệm

**Element / aggregation operators** thường **không trả `IEnumerable`** mà trả **một giá trị** (bool, int, T, …). Chúng **enumerate** nguồn ngay → **immediate execution**.

| Nhóm | Toán tử | Ý nghĩa |
|------|---------|---------|
| Kiểm tra | `Any`, `All`, `Contains` | Có ít nhất / tất cả / chứa |
| Một phần tử | `First*`, `Single*`, `Last*` | Lấy 1 phần tử theo quy tắc |
| Đếm / thống kê | `Count`, `Sum`, `Average`, `Min`, `Max` | Số liệu |

**First vs Single:**

- `First`: lấy phần tử đầu thỏa; **0 phần tử → throw**; nhiều → vẫn lấy đầu.
- `Single`: **đúng một** phần tử thỏa; 0 hoặc >1 → throw.
- `*OrDefault`: không thỏa → `default(T)` thay vì throw.

## 4. Mô hình tư duy

```text
Dãy: [a, b, c, …]

Any(pred?)     → duyệt đến khi gặp true (short-circuit)
All(pred)      → duyệt đến khi gặp false
First          → lấy phần tử đầu (hoặc đầu thỏa pred)
Single         → phải có đúng 1 — “assert uniqueness”
Count()        → nếu là ICollection<T>: O(1); không thì duyệt hết
Sum/Average…   → duyệt hết (trừ khi empty + không hợp lệ)
```

## 5. Cú pháp

```csharp
bool Any<T>(this IEnumerable<T> source);
bool Any<T>(this IEnumerable<T> source, Func<T, bool> predicate);
bool All<T>(this IEnumerable<T> source, Func<T, bool> predicate);
bool Contains<T>(this IEnumerable<T> source, T value);

T First<T>(this IEnumerable<T> source);
T First<T>(this IEnumerable<T> source, Func<T, bool> predicate);
T? FirstOrDefault<T>(this IEnumerable<T> source); // với class: T?

T Single<T>(this IEnumerable<T> source, Func<T, bool>? predicate = null);
T? SingleOrDefault<T>(this IEnumerable<T> source);

int Count<T>(this IEnumerable<T> source);
int Count<T>(this IEnumerable<T> source, Func<T, bool> predicate);
long LongCount<T>(this IEnumerable<T> source);

int Sum(this IEnumerable<int> source);
double Average(this IEnumerable<int> source);
T Min<T>(this IEnumerable<T> source);
T Max<T>(this IEnumerable<T> source);
// Overload với selector: Sum(x => x.Price), …
```

.NET còn overload `FirstOrDefault` nhận `defaultValue` tùy chỉnh (C# / BCL gần đây) — hữu ích khi `0` không phải “thiếu”.

## 6. Ví dụ

### Cơ bản

```csharp
var nums = new List<int> { 1, 2, 3, 4, 5 };

Console.WriteLine(nums.Any());                 // true
Console.WriteLine(nums.Any(n => n > 10));      // false
Console.WriteLine(nums.All(n => n > 0));       // true
Console.WriteLine(nums.Contains(3));           // true

Console.WriteLine(nums.First());               // 1
Console.WriteLine(nums.First(n => n % 2 == 0)); // 2
Console.WriteLine(nums.Last());                // 5
Console.WriteLine(nums.Count());               // 5
Console.WriteLine(nums.Sum());                 // 15
Console.WriteLine(nums.Average());             // 3
Console.WriteLine(nums.Min());                 // 1
Console.WriteLine(nums.Max());                 // 5
```

### Trung cấp — OrDefault và Single

```csharp
record Hero(string Name, string Role);

var heroes = new List<Hero>
{
    new("Ada", "Mage"),
    new("Bob", "Tank"),
    new("Cara", "Mage"),
};

Hero? tank = heroes.FirstOrDefault(h => h.Role == "Healer");
Console.WriteLine(tank is null ? "không có healer" : tank.Name);

// Single: kỳ vọng đúng 1 tank
Hero theTank = heroes.Single(h => h.Role == "Tank");

// SingleOrDefault với Mage → throw vì có 2 Mage
// heroes.Single(h => h.Role == "Mage"); // InvalidOperationException
Hero? maybe = heroes.SingleOrDefault(h => h.Name == "Zed"); // null
```

### Nâng cao — selector + empty

```csharp
record Sale(string Region, decimal Amount);

var sales = new List<Sale>
{
    new("North", 100m),
    new("South", 80m),
    new("North", 50m),
};

decimal northTotal = sales.Where(s => s.Region == "North").Sum(s => s.Amount);
decimal maxSale = sales.Max(s => s.Amount);
Sale top = sales.OrderByDescending(s => s.Amount).First();

var empty = new List<int>();
// empty.Average(); // InvalidOperationException
bool has = empty.Any(); // false — an toàn hơn Average trên empty
int countEven = sales.Count(s => s.Amount >= 100m);
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| `InvalidOperationException`: sequence empty | `First`/`Single`/`Last`/`Average`/`Max` trên rỗng | `*OrDefault` hoặc `Any()` trước |
| `InvalidOperationException`: more than one | `Single` khi >1 | `First` hoặc siết điều kiện |
| Nhầm `Count()` với `Count` property | Gọi LINQ trên `List` vẫn OK nhưng | `list.Count` rõ và O(1) |
| `Contains` chậm trên List lớn | Duyệt tuyến tính | Dùng `HashSet<T>.Contains` |
| `FirstOrDefault` với `int` trả 0 | 0 vừa là giá trị vừa default | Dùng `FirstOrDefault(…, -1)` hoặc nullable `int?` pattern |

## 8. Gỡ lỗi

1. Exception từ `Single`: in `Where(pred).ToList()` xem có bao nhiêu phần tử.
2. `Average`/`Max` empty: breakpoint trước gọi; thêm guard `if (!source.Any())`.
3. So sánh `Count()` vs điều kiện: thường `Any(pred)` nhanh hơn `Count(pred) > 0` vì short-circuit.
4. Unit test các biên: 0 phần tử, 1, nhiều.

## 9. Best practices

- Tồn tại? → `Any(pred)` không phải `Count(pred) > 0`.
- “Phải có đúng một” (business rule) → `Single` để fail sớm.
- UI/lookup có thể thiếu → `FirstOrDefault` / `SingleOrDefault`.
- Prefer `list.Count` / `array.Length` khi đã biết kiểu collection cụ thể.
- `Contains` trên nhiều lần lookup → materialize `HashSet`.

## 10. Bài tập

```csharp
record Student(string Name, int Score);
var data = new List<Student>
{
    new("An", 9), new("Bình", 7), new("Chi", 9), new("Dũng", 5)
};
```

**Bài 1** — Có ai `Score >= 8` không? (`Any`)

**Bài 2** — Tất cả đều `Score >= 5`? (`All`)

**Bài 3** — Lấy học viên tên `"Chi"` bằng `Single`.

**Bài 4** — Điểm trung bình, min, max; đếm số người `Score == 9`.

## 11. Gợi ý

- Bài 1: `data.Any(s => s.Score >= 8)`.
- Bài 2: `data.All(s => s.Score >= 5)`.
- Bài 3: `data.Single(s => s.Name == "Chi")`.
- Bài 4: `Average(s => s.Score)`, `Min`/`Max`, `Count(s => s.Score == 9)`.

## 12. Đáp án

```csharp
bool hasHigh = data.Any(s => s.Score >= 8);
bool allPass = data.All(s => s.Score >= 5);
Student chi = data.Single(s => s.Name == "Chi");
double avg = data.Average(s => s.Score);
int min = data.Min(s => s.Score);
int max = data.Max(s => s.Score);
int nines = data.Count(s => s.Score == 9);
```

## 13. Đáp án thay thế

Bài 3 dùng `First` nếu không cần assert uniqueness. Điểm cao nhất kèm tên:

```csharp
Student top = data.OrderByDescending(s => s.Score).First();
// hoặc
int best = data.Max(s => s.Score);
var tops = data.Where(s => s.Score == best); // có thể nhiều người
```

## 14. Thử thách

Viết method `T SingleOrThrow<T>(IEnumerable<T> source, Func<T, bool> pred, string message)` bọc `Single` và thay message exception dễ đọc hơn (catch `InvalidOperationException` rồi throw `InvalidOperationException(message, ex)`).

## 15. Ứng dụng thực tế

- Validation: `All(items => items.IsValid)`
- Feature flag / permission: `Any(r => r == "Admin")`
- Repository: `SingleOrDefault(x => x.Id == id)`
- Dashboard: Sum/Average doanh thu

## 16. Liên hệ Unity

- `FindObjectsByType<T>().Any(x => x.CompareTag("Player"))`
- Chọn mục tiêu gần nhất: `OrderBy(d => d).FirstOrDefault()`
- Tránh `Count()` mỗi frame trên query nặng — cache
- `Single` khi chắc chỉ một `GameManager` trong scene (debug build)

## 17. Kiểm tra kiến thức

1. `Any()` trên list rỗng trả về gì?  
   **Đáp án:** `false`.

2. Khi nào dùng `Single` thay `First`?  
   **Đáp án:** Khi business yêu cầu **đúng một** phần tử; sai số lượng phải fail.

3. `FirstOrDefault` trên `IEnumerable<int>` rỗng trả về?  
   **Đáp án:** `0` (`default(int)`).

4. Vì sao `Any(pred)` thường tốt hơn `Count(pred) > 0`?  
   **Đáp án:** `Any` dừng sớm khi gặp phần tử thỏa.

5. `Average` trên dãy rỗng?  
   **Đáp án:** Ném `InvalidOperationException`.
