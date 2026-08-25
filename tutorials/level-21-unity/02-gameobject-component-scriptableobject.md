# Chương 2 — GameObject, Component & ScriptableObject

## 1. Mục tiêu học

- Hiểu mô hình **composition**: GameObject + Components
- Phân biệt prefab instance vs asset
- Dùng **ScriptableObject** cho data chia sẻ
- Tách “data thiết kế” khỏi “state runtime”

## 2. Điều kiện tiên quyết

- Chương 1 lifecycle
- OOP composition (L2)

## 3. Khái niệm

**GameObject:** container có `Transform` (+ components).  
**Component:** hành vi/data gắn trên GO (`Rigidbody`, script của bạn…).  
**Prefab:** mẫu đóng gói để instantiate.  
**ScriptableObject (SO):** asset data trên đĩa, không cần GO; nhiều object đọc chung.

```text
Player (GameObject)
  Transform
  Rigidbody
  PlayerMotor (MB)
  PlayerHealth (MB) ──refs──► WarriorStats (ScriptableObject asset)
```

## 4. Mô hình tư duy

```text
Scene hierarchy = instances đang sống
Project window  = assets (prefab, SO, material)

Runtime state (HP hiện tại)  →  trên instance / class runtime
Design data (max HP, icon)   →  ScriptableObject / prefab defaults
```

## 5. Cú pháp

```csharp
[CreateAssetMenu(menuName = "Game/Hero Stats")]
public class HeroStats : ScriptableObject
{
    public string DisplayName;
    public int MaxHp;
    public float MoveSpeed;
}

public class Hero : MonoBehaviour
{
    [SerializeField] private HeroStats stats;
    private int _hp;

    void Awake() => _hp = stats.MaxHp;
}
```

```csharp
var go = Instantiate(prefab, position, rotation);
Destroy(go);
```

## 6. Ví dụ

### Cơ bản — GetComponent

```csharp
var anim = GetComponent<Animator>();
var child = transform.Find("Weapon");
```

### Trung cấp — SO database

```csharp
[CreateAssetMenu(menuName = "Game/Item Database")]
public class ItemDatabase : ScriptableObject
{
    public ItemDefinition[] Items;
    public ItemDefinition Get(string id) => System.Array.Find(Items, i => i.Id == id);
}
```

### Nâng cao — đừng sửa SO runtime nếu shared

```csharp
// Nguy hiểm: stats.MaxHp = 999; // sửa asset (Editor) / shared state
// Tốt: runtime copy hoặc field riêng _currentHp
```

Trong Editor, sửa SO lúc Play có thể ghi xuống asset — cẩn thận.

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Null stats | Quên gán Inspector | Require / validate Awake |
| Mọi enemy cùng HP bug | Sửa shared SO | Tách runtime state |
| `Find` sâu hierarchy | Mong manh | Serialize ref |
| God GameObject | Quá nhiều component | Tách child / hệ thống |

## 8. Gỡ lỗi

1. Inspector missing script (màu hồng).  
2. Prefab override vs apply.  
3. Log `stats.name` để chắc đúng asset.  
4. Debug.Destroy check đúng instance.

## 9. Best practices

- Composition > kế thừa sâu MonoBehaviour.  
- Một component một trách nhiệm (SRP).  
- SO cho data thiết kế; MB cho orchestration.  
- Prefab variant cho skin/stats gần giống.  
- Tránh singleton SO “global mutable”.

## 10. Bài tập

**Bài 1** — Tạo `EnemyStats` SO + 2 asset (Goblin, Orc); 1 prefab Enemy đọc stats.  
**Bài 2** — Instantiate 5 enemy lúc Start tại vị trí khác nhau.  
**Bài 3** — ItemDatabase SO + method Get.  
**Bài 4** — Cố ý sửa `stats.MaxHp` khi Play; quan sát; sửa lại thiết kế đúng.

## 11. Gợi ý

- Bài 1: CreateAssetMenu.  
- Bài 4: chỉ đổi `_hp` runtime.

## 12. Đáp án

**Enemy**:

```csharp
public class Enemy : MonoBehaviour
{
    [SerializeField] EnemyStats stats;
    int _hp;
    void Awake() => _hp = stats.MaxHp;
    public void TakeDamage(int dmg) => _hp -= dmg;
}
```

## 13. Đáp án thay thế

Addressable SO khác scene. Hybrid: SO base + `RuntimeHeroState` class thường.

## 14. Thử thách

Inventory definition = SO; inventory contents = runtime list serializable save (chuẩn bị Capstone).

## 15. Ứng dụng thực tế

Balance designer chỉnh SO không đụng code. Data-driven design.

## 16. Liên hệ C# thuần

SO ≈ file config + shared instance. Component ≈ strategy gắn object. Prefab ≈ prototype pattern.

## 17. Kiểm tra kiến thức

1. GameObject chứa gì tối thiểu?  
   **Đáp án:** Transform (và identity object).

2. ScriptableObject khác MonoBehaviour?  
   **Đáp án:** Asset data không cần scene GO; không có Update mặc định.

3. Vì sao không lưu current HP trên SO shared?  
   **Đáp án:** Mọi instance dùng chung sẽ dính state.

4. Prefab dùng để?  
   **Đáp án:** Tái sử dụng mẫu object có sẵn components/cấu hình.

5. Composition trong Unity nghĩa là?  
   **Đáp án:** Ghép hành vi bằng nhiều component thay vì kế thừa một class khổng lồ.
