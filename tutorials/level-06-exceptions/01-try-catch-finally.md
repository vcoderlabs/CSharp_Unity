# Chương 1 — try / catch / finally

## 1. Mục tiêu học

- Hiểu exception là gì và khi nào runtime ném chúng
- Dùng `try`, `catch`, `finally` đúng thứ tự
- Bắt nhiều kiểu exception; dùng `when` sơ lược (chi tiết chương 4)
- Liên hệ `using` / `IDisposable` với `finally` giải phóng tài nguyên

## 2. Điều kiện tiên quyết

- Level 1–2: biến, method, class
- Biết đọc thông báo lỗi console / IDE
- .NET 8+ console app

## 3. Khái niệm

**Exception** là đối tượng mô tả lỗi bất thường lúc chạy. Khi `throw`, runtime tìm `catch` phù hợp trên call stack; nếu không có → chương trình crash (unhandled).

| Thành phần | Vai trò |
|------------|---------|
| `try` | Code có thể lỗi |
| `catch` | Xử lý khi đúng kiểu exception |
| `finally` | Luôn chạy (dọn dẹp), trừ khi process bị kill |
| `throw` | Ném exception (chương 5 đi sâu) |
| `using` | `try/finally` tự động gọi `Dispose` |

Ví dụ lỗi phổ biến: chia cho 0 (`DivideByZeroException`), null (`NullReferenceException`), file không tồn tại (`FileNotFoundException`), parse sai (`FormatException`).

## 4. Mô hình tư duy

```text
try { ... }
  ↓ lỗi xảy ra → tạo Exception object
  ↓ tìm catch khớp kiểu (từ cụ thể → tổng quát)
  ↓ chạy catch
  ↓ luôn chạy finally (nếu có)
  ↓ tiếp tục sau khối try-catch-finally

Nếu không có catch khớp:
  finally vẫn chạy → exception lan lên caller
```

## 5. Cú pháp

```csharp
try
{
    // code có thể ném exception
}
catch (FormatException ex)
{
    Console.WriteLine($"Format: {ex.Message}");
}
catch (Exception ex)
{
    Console.WriteLine($"Khác: {ex.Message}");
}
finally
{
    // dọn dẹp: đóng file, trả lock, ...
}

// using ≈ try/finally + Dispose
using var reader = new StreamReader(path);
string text = reader.ReadToEnd();
```

`catch` không đối số (`catch { }`) bắt mọi thứ — hiếm khi cần; ưu tiên `catch (Exception)` nếu bắt buộc, và thường **rethrow** hoặc log rồi để lan.

## 6. Ví dụ

### Cơ bản

Parse số an toàn:

```csharp
Console.Write("Nhập số: ");
string? input = Console.ReadLine();

try
{
    int n = int.Parse(input!);
    Console.WriteLine($"Bình phương: {n * n}");
}
catch (FormatException)
{
    Console.WriteLine("Không phải số nguyên.");
}
catch (OverflowException)
{
    Console.WriteLine("Số quá lớn/nhỏ.");
}
```

### Trung cấp

Đọc file + `finally` đóng stream thủ công (học khái niệm; production dùng `using`):

```csharp
StreamReader? reader = null;
try
{
    reader = new StreamReader("data.txt");
    Console.WriteLine(reader.ReadToEnd());
}
catch (FileNotFoundException ex)
{
    Console.WriteLine($"Thiếu file: {ex.FileName}");
}
finally
{
    reader?.Dispose();
}
```

### Nâng cao

`using` + nhiều catch + giữ inner context:

```csharp
static string ReadConfig(string path)
{
    try
    {
        using var fs = File.OpenRead(path);
        using var sr = new StreamReader(fs);
        return sr.ReadToEnd();
    }
    catch (UnauthorizedAccessException ex)
    {
        throw new InvalidOperationException($"Không đọc được config: {path}", ex);
    }
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| `catch (Exception)` rồi nuốt im | Che bug | Log + xử lý hoặc rethrow |
| `catch` tổng quát trước `catch` cụ thể | Không compile / bắt nhầm | Cụ thể trước, tổng quát sau |
| Quên `Dispose` khi lỗi giữa chừng | Leak handle | `using` / `finally` |
| Dùng exception để validate input thường xuyên | Chậm, khó đọc | `TryParse`, validate trước |
| `NullReferenceException` rồi chỉ `catch (Exception)` | Không biết gốc | Fix null; bắt cụ thể nếu cần |

## 8. Gỡ lỗi

1. Đọc **stack trace**: dòng nào `throw`, call stack từ đâu.
2. Xem `InnerException` khi bọc lại.
3. Breakpoint trong `catch` — kiểm tra `ex.GetType()`, `Message`, `StackTrace`.
4. Tạm bỏ `catch` để crash sớm khi đang phát triển (fail fast).

## 9. Best practices

- Bắt **cụ thể** nhất có thể.
- `finally` / `using` cho tài nguyên (file, socket, DB connection).
- Không để `catch` trống.
- Prefer `int.TryParse` thay `Parse` + catch cho input người dùng.
- Khi bọc lại: truyền `innerException` để không mất gốc lỗi.

## 10. Bài tập

**Bài 1** — Đọc hai số từ console, chia `a/b`. Bắt `FormatException`, `DivideByZeroException`, in thông báo tiếng Việt.

**Bài 2** — Method `static int SafeDivide(int a, int b)` trả `-1` nếu chia 0 (dùng try/catch trong method — sau chương 5 bạn sẽ cân nhắc cách khác).

**Bài 3** — Đọc toàn bộ text từ path; nếu file thiếu in `"MISSING"`; luôn in `"DONE"` ở `finally`.

**Bài 4** — Viết lại Bài 3 bằng `using` (không `finally` thủ công cho StreamReader).

## 11. Gợi ý

- Bài 1: `int.Parse` hai lần trong `try`; hai `catch` riêng.
- Bài 2: `catch (DivideByZeroException) { return -1; }`.
- Bài 3: `File.ReadAllText` trong try; `Console.WriteLine("DONE")` trong finally.
- Bài 4: `using var sr = new StreamReader(path);`.

## 12. Đáp án

**Bài 1** — Chia hai số với bắt lỗi cụ thể:

```csharp
try
{
    Console.Write("a = ");
    int a = int.Parse(Console.ReadLine()!);
    Console.Write("b = ");
    int b = int.Parse(Console.ReadLine()!);
    Console.WriteLine($"a/b = {a / b}");
}
catch (FormatException)
{
    Console.WriteLine("Nhập không phải số nguyên.");
}
catch (DivideByZeroException)
{
    Console.WriteLine("Không chia được cho 0.");
}
```

**Bài 2** — Bọc phép chia, trả mã lỗi:

```csharp
static int SafeDivide(int a, int b)
{
    try
    {
        return a / b;
    }
    catch (DivideByZeroException)
    {
        return -1;
    }
}
```

**Bài 3** — `finally` luôn báo DONE:

```csharp
try
{
    string text = File.ReadAllText("data.txt");
    Console.WriteLine(text);
}
catch (FileNotFoundException)
{
    Console.WriteLine("MISSING");
}
finally
{
    Console.WriteLine("DONE");
}
```

**Bài 4** — Dùng `using` tự Dispose:

```csharp
try
{
    using var sr = new StreamReader("data.txt");
    Console.WriteLine(sr.ReadToEnd());
}
catch (FileNotFoundException)
{
    Console.WriteLine("MISSING");
}
Console.WriteLine("DONE");
```

## 13. Đáp án thay thế

Bài 1 dùng `int.TryParse` + kiểm `b == 0` — tránh exception cho input thường (xem chương 5). Bài 3 dùng `File.Exists` trước khi đọc.

## 14. Thử thách

Viết method đọc CSV dòng đầu (2 cột số), cộng chúng; nếu file/format lỗi, ném `InvalidOperationException` kèm `innerException` gốc.

## 15. Ứng dụng thực tế

- API/backend: middleware bắt exception → HTTP 400/500
- Tool CLI: thông báo lỗi thân thiện, exit code ≠ 0
- Batch job: log lỗi một record, tiếp tục record khác
- Desktop app: dialog lỗi thay vì crash im

## 16. Liên hệ Unity

- `try/catch` quanh `JsonUtility` / parse save file — tránh một file corrupt làm crash game
- **Không** bọc toàn bộ `Update()` bằng catch rộng — che bug gameplay
- Resource: texture/audio load fail → fallback asset
- Editor scripts: bắt lỗi import để hiện Dialog thay crash Editor

## 17. Kiểm tra kiến thức

1. `finally` có chạy khi `catch` đã xử lý không?  
   **Đáp án:** Có — `finally` chạy sau try/catch (nếu khối có finally).

2. Nên bắt `Exception` trước `IOException`?  
   **Đáp án:** Không — bắt cụ thể trước; tổng quát sau.

3. `using` thay thế phần nào?  
   **Đáp án:** `try/finally` gọi `Dispose`.

4. Unhandled exception xảy ra khi nào?  
   **Đáp án:** Không có `catch` phù hợp trên call stack.

5. Vì sao tránh `catch { }` trống?  
   **Đáp án:** Nuốt lỗi, khó debug, che bug nghiêm trọng.
