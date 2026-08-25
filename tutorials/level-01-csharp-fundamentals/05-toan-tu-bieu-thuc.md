# Chương 5 — Toán tử và biểu thức

## 1. Mục tiêu học

- Dùng toán tử số học, gán, so sánh, logic, điều kiện `?:`
- Hiểu độ ưu tiên toán tử và dùng ngoặc đúng
- Biết các toán tử hữu ích: `%`, `++`/`--`, `typeof`, `is`, `as` (giới thiệu nhẹ)

## 2. Điều kiện tiên quyết

- Thành thạo kiểu nguyên thủy (Chương 4)
- Biết biến và biểu thức đơn giản

## 3. Khái niệm

**Biểu thức (expression)** là tổ hợp toán hạng + toán tử tạo ra một giá trị (ví dụ `a + b * 2`).

**Toán tử (operator)** là ký hiệu thực hiện phép toán.

Nhóm chính:

| Nhóm | Ví dụ | Kết quả tiêu biểu |
|------|-------|-------------------|
| Số học | `+ - * / %` | số |
| Gán | `= += -= *= /=` | gán vào biến |
| So sánh | `== != < > <= >=` | `bool` |
| Logic | `&& \|\| !` | `bool` (short-circuit) |
| Điều kiện | `cond ? a : b` | kiểu của nhánh |
| Bit (giới thiệu) | `& \| ^ ~ << >>` | số nguyên |
| Khác | `typeof`, `nameof`, `is` | thông tin kiểu / pattern |

**Short-circuit:** với `&&`, nếu vế trái `false` thì không đánh giá vế phải; với `||`, vế trái `true` thì bỏ vế phải.

## 4. Mô hình tư duy

```text
độ ưu tiên (cao → thấp, rút gọn):
  ++ --  →  * / %  →  + -  →  so sánh  →  &&  →  ||  →  ?:  →  =
```

Khi phân vân: **thêm ngoặc** `( )` — rõ hơn là nhớ thuộc lòng bảng ưu tiên.

## 5. Cú pháp (C# thật)

```csharp
int a = 10, b = 3;
int sum = a + b;
int rem = a % b;          // 1
a += 5;                   // a = a + 5

bool ok = a > 0 && b != 0;
int absLike = a >= 0 ? a : -a;

int x = 5;
Console.WriteLine(x++);   // in 5, sau đó x=6
Console.WriteLine(++x);   // x=7, in 7
```

## 6. Ví dụ

### Cơ bản

Chia nguyên vs chia thực:

```csharp
Console.WriteLine(7 / 2);     // 3 (int)
Console.WriteLine(7 / 2.0);   // 3.5 (double)
Console.WriteLine(7 % 2);     // 1
```

### Trung cấp

Kiểm tra năm nhuận (đủ điều kiện logic):

```csharp
Console.Write("Năm: ");
int year = int.Parse(Console.ReadLine()!);
bool leap = (year % 400 == 0) || (year % 4 == 0 && year % 100 != 0);
Console.WriteLine(leap ? "Năm nhuận" : "Không nhuận");
```

### Nâng cao

Máy tính 2 toán hạng với toán tử nhập từ chuỗi:

```csharp
Console.Write("a op b (ví dụ: 3 * 4): ");
string? line = Console.ReadLine();
if (line is null) return;

string[] parts = line.Split(' ', StringSplitOptions.RemoveEmptyEntries);
if (parts.Length != 3
    || !double.TryParse(parts[0], out double a)
    || !double.TryParse(parts[2], out double b))
{
    Console.WriteLine("Sai định dạng");
    return;
}

double? result = parts[1] switch
{
    "+" => a + b,
    "-" => a - b,
    "*" => a * b,
    "/" => b == 0 ? null : a / b,
    "%" => b == 0 ? null : a % b,
    _ => null
};

Console.WriteLine(result is null ? "Không tính được" : result.Value.ToString());
```

## 7. Lỗi thường gặp

| Lỗi | Nguyên nhân | Cách xử lý |
|-----|-------------|------------|
| `7/2 = 3` bất ngờ | Chia nguyên | Ép `7/2.0` hoặc `(double)7/2` |
| Nhầm `=` và `==` | Gán vs so sánh | Trong điều kiện luôn dùng `==` |
| `&` vs `&&` | Bit / không short-circuit vs logic | Dùng `&&` / `\|\|` cho điều kiện |
| Quên ngoặc | Ưu tiên toán tử | Viết `(a + b) * c` rõ ràng |
| Chia cho 0 | Runtime exception (số nguyên) | Kiểm tra mẫu số |

## 8. Gỡ lỗi

1. Tách biểu thức dài thành biến trung gian có tên nghĩa.
2. In từng phần: `Console.WriteLine($"a={a}, b={b}, rem={a%b}")`.
3. Với `&&` / `||`, đặt breakpoint xem nhánh nào bị bỏ qua (short-circuit).
4. Cẩn thận `++` trong biểu thức phức tạp — tách dòng cho dễ đọc.

## 9. Best practices

- Ưu tiên rõ ràng bằng ngoặc.
- Tránh side-effect (`++`) trong biểu thức dài.
- So sánh số thực: đừng dùng `==` trừ khi có lý do đặc biệt.
- Dùng `nameof(variable)` trong thông báo lỗi thay vì hard-code tên.

## 10. Bài tập

**Bài 1 — Chu vi & diện tích HCN**  
*Input:* chiều dài, chiều rộng (`double`).  
*Output:* chu vi `2*(d+r)`, diện tích `d*r`.

**Bài 2 — Chẵn/lẻ**  
*Input:* `int n`.  
*Output:* `Chẵn` hoặc `Lẻ` dùng `%`.

**Bài 3 — Max 3 số**  
*Input:* ba số nguyên.  
*Output:* số lớn nhất (dùng `?:` hoặc so sánh lồng, chưa cần `Math.Max` — hoặc dùng `Math.Max` nếu muốn).

**Bài 4 — Điểm trung bình có trọng số**  
*Input:* `toan`, `van`, `anh` (double).  
*Output:* `(toan*2 + van + anh) / 4`.

**Bài 5 — Safe divide**  
*Input:* `a`, `b` double.  
*Output:* `a/b` nếu `b != 0`, ngược lại in `Không chia được`.

## 11. Gợi ý

- Bài 3: `int max = a > b ? a : b; max = max > c ? max : c;`
- Bài 5: kiểm tra `Math.Abs(b) < 1e-12` nếu muốn chắc với số thực (tùy mức).

## 12. Đáp án

**Bài 1** — HCN:

```csharp
Console.Write("Dài: ");
double length = double.Parse(Console.ReadLine()!);
Console.Write("Rộng: ");
double width = double.Parse(Console.ReadLine()!);
Console.WriteLine($"Chu vi = {2 * (length + width)}");
Console.WriteLine($"Diện tích = {length * width}");
```

**Bài 2** — Chẵn lẻ:

```csharp
Console.Write("n = ");
int n = int.Parse(Console.ReadLine()!);
Console.WriteLine(n % 2 == 0 ? "Chẵn" : "Lẻ");
```

**Bài 3** — Max 3 số:

```csharp
Console.Write("a b c: ");
string[] p = Console.ReadLine()!.Split(' ', StringSplitOptions.RemoveEmptyEntries);
int a = int.Parse(p[0]), b = int.Parse(p[1]), c = int.Parse(p[2]);
int max = a > b ? a : b;
max = max > c ? max : c;
Console.WriteLine($"Max = {max}");
```

**Bài 4** — Trung bình trọng số:

```csharp
Console.Write("Toán Văn Anh: ");
var parts = Console.ReadLine()!.Split(' ', StringSplitOptions.RemoveEmptyEntries);
double toan = double.Parse(parts[0]);
double van = double.Parse(parts[1]);
double anh = double.Parse(parts[2]);
double avg = (toan * 2 + van + anh) / 4.0;
Console.WriteLine($"ĐTB = {avg:F2}");
```

**Bài 5** — Chia an toàn:

```csharp
Console.Write("a = ");
double a = double.Parse(Console.ReadLine()!);
Console.Write("b = ");
double b = double.Parse(Console.ReadLine()!);
if (b == 0) Console.WriteLine("Không chia được");
else Console.WriteLine(a / b);
```

## 13. Đáp án thay thế

Bài 3 dùng `Math.Max`:

```csharp
int max = Math.Max(a, Math.Max(b, c));
```

## 14. Thử thách

Nhập biểu thức dạng `n` và in `n!` (giai thừa) bằng vòng lặp — kết hợp toán tử `*=` (xem trước vòng lặp Chương 6 nếu cần).

## 15. Ứng dụng thực tế

Công thức giá, điều kiện khuyến mãi (`&&` nhiều rule), feature toggle, validation form — tất cả là biểu thức. Biểu thức tối/thiếu ngoặc là nguồn bug production cổ điển.

## 16. Liên hệ Unity

Trong gameplay: `velocity = dir * speed * Time.deltaTime`, điều kiện `if (hp <= 0 && !isDead)`. Short-circuit giúp tránh null reference (`player != null && player.IsAlive`). Cẩn thận `float` khi so sánh khoảng cách.

## 17. Kiểm tra kiến thức

1. `7/2` trong C# (kiểu `int`) ra gì?  
   **Đáp án:** `3`

2. `&&` khác `&` (dùng như logic) chỗ nào?  
   **Đáp án:** `&&` short-circuit; `&` luôn đánh giá cả hai vế (và còn là bit AND).

3. Toán tử `%` dùng để làm gì?  
   **Đáp án:** Lấy phần dư của phép chia.

4. `a += 2` nghĩa là gì?  
   **Đáp án:** `a = a + 2`

5. Vì sao nên dùng ngoặc trong biểu thức dài?  
   **Đáp án:** Tránh nhầm độ ưu tiên, code dễ đọc và đúng ý hơn.
