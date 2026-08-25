# Chương 5 — UnityEvents vs C# events/delegates

## 1. Mục tiêu học

- So sánh **UnityEvent** (Inspector wiring) và **C# event**
- Chọn đúng chỗ: designer hook vs code architecture
- Tránh memory leak do quên unsubscribe
- Biết multicast và thứ tự gọi

## 2. Điều kiện tiên quyết

- Level 7 delegates/events
- OnEnable/OnDisable

## 3. Khái niệm

**C# `event Action`:** compile-time, rõ ràng, dễ test, không hiện Inspector mặc định.  
**UnityEvent:** serialize được, kéo thả method trên Inspector — tiện designer, dễ “spaghetti” và khó refactor rename method.

```csharp
// C#
public event Action<int> HealthChanged;

// Unity
public UnityEvent<int> OnHealthChanged;
```

`UnityEvent` nằm trong `UnityEngine.Events`.

## 4. Mô hình tư duy

```text
Designer bấm nút UI → UnityEvent OnClick (OK)
Combat system nội bộ → C# event / message bus (OK)
Đừng: mọi thứ UnityEvent + 20 slot Inspector khó review
```

## 5. Cú pháp

```csharp
using UnityEngine.Events;

public class Health : MonoBehaviour
{
    public event Action<int, int> Changed; // current, max
    [SerializeField] UnityEvent onDied;

    public void TakeDamage(int dmg)
    {
        _hp = Mathf.Max(0, _hp - dmg);
        Changed?.Invoke(_hp, _max);
        if (_hp == 0) onDied.Invoke();
    }
}

// Listener
void OnEnable() => health.Changed += OnChanged;
void OnDisable() => health.Changed -= OnChanged;
```

## 6. Ví dụ

### Cơ bản — UI Button

Button component dùng UnityEvent — giữ nguyên; logic gọi service C#.

### Trung cấp — event channel SO

```csharp
[CreateAssetMenu]
public class VoidEventChannel : ScriptableObject
{
    public event Action Raised;
    public void Raise() => Raised?.Invoke();
}
```

Listener MB subscribe channel — giảm coupling scene.

### Nâng cao — không leak

```csharp
void OnDisable()
{
    if (channel != null) channel.Raised -= Handler;
}
```

Static event / singleton event cực dễ leak.

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Listener vẫn chạy sau destroy | Quên -= | OnDisable |
| UnityEvent mất slot sau rename | Serialize method name | Gán lại / wrapper |
| Exception trong subscriber | Multicast dừng? | C# tiếp tục tùy; vẫn nên try riêng |
| Quá nhiều Inspector events | Khó diff Git | Chuyển code event |

## 8. Gỡ lỗi

1. Log khi subscribe/unsubscribe.  
2. Breakpoint Invoke.  
3. Missing reference UnityEvent (object bị xóa).  
4. Profiler: không trực tiếp — dùng logic đếm listener (debug only).

## 9. Best practices

- Boundary UI: UnityEvent OK.  
- Domain gameplay: C# event / bus.  
- Channel SO cho decouple scene.  
- Luôn pair += với -=.  
- Document ai Raise.

## 10. Bài tập

**Bài 1** — Health C# event; UI text listener.  
**Bài 2** — Thêm UnityEvent `onDied` gọi particle từ Inspector.  
**Bài 3** — VoidEventChannel SO: door raise, trap listen.  
**Bài 4** — Cố ý quên unsubscribe; quan sát lỗi; sửa.

## 11. Gợi ý

- Bài 1: OnEnable/OnDisable.  
- Bài 3: hai MB khác scene hierarchy.

## 12. Đáp án

Pattern OnEnable/OnDisable như mục Cú pháp + Channel Raise ở trigger `OnTriggerEnter`.

## 13. Đáp án thay thế

MessagePipe, UniRx/R3, Observable. MVC event aggregator.

## 14. Thử thách

Generic `EventChannel<T>` SO không alloc mỗi Raise (tránh LINQ).

## 15. Ứng dụng thực tế

Quest flag, UI binding, achievement, audio cues.

## 16. Liên hệ C# thuần

Giống L7 — Unity thêm lớp serialize Inspector. Capstone dùng Observer dày đặc.

## 17. Kiểm tra kiến thức

1. UnityEvent mạnh ở điểm nào?  
   **Đáp án:** Wiring trên Inspector không sửa code.

2. C# event mạnh ở điểm nào?  
   **Đáp án:** Type-safe, refactor, test, architecture rõ.

3. Leak event điển hình?  
   **Đáp án:** Object destroy nhưng vẫn subscribe static/channel.

4. Nên unsubscribe ở đâu?  
   **Đáp án:** `OnDisable` / `OnDestroy` tương ứng chỗ subscribe.

5. Event channel SO giúp gì?  
   **Đáp án:** Giảm reference cứng giữa object trong scene.
