# Chương 4 — CancellationToken

## 1. Mục tiêu học

- Tạo và truyền `CancellationToken` / `CancellationTokenSource`
- Hủy hợp tác (cooperative cancellation): kiểm tra token, `ThrowIfCancellationRequested`
- Liên kết timeout bằng `CancelAfter` / `CancellationTokenSource.CreateLinkedTokenSource`
- Phân biệt `OperationCanceledException` với lỗi thật

## 2. Điều kiện tiên quyết

- Chương 2–3: async Task
- Level 6: catch exception đúng loại

## 3. Khái niệm

Hủy trong .NET là **hợp tác**: bên gọi yêu cầu hủy; bên thực thi **phải** quan sát token và dừng sớm. Không có “kill thread” an toàn.

| Thành phần | Vai trò |
|------------|---------|
| `CancellationTokenSource` (CTS) | Chủ sở hữu: gọi `Cancel()` |
| `CancellationToken` | Bản sao nhẹ: truyền xuống API |
| `Register` | Callback khi bị hủy |
| `CancelAfter` | Tự hủy sau khoảng thời gian |

API tốt: mọi method async dài nhận `CancellationToken cancellationToken = default`.

## 4. Mô hình tư duy

```text
UI / Host                    Worker
   CTS.Cancel() ──────────►  token.IsCancellationRequested?
                             ThrowIfCancellationRequested()
                             hoặc truyền token xuống HttpClient / Delay
```

Linked tokens: hủy nếu **timeout** HOẶC **user cancel**.

## 5. Cú pháp

Tạo CTS, truyền token, hủy và timeout:

```csharp
using var cts = new CancellationTokenSource();
cts.CancelAfter(TimeSpan.FromSeconds(5));

try
{
    await WorkAsync(cts.Token);
}
catch (OperationCanceledException)
{
    Console.WriteLine("Đã hủy hoặc timeout");
}

static async Task WorkAsync(CancellationToken ct)
{
    for (int i = 0; i < 100; i++)
    {
        ct.ThrowIfCancellationRequested();
        await Task.Delay(100, ct); // Delay tôn trọng token
    }
}
```

Linked source:

```csharp
using var linked = CancellationTokenSource.CreateLinkedTokenSource(userToken, timeoutToken);
await WorkAsync(linked.Token);
```

## 6. Ví dụ

### Cơ bản

Hủy sau phím Enter:

```csharp
using var cts = new CancellationTokenSource();
var task = LoopAsync(cts.Token);
Console.ReadLine();
cts.Cancel();
try { await task; } catch (OperationCanceledException) { Console.WriteLine("stopped"); }

static async Task LoopAsync(CancellationToken ct)
{
    while (true)
    {
        Console.Write(".");
        await Task.Delay(200, ct);
    }
}
```

### Trung cấp

Vòng CPU kiểm tra token định kỳ:

```csharp
static int SumWithCancel(int n, CancellationToken ct)
{
    int sum = 0;
    for (int i = 0; i < n; i++)
    {
        if ((i & 0xFFFF) == 0) ct.ThrowIfCancellationRequested();
        sum += i;
    }
    return sum;
}
```

### Nâng cao

Đăng ký cleanup khi cancel:

```csharp
using var cts = new CancellationTokenSource();
using var reg = cts.Token.Register(() => Console.WriteLine("cleanup!"));
cts.Cancel();
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Hủy không dừng | Không truyền/kiểm tra token | Truyền xuống mọi await IO |
| `ObjectDisposedException` | Dùng token sau dispose CTS | Scope `using` rõ; copy hành vi docs |
| Nuốt `OperationCanceledException` | catch Exception chung | Tách catch OCE; rethrow nếu không phải cancel của mình |
| Timeout = fire quên cancel | Không `CancelAfter` | Dùng CTS timeout hoặc `WaitAsync` |

## 8. Gỡ lỗi

1. Log khi `Cancel()` và khi catch OCE.
2. Breakpoint tại `ThrowIfCancellationRequested`.
3. Kiểm tra API con có overload nhận token không (HttpClient, EF, Delay).

## 9. Best practices

- Truyền token xuyên suốt call chain.
- `CancelAfter` cho deadline; linked token cho nhiều nguồn hủy.
- Đừng gọi `Cancel` rồi tiếp tục dùng cùng CTS cho thao tác mới — tạo CTS mới.
- Coi OCE là kết quả bình thường của hủy, không phải bug (trừ khi hủy nhầm).

## 10. Bài tập

**Bài 1** — `DownloadFakeAsync(ct)` delay 3s; cancel sau 1s từ main.

**Bài 2** — Thêm `CancelAfter(2s)` và in rõ timeout vs user cancel (gợi ý: 2 CTS + linked, hoặc cờ).

**Bài 3** — CPU loop 1e8 lần có kiểm tra token mỗi 10_000 vòng.

**Bài 4** — `Register` để set `volatile bool` hoặc gọi `Dispose` tài nguyên giả.

## 11. Gợi ý

- Bài 1: `Task.Delay(3000, ct)` + `cts.CancelAfter(1000)` hoặc Cancel thủ công.
- Bài 2: hai nguồn — `CreateLinkedTokenSource`.
- Bài 3: như mục 6 trung cấp.
- Bài 4: `token.Register(() => disposed = true)`.

## 12. Đáp án

**Bài 1** — Hủy sớm:

```csharp
using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(1));
try
{
    await Task.Delay(3000, cts.Token);
}
catch (OperationCanceledException)
{
    Console.WriteLine("Hủy trước khi xong 3s");
}
```

**Bài 2** — Linked user + timeout:

```csharp
using var userCts = new CancellationTokenSource();
using var timeoutCts = new CancellationTokenSource(TimeSpan.FromSeconds(2));
using var linked = CancellationTokenSource.CreateLinkedTokenSource(userCts.Token, timeoutCts.Token);
// userCts.Cancel() từ thread khác nếu cần
await Task.Delay(10_000, linked.Token);
```

**Bài 3** — CPU có cancel:

```csharp
static long Sum(CancellationToken ct)
{
    long s = 0;
    for (int i = 0; i < 100_000_000; i++)
    {
        if ((i % 10_000) == 0) ct.ThrowIfCancellationRequested();
        s += i;
    }
    return s;
}
```

**Bài 4** — Register cleanup:

```csharp
bool cleaned = false;
using var cts = new CancellationTokenSource();
using var _ = cts.Token.Register(() => cleaned = true);
cts.Cancel();
Console.WriteLine(cleaned); // True
```

## 13. Đáp án thay thế

.NET 8: `await task.WaitAsync(timeout, ct)` thay một số pattern timeout thủ công.

## 14. Thử thách

Viết helper `static async Task WithTimeout(Task task, TimeSpan timeout)` dùng linked CTS; nếu timeout ném `TimeoutException` thay vì OCE.

## 15. Ứng dụng thực tế

- ASP.NET: `HttpContext.RequestAborted`
- HostedService: `stoppingToken` khi shutdown
- CLI: Ctrl+C → `Console.CancelKeyPress` → `cts.Cancel()`

## 16. Liên hệ Unity

- Đổi scene / Destroy object: hủy UniTask/`CancellationToken` gắn lifetime object.
- `destroyCancellationToken` (Unity mới) hủy khi object destroy.
- Tránh tiếp tục await sau khi object đã destroy rồi set field.

## 17. Kiểm tra kiến thức

1. Cancellation trong .NET là gì?  
   **Đáp án:** Cooperative — code phải kiểm tra token và dừng.

2. Ai gọi `Cancel()`?  
   **Đáp án:** Chủ `CancellationTokenSource`, không phải người chỉ giữ Token.

3. `Task.Delay(ms, token)` giúp gì?  
   **Đáp án:** Thoát sớm khi hủy thay vì chờ đủ ms.

4. `CreateLinkedTokenSource` dùng khi nào?  
   **Đáp án:** Hủy nếu bất kỳ nguồn nào (user, timeout, parent) hủy.

5. Bắt `OperationCanceledException` rồi nuốt — khi nào OK?  
   **Đáp án:** Ở biên khi hủy là hành vi mong đợi; không nuốt lỗi khác.
