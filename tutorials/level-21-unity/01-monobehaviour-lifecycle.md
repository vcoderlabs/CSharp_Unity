# Chương 1 — MonoBehaviour & script lifecycle

## 1. Mục tiêu học

- Hiểu Unity gọi method lifecycle khi nào
- Đặt init / frame logic / physics / cleanup đúng chỗ
- Tránh công việc nặng trong `Update`
- Phân biệt `Awake` vs `Start` vs `OnEnable`

## 2. Điều kiện tiên quyết

- Cài Unity Editor, tạo project 3D/2D trống
- Biết C# class, inheritance
- Script gắn trên GameObject trong scene

## 3. Khái niệm

`MonoBehaviour` là base class script Unity gắn **Component** trên GameObject. Engine gọi các “magic methods” theo vòng đời.

| Method | Khi nào (tóm tắt) |
|--------|-------------------|
| `Awake` | Object khởi tạo (một lần khi load) — kể cả inactive? Awake khi object active lần đầu được init |
| `OnEnable` | Mỗi lần enable |
| `Start` | Trước Update đầu tiên, sau mọi Awake liên quan đã chạy (cùng frame init) |
| `FixedUpdate` | Nhịp physics cố định |
| `Update` | Mỗi frame |
| `LateUpdate` | Sau mọi Update — camera follow |
| `OnDisable` | Khi disable / destroy |
| `OnDestroy` | Khi hủy |

Thứ tự chính xác có tài liệu Unity — học thuộc **vai trò**, không đoán.

## 4. Mô hình tư duy

```text
Scene load
  → Awake (wiring: GetComponent, cache ref)
  → OnEnable (subscribe events)
  → Start (phụ thuộc object khác đã Awake)
  → [Game loop]
       FixedUpdate* → Update → LateUpdate
  → OnDisable (unsubscribe)
  → OnDestroy (cleanup cuối)
```

## 5. Cú pháp

```csharp
using UnityEngine;

public class PlayerHealth : MonoBehaviour
{
    [SerializeField] private int maxHp = 100;
    private int _hp;

    private void Awake()
    {
        _hp = maxHp;
    }

    private void Update()
    {
        // tránh logic nặng
    }

    private void OnDestroy()
    {
        // cleanup nếu cần
    }
}
```

## 6. Ví dụ

### Cơ bản — log thứ tự

```csharp
void Awake() => Debug.Log($"{name} Awake");
void OnEnable() => Debug.Log($"{name} OnEnable");
void Start() => Debug.Log($"{name} Start");
void OnDisable() => Debug.Log($"{name} OnDisable");
```

Gắn 2 object, tắt/bật để quan sát.

### Trung cấp — cache component

```csharp
Rigidbody _rb;
void Awake() => _rb = GetComponent<Rigidbody>();
void FixedUpdate() => _rb.AddForce(Vector3.forward);
```

### Nâng cao — Script Execution Order

Khi A.Start cần B đã sẵn sàng nhưng thứ tự không đảm bảo giữa object — dùng Awake wiring + Start consume, hoặc Execution Order / bootstrap.

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Null ở Awake | Object kia chưa Awake / inactive | Start hoặc service locator/DI |
| Logic gameplay trong Update nặng | Hot path | Event / dirty flag / Job |
| Quên OnDisable unsubscribe | Leak | Pair Enable/Disable |
| Dùng `Find` mỗi Update | Chậm | Cache Awake |

## 8. Gỡ lỗi

1. `Debug.Log` có prefix lifecycle.  
2. Breakpoint trong Editor (có thể chậm).  
3. Inspector: script enabled? GameObject active?  
4. Documentation: Message Order.

## 9. Best practices

- `Awake`: tự setup; `Start`: phụ thuộc người khác.  
- Physics lực → `FixedUpdate`.  
- Camera follow → `LateUpdate`.  
- Không `new MonoBehaviour()`.  
- Giữ `Update` ngắn hoặc tách hệ thống tick.

## 10. Bài tập

**Bài 1** — Script log đủ Awake/OnEnable/Start/Update(1 lần flag)/OnDisable/OnDestroy.  
**Bài 2** — Object inactive lúc đầu: đoán method nào chạy khi nào; xác minh.  
**Bài 3** — Di chuyển bằng `transform` trong Update vs `Rigidbody` trong FixedUpdate — quan sát khác physics.  
**Bài 4** — Hai script: A đọc giá trị B set trong Awake — đặt code đúng chỗ để không null/sai thứ tự.

## 11. Gợi ý

- Bài 2: inactive → chưa Awake đến khi được activate (theo rule Unity version — kiểm chứng).  
- Bài 4: B set trong Awake; A đọc trong Start.

## 12. Đáp án

**Bài 4**:

```csharp
// B
public class ScoreBoard : MonoBehaviour {
    public int Score { get; private set; }
    void Awake() => Score = 10;
}

// A
public class ScoreReader : MonoBehaviour {
    [SerializeField] ScoreBoard board;
    void Start() => Debug.Log(board.Score);
}
```

## 13. Đáp án thay thế

Bootstrap `GameInstaller` chạy sớm (Execution Order -100). Event `RuntimeInitializeOnLoadMethod`.

## 14. Thử thách

Viết `GameTicker` gọi `ITickable.Tick(dt)` cho list service — MonoBehaviour chỉ forward `Update`.

## 15. Ứng dụng thực tế

Mọi gameplay script Unity dựa trên lifecycle đúng.

## 16. Liên hệ C# thuần

Không có `Update` trong console app — bạn tự loop. Unity = host gọi bạn. Giống game loop thủ công L0/L18.

## 17. Kiểm tra kiến thức

1. `Awake` vs `Start`?  
   **Đáp án:** Awake sớm để wiring; Start sau khi các Awake đã chạy — phụ thuộc chéo an toàn hơn.

2. Physics forces nên ở đâu?  
   **Đáp án:** `FixedUpdate`.

3. Camera follow?  
   **Đáp án:** `LateUpdate` (thường).

4. Có `new PlayerHealth()` được không?  
   **Đáp án:** Không cho MonoBehaviour — dùng AddComponent / prefab.

5. Subscribe event nên pair với?  
   **Đáp án:** `OnEnable` / `OnDisable` (hoặc Destroy tương ứng).
