# Project Level 6 — Logging System

## 1. Mục tiêu học

- Thiết kế **custom exception hierarchy** cho một app nhỏ
- Implement `ILogger` + `FileLogger` (level, lock, optional min level)
- Demo: thao tác nghiệp vụ thất bại → throw cụ thể → catch → log → (tuỳ) rethrow/map
- Viết console demo rõ ràng, có thể chạy bằng `dotnet run`

## 2. Điều kiện tiên quyết

- Hoàn thành 6 chương Level 6
- .NET 8 console app
- Biết tạo class/file tách module

## 3. Khái niệm / Yêu cầu sản phẩm

### Phần A — Exception hierarchy

```text
AppException
├── DomainException
│   ├── UserNotFoundException      (UserName)
│   ├── ValidationException        (Field)
│   └── InsufficientPermissionException
└── InfrastructureException
    ├── LogWriteException          (khi chính logger lỗi — cẩn thận đừng vòng vô hạn)
    └── DataStoreException         (inner IO)
```

### Phần B — Logging

| Thành phần | Yêu cầu |
|------------|---------|
| `LogLevel` | Debug, Info, Warning, Error, Critical |
| `ILogger` | `void Log(LogLevel, string message, Exception? ex = null)` |
| `FileLogger` | Ghi UTC timestamp, thread-safe `lock`, `minLevel` |
| `ConsoleLogger` | (tuỳ chọn) dual-write hoặc chỉ console lúc demo |

### Phần C — Mini domain: User Service

- `Register(username, password)` — ValidationException nếu username rỗng / password ngắn
- `GetUser(username)` — UserNotFoundException nếu thiếu
- `SaveUsers()` — ghi file JSON/text; IO lỗi → DataStoreException
- Mọi lỗi domain/infra: log Error kèm exception; Main quyết định in thân thiện

### Phần D — Demo script

1. Đăng ký user hợp lệ → Info log  
2. Đăng ký user invalid → catch ValidationException → Warning/Error log  
3. GetUser không tồn tại → UserNotFoundException  
4. Giả lập DataStoreException (path không ghi được)  
5. In nội dung file log ra console cuối chương trình

## 4. Mô hình tư duy

```text
Program / UI
    ↓ gọi
UserService (domain) — throw DomainException
    ↓ dùng
IUserStore — throw / wrap InfrastructureException
    ↓ song song
ILogger — ghi mọi mốc & lỗi

Không: catch Exception nuốt im trong service
Có: bắt cụ thể ở Program để UX; service log + throw tiếp nếu cần
```

## 5. Cú pháp / Skeleton

```bash
dotnet new console -n LoggingSystem -f net8.0
cd LoggingSystem
```

Gợi ý file:

- `Exceptions/*.cs`
- `Logging/ILogger.cs`, `FileLogger.cs`
- `Users/User.cs`, `UserService.cs`, `FileUserStore.cs`
- `Program.cs`

## 6. Ví dụ hướng dẫn

### Cơ bản — Base exceptions

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

public class UserNotFoundException : DomainException
{
    public string UserName { get; }
    public UserNotFoundException(string userName)
        : base($"User '{userName}' không tồn tại.")
        => UserName = userName;
}
```

### Trung cấp — FileLogger

```csharp
public sealed class FileLogger : ILogger
{
    private readonly string _path;
    private readonly LogLevel _min;
    private readonly object _gate = new();

    public FileLogger(string path, LogLevel min = LogLevel.Debug)
    {
        _path = path;
        _min = min;
    }

    public void Log(LogLevel level, string message, Exception? ex = null)
    {
        if (level < _min) return;
        var line = $"{DateTime.UtcNow:o} [{level}] {message}";
        if (ex is not null)
            line += $"{Environment.NewLine}  {ex}";
        lock (_gate)
            File.AppendAllText(_path, line + Environment.NewLine);
    }
}
```

### Nâng cao — Service + wrap IO

```csharp
public sealed class UserService
{
    private readonly IUserStore _store;
    private readonly ILogger _log;

    public void Register(string username, string password)
    {
        if (string.IsNullOrWhiteSpace(username))
            throw new ValidationException("username", "Username rỗng.");
        if (password is null || password.Length < 6)
            throw new ValidationException("password", "Password tối thiểu 6 ký tự.");

        _store.Add(new User(username, password)); // đừng log password!
        _log.Log(LogLevel.Info, $"Registered user={username}");
    }

    public User GetUser(string username)
    {
        if (!_store.TryGet(username, out var user))
            throw new UserNotFoundException(username);
        return user!;
    }

    public void Persist()
    {
        try
        {
            _store.Save();
            _log.Log(LogLevel.Info, "Users persisted");
        }
        catch (IOException ex)
        {
            var wrap = new DataStoreException("Không lưu được users.", ex);
            _log.Log(LogLevel.Error, "Persist failed", wrap);
            throw wrap;
        }
    }
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Log cả password | Nhầm lẫn context | Chỉ log username |
| Logger throw → crash trong catch | Ghi file fail | Try/catch trong logger hoặc fallback Console |
| Catch `AppException` nuốt mọi thứ sớm | Quá rộng ở tầng thấp | Bắt cụ thể; global chỉ ở Main |
| Không `lock` FileLogger | Xen dòng | Luôn lock khi append |
| Hierarchy phẳng không base | Khó catch nhóm | Đủ Domain/Infra |

## 8. Gỡ lỗi

1. Chạy demo từng bước; mở `logs/app.log` sau mỗi thao tác.  
2. Breakpoint trong `catch (ValidationException)`.  
3. Cố tình tắt quyền ghi thư mục data để thấy `DataStoreException` + log.  
4. Đảm bảo `InnerException` còn khi wrap.

## 9. Best practices

- Không log secret; redact nếu cần.
- Message log có operation name: `Register`, `Persist`.
- `throw;` hoặc throw wrap có inner — không `throw ex` (reset stack) trừ khi cố ý.
- Tách `IUserStore` để sau này fake trong test.
- Min level Info trên “prod” demo; Debug khi phát triển.

## 10. Bài tập (checklist project)

**Bài 1** — Tạo solution + đủ hierarchy exception (property như spec).

**Bài 2** — `FileLogger` + `ConsoleLogger` (hoặc composite ghi cả hai).

**Bài 3** — `UserService` + store file; không bao giờ ghi password vào log.

**Bài 4** — `Program` demo 4–5 kịch bản; cuối cùng `File.ReadAllText` in log.

## 11. Gợi ý

- Store: `Dictionary` + `File.WriteAllLines` dạng `user=hash` (hash đơn giản hoặc chỉ length — đừng lưu plain nếu bạn muốn làm tốt hơn).
- Composite logger: list `ILogger` và forward `Log`.
- Thư mục `logs/` tạo bằng `Directory.CreateDirectory`.

## 12. Đáp án

**Skeleton Program** — Điều phối demo và bắt theo tầng:

```csharp
var logPath = Path.Combine("logs", "app.log");
Directory.CreateDirectory("logs");
ILogger logger = new FileLogger(logPath, LogLevel.Debug);
var store = new FileUserStore(Path.Combine("data", "users.txt"), logger);
var users = new UserService(store, logger);

try
{
    users.Register("ada", "lovelace");
    users.Persist();
    Console.WriteLine(users.GetUser("ada").UserName);
}
catch (ValidationException ex)
{
    logger.Log(LogLevel.Warning, $"Validation field={ex.Field}", ex);
    Console.WriteLine($"Input lỗi: {ex.Message}");
}
catch (DomainException ex)
{
    logger.Log(LogLevel.Error, "Domain error", ex);
    Console.WriteLine(ex.Message);
}
catch (InfrastructureException ex)
{
    logger.Log(LogLevel.Critical, "Infra error", ex);
    Console.WriteLine("Hệ thống lưu trữ gặp sự cố.");
}

Console.WriteLine("--- LOG FILE ---");
Console.WriteLine(File.ReadAllText(logPath));
```

**ValidationException** — Gắn tên field:

```csharp
public class ValidationException : DomainException
{
    public string Field { get; }
    public ValidationException(string field, string message) : base(message)
        => Field = field;
}
```

**DataStoreException** — Wrap IO:

```csharp
public class DataStoreException : InfrastructureException
{
    public DataStoreException(string message, Exception inner)
        : base(message, inner) { }
}
```

## 13. Đáp án thay thế

- Dùng `Microsoft.Extensions.Logging` + provider file thay `FileLogger` tự viết.
- Lưu user bằng `System.Text.Json` serialize `List<UserDto>` (không có password plain — chỉ salt/hash giả lập).
- `Result<T>` cho Register thay ValidationException — vẫn log Warning khi Fail.

## 14. Thử thách

1. Rolling log khi file > 512 KB.  
2. Composite: Console + File.  
3. Unit test: Register password ngắn → `Assert.Throws<ValidationException>`.  
4. Thêm `InsufficientPermissionException` cho thao tác `Delete` chỉ admin.

## 15. Ứng dụng thực tế

Đây là xương sống của mọi service: **domain errors có kiểu**, **infra errors có wrap**, **log có level + context**. Mở rộng thành ASP.NET middleware hoặc worker process dễ dàng.

## 16. Liên hệ Unity

- Port `ILogger` → wrapper quanh `Debug.Log*` + ghi `Application.persistentDataPath/logs`.
- Domain exception cho inventory/quest; không throw trong `Update`.
- Save system: `DataStoreException` khi ghi save fail → UI “Retry”.
- Tắt Debug log trong build release (`[Conditional("ENABLE_LOG")]` hoặc min level).

## 17. Kiểm tra kiến thức

1. Vì sao cần `DomainException` và `InfrastructureException` tách?  
   **Đáp án:** UX/retry khác nhau; bắt nhóm rõ; không lộ chi tiết IO cho user.

2. FileLogger cần `lock` vì sao?  
   **Đáp án:** Nhiều thread có thể log đồng thời — tránh xen/corruption.

3. Có log password khi Register không?  
   **Đáp án:** Không.

4. Sau `DataStoreException`, Main nên làm gì tối thiểu?  
   **Đáp án:** Log Critical (nếu chưa), thông báo thân thiện, không crash im nếu tránh được.

5. `throw wrap` cần truyền gì để giữ gốc lỗi?  
   **Đáp án:** `innerException` (exception IO gốc).
