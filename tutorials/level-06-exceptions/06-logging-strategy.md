# Chương 6 — Logging Strategy

## 1. Mục tiêu học

- Hiểu vai trò log vs exception vs metric
- Chọn **log level** đúng (Trace/Debug/Info/Warning/Error/Critical)
- Log kèm context (correlation id, user id, operation) — không log secret
- Phác thảo chiến lược file logger đơn giản (chuẩn bị project)

## 2. Điều kiện tiên quyết

- Chương 1–5
- Biết ghi file text cơ bản (`File.AppendAllText`)
- (Tuỳ chọn) đã nghe `ILogger` / Serilog / NLog

## 3. Khái niệm

**Logging** ghi lại sự kiện để quan sát hệ thống khi chạy (dev + production).

| Level | Khi nào |
|-------|---------|
| Trace/Debug | Chi tiết phát triển, tắt trên prod hoặc sample |
| Information | Mốc nghiệp vụ quan trọng (user login, order placed) |
| Warning | Bất thường nhưng vẫn chạy (retry, fallback) |
| Error | Thao tác thất bại cần chú ý |
| Critical | Hệ thống không dùng được (DB down, disk full) |

**Exception** mang stack + ngữ cảnh lỗi một lần. **Log** ghi lại theo thời gian. Thường: catch → log Error kèm exception → xử lý hoặc rethrow.

Không log: mật khẩu, token, số thẻ, PII nhạy cảm nếu chưa mã hóa/policy.

## 4. Mô hình tư duy

```text
Sự kiện xảy ra
  → có phải lỗi? → log Error + exception object
  → bất thường tạm ổn? → Warning
  → mốc nghiệp vụ? → Information
  → chi tiết debug? → Debug (chỉ khi bật)

Mỗi dòng log nên trả lời:
  WHEN | LEVEL | WHAT | WHERE (operation) | WHO/CONTEXT | EXCEPTION?
```

## 5. Cú pháp

Interface tối giản:

```csharp
public enum LogLevel { Debug, Info, Warning, Error, Critical }

public interface ILogger
{
    void Log(LogLevel level, string message, Exception? ex = null);
}

public static class LoggerExtensions
{
    public static void Info(this ILogger log, string message) =>
        log.Log(LogLevel.Info, message);

    public static void Error(this ILogger log, string message, Exception ex) =>
        log.Log(LogLevel.Error, message, ex);
}
```

Ghi file đơn giản:

```csharp
public sealed class FileLogger : ILogger
{
    private readonly string _path;
    private readonly object _lock = new();

    public FileLogger(string path) => _path = path;

    public void Log(LogLevel level, string message, Exception? ex = null)
    {
        var line = $"{DateTime.UtcNow:o} [{level}] {message}";
        if (ex != null)
            line += $" | {ex.GetType().Name}: {ex.Message}";
        lock (_lock)
        {
            File.AppendAllText(_path, line + Environment.NewLine);
        }
    }
}
```

Production thật: dùng `Microsoft.Extensions.Logging`, Serilog sinks, structured logging JSON.

## 6. Ví dụ

### Cơ bản

```csharp
ILogger log = new FileLogger("app.log");
log.Info("Ứng dụng khởi động");
try
{
    File.ReadAllText("missing.txt");
}
catch (Exception ex)
{
    log.Error("Đọc file thất bại", ex);
}
```

### Trung cấp

Context trong message (hoặc dictionary properties):

```csharp
log.Log(LogLevel.Warning,
    $"Retry save slot={slot} attempt={attempt}/3");
```

### Nâng cao

Không nuốt exception sau log:

```csharp
catch (IOException ex)
{
    _log.Error($"Persist order {orderId} failed", ex);
    throw; // hoặc throw new InfrastructureException(..., ex);
}
```

Filter level:

```csharp
public void Log(LogLevel level, string message, Exception? ex = null)
{
    if (level < _minLevel) return;
    // write...
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Log mọi thứ Info | Nhiễu, tốn disk | Đúng level; sampling Debug |
| `Log(ex.ToString())` thiếu message | Khó tìm kiếm | Message + exception tách |
| Log password/token | Bảo mật | Redact; không bao giờ ghi secret |
| Nhiều thread ghi file không lock | Log hỏng/xen dòng | `lock` hoặc logger thread-safe |
| Catch log rồi nuốt | Mất failure signal | Rethrow hoặc trả Result Fail |

## 8. Gỡ lỗi

1. Thêm correlation id vào mỗi request/operation — grep một id là ra cả chuỗi.
2. Reproduce: bật Debug tạm thời.
3. Kiểm file log path, quyền ghi, disk đầy (Critical).

## 9. Best practices

- Structured logging (JSON fields) tốt hơn chuỗi tự do khi hệ thống lớn.
- Một message rõ + exception object; tránh chỉ `ex.Message`.
- Đồng bộ timezone: UTC trong log.
- Xoay file (rolling) theo ngày/size — project có thể làm đơn giản.
- Tách logger abstraction (`ILogger`) để test và đổi sink.

## 10. Bài tập

**Bài 1** — Implement `ConsoleLogger : ILogger` in ra `Console` có level.

**Bài 2** — `FileLogger` như skeleton; gọi Info + Error với exception giả.

**Bài 3** — Thêm `_minLevel`; Debug bị bỏ khi min = Info.

**Bài 4** — Method `Process` try/catch: log Error rồi ném lại `throw;`.

## 11. Gợi ý

- Bài 1: `$"{level}: {message}"`.
- Bài 3: `if (level < _minLevel) return;` (enum số tăng dần).
- Bài 4: `catch { log...; throw; }`.

## 12. Đáp án

**Bài 1** — Logger ra console:

```csharp
public sealed class ConsoleLogger : ILogger
{
    public void Log(LogLevel level, string message, Exception? ex = null)
    {
        Console.WriteLine($"{DateTime.UtcNow:HH:mm:ss} [{level}] {message}");
        if (ex != null)
            Console.WriteLine(ex);
    }
}
```

**Bài 2** — Ghi file kèm exception:

```csharp
var log = new FileLogger("app.log");
log.Log(LogLevel.Info, "Start");
try { throw new InvalidOperationException("demo"); }
catch (Exception ex)
{
    log.Log(LogLevel.Error, "Demo failure", ex);
}
```

**Bài 3** — Lọc theo min level:

```csharp
public sealed class FileLogger : ILogger
{
    private readonly string _path;
    private readonly LogLevel _minLevel;
    private readonly object _lock = new();

    public FileLogger(string path, LogLevel minLevel = LogLevel.Debug)
    {
        _path = path;
        _minLevel = minLevel;
    }

    public void Log(LogLevel level, string message, Exception? ex = null)
    {
        if (level < _minLevel) return;
        var line = $"{DateTime.UtcNow:o} [{level}] {message}";
        if (ex != null) line += $" | {ex.GetType().Name}: {ex.Message}";
        lock (_lock) File.AppendAllText(_path, line + Environment.NewLine);
    }
}
```

**Bài 4** — Log rồi rethrow:

```csharp
static void Process(ILogger log)
{
    try
    {
        throw new IOException("disk");
    }
    catch (Exception ex)
    {
        log.Log(LogLevel.Error, "Process failed", ex);
        throw;
    }
}
```

## 13. Đáp án thay thế

Dùng `Microsoft.Extensions.Logging.Console` trong app thật. Hoặc Serilog: `Log.Logger = new LoggerConfiguration().WriteTo.File(...).CreateLogger()`.

## 14. Thử thách

Thêm rolling đơn giản: nếu file > 1MB, rename `app.log` → `app.log.yyyyMMddHHmmss.bak` rồi tạo file mới. Thread-safe bằng lock.

## 15. Ứng dụng thực tế

- Microservices: centralized log (ELK, Seq, CloudWatch)
- SLA: alert khi Error/Critical tăng đột biến
- Audit: Info cho hành động nhạy cảm (không lẫn Debug spam)
- Hỗ trợ khách hàng: correlation id trên ticket

## 16. Liên hệ Unity

- `Debug.Log` / `LogWarning` / `LogError` ≈ Info/Warning/Error
- Production player: dùng logger riêng ghi file hoặc analytics; tắt log thừa
- `Debug.LogException(ex)` khi catch
- Không log mỗi frame trong `Update`
- Crash reporting (Cloud Diagnostics) bổ sung cho Critical

## 17. Kiểm tra kiến thức

1. Level nào cho “retry lần 2 vì timeout”?  
   **Đáp án:** Warning (thường).

2. Có nên log access token không?  
   **Đáp án:** Không.

3. Sau khi log Error, khi nào `throw;`?  
   **Đáp án:** Khi caller/layer trên vẫn cần biết thất bại.

4. Vì sao UTC trong timestamp?  
   **Đáp án:** Thống nhất môi trường/server khác múi giờ.

5. `ILogger` abstraction giúp gì?  
   **Đáp án:** Đổi sink (console/file/cloud), dễ test, tách hạ tầng.
