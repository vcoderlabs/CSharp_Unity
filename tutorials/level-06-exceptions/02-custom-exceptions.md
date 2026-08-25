# Chương 2 — Custom Exceptions

## 1. Mục tiêu học

- Tạo class exception riêng kế thừa `Exception`
- Cung cấp constructor message / inner exception theo convention
- Đặt tên và tổ chức exception theo domain
- Biết khi nào custom exception hữu ích hơn BCL có sẵn

## 2. Điều kiện tiên quyết

- Chương 1: try/catch/finally
- Level 2: kế thừa class
- Hiểu `base(...)` gọi constructor cha

## 3. Khái niệm

BCL có sẵn nhiều exception (`ArgumentNullException`, `InvalidOperationException`, …). **Custom exception** dùng khi:

- Domain có lỗi đặc thù (vd: `InsufficientGoldException` trong game)
- Caller cần `catch` riêng để xử lý khác nhau
- Muốn gắn thêm property ngữ cảnh (PlayerId, ErrorCode)

Convention .NET:

```csharp
public class MyAppException : Exception
{
    public MyAppException() { }
    public MyAppException(string message) : base(message) { }
    public MyAppException(string message, Exception inner) : base(message, inner) { }
}
```

Từ .NET hiện đại, serialization constructor cũ ít bắt buộc cho app mới; vẫn giữ 3 ctor trên là đủ thực tế.

## 4. Mô hình tư duy

```text
Lỗi kỹ thuật chung     → dùng BCL (ArgumentException, IOException, ...)
Lỗi nghiệp vụ rõ ràng  → CustomXxxException
Cần bắt riêng / thêm data → custom + property
Chỉ đổi message        → thường KHÔNG cần class mới
```

## 5. Cú pháp

```csharp
public class OrderNotFoundException : Exception
{
    public string OrderId { get; }

    public OrderNotFoundException(string orderId)
        : base($"Không tìm thấy đơn hàng '{orderId}'.")
    {
        OrderId = orderId;
    }

    public OrderNotFoundException(string orderId, Exception inner)
        : base($"Không tìm thấy đơn hàng '{orderId}'.", inner)
    {
        OrderId = orderId;
    }
}

// Ném
throw new OrderNotFoundException(id);

// Bắt
catch (OrderNotFoundException ex)
{
    Console.WriteLine(ex.OrderId);
}
```

## 6. Ví dụ

### Cơ bản

```csharp
public class EmptyNameException : Exception
{
    public EmptyNameException()
        : base("Tên không được rỗng.") { }
}

static void SetName(string name)
{
    if (string.IsNullOrWhiteSpace(name))
        throw new EmptyNameException();
}
```

### Trung cấp

Exception có mã lỗi:

```csharp
public class GameException : Exception
{
    public string Code { get; }

    public GameException(string code, string message)
        : base(message)
    {
        Code = code;
    }
}

throw new GameException("INV_FULL", "Túi đồ đã đầy.");
```

### Nâng cao

Bọc lỗi hạ tầng thành domain:

```csharp
public class SaveFailedException : Exception
{
    public string Slot { get; }

    public SaveFailedException(string slot, Exception inner)
        : base($"Lưu game thất bại (slot {slot}).", inner)
    {
        Slot = slot;
    }
}

static void Save(string slot, string json)
{
    try
    {
        File.WriteAllText($"{slot}.sav", json);
    }
    catch (IOException ex)
    {
        throw new SaveFailedException(slot, ex);
    }
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Custom exception không kế thừa `Exception` | Không bắt được như exception | `: Exception` hoặc lớp con |
| Quên truyền `inner` | Mất stack/nguyên nhân gốc | Luôn có ctor `(msg, inner)` |
| Hàng trăm exception class cho mọi if | Over-engineering | Chỉ tạo khi cần bắt riêng / gắn data |
| Đặt tên không có hậu tố `Exception` | Khó nhận diện | `SomethingException` |
| Throw `Exception` trần | Caller không phân loại | Throw cụ thể / custom |

## 8. Gỡ lỗi

1. Trong catch: in `ex.GetType().Name` + `ex.Message` + property custom.
2. Kiểm `InnerException` khi đã wrap.
3. Unit test: `Assert.Throws<OrderNotFoundException>(...)`.

## 9. Best practices

- Tên rõ: `PlayerNotFoundException`, không `MyError`.
- Thêm property hữu ích cho log/UI (Id, Code) — tránh parse `Message`.
- Seal class nếu không cần kế thừa thêm (`sealed class ...`).
- Document khi nào API ném exception nào.
- Đừng inherit `ApplicationException` (legacy, ít dùng).

## 10. Bài tập

**Bài 1** — Tạo `NegativeAmountException` với message cố định; method `Withdraw(decimal amount)` ném nếu `amount < 0`.

**Bài 2** — `UserNotFoundException` có property `string UserName`.

**Bài 3** — `ConfigLoadException(string path, Exception inner)` khi `File.ReadAllText` lỗi.

**Bài 4** — `InventoryException` với `string Code` và `string ItemId`; demo throw + catch in cả hai property.

## 11. Gợi ý

- Bài 1–2: ctor gọi `base(message)`.
- Bài 3: try/catch IO → `throw new ConfigLoadException(path, ex)`.
- Bài 4: hai auto-property set trong ctor.

## 12. Đáp án

**Bài 1** — Exception số âm + Withdraw:

```csharp
public class NegativeAmountException : Exception
{
    public NegativeAmountException()
        : base("Số tiền không được âm.") { }
}

static void Withdraw(decimal amount)
{
    if (amount < 0)
        throw new NegativeAmountException();
}
```

**Bài 2** — Gắn UserName vào exception:

```csharp
public class UserNotFoundException : Exception
{
    public string UserName { get; }

    public UserNotFoundException(string userName)
        : base($"Không tìm thấy user '{userName}'.")
    {
        UserName = userName;
    }
}
```

**Bài 3** — Wrap lỗi đọc config:

```csharp
public class ConfigLoadException : Exception
{
    public string Path { get; }

    public ConfigLoadException(string path, Exception inner)
        : base($"Không tải được config: {path}", inner)
    {
        Path = path;
    }
}

static string LoadConfig(string path)
{
    try
    {
        return File.ReadAllText(path);
    }
    catch (Exception ex) when (ex is IOException or UnauthorizedAccessException)
    {
        throw new ConfigLoadException(path, ex);
    }
}
```

**Bài 4** — Inventory với Code + ItemId:

```csharp
public class InventoryException : Exception
{
    public string Code { get; }
    public string ItemId { get; }

    public InventoryException(string code, string itemId, string message)
        : base(message)
    {
        Code = code;
        ItemId = itemId;
    }
}

// Demo
try
{
    throw new InventoryException("NOT_FOUND", "sword_01", "Item không tồn tại.");
}
catch (InventoryException ex)
{
    Console.WriteLine($"{ex.Code} / {ex.ItemId}: {ex.Message}");
}
```

## 13. Đáp án thay thế

Bài 1 có thể dùng `ArgumentOutOfRangeException(nameof(amount))` thay custom nếu không cần bắt riêng. Bài 3 bắt `IOException` thay `Exception when`.

## 14. Thử thách

Thiết kế 3 exception cho module thanh toán: `PaymentDeclinedException`, `InsufficientFundsException`, `PaymentGatewayException` (có inner). Viết method `Pay` giả lập 3 nhánh lỗi.

## 15. Ứng dụng thực tế

- Domain-Driven Design: exception theo bounded context
- Library công khai: exception ổn định là một phần API contract
- Microservices: map custom exception → mã lỗi API/gRPC
- Game: `QuestException`, `SkillCooldownException` cho UI tip

## 16. Liên hệ Unity

- `SaveGameException` khi ghi `persistentDataPath` fail
- `NetworkException` bọc lỗi UnityWebRequest (ở layer C#)
- Tránh throw trong hot path (`Update`) — dùng flag/error code
- Editor: custom exception giúp Dialog phân nhánh “retry / bỏ qua”

## 17. Kiểm tra kiến thức

1. Custom exception thường kế thừa class nào?  
   **Đáp án:** `Exception` (hoặc lớp con của nó).

2. Vì sao cần ctor `(string message, Exception inner)`?  
   **Đáp án:** Giữ nguyên nhân gốc khi wrap.

3. Khi nào *không* cần custom exception?  
   **Đáp án:** Khi BCL đã đủ và không cần bắt/gắn data riêng.

4. Nên parse `Message` để lấy Id không?  
   **Đáp án:** Không — dùng property trên exception.

5. Hậu tố tên class nên là gì?  
   **Đáp án:** `Exception`.
