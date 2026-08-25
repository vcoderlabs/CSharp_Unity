# Chương 2 — Records, tuples & deconstruction

## 1. Mục tiêu học

- Khai báo **`record`** / **`record class`** / **`record struct`**
- Dùng **positional record**, `with` expression, value-based equality
- Làm việc với **tuple** `(T1, T2, …)`, đặt tên, **deconstruction**
- Phân biệt record vs class vs struct vs tuple khi chọn mô hình dữ liệu

## 2. Điều kiện tiên quyết

- Level 2: class, property, equality ý niệm
- Level 3: value vs reference
- Chương 1: pattern trên property hữu ích với record

## 3. Khái niệm

### Record

`record` (mặc định reference type) sinh sẵn:

- Constructor positional + property `init`
- Deconstructor
- Equality theo **giá trị các property**
- `ToString` đẹp
- Hỗ trợ `with` để clone đổi vài field

`record struct` — value type có các tiện ích tương tự (cẩn thận copy).

### Tuple

`(int Id, string Name)` — kiểu giá trị nhẹ, không cần khai báo type riêng; tốt cho trả về tạm nhiều giá trị.

### Deconstruction

Tách thành phần: `var (id, name) = person;` — record positional và tuple hỗ trợ sẵn; class có thể viết `Deconstruct` method.

## 4. Mô hình tư duy

```text
DTO / sự kiện bất biến  → record (with để “sửa”)
Value nhỏ trên stack    → record struct hoặc struct
Trả về tạm 2–3 giá trị  → tuple
Entity có identity dài  → class (equality theo Id), không phải mọi thứ đều record
```

## 5. Cú pháp

```csharp
public record Player(string Name, int Level);

public record struct Point(int X, int Y);

public record Enemy(string Name, int Hp)
{
    public bool IsDead => Hp <= 0;
}

var p = new Player("Ada", 10);
var p2 = p with { Level = 11 };

(int x, int y) t = (3, 4);
var (a, b) = t;

void Deconstruct(out string name, out int level) // trên class thường
{
    name = Name; level = Level;
}
```

## 6. Ví dụ

### Cơ bản

```csharp
record Product(string Sku, decimal Price);

var a = new Product("SW-1", 100m);
var b = new Product("SW-1", 100m);
Console.WriteLine(a == b); // True — value equality
Console.WriteLine(a);      // Product { Sku = SW-1, Price = 100 }

var sale = a with { Price = 80m };
```

### Trung cấp — record struct + tuple return

```csharp
record struct Damage(int Physical, int Magical)
{
    public int Total => Physical + Magical;
}

static (bool Ok, string Error, Damage Dmg) TryParseAttack(string input)
{
    if (string.IsNullOrWhiteSpace(input))
        return (false, "empty", default);
    return (true, "", new Damage(10, 5));
}

var (ok, err, dmg) = TryParseAttack("x");
if (!ok) Console.WriteLine(err);
else Console.WriteLine(dmg.Total);
```

### Nâng cao — inheritance record + deconstruct nested

```csharp
abstract record Shape;
record Circle(double R) : Shape;
record Rect(double W, double H) : Shape;

static double Area(Shape s) => s switch
{
    Circle(var r) => Math.PI * r * r,
    Rect(var w, var h) => w * h,
    _ => 0
};

record LineItem(string Name, int Qty);
record Order(int Id, List<LineItem> Items);

var order = new Order(1, new() { new("A", 2), new("B", 1) });
var (id, items) = order;
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Đổi property record positional | Property là `{ get; init; }` | Dùng `with` tạo bản mới |
| Record equality bất ngờ | Chứa collection so reference | So sánh sâu thủ công / immutable list |
| Tuple không đặt tên mất nghĩa | `Item1`, `Item2` | Đặt tên `(int Id, string Name)` |
| `record struct` mutate nhầm copy | Value copy | `ref` / bất biến / class |
| Nhầm `record` vs `class` entity | Equality theo mọi field | Entity: class + Id |

## 8. Gỡ lỗi

1. In `==` và `ReferenceEquals` để phân biệt value equality vs cùng instance.
2. `with` không đổi bản gốc — assert `p.Level` cũ còn nguyên.
3. Deconstruct sai số phần tử → lỗi biên dịch — đếm lại.
4. Record kế thừa: kiểm tra `EqualityContract` / docs khi so sánh đa hình.

## 9. Best practices

- Positional record cho DTO ngắn; body record khi thêm logic.
- Prefer bất biến; `with` thay field public set.
- Tuple: tối đa ~3–4 phần tử có tên; lớn hơn → record.
- `record struct` cho điểm/toạ độ nhỏ.
- Không dùng record thay mọi domain entity có identity.

## 10. Bài tập

**Bài 1** — `record Book(string Title, string Author, int Year)`; tạo `with` đổi Year.

**Bài 2** — Chứng minh hai book cùng giá trị `==` true nhưng khác reference.

**Bài 3** — Method trả `(int Min, int Max)` từ mảng; deconstruct ở caller.

**Bài 4** — `record struct Vec2(float X, float Y)` + method `Length`.

## 11. Gợi ý

- Bài 2: `ReferenceEquals(a, b)` false khi `new` hai lần.
- Bài 3: duyệt min/max rồi `return (min, max)`.
- Bài 4: `MathF.Sqrt(X*X + Y*Y)`.

## 12. Đáp án

```csharp
record Book(string Title, string Author, int Year);

var b1 = new Book("Dune", "Herbert", 1965);
var b2 = b1 with { Year = 1965 };
var b3 = new Book("Dune", "Herbert", 1965);
Console.WriteLine(b1 == b3);                 // True
Console.WriteLine(ReferenceEquals(b1, b3));  // False

static (int Min, int Max) MinMax(int[] xs)
{
    int min = xs[0], max = xs[0];
    foreach (var n in xs)
    {
        if (n < min) min = n;
        if (n > max) max = n;
    }
    return (min, max);
}

record struct Vec2(float X, float Y)
{
    public float Length => MathF.Sqrt(X * X + Y * Y);
}
```

## 13. Đáp án thay thế

`class Book` + tự implement `Equals` — dài hơn record. Tuple không tên `(int, int)` vẫn chạy nhưng kém rõ. `readonly record struct` khi muốn siết immutability.

## 14. Thử thách

Thiết kế `record Result<T>(T? Value, string? Error)` với factory `Ok`/`Fail`; pattern match `if (result is { Error: null, Value: not null })`.

## 15. Ứng dụng thực tế

- API contracts / events
- Intermediate projection thay anonymous type dài hạn
- Return nhiều giá trị không out-param
- Domain events bất biến

## 16. Liên hệ Unity

- `record` cho data config (cẩn thận serialization Unity — thường vẫn class/`ScriptableObject`)
- `Vector` dùng `struct` có sẵn; `record struct` cho custom stats snapshot
- Tuple tạm trong editor tools
- Equality record hữu ích test logic thuần

## 17. Kiểm tra kiến thức

1. `with` làm gì trên record?  
   **Đáp án:** Tạo bản sao shallow với một số property được gán lại.

2. Hai record cùng dữ liệu: `==`?  
   **Đáp án:** Thường `true` (value-based equality).

3. Tuple khác record chỗ nào thực dụng?  
   **Đáp án:** Tuple nhẹ, không cần type riêng; record có tên kiểu, mở rộng member, rõ API hơn.

4. `record struct` là value hay reference?  
   **Đáp án:** Value type.

5. Deconstruction là gì?  
   **Đáp án:** Tách object/tuple thành các biến thành phần (`var (a,b) = …`).
