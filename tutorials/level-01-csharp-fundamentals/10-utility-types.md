# Chương 10 — Utility types: DateTime, TimeSpan, Math, Random, Console

## 1. Mục tiêu học

- Thao tác ngày giờ với `DateTime` / `DateTimeOffset` (giới thiệu) và khoảng thời gian `TimeSpan`
- Dùng `Math` cho làm tròn, min/max, lượng giác cơ bản, `Abs`, `Pow`, `Sqrt`
- Sinh số ngẫu nhiên an toàn-học với `Random`
- Thành thạo API `Console` cho CLI thân thiện (màu, key, clear)

## 2. Điều kiện tiên quyết

- Thành thạo method, vòng lặp, string (Chương 6–9)
- Biết kiểu số và format chuỗi (`{value:F2}`)

## 3. Khái niệm

| Kiểu / lớp | Vai trò |
|------------|---------|
| `DateTime` | Một điểm thời gian (ngày + giờ) theo lịch |
| `TimeSpan` | Khoảng thời gian (duration): ngày, giờ, phút, giây, ms |
| `Math` | Hàm toán học tĩnh (`static`) |
| `Random` | Sinh số pseudo-random (không dùng cho bảo mật) |
| `Console` | Đọc/ghi terminal, màu, phím |

`DateTime.Now` = giờ máy local; `DateTime.UtcNow` = UTC. Khi trừ hai `DateTime` → nhận `TimeSpan`.

`Random`: tạo **một** instance tái sử dụng; tránh `new Random()` trong loop sát nhau (cùng seed thời gian → trùng chuỗi).

## 4. Mô hình tư duy

```text
DateTime  : "mốc trên timeline"
TimeSpan  : "độ dài đoạn" giữa hai mốc
Math      : hộp công cụ số học
Random    : xúc xắc có trạng thái
Console   : bàn phím + màn hình text
```

```text
start ──────── duration (TimeSpan) ────────► end
         end - start = TimeSpan
         start + TimeSpan = end
```

## 5. Cú pháp (C# thật)

```csharp
DateTime now = DateTime.Now;
DateTime utc = DateTime.UtcNow;
DateTime birthday = new DateTime(2000, 1, 15);
TimeSpan age = now - birthday;

double x = Math.Sqrt(2);
int r = Math.Clamp(15, 0, 10); // 10

var rng = new Random();
int dice = rng.Next(1, 7);        // 1..6
double p = rng.NextDouble();      // [0,1)

Console.ForegroundColor = ConsoleColor.Green;
Console.WriteLine("OK");
Console.ResetColor();
```

## 6. Ví dụ

### Cơ bản

Đếm ngược đơn giản:

```csharp
DateTime deadline = DateTime.Now.AddMinutes(5);
TimeSpan left = deadline - DateTime.Now;
Console.WriteLine($"Còn {left.Minutes} phút {left.Seconds} giây (xấp xỉ)");
```

### Trung cấp

Dice roller + thống kê:

```csharp
var rng = new Random();
int[] counts = new int[7]; // index 1..6
for (int i = 0; i < 6000; i++)
{
    int face = rng.Next(1, 7);
    counts[face]++;
}
for (int f = 1; f <= 6; f++)
{
    Console.WriteLine($"{f}: {counts[f]}");
}
```

### Nâng cao

Stopwatch đo thời gian thuật toán + format đẹp:

```csharp
using System.Diagnostics;

Console.Write("N = ");
int n = int.Parse(Console.ReadLine()!);
var sw = Stopwatch.StartNew();
long sum = 0;
for (int i = 1; i <= n; i++) sum += i;
sw.Stop();

Console.ForegroundColor = ConsoleColor.Cyan;
Console.WriteLine($"Tổng={sum}, mất {sw.Elapsed.TotalMilliseconds:F3} ms");
Console.ResetColor();
```

## 7. Lỗi thường gặp

| Lỗi | Nguyên nhân | Cách xử lý |
|-----|-------------|------------|
| Parse ngày sai culture | `dd/MM` vs `MM/dd` | `DateTime.TryParseExact` + format rõ |
| `Random` trùng | `new Random()` liên tục | Giữ một field/instance |
| Nhầm `Next(1,7)` | Max là **exclusive** | `Next(1,7)` → 1..6 |
| So sánh giờ không UTC | Local lệch timezone | Thống nhất UTC khi lưu |
| Quên `ResetColor` | Màu “dính” sau lỗi | `try/finally` hoặc luôn reset |

## 8. Gỡ lỗi

1. In `DateTime` với format: `{dt:O}` (round-trip) để thấy đủ thông tin.
2. Với `Random`: in 20 số đầu — nếu toàn giống nhau, kiểm tra chỗ `new Random()`.
3. `Math`: kiểm tra miền hợp lệ (`Sqrt` số âm → `NaN`).
4. Console: trên một số IDE/redirect, màu/key có thể khác terminal thật — thử Terminal ngoài.

## 9. Best practices

- Lưu/truyền thời gian quan trọng theo **UTC**; hiển thị mới convert local.
- `TryParseExact` khi format cố định (`"yyyy-MM-dd"`).
- Một `Random` dùng chung trong app nhỏ; với bảo mật dùng `RandomNumberGenerator`.
- `Math.Clamp` thay vì tự viết min/max lồng nhau.
- CLI: in hướng dẫn rõ, trả mã màu cho lỗi/thành công.

## 10. Bài tập

**Bài 1 — Ngày trong tuần**  
*Input:* chuỗi ngày `yyyy-MM-dd`.  
*Output:* in thứ tiếng Việt (Chủ nhật…Thứ bảy). Sai format → thông báo.

**Bài 2 — Khoảng cách ngày**  
*Input:* hai ngày `yyyy-MM-dd`.  
*Output:* số ngày tuyệt đối giữa chúng (`TimeSpan.Days`).

**Bài 3 — Làm tròn tiền**  
*Input:* một `double`.  
*Output:* `Math.Round` 0 chữ số và 2 chữ số; giải thích nhanh banker's rounding nếu thấy bất ngờ.

**Bài 4 — Đoán số Random**  
*Input:* chương trình random `1..100`; người dùng đoán.  
*Output:* cao/thấp/đúng; giới hạn 7 lần; in thắng/thua.

**Bài 5 — Console UI mini**  
*Input:* phím mũi tên trái/phải chọn menu 3 mục, Enter chọn, Esc thoát.  
*Output:* highlight mục đang chọn bằng màu; in mục đã chọn.

## 11. Gợi ý

- Bài 1: `DateTime.TryParseExact(s, "yyyy-MM-dd", CultureInfo.InvariantCulture, DateTimeStyles.None, out var dt)` rồi `dt.DayOfWeek`.
- Bài 4: `rng.Next(1, 101)`.
- Bài 5: `Console.ReadKey(true)`, vòng `while`, vẽ lại menu mỗi lần.

## 12. Đáp án

**Bài 1** — Thứ tiếng Việt:

```csharp
using System.Globalization;

Console.Write("yyyy-MM-dd: ");
string s = Console.ReadLine() ?? "";
if (!DateTime.TryParseExact(s, "yyyy-MM-dd", CultureInfo.InvariantCulture,
        DateTimeStyles.None, out DateTime dt))
{
    Console.WriteLine("Sai định dạng");
    return;
}

string[] vi = { "Chủ nhật", "Thứ hai", "Thứ ba", "Thứ tư", "Thứ năm", "Thứ sáu", "Thứ bảy" };
Console.WriteLine(vi[(int)dt.DayOfWeek]);
```

**Bài 2** — Khoảng cách ngày:

```csharp
using System.Globalization;

static bool ReadDate(string prompt, out DateTime dt)
{
    Console.Write(prompt);
    return DateTime.TryParseExact(Console.ReadLine() ?? "", "yyyy-MM-dd",
        CultureInfo.InvariantCulture, DateTimeStyles.None, out dt);
}

if (!ReadDate("Ngày 1: ", out var a) || !ReadDate("Ngày 2: ", out var b))
{
    Console.WriteLine("Sai định dạng");
    return;
}
Console.WriteLine($"Cách nhau {Math.Abs((a - b).Days)} ngày");
```

**Bài 3** — Round:

```csharp
Console.Write("x = ");
double x = double.Parse(Console.ReadLine()!);
Console.WriteLine($"0 chữ số: {Math.Round(x, 0)}");
Console.WriteLine($"2 chữ số: {Math.Round(x, 2)}");
```

**Bài 4** — Đoán số:

```csharp
var rng = new Random();
int secret = rng.Next(1, 101);
bool win = false;
for (int attempt = 1; attempt <= 7; attempt++)
{
    Console.Write($"Lần {attempt}/7: ");
    if (!int.TryParse(Console.ReadLine(), out int g))
    {
        Console.WriteLine("Nhập số!");
        continue;
    }
    if (g < secret) Console.WriteLine("Thấp");
    else if (g > secret) Console.WriteLine("Cao");
    else { Console.WriteLine("Đúng!"); win = true; break; }
}
Console.WriteLine(win ? "Thắng" : $"Thua — số là {secret}");
```

**Bài 5** — Menu mũi tên (rút gọn):

```csharp
string[] items = { "Bắt đầu", "Cài đặt", "Thoát" };
int index = 0;
while (true)
{
    Console.Clear();
    for (int i = 0; i < items.Length; i++)
    {
        if (i == index)
        {
            Console.ForegroundColor = ConsoleColor.Yellow;
            Console.WriteLine($"> {items[i]}");
            Console.ResetColor();
        }
        else Console.WriteLine($"  {items[i]}");
    }

    var key = Console.ReadKey(true).Key;
    if (key == ConsoleKey.UpArrow) index = (index + items.Length - 1) % items.Length;
    else if (key == ConsoleKey.DownArrow) index = (index + 1) % items.Length;
    else if (key == ConsoleKey.Enter) { Console.WriteLine($"Chọn: {items[index]}"); break; }
    else if (key == ConsoleKey.Escape) break;
}
```

## 13. Đáp án thay thế

Dùng `DateOnly` / `TimeOnly` (.NET 6+) khi chỉ cần ngày hoặc giờ:

```csharp
DateOnly d = DateOnly.ParseExact("2026-08-25", "yyyy-MM-dd");
Console.WriteLine(d.DayOfWeek);
```

Sinh ngẫu nhiên không lặp seed: `Random.Shared.Next(1, 7)` (.NET 6+).

## 14. Thử thách

Viết chương trình **pomodoro CLI**: làm việc 25 phút / nghỉ 5 phút (có thể rút còn giây để demo). In còn lại mỗi giây (`Thread.Sleep(1000)`), đổi màu khi chuyển phase. Cho phép phím `q` hủy (cần đọc key non-blocking — có thể poll `Console.KeyAvailable`).

## 15. Ứng dụng thực tế

Lịch họp, hết hạn token, báo cáo theo tuần, A/B test (`Random`), CLI devops tool (`Console`). `Stopwatch` đo performance trước khi tối ưu.

## 16. Liên hệ Unity

Unity dùng `Time.deltaTime`, `Time.time` — khác API `DateTime` (vẫn dùng `DateTime` cho clock thế giới thật / save timestamp). `Random` của .NET khác `UnityEngine.Random` — đừng nhầm namespace. `Mathf` tương tự `Math` nhưng `float`. Không block main thread bằng `Thread.Sleep` trong gameplay.

## 17. Kiểm tra kiến thức

1. Trừ hai `DateTime` ra kiểu gì?  
   **Đáp án:** `TimeSpan`

2. `Random.Next(1, 7)` có ra `7` không?  
   **Đáp án:** Không — cận trên exclusive; ra 1..6.

3. `DateTime.UtcNow` khác `Now` chỗ nào?  
   **Đáp án:** UTC vs giờ local của máy.

4. Vì sao không `new Random()` trong vòng lặp sát?  
   **Đáp án:** Dễ trùng seed theo thời gian → chuỗi số giống nhau.

5. `Math.Clamp(x, min, max)` làm gì?  
   **Đáp án:** Giữ `x` trong đoạn `[min, max]`.
