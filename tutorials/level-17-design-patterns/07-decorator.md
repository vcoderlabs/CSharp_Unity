# Chương 7 — Decorator

## 1. Mục tiêu học

- Gắn thêm hành vi **động** bằng bọc cùng interface
- So sánh vs inheritance sâu
- Stack nhiều decorator (logging + caching + retry)
- Unity: buff/weapon modifier

## 2. Điều kiện tiên quyết

- Interface, composition
- OCP

## 3. Khái niệm

**Decorator** triển khai cùng abstraction với component, giữ reference tới component bên trong, ủy thác + thêm việc trước/sau.

## 4. Mô hình tư duy

```text
Client → INotifier
           ↑
     LoggingDecorator → EncryptedDecorator → CoreNotifier
```

## 5. Cú pháp

```csharp
public interface INotifier { void Send(string msg); }

public abstract class NotifierDecorator : INotifier
{
    protected readonly INotifier Inner;
    protected NotifierDecorator(INotifier inner) => Inner = inner;
    public virtual void Send(string msg) => Inner.Send(msg);
}
```

## 6. Ví dụ

### Cơ bản

```csharp
public sealed class LogNotifier : NotifierDecorator
{
    public LogNotifier(INotifier inner) : base(inner) { }
    public override void Send(string msg)
    {
        Console.WriteLine($"log:{msg}");
        base.Send(msg);
    }
}
```

### Trung cấp — stack

```csharp
INotifier n = new SmsNotifier();
n = new LogNotifier(n);
n = new RetryNotifier(n, times: 3);
n.Send("hi");
```

### Nâng cao / Unity

```csharp
public interface IDamage { int Compute(); }

public sealed class BaseDamage : IDamage
{
    private readonly int _v;
    public BaseDamage(int v) => _v = v;
    public int Compute() => _v;
}

public sealed class CritDecorator : IDamage
{
    private readonly IDamage _inner;
    public CritDecorator(IDamage inner) => _inner = inner;
    public int Compute() => (int)(_inner.Compute() * 1.5f);
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Cách xử lý |
|------------|------------|
| Thứ tự decorator sai | Document order; test |
| Decorator đổi contract | Vi phạm LSP — giữ đúng interface |
| Quá nhiều lớp | Pipeline/list middleware rõ ràng |

## 8. Gỡ lỗi

Unit test từng lớp; integration test chuỗi. Log “enter/exit” decorator.

## 9. Best practices

- Cùng interface; không downcast bắt buộc.  
- Immutably wrap khi có thể.  
- ASP.NET middleware = họ hàng ý tưởng.

## 10. Bài tập

**Bài 1** — `IStream` Compress + Encrypt decorators.  
**Bài 2** — Damage: Base → ArmorPen → Crit.  
**Bài 3** — Caching decorator cho `IRepo.Get`.  
**Bài 4** — Khác Proxy? (chương 9)

## 11. Gợi ý

Caching: nếu key có trong dict trả luôn; không thì gọi inner rồi lưu.

## 12. Đáp án

```csharp
public sealed class CachingRepo<T> : IReadRepository<T>
{
    private readonly IReadRepository<T> _inner;
    private readonly Dictionary<Guid, T> _cache = new();
    public CachingRepo(IReadRepository<T> inner) => _inner = inner;
    public T? Get(Guid id)
    {
        if (_cache.TryGetValue(id, out var v)) return v;
        var loaded = _inner.Get(id);
        if (loaded != null) _cache[id] = loaded;
        return loaded;
    }
}
```

## 13. Đáp án thay thế

Pipeline `IEnumerable<IMiddleware>` thay nested ctor — dễ config.

## 14. Thử thách

Skill effect stack: burn + slow decorators trên `IStatusReceiver`.

## 15. Ứng dụng thực tế

- `BufferedStream`, crypto streams  
- DI decorator registration

## 16. Liên hệ Unity

- Weapon gems / card modifiers  
- Không nhầm với Unity `Decorator` UI toolkit naming

## 17. Kiểm tra kiến thức

1. Decorator thêm gì? **Hành vi động cùng interface.**  
2. Khác inheritance? **Compose runtime, stack được.**  
3. Phải giữ LSP? **Có.**  
4. Khác Adapter? **Không đổi interface mục tiêu.**  
5. Khác Proxy? **Proxy kiểm soát truy cập; Decorator gắn thêm chức năng (ranh giới mờ).**
