# Chương 6 — Đồng bộ hóa: lock, Monitor, Mutex, Semaphore

## 1. Mục tiêu học

- Bảo vệ shared state bằng `lock` / `Monitor`
- Phân biệt `Mutex` (cross-process) và `Semaphore` / `SemaphoreSlim`
- Dùng `SemaphoreSlim` để giới hạn concurrency async
- Tránh giữ lock khi `await`

## 2. Điều kiện tiên quyết

- Chương 1, 5: đa luồng cơ bản
- Level 3: reference type (object lock)

## 3. Khái niệm

Khi nhiều thread đọc/ghi cùng dữ liệu → cần **critical section**.

| Công cụ | Phạm vi | Ghi chú |
|---------|---------|---------|
| `lock` / `Monitor` | Trong process | Phổ biến nhất; `lock` là syntactic sugar |
| `Mutex` | Cross-process | Nặng hơn; named mutex |
| `Semaphore` | Đếm số slot | Cross-process variant tồn tại |
| `SemaphoreSlim` | Trong process, async-friendly | `WaitAsync` |

`lock (obj) { ... }` ≡ `Monitor.Enter` / `Exit` với try/finally.

## 4. Mô hình tư duy

```text
lock: chỉ 1 thread trong block
Semaphore(3): tối đa 3 thread/task vào vùng
Mutex: giống lock nhưng có thể giữa các process

❌ lock (_gate) { await IoAsync(); }  // giữ lock qua await — cấm
✅ await sem.WaitAsync(); try { await IoAsync(); } finally { sem.Release(); }
```

## 5. Cú pháp

lock, Monitor, Mutex, SemaphoreSlim:

```csharp
private readonly object _gate = new();
private int _count;

void Inc()
{
    lock (_gate)
    {
        _count++;
    }
}

// SemaphoreSlim giới hạn 3
private readonly SemaphoreSlim _sem = new(3, 3);

async Task UsePoolAsync()
{
    await _sem.WaitAsync();
    try
    {
        await Task.Delay(100);
    }
    finally
    {
        _sem.Release();
    }
}
```

Mutex đơn giản (single-instance app):

```csharp
using var mutex = new Mutex(false, "Global\\MyApp_Unique");
if (!mutex.WaitOne(0)) { Console.WriteLine("Already running"); return; }
```

## 6. Ví dụ

### Cơ bản

Counter không an toàn vs có lock:

```csharp
int n = 0;
var bad = Parallel.For(0, 10_000, _ => { n++; }); // race
Console.WriteLine(n); // thường < 10000

int m = 0;
object gate = new();
Parallel.For(0, 10_000, _ => { lock (gate) m++; });
Console.WriteLine(m); // 10000
```

### Trung cấp

Monitor.Wait / Pulse (hàng đợi điều kiện tối giản):

```csharp
readonly object _lock = new();
bool _ready;

void Produce()
{
    lock (_lock)
    {
        _ready = true;
        Monitor.Pulse(_lock);
    }
}

void Consume()
{
    lock (_lock)
    {
        while (!_ready) Monitor.Wait(_lock);
        Console.WriteLine("got it");
    }
}
```

### Nâng cao

Giới hạn 5 download async:

```csharp
async Task DownloadAllAsync(IEnumerable<string> urls, CancellationToken ct)
{
    var sem = new SemaphoreSlim(5, 5);
    var tasks = urls.Select(async url =>
    {
        await sem.WaitAsync(ct);
        try { await FakeDownloadAsync(url, ct); }
        finally { sem.Release(); }
    });
    await Task.WhenAll(tasks);
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Deadlock | lock lồng nhau thứ tự khác nhau | Một thứ tự lock toàn cục; giảm số lock |
| lock `this` / string | Public object bị lock ngoài | `private readonly object _gate` |
| await trong lock | Compiler có thể cấm / nguy hiểm | SemaphoreSlim ngoài |
| Mutex quên Release | Process crash edge | `try/finally`; cân nhắc named carefully |
| SemaphoreSlim không Release | Exception path | `try/finally` |

## 8. Gỡ lỗi

1. Tái hiện race bằng vòng lặp lớn + Parallel.
2. Debugger: Threads window — ai đang giữ lock.
3. `lock` contention: giảm thời gian trong critical section.

## 9. Best practices

- Khóa object riêng, private, readonly.
- Critical section ngắn — không IO/sync HTTP trong lock.
- Async concurrency: `SemaphoreSlim`, không `lock` + await.
- Prefer `Interlocked` cho counter đơn giản.
- Mutex chỉ khi cần single-instance / IPC.

## 10. Bài tập

**Bài 1** — Sửa race `n++` bằng `lock`.

**Bài 2** — Cùng bài bằng `Interlocked.Increment`.

**Bài 3** — `SemaphoreSlim(2)` chạy 10 task Delay — chứng minh tối đa 2 cùng lúc (log timestamp).

**Bài 4** — Thử `lock` + `await` — quan sát lỗi compiler (CS1996) hoặc refactor đúng.

## 11. Gợi ý

- Bài 1–2: Parallel.For như mục 6.
- Bài 3: in `DateTime.UtcNow` khi vào/ra.
- Bài 4: dùng SemaphoreSlim thay lock.

## 12. Đáp án

**Bài 1** — lock:

```csharp
int n = 0;
object g = new();
Parallel.For(0, 10_000, _ => { lock (g) n++; });
```

**Bài 2** — Interlocked:

```csharp
int n = 0;
Parallel.For(0, 10_000, _ => Interlocked.Increment(ref n));
```

**Bài 3** — SemaphoreSlim(2):

```csharp
var sem = new SemaphoreSlim(2, 2);
var tasks = Enumerable.Range(0, 10).Select(async i =>
{
    await sem.WaitAsync();
    try
    {
        Console.WriteLine($"in {i} {DateTime.UtcNow:HH:mm:ss.fff}");
        await Task.Delay(200);
    }
    finally { sem.Release(); }
});
await Task.WhenAll(tasks);
```

**Bài 4** — Đúng cách async:

```csharp
await sem.WaitAsync();
try { await Task.Delay(1); }
finally { sem.Release(); }
```

## 13. Đáp án thay thế

`ReaderWriterLockSlim` khi đọc nhiều/ghi ít; hoặc concurrent collections (chương 7) giảm nhu cầu lock tay.

## 14. Thử thách

Implement rate limiter: tối đa N thao tác/giây bằng `SemaphoreSlim` + `PeriodicTimer` release — hoặc token bucket đơn giản.

## 15. Ứng dụng thực tế

- Bảo vệ dictionary cache in-memory
- Giới hạn connection tới service phụ
- Single-instance installer bằng named Mutex

## 16. Liên hệ Unity

- Gameplay trên main thread thường **không** cần lock nếu chỉ main đụng scene objects.
- Worker thread + main: queue message, đừng lock quanh `GameObject`.
- Job System có mô hình safety riêng — không trộn `lock` bừa bãi với NativeArray.

## 17. Kiểm tra kiến thức

1. `lock` dựa trên API nào?  
   **Đáp án:** `Monitor.Enter` / `Exit`.

2. Vì sao không `lock (this)`?  
   **Đáp án:** Bên ngoài có thể lock cùng instance → deadlock khó đoán.

3. Mutex khác lock chỗ nào?  
   **Đáp án:** Có thể đồng bộ cross-process; nặng hơn.

4. SemaphoreSlim hợp async vì sao?  
   **Đáp án:** Có `WaitAsync` — không block thread trong lúc chờ slot.

5. Có được `await` trong `lock` không?  
   **Đáp án:** Không — giữ lock qua yield gây deadlock/thiết kế sai.
