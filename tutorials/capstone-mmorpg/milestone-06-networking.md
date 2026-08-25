# Milestone 06 — Networking

## Requirements

- **HTTP REST** client (`HttpClient` / Unity `UnityWebRequest` — chọn một, nhất quán) gọi mock backend
- Ít nhất: `POST /auth/login` (fake) → token; `GET /profile` → JSON player summary
- **WebSocket** (hoặc giả lập bằng long-poll nếu WS quá nặng) cho 1 kênh realtime: chat hệ thống / presence “server tick” — tối thiểu nhận message
- Toàn bộ I/O **async/await** + `CancellationToken` khi rời scene
- JSON serialize (`System.Text.Json` hoặc Unity JsonUtility có DTO rõ)
- Không block main thread

**Mock server:** ASP.NET Minimal API local **hoặc** Node **hoặc** file-based fake + Tool giả — ghi rõ trong README.

---

## Architecture

```text
Unity Client                         Mock Server
┌──────────────────┐                ┌─────────────────┐
│ IAuthApi         │──REST login───►│ POST /auth/login │
│ IProfileApi      │──GET profile──►│ GET /profile     │
│ IRealtimeSocket  │◄──WS message───│ /ws              │
└────────┬─────────┘                └─────────────────┘
         │
         ▼
 SessionService (token, profile cache)
 UI LoginPresenter
```

```text
DTO:
  LoginRequest { username, password }
  LoginResponse { token, playerId }
  ProfileDto { name, level, gold }
  ServerPushDto { type, payload }
```

---

## Tasks

1. Chạy mock server (kèm script/README).  
2. `IAuthApi.LoginAsync`.  
3. Lưu token memory (không commit).  
4. `GetProfileAsync` gắn header Authorization.  
5. Màn login UI tối thiểu.  
6. WS: connect + nhận 1 heartbeat/chat log ra UI.  
7. Cancel khi OnDisable.  
8. Cập nhật ARCHITECTURE networking.

---

## Expected result

- Login sai → lỗi có message; login đúng → vào “game” với profile.  
- Profile hiện level/gold từ server mock.  
- WS message xuất hiện (hoặc document fallback).  
- Không `Thread.Sleep` trên main; không `.Result` deadlock.

---

## Exercises

**E1** — Retry 1 lần khi GET profile 5xx.  
**E2** — Timeout cấu hình qua Options.  
**E3** — Mock “server authority” gold: client không tự cộng gold local khi quest (gọi POST reward giả).  
**E4** — Log correlation id header.

---

## Hints

- Editor: `localhost` permission; HTTP cleartext nếu cần.  
- Unity 2022+: `UnityWebRequest` quen thuộc; `HttpClient` cần cân nhắc platform.  
- DTOs plain C# — không serialize MonoBehaviour.  
- Tách `Net` asmdef nếu muốn.

---

## Solution outline

Mock Minimal API:

```csharp
app.MapPost("/auth/login", (LoginRequest r) =>
    r.Password == "pass"
        ? Results.Ok(new LoginResponse("fake-token", "p1"))
        : Results.Unauthorized());

app.MapGet("/profile", (HttpRequest req) =>
{
    if (req.Headers.Authorization != "Bearer fake-token")
        return Results.Unauthorized();
    return Results.Ok(new ProfileDto("Hero", 5, 100));
});
```

Client:

```csharp
public sealed class AuthApi : IAuthApi
{
    readonly HttpClient _http;
    public async Task<LoginResponse> LoginAsync(LoginRequest req, CancellationToken ct)
    {
        var res = await _http.PostAsJsonAsync("/auth/login", req, ct);
        res.EnsureSuccessStatusCode();
        return await res.Content.ReadFromJsonAsync<LoginResponse>(ct);
    }
}
```

WS: `ClientWebRequest` package hoặc `System.Net.WebSockets.ClientWebSocket` loop receive async.

---

## Code review checklist

- [ ] Token không log full  
- [ ] CancellationToken được truyền  
- [ ] DTO tách khỏi View  
- [ ] README hướng dẫn chạy mock server  
- [ ] Timeout/retry có chủ đích  
- [ ] Không block main thread  
- [ ] Error surface ra UI  
- [ ] ARCHITECTURE có sequence login  
