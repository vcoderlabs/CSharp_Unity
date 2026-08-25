# Project Level 13 — REST API Client

## 1. Mục tiêu học

- Viết console **REST client** sạch: `HttpClient` + DTO + JSON
- Hỗ trợ GET list, GET by id, POST create, DELETE (nếu API cho phép)
- Xử lý lỗi HTTP có thông báo rõ
- Cấu hình BaseAddress + timeout + cancel Ctrl+C

## 2. Điều kiện tiên quyết

- Hoàn thành chương 1–3 Level 13 (WS/TCP tùy chọn trước project)
- Level 12 JSON
- `dotnet new console -n RestApiClient -f net8.0`

## 3. Khái niệm / Yêu cầu sản phẩm

Chọn **một** backend:

| Option | URL / ghi chú |
|--------|----------------|
| A (khuyến nghị offline-friendly) | [JSONPlaceholder](https://jsonplaceholder.typicode.com) — posts |
| B | API tự host minimal ASP.NET (nếu bạn đã biết) |
| C | httpbin + fake in-memory (hạn chế DELETE thật) |

**Chức năng CLI:**

```text
list
get <id>
create <title> <body>
delete <id>          # placeholder fake OK
exit
```

**Bắt buộc kỹ thuật:**

- `static`/singleton `HttpClient` với `BaseAddress`
- DTO `Post` record
- `System.Net.Http.Json`
- `CancellationToken` từ Ctrl+C
- In status khi lỗi

## 4. Mô hình tư duy

```text
Program → PostsApi (wrapper) → HttpClient
                ↓
        PostDto / CreatePostRequest
```

JSONPlaceholder: POST/DELETE **fake** (không persist) — vẫn đủ luyện client.

## 5. Cú pháp / Skeleton

```bash
dotnet new console -n RestApiClient -f net8.0
cd RestApiClient
```

```csharp
var http = new HttpClient
{
    BaseAddress = new Uri("https://jsonplaceholder.typicode.com/"),
    Timeout = TimeSpan.FromSeconds(30),
};
http.DefaultRequestHeaders.Accept.ParseAdd("application/json");

using var cts = new CancellationTokenSource();
Console.CancelKeyPress += (_, e) => { e.Cancel = true; cts.Cancel(); };

var api = new PostsApi(http);
// REPL commands...
```

## 6. Ví dụ hướng dẫn

```csharp
public record PostDto(int UserId, int Id, string Title, string Body);
public record CreatePost(string Title, string Body, int UserId);

sealed class PostsApi
{
    private readonly HttpClient _http;
    public PostsApi(HttpClient http) => _http = http;

    public async Task<List<PostDto>> ListAsync(int limit, CancellationToken ct)
    {
        var list = await _http.GetFromJsonAsync<List<PostDto>>($"posts?_limit={limit}", ct);
        return list ?? new();
    }

    public async Task<PostDto?> GetAsync(int id, CancellationToken ct)
        => await _http.GetFromJsonAsync<PostDto>($"posts/{id}", ct);

    public async Task<PostDto?> CreateAsync(CreatePost body, CancellationToken ct)
    {
        using var res = await _http.PostAsJsonAsync("posts", body, ct);
        await EnsureOkAsync(res, ct);
        return await res.Content.ReadFromJsonAsync<PostDto>(cancellationToken: ct);
    }

    public async Task DeleteAsync(int id, CancellationToken ct)
    {
        using var res = await _http.DeleteAsync($"posts/{id}", ct);
        await EnsureOkAsync(res, ct);
    }

    static async Task EnsureOkAsync(HttpResponseMessage res, CancellationToken ct)
    {
        if (res.IsSuccessStatusCode) return;
        string body = await res.Content.ReadAsStringAsync(ct);
        throw new InvalidOperationException($"HTTP {(int)res.StatusCode}: {body}");
    }
}
```

REPL:

```csharp
while (!cts.IsCancellationRequested)
{
    Console.Write("> ");
    string? line = Console.ReadLine();
    if (string.IsNullOrWhiteSpace(line)) continue;
    var parts = line.Split(' ', 3, StringSplitOptions.RemoveEmptyEntries);
    try
    {
        switch (parts[0].ToLowerInvariant())
        {
            case "list":
                foreach (var p in await api.ListAsync(5, cts.Token))
                    Console.WriteLine($"{p.Id}: {p.Title}");
                break;
            case "get" when parts.Length > 1 && int.TryParse(parts[1], out int id):
                Console.WriteLine(await api.GetAsync(id, cts.Token));
                break;
            case "create" when parts.Length >= 3:
                var created = await api.CreateAsync(new CreatePost(parts[1], parts[2], 1), cts.Token);
                Console.WriteLine(created);
                break;
            case "delete" when parts.Length > 1 && int.TryParse(parts[1], out int del):
                await api.DeleteAsync(del, cts.Token);
                Console.WriteLine("deleted (fake on placeholder)");
                break;
            case "exit": return;
            default: Console.WriteLine("commands: list|get|create|delete|exit"); break;
        }
    }
    catch (Exception ex) when (ex is not OperationCanceledException)
    {
        Console.WriteLine("ERR " + ex.Message);
    }
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| SSL/network fail | Firewall / offline | Kiểm net; mock handler |
| Null list | Deserialize fail | Options case insensitive |
| `create` parse sai | Split title/body | Dùng dấu `\|` hoặc JSON input |
| ObjectDisposed HttpClient | using quá sớm | Client sống suốt app |

## 8. Gỡ lỗi

1. In full URL = BaseAddress + relative.
2. Test cùng URL bằng curl.
3. Bật temporary log handler (thử thách ch.2).

## 9. Best practices

- Tách `PostsApi` khỏi UI REPL.
- Exception biên → message user-friendly.
- Không hardcode token; đọc env nếu có auth.
- Unit test bằng `HttpMessageHandler` fake (Level 14).

## 10. Bài tập (milestone)

1. **M1** — GET một post in Title.
2. **M2** — `PostsApi.ListAsync`.
3. **M3** — Create + Delete.
4. **M4** — REPL đủ lệnh.
5. **M5** — Ctrl+C cancel request đang chạy.

## 11. Gợi ý

- M1: `GetFromJsonAsync`.
- M2–3: mục 6.
- M4: switch lệnh.
- M5: truyền `cts.Token` mọi await HTTP.

## 12. Đáp án

Cốt lõi: mục 6. Bổ sung options JSON nếu cần:

```csharp
var json = new JsonSerializerOptions { PropertyNameCaseInsensitive = true };
await _http.GetFromJsonAsync<PostDto>($"posts/{id}", json, ct);
```

## 13. Đáp án thay thế

Dùng OpenAPI + NSwag generate `PostsClient`. Hoặc Refit interface:

```csharp
public interface IPostsApi
{
    [Get("/posts/{id}")]
    Task<PostDto> Get(int id);
}
```

## 14. Thử thách

- Thêm `PATCH` / cập nhật title.
- Basic auth hoặc Bearer từ biến môi trường.
- Export list ra `posts.json` (ôn Level 12).

## 15. Ứng dụng thực tế

- Nội bộ tool gọi API công ty
- Bot admin
- Smoke test API sau deploy

## 16. Liên hệ Unity

- Cùng pattern: service `PostsApi` gọi từ game UI.
- Marshal kết quả về main thread trước khi sửa UI.
- Cancel khi đóng panel / đổi scene.
- Capstone: REST cho account/shop; realtime protocol riêng cho world state.

## 17. Kiểm tra kiến thức

1. BaseAddress nên kết thúc thế nào thường gặp?  
   **Đáp án:** Có `/` cuối; relative path không bắt đầu `/` nếu muốn nối path (hiểu quy tắc Uri combine).

2. JSONPlaceholder Delete có xóa thật không?  
   **Đáp án:** Thường fake 200 — chỉ để test client.

3. Vì sao bọc `PostsApi`?  
   **Đáp án:** Tách HTTP khỏi UI; dễ mock/test.

4. Non-2xx nên làm gì?  
   **Đáp án:** Đọc body lỗi và báo/throw rõ ràng.

5. Ctrl+C liên quan token thế nào?  
   **Đáp án:** `CancelKeyPress` → `cts.Cancel()` → HTTP abort.
