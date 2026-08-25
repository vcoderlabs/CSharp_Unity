# Chương 1 — Thread vs Task

## 1. Mục tiêu học

- Phân biệt **OS thread** và **Task** (đơn vị công việc trên ThreadPool)
- Biết khi nào tạo `Thread` thủ công vs `Task.Run` / async IO
- Hiểu chi phí context switch và vì sao ThreadPool tồn tại
- Liên hệ: blocking vs non-blocking trong app/game

## 2. Điều kiện tiên quyết

- Level 1–2: method, class, exception cơ bản
- Level 7: delegate/`Action` (callback)
- .NET 8+ console

## 3. Khái niệm

**Thread** là luồng thực thi do OS quản lý: có stack riêng, tốn bộ nhớ và chi phí chuyển ngữ cảnh (context switch).

**Task** là abstraction: “một công việc có thể hoàn thành trong tương lai”. Nhiều Task chạy trên **ThreadPool** — pool thread tái sử dụng, không tạo thread mới mỗi lần.

| | Thread | Task |
|--|--------|------|
| Mức | OS / CLR | Thư viện `System.Threading.Tasks` |
| Chi phí tạo | Cao | Thấp (queue work item) |
| Kết quả | Khó trả giá trị sạch | `Task<T>` có `.Result` / `await` |
| Hủy | Abort (đã obsolete) | `CancellationToken` |
| Phù hợp | Hiếm: affinity, COM STA… | Hầu hết concurrency hiện đại |

IO (mạng, file) **không cần** chiếm thread trong lúc chờ — dùng async IO + Task. CPU-bound: `Task.Run` / `Parallel` đưa lên pool.

## 4. Mô hình tư duy

```text
Thread (OS):     [==== busy CPU ====][==== busy ====]
                    đắt, số lượng hạn chế

Task / Pool:     queue → worker threads lấy việc
                 Task A (await IO) → nhả thread
                 Task B chạy trên cùng worker
```

`Thread.Sleep` **chặn** thread. `await Task.Delay` **nhả** thread về pool trong lúc chờ.

## 5. Cú pháp

Tạo thread thủ công và chạy work trên ThreadPool qua Task:

```csharp
using System.Threading;
using System.Threading.Tasks;

// Thread thủ công
var t = new Thread(() => Console.WriteLine($"Thread {Environment.CurrentManagedThreadId}"));
t.IsBackground = true;
t.Start();
t.Join();

// Task trên ThreadPool
await Task.Run(() =>
{
    Console.WriteLine($"Pool thread {Environment.CurrentManagedThreadId}");
    Thread.Sleep(100); // CPU/block demo — tránh trong production async
});
```

## 6. Ví dụ

### Cơ bản

So sánh Sleep chặn thread với Delay không chặn:

```csharp
Console.WriteLine("Sleep bắt đầu");
Thread.Sleep(500); // chiếm thread 500ms
Console.WriteLine("Sleep xong");

Console.WriteLine("Delay bắt đầu");
await Task.Delay(500); // nhả thread, tiếp tục sau 500ms
Console.WriteLine("Delay xong");
```

### Trung cấp

Chạy 2 việc song song bằng Task:

```csharp
async Task DemoParallelWork()
{
    var t1 = Task.Run(() => HeavyCpu(1));
    var t2 = Task.Run(() => HeavyCpu(2));
    await Task.WhenAll(t1, t2);
    Console.WriteLine($"Kết quả: {t1.Result}, {t2.Result}");
}

static int HeavyCpu(int id)
{
    int sum = 0;
    for (int i = 0; i < 5_000_000; i++) sum += i % 7;
    return sum + id;
}
```

### Nâng cao

Đếm thread ID trước/sau await — thường khác nhau (continuation trên pool):

```csharp
async Task ShowThreadHop()
{
    Console.WriteLine($"Trước: {Environment.CurrentManagedThreadId}");
    await Task.Delay(10);
    Console.WriteLine($"Sau:   {Environment.CurrentManagedThreadId}");
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| App “đơ” | `Thread.Sleep` / sync block trên UI/main | `await Task.Delay` / async API |
| Quá nhiều `new Thread` | Không dùng pool | `Task.Run` / `ThreadPool.QueueUserWorkItem` |
| Exception “mất” trên Thread | Không observe | Dùng Task + `await` / `Wait` có try |
| `Task.Run` cho mọi IO | Lãng phí pool thread | Dùng async IO thuần (`HttpClient`, `FileStream` async) |

## 8. Gỡ lỗi

1. Log `Environment.CurrentManagedThreadId` tại điểm nghi ngờ.
2. Visual Studio / Rider: cửa sổ **Threads** / **Tasks**.
3. Nếu CPU 100% với nhiều thread: xem có spin-wait hoặc vòng lặp vô hạn không.

## 9. Best practices

- Mặc định: **async Task** cho IO; **Task.Run** chỉ khi CPU-bound cần offload.
- Tránh tạo Thread thủ công trừ khi API cũ bắt buộc.
- Đặt `IsBackground = true` nếu bắt buộc dùng Thread (không chặn process exit).
- Đừng giả định thread ID ổn định sau `await`.

## 10. Bài tập

**Bài 1** — In thread ID của main và của `Task.Run`.

**Bài 2** — Viết method `static async Task TickAsync(int times)` in số mỗi 200ms bằng `Task.Delay`.

**Bài 3** — So sánh thời gian: 3 lần `Thread.Sleep(300)` tuần tự vs 3 `Task.Delay(300)` với `WhenAll`.

**Bài 4** — Dùng `ThreadPool.GetAvailableThreads` in trước/sau khi `Task.Run` nhiều việc ngắn.

## 11. Gợi ý

- Bài 1: `Console.WriteLine` trong lambda `Task.Run`.
- Bài 2: vòng `for` + `await Task.Delay(200)`.
- Bài 3: `Stopwatch`.
- Bài 4: `Task.WhenAll(Enumerable.Range(0, 50).Select(_ => Task.Run(() => Thread.Sleep(50))))`.

## 12. Đáp án

**Bài 1** — So sánh ID main và pool:

```csharp
Console.WriteLine($"Main: {Environment.CurrentManagedThreadId}");
await Task.Run(() =>
    Console.WriteLine($"Task.Run: {Environment.CurrentManagedThreadId}"));
```

**Bài 2** — Tick bằng Delay:

```csharp
static async Task TickAsync(int times)
{
    for (int i = 1; i <= times; i++)
    {
        Console.WriteLine(i);
        await Task.Delay(200);
    }
}
```

**Bài 3** — WhenAll nhanh hơn Sleep tuần tự (~300ms vs ~900ms):

```csharp
var sw = Stopwatch.StartNew();
Thread.Sleep(300); Thread.Sleep(300); Thread.Sleep(300);
Console.WriteLine($"Sleep: {sw.ElapsedMilliseconds}");

sw.Restart();
await Task.WhenAll(Task.Delay(300), Task.Delay(300), Task.Delay(300));
Console.WriteLine($"WhenAll Delay: {sw.ElapsedMilliseconds}");
```

**Bài 4** — Quan sát pool:

```csharp
ThreadPool.GetAvailableThreads(out int w1, out int io1);
Console.WriteLine($"Trước worker={w1} io={io1}");
await Task.WhenAll(Enumerable.Range(0, 50).Select(_ => Task.Run(() => Thread.Sleep(50))));
ThreadPool.GetAvailableThreads(out int w2, out int io2);
Console.WriteLine($"Sau worker={w2} io={io2}");
```

## 13. Đáp án thay thế

Bài 2 có thể dùng `PeriodicTimer` (.NET 6+): `await using var timer = new PeriodicTimer(...)` + `WaitForNextTickAsync`.

## 14. Thử thách

Viết demo: 1000 `new Thread(...).Start()` vs 1000 `Task.Run` — đo thời gian tạo + RAM (`GC.GetTotalMemory`). Rút kết luận.

## 15. Ứng dụng thực tế

- Web API: mỗi request là async pipeline trên Kestrel — không spawn Thread/request.
- Background worker / hosted service: `Task` dài hạn + cancel khi shutdown.
- Tool CLI: tải file song song bằng nhiều Task IO.

## 16. Liên hệ Unity

- **Main thread** Unity: Update/physics/API Unity chỉ an toàn trên main.
- `Thread` / `Task.Run` cho pathfinding nặng, generation — rồi marshal kết quả về main (`SynchronizationContext`, `UniTask`, hoặc queue).
- `Coroutine` ≠ Thread: coroutine chạy trên main, yield không tạo OS thread.
- Job System / Burst: mô hình khác (data-oriented), không thay thế hoàn toàn `Task` cho IO.

## 17. Kiểm tra kiến thức

1. Vì sao tạo hàng nghìn `Thread` tệ hơn hàng nghìn `Task`?  
   **Đáp án:** Mỗi Thread tốn stack/OS resource; Task queue trên pool tái sử dụng thread.

2. `Thread.Sleep` khác `await Task.Delay` chỗ nào?  
   **Đáp án:** Sleep chặn thread; Delay nhả thread, continuation sau khoảng chờ.

3. CPU-bound nên dùng gì?  
   **Đáp án:** `Task.Run` / `Parallel` / dedicated worker — không “async hóa” vòng tính toán thuần.

4. IO-bound nên ưu tiên gì?  
   **Đáp án:** Async IO API trả về `Task` — tránh chiếm pool thread chỉ để chờ.

5. Task có phải luôn chạy song song thật không?  
   **Đáp án:** Không — scheduling phụ thuộc pool/CPU; `WhenAll` bắt đầu cùng lúc nhưng hoàn thành theo tài nguyên.
