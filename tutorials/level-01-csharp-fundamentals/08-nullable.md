# Chương 8 — Nullable value types (`int?` và họ hàng)

## 1. Mục tiêu học

- Hiểu vì sao `int` bình thường không thể là “không có giá trị”
- Dùng `int?`, `bool?`, `DateTime?` … đúng cách
- Thành thạo `HasValue`, `.Value`, `??`, `??=`, và pattern `is null`

## 2. Điều kiện tiên quyết

- Biết value types cơ bản (`int`, `bool`, `double`, …) — Chương 4
- Biết method và `return` — Chương 7
- Đã thấy `string?` / nullable reference (bật sẵn trong template .NET) — phân biệt với nullable **value** type

## 3. Khái niệm

`int`, `bool`, `DateTime` là **value types**: luôn có một giá trị bit cụ thể. Không có trạng thái “trống” tự nhiên như `string` có thể `null`.

**Nullable value type** `T?` (tương đương `Nullable<T>`) bọc value type để thêm trạng thái **không có giá trị**.

Ví dụ thực tế: tuổi chưa nhập, điểm bài kiểm tra vắng, ngày kết thúc hợp đồng chưa biết.

Với **nullable reference types** (`string?`): là cảnh báo compiler về reference có thể null — khác cơ chế `Nullable<T>` nhưng cùng ký hiệu `?`.

## 4. Mô hình tư duy

```text
int     : luôn có số (0 cũng là giá trị hợp lệ!)
int?    : "có số" hoặc "không có"

box int?
  [HasValue=false]        → null
  [HasValue=true, 42]     → 42
```

`0` ≠ “không có”: nếu dùng `0` làm sentinel cho “chưa nhập tuổi”, bạn không phân biệt được trẻ sơ sinh tuổi 0 (hiếm) hoặc dữ liệu thiếu — `int?` rõ nghĩa hơn.

## 5. Cú pháp (C# thật)

```csharp
int? age = null;
age = 20;

if (age.HasValue)
{
    Console.WriteLine(age.Value);
}

int x = age ?? -1;          // nếu null thì -1
age ??= 18;                 // nếu null thì gán 18

if (age is int a)
{
    Console.WriteLine(a);
}

int? ParseAge(string? s)
    => int.TryParse(s, out int v) ? v : null;
```

## 6. Ví dụ

### Cơ bản

Nhập tuổi tùy chọn — Enter trống → null:

```csharp
Console.Write("Tuổi (Enter để bỏ qua): ");
string? line = Console.ReadLine();
int? age = string.IsNullOrWhiteSpace(line) ? null : int.Parse(line);

Console.WriteLine(age is null ? "Chưa có tuổi" : $"Tuổi = {age}");
```

### Trung cấp

Trung bình danh sách điểm, bỏ qua bài thiếu (`null`):

```csharp
int?[] scores = { 8, null, 7, null, 9 };
int count = 0;
int sum = 0;
foreach (int? s in scores)
{
    if (s is null) continue;
    sum += s.Value;
    count++;
}
Console.WriteLine(count == 0 ? "Không có điểm" : $"TB = {(double)sum / count:F2}");
```

### Nâng cao

API trả `int?` và dùng null-coalescing trong biểu thức:

```csharp
static int? FindIndex(string[] items, string target)
{
    for (int i = 0; i < items.Length; i++)
    {
        if (items[i] == target) return i;
    }
    return null;
}

string[] names = { "An", "Bình", "Chi" };
int index = FindIndex(names, "Bình") ?? -1;
Console.WriteLine(index);
```

## 7. Lỗi thường gặp

| Lỗi | Nguyên nhân | Cách xử lý |
|-----|-------------|------------|
| `Nullable object must have a value` | Gọi `.Value` khi null | Kiểm `HasValue` / dùng `??` / pattern |
| Gán `int?` vào `int` trực tiếp | Thiếu xử lý null | `int x = n ?? 0` hoặc `.Value` sau khi chắc |
| Nhầm `0` với null | Sentinel mơ hồ | Dùng `int?` đúng nghĩa “thiếu” |
| So sánh `age == null` vs `!age.HasValue` | Cả hai gần tương đương | Chọn một style nhất quán (`is null` hiện đại) |

## 8. Gỡ lỗi

1. Khi exception `.Value`: tìm chỗ không kiểm tra null.
2. In `age.HasValue` và `age` (sẽ hiện trống nếu null).
3. Trong IDE, tip kiểu `int?` vs `int` giúp phát hiện gán sai.
4. Viết test case: input trống, input `"abc"`, input `"0"` — ba đường khác nhau.

## 9. Best practices

- Dùng `T?` khi “thiếu giá trị” là trạng thái hợp lệ của domain.
- Prefer `??` / pattern matching hơn `.Value` trần.
- Đừng abuse `null` — đôi khi `bool TryGet(out T)` rõ hơn `T?`.
- Phân biệt rõ nullable value (`int?`) và nullable reference (`string?`).

## 10. Bài tập

**Bài 1 — Parse tùy chọn**  
*Input:* một dòng.  
*Output:* nếu parse được `int` thì in số; nếu trống hoặc sai → in `null` (chữ).

**Bài 2 — Min nullable**  
*Input:* ba dòng, mỗi dòng số hoặc trống.  
*Output:* số nhỏ nhất trong các giá trị *có*; nếu tất cả trống → `Không có dữ liệu`.

**Bài 3 — `??` mặc định**  
*Input:* chuỗi số lượng; trống → coi như 1.  
*Output:* in `Số lượng = ...` dùng `int?` + `??`.

**Bài 4 — Điểm danh**  
*Input:* `n` rồi `n` dòng điểm (`int` hoặc chữ `v` = vắng → null).  
*Output:* số buổi có mặt và điểm trung bình chỉ tính buổi có điểm.

**Bài 5 — API `Try` vs nullable**  
*Input:* chuỗi.  
*Output:* viết cả hai: `bool TryParsePositive(string s, out int v)` và `int? ParsePositive(string s)`; minh họa gọi cả hai với cùng input.

## 11. Gợi ý

- Bài 2: giữ `int? min = null`; khi gặp số, `min = min is null ? x : Math.Min(min.Value, x)`.
- Bài 4: `v` / `V` → null; số → `int`.
- Đừng gọi `.Value` trước khi chắc `HasValue`.

## 12. Đáp án

**Bài 1** — Parse tùy chọn:

```csharp
Console.Write("Input: ");
string? line = Console.ReadLine();
int? n = int.TryParse(line, out int v) ? v : null;
Console.WriteLine(n is null ? "null" : n.Value.ToString());
```

**Bài 2** — Min nullable:

```csharp
int? min = null;
for (int i = 0; i < 3; i++)
{
    Console.Write($"Số {i + 1} (trống = bỏ): ");
    string? line = Console.ReadLine();
    if (string.IsNullOrWhiteSpace(line)) continue;
    if (!int.TryParse(line, out int x)) continue;
    min = min is null ? x : Math.Min(min.Value, x);
}
Console.WriteLine(min is null ? "Không có dữ liệu" : min.Value.ToString());
```

**Bài 3** — Mặc định 1:

```csharp
Console.Write("Số lượng: ");
string? line = Console.ReadLine();
int? qty = int.TryParse(line, out int q) ? q : null;
int finalQty = qty ?? 1;
Console.WriteLine($"Số lượng = {finalQty}");
```

**Bài 4** — Điểm danh (rút gọn):

```csharp
Console.Write("n = ");
int n = int.Parse(Console.ReadLine()!);
int present = 0, sum = 0;
for (int i = 0; i < n; i++)
{
    Console.Write($"Điểm[{i}]: ");
    string? line = (Console.ReadLine() ?? "").Trim();
    int? score = line.Equals("v", StringComparison.OrdinalIgnoreCase)
        ? null
        : int.TryParse(line, out int s) ? s : null;

    if (score is null) continue;
    present++;
    sum += score.Value;
}
Console.WriteLine(present == 0
    ? "Không có điểm"
    : $"Có mặt {present}, TB = {(double)sum / present:F2}");
```

**Bài 5** — Hai phong cách API:

```csharp
static bool TryParsePositive(string s, out int v)
{
    v = 0;
    return int.TryParse(s, out v) && v > 0;
}

static int? ParsePositive(string s)
    => TryParsePositive(s, out int v) ? v : null;

string input = Console.ReadLine() ?? "";
Console.WriteLine(TryParsePositive(input, out int a) ? a : "fail");
Console.WriteLine(ParsePositive(input) is int b ? b : "null");
```

## 13. Đáp án thay thế

Dùng `GetValueOrDefault()`:

```csharp
int x = age.GetValueOrDefault(-1);
```

Lifted operators: `int? a = 3; int? b = null; int? c = a + b; // null`.

## 14. Thử thách

Viết method `int? IndexOf(int[] arr, int target)` trả về chỉ số đầu tiên hoặc `null`. So sánh với trả `-1` kiểu C cổ điển — viết đoạn comment giải thích trade-off.

## 15. Ứng dụng thực tế

Database NULL map sang `int?` / `DateTime?` trong C#; JSON thiếu field; form optional. Sai xử lý null là nguồn `NullReferenceException` / invalid operation hàng đầu.

## 16. Liên hệ Unity

Unity API cổ thường dùng sentinel (`null` cho reference, `-1` cho index). Với C# hiện đại trong project, `int?` hữu ích cho data layer / tool editor. Trên hot path gameplay, đôi khi team vẫn prefer `bool TryGet` để tránh overhead/`Nullable<T>` — chọn theo convention dự án.

## 17. Kiểm tra kiến thức

1. `int` có thể gán `null` không?  
   **Đáp án:** Không — phải dùng `int?`.

2. `.Value` khi `HasValue == false` gây ra gì?  
   **Đáp án:** Ném `InvalidOperationException`.

3. `??` làm gì?  
   **Đáp án:** Trả vế trái nếu không null; ngược lại trả vế phải.

4. `0` có phải là `null` của `int?` không?  
   **Đáp án:** Không — `0` là giá trị hợp lệ; `null` là không có giá trị.

5. `T?` với value type thực chất là gì?  
   **Đáp án:** `Nullable<T>`.
