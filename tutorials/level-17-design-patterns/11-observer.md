# Chương 11 — Observer

## 1. Mục tiêu học

- Phát/thu sự kiện: Subject thông báo Observers khi state đổi
- C# `event` / delegate = Observer ngôn ngữ tích hợp
- Tránh leak subscribe (đặc biệt Unity)
- Ứng dụng quest, UI HP, achievement

## 2. Điều kiện tiên quyết

- [Level 7 — Delegates & Events](../level-07-delegates-events/)
- DIP nhẹ: phụ thuộc `Action`/`event` thay hard ref UI

## 3. Khái niệm

**Observer:** one-to-many — khi Subject đổi, mọi Observer được notify.  
C# idiom: `event EventHandler<>`; Unity: `UnityEvent`, message bus, Reactive Extensions (nâng cao).

## 4. Mô hình tư duy

```text
Health (Subject) ──event Damaged──► Hud, CombatLog, Achievement
Observers không biết nhau — giảm coupling
```

## 5. Cú pháp

```csharp
public sealed class Health
{
    public int Current { get; private set; }
    public event Action<int>? Changed;

    public void Damage(int n)
    {
        Current = Math.Max(0, Current - n);
        Changed?.Invoke(Current);
    }
}
```

## 6. Ví dụ

### Cơ bản

```csharp
var hp = new Health();
hp.Changed += v => Console.WriteLine($"HP={v}");
hp.Damage(3);
```

### Trung cấp — nhiều listener + unsubscribe

```csharp
void OnEnable() => _health.Changed += OnHp;
void OnDisable() => _health.Changed -= OnHp;
void OnHp(int v) => _slider.value = v;
```

### Nâng cao / Unity MMORPG

```csharp
public sealed class QuestEvents
{
    public event Action<QuestId>? Completed;
    public void RaiseCompleted(QuestId id) => Completed?.Invoke(id);
}

// UI, loot table, analytics subscribe — QuestSystem không FindObject UI
```

Event bus toàn cục: tiện nhưng dễ God Bus — cân nhắc Mediator (chương 16) theo vùng.

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| NRE sau destroy | Quên unsubscribe | `-=` OnDisable; weak pattern |
| Thứ tự listener phụ thuộc | Order không đảm bảo | Không logic order-critical trong event |
| Event storm | Notify quá dày | Coalesce / dirty flag |

## 8. Gỡ lỗi

1. Breakpoint invoke list.  
2. Log `GetInvocationList().Length`.  
3. Unity: Domains reload — static events cực kỳ dễ leak.

## 9. Best practices

- `event` public add/remove, raise private.  
- Payload rõ (`HpChangedEvent`), tránh `object`.  
- Unsubscribe đối xứng lifecycle.

## 10. Bài tập

**Bài 1** — `Wallet.GoldChanged` cập nhật 2 listener.  
**Bài 2** — Quest complete mở achievement.  
**Bài 3** — Chứng minh leak nếu không `-=`.  
**Bài 4** — So `event` vs `UnityEvent`.

## 11. Gợi ý

Bài 3: giữ subject sống, listener nên chết — static/event vẫn giữ ref.

## 12. Đáp án

```csharp
public sealed class Wallet
{
    public int Gold { get; private set; }
    public event Action<int>? GoldChanged;
    public void Add(int n) { Gold += n; GoldChanged?.Invoke(Gold); }
}
```

## 13. Đáp án thay thế

`IObservable` / UniRx / R3 — cùng Observer, API stream.

## 14. Thử thách

Combat: `OnKill` → quest progress + loot roll mà `Enemy` không reference UI.

## 15. Ứng dụng thực tế

- UI binding  
- Domain events  
- Pub/sub message brokers (scale lớn)

## 16. Liên hệ Unity

- **Critical MMORPG:** UI, quest, achievement, audio stingers  
- `UnityEvent` SerializeField designer-friendly  
- Cẩn thận ScriptableObject event channels (pattern phổ biến)

## 17. Kiểm tra kiến thức

1. Observer giải quyết? **Notify nhiều listener khi state đổi.**  
2. C# map sang? **`event` / delegate.**  
3. Leak khi? **Subscribe mà không unsubscribe.**  
4. Subject nên biết concrete UI? **Không.**  
5. Event bus rủi ro? **Coupling ẩn + God bus.**
