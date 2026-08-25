# Chương 3 — REST API

## 1. Mục tiêu học

- Hiểu phong cách REST: resource, URL, method
- Gọi API với JSON bằng `System.Net.Http.Json`
- Map DTO ↔ JSON; xử lý 4xx có body lỗi
- Thiết kế lớp `ApiClient` mỏng, testable

## 2. Điều kiện tiên quyết

- Chương 2: HttpClient
- Level 12: System.Text.Json

## 3. Khái niệm

**REST** (thực dụng): tài nguyên có URL; thao tác qua HTTP methods; thường JSON.

```text
GET    /products       → danh sách
GET    /products/{id}  → một sp
POST   /products       → tạo (body JSON)
PUT    /products/{id}  → thay thế
PATCH  /products/{id}  → sửa một phần
DELETE /products/{id}  → xóa
```

Không phải API nào cũng “thuần REST” — quan trọng là **docs** và status code nhất quán.

`GetFromJsonAsync` / `PostAsJsonAsync` nằm trong package/extension `System.Net.Http.Json` (.NET có sẵn).

## 4. Mô hình tư duy

```text
UI/CLI → ApiClient → HttpClient → HTTP → API
              ↑ DTO + JsonSerializerOptions
```

Tách **DTO wire** khỏi model domain nếu API đổi field thường xuyên.

## 5. Cú pháp

GET/POST JSON:

```csharp
using System.Net.Http.Json;

var products = await Http.GetFromJsonAsync<List<ProductDto>>("https://jsonplaceholder.typicode.com/posts?_limit=3");

var created = await Http.PostAsJsonAsync("https://jsonplaceholder.typicode.com/posts",
    new { title = "foo", body = "bar", userId = 1 });
created.EnsureSuccessStatusCode();
var post = await created.Content.ReadFromJsonAsync<PostDto>();
```

## 6. Ví dụ

### Cơ bản

DTO và GET:

```csharp
public record PostDto(int UserId, int Id, string Title, string Body);

var post = await Http.GetFromJsonAsync<PostDto>("https://jsonplaceholder.typicode.com/posts/1");
Console.WriteLine(post?.Title);
```

### Trung cấp

Kiểm tra lỗi có body:

```csharp
using var res = await Http.GetAsync("https://httpbin.org/status/404");
if (!res.IsSuccessStatusCode)
{
    string err = await res.Content.ReadAsStringAsync();
    Console.WriteLine($"HTTP {(int)res.StatusCode}: {err}");
}
```

### Nâng cao

ApiClient nhỏ:

```csharp
sealed class PostsClient
{
    private readonly HttpClient _http;
    private readonly JsonSerializerOptions _json = new()
    {
        PropertyNameCaseInsensitive = true,
        PropertyNamingPolicy = JsonNamingPolicy.CamelCase,
    };

    public PostsClient(HttpClient http) => _http = http;

    public async Task<PostDto?> GetAsync(int id, CancellationToken ct = default)
        => await _http.GetFromJsonAsync<PostDto>($"posts/{id}", _json, ct);

    public async Task<PostDto?> CreateAsync(object body, CancellationToken ct = default)
    {
        using var res = await _http.PostAsJsonAsync("posts", body, _json, ct);
        res.EnsureSuccessStatusCode();
        return await res.Content.ReadFromJsonAsync<PostDto>(_json, ct);
    }
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Null DTO | Sai shape JSON / case | Options case insensitive; kiểm JSON |
| 415 | Sai Content-Type | `application/json` |
| 400 | Validation server | Đọc problem+json body |
| Nối string URL sai | Thiếu encode | `Uri.EscapeDataString` |

## 8. Gỡ lỗi

1. Log request URL + status + body lỗi.
2. So JSON mẫu với DTO property.
3. Dùng httpbin để echo request bạn gửi.

## 9. Best practices

- Một `HttpClient` + `BaseAddress`.
- CancellationToken trên mọi call.
- Không nuốt non-success im lặng.
- Retry chỉ với idempotent + transient (policy riêng).
- Versioning và auth header tập trung một chỗ.

## 10. Bài tập

**Bài 1** — GET list posts (limit 5), in Title.

**Bài 2** — POST tạo post giả, in Id trả về.

**Bài 3** — Viết `GetOrThrowAsync` ném exception custom khi !2xx kèm body.

**Bài 4** — Thêm header `Accept: application/json` default.

## 11. Gợi ý

- Bài 1–2: jsonplaceholder.
- Bài 3: đọc string nếu fail rồi throw.
- Bài 4: `Http.DefaultRequestHeaders.Accept.ParseAdd("application/json")`.

## 12. Đáp án

**Bài 1** — List:

```csharp
var list = await Http.GetFromJsonAsync<List<PostDto>>(
    "https://jsonplaceholder.typicode.com/posts?_limit=5");
foreach (var p in list!) Console.WriteLine(p.Title);
```

**Bài 2** — POST:

```csharp
using var res = await Http.PostAsJsonAsync(
    "https://jsonplaceholder.typicode.com/posts",
    new { title = "t", body = "b", userId = 1 });
res.EnsureSuccessStatusCode();
var dto = await res.Content.ReadFromJsonAsync<PostDto>();
Console.WriteLine(dto?.Id);
```

**Bài 3** — Throw kèm body:

```csharp
static async Task<T> GetOrThrowAsync<T>(HttpClient http, string url, CancellationToken ct)
{
    using var res = await http.GetAsync(url, ct);
    if (res.IsSuccessStatusCode)
        return (await res.Content.ReadFromJsonAsync<T>(cancellationToken: ct))!;
    string body = await res.Content.ReadAsStringAsync(ct);
    throw new InvalidOperationException($"HTTP {(int)res.StatusCode}: {body}");
}
```

**Bài 4** — Accept header:

```csharp
Http.DefaultRequestHeaders.Accept.ParseAdd("application/json");
```

## 13. Đáp án thay thế

Refit / NSwagger generate client từ OpenAPI — production team thường dùng.

## 14. Thử thách

Parse `application/problem+json` (RFC 7807) thành record `ProblemDetails`.

## 15. Ứng dụng thực tế

- Mobile/desktop gọi backend
- Integrator thanh toán / email API
- BFF (Backend for Frontend)

## 16. Liên hệ Unity

- Login, inventory cloud, leaderboard = REST + JSON.
- Cache response; xử lý offline.
- Không tin client: mọi quyền quyết định trên server (anti-cheat).

## 17. Kiểm tra kiến thức

1. REST resource thường map URL thế nào?  
   **Đáp án:** Danh từ số nhiều `/products`, kèm id `/products/{id}`.

2. `PostAsJsonAsync` giúp gì?  
   **Đáp án:** Serialize object sang JSON + set Content-Type + POST.

3. Vì sao cần DTO?  
   **Đáp án:** Khớp shape JSON; tách khỏi model nội bộ.

4. 400 khác 500?  
   **Đáp án:** 400 lỗi client/input; 500 lỗi server.

5. BaseAddress dùng để làm gì?  
   **Đáp án:** Gọi relative path gọn trên một host API.
