# Chương 3 — Task\<T\> và ValueTask

## 1. Mục tiêu học

- Thành thạo `Task` / `Task<T>`: tạo, hoàn thành, lỗi, hủy
- Đọc trạng thái `IsCompleted`, `IsFaulted`, `Status`
- Hiểu `ValueTask` / `ValueTask<T>`: khi nào có lợi, khi nào tránh
- Tránh anti-pattern: `GetAwaiter().GetResult()`, double-await ValueTask

## 2. Điều kiện tiên quyết

- Chương 1–2: Task, async/await
- Level 5: generics (`Task<T>`)
- Level 6: exception propagation

## 3. Khái niệm

**`Task`**: đại diện thao tác không trả giá trị (hoặc chỉ side-effect).  
**`Task<T>`**: thao tác trả `T` khi thành công.

Tạo sẵn kết quả không cần async state machine:

```csharp
Task.FromResult(42);
Task.CompletedTask;
Task.FromException(new Exception("x"));
Task.FromCanceled(token);
```

**`ValueTask` / `ValueTask<T>`**: struct có thể wrap kết quả **đồng bộ đã có** (tránh allocate `Task`) hoặc wrap `Task`/`IValueTaskSource`. Phù hợp hot path (cache hit). **Không** await hai lần cùng một instance; **không** dùng khi API phức tạp trừ khi đo được lợi ích.

## 4. Mô hình tư duy

```text
Task<T>:  reference type, có thể cache/shared, await nhiều lần OK (cùng kết quả)

ValueTask<T>:
  - đồng bộ: chứa T trực tiếp (0 alloc)
  - bất đồng bộ: chứa Task hoặc IValueTaskSource
  - await một lần; rồi bỏ
```

Quy tắc thực dụng: **mặc định `Task<T>`**; chỉ đổi `ValueTask` khi profiling chỉ ra allocation nóng.

## 5. Cú pháp

Tạo Task bằng `Task.Run`, factory, và ValueTask từ cache:

```csharp
Task<int> t1 = Task.Run(() => Compute());
Task<int> t2 = Task.FromResult(10);

ValueTask<int> GetCachedOrLoad(bool hit)
{
    if (hit) return new ValueTask<int>(99);          // sync
    return new ValueTask<int>(LoadAsync());          // async Task
}

async Task<int> LoadAsync()
{
    await Task.Delay(10);
    return 1;
}
```

`TaskCompletionSource<T>`: hoàn thành Task thủ công (bridge callback → Task):

```csharp
var tcs = new TaskCompletionSource<int>(TaskCreationOptions.RunContinuationsAsynchronously);
tcs.TrySetResult(5);
int v = await tcs.Task;
```

## 6. Ví dụ

### Cơ bản

Await `Task<string>`:

```csharp
static async Task<string> ReadAsync()
{
    await Task.Delay(20);
    return "data";
}

string s = await ReadAsync();
```

### Trung cấp

Xử lý faulted Task:

```csharp
static async Task DemoFault()
{
    Task<int> bad = Task.FromException<int>(new InvalidOperationException("fail"));
    try
    {
        _ = await bad;
    }
    catch (InvalidOperationException ex)
    {
        Console.WriteLine(ex.Message);
    }
}
```

### Nâng cao

API hot-path với ValueTask + cache:

```csharp
sealed class PriceService
{
    private int? _cache;

    public ValueTask<int> GetPriceAsync()
    {
        if (_cache is int c) return new ValueTask<int>(c);
        return new ValueTask<int>(LoadCoreAsync());
    }

    async Task<int> LoadCoreAsync()
    {
        await Task.Delay(30);
        _cache = 100;
        return 100;
    }
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| `InvalidOperationException` ValueTask | Await 2 lần / `.Result` rồi await | Chỉ consume một lần; dùng `AsTask()` nếu cần share |
| Alloc vẫn cao | ValueTask luôn async | Không giúp — giữ `Task` |
| Deadlock | `.Result` / `.Wait()` | `await` |
| Mất exception | Không await Task | Luôn observe Task |
| `TaskCompletionSource` continuation trên thread lạ | SetResult đồng bộ | `RunContinuationsAsynchronously` |

## 8. Gỡ lỗi

1. Inspect `task.Status` / `Exception` trong debugger.
2. `await task` trong try — đừng chỉ check `IsFaulted` rồi quên throw.
3. Benchmark: `BenchmarkDotNet` so `Task` vs `ValueTask` trên cache hit/miss.

## 9. Best practices

- Public API: `Task`/`Task<T>` trừ khi bạn là maintainer hot path đã đo.
- Prefer `await` thay sync-over-async.
- `Task.WhenAll` nhận `Task[]` — convert ValueTask bằng `AsTask()` nếu cần.
- `TaskCreationOptions.LongRunning` chỉ khi thật sự chiếm thread lâu (hiếm).
- Trả `Task.FromResult` thay vì `async` method không có await.

## 10. Bài tập

**Bài 1** — Viết `ParseIntAsync(string)` trả `Task<int>`; chuỗi lỗi → faulted task.

**Bài 2** — Dùng `TaskCompletionSource<string>` giả lập callback hoàn thành sau 100ms.

**Bài 3** — Class cache `GetAsync(key)` trả `ValueTask<string>` (hit sync / miss async).

**Bài 4** — So sánh `IsCompletedSuccessfully` trước khi await.

## 11. Gợi ý

- Bài 1: try parse; fail thì `Task.FromException` hoặc throw trong async.
- Bài 2: `Task.Delay(100).ContinueWith(_ => tcs.TrySetResult("ok"))` hoặc async fire.
- Bài 3: `Dictionary` + như mục 6.
- Bài 4: `if (task.IsCompletedSuccessfully) return task.Result;` else await — cẩn thận race; chủ yếu học API.

## 12. Đáp án

**Bài 1** — Parse an toàn:

```csharp
static Task<int> ParseIntAsync(string s)
{
    if (int.TryParse(s, out int v)) return Task.FromResult(v);
    return Task.FromException<int>(new FormatException($"Không parse: {s}"));
}
```

**Bài 2** — TCS + Delay:

```csharp
static Task<string> After100MsAsync()
{
    var tcs = new TaskCompletionSource<string>(TaskCreationOptions.RunContinuationsAsynchronously);
    _ = Complete();
    return tcs.Task;

    async Task Complete()
    {
        await Task.Delay(100);
        tcs.TrySetResult("ok");
    }
}
```

**Bài 3** — Cache ValueTask:

```csharp
sealed class StringCache
{
    private readonly Dictionary<string, string> _map = new();

    public ValueTask<string> GetAsync(string key)
    {
        if (_map.TryGetValue(key, out var v)) return new ValueTask<string>(v);
        return new ValueTask<string>(LoadAsync(key));
    }

    async Task<string> LoadAsync(string key)
    {
        await Task.Delay(20);
        var v = $"value:{key}";
        _map[key] = v;
        return v;
    }
}
```

**Bài 4** — Kiểm tra hoàn thành sớm:

```csharp
static async Task<int> UnwrapAsync(Task<int> task)
{
    if (task.IsCompletedSuccessfully) return task.Result;
    return await task.ConfigureAwait(false);
}
```

## 13. Đáp án thay thế

Bài 1 dùng `async Task<int>` + `throw` — đơn giản hơn cho người mới. Bài 3 dùng `IMemoryCache`.

## 14. Thử thách

Implement `IValueTaskSource<int>` tối giản (advanced) hoặc đọc docs — tóm tắt 3 rule ValueTask bằng tiếng Việt.

## 15. Ứng dụng thực tế

- `HttpClient.GetAsync` → `Task<HttpResponseMessage>`
- Channel / pipeline: `TaskCompletionSource` cầu nối
- Kestrel / BCL hot path: `ValueTask` để giảm GC

## 16. Liên hệ Unity

- Addressables / UnityWebRequest: wrap callback → `Task`/`UniTask` bằng TCS tương đương.
- Tránh `.Result` trên main thread khi task cần main để complete.
- `ValueTask` ít gặp trong gameplay scripts — `UniTask` phổ biến hơn trong ecosystem Unity.

## 17. Kiểm tra kiến thức

1. `Task.FromResult` dùng khi nào?  
   **Đáp án:** Đã có kết quả đồng bộ, cần trả `Task<T>` mà không chạy async thật.

2. Khác biệt lớn Task vs ValueTask?  
   **Đáp án:** ValueTask là struct, tối ưu sync-complete; hạn chế consume một lần.

3. Vì sao không await ValueTask hai lần?  
   **Đáp án:** Có thể invalid / undefined sau lần consume đầu (đặc biệt IValueTaskSource).

4. `TaskCompletionSource` giải quyết gì?  
   **Đáp án:** Biến callback/event hoàn thành thành `Task` awaitable.

5. Mặc định nên expose `Task` hay `ValueTask` trên API app?  
   **Đáp án:** `Task`/`Task<T>` — ValueTask khi đã đo và cần.
