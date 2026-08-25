# Chương 7 — Methods: tham số, return, overloading, scope

## 1. Mục tiêu học

- Viết method có tham số và giá trị trả về
- Hiểu overloading (nạp chồng) và chọn đúng chữ ký
- Phân biệt scope biến cục bộ / tham số / field (giới thiệu)
- Tách logic lặp lại thành method rõ trách nhiệm

## 2. Điều kiện tiên quyết

- Thành thạo biến, kiểu, điều khiển luồng (Chương 3–6)
- Biết tạo project console nhiều file (Chương 2)

## 3. Khái niệm

**Method** là khối lệnh có tên, có thể nhận **tham số (parameters)** và **trả về (return)** một giá trị (hoặc `void` nếu không trả).

**Chữ ký (signature)** gồm tên method + danh sách kiểu tham số — dùng để phân biệt overload.

**Overloading:** nhiều method cùng tên nhưng khác tham số (số lượng / kiểu). Không chỉ khác kiểu trả về.

**Scope:**

- Tham số và biến cục bộ chỉ sống trong method.
- Biến cục bộ che (shadow) tên bên ngoài nếu trùng — nên tránh.

Giới thiệu nhẹ: `ref` / `out` sẽ học sâu ở Level 3; ở Level 1 ưu tiên return giá trị rõ ràng.

## 4. Mô hình tư duy

```text
Input  →  [ Method: làm một việc ]  →  Output
           tên rõ + tham số tối thiểu

Overload giống "cùng dyn năng, nhiều cách gọi":
  Add(int,int)
  Add(double,double)
  Add(int,int,int)
```

Một method nên làm **một việc** — tên là động từ: `CalculateTax`, `PrintBanner`, `TryReadInt`.

## 5. Cú pháp (C# thật)

```csharp
static int Add(int a, int b)
{
    return a + b;
}

static void PrintSum(int a, int b)
{
    Console.WriteLine(Add(a, b));
}

// Expression-bodied
static int Square(int x) => x * x;

// Optional parameter
static void Greet(string name, string prefix = "Xin chào")
    => Console.WriteLine($"{prefix}, {name}!");

// Overloads
static double Area(double radius) => Math.PI * radius * radius;
static double Area(double width, double height) => width * height;
```

Với top-level statements, method cục bộ / `static` local function đặt sau câu lệnh top-level, hoặc dùng class `Program` tường minh.

## 6. Ví dụ

### Cơ bản

Method cộng và gọi từ điểm vào:

```csharp
static int Add(int a, int b) => a + b;

Console.WriteLine(Add(2, 3));
```

### Trung cấp

Đọc số an toàn — trả `bool` + `out` (làm quen nhẹ):

```csharp
static bool TryReadInt(string prompt, out int value)
{
    Console.Write(prompt);
    return int.TryParse(Console.ReadLine(), out value);
}

if (!TryReadInt("a = ", out int a) || !TryReadInt("b = ", out int b))
{
    Console.WriteLine("Input sai");
    return;
}
Console.WriteLine(a + b);
```

### Nâng cao

Overload tính diện tích + method in kết quả có format:

```csharp
static double AreaCircle(double r) => Math.PI * r * r;
static double AreaRect(double w, double h) => w * h;

static void PrintArea(string label, double area)
    => Console.WriteLine($"{label}: {area:F2}");

PrintArea("Hình tròn r=2", AreaCircle(2));
PrintArea("HCN 3x4", AreaRect(3, 4));
```

Hoặc cùng tên `Area` overload nếu đặt trong class:

```csharp
static class Geo
{
    public static double Area(double r) => Math.PI * r * r;
    public static double Area(double w, double h) => w * h;
}
```

## 7. Lỗi thường gặp

| Lỗi | Nguyên nhân | Cách xử lý |
|-----|-------------|------------|
| `not all code paths return a value` | Có nhánh không `return` | Đảm bảo mọi đường đi trả giá trị |
| Overload chỉ khác return type | Không hợp lệ | Đổi tham số hoặc đổi tên |
| Gọi method instance như static | Thiếu object | Thêm `static` hoặc `new` rồi gọi |
| Side-effect khó đoán | Method vừa tính vừa in lung tung | Tách `Compute` và `Print` |
| Quên truyền đủ tham số | Sai chữ ký | Đọc gợi ý IDE / lỗi compile |

## 8. Gỡ lỗi

1. Đặt breakpoint ở dòng đầu method — xem giá trị tham số (Locals).
2. Tạm thêm `Console.WriteLine` vào đầu/cuối method để xác nhận được gọi.
3. Nếu kết quả sai: unit-test tay bằng vài cặp input đã biết đáp án.
4. Với overload: IDE sẽ hiện danh sách chữ ký — chọn đúng.

## 9. Best practices

- Tên động từ, rõ nghĩa; tránh `DoStuff`, `Handle1`.
- Method ngắn (< ~20–30 dòng khi mới học); dài thì tách.
- Prefer `return` sớm cho validation (`guard clauses`).
- Không lạm dụng optional parameters đến mức khó đọc — overload có thể rõ hơn.

## 10. Bài tập

**Bài 1 — `Max`**  
*Input:* hai số nguyên từ bàn phím.  
*Output:* in kết quả `Max(a,b)` do bạn viết (không bắt buộc cấm `Math.Max`, nhưng hãy tự viết để luyện).

**Bài 2 — `IsPrime`**  
*Input:* `n`.  
*Output:* `true`/`false` (in `Nguyên tố` / `Không`). Method `bool IsPrime(int n)`.

**Bài 3 — Overload `Repeat`**  
*Input:* chuỗi và số lần; thêm overload `Repeat(string s)` mặc định 2 lần.  
*Output:* chuỗi lặp nối nhau.

**Bài 4 — `Average`**  
*Input:* ba `double`.  
*Output:* trung bình qua `double Average(double a, double b, double c)`.

**Bài 5 — Máy tính mini bằng method**  
*Input:* `a`, `op`, `b`.  
*Output:* gọi `Calculate(a, op, b)` trả `double?` (`null` nếu lỗi). In kết quả hoặc thông báo lỗi.

## 11. Gợi ý

- `IsPrime`: `n < 2` → false; vòng tới `sqrt(n)`.
- `Repeat`: vòng `for` nối `StringBuilder` hoặc `string` (Chương 9 sẽ tối ưu).
- Guard clause: kiểm tra `op` hợp lệ trước khi tính.

## 12. Đáp án

**Bài 1** — Max:

```csharp
static int Max(int a, int b) => a > b ? a : b;

Console.Write("a b: ");
var p = Console.ReadLine()!.Split(' ', StringSplitOptions.RemoveEmptyEntries);
int a = int.Parse(p[0]), b = int.Parse(p[1]);
Console.WriteLine(Max(a, b));
```

**Bài 2** — IsPrime:

```csharp
static bool IsPrime(int n)
{
    if (n < 2) return false;
    for (int d = 2; d * d <= n; d++)
    {
        if (n % d == 0) return false;
    }
    return true;
}

Console.Write("n = ");
int n = int.Parse(Console.ReadLine()!);
Console.WriteLine(IsPrime(n) ? "Nguyên tố" : "Không");
```

**Bài 3** — Repeat overload:

```csharp
static string Repeat(string s, int times)
{
    string result = "";
    for (int i = 0; i < times; i++) result += s;
    return result;
}

static string Repeat(string s) => Repeat(s, 2);

Console.WriteLine(Repeat("ha", 3)); // hahaha
Console.WriteLine(Repeat("hi"));    // hihi
```

**Bài 4** — Average:

```csharp
static double Average(double a, double b, double c) => (a + b + c) / 3.0;

Console.WriteLine(Average(1, 2, 3)); // 2
```

**Bài 5** — Calculate:

```csharp
static double? Calculate(double a, string op, double b)
{
    return op switch
    {
        "+" => a + b,
        "-" => a - b,
        "*" => a * b,
        "/" when b != 0 => a / b,
        _ => null
    };
}

Console.Write("a op b: ");
var parts = Console.ReadLine()!.Split(' ', StringSplitOptions.RemoveEmptyEntries);
double x = double.Parse(parts[0]);
string op = parts[1];
double y = double.Parse(parts[2]);
double? r = Calculate(x, op, y);
Console.WriteLine(r is null ? "Lỗi" : r.Value.ToString());
```

## 13. Đáp án thay thế

`Repeat` dùng `string.Concat(Enumerable.Repeat(s, times))` (cần LINQ) hoặc `StringBuilder` (Chương 9).

Local function trong top-level:

```csharp
int Max(int a, int b) => a > b ? a : b;
Console.WriteLine(Max(1, 2));
```

## 14. Thử thách

Viết bộ method: `ReadInt`, `ReadDouble`, `ReadOp` cho calculator; `Main` chỉ còn điều phối. Sau đó thêm overload `Calculate` cho `int` và `double`.

## 15. Ứng dụng thực tế

Service layer, helper validation, mapping DTO — production code là tập hợp method có trách nhiệm rõ. Overload giúp API dễ dùng (`Write` nhiều kiểu tham số).

## 16. Liên hệ Unity

`void Update()`, `void Start()` là method được engine gọi. Bạn tự viết `TakeDamage(int amount)`, overload `Spawn(Vector3)` / `Spawn(Vector3, Quaternion)`. Tránh method quá dài trong `Update` — tách `HandleInput()`, `Move()`, `Animate()`.

## 17. Kiểm tra kiến thức

1. Method `void` nghĩa là gì?  
   **Đáp án:** Không trả giá trị qua `return` (có thể `return;` để thoát sớm).

2. Overload hợp lệ khi nào?  
   **Đáp án:** Cùng tên nhưng khác danh sách tham số (kiểu/số lượng/thứ tự kiểu).

3. Chỉ khác kiểu trả về có overload được không?  
   **Đáp án:** Không.

4. Biến cục bộ trong method A dùng được ở method B không?  
   **Đáp án:** Không — khác scope; phải truyền tham số hoặc dùng field/shared state.

5. Optional parameter giúp gì?  
   **Đáp án:** Cho phép gọi method bỏ qua tham số cuối, dùng giá trị mặc định.
