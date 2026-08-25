# Project Level 7 — Event-driven Notification System

## 1. Mục tiêu học

- Thiết kế **publisher / subscriber** với `event` (không public Action field)
- Dùng `EventHandler<TEventArgs>` hoặc `Action<T>` có kiểm soát
- Nhiều kênh thông báo (Email / SMS / Push giả lập) đăng ký multicast
- Unsubscribe sạch; SafeRaise khi một channel lỗi
- Viết ghi chú **map sang UnityEvent** cho team game

## 2. Điều kiện tiên quyết

- Hoàn thành 6 chương Level 7
- Level 6 hữu ích cho SafeRaise + log lỗi channel
- .NET 8 console app

## 3. Khái niệm / Yêu cầu sản phẩm

### Phần A — Domain

```text
Notification
  Id, Title, Body, Severity (Info/Warning/Critical), CreatedAtUtc

NotificationHub (publisher)
  event EventHandler<NotificationEventArgs>? Published
  void Publish(Notification n)
  void PublishSafe(Notification n)  // GetInvocationList + try/catch từng handler
```

### Phần B — Subscribers (channels)

| Channel | Hành vi demo |
|---------|----------------|
| `ConsoleChannel` | In màu/severity theo Severity |
| `FileChannel` | Append vào `notifications.log` |
| `FlakySmsChannel` | Random hoặc luôn throw lần đầu — để test SafeRaise |

Mỗi channel có method `Handle(object? sender, NotificationEventArgs e)` hoặc đăng ký lambda **giữ reference**.

### Phần C — API Program

1. Tạo hub, đăng ký 3 channel  
2. Publish vài notification  
3. Gỡ `FileChannel`, publish thêm — file không thêm dòng  
4. Bật `PublishSafe` với Flaky channel — channel khác vẫn chạy  
5. (Bonus) Filter: chỉ Critical mới tới Sms  

### Phần D — Unity bridge notes

Trong README project, viết bảng:

| Console project | Unity |
|-----------------|-------|
| `NotificationHub.Published` | C# event trên GameManager **hoặc** `UnityEvent<Notification>` |
| `+= Handle` | `AddListener` |
| `-= Handle` | `RemoveListener` / OnDisable |
| Severity | ScriptableObject config / enum |

## 4. Mô hình tư duy

```text
[Producer: Game / Service]
        | Publish(notification)
        v
  NotificationHub  ----event---->  ConsoleChannel
                     |----------->  FileChannel
                     |----------->  SmsChannel (có thể lỗi)

Subscriber không biết nhau.
Hub không biết chi tiết channel — chỉ Invoke.
```

## 5. Cú pháp / Skeleton

```bash
dotnet new console -n NotificationSystem -f net8.0
cd NotificationSystem
```

Gợi ý file: `Notification.cs`, `NotificationEventArgs.cs`, `NotificationHub.cs`, `Channels/*.cs`, `Program.cs`, `UNITY.md`.

## 6. Ví dụ hướng dẫn

### Cơ bản — Model + Args

```csharp
public enum Severity { Info, Warning, Critical }

public sealed class Notification
{
    public Guid Id { get; } = Guid.NewGuid();
    public string Title { get; }
    public string Body { get; }
    public Severity Severity { get; }
    public DateTime CreatedAtUtc { get; } = DateTime.UtcNow;

    public Notification(string title, string body, Severity severity)
    {
        Title = title;
        Body = body;
        Severity = severity;
    }
}

public sealed class NotificationEventArgs : EventArgs
{
    public Notification Notification { get; }
    public NotificationEventArgs(Notification notification) =>
        Notification = notification;
}
```

### Trung cấp — Hub

```csharp
public sealed class NotificationHub
{
    public event EventHandler<NotificationEventArgs>? Published;

    public void Publish(Notification notification)
    {
        ArgumentNullException.ThrowIfNull(notification);
        Published?.Invoke(this, new NotificationEventArgs(notification));
    }

    public void PublishSafe(Notification notification)
    {
        ArgumentNullException.ThrowIfNull(notification);
        var args = new NotificationEventArgs(notification);
        var handlers = Published;
        if (handlers is null) return;

        foreach (var d in handlers.GetInvocationList())
        {
            try
            {
                ((EventHandler<NotificationEventArgs>)d)(this, args);
            }
            catch (Exception ex)
            {
                Console.WriteLine($"[Hub] channel lỗi: {ex.Message}");
            }
        }
    }
}
```

### Nâng cao — Channel + filter

```csharp
public sealed class ConsoleChannel
{
    public void Handle(object? sender, NotificationEventArgs e)
    {
        var n = e.Notification;
        Console.WriteLine($"[{n.Severity}] {n.Title}: {n.Body}");
    }
}

public sealed class CriticalOnlySmsChannel
{
    private readonly Action<Notification> _inner;
    public CriticalOnlySmsChannel(Action<Notification> inner) => _inner = inner;

    public void Handle(object? sender, NotificationEventArgs e)
    {
        if (e.Notification.Severity != Severity.Critical) return;
        _inner(e.Notification);
    }
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| `-=` không gỡ | Đăng ký bằng lambda mới | Method group / field lưu handler |
| FileChannel thiếu dòng sau unsubscribe | Đúng nếu đã -= | Verify Length invocation list |
| Flaky làm chết hết channel | Dùng `Publish` không Safe | `PublishSafe` |
| Hub public field Action | Bị `= null` từ ngoài | Chỉ `event` |
| Thread-safe raise | Copy local `var h = Published` | Đã làm trong PublishSafe pattern |

## 8. Gỡ lỗi

1. In `Published?.GetInvocationList().Length` trước/sau -=.  
2. Breakpoint từng `Handle`.  
3. Cố tình throw trong FileChannel — so Publish vs PublishSafe.  
4. Đọc `notifications.log` bằng editor.

## 9. Best practices

- EventArgs immutable; Notification immutable nếu được.
- Channel không throw vì lỗi format nhỏ — log và return; throw chỉ lỗi thật.
- Tách filter (CriticalOnly) khỏi transport (Sms).
- Document: Publish không guarantee tiếp tục sau exception.
- Unity: mirror lifecycle OnEnable/OnDisable trong `UNITY.md`.

## 10. Bài tập (checklist project)

**Bài 1** — Model + EventArgs + Hub.Publish.

**Bài 2** — ConsoleChannel + FileChannel đăng ký method group.

**Bài 3** — Demo unsubscribe FileChannel.

**Bài 4** — FlakySmsChannel + PublishSafe; viết `UNITY.md` (1 trang).

## 11. Gợi ý

- FileChannel: `File.AppendAllText` với dòng UTC + title.  
- Flaky: `throw new InvalidOperationException("SMS gateway down");` luôn hoặc `_failOnce`.  
- Program: `hub.Published += console.Handle;`.

## 12. Đáp án

**FileChannel** — Ghi log file:

```csharp
public sealed class FileChannel
{
    private readonly string _path;
    public FileChannel(string path) => _path = path;

    public void Handle(object? sender, NotificationEventArgs e)
    {
        var n = e.Notification;
        var line = $"{n.CreatedAtUtc:o} [{n.Severity}] {n.Title} — {n.Body}";
        File.AppendAllText(_path, line + Environment.NewLine);
    }
}
```

**FlakySmsChannel** — Luôn lỗi để test Safe:

```csharp
public sealed class FlakySmsChannel
{
    public void Handle(object? sender, NotificationEventArgs e)
    {
        throw new InvalidOperationException(
            $"SMS failed for '{e.Notification.Title}'");
    }
}
```

**Program demo** — Đăng ký, publish, gỡ, SafeRaise:

```csharp
var hub = new NotificationHub();
var console = new ConsoleChannel();
var file = new FileChannel("notifications.log");
var sms = new FlakySmsChannel();

hub.Published += console.Handle;
hub.Published += file.Handle;
hub.Published += sms.Handle;

hub.PublishSafe(new Notification("Welcome", "Hello", Severity.Info));
hub.PublishSafe(new Notification("Server", "Disk full", Severity.Critical));

hub.Published -= file.Handle;
hub.PublishSafe(new Notification("After unsub", "No file", Severity.Warning));
```

**UNITY.md (mẫu ngắn)** — Bridge tư duy:

```markdown
# Unity mapping
- NotificationHub → MonoBehaviour hoặc ScriptableObject event channel
- Published → UnityEvent<NotificationDto> cho Designer + C# event cho code
- OnEnable: hub.Published += Handle / onNotify.AddListener
- OnDisable: -= / RemoveListener
- Không Publish trong Update mỗi frame
```

## 13. Đáp án thay thế

- `event Action<Notification>?` thay EventHandler cho gọn.  
- MediatR / message pipe khi lớn hơn.  
- Channel implement `INotificationChannel { void Send(Notification); }` và hub giữ `List<>` thay multicast — dễ SafeRaise và thứ tự tường minh.

## 14. Thử thách

1. Severity filter middleware đăng ký một lần trên hub.  
2. Async: `Func<Notification, Task>` + `PublishAsync` tuần tự.  
3. Unit test: mock channel đếm số lần gọi sau unsubscribe = không tăng.  
4. Port tối giản sang Unity: một script Hub + UnityEvent + 2 listener Debug.Log.

## 15. Ứng dụng thực tế

Đây là xương sống **Observer** cho UI toast, achievement, analytics, anti-cheat alert. Trong backend: notification fan-out. Trong game: combat floating text + sound + quest progress cùng một “Publish”.

## 16. Liên hệ Unity

- **Bắt buộc nhớ:** C# `event` ≠ component `UnityEvent`, nhưng **cùng mô hình** pub/sub.  
- Dùng UnityEvent khi cần kéo thả trong Inspector (UI Button đã vậy).  
- Dùng C# event khi hệ thống thuần code, nhiều subscriber động, cần `-=` rõ.  
- Event Bus ScriptableObject (pattern phổ biến): SO chứa `event Action<T>` / UnityEvent — scene không hard-reference nhau.  
- Luôn unsubscribe khi disable object.

## 17. Kiểm tra kiến thức

1. Vì sao project dùng `event` thay public `Action` field?  
   **Đáp án:** Encapsulation — ngoài chỉ +=/-= , không gán đè hay Invoke tùy tiện.

2. PublishSafe giải quyết vấn đề gì?  
   **Đáp án:** Một channel throw không chặn các channel còn lại.

3. Làm sao `-=` FileChannel thành công?  
   **Đáp án:** Đăng ký cùng method group / cùng delegate instance đã +=.

4. Unity tương đương `+=` là gì?  
   **Đáp án:** `UnityEvent.AddListener` (hoặc C# `+=` nếu dùng C# event).

5. Khi nào nên chọn UnityEvent thay C# event?  
   **Đáp án:** Khi Designer/Inspector cần gán listener không sửa code.
