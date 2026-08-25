# Chương 3 — Exception Hierarchy

## 1. Mục tiêu học

- Hiểu cây kế thừa exception trong .NET
- Tổ chức hierarchy domain (base → nhóm → cụ thể)
- Bắt exception đúng tầng (polymorphism với catch)
- Tránh bắt quá rộng hoặc quá hẹp sai chỗ

## 2. Điều kiện tiên quyết

- Chương 1–2: try/catch, custom exception
- Level 2: inheritance, `is` / polymorphism

## 3. Khái niệm

Mọi exception đều đi từ `System.Exception`. Một số nhánh quan trọng:

```text
Exception
├── SystemException (nhiều lỗi runtime)
│   ├── ArgumentException
│   │   ├── ArgumentNullException
│   │   └── ArgumentOutOfRangeException
│   ├── InvalidOperationException
│   ├── NullReferenceException
│   ├── IndexOutOfRangeException
│   └── IOException
│       ├── FileNotFoundException
│       └── DirectoryNotFoundException
└── (Application / custom của bạn)
```

**Hierarchy của bạn** ví dụ:

```text
AppException
├── DomainException
│   ├── OrderException
│   │   ├── OrderNotFoundException
│   │   └── OrderAlreadyPaidException
│   └── InventoryException
└── InfrastructureException
    ├── DatabaseException
    └── FileStorageException
```

`catch (OrderException)` sẽ bắt mọi con của `OrderException`.

## 4. Mô hình tư duy

```text
Caller gần UI/API:
  catch cụ thể → thông báo/UX riêng
  catch nhóm (OrderException) → xử lý chung domain
  catch AppException → log + lỗi tổng quát
  không bắt Exception trừ biên hệ thống

Ném: càng cụ thể càng tốt (OrderNotFoundException)
Bắt: đủ cụ thể để xử lý đúng, đủ tổng quát để không lặp code
```

## 5. Cú pháp

```csharp
public class AppException : Exception
{
    public AppException(string message) : base(message) { }
    public AppException(string message, Exception inner) : base(message, inner) { }
}

public class DomainException : AppException
{
    public DomainException(string message) : base(message) { }
}

public class OrderNotFoundException : DomainException
{
    public string OrderId { get; }
    public OrderNotFoundException(string orderId)
        : base($"Order '{orderId}' not found.")
        => OrderId = orderId;
}

try { PlaceOrder(id); }
catch (OrderNotFoundException ex) { /* UX: 404 */ }
catch (DomainException ex) { /* UX: lỗi nghiệp vụ */ }
catch (AppException ex) { /* log + generic */ }
```

Thứ tự `catch`: **con trước, cha sau**.

## 6. Ví dụ

### Cơ bản

```csharp
try
{
    throw new FileNotFoundException("missing", "a.txt");
}
catch (FileNotFoundException)
{
    Console.WriteLine("File cụ thể");
}
catch (IOException)
{
    Console.WriteLine("IO chung");
}
```

### Trung cấp

```csharp
public class InventoryException : DomainException
{
    public InventoryException(string message) : base(message) { }
}

public class ItemNotFoundException : InventoryException
{
    public string ItemId { get; }
    public ItemNotFoundException(string itemId)
        : base($"Item '{itemId}' không tồn tại.")
        => ItemId = itemId;
}

public class InventoryFullException : InventoryException
{
    public InventoryFullException() : base("Túi đã đầy.") { }
}
```

### Nâng cao

Layer biên dịch exception hạ tầng → domain:

```csharp
static Order GetOrder(string id)
{
    try
    {
        return _db.FindOrder(id) ?? throw new OrderNotFoundException(id);
    }
    catch (TimeoutException ex)
    {
        throw new DatabaseException("Timeout khi lấy order.", ex);
    }
}
```

`DatabaseException : InfrastructureException : AppException`.

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| CS0160: catch cha trước con | Thứ tự sai | Đảo: cụ thể → tổng quát |
| Mọi thứ đều `: Exception` phẳng | Không nhóm được | Thêm lớp trung gian Domain/Infra |
| Catch `Exception` ở mọi method | Che lỗi lập trình | Chỉ ở biên (Main, middleware) |
| Hierarchy quá sâu (6–7 tầng) | Phức tạp vô ích | 2–3 tầng thường đủ |

## 8. Gỡ lỗi

1. In `ex.GetType().FullName` để xem đúng lớp nào được ném.
2. Vẽ sơ đồ inheritance khi thêm exception mới.
3. Test: ném con, đảm bảo `catch (Cha)` vẫn bắt được nếu đó là ý đồ.

## 9. Best practices

- Base domain exception cho phép `catch (DomainException)` một lần.
- Tách Infrastructure vs Domain — UI có thể map khác nhau.
- Không bắt `NullReferenceException` / `StackOverflowException` như luồng bình thường — fix code.
- Document public API: “ném những gì”.
- Prefer sealed leaf exceptions nếu không mở rộng thêm.

## 10. Bài tập

**Bài 1** — Tạo `AppException` → `PaymentException` → `CardDeclinedException`. Ném cái cuối, bắt bằng `PaymentException`.

**Bài 2** — Thêm `InsufficientFundsException : PaymentException`. Method `Pay` ném một trong hai; Main có 3 catch: cụ thể từng loại + `PaymentException`.

**Bài 3** — Giải thích (viết comment) vì sao `catch (Exception)` rồi `catch (IOException)` không hợp lệ / vô nghĩa.

**Bài 4** — Map: khi `FileNotFoundException`, ném `SaveSlotMissingException : InfrastructureException`.

## 11. Gợi ý

- Bài 1: `catch (PaymentException ex)` in `ex.GetType().Name`.
- Bài 2: thứ tự catch: CardDeclined → InsufficientFunds → Payment.
- Bài 4: wrap trong catch FileNotFound.

## 12. Đáp án

**Bài 1** — Hierarchy 3 tầng + bắt nhóm:

```csharp
public class AppException : Exception
{
    public AppException(string message) : base(message) { }
}

public class PaymentException : AppException
{
    public PaymentException(string message) : base(message) { }
}

public class CardDeclinedException : PaymentException
{
    public CardDeclinedException() : base("Thẻ bị từ chối.") { }
}

try { throw new CardDeclinedException(); }
catch (PaymentException ex)
{
    Console.WriteLine($"Bắt nhóm: {ex.GetType().Name}");
}
```

**Bài 2** — Nhiều catch theo tầng:

```csharp
public class InsufficientFundsException : PaymentException
{
    public InsufficientFundsException() : base("Không đủ tiền.") { }
}

static void Pay(string mode)
{
    if (mode == "declined") throw new CardDeclinedException();
    if (mode == "funds") throw new InsufficientFundsException();
    throw new PaymentException("Lỗi thanh toán chung.");
}

try { Pay(args[0]); }
catch (CardDeclinedException) { Console.WriteLine("Thử thẻ khác"); }
catch (InsufficientFundsException) { Console.WriteLine("Nạp thêm"); }
catch (PaymentException ex) { Console.WriteLine(ex.Message); }
```

**Bài 3** — Comment giải thích:

```csharp
// catch (Exception) bắt MỌI exception — các catch IOException phía sau
// không bao giờ tới (và compiler báo lỗi thứ tự). Ngay cả khi đảo,
// bắt Exception quá sớm sẽ nuốt cả lỗi lập trình (NullReference, ...).
```

**Bài 4** — Map file thiếu → infra exception:

```csharp
public class InfrastructureException : AppException
{
    public InfrastructureException(string message, Exception? inner = null)
        : base(message, inner!) { }
}

public class SaveSlotMissingException : InfrastructureException
{
    public SaveSlotMissingException(string slot, Exception inner)
        : base($"Save slot '{slot}' không tồn tại.", inner) { }
}

static string Load(string slot)
{
    try
    {
        return File.ReadAllText($"{slot}.sav");
    }
    catch (FileNotFoundException ex)
    {
        throw new SaveSlotMissingException(slot, ex);
    }
}
```

> Lưu ý: nếu `InfrastructureException` cần ctor không inner, thêm overload `(string message) : base(message)` cho sạch hơn bài tập.

## 13. Đáp án thay thế

Dùng `record` không phù hợp cho exception hierarchy cổ điển. Có thể dùng một `AppException` + enum `ErrorCode` thay hierarchy sâu — trade-off: ít class hơn nhưng `catch` kém tinh tế.

## 14. Thử thách

Thiết kế hierarchy cho MMORPG nhỏ: `GameException` → `CombatException` / `TradeException` / `AuthException`, mỗi nhánh ≥ 2 leaf. Viết bảng “ai bắt ở đâu” (UI combat / UI trade / global handler).

## 15. Ứng dụng thực tế

- ASP.NET: exception filter / middleware map type → status code
- Library NuGet: hierarchy ổn định across versions
- Desktop: global handler bắt `AppException`, crash reporter cho còn lại
- ETL: domain vs infra quyết định retry hay skip

## 16. Liên hệ Unity

- `GameException` base cho gameplay systems
- UI layer bắt `TradeException` hiện panel; không lộ `IOException` thô
- Global `Application.logMessageReceived` / callback khác với C# exception hierarchy — đừng nhầm
- Addressables/load fail → infra exception → UI “thử lại”

## 17. Kiểm tra kiến thức

1. `catch (IOException)` có bắt `FileNotFoundException` không?  
   **Đáp án:** Có — vì `FileNotFoundException` kế thừa `IOException`.

2. Thứ tự catch đúng là gì?  
   **Đáp án:** Từ kiểu cụ thể (con) đến tổng quát (cha).

3. Vì sao cần lớp `DomainException` trung gian?  
   **Đáp án:** Bắt chung mọi lỗi nghiệp vụ mà không bắt infra/lỗi hệ thống.

4. Có nên catch `NullReferenceException` thường xuyên không?  
   **Đáp án:** Không — đó là bug cần sửa, không phải luồng nghiệp vụ.

5. Ném cụ thể hay tổng quát hơn?  
   **Đáp án:** Ném càng cụ thể càng tốt.
