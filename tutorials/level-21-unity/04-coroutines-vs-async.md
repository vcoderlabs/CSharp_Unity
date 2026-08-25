# Chương 4 — Coroutines vs async/await trong Unity

## 1. Mục tiêu học

- Hiểu **coroutine** chạy trên Player loop
- Biết **async/await** trong Unity (Task + `Awaitable` / UniTask tùy version)
- Chọn đúng công cụ: timing gameplay vs I/O
- Tránh fire-and-forget lỗi và hủy khi Destroy

## 2. Điều kiện tiên quyết

- Level 11 async khái niệm
- Lifecycle chương 1
- `IEnumerator` / `yield`

## 3. Khái niệm

**Coroutine:** method trả `IEnumerator`, Unity resume theo `yield return` (`null` = chờ frame, `WaitForSeconds`, `WaitUntil`…).

**async/await:** state machine .NET; tốt cho I/O, Addressables, HTTP. Cần đồng bộ về main thread khi đụng API Unity.

| | Coroutine | async |
|--|-----------|-------|
| Tích hợp Update timing | Tốt | Cần wrapper |
| HTTP / file | Được nhưng cũ | Tốt hơn |
| Hủy | `StopCoroutine` | `CancellationToken` |
| Exception | Dễ nuốt | Propagate Task |

## 4. Mô hình tư duy

```text
Gameplay nhịp frame / delay đơn giản  →  Coroutine hoặc Awaitable.WaitForSeconds
Network / Addressables / DB             →  async
Đừng Block main thread                 →  không Task.Wait() trên main
```

## 5. Cú pháp

```csharp
IEnumerator Fade()
{
    float t = 0;
    while (t < 1f)
    {
        t += Time.deltaTime;
        yield return null;
    }
}

void Start() => StartCoroutine(Fade());
```

```csharp
async void OnButton() // tránh async void trừ event
{
    await LoadAsync();
}

async Awaitable LoadAsync() // Unity 6 Awaitable
{
    await Awaitable.WaitForSecondsAsync(1f);
}
```

UniTask (package phổ biến): `await UniTask.Delay(1000, cancellationToken: token)`.

## 6. Ví dụ

### Cơ bản — cooldown coroutine

```csharp
IEnumerator Cooldown(float s)
{
    _ready = false;
    yield return new WaitForSeconds(s);
    _ready = true;
}
```

Cache `WaitForSeconds` nếu spam (giảm alloc).

### Trung cấp — async load text

```csharp
async Awaitable<string> ReadStreamingAsync(string path)
{
    string full = Path.Combine(Application.streamingAssetsPath, path);
    return await File.ReadAllTextAsync(full);
}
```

### Nâng cao — hủy khi destroy

```csharp
CancellationTokenSource _cts;

void OnEnable() => _cts = new CancellationTokenSource();
void OnDisable() => _cts.Cancel();

async Awaitable LoopAsync()
{
    while (!_cts.IsCancellationRequested)
    {
        await Awaitable.WaitForSecondsAsync(1f);
        // work
    }
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| `Wait` trên main | Deadlock/jank | await không sync wait |
| async void exception | Nuốt khó bắt | async Awaitable/Task + try |
| Coroutine sau Destroy | Tiếp tục đụng null | Stop / check null |
| `new WaitForSeconds` mỗi frame | Alloc | Cache instance |

## 8. Gỡ lỗi

1. Log khi vào/ra coroutine.  
2. Editor: script đang chạy coroutine?  
3. Exception trong coroutine: bật break on exception.  
4. async: quan sát continuation thread (chỉ main được gọi Unity API).

## 9. Best practices

- UI button: async method có try/catch + disable button.  
- Gameplay delay ngắn: coroutine OK.  
- Luôn có chiến lược cancel khi OnDisable.  
- Prefers `Awaitable`/`UniTask` hơn `async void`.  
- Không trộn hai hệ vô tổ chức trong một feature.

## 10. Bài tập

**Bài 1** — Coroutine nhấp nháy màu material 3 lần.  
**Bài 2** — Async đếm ngược 5 giây cập nhật TMP mỗi giây.  
**Bài 3** — Hủy countdown khi disable object.  
**Bài 4** — So alloc: `new WaitForSeconds(1)` trong loop vs cache field.

## 11. Gợi ý

- Bài 1: `yield return WaitForSeconds`.  
- Bài 3: CTS hoặc stop coroutine flag.

## 12. Đáp án

```csharp
WaitForSeconds _wait = new(0.2f);
IEnumerator Blink()
{
    for (int i = 0; i < 6; i++)
    {
        // toggle color
        yield return _wait;
    }
}
```

## 13. Đáp án thay thế

DOTween / animation timeline thay coroutine fade. Reactive extensions.

## 14. Thử thách

Viết `AsyncService` load JSON config bằng Addressables + hủy khi đổi scene.

## 15. Ứng dụng thực tế

Skill wind-up, cutscene timing, login HTTP, cloud save.

## 16. Liên hệ C# thuần

Coroutine ≈ iterator + game scheduler. async ≈ Task ecosystem. Unity main thread = SynchronizationContext đặc thù.

## 17. Kiểm tra kiến thức

1. `yield return null` nghĩa là?  
   **Đáp án:** Tiếp tục frame sau.

2. Vì sao tránh `Task.Result` trên main?  
   **Đáp án:** Có thể block/deadlock.

3. Hủy coroutine bằng?  
   **Đáp án:** `StopCoroutine` / `StopAllCoroutines` / disable.

4. async phù hợp việc gì hơn coroutine cổ điển?  
   **Đáp án:** I/O, API hiện đại, composition Task.

5. `async void` nguy hiểm vì?  
   **Đáp án:** Exception khó quan sát; không await được từ ngoài.
