# Chương 5 — Events & Event Handlers

## 1. Mục tiêu học

- Hiểu từ khóa **`event`** và khác biệt với public delegate field
- Dùng `EventHandler`, `EventHandler<TEventArgs>`, custom EventArgs
- Pattern: subscribe `+=`, unsubscribe `-=`, raise an toàn null
- Liên hệ chặt với **UnityEvent** (Inspector + runtime)

## 2. Điều kiện tiên quyết

- Chương 1–2: delegate, Action, multicast sơ bộ (`+=`)
- Level 2: class, inheritance (`EventArgs`)

## 3. Khái niệm

**Event** là điểm “phát sóng” trong publisher: bên ngoài chỉ được **đăng ký/hủy**, không gán đè hay gọi trực tiếp (trừ cùng class).

```csharp
public class Button
{
    public event EventHandler? Clicked;

    public void Press() => Clicked?.Invoke(this, EventArgs.Empty);
}

var btn = new Button();
btn.Clicked += (_, _) => Console.WriteLine("click");
btn.Press();
// btn.Clicked = null;        // lỗi ngoài class
// btn.Clicked.Invoke(...);   // lỗi ngoài class
```

### So với field

```csharp
public Action? OnClick; // bên ngoài: OnClick = null; OnClick.Invoke(); — nguy hiểm
```

### EventHandler chuẩn

```csharp
void Handler(object? sender, EventArgs e);
// Generic:
void Handler(object? sender, TEventArgs e) where TEventArgs : EventArgs
```

## 4. Mô hình tư duy

```text
Publisher (chủ event)
  - khai báo event
  - Raise khi điều kiện (private/protected method OnXxx)
Subscriber
  - += handler khi cần nghe
  - -= khi không cần (tránh leak!)

Unity:
  C# event          ↔ code-only, nhẹ
  UnityEvent        ↔ serialize Inspector, AddListener/RemoveListener
  Tư duy pub/sub giống nhau
```

## 5. Cú pháp

```csharp
public class HpChangedEventArgs : EventArgs
{
    public int OldHp { get; }
    public int NewHp { get; }
    public HpChangedEventArgs(int oldHp, int newHp)
    {
        OldHp = oldHp;
        NewHp = newHp;
    }
}

public class Player
{
    private int _hp = 100;
    public event EventHandler<HpChangedEventArgs>? HpChanged;

    public void TakeDamage(int dmg)
    {
        int old = _hp;
        _hp = Math.Max(0, _hp - dmg);
        OnHpChanged(old, _hp);
    }

    protected virtual void OnHpChanged(int oldHp, int newHp)
    {
        HpChanged?.Invoke(this, new HpChangedEventArgs(oldHp, newHp));
    }
}
```

Cũng phổ biến: `event Action<int>? HpChanged;` — ngắn hơn, kém “chuẩn BCL sender/args”.

## 6. Ví dụ

### Cơ bản

```csharp
public class Timer
{
    public event Action? Elapsed;
    public void Tick() => Elapsed?.Invoke();
}

var t = new Timer();
t.Elapsed += () => Console.WriteLine("tick");
t.Tick();
```

### Trung cấp

```csharp
void OnHp(object? sender, HpChangedEventArgs e)
{
    Console.WriteLine($"{e.OldHp} → {e.NewHp}");
}

player.HpChanged += OnHp;
player.TakeDamage(30);
player.HpChanged -= OnHp; // quan trọng
```

### Nâng cao — custom accessor (hiếm khi cần)

```csharp
private EventHandler? _clicked;
public event EventHandler Clicked
{
    add { _clicked += value; /* log */ }
    remove { _clicked -= value; }
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Không gọi được event từ ngoài | Đúng thiết kế | Publisher có method Raise |
| Memory leak subscriber | Quên `-=` | Unsubscribe OnDisable/Dispose |
| `+=` lambda rồi không `-=` được | Instance khác nhau | Giữ field reference handler |
| Dùng public `Action` field thay event | Bị gán `=` mất subscriber | Đổi `event` |
| Raise không null-safe | Race/null | `?.Invoke` hoặc bản copy local |

## 8. Gỡ lỗi

1. Log số subscriber: `HpChanged?.GetInvocationList().Length`.  
2. Breakpoint trong OnHpChanged.  
3. Kiểm lifecycle Unity: subscribe `OnEnable`, unsubscribe `OnDisable`.

## 9. Best practices

- Raise qua `OnXxx` protected virtual (cho phép subclass).
- `EventArgs.Empty` khi không có data.
- Document thread: event có thread-safe không?
- Unity: code gameplay thuần → C# event; Designer hook → UnityEvent.
- Luôn có chiến lược unsubscribe.

## 10. Bài tập

**Bài 1** — Class `Alarm` với `event Action? Rang`; method `Trigger()`.

**Bài 2** — `BankAccount` event `EventHandler<BalanceChangedEventArgs>` khi Deposit/Withdraw.

**Bài 3** — Subscriber in số dư; sau đó `-=` và chứng minh không in nữa.

**Bài 4** — So sánh: field `public Action? X` vs `event Action? X` — viết 2 dòng comment chuyện bên ngoài làm được gì.

## 11. Gợi ý

- Bài 2: EventArgs có `decimal OldBalance`, `NewBalance`.
- Bài 3: method có tên để `-=`.
- Raise: `BalanceChanged?.Invoke(this, args)`.

## 12. Đáp án

**Bài 1** — Alarm đơn giản:

```csharp
public class Alarm
{
    public event Action? Rang;
    public void Trigger() => Rang?.Invoke();
}

var a = new Alarm();
a.Rang += () => Console.WriteLine("Ring!");
a.Trigger();
```

**Bài 2** — BankAccount + EventArgs:

```csharp
public class BalanceChangedEventArgs : EventArgs
{
    public decimal OldBalance { get; }
    public decimal NewBalance { get; }
    public BalanceChangedEventArgs(decimal oldB, decimal newB)
    {
        OldBalance = oldB;
        NewBalance = newB;
    }
}

public class BankAccount
{
    private decimal _balance;
    public event EventHandler<BalanceChangedEventArgs>? BalanceChanged;

    public void Deposit(decimal amount)
    {
        var old = _balance;
        _balance += amount;
        BalanceChanged?.Invoke(this, new BalanceChangedEventArgs(old, _balance));
    }
}
```

**Bài 3** — Subscribe / unsubscribe:

```csharp
void Print(object? s, BalanceChangedEventArgs e) =>
    Console.WriteLine(e.NewBalance);

account.BalanceChanged += Print;
account.Deposit(10);  // in
account.BalanceChanged -= Print;
account.Deposit(10);  // không in
```

**Bài 4** — Comment so sánh:

```csharp
// public Action? X: ngoài class có thể X = null; X.Invoke(); X = Other;
// event Action? X: ngoài class chỉ += và -= ; không gán đè / không Invoke
```

## 13. Đáp án thay thế

`event Action<decimal, decimal>?` thay EventHandler để ngắn. Record cho EventArgs nếu muốn (`HpChangedEventArgs` class vẫn phổ biến).

## 14. Thử thách

Viết `GameEvents` static hub (`event Action<Player>? PlayerDied`) — rồi giải thích rủi ro leak/scene; đề xuất instance EventBus thay static.

## 15. Ứng dụng thực tế

- UI controls, file watcher, message bus
- Domain events trong DDD nhẹ
- Observer pattern chuẩn .NET
- Plugin: host expose events

## 16. Liên hệ Unity — CRITICAL

| C# | Unity |
|----|--------|
| `event Action` / `EventHandler` | Code-only, nhanh, không hiện Inspector |
| — | `UnityEvent` / `UnityEvent<T>` trên MonoBehaviour — gán trong Inspector |
| `+=` / `-=` | `AddListener` / `RemoveListener` |
| `Invoke` | `unityEvent.Invoke()` |
| Quên `-=` | Quên `RemoveListener` → leak / gọi vào object destroyed |

```csharp
// Pure C# trên MonoBehaviour
public event Action? Died;
void Die() => Died?.Invoke();

// UnityEvent cho Designer
public UnityEvent onDied;
void Die() => onDied.Invoke();
```

Có thể kết hợp: C# event nội bộ + UnityEvent bridge cho Designer.

**Quy tắc vàng:** `OnEnable` subscribe, `OnDisable` unsubscribe (cả C# event lẫn UnityEvent runtime listeners).

## 17. Kiểm tra kiến thức

1. Bên ngoài class làm được gì với `event`?  
   **Đáp án:** Chỉ `+=` và `-=`.

2. `EventHandler<T>` tham số thứ hai thường là gì?  
   **Đáp án:** `T` kế thừa `EventArgs` (dữ liệu sự kiện).

3. Vì sao cần `-=`?  
   **Đáp án:** Tránh memory leak / gọi handler sau khi object không còn hợp lệ.

4. UnityEvent khác C# event điểm lớn?  
   **Đáp án:** UnityEvent serializable trên Inspector; API AddListener; không phải từ khóa `event` của C#.

5. Raise an toàn khi không có subscriber?  
   **Đáp án:** `EventName?.Invoke(...)`.
