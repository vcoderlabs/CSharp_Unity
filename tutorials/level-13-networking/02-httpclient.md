# Chương 2 — HttpClient

## 1. Mục tiêu học

- Tạo và **tái sử dụng** `HttpClient`
- Gửi GET/POST với `HttpRequestMessage` / helpers
- Đọc response: status, headers, content
- Timeout, cancel, lỗi mạng vs HTTP error

## 2. Điều kiện tiên quyết

- Chương 1: HTTP basics
- Level 11: async, CancellationToken

## 3. Khái niệm

`HttpClient` là API .NET để gửi HTTP. Handler pipeline bên dưới (`HttpClientHandler` / `SocketsHttpHandler`).

**Anti-pattern:** `using var client = new HttpClient()` mỗi lần gọi → không đóng socket kịp → `SocketException` / hết ephemeral port.

**Đúng:** một instance lâu dài; hoặc `IHttpClientFactory` trong generic host/ASP.NET.

`EnsureSuccessStatusCode()` ném nếu không 2xx — hoặc tự kiểm `IsSuccessStatusCode`.

## 4. Mô hình tư duy

```text
HttpClient (reuse)
   └─ GetAsync / SendAsync
         └─ HttpResponseMessage
               └─ Content.ReadAsStringAsync / ReadAsStreamAsync
```

BaseAddress + relative URI giúp gọn API client.

## 5. Cú pháp

Client tái sử dụng và GET:

```csharp
static readonly HttpClient Http = new()
{
    BaseAddress = new Uri("https://httpbin.org/"),
    Timeout = TimeSpan.FromSeconds(30),
};

using var response = await Http.GetAsync("get", cancellationToken);
response.EnsureSuccessStatusCode();
string body = await response.Content.ReadAsStringAsync(cancellationToken);
Console.WriteLine(body);
```

POST JSON:

```csharp
using var content = new StringContent("""{"name":"Ada"}""", Encoding.UTF8, "application/json");
using var response = await Http.PostAsync("post", content, cancellationToken);
```

## 6. Ví dụ

### Cơ bản

In status code:

```csharp
using var res = await Http.GetAsync("https://example.com");
Console.WriteLine((int)res.StatusCode);
```

### Trung cấp

Header tùy chỉnh:

```csharp
using var req = new HttpRequestMessage(HttpMethod.Get, "https://httpbin.org/headers");
req.Headers.TryAddWithoutValidation("X-Debug", "1");
using var res = await Http.SendAsync(req);
Console.WriteLine(await res.Content.ReadAsStringAsync());
```

### Nâng cao

Stream response (không load hết RAM):

```csharp
using var res = await Http.GetAsync(url, HttpCompletionOption.ResponseHeadersRead, ct);
await using var stream = await res.Content.ReadAsStreamAsync(ct);
await stream.CopyToAsync(File.Create("out.bin"), ct);
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Socket exhaustion | new client mỗi lần | Static/singleton/factory |
| TaskCanceledException | Timeout hoặc ct | Phân biệt timeout vs cancel |
| Bỏ qua SSL | Handler bypass | Chỉ lab; production đừng |
| Nuốt non-2xx | Không check status | `EnsureSuccessStatusCode` hoặc handle |

## 8. Gỡ lỗi

1. Log method + URL + status.
2. Đọc body lỗi 4xx (thường JSON message).
3. `HttpClient.Timeout` vs CTS riêng.

## 9. Best practices

- Reuse client; set `DefaultRequestHeaders` cẩn thận (thread-safe hạn chế — ưu tiên per-request).
- `ResponseHeadersRead` cho download lớn.
- Dispose `HttpResponseMessage` / content khi xong.
- Console app: `static readonly HttpClient`.
- ASP.NET: `IHttpClientFactory.CreateClient("name")`.

## 10. Bài tập

**Bài 1** — GET `https://httpbin.org/get`, in status + 100 ký tự đầu body.

**Bài 2** — GET với query `?hello=world` (UriBuilder hoặc string).

**Bài 3** — Cố ý URL sai DNS — bắt exception, in loại.

**Bài 4** — Timeout 1s gọi delay httpbin `/delay/5` — quan sát hủy.

## 11. Gợi ý

- Bài 1: mục 5.
- Bài 2: `get?hello=world`.
- Bài 3: try/catch `HttpRequestException`.
- Bài 4: `Timeout = TimeSpan.FromSeconds(1)`.

## 12. Đáp án

**Bài 1** — GET cơ bản:

```csharp
using var res = await Http.GetAsync("https://httpbin.org/get");
Console.WriteLine(res.StatusCode);
var text = await res.Content.ReadAsStringAsync();
Console.WriteLine(text[..Math.Min(100, text.Length)]);
```

**Bài 2** — Query:

```csharp
using var res = await Http.GetAsync("https://httpbin.org/get?hello=world");
```

**Bài 3** — DNS fail:

```csharp
try
{
    using var res = await Http.GetAsync("https://no-such-host-abcxyz.example");
}
catch (HttpRequestException ex)
{
    Console.WriteLine(ex.GetType().Name + ": " + ex.Message);
}
```

**Bài 4** — Timeout:

```csharp
using var shortTimeout = new HttpClient { Timeout = TimeSpan.FromSeconds(1) };
try
{
    using var res = await shortTimeout.GetAsync("https://httpbin.org/delay/5");
}
catch (TaskCanceledException)
{
    Console.WriteLine("Timeout/cancel");
}
```

## 13. Đáp án thay thế

`HttpClient.GetFromJsonAsync<T>` (extension `System.Net.Http.Json`) — chương 3 dùng nhiều hơn.

## 14. Thử thách

Custom `DelegatingHandler` log mọi request duration — gắn `HttpClient(handler)`.

## 15. Ứng dụng thực tế

- Microservice gọi nhau
- Download Manager (Level 11 project)
- Webhook sender

## 16. Liên hệ Unity

- `UnityWebRequest` song song với HttpClient (IL2CPP/platform).
- Một số platform hạn chế socket — test device thật.
- Vẫn áp dụng: không spam tạo client; cancel khi scene unload.

## 17. Kiểm tra kiến thức

1. Vì sao không `new HttpClient` mỗi request?  
   **Đáp án:** Cạn socket / chậm; nên reuse.

2. `EnsureSuccessStatusCode` làm gì?  
   **Đáp án:** Ném nếu status không thành công (2xx).

3. `HttpCompletionOption.ResponseHeadersRead` giúp gì?  
   **Đáp án:** Trả về khi có headers — stream body sau, tiết kiệm RAM.

4. Timeout mặc định HttpClient?  
   **Đáp án:** 100 giây (có thể đổi).

5. ASP.NET nên dùng gì thay static client?  
   **Đáp án:** `IHttpClientFactory`.
