# Level 13 — Networking (~15 giờ)

Level này dạy bạn nền tảng **HTTP/HTTPS**, dùng **`HttpClient` đúng cách**, gọi **REST API**, làm quen **WebSocket**, và khái niệm **TCP/UDP** (liên hệ game multiplayer).

**Điều kiện:** Đã hoàn thành [Level 12 — File/IO/JSON](../level-12-file-io/) và [Level 11 — Async](../level-11-async/). JSON + `async` là xương sống khi gọi API.

**Tiếp theo:** [Level 14 — Testing](../level-14-testing/) (mock HttpMessageHandler).

---

## Mục tiêu cấp độ

Sau Level 13 bạn sẽ:

- Giải thích HTTP method, status code, header, HTTPS TLS ở mức thực dụng
- Dùng `HttpClient` / `IHttpClientFactory` pattern (console: singleton client)
- Gọi REST: GET/POST + JSON body, xử lý lỗi
- Kết nối WebSocket client cơ bản
- Phân biệt TCP vs UDP và liên hệ netcode game
- Xây **REST API Client** console hoàn chỉnh

---

## Cảnh báo xuyên suốt Level 13

> **`new HttpClient()` mỗi request** → socket exhaustion. Dùng lại một instance (hoặc factory trong ASP.NET).  
> Luôn `await` + truyền `CancellationToken`. Đừng bỏ qua HTTPS lỗi certificate trong production.  
> Game multiplayer thật (Unity Netcode, Mirror, dedicated server) phức tạp hơn TCP tutorial — ở đây học **khái niệm** để sau học engine networking.

---

## 5 chương + 1 project

| # | File | Nội dung | ~Giờ |
|---|------|----------|------|
| 1 | [01-http-https.md](./01-http-https.md) | HTTP/HTTPS fundamentals | 2–3 |
| 2 | [02-httpclient.md](./02-httpclient.md) | `HttpClient` đúng cách | 3–4 |
| 3 | [03-rest-api.md](./03-rest-api.md) | REST + JSON | 3–4 |
| 4 | [04-websocket.md](./04-websocket.md) | WebSocket client | 2–3 |
| 5 | [05-tcp-udp-concepts.md](./05-tcp-udp-concepts.md) | TCP/UDP & game multiplayer | 2–3 |
| — | [project-rest-api-client.md](./project-rest-api-client.md) | REST API Client | 3–4 |

**Tổng ước lượng: ~15 giờ**

---

## Cách học đề xuất

1. Chương 1: dùng browser DevTools / `curl` xem request thật.
2. Chương 2: GET https://httpbin.org/get hoặc jsonplaceholder.
3. Chương 3: POST JSON, đọc status + deserialize.
4. Chương 4: public echo WebSocket (hoặc server nhỏ).
5. Chương 5: vẽ sơ đồ authoritative server.
6. Project: client CRUD chống API công khai hoặc local.

---

## Checklist hoàn thành Level 13

- [ ] Giải thích được request/response HTTP và vì sao HTTPS
- [ ] Dùng `HttpClient` tái sử dụng + async đúng
- [ ] Gọi REST GET/POST với System.Text.Json
- [ ] Mở được WebSocket, gửi/nhận message
- [ ] So sánh TCP vs UDP trong ngữ cảnh game
- [ ] Hoàn thành REST API Client
- [ ] Trả lời đúng ≥ 4/5 câu **Kiểm tra kiến thức** mỗi chương

Khi xong checklist → chuyển sang **Level 14**.
