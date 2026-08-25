# Chương 5 — TCP / UDP và liên hệ game multiplayer

## 1. Mục tiêu học

- Phân biệt TCP vs UDP ở tầng transport
- Biết khi game chọn reliable vs unreliable
- Hiểu authoritative server, tick, lag compensation (khái niệm)
- Liên hệ API .NET (`TcpClient`, `UdpClient`) ở mức giới thiệu — không xây netcode production

## 2. Điều kiện tiên quyết

- Chương 1–4: HTTP/WS (chạy trên TCP)
- Tư duy Level 11 concurrency

## 3. Khái niệm

| | TCP | UDP |
|--|-----|-----|
| Kết nối | Connection-oriented | Datagram, connectionless |
| Đảm bảo | Ordered, reliable, congestion control | Best-effort, có thể mất/lộn |
| Latency | Có thể head-of-line blocking | Thấp hơn nếu chấp nhận mất gói |
| Ví dụ | HTTP, WebSocket, FTP | DNS, voice, nhiều FPS state |

**Game multiplayer:**

- **TCP**: turn-based, MMO chat/inventory, một số mobile casual.
- **UDP** (+ reliable layer tự xây hoặc lib): FPS/action — vị trí gửi thường xuyên, mất vài gói OK.
- **Hybrid**: TCP cho login/shop; UDP cho gameplay.

**Authoritative server:** server là nguồn sự thật — client chỉ gửi input; chống cheat cơ bản.

## 4. Mô hình tư duy

```text
Client input ──► Server simulate tick ──► snapshot/delta ──► Clients render
                      ^
              không tin client "tôi có 999 gold"

TCP: stream bytes (cần length-prefix / delimiter message)
UDP: mỗi packet độc lập (cần id/sequence tự quản)
```

## 5. Cú pháp

TCP echo tối giản (lab local):

```csharp
// Server
var listener = new TcpListener(IPAddress.Loopback, 5000);
listener.Start();
using var client = await listener.AcceptTcpClientAsync();
await using var stream = client.GetStream();
byte[] buffer = new byte[256];
int n = await stream.ReadAsync(buffer);
await stream.WriteAsync(buffer.AsMemory(0, n));

// Client
using var c = new TcpClient();
await c.ConnectAsync(IPAddress.Loopback, 5000);
await using var s = c.GetStream();
byte[] msg = Encoding.UTF8.GetBytes("hi");
await s.WriteAsync(msg);
```

UDP gửi/nhận:

```csharp
using var udp = new UdpClient(0);
await udp.SendAsync(msg, msg.Length, new IPEndPoint(IPAddress.Loopback, 6000));
var result = await udp.ReceiveAsync();
```

## 6. Ví dụ

### Cơ bản

So sánh tính chất bằng bảng tự viết lại (không code).

### Trung cấp

Framing TCP — độ dài prefix 4 byte:

```csharp
static async Task SendFrameAsync(Stream stream, byte[] payload, CancellationToken ct)
{
    byte[] len = BitConverter.GetBytes(payload.Length);
    await stream.WriteAsync(len, ct);
    await stream.WriteAsync(payload, ct);
}
```

### Nâng cao

Sequence số trên UDP giả lập:

```csharp
public readonly record struct Packet(uint Seq, byte[] Payload);
// Gửi Seq tăng dần; bên nhận bỏ packet Seq cũ hơn last
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| TCP “dính” message | Stream không có boundary | Length-prefix / delimiter |
| UDP mất gói | Bình thường | Interpolation; resend quan trọng |
| NAT | Home router | Relay/STUN/TURN; listen server khó |
| Tin client | Cheat | Server authoritative |

## 8. Gỡ lỗi

1. Loopback trước khi ra LAN.
2. Wireshark filter `tcp.port == 5000` / `udp`.
3. Log sequence / tick id.

## 9. Best practices

- Đừng reinvent netcode — học khái niệm rồi dùng Unity Netcode / Mirror / Fish-Net / custom studio stack.
- Tách channel: reliable vs unreliable.
- Bandwidth budget; compress; không spam full state mỗi frame nếu có delta.
- Security: encrypt (DTLS/TLS), authenticate.

## 10. Bài tập

**Bài 1** — Viết 5 điểm khác TCP/UDP bằng tiếng Việt.

**Bài 2** — TCP client/server echo local (hai terminal hoặc 2 task).

**Bài 3** — UDP ping: server reply “pong”.

**Bài 4** — Giải thích vì sao FPS ít dùng TCP thuần cho vị trí mỗi frame.

## 11. Gợi ý

- Bài 2: mục 5; chạy listener trước.
- Bài 3: `UdpClient(6000)` server Receive/Send.
- Bài 4: latency + head-of-line khi mất/resend.

## 12. Đáp án

**Bài 1** — Gợi ý: kết nối, reliability, order, latency, use-case.

**Bài 2–3** — Code khung mục 5; tự chạy kiểm tra.

**Bài 4** — TCP đảm bảo thứ tự/resend làm tăng độ trễ và block gói sau khi một gói mất; vị trí game thường chấp nhận mất và nội suy.

## 13. Đáp án thay thế

Dùng Kcp / ENet / Steam relay — reliable UDP libraries.

## 14. Thử thách

Đọc bài viết “Snapshot interpolation” hoặc “Client-side prediction” — tóm tắt 10 dòng áp dụng MMORPG vs FPS.

## 15. Ứng dụng thực tế

- Custom protocol IoT
- Realtime services
- Nền tảng hiểu Netcode Unity / dedicated server

## 16. Liên hệ Unity

- **Unity Netcode for GameObjects**, Mirror, Fish-Net, Photon, Facepunch Steamworks…
- Capstone MMORPG: thường TCP hoặc reliable UDP + authoritative zone server — không sync mọi `transform` mọi client kiểu peer naive.
- Fixed timestep server (`Time.fixedDeltaTime` / custom tick).
- Mobile: UDP bị NAT/carrier — cần relay.
- WebGL: hạn chế socket thô — thường WebSocket/HTTPS.

## 17. Kiểm tra kiến thức

1. TCP đảm bảo gì UDP không?  
   **Đáp án:** Giao hàng đáng tin cậy, thứ tự (trong kết nối), kiểm soát tắc nghẽn.

2. Vì sao UDP hợp state vị trí?  
   **Đáp án:** Gói mới thay thế cũ; mất vài gói chấp nhận được.

3. Authoritative server nghĩa?  
   **Đáp án:** Server quyết định kết quả game; client chủ yếu gửi input.

4. TCP stream cần framing vì sao?  
   **Đáp án:** Không có ranh giới message sẵn — byte chảy liên tục.

5. HTTP chạy trên TCP hay UDP?  
   **Đáp án:** Chủ yếu TCP (HTTP/3 dùng QUIC/UDP — biết là ngoại lệ hiện đại).
