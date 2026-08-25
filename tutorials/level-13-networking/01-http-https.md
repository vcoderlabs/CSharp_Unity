# Chương 1 — HTTP và HTTPS

## 1. Mục tiêu học

- Hiểu mô hình request/response: method, URL, header, body, status
- Phân biệt HTTP vs HTTPS (TLS)
- Nhận biết status code nhóm 2xx/4xx/5xx
- Đọc URL: scheme, host, path, query

## 2. Điều kiện tiên quyết

- Level 12: JSON text
- Trình duyệt hoặc `curl` để quan sát

## 3. Khái niệm

**HTTP** — giao thức ứng dụng: client gửi request, server trả response.

```text
GET /users/1 HTTP/1.1
Host: api.example.com
Accept: application/json

HTTP/1.1 200 OK
Content-Type: application/json

{"id":1,"name":"Ada"}
```

| Method | Ý nghĩa thông dụng |
|--------|-------------------|
| GET | Đọc |
| POST | Tạo / submit |
| PUT / PATCH | Cập nhật |
| DELETE | Xóa |

**HTTPS** = HTTP trên **TLS**: mã hóa + xác thực server (certificate). Chống nghe lén/sửa trên đường truyền.

## 4. Mô hình tư duy

```text
Client                Server
  |---- request ------>|
  |<--- response ------|

URL: https://host:443/path?query#fragment
          ^TLS      ^default port HTTPS
```

Idempotent: GET/PUT/DELETE (ý tưởng) — gọi lại “cùng hiệu ứng”; POST thường không.

## 5. Cú pháp

Quan sát bằng curl (không phải C#, nhưng cần biết):

```bash
curl -i https://httpbin.org/get
curl -i -X POST https://httpbin.org/post -H "Content-Type: application/json" -d '{"a":1}'
```

Trong C# bạn sẽ dùng `HttpClient` (chương 2) — ở đây nắm vocabulary.

## 6. Ví dụ

### Cơ bản

Phân tích URL trong code:

```csharp
var uri = new Uri("https://api.example.com:443/v1/items?page=2");
Console.WriteLine(uri.Scheme); // https
Console.WriteLine(uri.Host);   // api.example.com
Console.WriteLine(uri.AbsolutePath); // /v1/items
Console.WriteLine(uri.Query);  // ?page=2
```

### Trung cấp

Bảng status cần nhớ:

| Code | Nghĩa |
|------|--------|
| 200 | OK |
| 201 | Created |
| 204 | No Content |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 429 | Too Many Requests |
| 500 | Server Error |
| 503 | Unavailable |

### Nâng cao

Header quan trọng: `Authorization`, `Content-Type`, `Accept`, `User-Agent`, `Cache-Control`, `ETag`.

```csharp
// Chỉ minh họa khái niệm — gửi thật ở ch.2
Console.WriteLine("Content-Type: application/json");
Console.WriteLine("Authorization: Bearer <token>");
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Mixed content | HTTPS page gọi HTTP | Chỉ HTTPS |
| 404 | Sai path | Kiểm tra URL/version API |
| 401/403 | Thiếu/sai credentials | Token/API key |
| Hiểu POST = luôn tạo | API design khác nhau | Đọc docs API |

## 8. Gỡ lỗi

1. Browser DevTools → Network: xem headers/body.
2. `curl -v` xem TLS handshake lỗi gì.
3. Phân biệt lỗi DNS / TLS / HTTP status.

## 9. Best practices

- Production: HTTPS everywhere.
- Không log token/full Authorization.
- Tôn trọng `Retry-After` khi 429.
- Version API (`/v1/`) khi thiết kế.

## 10. Bài tập

**Bài 1** — Phân tích 3 URL bất kỳ bằng `Uri` (scheme/host/path).

**Bài 2** — Với mỗi status 200/404/500, viết một câu “client nên làm gì”.

**Bài 3** — Dùng curl GET một API công khai, lưu status + `Content-Type`.

**Bài 4** — Giải thích sự khác GET vs POST khi gửi dữ liệu (query vs body).

## 11. Gợi ý

- Bài 1: như mục 6.
- Bài 2: 200 dùng body; 404 báo user; 500 retry có giới hạn.
- Bài 3: `curl -i`.
- Bài 4: GET không nên body (thực tế); POST mang JSON body.

## 12. Đáp án

**Bài 1** — Ví dụ:

```csharp
foreach (var s in new[] {
    "https://example.com/a",
    "http://localhost:5000/health",
    "https://cdn.example.com/x.png?v=3" })
{
    var u = new Uri(s);
    Console.WriteLine($"{u.Scheme} | {u.Host} | {u.AbsolutePath}");
}
```

**Bài 2** — Gợi ý đáp án ngắn: 200 xử lý JSON; 404 không retry vô hạn; 500 backoff retry / báo lỗi.

**Bài 3** — Thực hành tay với curl (không cần code).

**Bài 4** — GET lấy tài nguyên, tham số trên query; POST gửi payload (JSON) trong body, có thể tạo tài nguyên.

## 13. Đáp án thay thế

Dùng Postman / Insomnia / Bruno thay curl.

## 14. Thử thách

Đọc nhanh về TLS certificate chain (root / intermediate / leaf) — viết 5 câu tiếng Việt.

## 15. Ứng dụng thực tế

- Mọi REST/GraphQL API
- OAuth token over HTTPS
- Webhook gọi vào server bạn

## 16. Liên hệ Unity

- UnityWebRequest cũng là HTTP client.
- Mobile: ATS/cleartext — HTTP thường bị chặn, cần HTTPS.
- Login/game shop API: luôn HTTPS + không hardcode secret trong client (client không tin được).

## 17. Kiểm tra kiến thức

1. HTTPS khác HTTP chỗ cốt lõi?  
   **Đáp án:** TLS mã hóa và xác thực server.

2. Status 401 nghĩa là gì?  
   **Đáp án:** Chưa xác thực / token sai (Unauthorized).

3. Method idempotent tiêu biểu?  
   **Đáp án:** GET (và thường PUT/DELETE theo thiết kế).

4. Query string nằm ở đâu trong URL?  
   **Đáp án:** Sau `?` (ví dụ `?page=2`).

5. `Content-Type: application/json` nói gì?  
   **Đáp án:** Body đang là JSON.
