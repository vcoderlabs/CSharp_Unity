# Chương 6 — Luồng điều khiển: if, switch, vòng lặp

## 1. Mục tiêu học

- Viết nhánh `if` / `else if` / `else` và `switch` (statement + expression)
- Thành thạo `for`, `while`, `do-while`, `foreach`
- Dùng `break` / `continue` đúng mục đích

## 2. Điều kiện tiên quyết

- Thành thạo toán tử so sánh và logic (Chương 5)
- Biết biến, kiểu số và `string` cơ bản

## 3. Khái niệm

**Luồng điều khiển** quyết định thứ tự câu lệnh được thực thi — không phải lúc nào cũng chạy tuần tự từ trên xuống.

- **Rẽ nhánh:** chọn một khối lệnh theo điều kiện (`if`, `switch`).
- **Lặp:** lặp lại khối lệnh khi điều kiện còn đúng (hoặc duyệt tập phần tử).

| Cấu trúc | Khi nào dùng |
|----------|--------------|
| `if/else` | Điều kiện `bool` tự do |
| `switch` | Nhiều nhánh theo giá trị rời rạc / pattern |
| `for` | Biết số lần / có biến đếm |
| `while` | Lặp theo điều kiện, chưa biết trước số lần |
| `do-while` | Chạy ít nhất một lần rồi mới kiểm tra |
| `foreach` | Duyệt từng phần tử collection/chuỗi |
| `break` | Thoát vòng lặp / `switch` sớm |
| `continue` | Bỏ phần còn lại của lần lặp hiện tại |

## 4. Mô hình tư duy

```text
if        : "nếu ... thì ... không thì ..."
switch    : "theo giá trị X, vào nhánh tương ứng"
for       : "đếm i từ A đến B"
while     : "trong khi còn đúng thì làm"
do-while  : "làm đã, rồi hỏi còn làm tiếp không"
foreach   : "với mỗi phần tử trong tập"
```

Tránh vòng lặp vô hạn: luôn đảm bảo điều kiện sẽ `false` hoặc có `break`.

## 5. Cú pháp (C# thật)

```csharp
if (score >= 8) { /* ... */ }
else if (score >= 5) { /* ... */ }
else { /* ... */ }

switch (day)
{
    case 1: Console.WriteLine("Mon"); break;
    case 2: Console.WriteLine("Tue"); break;
    default: Console.WriteLine("Other"); break;
}

string label = day switch
{
    1 => "Mon",
    2 => "Tue",
    _ => "Other"
};

for (int i = 0; i < 5; i++) { }
while (n > 0) { n--; }
do { /* ... */ } while (again);

foreach (char c in "abc") { }
```

## 6. Ví dụ

### Cơ bản

In số từ 1 đến N:

```csharp
Console.Write("N = ");
int n = int.Parse(Console.ReadLine()!);
for (int i = 1; i <= n; i++)
{
    Console.Write(i + " ");
}
```

### Trung cấp

Menu lặp đến khi chọn thoát:

```csharp
string cmd;
do
{
    Console.WriteLine("1) Xin chào  2) Giờ hệ thống  0) Thoát");
    Console.Write("Chọn: ");
    cmd = Console.ReadLine() ?? "";
    switch (cmd)
    {
        case "1":
            Console.WriteLine("Hello!");
            break;
        case "2":
            Console.WriteLine(DateTime.Now);
            break;
        case "0":
            Console.WriteLine("Bye");
            break;
        default:
            Console.WriteLine("Không hợp lệ");
            break;
    }
} while (cmd != "0");
```

### Nâng cao

Tìm số nguyên tố trong khoảng, dùng `continue` / `break`:

```csharp
Console.Write("Đến N: ");
int n = int.Parse(Console.ReadLine()!);
for (int x = 2; x <= n; x++)
{
    bool prime = true;
    for (int d = 2; d * d <= x; d++)
    {
        if (x % d == 0)
        {
            prime = false;
            break;
        }
    }
    if (!prime) continue;
    Console.Write(x + " ");
}
```

## 7. Lỗi thường gặp

| Lỗi | Nguyên nhân | Cách xử lý |
|-----|-------------|------------|
| Vòng lặp vô hạn | Quên cập nhật biến điều kiện | Kiểm tra biến đếm / cờ thoát |
| `case` thiếu `break` (switch cổ điển) | Fall-through không chủ đích | Thêm `break` hoặc dùng switch expression |
| Off-by-one | `i <= n` vs `i < n` | Viết ví dụ nhỏ trên giấy |
| Sửa collection trong `foreach` | Không được phép theo cách thông thường | Dùng `for` ngược / copy list |
| Nhầm `=` trong `if` | Không compile với `bool` | Dùng `==` |

## 8. Gỡ lỗi

1. In biến đếm mỗi vòng: `Console.WriteLine($"i={i}")`.
2. Đặt breakpoint tại điều kiện `while` / `for`.
3. Vẽ flowchart cho bài có nhiều nhánh.
4. Với menu: log `cmd` ngay sau `ReadLine` để xem ký tự thừa/space.

## 9. Best practices

- Prefer `foreach` khi chỉ đọc từng phần tử.
- Switch expression khi chỉ map giá trị → giá trị.
- Giữ thân vòng lặp ngắn; tách method nếu quá dài (Chương 7).
- Đặt tên cờ rõ: `bool keepRunning = true`.

## 10. Bài tập

**Bài 1 — FizzBuzz**  
*Input:* `N`.  
*Output:* với mỗi `i` từ 1..N: chia hết 15 → `FizzBuzz`, 3 → `Fizz`, 5 → `Buzz`, else in `i` (mỗi giá trị một dòng).

**Bài 2 — Tổng 1..N**  
*Input:* `N` (≥1).  
*Output:* tổng bằng `while` (không dùng công thức `N*(N+1)/2` — trừ khi làm đáp án thay thế).

**Bài 3 — Đoán số**  
*Input:* chương trình chọn sẵn `secret = 7` (cố định để chấm); người dùng đoán nhiều lần.  
*Output:* mỗi lần: `Lớn hơn` / `Nhỏ hơn` / `Đúng!`; dừng khi đúng. In số lần đoán.

**Bài 4 — Đảo chuỗi**  
*Input:* một chuỗi.  
*Output:* chuỗi đảo bằng vòng lặp `for` (chưa cần LINQ/`Reverse`).

**Bài 5 — Menu tính toán**  
*Input:* lặp menu `+ - * /` hai số; `q` thoát.  
*Output:* kết quả hoặc lỗi chia 0; bỏ qua input lệnh không hợp lệ (`continue`).

## 11. Gợi ý

- FizzBuzz: kiểm tra `% 15` trước `% 3` / `% 5`.
- Đoán số: `while (true)` + `break` khi đúng.
- Đảo chuỗi: duyệt `i` từ `Length-1` về `0`, nối vào `result`.

## 12. Đáp án

**Bài 1** — FizzBuzz:

```csharp
Console.Write("N = ");
int n = int.Parse(Console.ReadLine()!);
for (int i = 1; i <= n; i++)
{
    if (i % 15 == 0) Console.WriteLine("FizzBuzz");
    else if (i % 3 == 0) Console.WriteLine("Fizz");
    else if (i % 5 == 0) Console.WriteLine("Buzz");
    else Console.WriteLine(i);
}
```

**Bài 2** — Tổng bằng `while`:

```csharp
Console.Write("N = ");
int n = int.Parse(Console.ReadLine()!);
int i = 1;
int sum = 0;
while (i <= n)
{
    sum += i;
    i++;
}
Console.WriteLine($"Tổng = {sum}");
```

**Bài 3** — Đoán số:

```csharp
const int Secret = 7;
int guesses = 0;
while (true)
{
    Console.Write("Đoán: ");
    if (!int.TryParse(Console.ReadLine(), out int g))
    {
        Console.WriteLine("Nhập số!");
        continue;
    }
    guesses++;
    if (g < Secret) Console.WriteLine("Nhỏ hơn");
    else if (g > Secret) Console.WriteLine("Lớn hơn");
    else
    {
        Console.WriteLine($"Đúng! Số lần đoán: {guesses}");
        break;
    }
}
```

**Bài 4** — Đảo chuỗi:

```csharp
Console.Write("Chuỗi: ");
string s = Console.ReadLine() ?? "";
string rev = "";
for (int i = s.Length - 1; i >= 0; i--)
{
    rev += s[i];
}
Console.WriteLine(rev);
```

**Bài 5** — Menu tính toán (rút gọn):

```csharp
while (true)
{
    Console.Write("Lệnh (+ - * / hoặc q): ");
    string op = (Console.ReadLine() ?? "").Trim();
    if (op == "q") break;
    if (op is not ("+" or "-" or "*" or "/"))
    {
        Console.WriteLine("Lệnh sai");
        continue;
    }

    Console.Write("a = ");
    if (!double.TryParse(Console.ReadLine(), out double a)) { Console.WriteLine("Sai số"); continue; }
    Console.Write("b = ");
    if (!double.TryParse(Console.ReadLine(), out double b)) { Console.WriteLine("Sai số"); continue; }

    if (op == "/" && b == 0) { Console.WriteLine("Chia 0"); continue; }

    double r = op switch
    {
        "+" => a + b,
        "-" => a - b,
        "*" => a * b,
        _ => a / b
    };
    Console.WriteLine($"= {r}");
}
```

## 13. Đáp án thay thế

Tổng 1..N bằng công thức (đối chiếu kết quả vòng lặp):

```csharp
long sum = (long)n * (n + 1) / 2;
```

Đảo chuỗi bằng `string.Create` / mảng `char` (hiệu quả hơn nối `+=`):

```csharp
char[] chars = s.ToCharArray();
Array.Reverse(chars);
Console.WriteLine(new string(chars));
```

## 14. Thử thách

In tam giác số:

```text
1
1 2
1 2 3
...
1..N
```

Dùng vòng lặp lồng nhau. Sau đó in tam giác ngược.

## 15. Ứng dụng thực tế

Workflow phê duyệt đơn hàng, retry gọi API (`while` + đếm), parser đơn giản, game loop console — đều là control flow. Bug off-by-one rất phổ biến ở đây.

## 16. Liên hệ Unity

`Update()` là vòng lặp do engine gọi mỗi frame — bạn ít viết `while(true)` trên main thread (sẽ treo game). Trong method gameplay vẫn dùng `if`/`switch`/`for` bình thường. Coroutine / async là chủ đề sau; đừng `while(true)` busy-wait trong `Update`.

## 17. Kiểm tra kiến thức

1. `do-while` khác `while` chỗ nào?  
   **Đáp án:** `do-while` luôn chạy thân ít nhất một lần.

2. `break` làm gì trong vòng lặp?  
   **Đáp án:** Thoát ngay khỏi vòng lặp đang chứa nó.

3. `continue` làm gì?  
   **Đáp án:** Bỏ phần còn lại của lần lặp hiện tại, sang lần kế.

4. Khi nào nên dùng `foreach`?  
   **Đáp án:** Khi cần duyệt từng phần tử và không cần chỉ số phức tạp / sửa cấu trúc collection.

5. Switch expression khác switch statement?  
   **Đáp án:** Expression trả về giá trị; gọn khi chỉ map giá trị, không cần `break`.
