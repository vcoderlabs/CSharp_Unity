# Chương 3 — Abstract Factory

## 1. Mục tiêu học

- Tạo **họ sản phẩm liên quan** (theme/platform) mà không gắn concrete
- Phân biệt Abstract Factory vs Factory Method
- Biết khi chỉ cần Simple Factory là đủ

## 2. Điều kiện tiên quyết

- Factory (chương 2)
- Interface segregation nhẹ

## 3. Khái niệm

**Abstract Factory:** interface factory tạo *bộ* product (`Button` + `Checkbox` cùng theme). Client dùng factory abstraction — đảm bảo sản phẩm **đồng bộ họ**.

## 4. Mô hình tư duy

```text
IUIFactory → CreateButton(), CreateCheckbox()
DarkFactory / LightFactory
Client chỉ biết IButton, ICheckbox
```

## 5. Cú pháp

```csharp
public interface IUIFactory
{
    IButton Button();
    ICheckbox Checkbox();
}
```

## 6. Ví dụ

### Cơ bản

```csharp
public sealed class DarkUIFactory : IUIFactory
{
    public IButton Button() => new DarkButton();
    public ICheckbox Checkbox() => new DarkCheckbox();
}

void Build(IUIFactory f)
{
    f.Button().Render();
    f.Checkbox().Render();
}
```

### Trung cấp

Chọn factory theo config platform (`MobileFactory` / `DesktopFactory`).

### Nâng cao / Unity

```csharp
public interface IVfxFactory
{
    ParticleSystem Hit();
    AudioClip HitSfx();
}
// FireFactionVfxFactory vs IceFactionVfxFactory — đồng bộ skin phe
```

## 7. Lỗi thường gặp

| Hiện tượng | Cách xử lý |
|------------|------------|
| Abstract Factory cho 1 product | Dùng Simple Factory |
| Họ product lệch nhau | Giữ cùng interface set |
| Quá nhiều class | Theme data-driven |

## 8. Gỡ lỗi

Thêm product thứ 3 vào họ → mọi concrete factory phải cập nhật (trade-off có chủ đích).

## 9. Best practices

- Dùng khi **bắt buộc đồng bộ họ**.  
- Đừng Abstract Factory “cho có”.

## 10. Bài tập

**Bài 1** — `IWorldFactory`: Enemy + LootTable cho biome Forest/Desert.  
**Bài 2** — Client `Encounter` chỉ nhận `IWorldFactory`.  
**Bài 3** — So sánh LOC với Simple Factory đơn.  
**Bài 4** — Khi nào không dùng? (chương 18)

## 11. Gợi ý

ForestEnemy + ForestLoot phải đi cùng — đó là lý do Abstract Factory.

## 12. Đáp án

```csharp
public interface IWorldFactory
{
    IEnemy CreateEnemy();
    ILootTable CreateLoot();
}

public sealed class ForestWorldFactory : IWorldFactory
{
    public IEnemy CreateEnemy() => new Wolf();
    public ILootTable CreateLoot() => new ForestLoot();
}
```

## 13. Đáp án thay thế

Một ScriptableObject “BiomeKit” chứa prefab refs — Abstract Factory kiểu data.

## 14. Thử thách

UI shop + tooltip cùng style pack — thêm Neon style không sửa màn shop.

## 15. Ứng dụng thực tế

- Cross-platform UI kits  
- Multi-DB driver families (hiếm)

## 16. Liên hệ Unity

- Faction cosmetics kit  
- Platform input + UI haptic packs

## 17. Kiểm tra kiến thức

1. Abstract Factory tạo gì? **Họ product liên quan.**  
2. Khác Factory Method? **Nhiều product/family vs một product trong creator.**  
3. Lợi ích chính? **Đồng bộ theme/platform.**  
4. Chi phí? **Nhiều class; thêm product đụng mọi factory.**  
5. Luôn cần? **Không — YAGNI.**
