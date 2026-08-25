# Chương 3 — Coding standards & logging

## 1. Mục tiêu học

- Thiết lập **EditorConfig** + analyzer cơ bản
- Viết code nhất quán tên / async / nullable
- Dùng **ILogger** có level và structured properties
- Tránh log secret / spam hot path

## 2. Điều kiện tiên quyết

- Solution nhiều project (chương 2)
- Biết exception cơ bản (L6)

## 3. Khái niệm

**Coding standards:** quy ước team — không phải “đẹp cá nhân”. Tool hóa bằng `.editorconfig`, Roslyn analyzers, `dotnet format`.

**Logging:** ghi sự kiện để quan sát hệ thống. Khác `Console.WriteLine` ở chỗ: level, category, sink (file, Seq, App Insights), structured fields.

| Level | Khi nào |
|-------|---------|
| Trace/Debug | Chi tiết dev |
| Information | Luồng bình thường quan trọng |
| Warning | Bất thường nhưng xử lý được |
| Error | Thất bại thao tác |
| Critical | Hệ thống không tiếp tục |

## 4. Mô hình tư duy

```text
Code style  → máy enforce (CI format/analyze)
Logging     → người + máy đọc khi 3AM

Bad:  Log.Info("User " + user.Password)
Good: logger.LogInformation("User {UserId} placed order {OrderId}", userId, orderId);
```

## 5. Cú pháp

`.editorconfig` (rút gọn):

```ini
root = true

[*.cs]
indent_style = space
indent_size = 4
dotnet_sort_system_directives_first = true
csharp_style_var_for_built_in_types = true:suggestion
```

Logging:

```csharp
public sealed class OrderService(ILogger<OrderService> logger)
{
    public void Place(Guid orderId, Guid userId)
    {
        logger.LogInformation("Placing order {OrderId} for {UserId}", orderId, userId);
        try
        {
            // ...
        }
        catch (Exception ex)
        {
            logger.LogError(ex, "Failed order {OrderId}", orderId);
            throw;
        }
    }
}
```

## 6. Ví dụ

### Cơ bản — thay WriteLine

```csharp
logger.LogDebug("Cache miss for {Key}", key);
```

### Trung cấp — scope

```csharp
using (logger.BeginScope(new Dictionary<string, object> { ["RequestId"] = requestId }))
{
    logger.LogInformation("Start handle");
}
```

### Nâng cao — Serilog sink

```csharp
Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Information()
    .Enrich.FromLogContext()
    .WriteTo.Console()
    .WriteTo.File("logs/app.log", rollingInterval: RollingInterval.Day)
    .CreateLogger();
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| `$"..."` trong template | Mất structured field | Dùng `{Property}` + args |
| Log trong vòng chặt | I/O + alloc | Sample / Debug only |
| Nuốt exception sau log | `catch { log; }` | Re-throw hoặc trả Result |
| Style war trong PR | Không có editorconfig | Commit config chung |

## 8. Gỡ lỗi

1. Không thấy log: kiểm tra MinimumLevel + sink.
2. `dotnet format` local trước CI.
3. Analyzer warning: đọc mã quy tắc (IDE00xx / CAxxxx).
4. Correlation: gắn RequestId/TraceId vào mọi log một request.

## 9. Best practices

- Nullable enable + treat warnings seriously.
- Tên: Pascal class/method, camel `_field` theo team.
- Async method suffix `Async`.
- Log **hành động nghiệp vụ** + id; không dump object khổng lồ mặc định.
- Không log PII/secret (token, mật khẩu, full card).

## 10. Bài tập

**Bài 1** — Thêm `.editorconfig` và chạy `dotnet format`.

**Bài 2** — Inject `ILogger<T>` vào một service, thay 3 `Console.WriteLine`.

**Bài 3** — Cố ý log sai bằng string concat; sửa thành structured.

**Bài 4** — Viết guideline 10 dòng cho team (async, nullable, log level).

## 11. Gợi ý

- Bài 2: host `AddLogging` / generic host.
- Bài 3: template + placeholder.
- Bài 4: đặt trong `CONTRIBUTING.md`.

## 12. Đáp án

**Bài 3** — Trước: `logger.LogInformation("User " + id);`  
Sau: `logger.LogInformation("User {UserId}", id);`

**Bài 2** — Constructor injection như mục Cú pháp.

## 13. Đáp án thay thế

StyleCop.Analyzers, SonarAnalyzer. OpenTelemetry thay/kèm logging truyền thống cho traces.

## 14. Thử thách

Thêm middleware/pipeline log duration mỗi request + status code; không log body mặc định.

## 15. Ứng dụng thực tế

- On-call đọc log theo OrderId
- Audit trail nghiệp vụ
- SIEM tập trung

## 16. Liên hệ Unity

- `Debug.Log` ≈ Information; dùng wrapper có level
- Conditional log strip trong release
- Tránh log per-frame
- Cloud Diagnostics / crash reporting

## 17. Kiểm tra kiến thức

1. Structured logging khác gì nối chuỗi?  
   **Đáp án:** Field tách được để query; template ổn định.

2. `LogError(ex, ...)` nên truyền exception?  
   **Đáp án:** Có — giữ stack trace.

3. EditorConfig để làm gì?  
   **Đáp án:** Thống nhất style, giảm tranh luận.

4. Level nào cho “thẻ tín dụng sai format”?  
   **Đáp án:** Warning hoặc Information tùy nghiệp vụ — không Critical nếu app vẫn chạy.

5. Có log password không?  
   **Đáp án:** Không bao giờ.
