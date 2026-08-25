# Chương 4 — IDisposable, `using` & using declaration

## 1. Mục tiêu học

- Hiểu **`IDisposable`**: giải phóng **deterministic** (không chờ GC)
- Dùng **`using` statement** và **`using` declaration** (.NET / C# hiện đại)
- Biết khi nào cần Dispose vs khi nào chỉ cần GC
- Tránh resource leak (file, stream, timer, Unity native wrappers có `IDisposable`)

## 2. Điều kiện tiên quyết

- Chương 3: object lifetime vs deterministic cleanup
- Level 6 (Exceptions) hữu ích: `using` ≈ try/finally
- C# 8+ using declaration; target **.NET 8+**

## 3. Khái niệm

### GC ≠ đóng resource kịp thời

GC thu **memory managed**. Nó **không** đóng file handle, socket, DB connection đúng lúc bạn cần. `IDisposable.Dispose()` = “dọn **ngay**”.

### Interface

```csharp
public interface IDisposable
{
    void Dispose();
}
```

### `using` statement (truyền thống)

Biên dịch thành `try` / `finally` gọi `Dispose()` — kể cả khi exception.

### `using` declaration (C# 8+)

```csharp
using var reader = new StreamReader(path);
// Dispose khi hết scope khối chứa declaration (thường hết method)
```

Gọn hơn, cùng đảm bảo Dispose cuối scope.

### `await using` (async disposable)

`IAsyncDisposable` + `await using` cho cleanup async (stream async, v.v.) — biết tồn tại; chi tiết sâu hơn ở async level nếu cần.

### Khi nào implement IDisposable?

| Cần Dispose | Không cần chỉ vì “có class” |
|-------------|------------------------------|
| FileStream, SqlConnection, HttpClient *factory patterns* | POCO / DTO thuần managed |
| `Timer`, event subscription owner (thường) | Struct nhỏ không giữ resource |
| Wrap native handle | Object chỉ có `List`/`string` |
| Object thuê từ pool đôi khi dùng Dispose = Return | — |

> **HttpClient:** đừng `using` short-lived bừa bãi (socket exhaustion) — dùng `IHttpClientFactory` / singleton có chủ đích. Đây là ngoại lệ design, không phá quy tắc Dispose nói chung.

## 4. Mô hình tư duy

```text
KHÔNG using:
  Open file → dùng → quên Close → handle leak → OS exhaustion

CÓ using:
  Open → dùng → [finally] Dispose → handle trả OS
                 ▲
                 └── chạy cả khi throw

Scope:
  {
    using var fs = File.OpenRead(path);
    // ...
  } // ← Dispose ở đây
```

**Unity:** nhiều API không phải `IDisposable` mà là `Destroy` / `OnDisable`. Vẫn học `using` cho tooling, editor scripts, file save/load, `BinaryReader`, native collections (`NativeArray.Dispose()`!).

## 5. Cú pháp

```csharp
// using statement
using (var fs = File.OpenRead("data.bin"))
{
    int b = fs.ReadByte();
} // Dispose

// using declaration — .NET 8 / C# hiện đại
using var fs2 = File.OpenRead("data.bin");
int b2 = fs2.ReadByte();
// Dispose cuối scope

// Nhiều resource
using var a = OpenA();
using var b = OpenB();

// Implement tối giản
sealed class LogFile : IDisposable
{
    private StreamWriter? _writer;
    public LogFile(string path) => _writer = new StreamWriter(path);
    public void Write(string line) => _writer?.WriteLine(line);
    public void Dispose()
    {
        _writer?.Dispose();
        _writer = null;
    }
}
```

## 6. Ví dụ

### Cơ bản — Đọc file an toàn

```csharp
static string ReadAllSafe(string path)
{
    using var reader = new StreamReader(path);
    return reader.ReadToEnd();
}
```

### Trung cấp — Class sở hữu resource

```csharp
sealed class ScoreStore : IDisposable
{
    private readonly FileStream _stream;
    private bool _disposed;

    public ScoreStore(string path)
    {
        _stream = File.Open(path, FileMode.OpenOrCreate, FileAccess.ReadWrite);
    }

    public void Append(byte[] data)
    {
        ObjectDisposedException.ThrowIf(_disposed, this);
        _stream.Write(data);
    }

    public void Dispose()
    {
        if (_disposed) return;
        _stream.Dispose();
        _disposed = true;
    }
}
```

### Nâng cao — using + exception

```csharp
static void Copy(string src, string dst)
{
    using var input = File.OpenRead(src);
    using var output = File.Create(dst);
    input.CopyTo(output);
    // Nếu CopyTo throw, cả hai stream vẫn Dispose nhờ using
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Quên `using` | Dựa GC/finalizer | Luôn `using` khi type là `IDisposable` |
| Double Dispose crash | Không idempotent | Guard `_disposed` |
| Dùng object sau Dispose | Lifetime sai | `ObjectDisposedException.ThrowIf` |
| `using` với null | C# cho phép null-noop | OK; vẫn rõ ownership |
| Dispose rồi trả pool nhầm | Design mơ hồ | Document: Dispose = destroy hay Return? |

## 8. Gỡ lỗi

1. Cảnh báo analyzer CA2000 / IDE: “Dispose object before lost scope”.
2. Repro handle leak: mở file vòng lặp không `using` → lỗi “file in use”.
3. Breakpoint trong `Dispose` — gọi bao nhiêu lần, từ đâu (call stack).
4. Unity NativeArray: quên `Dispose` → leak native + error safety check.

## 9. Best practices

- Prefer `using` / `await using` thay gọi `Dispose` tay.
- `Dispose()` phải **an toàn gọi nhiều lần** (idempotent).
- Sau Dispose, method public nên throw `ObjectDisposedException`.
- Sealed class đơn giản: Dispose ngắn; inheritance → Dispose pattern (chương 5).
- Không nuốt exception quan trọng bên trong Dispose trừ resource cleanup best-effort.

## 10. Bài tập

**Bài 1** — Viết `TempTextFile : IDisposable` tạo file temp, `Write`, xóa file trong `Dispose`.

**Bài 2** — Đọc 3 file tuần tự bằng `using` declaration; in số dòng mỗi file.

**Bài 3** — Cố tình bỏ `using` mở cùng một path 1000 lần `File.OpenWrite` — quan sát lỗi; sửa bằng `using`.

**Bài 4** — Giải thích `using var x = ...` Dispose lúc nào so với `using { }` khối hẹp.

## 11. Gợi ý

- Bài 1: `Path.GetTempFileName()`, `File.Delete` trong Dispose.
- Bài 2: vòng `foreach` path; mỗi lần một `StreamReader`.
- Bài 3: handle exhaustion / sharing — `using` đóng mỗi vòng.
- Bài 4: declaration = hết scope chứa nó; block = hết `}`.

## 12. Đáp án

**Bài 1**:

```csharp
sealed class TempTextFile : IDisposable
{
    private readonly string _path = Path.GetTempFileName();
    private StreamWriter? _writer;
    private bool _disposed;

    public TempTextFile() => _writer = new StreamWriter(_path);

    public void Write(string line)
    {
        ObjectDisposedException.ThrowIf(_disposed, this);
        _writer!.WriteLine(line);
    }

    public string Path => _path;

    public void Dispose()
    {
        if (_disposed) return;
        _writer?.Dispose();
        _writer = null;
        if (File.Exists(_path)) File.Delete(_path);
        _disposed = true;
    }
}
```

**Bài 2**:

```csharp
foreach (var path in paths)
{
    using var r = new StreamReader(path);
    int lines = 0;
    while (r.ReadLine() is not null) lines++;
    Console.WriteLine($"{path}: {lines}");
}
```

**Bài 3** — Không `using`: nhiều handle mở / file lock. Có `using`: mỗi iteration đóng stream.

**Bài 4** — `using var` Dispose khi rời scope bao quanh (method/block cha). `using (...) { }` Dispose ngay khi ra khỏi khối — hẹp hơn, tốt khi muốn giải phóng sớm giữa method dài.

## 13. Đáp án thay thế

Bài 1 có thể implement `IAsyncDisposable` nếu ghi async. Dùng `try/finally` thủ công tương đương `using` để hiểu desugar.

## 14. Thử thách

Viết helper `static TResult Use<TResource, TResult>(TResource resource, Func<TResource, TResult> body) where TResource : IDisposable` gọi body rồi Dispose — giống pattern loan.

## 15. Ứng dụng thực tế

- Đọc config/save game từ disk
- Export log, CSV tooling
- DB/commands trong backend
- `MemoryStream` ngắn hạn vẫn nên `using` (deterministic)

## 16. Liên hệ Unity

- `NativeArray<T>`, `NativeList<T>`, job buffers: **bắt buộc** `Dispose` — leak native ≠ GC spike nhưng crash/leak RAM device.
- Editor scripts: `using` khi parse asset file.
- Runtime gameplay object: thường pool/`Destroy`, không phải `IDisposable` — nhưng service locator tải file thì có.
- `CancellationTokenSource` implement `IDisposable` — nhớ dispose khi tạo nhiều CTS.
- GC spike ≠ quên Dispose file; nhưng quên Dispose native + spam managed = **hai** nguồn hitch.

## 17. Kiểm tra kiến thức

1. `IDisposable` giải quyết vấn đề gì GC không giải quyết kịp?  
   **Đáp án:** Giải phóng resource deterministic (handle, native, kết nối…).

2. `using` biên dịch thành gì?  
   **Đáp án:** `try`/`finally` gọi `Dispose`.

3. Using declaration Dispose khi nào?  
   **Đáp án:** Khi kết thúc scope chứa declaration.

4. Có cần `IDisposable` cho mọi class không?  
   **Đáp án:** Không — chỉ khi sở hữu resource cần cleanup sớm / unmanaged.

5. Gọi `Dispose` hai lần nên ra sao?  
   **Đáp án:** An toàn (idempotent), không throw vì lần hai.
