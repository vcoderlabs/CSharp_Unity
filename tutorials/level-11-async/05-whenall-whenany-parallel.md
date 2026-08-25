# Chương 5 — WhenAll / WhenAny và Parallel

## 1. Mục tiêu học

- Chạy nhiều Task đồng thời với `Task.WhenAll` / `Task.WhenAny`
- Xử lý exception dạng `AggregateException` / từng Task
- Dùng `Parallel.For` / `Parallel.ForEach` / `Parallel.ForEachAsync` cho CPU-bound
- Chọn đúng: async fan-out vs Parallel vs PLINQ

## 2. Điều kiện tiên quyết

- Chương 1–4: Task, await, cancel
- Level 4: collections
- Level 8: LINQ (tuỳ chọn PLINQ)

## 3. Khái niệm

| API | Ý nghĩa |
|-----|---------|
| `WhenAll` | Chờ **tất cả** hoàn thành; một lỗi → Task faulted (sau khi các task settle) |
| `WhenAny` | Chờ **một** task xong trước (thành công hoặc lỗi) |
| `Parallel.*` | Chia vòng lặp CPU trên nhiều thread pool |
| `Parallel.ForEachAsync` | .NET 6+: song song + async body + degree of parallelism |

**IO-bound nhiều request:** tạo nhiều `Task` async + `WhenAll` (không cần `Parallel`).  
**CPU-bound:** `Parallel` / `Task.Run` nhiều chunk.

## 4. Mô hình tư duy

```text
WhenAll:   [T1][T2][T3] ──► tất cả xong ──► tiếp
WhenAny:   [T1][T2][T3] ──► ai xong trước ──► (có thể cancel phần còn lại)

Parallel.For: chia range → nhiều worker chạy body đồng bộ
```

## 5. Cú pháp

WhenAll / WhenAny và Parallel:

```csharp
Task<int>[] tasks = { DownloadAsync(1), DownloadAsync(2), DownloadAsync(3) };
int[] results = await Task.WhenAll(tasks);

Task<int> first = await Task.WhenAny(tasks);
int value = await first; // vẫn await để propagate exception

Parallel.For(0, 100, i => { /* CPU work */ });

await Parallel.ForEachAsync(urls, new ParallelOptions { MaxDegreeOfParallelism = 4 },
    async (url, ct) => await FetchAsync(url, ct));
```

## 6. Ví dụ

### Cơ bản

Ba delay song song:

```csharp
var sw = Stopwatch.StartNew();
await Task.WhenAll(Task.Delay(300), Task.Delay(300), Task.Delay(300));
Console.WriteLine(sw.ElapsedMilliseconds); // ~300ms
```

### Trung cấp

WhenAny + hủy phần còn lại:

```csharp
static async Task<string> FirstWinsAsync()
{
    using var cts = new CancellationTokenSource();
    var tasks = new[]
    {
        SlowAsync("A", 500, cts.Token),
        SlowAsync("B", 200, cts.Token),
        SlowAsync("C", 400, cts.Token),
    };
    var winner = await Task.WhenAny(tasks);
    cts.Cancel(); // hủy các task còn lại (nếu chúng tôn trọng token)
    return await winner;
}

static async Task<string> SlowAsync(string name, int ms, CancellationToken ct)
{
    await Task.Delay(ms, ct);
    return name;
}
```

### Nâng cao

Parallel CPU + giới hạn độ song song:

```csharp
var bag = new ConcurrentBag<int>();
Parallel.For(0, 1000, new ParallelOptions { MaxDegreeOfParallelism = 4 }, i =>
{
    bag.Add(i * i);
});
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Chỉ thấy 1 exception | await WhenAll | Kiểm tra từng `task.Exception` hoặc `await Task.WhenAll` trong try + flatten |
| WhenAny “nuốt” lỗi task khác | Không observe task thua | Cancel + await/observe hoặc `WhenAll` sau |
| Parallel trên UI | Block main | Offload `Task.Run(() => Parallel.For...)` |
| Dùng Parallel cho HttpClient | Occupies threads | async + WhenAll / ForEachAsync |

## 8. Gỡ lỗi

1. In `task.IsFaulted` cho từng phần tử sau WhenAll fail.
2. `AggregateException.Flatten().InnerExceptions`.
3. Đo `MaxDegreeOfParallelism` — quá cao có thể chậm hơn.

## 9. Best practices

- IO: async fan-out + giới hạn concurrency (`SemaphoreSlim` hoặc `ForEachAsync`).
- CPU: `Parallel` với DOP hợp lý (= số core hoặc ít hơn nếu còn việc khác).
- Sau `WhenAny`, luôn `await` winner và hủy/observe losers.
- Tránh `Task.Run` bọc mỗi lần `await HttpGet` — thừa.

## 10. Bài tập

**Bài 1** — WhenAll 5 `Task.FromResult` cộng tổng kết quả.

**Bài 2** — WhenAny giữa Delay 100/300/500 — in cái thắng.

**Bài 3** — `Parallel.For` tính tổng bình phương 1..1_000_000 (dùng `Interlocked` hoặc local finally).

**Bài 4** — `Parallel.ForEachAsync` trên 10 URL giả với DOP=3.

## 11. Gợi ý

- Bài 1: `await Task.WhenAll` rồi `results.Sum()`.
- Bài 2: so sánh `task == delays[i]`.
- Bài 3: `Parallel.For` overload với `localInit`/`localFinally` hoặc `Interlocked.Add`.
- Bài 4: list string + `Task.Delay` giả download.

## 12. Đáp án

**Bài 1** — Tổng WhenAll:

```csharp
var tasks = Enumerable.Range(1, 5).Select(i => Task.FromResult(i)).ToArray();
int[] vals = await Task.WhenAll(tasks);
Console.WriteLine(vals.Sum()); // 15
```

**Bài 2** — WhenAny:

```csharp
var a = Task.Delay(100);
var b = Task.Delay(300);
var c = Task.Delay(500);
var w = await Task.WhenAny(a, b, c);
Console.WriteLine(w == a ? "A" : w == b ? "B" : "C");
await Task.WhenAll(a, b, c); // observe all
```

**Bài 3** — Parallel sum of squares:

```csharp
long total = 0;
Parallel.For(1, 1_000_001, () => 0L, (i, _, local) => local + (long)i * i,
    local => Interlocked.Add(ref total, local));
Console.WriteLine(total);
```

**Bài 4** — ForEachAsync giả:

```csharp
var urls = Enumerable.Range(1, 10).Select(i => $"https://example/{i}").ToList();
await Parallel.ForEachAsync(urls, new ParallelOptions { MaxDegreeOfParallelism = 3 },
    async (url, ct) =>
    {
        await Task.Delay(100, ct);
        Console.WriteLine(url);
    });
```

## 13. Đáp án thay thế

Bài 3 dùng PLINQ: `Enumerable.Range(1, 1_000_000).AsParallel().Sum(i => (long)i * i)`.

## 14. Thử thách

Giới hạn 100 download đồng thời bằng `SemaphoreSlim` + `WhenAll` — so sánh với `ForEachAsync`.

## 15. Ứng dụng thực tế

- Crawl/API batch
- Image resize CPU song song
- Health check nhiều dependency bằng WhenAll

## 16. Liên hệ Unity

- Sinh map/chunk: `Parallel` trên data thuần; apply mesh trên main thread.
- Nhiều web request: UniTask.WhenAll tương đương.
- Tránh Parallel sửa `List` shared không an toàn — dùng concurrent hoặc local buffer.

## 17. Kiểm tra kiến thức

1. WhenAll trên 3 Delay 1s mất khoảng bao lâu?  
   **Đáp án:** ~1 giây (song song), không phải 3.

2. WhenAny trả về gì?  
   **Đáp án:** Task hoàn thành đầu tiên (cần await tiếp để lấy kết quả/exception).

3. Parallel phù hợp workload nào?  
   **Đáp án:** CPU-bound chia được thành phần độc lập.

4. Vì sao Parallel.For trên HttpClient kém?  
   **Đáp án:** Chặn thread pool chờ IO; async mới đúng.

5. Làm gì với các task thua sau WhenAny?  
   **Đáp án:** Cancel nếu được và/hoặc observe để không mất exception.
