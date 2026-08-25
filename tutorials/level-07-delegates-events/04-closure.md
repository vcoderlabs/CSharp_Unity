# Chương 4 — Closure

## 1. Mục tiêu học

- Hiểu **closure**: lambda/anonymous bắt biến từ scope ngoài
- Phân biệt capture **giá trị** vs **biến** (reference tới storage)
- Tránh bug kinh điển vòng `for` / `foreach`
- Biết hệ quả vòng đời: object sống lâu vì bị capture

## 2. Điều kiện tiên quyết

- Chương 3: lambda
- Level 3: value vs reference (hữu ích khi capture object)

## 3. Khái niệm

**Closure** xảy ra khi hàm lồng “nhớ” biến bên ngoài sau khi scope ngoài đã kết thúc về mặt cú pháp.

```csharp
Func<int> Counter()
{
    int n = 0;
    return () => ++n; // capture biến n
}

var c = Counter();
Console.WriteLine(c()); // 1
Console.WriteLine(c()); // 2
```

Compiler sinh class hiển thị (`DisplayClass`) giữ field `n`.

### Pitfall vòng lặp (hiểu hiện đại)

C# 5+ `foreach` mỗi iteration biến mới. `for` với biến `i` **một** biến — lambda capture `i` dễ lệch:

```csharp
var actions = new List<Action>();
for (int i = 0; i < 3; i++)
    actions.Add(() => Console.WriteLine(i));

foreach (var a in actions)
    a(); // thường in 3 3 3 — vì i đã = 3 khi gọi
```

Fix: bản sao local:

```csharp
for (int i = 0; i < 3; i++)
{
    int copy = i;
    actions.Add(() => Console.WriteLine(copy));
}
```

## 4. Mô hình tư duy

```text
Scope ngoài có biến x
Lambda dùng x → x được “đóng gói” vào closure object
Khi lambda chạy sau này → đọc/ghi cùng storage đó

Muốn “chụp ảnh” giá trị lúc tạo → copy vào biến local riêng
Muốn chia sẻ counter → cố ý dùng chung một biến
```

## 5. Cú pháp

```csharp
int factor = 10;
Func<int, int> mul = x => x * factor; // capture factor
factor = 2;
Console.WriteLine(mul(5)); // 10 — dùng giá trị hiện tại của factor (=2) → 10

// Factory closure
static Predicate<int> Between(int min, int max) =>
    n => n >= min && n <= max;
```

## 6. Ví dụ

### Cơ bản

```csharp
string name = "Ada";
Action hi = () => Console.WriteLine($"Hi {name}");
name = "Grace";
hi(); // Hi Grace — capture biến, không phải snapshot chuỗi lúc tạo (trừ khi copy)
```

### Trung cấp

```csharp
static Func<int> MakeAdder(int start)
{
    int total = start;
    return () =>
    {
        total += 1;
        return total;
    };
}
```

### Nâng cao

Tránh giữ object lớn:

```csharp
byte[] huge = LoadMegabytes();
// Bad nếu chỉ cần length nhưng capture cả huge:
// Func<int> f = () => huge.Length;

int len = huge.Length;
Func<int> f = () => len; // chỉ giữ int
huge = null; // có thể GC mảng nếu không ai giữ
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Mọi callback in cùng index cuối | Capture biến `i` vòng for | `int copy = i;` |
| Memory không giảm | Closure giữ object lớn / MonoBehaviour | Capture field cần thiết; hủy subscribe |
| State “nhảy” giữa lần gọi | Cố ý share biến | Document hoặc tách instance factory |
| Nghĩ capture luôn copy value type | Value type cũng có thể là biến shared | Copy tường minh nếu cần snapshot |

## 8. Gỡ lỗi

1. Decompile / Breakpoint: xem `this` của closure là `DisplayClass`.  
2. In giá trị biến ngay trong lambda khi tạo vs khi gọi.  
3. Unit test vòng for tạo list Action — assert output.

## 9. Best practices

- Capture tối thiểu; snapshot khi tạo delayed jobs.
- Vòng for + async/lambda: luôn cẩn thận copy.
- Đặt tên `capturedX` / `localCopy` khi review khó.
- Unity: cẩn thận capture `this` — object Destroy vẫn bị giữ bởi event tĩnh/singleton.

## 10. Bài tập

**Bài 1** — `MakeCounter(int start)` trả `Func<int>` tăng dần.

**Bài 2** — Tái hiện bug for 0..2 in ra 3,3,3; rồi sửa bằng copy.

**Bài 3** — `GreaterThan(int th)` trả `Predicate<int>`.

**Bài 4** — Hai counter độc lập từ `MakeCounter(0)` — chứng minh không share state.

## 11. Gợi ý

- Bài 1: `int n = start; return () => ++n;`
- Bài 2: list `Action`, gọi sau vòng lặp.
- Bài 4: `var a = MakeCounter(0); var b = MakeCounter(0);`

## 12. Đáp án

**Bài 1** — Counter có closure:

```csharp
static Func<int> MakeCounter(int start)
{
    int n = start;
    return () => ++n;
}

var c = MakeCounter(10);
Console.WriteLine(c()); // 11
Console.WriteLine(c()); // 12
```

**Bài 2** — Bug và fix:

```csharp
// Bug
var buggy = new List<Action>();
for (int i = 0; i < 3; i++)
    buggy.Add(() => Console.WriteLine(i));
foreach (var a in buggy) a(); // 3 3 3

// Fix
var ok = new List<Action>();
for (int i = 0; i < 3; i++)
{
    int copy = i;
    ok.Add(() => Console.WriteLine(copy));
}
foreach (var a in ok) a(); // 0 1 2
```

**Bài 3** — Predicate từ threshold:

```csharp
static Predicate<int> GreaterThan(int threshold) =>
    value => value > threshold;

var gt5 = GreaterThan(5);
Console.WriteLine(gt5(6)); // True
```

**Bài 4** — Hai instance độc lập:

```csharp
var a = MakeCounter(0);
var b = MakeCounter(0);
Console.WriteLine(a()); // 1
Console.WriteLine(a()); // 2
Console.WriteLine(b()); // 1  — không bị a ảnh hưởng
```

## 13. Đáp án thay thế

Dùng class `Counter { public int Next() ... }` thay closure — rõ ràng hơn khi state phức tạp. Local function capture tương tự lambda.

## 14. Thử thách

Tạo `List<Func<int>>` trong `for` trả về `i * i` đúng với từng i (0..9) khi gọi sau — không dùng `copy` sai chỗ.

## 15. Ứng dụng thực tế

- Factory cấu hình (min/max, retry count)
- Deferred execution LINQ (capture)
- UI: nút trong vòng tạo — bug index kinh điển web/game UI
- Middleware gắn request id

## 16. Liên hệ Unity

- Spawn 10 button trong loop: `onClick.AddListener(() => Pick(i))` → copy `i`
- Closure giữ MonoBehaviour → ngăn GC / giả “sống” sau Destroy
- Coroutine + lambda capture cùng pitfall
- Singleton event tĩnh + lambda `this` = leak scene objects

## 17. Kiểm tra kiến thức

1. Closure là gì (một câu)?  
   **Đáp án:** Hàm nhớ/sử dụng biến từ scope bao ngoài nhờ object compiler sinh ra.

2. Vì sao for + lambda hay in cùng giá trị cuối?  
   **Đáp án:** Mọi lambda bắt cùng một biến `i` đã tăng hết vòng.

3. Cách fix phổ biến?  
   **Đáp án:** Biến local copy trong vòng lặp.

4. Capture có thể giữ object sống lâu hơn không?  
   **Đáp án:** Có — root từ delegate/event còn sống.

5. Hai lần gọi `MakeCounter(0)` share `n` không?  
   **Đáp án:** Không — mỗi lần một closure/storage riêng.
