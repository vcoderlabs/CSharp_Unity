# Chương 9 — Proxy

## 1. Mục tiêu học

- Kiểm soát truy cập tới object thật (lazy, remote, protection, virtual)
- Phân biệt Proxy vs Decorator
- Ví dụ virtual proxy load nặng; protection proxy quyền

## 2. Điều kiện tiên quyết

- Cùng interface composition như Decorator
- Async hữu ích (L11) cho remote proxy

## 3. Khái niệm

**Proxy** đứng thay subject, cùng interface, quyết định *khi nào/cách nào* gọi real subject: lazy create, cache, auth, remote stub.

## 4. Mô hình tư duy

```text
Client → IImage → ProxyImage → (lazy) RealImage
```

## 5. Cú pháp

```csharp
public interface IImage { void Display(); }

public sealed class LazyImageProxy : IImage
{
    private readonly string _path;
    private RealImage? _real;
    public LazyImageProxy(string path) => _path = path;
    public void Display()
    {
        _real ??= new RealImage(_path); // load nặng một lần
        _real.Display();
    }
}
```

## 6. Ví dụ

### Cơ bản — Virtual/Lazy proxy

Như trên.

### Trung cấp — Protection

```csharp
public sealed class SecureDoorProxy : IDoor
{
    private readonly IDoor _door;
    private readonly IAuth _auth;
    public void Open()
    {
        if (!_auth.IsAdmin) throw new UnauthorizedAccessException();
        _door.Open();
    }
}
```

### Nâng cao / Unity

Addressables lazy load prefab qua proxy `IAsset<T>` — chỉ load khi `Get()`. Remote proxy: client gọi local interface, gửi message server (networking L13).

## 7. Lỗi thường gặp

| Hiện tượng | Cách xử lý |
|------------|------------|
| Proxy = Decorator không kiểm soát truy cập | Đặt đúng intent |
| Lazy không thread-safe | Lock / Lazy\<T\> |
| Proxy leak real type | Chỉ expose interface |

## 8. Gỡ lỗi

Log “create real” chỉ một lần. Test unauthorized path của protection proxy.

## 9. Best practices

- Giữ proxy mỏng.  
- Document loại proxy (virtual/protection/remote).  
- C# `DispatchProxy` cho AOP nhẹ — advanced.

## 10. Bài tập

**Bài 1** — Lazy `IDatabase` mở connection lúc query.  
**Bài 2** — Protection `IAdminPanel`.  
**Bài 3** — Caching proxy (so Decorator).  
**Bài 4** — Remote proxy stub `IPlayerService.GetGold()`.

## 11. Gợi ý

Caching vừa proxy vừa decorator — chọn tên theo *ý định chính*.

## 12. Đáp án

```csharp
public sealed class LazyDb : IDatabase
{
    private readonly Func<IDatabase> _factory;
    private IDatabase? _db;
    public LazyDb(Func<IDatabase> factory) => _factory = factory;
    public void Query(string sql) => (_db ??= _factory()).Query(sql);
}
```

## 13. Đáp án thay thế

Lazy injection DI container (`Lazy<T>`) — proxy “có sẵn” từ infrastructure.

## 14. Thử thách

Proxy rate-limit gọi API 5 req/s.

## 15. Ứng dụng thực tế

- gRPC/HTTP client stubs  
- ORM lazy loading (cẩn thận N+1)

## 16. Liên hệ Unity

- Lazy load world chunks  
- Cheat-protection thin proxy trên lệnh admin (client vẫn không tin tưởng — server authoritative)

## 17. Kiểm tra kiến thức

1. Proxy kiểm soát gì? **Truy cập tới real subject.**  
2. Virtual proxy? **Lazy tạo object đắt.**  
3. Khác Decorator? **Intent: control access vs add behavior (ranh giới mờ).**  
4. Remote proxy? **Đại diện object ở process khác.**  
5. Protection proxy? **Kiểm tra quyền trước khi ủy thác.**
