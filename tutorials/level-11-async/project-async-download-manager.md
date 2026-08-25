# Project Level 11 — Async Download Manager

## 1. Mục tiêu học

- Ghép `HttpClient` (hoặc giả lập IO) + `async`/`await` + `WhenAll`
- Giới hạn concurrency bằng `SemaphoreSlim`
- Hủy bằng `CancellationToken` (Ctrl+C)
- Báo cáo tiến độ thread-safe (`Interlocked` / channel log)
- Tránh race khi ghi file / cập nhật trạng thái

## 2. Điều kiện tiên quyết

- Hoàn thành 7 chương Level 11
- .NET 8 console: `dotnet new console -n AsyncDownloadManager -f net8.0`
- Biết HTTP cơ bản (Level 13 sẽ sâu hơn — ở đây đủ dùng `HttpClient.GetAsync`)

## 3. Khái niệm / Yêu cầu sản phẩm

Xây CLI **Async Download Manager**:

| Tính năng | Bắt buộc |
|-----------|----------|
| Đọc danh sách URL từ file `urls.txt` (một URL/dòng) | Có |
| Tải song song, **max N** (mặc định 4) | Có |
| Lưu vào thư mục `downloads/` | Có |
| Progress: số hoàn thành / tổng, byte (nếu có) | Có |
| Ctrl+C hủy còn lại sạch sẽ | Có |
| Log lỗi từng URL không crash toàn bộ | Có |
| Retry 1–2 lần (tuỳ chọn) | Bonus |

Chấp nhận chế độ **Demo** không mạng: URL dạng `demo://delay/{ms}/{name}` sleep rồi ghi file giả — để chấm bài offline.

## 4. Mô hình tư duy

```text
urls.txt → parse list
         → SemaphoreSlim(max)
         → foreach url: WaitAsync → DownloadOneAsync → Release
         → WhenAll
Ctrl+C → cts.Cancel()
Mỗi download: HttpCompletionOption.ResponseHeadersRead + CopyToAsync(stream, ct)
State: ConcurrentDictionary<url, Status> hoặc record immutable updates
```

## 5. Cú pháp / Skeleton

Tạo project và khung Program:

```bash
dotnet new console -n AsyncDownloadManager -f net8.0
cd AsyncDownloadManager
```

```csharp
using var cts = new CancellationTokenSource();
Console.CancelKeyPress += (_, e) =>
{
    e.Cancel = true;
    cts.Cancel();
};

var urls = await File.ReadAllLinesAsync("urls.txt", cts.Token);
var manager = new DownloadManager(maxConcurrent: 4, outputDir: "downloads");
await manager.DownloadAllAsync(urls, cts.Token);
```

## 6. Ví dụ hướng dẫn

`DownloadManager` lõi:

```csharp
sealed class DownloadManager
{
    private readonly HttpClient _http = new() { Timeout = TimeSpan.FromMinutes(5) };
    private readonly int _max;
    private readonly string _dir;

    public DownloadManager(int maxConcurrent, string outputDir)
    {
        _max = maxConcurrent;
        _dir = outputDir;
        Directory.CreateDirectory(_dir);
    }

    public async Task DownloadAllAsync(IEnumerable<string> urls, CancellationToken ct)
    {
        var list = urls.Where(u => !string.IsNullOrWhiteSpace(u)).Select(u => u.Trim()).ToList();
        int done = 0;
        using var sem = new SemaphoreSlim(_max, _max);

        var tasks = list.Select(async url =>
        {
            await sem.WaitAsync(ct);
            try
            {
                await DownloadOneAsync(url, ct);
            }
            catch (OperationCanceledException) { throw; }
            catch (Exception ex)
            {
                Console.WriteLine($"FAIL {url}: {ex.Message}");
            }
            finally
            {
                sem.Release();
                int d = Interlocked.Increment(ref done);
                Console.WriteLine($"Progress {d}/{list.Count}");
            }
        });

        try { await Task.WhenAll(tasks); }
        catch (OperationCanceledException) { Console.WriteLine("Cancelled."); }
    }

    async Task DownloadOneAsync(string url, CancellationToken ct)
    {
        if (url.StartsWith("demo://", StringComparison.OrdinalIgnoreCase))
        {
            await DemoAsync(url, ct);
            return;
        }

        using var resp = await _http.GetAsync(url, HttpCompletionOption.ResponseHeadersRead, ct);
        resp.EnsureSuccessStatusCode();
        var name = Path.GetFileName(new Uri(url).LocalPath);
        if (string.IsNullOrEmpty(name)) name = Guid.NewGuid().ToString("N");
        var path = Path.Combine(_dir, name);
        await using var fs = new FileStream(path, FileMode.Create, FileAccess.Write, FileShare.None, 81920, true);
        await resp.Content.CopyToAsync(fs, ct);
    }

    static async Task DemoAsync(string url, CancellationToken ct)
    {
        // demo://delay/300/file.bin
        var parts = url["demo://".Length..].Split('/');
        int ms = parts is [ "delay", var m, .. ] && int.TryParse(m, out var x) ? x : 200;
        string file = parts.Length >= 3 ? parts[^1] : "demo.bin";
        await Task.Delay(ms, ct);
        await File.WriteAllTextAsync(Path.Combine("downloads", file), "demo", ct);
    }
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Vẫn tải sau Ctrl+C | Không truyền `ct` xuống GetAsync/Copy | Truyền token xuyên suốt |
| File hỏng khi cancel | Không xóa file dở | `try/finally` xóa nếu !completed |
| Hết socket | `new HttpClient` mỗi request | Một instance dùng lại |
| Race tên file | Cùng LocalPath | Thêm hash/host vào tên |
| Progress sai | `done++` không Interlocked | `Interlocked.Increment` |

## 8. Gỡ lỗi

1. Chế độ demo trước — không phụ thuộc mạng.
2. Log thread id nếu nghi cập nhật console loạn (thường OK).
3. Fiddler/Wireshark chỉ khi debug HTTP thật.

## 9. Best practices

- Một `HttpClient` (hoặc `IHttpClientFactory` trong host lớn).
- `ResponseHeadersRead` + stream — không `ReadAsByteArray` file lớn.
- Giới hạn concurrency luôn.
- Phân biệt cancel vs fail trong báo cáo cuối.
- `ConfigureAwait(false)` trong thư viện manager nếu tách lib.

## 10. Bài tập (milestone)

1. **M1** — Đọc `urls.txt`, in danh sách, tạo `downloads/`.
2. **M2** — Download tuần tự async + progress.
3. **M3** — SemaphoreSlim max=4 + WhenAll.
4. **M4** — Ctrl+C cancel; xóa file incomplete.
5. **M5** — Báo cáo cuối: OK / Fail / Cancelled counts.

## 11. Gợi ý

- M1: `File.ReadAllLinesAsync`.
- M2: `foreach` + `await DownloadOne`.
- M3: pattern mục 6.
- M4: `CancelKeyPress` + temp file `.partial` rồi rename.
- M5: `ConcurrentDictionary` hoặc `Interlocked` counters.

## 12. Đáp án

Cốt lõi nằm ở mục 6. Bổ sung đếm trạng thái:

```csharp
int ok = 0, fail = 0;
// trong success: Interlocked.Increment(ref ok);
// trong catch (không phải cancel): Interlocked.Increment(ref fail);
Console.WriteLine($"OK={ok} FAIL={fail}");
```

File partial:

```csharp
var partial = path + ".partial";
await using (var fs = new FileStream(partial, FileMode.Create, FileAccess.Write, FileShare.None, 81920, true))
{
    await resp.Content.CopyToAsync(fs, ct);
}
File.Move(partial, path, overwrite: true);
```

Trong `catch (OperationCanceledException)`: `File.Delete(partial)` nếu tồn tại.

## 13. Đáp án thay thế

Dùng `Parallel.ForEachAsync(urls, new ParallelOptions { MaxDegreeOfParallelism = 4, CancellationToken = ct }, ...)`.  
Hoặc Channel pipeline: url writer → N workers reader.

## 14. Thử thách

- Retry exponential backoff 2 lần cho 5xx/timeout.
- Resume download (Range headers) — nâng cao.
- Hiển thị tốc độ KB/s bằng cách đọc stream thủ công từng buffer.

## 15. Ứng dụng thực tế

- CDN mirror / asset prefetch tools
- CI tải artifact song song
- Game launcher tải patch (cùng ý tưởng + checksum)

## 16. Liên hệ Unity

- Addressables / patcher: giới hạn concurrent download trên mobile.
- Progress bar: marshal % về main thread.
- Cancel khi user bấm Back — `destroyCancellationToken` / CTS gắn UI.

## 17. Kiểm tra kiến thức

1. Vì sao một `HttpClient` dùng lại?  
   **Đáp án:** Tránh socket exhaustion / tăng hiệu năng DNS/handler.

2. SemaphoreSlim trong project này làm gì?  
   **Đáp án:** Giới hạn số download đồng thời.

3. Token phải tới đâu?  
   **Đáp án:** `GetAsync`, `CopyToAsync`, `Delay`, mọi await dài.

4. Progress `done++` đa task sai vì sao?  
   **Đáp án:** Race — cần `Interlocked` hoặc lock.

5. Cancel giữa chừng nên xử lý file thế nào?  
   **Đáp án:** Xóa hoặc không rename file `.partial` dở.
