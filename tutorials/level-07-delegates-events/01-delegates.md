# Chương 1 — Delegates

## 1. Mục tiêu học

- Hiểu delegate là **kiểu tham chiếu tới method** có chữ ký cố định
- Khai báo, gán method group, `Invoke` / gọi trực tiếp
- Phân biệt delegate instance với method tĩnh/instance
- Thấy vai trò callback trước khi học event

## 2. Điều kiện tiên quyết

- Level 2: method, static vs instance
- Level 5: quen với generic (chuẩn bị Action/Func)
- .NET 8+

## 3. Khái niệm

**Delegate** = hợp đồng: “method nào nhận tham số thế này, trả về thế kia, đều gán được vào biến kiểu delegate”.

```csharp
delegate int Combiner(int a, int b);

static int Add(int x, int y) => x + y;
Combiner op = Add;
int result = op(2, 3); // 5
```

Dùng cho: callback, plugin strategy, sự kiện (nền của event), LINQ (bên dưới là Func).

Trước C# hiện đại người ta tự `delegate void MyHandler(...)`. Nay ưu tiên `Action`/`Func` — nhưng hiểu delegate gốc vẫn bắt buộc (Unity docs, legacy, multicast).

## 4. Mô hình tư duy

```text
Định nghĩa kiểu:  delegate void HitHandler(int damage);
Biến:             HitHandler onHit;
Gán:              onHit = TakeDamage;   // method group
Gọi:              onHit?.Invoke(10);  // hoặc onHit(10)

Giống “ổ cắm”: chỉ cắm được phích đúng hình (chữ ký).
Null = chưa có ai lắng nghe / chưa gán.
```

## 5. Cú pháp

```csharp
// Khai báo kiểu delegate
public delegate void Notify(string message);

// Gán
Notify n = Console.WriteLine; // method group khớp chữ ký
n("hello");

// Instance method
var greeter = new Greeter();
Notify n2 = greeter.Say;

// Đổi target
n = OtherMethod;

// Null-safe
n?.Invoke("ping");
```

Generic delegate tự định nghĩa:

```csharp
public delegate TResult Transformer<T, TResult>(T input);
```

## 6. Ví dụ

### Cơ bản

```csharp
delegate int Op(int a, int b);

static int Mul(int a, int b) => a * b;

Op op = Mul;
Console.WriteLine(op(4, 5)); // 20
```

### Trung cấp

Callback sau khi tải xong:

```csharp
delegate void DoneCallback(bool success);

static void Load(string path, DoneCallback done)
{
    bool ok = File.Exists(path);
    done(ok);
}

Load("a.txt", success => Console.WriteLine(success ? "OK" : "FAIL"));
```

(Lambda xem chương 3; ở đây có thể dùng method riêng thay lambda.)

### Nâng cao

Strategy pattern:

```csharp
delegate decimal PriceRule(decimal basePrice);

static decimal Vat(decimal p) => p * 1.1m;
static decimal Discount(decimal p) => p * 0.9m;

static decimal Checkout(decimal price, PriceRule rule) => rule(price);

Console.WriteLine(Checkout(100, Vat));
Console.WriteLine(Checkout(100, Discount));
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| CS0123 chữ ký không khớp | Sai tham số/return | Sửa method hoặc delegate |
| `NullReferenceException` khi gọi | Delegate null | `?.Invoke` |
| Nhầm tên kiểu delegate với biến | Đặt tên giống nhau | `XxxHandler` cho type |
| Gán lambda vào delegate cũ khó đọc | Style | Chuyển Action/Func (chương 2) |

## 8. Gỡ lỗi

1. Hover biến delegate trong IDE: xem target method.  
2. `op.Method.Name` / `op.Target` lúc debug.  
3. Breakpoint trong method được gán — xác nhận có được gọi.

## 9. Best practices

- Ưu tiên `Action`/`Func` trừ khi thư viện yêu cầu delegate đặt tên.
- Luôn cân nhắc null trước khi gọi.
- Đặt tên rõ vai trò: `CalculationHandler`, không `MyDel`.
- Method group rõ hơn anonymous khi tái sử dụng nhiều nơi.

## 10. Bài tập

**Bài 1** — `delegate int BinaryOp(int a, int b);` gán Add và Sub, in kết quả.

**Bài 2** — `delegate bool NumberFilter(int n);` method `IsEven`; lọc mảng in số chẵn (vòng for thủ công).

**Bài 3** — Method `RunTwice(Notify n)` gọi callback 2 lần.

**Bài 4** — `Transformer<string, int>` đếm độ dài chuỗi.

## 11. Gợi ý

- Bài 2: `if (filter(x)) Console.WriteLine(x);`.
- Bài 3: `n?.Invoke("1"); n?.Invoke("2");`.
- Bài 4: `delegate TOut Transformer<TIn, TOut>(TIn input);`.

## 12. Đáp án

**Bài 1** — Hai phép toán qua delegate:

```csharp
delegate int BinaryOp(int a, int b);

static int Add(int a, int b) => a + b;
static int Sub(int a, int b) => a - b;

BinaryOp op = Add;
Console.WriteLine(op(10, 3)); // 13
op = Sub;
Console.WriteLine(op(10, 3)); // 7
```

**Bài 2** — Filter mảng bằng delegate:

```csharp
delegate bool NumberFilter(int n);
static bool IsEven(int n) => n % 2 == 0;

int[] data = { 1, 2, 3, 4, 5 };
NumberFilter filter = IsEven;
foreach (int n in data)
    if (filter(n)) Console.WriteLine(n);
```

**Bài 3** — Gọi callback hai lần:

```csharp
delegate void Notify(string message);

static void RunTwice(Notify n)
{
    n?.Invoke("lần 1");
    n?.Invoke("lần 2");
}

RunTwice(Console.WriteLine);
```

**Bài 4** — Transformer generic:

```csharp
delegate TOut Transformer<TIn, TOut>(TIn input);

static int Length(string s) => s.Length;

Transformer<string, int> t = Length;
Console.WriteLine(t("hello")); // 5
```

## 13. Đáp án thay thế

Bài 2 dùng `List<int>.FindAll` với `Predicate<int>` (chương 2). Bài 4 dùng sẵn `Func<string, int>`.

## 14. Thử thách

Viết `Pipe`: nhận `int`, apply lần lượt mảng `Transformer<int, int>`, trả kết quả cuối — không dùng LINQ.

## 15. Ứng dụng thực tế

- Plugin: đăng ký hàm xử lý message
- Retry policy: truyền hàm “hành động” vào helper
- Sort: `Comparison<T>` là delegate
- Middleware pipeline tối giản

## 16. Liên hệ Unity

- Callback C#: `System.Action` trên script = nền tảng trước UnityEvent
- Nhiều API Unity nhận `UnityAction` (gần Action)
- Hiểu delegate giúp đọc `delegate void UnityAction()` trong docs
- Tránh giữ delegate tới object đã Destroy — memory/leak logic

## 17. Kiểm tra kiến thức

1. Delegate mô tả điều gì?  
   **Đáp án:** Chữ ký method (tham số + kiểu trả về) có thể gán và gọi gián tiếp.

2. Method group là gì?  
   **Đáp án:** Tên method không ngoặc — compiler tạo delegate trỏ tới method đó.

3. Gọi an toàn khi có thể null?  
   **Đáp án:** `handler?.Invoke(...)`.

4. Instance method gán vào delegate cần gì?  
   **Đáp án:** Instance cụ thể (`obj.Method`) — Target là object đó.

5. Vì sao vẫn học delegate khi đã có Action/Func?  
   **Đáp án:** Nền tảng event/multicast; gặp trong API/tài liệu; hiểu lỗi chữ ký.
