# Chương 3 — Serialization & Inspector

## 1. Mục tiêu học

- Hiểu Unity **serialize** field nào lên Inspector / scene / prefab
- Dùng `[SerializeField]`, `[FormerlySerializedAs]`, custom drawer cơ bản (ý tưởng)
- Tránh mất dữ liệu khi đổi tên field
- Phân biệt serialize Unity vs `System.Text.Json`

## 2. Điều kiện tiên quyết

- Chương 2 SO / prefab
- Biết access modifier C#

## 3. Khái niệm

Unity serialize **public fields** và **private fields có `[SerializeField]`** (kiểu hỗ trợ): số, string, enum, Vector, reference UnityEngine.Object, List/array một số kiểu, class `[Serializable]`.

**Không** serialize tự động: property thuần, `dictionary` (trừ giải pháp custom), nhiều kiểu .NET phức tạp.

Inspector = UI chỉnh dữ liệu đã serialize trên instance/asset.

## 4. Mô hình tư duy

```text
Code field  ←serialize→  YAML scene/prefab
Đổi tên field không FormerlySerializedAs → mất giá trị Inspector (về default)
```

## 5. Cú pháp

```csharp
[SerializeField] private float speed = 5f;
[SerializeField] private Transform target;
[SerializeField] private List<AudioClip> clips;

[System.Serializable]
public class LootEntry
{
    public string ItemId;
    public int Weight;
}

[SerializeField] private LootEntry[] table;
```

```csharp
using UnityEngine.Serialization;
[FormerlySerializedAs("spd")]
[SerializeField] private float speed;
```

## 6. Ví dụ

### Cơ bản — ẩn public API

```csharp
// Prefer
[SerializeField] int maxHp;
// thay vì public int maxHp; // ai cũng gán từ code ngoài
```

### Trung cấp — OnValidate

```csharp
void OnValidate()
{
    maxHp = Mathf.Max(1, maxHp);
}
```

### Nâng cao — SerializeReference (polymorphism)

```csharp
[SerializeReference] private IEffect effect;
```

(Cần hiểu thêm — dùng khi danh sách effect đa hình trong Inspector.)

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Mất giá trị sau rename | Không FormerlySerializedAs | Attribute hoặc reassign |
| Null reference runtime | Quên kéo ref | RequireComponent / reset check |
| Dictionary không hiện | Không hỗ trợ native | SO list / Odin / custom |
| Property không lưu | Không serialize | Backing field SerializeField |

## 8. Gỡ lỗi

1. Inspector Debug mode xem raw.  
2. Git diff prefab YAML tìm field.  
3. `Debug.Log(target == null)`.  
4. Prefab apply/revert nhầm override.

## 9. Best practices

- Private + SerializeField cho tuning.  
- Range/Min/Header/Tooltip attributes cho designer.  
- Không serialize cache runtime (`_rb`) nếu không cần — gán Awake.  
- Đổi tên cẩn thận trên project đã có content.  
- Versioning save game ≠ Inspector serialize — Capstone M7.

## 10. Bài tập

**Bài 1** — WeaponStats: damage, cooldown, clip — chỉnh trên prefab.  
**Bài 2** — Rename field có FormerlySerializedAs giữ giá trị.  
**Bài 3** — Serializable `WaveConfig` chứa list enemy prefab + count.  
**Bài 4** — OnValidate clamp damage ≥ 0.

## 11. Gợi ý

- Bài 2: đổi tên trong code + attribute cũ.  
- Bài 3: `[System.Serializable]` class lồng.

## 12. Đáp án

```csharp
[System.Serializable]
public class WaveEnemy
{
    public GameObject Prefab;
    public int Count;
}

public class WaveConfig : MonoBehaviour
{
    [SerializeField] WaveEnemy[] entries;
}
```

## 13. Đáp án thay thế

Odin Inspector / UI Toolkit custom editors cho UX designer tốt hơn.

## 14. Thử thách

CustomPropertyDrawer đơn giản hiện màu warning nếu ref null.

## 15. Ứng dụng thực tế

Toàn bộ workflow designer–programmer dựa trên serialization ổn định.

## 16. Liên hệ C# thuần

`[Serializable]` .NET BinaryFormatter lỗi thời; Unity có serializer riêng. Đừng nhầm với JSON save.

## 17. Kiểm tra kiến thức

1. Private field hiện Inspector bằng gì?  
   **Đáp án:** `[SerializeField]`.

2. Property tự serialize?  
   **Đáp án:** Không (trừ pattern backing field).

3. Đổi tên field an toàn bằng?  
   **Đáp án:** `[FormerlySerializedAs("oldName")]`.

4. Vì sao không public hết field?  
   **Đáp án:** Phá encapsulation; API bề mặt rộng.

5. Dictionary Unity serialize native?  
   **Đáp án:** Không hỗ trợ tốt/out-of-box như List.
