# Chương 1 — Pattern matching & switch expressions

## 1. Mục tiêu học

- Dùng **type pattern**, **declaration pattern**, **property pattern**, **relational / logical pattern**
- Viết **switch expression** thay if/switch statement dài
- Kết hợp `is` / `switch` với null-check và discard `_`
- Biết giới hạn và khi nào pattern giúp đọc code rõ hơn (.NET 8 / C# 12 thân thiện)

## 2. Điều kiện tiên quyết

- Level 1–2: if, switch statement, class/interface, inheritance
- Level 3: reference null cơ bản

## 3. Khái niệm

**Pattern matching** kiểm tra hình dạng dữ liệu và (tuỳ chọn) trích xuất giá trị trong một biểu thức.

| Pattern | Ví dụ ý niệm |
|---------|----------------|
| Type / declaration | `x is int i` |
| Constant | `x is null`, `x is 0` |
| Property | `p is { Name: "Ada", Level: > 10 }` |
| Relational | `x is > 0 and < 100` |
| List (C# 11+) | `arr is [1, 2, ..]` |
| var / discard | `x is var v`, `x is _` |

**Switch expression** trả về giá trị:

```csharp
string label = value switch
{
    1 => "one",
    2 => "two",
    _ => "other"
};
```

## 4. Mô hình tư duy

```text
Dữ liệu vào
    │
    ├─ khớp hình dạng? (kiểu, null, property, khoảng…)
    │       │
    │       └─ bind biến cục bộ (i, name, …)
    │
    └─ chọn nhánh → biểu thức kết quả (switch expression)
```

## 5. Cú pháp

```csharp
if (obj is string s)
    Console.WriteLine(s.Length);

if (obj is not null) { }

decimal discount = age switch
{
    < 12 => 0.5m,
    >= 12 and < 18 => 0.2m,
    >= 65 => 0.3m,
    _ => 0m
};

string Describe(object shape) => shape switch
{
    Circle { Radius: var r } => $"circle r={r}",
    Rectangle { Width: var w, Height: var h } => $"rect {w}x{h}",
    null => "none",
    _ => "unknown"
};
```

## 6. Ví dụ

### Cơ bản

```csharp
object box = 42;

if (box is int n)
    Console.WriteLine(n + 1);

string kind = box switch
{
    int => "number",
    string => "text",
    _ => "other"
};
```

### Trung cấp — property + relational

```csharp
record Player(string Name, int Level, bool IsVip);

string Tier(Player p) => p switch
{
    { Level: < 10 } => "Newbie",
    { Level: >= 10 and < 30, IsVip: false } => "Regular",
    { Level: >= 10, IsVip: true } => "VIP",
    { Level: >= 30 } => "Veteran",
    _ => "Unknown"
};
```

### Nâng cao — nested / list pattern

```csharp
int SumFirstTwo(int[] xs) => xs switch
{
    [] => 0,
    [var a] => a,
    [var a, var b, ..] => a + b,
};

string Route(object msg) => msg switch
{
    ("ping", var id) => $"pong {id}",
    ("login", string user, _) when user.Length > 0 => $"hi {user}",
    _ => "ignore"
};
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| CS8509 không cover hết | Switch expression thiếu nhánh | Thêm `_` hoặc đủ case |
| Pattern không khớp record | Sai tên property (phân biệt hoa thường) | Đúng tên property |
| `is int` với boxed null | `null` không phải int | Xử lý `null` riêng |
| Lạm dụng when phức tạp | Khó đọc | Tách method `bool IsEligible(...)` |
| Nhầm `switch` statement vs expression | Thiếu `=>` / có `break` | Expression dùng `=>`, không break |

## 8. Gỡ lỗi

1. Thêm case `_ => throw new Exception($"unhandled {value}")` tạm để bắt miss.
2. Đặt breakpoint trong từng nhánh switch expression (IDE hỗ trợ).
3. Với property pattern: in object trước khi match.
4. Cảnh báo exhaustiveness trên enum — bật và sửa hết.

## 9. Best practices

- Prefer switch expression khi mọi nhánh trả cùng kiểu kết quả.
- Đặt case cụ thể trước, `_` cuối.
- Property pattern thay chuỗi `if (p != null && p.Level > …)`.
- Đừng nhồi business dài trong nhánh — gọi hàm.
- Kết hợp `when` vừa phải; quá nhiều → khó maintain.

## 10. Bài tập

**Bài 1** — `Describe(object o)` trả `"null"` / `"int:N"` / `"str:S"` / `"other"`.

**Bài 2** — `Grade(int score)` bằng relational pattern → A/B/C/D/F.

**Bài 3** — Pattern trên `Player` VIP và Level ≥ 20 → `"elite"`.

**Bài 4** — List pattern: mảng 0/1/nhiều phần tử trả mô tả khác nhau.

## 11. Gợi ý

- Bài 1: `o switch { null =>, int n =>, string s =>, _ => }`.
- Bài 2: `>= 90`, `>= 80`, … nhớ thứ tự từ cao xuống.
- Bài 3: `{ IsVip: true, Level: >= 20 }`.
- Bài 4: `[]`, `[var x]`, `[_, _, ..]`.

## 12. Đáp án

```csharp
static string Describe(object? o) => o switch
{
    null => "null",
    int n => $"int:{n}",
    string s => $"str:{s}",
    _ => "other"
};

static char Grade(int score) => score switch
{
    >= 90 => 'A',
    >= 80 => 'B',
    >= 70 => 'C',
    >= 60 => 'D',
    _ => 'F'
};

static string Tag(Player p) => p is { IsVip: true, Level: >= 20 } ? "elite" : "normal";

static string Arr(int[] xs) => xs switch
{
    [] => "empty",
    [var a] => $"one:{a}",
    _ => $"many:{xs.Length}"
};
```

## 13. Đáp án thay thế

Bài 2 dùng switch statement cổ điển; bài 3 dùng `if (p is { ... })`. List pattern có thể thay bằng `xs.Length` if — pattern gọn hơn khi trích phần tử.

## 14. Thử thách

Viết interpreter tối giản cho AST: `Expr` = `Lit(int)`, `Add(Expr, Expr)`, `Neg(Expr)` — `Eval` bằng switch expression đệ quy.

## 15. Ứng dụng thực tế

- Mapping DTO / domain theo kiểu message
- HTTP result handling (`is Ok`, `is Error`)
- Rule engine nhỏ
- UI state machine gọn

## 16. Liên hệ Unity

- `switch (collision.gameObject.tag)`
- Pattern trên `Component` / interface ability
- Input action: match binding path
- Tránh allocate trong pattern nóng mỗi frame nếu tạo object tạm

## 17. Kiểm tra kiến thức

1. Switch expression khác switch statement chỗ nào?  
   **Đáp án:** Expression trả về giá trị; không dùng `break`, nhánh là biểu thức.

2. `x is int n` làm gì?  
   **Đáp án:** Kiểm tra kiểu int và gán vào biến `n` nếu đúng.

3. Property pattern viết thế nào?  
   **Đáp án:** `obj is { Prop: value/pattern }`.

4. Nhánh `_` nghĩa là gì?  
   **Đáp án:** Discard — khớp mọi trường hợp còn lại.

5. `>= 90 and < 100` là dạng pattern nào?  
   **Đáp án:** Relational kết hợp logical (`and`).
