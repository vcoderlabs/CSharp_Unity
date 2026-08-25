# Chương 2 — async / await và state machine

## 1. Mục tiêu học

- Hiểu `async`/`await` **không tạo thread mới** tự động
- Hình dung compiler biến method `async` thành **state machine**
- Phân biệt `async Task`, `async Task<T>`, `async void`
- Biết continuation chạy ở đâu sau `await`

## 2. Điều kiện tiên quyết

- Chương 1: Thread vs Task
- Level 6: try/catch/finally
- Level 7: callback mindset

## 3. Khái niệm

`await` trên một `Task` (hoặc awaitable): nếu chưa xong, method **tạm dừng logic**, đăng ký continuation, **trả quyền điều khiển** cho caller. Khi Task hoàn thành, continuation chạy tiếp phần sau `await`.

Compiler sinh một **struct/class state machine** (`IAsyncStateMachine`) với các state (`0`, `1`, …) tương ứng mỗi `await`.

```text
async Task FooAsync() {
  A();
  await SomethingAsync();  // ← điểm yield
  B();
}
```

Tương đương ý tưởng: chạy `A`, nếu `Something` chưa xong thì return Task chưa hoàn thành; sau đó chạy `B` rồi complete Task của `FooAsync`.

**`async void`**: không await được, exception khó bắt — chỉ chấp nhận cho event handler UI cũ.

## 4. Mô hình tư duy

```text
Caller
  └─ await FooAsync()
        state 0: code trước await
        await bar → unfinished? return incomplete Task
        (sau này) state 1: code sau await
        SetResult / SetException
```

IO async: thread pool **không** bị chiếm trong lúc chờ socket. CPU sync trong `async` method vẫn chạy trên thread đang thực thi (có thể là pool hoặc UI).

## 5. Cú pháp

Khai báo và gọi async method chuẩn:

```csharp
async Task DoWorkAsync()
{
    Console.WriteLine("bắt đầu");
    await Task.Delay(100);
    Console.WriteLine("xong");
}

async Task<int> GetNumberAsync()
{
    await Task.Delay(50);
    return 42;
}

// Gọi
await DoWorkAsync();
int n = await GetNumberAsync();
```

`ConfigureAwait(false)`: không marshal về SynchronizationContext gốc (thư viện; console thường không có context).

```csharp
await SomeIoAsync().ConfigureAwait(false);
```

## 6. Ví dụ

### Cơ bản

Method async trả về chuỗi sau delay:

```csharp
static async Task<string> LoadNameAsync()
{
    await Task.Delay(100);
    return "Ada";
}

string name = await LoadNameAsync();
Console.WriteLine(name);
```

### Trung cấp

Nhiều await tuần tự + try/catch:

```csharp
static async Task<string> PipelineAsync()
{
    try
    {
        var a = await StepAsync("A");
        var b = await StepAsync(a + "-B");
        return b;
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Lỗi: {ex.Message}");
        throw;
    }
}

static async Task<string> StepAsync(string s)
{
    await Task.Delay(30);
    if (s.Contains("fail")) throw new InvalidOperationException("boom");
    return s;
}
```

### Nâng cao

State machine “nhìn thấy” qua nhiều await và biến local sống qua await:

```csharp
static async Task DemoLocalsAcrossAwait()
{
    int x = 10; // compiler nâng thành field của state machine
    await Task.Delay(1);
    x += 5;
    await Task.Delay(1);
    Console.WriteLine(x); // 15
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| CS4008 / không await được | Thiếu `async` hoặc không trả Task | `async Task` |
| Exception nuốt | `async void` | Đổi `async Task` |
| Deadlock UI | `.Result` / `.Wait()` trên UI + sync context | Chỉ `await` |
| Warning CS4014 | Fire-and-forget không await | `await` hoặc `_ =` có chủ đích + log lỗi |
| Nghĩ async = nhanh hơn CPU | Async không song song hóa CPU | Offload bằng `Task.Run` nếu cần |

## 8. Gỡ lỗi

1. Breakpoint sau `await` — xem call stack “async”.
2. Exception: nhìn **InnerException** / Aggregate trên `WhenAll`.
3. Bật Break on Thrown cho `Exception` khi nghi nuốt lỗi `async void`.

## 9. Best practices

- API public: `Task` / `Task<T>` / `ValueTask` — **không** `async void`.
- Đặt tên method kết thúc `Async`.
- Tránh `async` nếu method chỉ `return Task.CompletedTask` / `Task.FromResult` không await — trả Task trực tiếp.
- Thư viện: cân nhắc `ConfigureAwait(false)`.
- Luôn propagate exception; log ở biên (UI/host).

## 10. Bài tập

**Bài 1** — Viết `FetchAsync` trả `Task<string>` giả lập 3 bước await Delay.

**Bài 2** — So sánh: gọi `async void` vs `async Task` và cố tình throw — quan sát hành vi.

**Bài 3** — Viết `RetryAsync(Func<Task>, int times)` retry khi exception.

**Bài 4** — Method có 2 await; đo thread ID trước mỗi await và sau mỗi await.

## 11. Gợi ý

- Bài 1: chuỗi `await Task.Delay` + nối string.
- Bài 2: `try/catch` quanh `await TaskMethod()`; `async void` khó catch từ caller.
- Bài 3: vòng for + try/catch + delay backoff.
- Bài 4: `Environment.CurrentManagedThreadId`.

## 12. Đáp án

**Bài 1** — Ba bước giả lập:

```csharp
static async Task<string> FetchAsync()
{
    await Task.Delay(50);
    await Task.Delay(50);
    await Task.Delay(50);
    return "done";
}
```

**Bài 2** — Task bắt được; void thì caller không await/catch được:

```csharp
static async Task BoomTask() { await Task.Delay(1); throw new Exception("task"); }
static async void BoomVoid() { await Task.Delay(1); throw new Exception("void"); }

try { await BoomTask(); } catch (Exception ex) { Console.WriteLine(ex.Message); }
BoomVoid(); // exception trên sync context / process — nguy hiểm
```

**Bài 3** — Retry đơn giản:

```csharp
static async Task RetryAsync(Func<Task> action, int times)
{
    for (int i = 1; ; i++)
    {
        try
        {
            await action();
            return;
        }
        catch when (i < times)
        {
            await Task.Delay(100 * i);
        }
    }
}
```

**Bài 4** — Log thread hop (tự viết print quanh các await như mục 6 nâng cao).

## 13. Đáp án thay thế

Bài 3 dùng Polly library; hoặc `ValueTask` nếu hot path. Retry có thể nhận `CancellationToken`.

## 14. Thử thách

Dùng [ILSpy](https://github.com/icsharpcode/ILSpy) / Rider IL xem method `async` — tìm struct state machine và các field `<x>5__1`. Viết ghi chú 5 dòng.

## 15. Ứng dụng thực tế

- ASP.NET Core action `async Task<IActionResult>`
- HttpClient / DB driver async end-to-end
- Desktop: giữ UI responsive khi tải dữ liệu

## 16. Liên hệ Unity

- Unity 2023+ / Player loop: `await` với `UniTask` hoặc `Awaitable` — cẩn thận sync context.
- `async void` Start() từng phổ biến — vẫn khó bắt lỗi; prefer UniTask void có exception handler.
- Sau `await` trên pool: **không** gọi `transform.position` — quay về main thread.
- Coroutine có thể kết hợp nhưng async hiện đại thường rõ hơn cho IO.

## 17. Kiểm tra kiến thức

1. `await` có tạo thread mới không?  
   **Đáp án:** Không bắt buộc — chỉ lên lịch continuation khi awaitable hoàn thành.

2. Compiler làm gì với `async` method?  
   **Đáp án:** Sinh state machine với các state tại mỗi `await`.

3. Vì sao tránh `async void`?  
   **Đáp án:** Không await được; exception khó observe/propagate.

4. `.Result` trên UI thread nguy hiểm vì sao?  
   **Đáp án:** Có thể deadlock với SynchronizationContext.

5. `ConfigureAwait(false)` dùng để làm gì?  
   **Đáp án:** Không bắt buộc tiếp tục trên sync context gốc — giảm deadlock/chi phí (thư viện).
