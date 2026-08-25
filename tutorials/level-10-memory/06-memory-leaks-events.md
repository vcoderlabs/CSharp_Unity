# Chương 6 — Memory Leaks & Event Leaks

## 1. Mục tiêu học

- Hiểu **memory leak** trong môi trường có GC (vẫn leak được!)
- Nhận diện **event / delegate leak** — thủ phạm phổ biến nhất
- Unsubscribe đúng chỗ; biết pattern yếu / “unsubscribe all”
- Liên hệ Unity: `OnEnable`/`OnDisable`, static events, singleton

## 2. Điều kiện tiên quyết

- Chương 3: roots & lifetime
- Level 7 (Delegates/Events) — hoặc nắm `event`, `+=` / `-=`
- Chương 2: object bị giữ → promote Gen2 → full GC đắt / RAM tăng

## 3. Khái niệm

### “Có GC thì sao còn leak?”

Leak managed = **object không còn dùng nhưng vẫn reachable** từ root → GC **không** thu → RAM tăng → thỉnh thoảng full GC nặng → **spike / hitch**.

### Event leak (cốt lõi)

```text
Publisher.OnHit += subscriber.Handle;

Publisher sống lâu (singleton / GameManager)
Subscriber muốn chết (UI screen, enemy despawn)
Nhưng delegate trong event vẫn trỏ tới subscriber
→ subscriber SỐNG mãi → LEAK
```

```text
Publisher (static/singleton)
    event Action OnHit
         │
         ├─► Subscriber A (UI đã “đóng”)  ← vẫn sống!
         └─► Subscriber B
```

### Thủ phạm khác

| Nguồn | Ví dụ |
|-------|--------|
| Static collections | `static List<Enemy> All` không Remove |
| Cache không bound | Dictionary id → entity vô hạn |
| Timer / callback | `Timer` giữ `this` |
| Lambda capture | `+= () => this.Foo()` khó `-=` |
| Unity scene unload | Singleton vẫn giữ ref scene object |
| Event static | `Enemy.OnAnyDied` toàn cục |

### Lambda và `-=`

```csharp
button.OnClick += () => Refresh(); // không trừ được cùng instance dễ dàng
```

Lưu handler vào field rồi `+=` / `-=` cùng reference.

## 4. Mô hình tư duy

```text
ĐÚNG (Unity-ish):
  OnEnable  → Subscribe
  OnDisable → Unsubscribe
  (object disable/destroy → không bị publisher giữ)

SAI:
  Start → Subscribe
  (không OnDisable) → Destroy subscriber → LEAK qua event
```

```text
Lifetime lifetime diagram:

GameManager (cả game) ──event──► UIScreen (chỉ 1 scene)
                                      ▲
                                      └── nếu không -=  thì UIScreen không chết
```

## 5. Cú pháp

```csharp
class Publisher
{
    public event Action<int>? HealthChanged;
    public void SetHealth(int hp) => HealthChanged?.Invoke(hp);
}

class Subscriber : IDisposable
{
    private readonly Publisher _pub;
    public Subscriber(Publisher pub)
    {
        _pub = pub;
        _pub.HealthChanged += OnHealth; // subscribe
    }

    private void OnHealth(int hp) => Console.WriteLine(hp);

    public void Dispose()
    {
        _pub.HealthChanged -= OnHealth; // bắt buộc
    }
}

// Lambda an toàn để -= 
Action? handler = null;
handler = () => DoWork();
evt += handler;
evt -= handler;
```

## 6. Ví dụ

### Cơ bản — Leak cố ý rồi sửa

```csharp
static event Action? Tick;

static void LeakDemo()
{
    var worker = new Worker();
    Tick += worker.OnTick; // leak nếu không trừ
    // Tick -= worker.OnTick; // sửa
    worker = null;
    GC.Collect();
    // Worker vẫn sống vì Tick giữ
}

class Worker
{
    public void OnTick() { }
}
```

### Trung cấp — Using ownership

```csharp
using var sub = new Subscriber(publisher);
publisher.SetHealth(10);
// Dispose → unsubscribe
```

### Nâng cao — Đếm subscriber + clear

```csharp
class Hub
{
    private event Action? _closed;
    public event Action Closed
    {
        add => _closed += value;
        remove => _closed -= value;
    }

    public void DisposeHub()
    {
        _closed = null; // cắt tất cả — khi hub chết
    }
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| RAM tăng mỗi lần mở UI | Subscribe không Unsubscribe | OnDisable/Dispose |
| `-=` không ăn | Lambda khác instance | Field lưu handler |
| Static event toàn project | Mọi enemy += | Cân nhắc non-static / message bus có unregister |
| “Tôi Destroy rồi” | C# ref + event còn | Clear event + null |
| Thread timer callback | Giữ object | Stop timer + Dispose |

## 8. Gỡ lỗi

1. Memory Profiler: path to root đi qua `Delegate` / `_invocationList`.
2. Thêm counter static `AliveWorkers`; `+=` tăng trong ctor, giảm khi unsubscribe — nếu Destroy mà counter không giảm = leak.
3. Tạm `event = null` trên publisher khi scene unload (nếu bạn sở hữu hub).
4. Unity: tìm mọi `+=` trong project; đối chiếu có `-=` không.

## 9. Best practices

- Subscribe/Unsubscribe **đối xứng** theo lifecycle (`OnEnable`/`OnDisable`).
- Tránh static event nếu không có chiến lược clear.
- Không anonymous lambda trên event dài hạn trừ khi short-lived publisher.
- Cân nhắc C# event-free patterns: message pipe, `event Action` trung gian clear được, hoặc weak event (cẩn thận).
- Pool + event: khi Return pool phải unsubscribe / reset.

## 10. Bài tập

**Bài 1** — Viết `Button` giả với `event Action Clicked`; `Dialog` subscribe trong ctor; chứng minh leak bằng static counter + `WeakReference` nếu không Dispose.

**Bài 2** — Sửa Bài 1 bằng `IDisposable` unsubscribe.

**Bài 3** — Tạo bug lambda `+= () => Count++` và giải thích vì sao `-=` khó.

**Bài 4** — Viết checklist 5 dòng cho code review “event safety” trong Unity.

## 11. Gợi ý

- Bài 1: `static int Alive`; ctor++; finalizer hoặc Dispose-- (dùng WeakRef + GC dễ hơn finalizer).
- Bài 2: `Clicked -= Handler` trong Dispose.
- Bài 3: mỗi lambda là delegate instance mới.
- Bài 4: OnEnable/OnDisable, không static bừa, lưu handler, pool reset, profiler.

## 12. Đáp án

**Bài 1–2** (gộp minh họa):

```csharp
class Button
{
    public event Action? Clicked;
    public void Click() => Clicked?.Invoke();
}

sealed class Dialog : IDisposable
{
    public static int Alive;
    private readonly Button _button;
    public Dialog(Button button)
    {
        Alive++;
        _button = button;
        _button.Clicked += OnClick;
    }
    void OnClick() { }
    public void Dispose()
    {
        _button.Clicked -= OnClick;
        Alive--;
    }
}

// Leak: new Dialog(btn); không Dispose → Alive không về 0, GC không thu
// Fix: using var d = new Dialog(btn);
```

**Bài 3** — `+= () => Count++` tạo delegate mới; `-= () => Count++` là **delegate khác** → không gỡ cái cũ.

**Bài 4** — Checklist gợi ý:

1. Mỗi `+=` có `-=` đối xứng lifecycle?  
2. Có static event không — ai clear?  
3. Handler có phải method group / field cố định?  
4. Object pooled có reset event không?  
5. Đã kiểm tra Memory Profiler khi mở/đóng UI 50 lần?

## 13. Đáp án thay thế

Dùng `WeakEventManager` / weak reference invocation list (phức tạp, edge case). Unity `UnityEvent` cũng cần `RemoveListener`.

## 14. Thử thách

Mô phỏng scene load/unload: `SceneHub` static event; 100 `Enemy` subscribe; unload gán list enemy = null nhưng **quên** unsubscribe — đo `Alive`. Sau đó sửa bằng hub `Clear()` hoặc unsubscribe loop.

## 15. Ứng dụng thực tế

- UI desktop WPF/WinForms event leak kinh điển
- Server long-lived bus + short-lived connection
- Plugin systems: unload module phải tháo event
- Mobile app: activity/fragment tương đương OnEnable/Disable

## 16. Liên hệ Unity

- **Luật vàng:** `OnEnable` subscribe, `OnDisable` unsubscribe.
- `GameManager.OnPlayerDied +=` từ UI — quên gỡ khi đóng panel = leak panel + sprite/atlas refs → RAM + đôi khi Gen2 pressure.
- Static `event Action OnGlobal` tiện nhưng nguy hiểm trong MMORPG nhiều hệ thống.
- Object pooling: khi `SetActive(false)` / Return, **phải** unsubscribe hoặc reset delegate.
- Event leak thường **không** hiện ngay GC Alloc cột Profiler (0B/frame) — RAM **leo dần** từng phút. Đo bằng Memory Profiler / mở-đóng UI lặp.
- Vẫn liên quan spike: nhiều object Gen2 chết muộn → full GC nặng hơn khi cuối cùng collect.

## 17. Kiểm tra kiến thức

1. GC có ngăn mọi memory leak không?  
   **Đáp án:** Không — object còn reference thì không thu.

2. Event giữ reference theo hướng nào?  
   **Đáp án:** Publisher giữ subscriber qua delegate.

3. Nên subscribe ở đâu trong Unity?  
   **Đáp án:** Thường `OnEnable`, gỡ `OnDisable`.

4. Vì sao lambda khó unsubscribe?  
   **Đáp án:** Mỗi lambda là instance delegate khác nhau.

5. Pool + event cần lưu ý gì?  
   **Đáp án:** Reset/unsubscribe khi trả pool kẻo giữ ref hoặc gọi nhầm.
