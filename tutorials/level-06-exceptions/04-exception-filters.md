# Chương 4 — Exception Filters (`when`)

## 1. Mục tiêu học

- Dùng `catch (Ex ex) when (condition)` đúng ngữ cảnh
- Phân biệt filter với `if` bên trong `catch`
- Hiểu filter **không** unwind stack nếu điều kiện false (khác if trong catch)
- Áp dụng filter theo `HResult`, message pattern, hoặc property custom

## 2. Điều kiện tiên quyết

- Chương 1–3: catch, hierarchy, custom property
- Biểu thức boolean C#

## 3. Khái niệm

**Exception filter** là điều kiện sau `when`. Chỉ khi điều kiện `true`, khối `catch` mới nhận exception.

```csharp
catch (IOException ex) when (ex.Message.Contains("locked", StringComparison.OrdinalIgnoreCase))
{
    // chỉ khi file bị khóa
}
```

Khác biệt quan trọng với:

```csharp
catch (IOException ex)
{
    if (!ex.Message.Contains("locked"))
        throw; // đã vào catch → stack có thể đã bị ảnh hưởng / debug khác
    // xử lý
}
```

Với `when`: nếu false, runtime thử `catch` khác hoặc lan tiếp — **như chưa bắt**. Hữu ích khi debug (stack gốc giữ nguyên hơn) và khi nhiều handler cùng kiểu nhưng khác điều kiện.

## 4. Mô hình tư duy

```text
Exception bay lên
  → xét catch #1: kiểu khớp? → when true? → vào catch
  → không → catch #2 ...
  → không ai nhận → lan caller

when giống “bảo vệ cửa”: không đủ điều kiện thì không mở cửa catch đó.
```

## 5. Cú pháp

```csharp
try
{
    DoWork();
}
catch (HttpRequestException ex) when (ex.StatusCode == System.Net.HttpStatusCode.NotFound)
{
    Console.WriteLine("404");
}
catch (HttpRequestException ex) when ((int?)ex.StatusCode is >= 500 and < 600)
{
    Console.WriteLine("5xx — có thể retry");
}
catch (HttpRequestException ex)
{
    Console.WriteLine($"HTTP khác: {ex.Message}");
}
```

Lưu ý: tránh side-effect nặng trong `when` (I/O, thay đổi state) — filter nên **thuần** kiểm tra.

## 6. Ví dụ

### Cơ bản

```csharp
try
{
    throw new ArgumentException("bad", "email");
}
catch (ArgumentException ex) when (ex.ParamName == "email")
{
    Console.WriteLine("Lỗi tham số email");
}
```

### Trung cấp

Custom exception + filter theo Code:

```csharp
catch (InventoryException ex) when (ex.Code == "FULL")
{
    UI.Show("Túi đầy");
}
catch (InventoryException ex) when (ex.Code == "NOT_FOUND")
{
    UI.Show("Không có item");
}
```

### Nâng cao

Filter kết hợp logging quyết định (vẫn giữ when nhẹ):

```csharp
bool IsTransient(IOException ex) =>
    ex.HResult is unchecked((int)0x80070020) // sharing violation ví dụ
    || ex.Message.Contains("temporarily", StringComparison.OrdinalIgnoreCase);

try
{
    WriteFile();
}
catch (IOException ex) when (IsTransient(ex))
{
    Retry();
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Side-effect trong `when` | Khó đoán, chạy cả khi không catch | Chỉ đọc property |
| Filter quá phức tạp | Khó đọc | Tách method `bool CanHandle(Ex)` |
| Nhầm `when` với LINQ `Where` | Cú pháp khác ngữ cảnh | `when` chỉ sau catch |
| Dùng filter thay hierarchy | Code rải điều kiện | Kết hợp hierarchy + when vừa đủ |

## 8. Gỡ lỗi

1. Đặt breakpoint trong `when` method helper — xem true/false.
2. So sánh: tạm đổi sang `if` + `throw;` để thấy khác biệt khi debug step.
3. Log `ex` chỉ **trong** catch body, không trong when nếu when bị gọi nhiều lần thử.

## 9. Best practices

- `when` cho phân nhánh cùng kiểu exception.
- Điều kiện dựa property ổn định (`Code`, `StatusCode`, `ParamName`), không parse message mong manh nếu tránh được.
- Kết hợp hierarchy: bắt leaf khi có thể; `when` khi cùng class khác mã.
- Giữ filter không ném exception (nếu when throw → hành vi phức tạp/lỗi).

## 10. Bài tập

**Bài 1** — Ném `ArgumentException` với `ParamName` khác nhau; hai `catch when` cho `"name"` và `"age"`.

**Bài 2** — `GameException` có `int ErrorCode`; bắt khi `ErrorCode == 1001` và `== 1002`.

**Bài 3** — Method `IsFileLocked(IOException ex)` dùng trong `when`; giả lập bằng kiểm tra message chứa `"locked"`.

**Bài 4** — Viết cùng logic bằng `if` trong catch + `throw;`; ghi chú 2–3 dòng khác biệt với `when`.

## 11. Gợi ý

- Bài 1: `when (ex.ParamName == "name")`.
- Bài 2: property `ErrorCode` trên custom exception.
- Bài 3: `static bool IsFileLocked(IOException ex) => ex.Message.Contains("locked", ...);`.

## 12. Đáp án

**Bài 1** — Filter theo ParamName:

```csharp
try
{
    throw new ArgumentException("invalid", "age");
}
catch (ArgumentException ex) when (ex.ParamName == "name")
{
    Console.WriteLine("Sai name");
}
catch (ArgumentException ex) when (ex.ParamName == "age")
{
    Console.WriteLine("Sai age");
}
```

**Bài 2** — Filter theo ErrorCode:

```csharp
public class GameException : Exception
{
    public int ErrorCode { get; }
    public GameException(int code, string message) : base(message) => ErrorCode = code;
}

try { throw new GameException(1002, "Cooldown"); }
catch (GameException ex) when (ex.ErrorCode == 1001)
{
    Console.WriteLine("Not enough mana");
}
catch (GameException ex) when (ex.ErrorCode == 1002)
{
    Console.WriteLine("Skill cooldown");
}
```

**Bài 3** — Helper cho when:

```csharp
static bool IsFileLocked(IOException ex) =>
    ex.Message.Contains("locked", StringComparison.OrdinalIgnoreCase);

try
{
    throw new IOException("File is locked by another process");
}
catch (IOException ex) when (IsFileLocked(ex))
{
    Console.WriteLine("Retry sau...");
}
```

**Bài 4** — if + rethrow (so sánh):

```csharp
catch (IOException ex)
{
    if (!IsFileLocked(ex))
        throw; // exception đã bị catch rồi mới ném lại
    Console.WriteLine("Retry sau...");
}
// Khác when: điều kiện when false → catch này không nhận, stack giữ như chưa bắt.
// when phù hợp hơn khi có nhiều catch cùng kiểu hoặc muốn debugger/stack sạch hơn.
```

## 13. Đáp án thay thế

Dùng `switch` expression trên `ex` sau khi bắt cha — rõ ràng với C# hiện đại nhưng khác semantics “chưa bắt” của `when`.

## 14. Thử thách

Giả lập 3 HTTP status trên một `HttpSimException { int Status }`. Dùng chuỗi `catch when` cho 404, 409, 5xx; còn lại log generic.

## 15. Ứng dụng thực tế

- Retry chỉ với lỗi transient (timeout, locked file)
- Phân nhánh lỗi thanh toán theo mã từ cổng
- Multi-tenant: `when (ex.TenantId == current)`
- Telemetry: chỉ bắt một subset để metric

## 16. Liên hệ Unity

- `catch (Exception ex) when (ex is not OperationCanceledException)` pattern tương tự async
- Network layer: retry khi error code timeout
- Tránh `when` gọi Unity API không thread-safe nếu exception từ background thread
- Serialize error code trên custom exception thay parse chuỗi log Unity

## 17. Kiểm tra kiến thức

1. `when` viết ở đâu?  
   **Đáp án:** Ngay sau khai báo `catch (Type ex)`.

2. `when` false thì sao?  
   **Đáp án:** Catch đó không nhận; thử catch khác hoặc lan tiếp.

3. Nên làm I/O trong `when` không?  
   **Đáp án:** Không — giữ filter thuần/nhẹ.

4. `when` khác `if` trong catch chỗ nào cốt lõi?  
   **Đáp án:** `when` false ≈ chưa bắt; `if` false thường cần `throw` lại sau khi đã vào catch.

5. Filter tốt dựa trên gì?  
   **Đáp án:** Property ổn định (code, status, param name), không side-effect.
