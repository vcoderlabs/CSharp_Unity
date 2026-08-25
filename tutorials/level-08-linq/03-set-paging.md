# Chương 3 — Distinct, Skip, Take, Chunk

## 1. Mục tiêu học

- Loại phần tử trùng bằng **Distinct** (và comparer tùy chỉnh)
- Phân trang / cửa sổ bằng **Skip** và **Take**
- Chia batch bằng **Chunk** (.NET 6+, dùng tốt trên .NET 8)
- Kết hợp với OrderBy để paging ổn định

## 2. Điều kiện tiên quyết

- Chương 1–2: Where, OrderBy, Count
- Hiểu `IEquatable<T>` / `IEqualityComparer<T>` cơ bản

## 3. Khái niệm

| Toán tử | Việc làm |
|---------|----------|
| `Distinct` | Giữ phần tử đầu tiên của mỗi giá trị “bằng nhau” |
| `Skip(n)` | Bỏ n phần tử đầu |
| `Take(n)` | Lấy tối đa n phần tử tiếp |
| `Chunk(size)` | Cắt thành các mảng cố định độ dài `size` |

**Paging cổ điển:** trang `pageIndex` (0-based), kích thước `pageSize`:

```csharp
source.Skip(pageIndex * pageSize).Take(pageSize)
```

**Distinct** mặc định dùng equality của kiểu (`Equals`/`GetHashCode`). Reference type chưa override → so sánh theo reference, không phải theo field.

## 4. Mô hình tư duy

```text
[1, 2, 2, 3, 3, 3]  Distinct → [1, 2, 3]

[A B C D E F]
 Skip(2) → [C D E F]
 Take(3) → [C D E]

Chunk(2) → [ [A B], [C D], [E F] ]
Chunk(4) → [ [A B C D], [E F] ]   // chunk cuối có thể ngắn hơn
```

## 5. Cú pháp

```csharp
IEnumerable<T> Distinct<T>(this IEnumerable<T> source);
IEnumerable<T> Distinct<T>(this IEnumerable<T> source, IEqualityComparer<T> comparer);
IEnumerable<T> DistinctBy<T, TKey>(this IEnumerable<T> source, Func<T, TKey> keySelector); // .NET 6+

IEnumerable<T> Skip<T>(this IEnumerable<T> source, int count);
IEnumerable<T> Take<T>(this IEnumerable<T> source, int count);
IEnumerable<T> TakeWhile<T>(this IEnumerable<T> source, Func<T, bool> predicate);
IEnumerable<T> SkipWhile<T>(this IEnumerable<T> source, Func<T, bool> predicate);

IEnumerable<T[]> Chunk<T>(this IEnumerable<T> source, int size);
```

## 6. Ví dụ

### Cơ bản

```csharp
var tags = new[] { "pvp", "rpg", "pvp", "mmo", "rpg" };
var unique = tags.Distinct().OrderBy(t => t);
// mmo, pvp, rpg

var page = Enumerable.Range(1, 100).Skip(20).Take(10);
// 21..30
```

### Trung cấp — DistinctBy + paging

```csharp
record Item(int Id, string Name, string Category);

var items = new List<Item>
{
    new(1, "Sword", "Weapon"),
    new(2, "Axe", "Weapon"),
    new(3, "Potion", "Consumable"),
    new(4, "Herb", "Consumable"),
};

// Một item đại diện mỗi Category
var onePerCat = items.DistinctBy(i => i.Category);

const int pageSize = 2;
int pageIndex = 1; // trang thứ 2
var pageItems = items
    .OrderBy(i => i.Id)
    .Skip(pageIndex * pageSize)
    .Take(pageSize);
```

### Nâng cao — Chunk batch xử lý

```csharp
var ids = Enumerable.Range(1, 10);

foreach (int[] batch in ids.Chunk(3))
{
    Console.WriteLine($"Batch ({batch.Length}): {string.Join(',', batch)}");
}
// 1,2,3 | 4,5,6 | 7,8,9 | 10

// Giả lập gửi API tối đa 100 id/lần
void SendBatches(IEnumerable<int> all)
{
    foreach (var chunk in all.Chunk(100))
        Console.WriteLine($"Send {chunk.Length} ids");
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Distinct “không ăn” trên class | Chưa override Equals/GetHashCode | `DistinctBy` theo key hoặc comparer |
| Paging sai trang | Quên OrderBy → thứ tự không ổn định | Luôn `OrderBy` trước Skip/Take |
| `Skip`/`Take` số âm | ArgumentOutOfRange | Validate input |
| `Chunk(0)` | Throw | `size >= 1` |
| Take hết rồi vẫn nghĩ còn data | Hết nguồn | Kiểm tra `Count` hoặc sentinel |

## 8. Gỡ lỗi

1. In `Count()` trước/sau Distinct để xem bao nhiêu trùng bị loại.
2. Paging: log `pageIndex`, `pageSize`, `Skip` offset.
3. Chunk: in `batch.Length` từng vòng — chunk cuối thường ngắn.
4. Với string Distinct: cân nhắc `StringComparer.OrdinalIgnoreCase`.

## 9. Best practices

- Paging: **OrderBy khóa ổn định** (Id, CreatedAt) trước Skip/Take.
- Prefer `DistinctBy(x => x.Id)` hơn Distinct trên entity phức tạp.
- `Chunk` cho batch DB/API/file — rõ ràng hơn vòng for thủ công.
- UI infinite scroll: `Skip` theo cursor/key thường tốt hơn offset lớn (nhưng in-memory thì Skip ổn).
- `TakeWhile`/`SkipWhile` khi điều kiện theo giá trị, không theo số lượng cố định.

## 10. Bài tập

**Bài 1** — `new[] { 1, 1, 2, 3, 3, 3 }` → Distinct → danh sách tăng dần.

**Bài 2** — Từ `Enumerable.Range(1, 50)`, lấy trang 3 với `pageSize = 10` (pageIndex 0-based → trang 3 là index 2).

**Bài 3** — `DistinctBy` trên `List<(int Id, string Name)>` theo `Name` (giữ bản ghi đầu).

**Bài 4** — Chia `Range(1, 25)` thành chunk 10; in số chunk và độ dài chunk cuối.

## 11. Gợi ý

- Bài 1: `Distinct().OrderBy(x => x)`.
- Bài 2: `Skip(2 * 10).Take(10)` → 21..30.
- Bài 3: `list.DistinctBy(x => x.Name)`.
- Bài 4: `Chunk(10)` — 3 chunk, cuối dài 5.

## 12. Đáp án

```csharp
var a = new[] { 1, 1, 2, 3, 3, 3 }.Distinct().OrderBy(x => x);

var page = Enumerable.Range(1, 50).Skip(2 * 10).Take(10);

var people = new List<(int Id, string Name)>
{
    (1, "An"), (2, "An"), (3, "Bình")
};
var distinctNames = people.DistinctBy(p => p.Name);

var chunks = Enumerable.Range(1, 25).Chunk(10).ToList();
Console.WriteLine(chunks.Count);          // 3
Console.WriteLine(chunks[^1].Length);     // 5
```

## 13. Đáp án thay thế

Distinct thủ công bằng `HashSet`:

```csharp
static IEnumerable<T> DistinctManual<T>(IEnumerable<T> source)
{
    var seen = new HashSet<T>();
    foreach (var item in source)
        if (seen.Add(item))
            yield return item;
}
```

Paging helper:

```csharp
static IEnumerable<T> Page<T>(IEnumerable<T> src, int pageIndex, int pageSize) =>
    src.Skip(pageIndex * pageSize).Take(pageSize);
```

## 14. Thử thách

Viết `PageInfo<T>` trả `(Items, TotalCount, TotalPages)` — **cẩn thận**: đếm total có thể enumerate hai lần (xem chương 5). Cách tốt: `ToList()` một lần hoặc `Count` trên `ICollection`.

## 15. Ứng dụng thực tế

- API phân trang
- Export CSV theo batch (`Chunk`)
- Unique email/tag trong import
- Sliding window demo với Skip/Take

## 16. Liên hệ Unity

- Inventory unique item ids: `Distinct`
- Leaderboard page: OrderBy score → Skip/Take
- Spawn enemies theo wave batch: `Chunk`
- Tránh Distinct mỗi frame trên list lớn — cache

## 17. Kiểm tra kiến thức

1. Công thức Skip cho trang `pageIndex` (0-based)?  
   **Đáp án:** `Skip(pageIndex * pageSize).Take(pageSize)`.

2. `Chunk(3)` trên 10 phần tử cho bao nhiêu chunk; chunk cuối dài bao nhiêu?  
   **Đáp án:** 4 chunk; cuối dài 1.

3. Vì sao Distinct trên class tùy chỉnh thường “sai”?  
   **Đáp án:** Equality theo reference nếu chưa override / không dùng DistinctBy.

4. `DistinctBy` xuất hiện từ .NET nào (gần đúng)?  
   **Đáp án:** .NET 6+.

5. Có nên OrderBy trước khi Skip/Take khi paging không?  
   **Đáp án:** Có — để thứ tự ổn định, trang không nhảy lung tung.
