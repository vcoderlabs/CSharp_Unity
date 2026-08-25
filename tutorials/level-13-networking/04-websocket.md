# Chương 4 — WebSocket

## 1. Mục tiêu học

- Hiểu WebSocket: kết nối đầy đủ song công sau handshake HTTP
- Dùng `ClientWebSocket` gửi/nhận text
- Vòng nhận message + hủy bằng CancellationToken
- So sánh với HTTP request/response và long polling

## 2. Điều kiện tiên quyết

- Chương 1–2: HTTP, HttpClient
- Level 11: async loops + cancel

## 3. Khái niệm

**WebSocket** giữ TCP (thường) connection mở: server/client **đẩy** message bất cứ lúc nào — phù hợp chat, live price, game realtime nhẹ trên web.

```text
HTTP Upgrade: websocket  →  101 Switching Protocols
sau đó: frames binary/text hai chiều
```

| | HTTP REST | WebSocket |
|--|-----------|-----------|
| Mô hình | Request/response | Full-duplex messages |
| Overhead | Header mỗi request | Handshake một lần |
| Phù hợp | CRUD, tài nguyên | Event stream, chat |

`System.Net.WebSockets.ClientWebSocket` là client BCL.

## 4. Mô hình tư duy

```text
ConnectAsync(uri)
  └─ SendAsync(buffer)
  └─ ReceiveAsync(buffer) loop → đóng khi Close
Dispose / Abort khi cancel
```

Buffer nhận có thể cần ghép nhiều frame (`EndOfMessage == false`).

## 5. Cú pháp

Kết nối, gửi, nhận một message:

```csharp
using var ws = new ClientWebSocket();
await ws.ConnectAsync(new Uri("wss://echo.websocket.events"), CancellationToken.None);

byte[] send = Encoding.UTF8.GetBytes("hello");
await ws.SendAsync(send, WebSocketMessageType.Text, endOfMessage: true, CancellationToken.None);

byte[] buffer = new byte[4096];
var result = await ws.ReceiveAsync(buffer, CancellationToken.None);
string text = Encoding.UTF8.GetString(buffer, 0, result.Count);
Console.WriteLine(text);

await ws.CloseAsync(WebSocketCloseStatus.NormalClosure, "bye", CancellationToken.None);
```

> URL echo công khai có thể đổi theo thời gian — nếu down, tự host echo nhỏ hoặc dùng service khác.

## 6. Ví dụ

### Cơ bản

Kiểm state:

```csharp
Console.WriteLine(ws.State); // Open / Closed / Aborted...
```

### Trung cấp

Vòng nhận đến khi Close:

```csharp
async Task ReceiveLoopAsync(ClientWebSocket ws, CancellationToken ct)
{
    var buffer = new byte[4096];
    while (ws.State == WebSocketState.Open && !ct.IsCancellationRequested)
    {
        var result = await ws.ReceiveAsync(buffer, ct);
        if (result.MessageType == WebSocketMessageType.Close)
        {
            await ws.CloseAsync(WebSocketCloseStatus.NormalClosure, null, ct);
            break;
        }
        Console.WriteLine(Encoding.UTF8.GetString(buffer, 0, result.Count));
    }
}
```

### Nâng cao

Ghép message nhiều frame:

```csharp
async Task<string> ReceiveTextAsync(ClientWebSocket ws, CancellationToken ct)
{
    using var ms = new MemoryStream();
    var buffer = new byte[1024];
    WebSocketReceiveResult result;
    do
    {
        result = await ws.ReceiveAsync(buffer, ct);
        ms.Write(buffer, 0, result.Count);
    } while (!result.EndOfMessage);
    return Encoding.UTF8.GetString(ms.ToArray());
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Connect fail | URL `ws` vs `wss`, firewall | Đúng scheme; HTTPS proxy |
| Message cắt | Không ghép frame | Đọc đến `EndOfMessage` |
| Treo Receive | Không cancel | CTS khi shutdown |
| Gửi khi Closed | Race | Kiểm `State == Open` |

## 8. Gỡ lỗi

1. Log `State` trước Send/Receive.
2. Wireshark / browser WS frames nếu web.
3. Bắt `WebSocketException` / `OperationCanceledException`.

## 9. Best practices

- Luôn có receive loop + cancel sạch.
- Ping/pong / application heartbeat nếu proxy cắt idle.
- Giới hạn kích thước message.
- Reconnect với backoff khi drop.
- Auth: token trong query/header lúc handshake (cẩn thận log URL).

## 10. Bài tập

**Bài 1** — Connect echo, gửi `"ping"`, in response.

**Bài 2** — Gửi 5 message, nhận 5 lần.

**Bài 3** — Cancel receive loop sau 3 giây.

**Bài 4** — Phân biệt `ws://` và `wss://` bằng 3 câu.

## 11. Gợi ý

- Bài 1–2: mục 5–6.
- Bài 3: `CancelAfter(3000)` + try/catch OCE.
- Bài 4: wss = TLS tương tự https.

## 12. Đáp án

**Bài 1** — Echo một lần (như mục 5).

**Bài 2** — Vòng for Send + Receive.

```csharp
for (int i = 0; i < 5; i++)
{
    var bytes = Encoding.UTF8.GetBytes($"msg-{i}");
    await ws.SendAsync(bytes, WebSocketMessageType.Text, true, ct);
    var buffer = new byte[256];
    var r = await ws.ReceiveAsync(buffer, ct);
    Console.WriteLine(Encoding.UTF8.GetString(buffer, 0, r.Count));
}
```

**Bài 3** — Cancel:

```csharp
using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(3));
try { await ReceiveLoopAsync(ws, cts.Token); }
catch (OperationCanceledException) { Console.WriteLine("stopped"); }
```

**Bài 4** — `wss` mã hóa TLS; `ws` plain — production dùng `wss`.

## 13. Đáp án thay thế

Thư viện `Websocket.Client` bọc reconnect; SignalR dùng Hub protocol trên WebSocket.

## 14. Thử thách

Mini chat console: 2 task — một đọc stdin gửi WS; một receive in ra console.

## 15. Ứng dụng thực tế

- Chat / notification
- Collaborative editing presence
- Live dashboard

## 16. Liên hệ Unity

- Nhiều game web/mobile dùng WS hoặc UDP riêng.
- NativeWebSocket / websocket-sharp packages phổ biến.
- Nhận message trên background → queue về main thread cập nhật UI/gameplay.
- MMORPG nặng thường **không** chỉ dựa WS trình duyệt — server dedicated + UDP/TCP tùy genre.

## 17. Kiểm tra kiến thức

1. WebSocket khác REST chỗ nào?  
   **Đáp án:** Kết nối dài, hai chiều, không cần request mới mỗi lần đẩy event.

2. Handshake bắt đầu bằng gì?  
   **Đáp án:** HTTP Upgrade lên websocket.

3. `EndOfMessage == false` nghĩa?  
   **Đáp án:** Còn frame tiếp — cần đọc tiếp để đủ message.

4. `wss://` là gì?  
   **Đáp án:** WebSocket over TLS.

5. Vì sao cần CancellationToken trên ReceiveAsync?  
   **Đáp án:** Thoát loop khi shutdown/scene đổi — tránh treo.
