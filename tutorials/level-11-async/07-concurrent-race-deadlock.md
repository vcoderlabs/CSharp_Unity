# Chương 7 — Concurrent collections, race, deadlock, SynchronizationContext

## 1. Mục tiêu học

- Dùng `ConcurrentDictionary`, `ConcurrentQueue`, `ConcurrentBag`, `Channel`
- Nhận diện và sửa **race condition**
- Phân tích **deadlock** cổ điển (lock order, sync-over-async)
- Hiểu **SynchronizationContext** và hậu quả trong UI/Unity

## 2. Điều kiện tiên quyết

- Chương 5–6: Parallel, lock
- Level 4: Dictionary, Queue

## 3. Khái niệm

**Race condition:** kết quả phụ thuộc thứ tự xen kẽ không kiểm soát của thread.

**Deadlock:** các bên chờ nhau mãi — ví dụ thread A giữ lock1 chờ lock2; B giữ lock2 chờ lock1. Hoặc UI thread `.Result` trong khi continuation cần UI.

**Concurrent collections:** API thread-safe cho kịch bản đa luồng phổ biến (không phải mọi thao tác phức hợp đều atomic — chú `GetOrAdd` factory có thể chạy trùng).

**SynchronizationContext:** “bối cảnh” để post callback về (UI thread WinForms/WPF; ASP.NET cũ; Unity main). `await` mặc định capture context (trừ `ConfigureAwait(false)`).

## 4. Mô hình tư duy

```text
Race:          read-modify-write không atomic trên shared field
Deadlock lock: A→L1→L2  vs  B→L2→L1
Deadlock sync: UI waits Task ; Task needs UI to continue

ConcurrentDictionary: tốt cho cache đơn giản
Channel<T>: producer/consumer hiện đại (System.Threading.Channels)
```

## 5. Cú pháp

Concurrent collections và Channel:

```csharp
using System.Collections.Concurrent;
using System.Threading.Channels;

var dict = new ConcurrentDictionary<string, int>();
dict.AddOrUpdate("a", 1, (_, old) => old + 1);

var q = new ConcurrentQueue<int>();
q.Enqueue(1);
q.TryDequeue(out int x);

var channel = Channel.CreateBounded<int>(10);
await channel.Writer.WriteAsync(42);
channel.Writer.Complete();
await foreach (var item in channel.Reader.ReadAllAsync())
    Console.WriteLine(item);
```

## 6. Ví dụ

### Cơ bản

Race trên List vs ConcurrentBag:

```csharp
var list = new List<int>();
try
{
    Parallel.For(0, 10_000, i => { lock (list) list.Add(i); }); // hoặc đừng dùng List
}
catch { /* List không an toàn nếu thiếu lock */ }

var bag = new ConcurrentBag<int>();
Parallel.For(0, 10_000, i => bag.Add(i));
Console.WriteLine(bag.Count);
```

### Trung cấp

Deadlock lock order (đừng chạy production — demo timeout):

```csharp
object a = new(), b = new();
var t1 = Task.Run(() =>
{
    lock (a) { Thread.Sleep(50); lock (b) { } }
});
var t2 = Task.Run(() =>
{
    lock (b) { Thread.Sleep(50); lock (a) { } }
});
// Có thể treo — dùng Wait(timeout) khi thử nghiệm
Task.WaitAll(new[] { t1, t2 }, TimeSpan.FromSeconds(2));
```

### Nâng cao

Sync context giả lập vấn đề `.Result`:

```csharp
// Trên UI app: đừng làm thế này
// var data = GetDataAsync().Result; // deadlock nếu GetDataAsync cần UI sau await

// Đúng:
// var data = await GetDataAsync();
```

Producer/consumer Channel:

```csharp
var ch = Channel.CreateUnbounded<string>();
var producer = Task.Run(async () =>
{
    for (int i = 0; i < 5; i++) await ch.Writer.WriteAsync($"msg{i}");
    ch.Writer.Complete();
});
var consumer = Task.Run(async () =>
{
    await foreach (var m in ch.Reader.ReadAllAsync())
        Console.WriteLine(m);
});
await Task.WhenAll(producer, consumer);
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Count sai / exception List | Shared List không lock | Concurrent* hoặc lock |
| Deadlock lặng | Khóa hai chiều / `.Result` | Một chiều khóa; chỉ await |
| `GetOrAdd` chạy factory 2 lần | Không guarantee single factory | `Lazy<T>` value hoặc lock riêng |
| Sửa scene Unity từ pool | Mất sync context | Marshal về main |

## 8. Gỡ lỗi

1. Căng race: tăng số vòng Parallel, chạy Release.
2. Deadlock: pause debugger — nhìn call stacks chờ lock.
3. Unity: enable Main Thread checkers / log thread id trước API Unity.

## 9. Best practices

- Ưu tiên **immutability** / không share mutable state.
- Concurrent collections cho thao tác đơn giản; logic đa bước → lock rõ hoặc actor/queue một consumer.
- `Channel` cho pipeline.
- Thư viện: `ConfigureAwait(false)`. App UI: await trên UI context có chủ đích.
- Document thread affinity (Unity main).

## 10. Bài tập

**Bài 1** — Race `Dictionary` Add song song — bắt exception; sửa bằng `ConcurrentDictionary`.

**Bài 2** — Deadlock 2 lock; sửa bằng **cùng thứ tự** `lock(a)` rồi `lock(b)` cả hai bên.

**Bài 3** — Producer 100 số vào `ConcurrentQueue`, 4 consumer `TryDequeue` đến hết.

**Bài 4** — In `SynchronizationContext.Current` trước/sau `await Task.Delay` trên console (thường null).

## 11. Gợi ý

- Bài 1: `Parallel.For` + `dict[i]=i`.
- Bài 2: cả hai task `lock(a) lock(b)`.
- Bài 3: `while (q.TryDequeue...)` hoặc đếm `Interlocked`.
- Bài 4: `SynchronizationContext.Current?.GetType().Name ?? "null"`.

## 12. Đáp án

**Bài 1** — ConcurrentDictionary:

```csharp
var cd = new ConcurrentDictionary<int, int>();
Parallel.For(0, 10_000, i => cd[i] = i);
Console.WriteLine(cd.Count);
```

**Bài 2** — Cùng thứ tự khóa:

```csharp
void Both()
{
    lock (a)
    lock (b)
    { /* work */ }
}
// cả t1 và t2 gọi cùng thứ tự
```

**Bài 3** — Queue đa consumer:

```csharp
var q = new ConcurrentQueue<int>();
for (int i = 0; i < 100; i++) q.Enqueue(i);
int taken = 0;
Parallel.For(0, 4, _ =>
{
    while (q.TryDequeue(out _)) Interlocked.Increment(ref taken);
});
Console.WriteLine(taken);
```

**Bài 4** — Sync context console:

```csharp
Console.WriteLine(SynchronizationContext.Current?.ToString() ?? "null");
await Task.Delay(1);
Console.WriteLine(SynchronizationContext.Current?.ToString() ?? "null");
```

## 13. Đáp án thay thế

Bài 3 dùng `Channel.CreateBounded` + nhiều reader (`ReadAllAsync` chỉ một reader logic — hoặc nhiều worker `WaitToReadAsync`/`TryRead`).

## 14. Thử thách

Tái hiện deadlock sync-over-async trên WinForms/WPF hoặc Unity: gọi `.Result` từ main. Viết báo cáo 10 dòng cách tránh.

## 15. Ứng dụng thực tế

- Cache thread-safe `ConcurrentDictionary`
- Work queue background agents
- Tránh race trong metric counters (`Interlocked`)

## 16. Liên hệ Unity

- **SynchronizationContext** của Unity đưa continuation về main (tùy version/player loop integration).
- Race: worker sửa shared List mà Update cũng đọc → crash/nan.
- Pattern an toàn: worker tính toán → enqueue kết quả → main `Update` dequeue apply.
- Deadlock: `Wait()` trên main cho task cần main → treo editor/player.

## 17. Kiểm tra kiến thức

1. Race condition là gì?  
   **Đáp án:** Kết quả sai vì thứ tự xen kẽ không được đồng bộ hóa.

2. `ConcurrentDictionary` có làm mọi compound operation atomic không?  
   **Đáp án:** Không — check-then-act phức tạp vẫn cần thiết kế cẩn thận.

3. Deadlock cổ điển 2 lock sửa thế nào?  
   **Đáp án:** Thống nhất thứ tự lấy khóa (hoặc một khóa / lock-free thiết kế).

4. SynchronizationContext dùng để làm gì?  
   **Đáp án:** Post/work về bối cảnh gốc (thường UI/main thread).

5. Vì sao `.Result` trên UI dễ deadlock?  
   **Đáp án:** UI bị block chờ Task; continuation cần UI mới chạy tiếp.
