# Chương 8 — Index, range, local functions, anonymous types

## 1. Mục tiêu học

- Dùng **`Index`** (`^`) và **`Range`** (`..`) trên mảng / `Span` / kiểu hỗ trợ
- Viết **local function** trong method (kể cả static local)
- Dùng **anonymous types** (`new { ... }`) đúng ngữ cảnh — và biết khi nên chuyển sang `record`
- Kết hợp với LINQ projection ngắn

## 2. Điều kiện tiên quyết

- Level 1: mảng, method
- Level 8: Select projection
- Chương 2: record vs anonymous

## 3. Khái niệm

### Index & Range (C# 8+)

| Cú pháp | Nghĩa |
|---------|--------|
| `^1` | Phần tử cuối |
| `^2` | Kế cuối |
| `a..b` | Từ index a đến **trước** b |
| `a..` | Từ a đến hết |
| `..b` | Từ đầu đến trước b |
| `^3..^1` | Cắt từ đuôi |

`arr[^1]` ≡ `arr[arr.Length - 1]` (với length > 0).

### Local functions

Hàm khai báo trong thân method — đóng scope, có thể capture biến cục bộ, không làm bẩn class API.

### Anonymous types

`new { Name = "Ada", Level = 10 }` — class bất biến do compiler sinh, chỉ dùng rõ trong method/assembly projection tạm. Không tiện làm public API return type (kiểu là `internal`).

## 4. Mô hình tư duy

```text
Index từ đầu: 0,1,2,…
Index từ cuối: ^1, ^2, ^3,…

Range = nửa khoảng [start, end)

Local function = helper “riêng tư” của một method
Anonymous = túi property tạm cho LINQ / log
Record đặt tên = khi túi đó sống lâu / public
```

## 5. Cú pháp

```csharp
int[] a = [1, 2, 3, 4, 5]; // collection expression (.NET 8 / C# 12)
int last = a[^1];
int[] mid = a[1..^1]; // 2,3,4

void Outer(int n)
{
    int Factorial(int x) => x <= 1 ? 1 : x * Factorial(x - 1);
    Console.WriteLine(Factorial(n));

    static int Square(int x) => x * x; // không capture
    Console.WriteLine(Square(n));
}

var anon = new { Name = "Ada", Level = 10 };
Console.WriteLine(anon.Name);
```

## 6. Ví dụ

### Cơ bản — index/range

```csharp
string s = "Hello";
Console.WriteLine(s[^1]);   // o
Console.WriteLine(s[1..^1]); // ell

var page = Enumerable.Range(1, 10).ToArray()[2..5]; // 3,4,5
```

### Trung cấp — local function

```csharp
static int CountValid(IEnumerable<string?> lines)
{
    bool IsValid(string? line) => !string.IsNullOrWhiteSpace(line);

    int count = 0;
    foreach (var line in lines)
        if (IsValid(line))
            count++;
    return count;
}
```

### Nâng cao — anonymous + LINQ rồi materialize record

```csharp
record Row(string Region, decimal Total);

var sales = new[]
{
    new { Region = "N", Amount = 10m },
    new { Region = "N", Amount = 5m },
    new { Region = "S", Amount = 8m },
};

var rows = sales
    .GroupBy(x => x.Region)
    .Select(g => new { Region = g.Key, Total = g.Sum(x => x.Amount) }) // anonymous
    .Select(x => new Row(x.Region, x.Total)) // đổi sang record nếu cần trả về
    .ToList();
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| `IndexOutOfRange` với `^1` | Mảng rỗng | Check Length |
| Nhầm `a[1..3]` gồm 3? | End exclusive | Gồm index 1,2 — không gồm 3 |
| Trả anonymous từ public API | Kiểu không nói được rõ | Dùng record/tuple |
| Local function capture nhầm vòng lặp | Closure | Copy biến cục bộ / static local |
| `..` trên List cũ | OK trên List; một số kiểu custom cần indexer | Dùng LINQ Skip/Take |

## 8. Gỡ lỗi

1. In `Length` trước `[^1]`.
2. Với range: tính `start`/`end` ra số nguyên rồi slice.
3. Local: đặt breakpoint trong local function.
4. Anonymous: hover trong IDE xem kiểu compiler sinh `<>f__AnonymousType…`.

## 9. Best practices

- `^1` thay `Length - 1` khi rõ ràng hơn.
- Local function cho helper chỉ dùng một chỗ — giảm member class.
- `static` local khi không cần capture — rõ ràng + đôi khi tối ưu.
- Anonymous OK trong method; public surface → `record`.
- Collection expressions `[1,2,3]` (.NET 8) gọn hơn `new[]` khi phù hợp.

## 10. Bài tập

**Bài 1** — Lấy phần tử kế cuối của mảng bằng `^`.

**Bài 2** — Cắt bỏ phần tử đầu và cuối bằng range.

**Bài 3** — Method `PrintOdds(int[] xs)` với local function `IsOdd`.

**Bài 4** — LINQ Select anonymous `{ n, Square = n*n }` rồi in.

## 11. Gợi ý

- Bài 1: `xs[^2]` (cần length ≥ 2).
- Bài 2: `xs[1..^1]`.
- Bài 3: local `bool IsOdd(int n) => n % 2 != 0`.
- Bài 4: `Select(n => new { n, Square = n * n })`.

## 12. Đáp án

```csharp
static int SecondLast(int[] xs) => xs[^2];

static int[] TrimEnds(int[] xs) => xs[1..^1];

static void PrintOdds(int[] xs)
{
    bool IsOdd(int n) => n % 2 != 0;
    foreach (var n in xs)
        if (IsOdd(n))
            Console.WriteLine(n);
}

foreach (var row in new[] { 2, 3, 4 }.Select(n => new { n, Square = n * n }))
    Console.WriteLine($"{row.n}^2={row.Square}");
```

## 13. Đáp án thay thế

Bài 2: `Skip(1).SkipLast(1).ToArray()`. Bài 4: dùng `record` ngay từ Select. Local `IsOdd` có thể là static local.

## 14. Thử thách

Viết indexer-friendly wrapper `RingBuffer<T>` hỗ trợ `this[Index i]` và `this[Range r]` (implement bằng `Length` + chuyển Index).

## 15. Ứng dụng thực tế

- Xử lý chuỗi/path slice
- Parser token windows
- LINQ tạm trong controller/service
- Giữ class sạch bằng local helpers

## 16. Liên hệ Unity

- Mảng waypoints: `points[^1]` đích cuối
- Slice animation key range
- Local function trong method dài MonoBehaviour (vẫn nên tách khi quá lớn)
- Anonymous ít dùng serialize — Unity cần kiểu tường minh

## 17. Kiểm tra kiến thức

1. `^1` là gì?  
   **Đáp án:** Index từ cuối — phần tử cuối cùng.

2. `a[1..3]` lấy những index nào?  
   **Đáp án:** 1 và 2 (end exclusive).

3. Local function khác private method?  
   **Đáp án:** Scope trong method cha; có thể capture biến cục bộ; không phải member class.

4. Vì sao hạn chế trả anonymous type public?  
   **Đáp án:** Tên kiểu do compiler sinh, không ổn định/rõ cho API; khó khai báo kiểu trả về.

5. `static` local function nghĩa là gì?  
   **Đáp án:** Không capture biến từ enclosing scope.
