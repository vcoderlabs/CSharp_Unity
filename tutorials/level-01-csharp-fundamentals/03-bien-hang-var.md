# Chương 3 — Biến, hằng và `var`

## 1. Mục tiêu học

- Khai báo biến, gán giá trị, hiểu phạm vi tên
- Phân biệt `const`, biến thường, và (giới thiệu) `readonly` ở mức field
- Dùng `var` (type inference) đúng lúc, tránh lạm dụng

## 2. Điều kiện tiên quyết

- Đã tạo và chạy được project console (Chương 1–2)
- Biết khái niệm “ô nhớ mang tên” từ Level 0

## 3. Khái niệm

**Biến (variable)** là tên gắn với vùng nhớ chứa giá trị có thể thay đổi (trừ khi bạn đánh dấu không đổi).

**Khai báo** cho compiler biết tên + kiểu: `int age;`  
**Khởi tạo** gán giá trị lần đầu: `int age = 18;`

**Hằng `const`** phải biết giá trị lúc biên dịch; không thể gán lại. Ví dụ: `const double Pi = 3.14159;`

**`var`** không phải “kiểu động” như một số ngôn ngữ — compiler **suy luận kiểu** từ biểu thức bên phải tại compile-time. Sau khi suy luận, biến vẫn **tĩnh kiểu** (static typing).

**Phạm vi (scope):** biến cục bộ chỉ tồn tại trong khối `{ }` nơi nó được khai báo.

## 4. Mô hình tư duy

```text
const  → giá trị đóng băng từ lúc compile
biến   → hộp có nhãn + kiểu; có thể đổi nội dung
var x = biểu_thức;
         └── compiler điền kiểu thật (int, string, ...)
```

Sai lầm phổ biến: nghĩ `var` giống `dynamic` — **không**. `dynamic` bỏ kiểm tra kiểu lúc compile; `var` thì không.

## 5. Cú pháp (C# thật)

```csharp
int score = 0;
score = 10;

const int MaxLives = 3;
// MaxLives = 5; // lỗi biên dịch

var message = "Hello";      // string
var count = 42;             // int
var ratio = 3.14;           // double

// var bad;                 // lỗi: không có khởi tạo để suy luận
// var x = null;            // lỗi: không suy ra kiểu rõ
```

Nhiều biến cùng kiểu:

```csharp
int x = 1, y = 2, z = 3;
```

## 6. Ví dụ

### Cơ bản

Đổi chỗ hai số bằng biến tạm:

```csharp
int a = 5;
int b = 9;
Console.WriteLine($"Trước: a={a}, b={b}");

int temp = a;
a = b;
b = temp;

Console.WriteLine($"Sau: a={a}, b={b}");
```

### Trung cấp

Dùng `const` cho cấu hình đơn giản và `var` khi kiểu lộ rõ:

```csharp
const double TaxRate = 0.08;
var price = 100.0;           // double
var tax = price * TaxRate;   // double
var total = price + tax;

Console.WriteLine($"Tổng sau thuế: {total:F2}");
```

### Nâng cao

Minh họa scope — biến trong `if` không dùng được bên ngoài:

```csharp
int outer = 1;
if (outer > 0)
{
    var inner = outer * 2;
    Console.WriteLine(inner);
}
// Console.WriteLine(inner); // lỗi: inner không tồn tại ở đây

for (int i = 0; i < 3; i++)
{
    Console.Write(i + " ");
}
// Console.WriteLine(i); // lỗi: i thuộc scope for
```

## 7. Lỗi thường gặp

| Lỗi | Nguyên nhân | Cách xử lý |
|-----|-------------|------------|
| Use of unassigned local variable | Dùng biến chưa gán | Khởi tạo trước khi đọc |
| Cannot assign to const | Gán lại `const` | Dùng biến thường hoặc thiết kế lại |
| var requires initializer | `var x;` không hợp lệ | Viết `var x = ...` hoặc ghi rõ kiểu |
| Nhầm `var` với kiểu yếu | Muốn “mọi thứ đều object” | Học đúng static typing; dùng kiểu rõ khi cần đọc code |

## 8. Gỡ lỗi

1. Đưa chuột / dùng tip của IDE để xem kiểu thật của `var`.
2. Nếu “không gán”: theo mọi nhánh `if` — có nhánh nào bỏ quên khởi tạo không.
3. Đặt tên biến có nghĩa (`playerHealth` thay vì `x1`) để giảm bug logic.
4. Biên dịch thường xuyên (`dotnet build`) thay vì viết dài rồi mới build.

## 9. Best practices

- Khởi tạo khi khai báo nếu đã biết giá trị.
- `const` cho hằng số miền (giới hạn, tỉ lệ cố định).
- `var` khi kiểu **rõ từ vế phải** (`var list = new List<int>();`); viết rõ kiểu khi vế phải mơ hồ (`int id = GetId();` cũng tốt).
- Tên: camelCase cho biến cục bộ (`itemCount`), PascalCase cho `const` công khai thường thấy trong style guide (`MaxSpeed`).

## 10. Bài tập

**Bài 1 — BMI đơn giản**  
*Input:* chiều cao (m, double) và cân nặng (kg, double) từ bàn phím.  
*Output:* BMI = cân / (cao * cao), in 2 chữ số thập phân.

**Bài 2 — Đổi nhiệt độ**  
*Input:* một số Celsius.  
*Output:* Fahrenheit = C * 9/5 + 32. Dùng `const` cho 9, 5, 32 nếu hợp lý (hoặc const cho công thức từng phần).

**Bài 3 — Đếm ký tự tên**  
*Input:* một chuỗi tên.  
*Output:* in độ dài sau `Trim()`; dùng `var` cho biến chuỗi.

**Bài 4 — Scope**  
*Input:* số nguyên `n`.  
*Output:* nếu `n > 0` tạo biến `message = "dương"` và in; nếu không, in `"không dương"`. Không được để biến `message` “rò” scope sai gây lỗi — tự thiết kế đúng.

**Bài 5 — Hoán vị không dùng `temp`? (thử)**  
*Input:* hai số nguyên `a`, `b`.  
*Output:* đổi chỗ và in. Có thể dùng `temp` (an toàn) hoặc thủ thuật cộng trừ (cẩn thận tràn số).

## 11. Gợi ý

- Đọc số: `double.TryParse(Console.ReadLine(), out var h)` (pattern hay gặp).
- BMI: kiểm tra chiều cao > 0 tránh chia 0.
- Bài 4: khai báo `string message` ngoài `if`, hoặc in trực tiếp trong từng nhánh.

## 12. Đáp án

**Bài 1** — Tính BMI:

```csharp
Console.Write("Chiều cao (m): ");
double height = double.Parse(Console.ReadLine()!);
Console.Write("Cân nặng (kg): ");
double weight = double.Parse(Console.ReadLine()!);

if (height <= 0)
{
    Console.WriteLine("Chiều cao không hợp lệ.");
    return;
}

double bmi = weight / (height * height);
Console.WriteLine($"BMI = {bmi:F2}");
```

**Bài 2** — Celsius → Fahrenheit:

```csharp
const double Factor = 9.0 / 5.0;
const double Offset = 32.0;

Console.Write("Celsius: ");
var c = double.Parse(Console.ReadLine()!);
var f = c * Factor + Offset;
Console.WriteLine($"Fahrenheit = {f:F2}");
```

**Bài 3** — Độ dài tên:

```csharp
Console.Write("Tên: ");
var name = Console.ReadLine() ?? "";
var cleaned = name.Trim();
Console.WriteLine($"Độ dài: {cleaned.Length}");
```

**Bài 4** — Message theo dấu:

```csharp
Console.Write("n = ");
int n = int.Parse(Console.ReadLine()!);

if (n > 0)
{
    var message = "dương";
    Console.WriteLine(message);
}
else
{
    Console.WriteLine("không dương");
}
```

**Bài 5** — Hoán vị với `temp`:

```csharp
Console.Write("a = ");
int a = int.Parse(Console.ReadLine()!);
Console.Write("b = ");
int b = int.Parse(Console.ReadLine()!);

int temp = a;
a = b;
b = temp;
Console.WriteLine($"a={a}, b={b}");
```

## 13. Đáp án thay thế

Bài 1 dùng `TryParse` an toàn hơn:

```csharp
Console.Write("Chiều cao (m): ");
if (!double.TryParse(Console.ReadLine(), out double height) || height <= 0)
{
    Console.WriteLine("Input không hợp lệ.");
    return;
}

Console.Write("Cân nặng (kg): ");
if (!double.TryParse(Console.ReadLine(), out double weight))
{
    Console.WriteLine("Input không hợp lệ.");
    return;
}

Console.WriteLine($"BMI = {weight / (height * height):F2}");
```

## 14. Thử thách

Viết chương trình nhập 3 số thực, in min và max **chỉ dùng biến** (chưa cần mảng). So sánh từng cặp.

## 15. Ứng dụng thực tế

Biến và hằng xuất hiện mọi nơi: cấu hình thuế, giới hạn retry API, điểm người chơi. Đặt tên rõ và `const` đúng chỗ giảm magic number trong production.

## 16. Liên hệ Unity

Trong Unity bạn thường thấy:

```csharp
[SerializeField] float moveSpeed = 5f;
const int MaxInventorySlots = 30;
```

`var` dùng được trong method Unity; field trên MonoBehaviour thường ghi rõ kiểu để Inspector hiện đúng. Tránh `const` cho giá trị cần chỉnh trong Inspector — dùng field thường / `[SerializeField]`.

## 17. Kiểm tra kiến thức

1. `var` có làm C# thành dynamic typing không?  
   **Đáp án:** Không — kiểu được suy luận lúc compile và cố định.

2. Khi nào dùng `const`?  
   **Đáp án:** Khi giá trị biết trước lúc biên dịch và không bao giờ đổi.

3. Vì sao `var x;` lỗi?  
   **Đáp án:** Không có biểu thức khởi tạo để suy luận kiểu.

4. Biến khai báo trong `for (...)` dùng được sau vòng lặp không?  
   **Đáp án:** Không — hết scope của `for`.

5. `const` khác biến thường chỗ nào?  
   **Đáp án:** Không gán lại được; giá trị phải là compile-time constant.
