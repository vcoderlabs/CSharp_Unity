# Chương 5 — Dynamic & expression trees

## 1. Mục tiêu học

- Hiểu **`dynamic`**: liên kết muộn (runtime binding), khác `object`
- Biết rủi ro và khi nào *không* dùng dynamic
- Đọc **expression trees** (`Expression<Func<…>>`) — dữ liệu mô tả code
- Liên hệ `IQueryable` / LINQ provider dịch cây biểu thức

## 2. Điều kiện tiên quyết

- Level 7: `Func`, lambda
- Level 8 chương 5: `IEnumerable` vs `IQueryable`
- Chương 4: reflection (đối chiếu với dynamic)

## 3. Khái niệm

### dynamic

`dynamic` tắt kiểm tra kiểu lúc biên dịch cho thành viên gọi trên biến đó. Runtime binder (`Microsoft.CSharp`) resolve lúc chạy → sai tên method = runtime exception.

```csharp
dynamic d = "hello";
int n = d.Length; // OK runtime
// d.Foo(); // RuntimeBinderException
```

Khác `object`: với `object` bạn phải cast trước khi gọi thành viên.

### Expression trees

Lambda có thể compile thành **cây nút** (`Expression.Constant`, `.Property`, `.Call`, …) thay vì IL delegate — để provider **phân tích và dịch** (SQL, …).

```csharp
Func<int, bool> f = x => x > 0;              // delegate chạy được
Expression<Func<int, bool>> e = x => x > 0;  // cây — chưa chạy; có thể Compile()
```

## 4. Mô hình tư duy

```text
dynamic:  “tin runtime sẽ có member này”
          mất IntelliSense/an toàn biên dịch

Expression: “mô tả phép tính” → AI/ORM đọc cây
            e.Compile() → thành Func để chạy in-memory
```

## 5. Cú pháp

```csharp
using System.Linq.Expressions;

dynamic bag = new System.Dynamic.ExpandoObject();
bag.Name = "Ada";
Console.WriteLine(bag.Name);

Expression<Func<int, int>> twice = x => x * 2;
Func<int, int> fn = twice.Compile();
Console.WriteLine(fn(21)); // 42

// Xây cây thủ công
ParameterExpression p = Expression.Parameter(typeof(int), "x");
Expression body = Expression.Multiply(p, Expression.Constant(2));
var expr = Expression.Lambda<Func<int, int>>(body, p);
```

## 6. Ví dụ

### Cơ bản — dynamic

```csharp
static int GetLength(dynamic value) => value.Length;

Console.WriteLine(GetLength("abc"));
Console.WriteLine(GetLength(new int[5]));
// GetLength(3); // runtime fail — int không có Length
```

### Trung cấp — ExpandoObject / JSON-like

```csharp
using System.Dynamic;
using System.Text.Json;

dynamic obj = JsonSerializer.Deserialize<ExpandoObject>("""{"hp":100,"name":"Orc"}""")!;
// Tuỳ version/API — hoặc dùng JsonElement an toàn hơn dynamic

dynamic e = new ExpandoObject();
e.Hp = 100;
e.Name = "Orc";
Console.WriteLine($"{e.Name}:{e.Hp}");
```

### Nâng cao — expression cho filter tái sử dụng

```csharp
Expression<Func<Player, bool>> AtLeast(int level) => p => p.Level >= level;

var pred = AtLeast(10).Compile();
var players = new List<Player> { new("A", 5), new("B", 12) };
var filtered = players.Where(pred);

// IQueryable giả định:
// db.Players.Where(AtLeast(10)); // provider đọc Expression, không Compile ngay
record Player(string Name, int Level);
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| `RuntimeBinderException` | Member không tồn tại | Kiểm tra kiểu runtime / bỏ dynamic |
| Không gán lambda cho Expression | Có statement body `{ }` | Expression cần expression-bodied lambda |
| `Compile` mỗi lần nóng | Đắt | Cache delegate đã compile |
| Dùng dynamic thay generics | Mất an toàn | Prefer generic / interface |
| Nhầm `object` và `dynamic` | Cast vs runtime bind | Chọn đúng nhu cầu |

## 8. Gỡ lỗi

1. Dynamic fail: in `value.GetType()` trước khi gọi member.
2. Expression: debug `expr.Body.NodeType`, `ToString()` trên expression.
3. Không step vào binder dễ — unit test input đa kiểu.
4. So sánh: viết cùng logic bằng generic — thấy chỗ dynamic thừa.

## 9. Best practices

- `dynamic` chỉ interop COM/JSON ad-hoc/Iron* — không domain model chính.
- Prefer `JsonDocument`/`JsonElement` hoặc deserialize typed hơn dynamic JSON.
- Expression: để cho LINQ provider; tự viết cây khi làm rule engine.
- Cache `Compile()` results.
- Public API: tránh `dynamic` parameters nếu có thể.

## 10. Bài tập

**Bài 1** — Method nhận `dynamic`, trả `ToString()` của nó.

**Bài 2** — `Expression<Func<int,int>>` cộng 1; `Compile` và gọi.

**Bài 3** — Giải thích vì sao `Expression<Func<…>> e = x => { return x; };` lỗi.

**Bài 4** — Viết `Expression` thủ công: `x => x > 5` bằng API `Expression.*`.

## 11. Gợi ý

- Bài 1: `(string)value.ToString()` hoặc `$"{value}"` — cẩn thận null.
- Bài 3: expression tree không hỗ trợ statement block.
- Bài 4: `Parameter`, `Constant(5)`, `GreaterThan`, `Lambda`.

## 12. Đáp án

```csharp
static string AsText(dynamic d) => d is null ? "" : d.ToString();

Expression<Func<int, int>> inc = x => x + 1;
var f = inc.Compile();
Console.WriteLine(f(41));

// Bài 3: body phải là expression, không phải block statements.

ParameterExpression x = Expression.Parameter(typeof(int), "x");
var gt = Expression.GreaterThan(x, Expression.Constant(5));
var lambda = Expression.Lambda<Func<int, bool>>(gt, x);
Console.WriteLine(lambda.Compile()(6)); // True
```

## 13. Đáp án thay thế

Bài 1 dùng `object` + `ToString()` — thường đủ, không cần dynamic. Bài 4 dùng chỉ lambda `Expression<Func<int,bool>> e = x => x > 5`.

## 14. Thử thách

Mini rules: map tên property string + giá trị → xây `Expression<Func<T,bool>>` so sánh bằng reflection + Expression API; `Compile` và lọc list.

## 15. Ứng dụng thực tế

- EF Core / LINQ to SQL
- Serialization binders
- Scripting host nhẹ
- Interop Office/COM (hiếm hơn ngày nay)

## 16. Liên hệ Unity

- Tránh `dynamic` trong gameplay code
- Expression ít dùng runtime game — có thể dùng editor tooling
- Prefer typed ScriptableObject
- Reflection + cached delegate thay dynamic nếu cần plugin

## 17. Kiểm tra kiến thức

1. `dynamic` khác `object` chỗ gọi member?  
   **Đáp án:** Dynamic bind lúc runtime không cần cast; object cần cast/reflection.

2. `Expression<Func<T>>` dùng để làm gì điển hình?  
   **Đáp án:** Mô tả code cho provider phân tích (ví dụ dịch SQL), hoặc Compile chạy.

3. Làm sao chạy một expression tree in-memory?  
   **Đáp án:** `.Compile()` thành delegate rồi gọi.

4. Vì sao tránh dynamic trong API lớn?  
   **Đáp án:** Mất an toàn biên dịch, lỗi muộn, khó refactor.

5. Lambda block `{ return x; }` gán Expression được không?  
   **Đáp án:** Không — cần expression body.
