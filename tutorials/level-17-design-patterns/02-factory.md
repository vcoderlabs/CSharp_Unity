# Chương 2 — Factory (Simple / Factory Method)

## 1. Mục tiêu học

- Phân biệt Simple Factory vs Factory Method
- Giấu logic tạo object phức tạp / theo type
- Liên hệ OCP khi thêm product mới
- Unity: spawn enemy/projectile qua factory (+ pool sau)

## 2. Điều kiện tiên quyết

- Interface, inheritance
- OCP (L16)

## 3. Khái niệm

**Simple Factory:** một class/`static` method chọn tạo concrete.  
**Factory Method:** subclass/`creator` quyết định product cụ thể (GoF).

Mục tiêu: caller phụ thuộc `IProduct`, không `new Concrete` rải rác.

## 4. Mô hình tư duy

```text
Client → IFactory.Create(params) → IProduct
Thêm loại mới: mở factory / thêm creator — ít sửa client
```

## 5. Cú pháp

```csharp
public interface IEnemy { void Spawn(); }

public static class EnemyFactory
{
    public static IEnemy Create(string id) => id switch
    {
        "slime" => new Slime(),
        "orc" => new Orc(),
        _ => throw new ArgumentException(id)
    };
}
```

## 6. Ví dụ

### Cơ bản — Simple Factory

```csharp
var e = EnemyFactory.Create("slime");
e.Spawn();
```

### Trung cấp — Factory Method

```csharp
public abstract class MissionCreator
{
    public Mission Start()
    {
        var m = CreateMission();
        m.OnStart();
        return m;
    }
    protected abstract Mission CreateMission();
}

public sealed class DailyMissionCreator : MissionCreator
{
    protected override Mission CreateMission() => new DailyMission();
}
```

### Nâng cao / Unity

```csharp
public class ProjectileFactory : MonoBehaviour
{
    [SerializeField] private Projectile _prefab;
    public Projectile Create(Vector3 pos, Quaternion rot)
        => Instantiate(_prefab, pos, rot);
    // Production: lấy từ Object Pool thay Instantiate mỗi lần
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Cách xử lý |
|------------|------------|
| Factory chứa cả business | Tách create vs use |
| String magic id | Enum / catalog SO |
| Switch khổng lồ | Registry `Dictionary` + OCP |

## 8. Gỡ lỗi

1. Client còn `new ConcreteX` — factory chưa bao phủ.  
2. Null prefab Unity — chưa assign SerializeField.

## 9. Best practices

- Trả về abstraction.  
- Factory không phải God.  
- Kết hợp Pool cho Instantiate nóng.

## 10. Bài tập

**Bài 1** — Factory `IShape` Circle/Rect.  
**Bài 2** — Registry `Dictionary<string, Func<IEnemy>>`.  
**Bài 3** — Factory Method 2 loại notification Email/Push.  
**Bài 4** — Giải thích khác Abstract Factory (chương 3).

## 11. Gợi ý

Registry: `Register("slime", () => new Spime())` rồi `Create(id)`.

## 12. Đáp án

```csharp
public static class ShapeFactory
{
    public static IShape Create(string t) => t switch
    {
        "circle" => new Circle(),
        "rect" => new Rect(),
        _ => throw new ArgumentException(t)
    };
}
```

## 13. Đáp án thay thế

DI container resolve theo key — factory “ẩn” trong container.

## 14. Thử thách

Quest reward factory: XP/Item/Currency — thêm Title không sửa client grant.

## 15. Ứng dụng thực tế

- Parser tạo node AST  
- UI control factory theo theme

## 16. Liên hệ Unity

- Prefab factory + pool (combat VFX)  
- ScriptableObject factory data

## 17. Kiểm tra kiến thức

1. Factory giúp gì? **Giấu tạo concrete, tập trung OCP.**  
2. Simple vs Method? **Static/chọn tập trung vs creator override.**  
3. Trả concrete type xấu? **Client còn coupling.**  
4. Pool liên hệ? **Factory lấy/trả instance tái sử dụng.**  
5. Switch trong factory OK? **Tạm; registry scale tốt hơn.**
