# Chương 3 — Lambda & Anonymous Methods

## 1. Mục tiêu học

- Viết **lambda expression** (`=>`) một biểu thức và khối lệnh
- Biết **anonymous method** (`delegate (...) { }`) legacy
- Suy luận kiểu tham số; `discards` `_`
- Chọn lambda vs method có tên khi nào

## 2. Điều kiện tiên quyết

- Chương 1–2: delegate, Action, Func
- Biểu thức và block C#

## 3. Khái niệm

**Lambda** là cú pháp ngắn tạo anonymous function gán được vào delegate/`Expression` (LINQ to Entities khác — Level 8).

```csharp
Func<int, int> square = x => x * x;
Func<int, int, int> add = (a, b) => a + b;
Action greet = () => Console.WriteLine("hi");
Action<int> block = x =>
{
    var y = x * 2;
    Console.WriteLine(y);
};
```

**Anonymous method** (C# 2):

```csharp
Action<string> a = delegate(string s) { Console.WriteLine(s); };
// Có thể bỏ danh sách tham số nếu không dùng:
EventHandler h = delegate { Console.WriteLine("fired"); };
```

Lambda hiện đại thay thế hầu hết anonymous method.

## 4. Mô hình tư duy

```text
(tham_số) => biểu_thức
(tham_số) => { các_lệnh; return ...; }

0 tham số:     () => ...
1 tham số:     x => ...   (ngoặc tùy chọn)
nhiều:         (a, b) => ...

Compiler suy ra kiểu từ biến/tham số đích (Action/Func/...).
```

## 5. Cú pháp

```csharp
// Expression-bodied
Predicate<int> p = n => n > 0;

// Statement lambda
Func<int, int> abs = n =>
{
    if (n < 0) return -n;
    return n;
};

// Kiểu tường minh khi cần
Func<object, string> toStr = (object o) => o.ToString() ?? "";

// Discard
Action<int, int> ignoreFirst = (_, y) => Console.WriteLine(y);

// Anonymous method
Func<int, int> twice = delegate(int x) { return x * 2; };
```

## 6. Ví dụ

### Cơ bản

```csharp
var nums = new List<int> { 1, 2, 3, 4 };
var evens = nums.FindAll(n => n % 2 == 0);
```

### Trung cấp

```csharp
static void SortByLength(List<string> list)
{
    list.Sort((a, b) => a.Length.CompareTo(b.Length));
}
```

### Nâng cao

Trả lambda từ factory (chuẩn bị closure):

```csharp
static Func<int, bool> GreaterThan(int threshold) =>
    value => value > threshold;

var gt10 = GreaterThan(10);
Console.WriteLine(gt10(11)); // True
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Không suy luận được kiểu | `var x = n => n;` | Khai báo `Func<...>` tường minh |
| Quên `return` trong block | Statement lambda | Thêm return |
| Side-effect trong lambda LINQ | Khó debug | Tách method / cẩn thận |
| Nhầm `=>` với expression-bodied member | Ngữ cảnh khác | Xem đang gán delegate hay định nghĩa method |

## 8. Gỡ lỗi

1. Đặt breakpoint trong statement lambda `{ }`.  
2. Expression lambda khó step — đổi tạm sang block.  
3. Xem biến đích kiểu gì khi “cannot convert lambda”.

## 9. Best practices

- Ngắn (1–3 dòng): lambda OK; dài: named method.
- Đặt tên biến Func rõ: `isEligible`, không `f`.
- Tránh capture quá nhiều state (xem chương 4).
- Public API quan trọng: method có tên dễ test/mock hơn lambda ẩn.

## 10. Bài tập

**Bài 1** — `Func<string, string>` lambda viết hoa (`ToUpperInvariant`).

**Bài 2** — `Action<int>` block: in `"n={n}"` và `"n^2={n*n}"`.

**Bài 3** — `list.FindAll` với lambda số chia hết 3.

**Bài 4** — Viết lại Bài 1 bằng anonymous method `delegate`.

## 11. Gợi ý

- Bài 1: `s => s.ToUpperInvariant()`.
- Bài 3: `n => n % 3 == 0`.
- Bài 4: `delegate(string s) { return s.ToUpperInvariant(); }`.

## 12. Đáp án

**Bài 1** — Lambda ToUpper:

```csharp
Func<string, string> up = s => s.ToUpperInvariant();
Console.WriteLine(up("csharp"));
```

**Bài 2** — Statement lambda:

```csharp
Action<int> dump = n =>
{
    Console.WriteLine($"n={n}");
    Console.WriteLine($"n^2={n * n}");
};
dump(5);
```

**Bài 3** — FindAll chia 3:

```csharp
var list = new List<int> { 1, 3, 4, 6, 9 };
var div3 = list.FindAll(n => n % 3 == 0);
Console.WriteLine(string.Join(",", div3)); // 3,6,9
```

**Bài 4** — Anonymous method tương đương:

```csharp
Func<string, string> up = delegate(string s)
{
    return s.ToUpperInvariant();
};
```

## 13. Đáp án thay thế

Local function C# thay lambda khi cần đệ quy/tên:

```csharp
int Fact(int n) => n <= 1 ? 1 : n * Fact(n - 1);
```

## 14. Thử thách

Viết `Compose(Func<A,B> f, Func<B,C> g)` trả `Func<A,C>` — `x => g(f(x))`.

## 15. Ứng dụng thực tế

- Cấu hình fluent API
- LINQ queries
- Event subscribe nhanh: `btn.Click += (s,e) => ...`
- Test: assert callback được gọi

## 16. Liên hệ Unity

- `button.onClick.AddListener(() => ...)` — UnityAction nhận lambda
- Lambda capture `this` MonoBehaviour — cẩn thận khi Destroy
- Prefer method có tên cho listener lâu dài để `RemoveListener` dễ
- Expression lambda không gỡ bằng `RemoveListener` cùng “chữ ký” nếu là instance khác — giữ reference

## 17. Kiểm tra kiến thức

1. Lambda một tham số có bắt buộc `(x)` không?  
   **Đáp án:** Không — `x =>` đủ.

2. `() =>` nghĩa là gì?  
   **Đáp án:** Không tham số.

3. Anonymous method dùng từ khóa nào?  
   **Đáp án:** `delegate`.

4. Khi nào cần kiểu tường minh cho tham số lambda?  
   **Đáp án:** Khi compiler không suy luận được / overload mơ hồ.

5. Statement lambda khác expression lambda chỗ nào?  
   **Đáp án:** Dùng khối `{ }` và thường có `return` tường minh.
