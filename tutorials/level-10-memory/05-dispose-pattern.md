# Chương 5 — Dispose Pattern

## 1. Mục tiêu học

- Implement **Dispose pattern** đầy đủ khi có inheritance / unmanaged
- Phân biệt `Dispose(bool disposing)`, `GC.SuppressFinalize`
- Biết khi nào pattern đầy đủ vs `sealed` + Dispose ngắn
- Áp dụng đúng trong library / wrapper Unity native

## 2. Điều kiện tiên quyết

- Chương 4: `IDisposable`, `using`
- Level 2: inheritance, `virtual`/`override`, `sealed`

## 3. Khái niệm

### Vì sao cần “pattern”?

Khi class **không sealed** và có thể có:

- Unmanaged resources (handle thô)
- Managed resources (`Stream`, khác `IDisposable`)
- Finalizer như lưới an toàn cuối cùng

Cần tách: cleanup từ `Dispose()` (deterministic) vs từ finalizer (chỉ unmanaged).

### Khung chuẩn (classic)

```csharp
public class ResourceBase : IDisposable
{
    private bool _disposed;

    public void Dispose()
    {
        Dispose(disposing: true);
        GC.SuppressFinalize(this);
    }

    protected virtual void Dispose(bool disposing)
    {
        if (_disposed) return;
        if (disposing)
        {
            // free managed: streams, timers, other IDisposable
        }
        // free unmanaged: handles, IntPtr
        _disposed = true;
    }

    ~ResourceBase() => Dispose(disposing: false);
}
```

- `disposing == true`: gọi từ `Dispose()` — được đụng managed objects
- `disposing == false`: gọi từ finalizer — **chỉ** unmanaged; managed có thể đã finalized

### `GC.SuppressFinalize(this)`

Sau Dispose thành công, bảo GC **không** cần chạy finalizer → rẻ hơn, khỏi hàng đợi finalizer.

### Thời hiện đại

- Prefer **`SafeHandle`** / wrapper BCL thay IntPtr trần + finalizer tự viết
- Prefer **`sealed` class** + Dispose đơn giản nếu không cần kế thừa
- `IAsyncDisposable` khi cleanup async

## 4. Mô hình tư duy

```text
Caller: using var x = new MyResource();
            │
            ▼
        Dispose()
            │
            ├─ Dispose(true)  → managed + unmanaged
            └─ SuppressFinalize → bỏ ~MyResource

Nếu quên using:
        GC → ~MyResource → Dispose(false) → chỉ unmanaged (lưới an toàn)
```

```text
Derived.Dispose(bool)
  1. cleanup của Derived
  2. base.Dispose(disposing)
```

## 5. Cú pháp

```csharp
public class FileMap : IDisposable
{
    private IntPtr _handle; // minh họa — thực tế dùng SafeHandle/MemoryMappedFile
    private Stream? _sideLog;
    private bool _disposed;

    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }

    protected virtual void Dispose(bool disposing)
    {
        if (_disposed) return;
        if (disposing)
        {
            _sideLog?.Dispose();
            _sideLog = null;
        }
        if (_handle != IntPtr.Zero)
        {
            // NativeClose(_handle);
            _handle = IntPtr.Zero;
        }
        _disposed = true;
    }

    ~FileMap() => Dispose(false);
}

public sealed class SimpleOwner : IDisposable
{
    private Stream? _stream = new MemoryStream();
    public void Dispose()
    {
        _stream?.Dispose();
        _stream = null;
    }
}
```

## 6. Ví dụ

### Cơ bản — Sealed đủ dùng

```csharp
sealed class BulletTrailBuffer : IDisposable
{
    private byte[]? _buffer = new byte[4096];
    public Span<byte> Span => _buffer ?? throw new ObjectDisposedException(nameof(BulletTrailBuffer));
    public void Dispose() => _buffer = null; // nhường GC; hoặc trả ArrayPool
}
```

### Trung cấp — Pattern + derived

```csharp
class ManagedHub : IDisposable
{
    private Timer? _timer;
    private bool _disposed;

    public ManagedHub() => _timer = new Timer(_ => { }, null, 1000, 1000);

    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }

    protected virtual void Dispose(bool disposing)
    {
        if (_disposed) return;
        if (disposing)
            _timer?.Dispose();
        _timer = null;
        _disposed = true;
    }
}

sealed class LoggedHub : ManagedHub
{
    private StreamWriter? _log = new("hub.log");

    protected override void Dispose(bool disposing)
    {
        if (disposing)
        {
            _log?.Dispose();
            _log = null;
        }
        base.Dispose(disposing);
    }
}
```

### Nâng cao — Trả pool trong Dispose

```csharp
sealed class PooledBuffer : IDisposable
{
    private byte[]? _rented = ArrayPool<byte>.Shared.Rent(4096);
    public byte[] Buffer => _rented ?? throw new ObjectDisposedException(nameof(PooledBuffer));

    public void Dispose()
    {
        var buf = Interlocked.Exchange(ref _rented, null);
        if (buf is not null)
            ArrayPool<byte>.Shared.Return(buf);
    }
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Finalizer đụng managed field | `Dispose(false)` sai | Chỉ unmanaged khi `!disposing` |
| Quên `SuppressFinalize` | Finalizer vẫn chạy | Gọi sau Dispose thành công |
| Derived quên `base.Dispose` | Leak base resource | Luôn gọi base |
| Throw trong Dispose | Phá using chain | Tránh throw; log |
| Public `Dispose(bool)` | Sai encapsulation | `protected virtual` |

## 8. Gỡ lỗi

1. Unit test: Dispose hai lần không throw; method sau Dispose throw `ObjectDisposedException`.
2. Fail test cố ý: không `using` + finalizer log (chỉ diagnostic).
3. Analyzer: implement pattern đúng chữ ký.
4. Memory Profiler: sau Dispose, object còn bị root không?

## 9. Best practices

- **Sealed + đơn giản** nếu không cần inherit.
- Có unmanaged thật → `SafeHandle` hơn tự viết finalizer.
- `Dispose` idempotent; field null sau cleanup.
- Document ownership: ai Dispose.
- Unity: với `NativeArray`, follow Unity dispose rules; đừng mix finalizer lung tung trên MB.

## 10. Bài tập

**Bài 1** — Viết `sealed class CountingStream` wrap `FileStream`, đếm số byte write; Dispose bên trong.

**Bài 2** — Base `ResourceBase` + derived `FileResource` theo Dispose pattern đủ `Dispose(bool)`.

**Bài 3** — Giải thích vì sao `Dispose(false)` không được gọi `_stream.Dispose()`.

**Bài 4** — Thêm `ObjectDisposedException.ThrowIf` vào method `Write` của Bài 1.

## 11. Gợi ý

- Bài 1: field `FileStream`, `_disposed`, `Write` tăng counter.
- Bài 2: copy khung classic; derived override rồi `base.Dispose`.
- Bài 3: managed có thể đã không còn safe trong finalizer order.
- Bài 4: đầu method `Write`.

## 12. Đáp án

**Bài 1**:

```csharp
sealed class CountingStream : IDisposable
{
    private readonly FileStream _stream;
    private bool _disposed;
    public long BytesWritten { get; private set; }

    public CountingStream(string path) =>
        _stream = File.Open(path, FileMode.Create, FileAccess.Write);

    public void Write(ReadOnlySpan<byte> data)
    {
        ObjectDisposedException.ThrowIf(_disposed, this);
        _stream.Write(data);
        BytesWritten += data.Length;
    }

    public void Dispose()
    {
        if (_disposed) return;
        _stream.Dispose();
        _disposed = true;
    }
}
```

**Bài 2** — Dùng khung mục 5; `FileResource` giữ `FileStream?`, dispose trong `if (disposing)`, rồi `base.Dispose(disposing)`.

**Bài 3** — Khi finalizer chạy, object managed khác có thể đã được thu/finalized; gọi `Dispose` chúng không an toàn / không deterministic. Chỉ giải phóng unmanaged của chính mình.

**Bài 4** — Như đoạn `ThrowIf` trong đáp án Bài 1.

## 13. Đáp án thay thế

Implement chỉ `IDisposable` không finalizer nếu 100% managed (vẫn OK). Dùng `MemoryMappedFile.CreateFromFile` thay IntPtr demo.

## 14. Thử thách

Viết `CompositeDisposable` giữ `List<IDisposable>`, `Add`, và `Dispose` tất cả (nuốt từng exception hoặc aggregate — chọn một policy và document).

## 15. Ứng dụng thực tế

- Library wrap OS resource
- Game tool import pipeline
- `CompositeDisposable` trong reactive UI
- Server connection multiplex ownership

## 16. Liên hệ Unity

- `NativeArray` / `Allocator.TempJob` — dispose đúng thời điểm job complete.
- Custom native plugin wrapper C#: SafeHandle + Dispose pattern.
- `MonoBehaviour` **không** thay bằng `IDisposable` — lifecycle Unity riêng; nhưng service thuần C# trong project thì có.
- Pool item: đôi khi `Dispose` = return pool — document rõ để không double-free.
- Sai Dispose native → RAM device tăng; sai managed spam → **GC spike**. Hai mặt đều phải sạch.

## 17. Kiểm tra kiến thức

1. `GC.SuppressFinalize` để làm gì?  
   **Đáp án:** Bỏ finalizer sau Dispose — tránh cleanup kép / hàng đợi đắt.

2. `Dispose(false)` được phép đụng managed không?  
   **Đáp án:** Không.

3. Khi nào đủ sealed Dispose ngắn?  
   **Đáp án:** Không cần inherit / không finalizer tự viết / chỉ managed rõ ràng.

4. Derived phải gọi gì?  
   **Đáp án:** `base.Dispose(disposing)` trong override.

5. Dispose nên idempotent nghĩa là gì?  
   **Đáp án:** Gọi nhiều lần an toàn, lần sau no-op.
