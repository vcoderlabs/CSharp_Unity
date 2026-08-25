# Chương 2 — OCP (Open/Closed Principle)

## 1. Mục tiêu học

- Hiểu OCP: **mở để mở rộng, đóng để sửa**
- Nhận diện `switch`/if-else theo “loại” phình ra mỗi feature
- Refactor sang polymorphism / strategy / pipeline
- Áp dụng Unity (weapon, skill, quest reward)

## 2. Điều kiện tiên quyết

- Level 2: inheritance, interface, virtual/override
- Level 16 chương 1 (SRP) — tách class trước khi mở rộng
- Biết delegate/`Func` hữu ích (L7)

## 3. Khái niệm

**OCP (Open/Closed Principle):** Behavior của module nên **mở để mở rộng** (thêm class/plugin) nhưng **đóng với sửa đổi** (không phải mở file lõi mỗi lần thêm loại mới).

Thực tế: không tuyệt đối “không bao giờ sửa”. Mục tiêu là **điểm mở rộng ổn định** (abstraction) để feature mới ít đụng code cũ đã test.

| Dấu hiệu vi phạm | Hướng xử lý |
|------------------|-------------|
| `switch (type)` thêm case liên tục | Interface + class mới |
| Sửa `GameBalance` mỗi skill mới | Data-driven / ScriptableObject + handler |
| Copy-paste block gần giống | Template Method / Strategy (L17) |

## 4. Mô hình tư duy

```text
Đóng:  lõi đã ổn định (dispatcher / pipeline)
Mở:    plugin mới implement cùng contract

Bad:  DiscountService + case Vip + case BlackFriday + case ...
Good: IDiscountRule[]  →  mỗi rule một class
```

## 5. Cú pháp

```csharp
public interface IDiscountRule
{
    bool IsMatch(Order order);
    decimal Apply(Order order, decimal current);
}

public sealed class DiscountPipeline
{
    private readonly IReadOnlyList<IDiscountRule> _rules;
    public DiscountPipeline(IEnumerable<IDiscountRule> rules) => _rules = rules.ToList();

    public decimal Calculate(Order order)
    {
        decimal total = order.Subtotal;
        foreach (var rule in _rules)
            if (rule.IsMatch(order))
                total = rule.Apply(order, total);
        return total;
    }
}
```

## 6. Ví dụ

### Bad code

```csharp
public static class DamageService
{
    public static int Calc(string weaponType, int baseAtk)
    {
        switch (weaponType)
        {
            case "Sword": return baseAtk + 5;
            case "Bow": return baseAtk + 3;
            case "Staff": return baseAtk + 8;
            // thêm "Axe" → sửa file này + risk regression
            default: return baseAtk;
        }
    }
}
```

### Problem

- Mỗi weapon mới = sửa + retest toàn `DamageService`
- Stringly-typed dễ typo
- Khó unit test từng nhánh độc lập khi file phình

### Refactor

1. Định nghĩa `IWeaponDamage`.
2. Mỗi weapon một class (hoặc data + formula strategy).
3. Registry/DI cung cấp implementation — lõi không `switch` theo tên.

### Good code

```csharp
public interface IWeaponDamage
{
    int Calculate(int baseAtk);
}

public sealed class SwordDamage : IWeaponDamage
{
    public int Calculate(int baseAtk) => baseAtk + 5;
}

public sealed class BowDamage : IWeaponDamage
{
    public int Calculate(int baseAtk) => baseAtk + 3;
}

public sealed class CombatService
{
    public int Hit(IWeaponDamage weapon, int baseAtk) => weapon.Calculate(baseAtk);
}

// Thêm Axe = class mới, không sửa CombatService
public sealed class AxeDamage : IWeaponDamage
{
    public int Calculate(int baseAtk) => baseAtk + 7;
}
```

### Unity example

```csharp
public abstract class SkillEffect : ScriptableObject
{
    public abstract void Apply(GameObject caster, GameObject target);
}

[CreateAssetMenu(menuName = "Skills/Damage")]
public class DamageEffect : SkillEffect
{
    public int Amount;
    public override void Apply(GameObject caster, GameObject target)
    {
        target.GetComponent<Health>()?.Damage(Amount);
    }
}

[CreateAssetMenu(menuName = "Skills/Heal")]
public class HealEffect : SkillEffect
{
    public int Amount;
    public override void Apply(GameObject caster, GameObject target)
    {
        target.GetComponent<Health>()?.Heal(Amount);
    }
}

// SkillRunner không sửa khi thêm effect mới — chỉ assign asset
public class SkillRunner : MonoBehaviour
{
    [SerializeField] private SkillEffect _effect;
    public void Cast(GameObject target) => _effect.Apply(gameObject, target);
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Abstraction vô nghĩa | Interface 1 implement mãi mãi | Đợi ≥ 2 biến thể thật (YAGNI) |
| Vẫn sửa lõi mỗi lần | Chưa inject danh sách plugin | DI / config đăng ký rule |
| Explosion class | Mỗi case 1 class quá sớm | Data-driven nếu chỉ khác số |
| `default` nuốt lỗi | Quên đăng ký type | Fail-fast hoặc explicit map |

## 8. Gỡ lỗi

1. Tìm `switch`/`if` theo enum type trong service lõi — ứng viên OCP.
2. Thêm loại giả lập: phải sửa bao nhiêu file? >1 file lõi đã ổn → xấu.
3. Test: thêm rule mới chỉ cần test class mới + 1 integration nhỏ.

## 9. Best practices

- Ổn định **contract** (`IDiscountRule`), thay đổi **implement**.
- Kết hợp OCP với DIP (chương 5): phụ thuộc abstraction.
- Unity: ScriptableObject = điểm mở rộng data + behavior.
- Đừng OCP hóa mọi thứ ngày đầu — tách khi có biến thể thứ 2–3.

## 10. Bài tập

**Bài 1** — Refactor `ShippingCost(string method)` (Standard/Express/Drone) sang interface.

**Bài 2** — Pipeline validation: `IOrderValidator` với `EmptyCart`, `MaxQuantity`.

**Bài 3** — Thêm validator mới `MinTotal` **không sửa** class điều phối.

**Bài 4** — So sánh: khi nào dùng bảng config thay vì class mới?

## 11. Gợi ý

- Bài 1–3: ctor nhận `IEnumerable<T>`.
- Bài 4: chỉ khác số liệu → config; khác công thức → class/strategy.

## 12. Đáp án

```csharp
public interface IShipping
{
    decimal Cost(Order order);
}

public sealed class ExpressShipping : IShipping
{
    public decimal Cost(Order order) => 5m + order.WeightKg * 1.2m;
}

public sealed class ShippingCalculator
{
    private readonly IShipping _shipping;
    public ShippingCalculator(IShipping shipping) => _shipping = shipping;
    public decimal Calc(Order order) => _shipping.Cost(order);
}
```

Pipeline validator:

```csharp
public interface IOrderValidator
{
    void Validate(Order order); // throw nếu invalid
}

public sealed class OrderValidationService
{
    private readonly IReadOnlyList<IOrderValidator> _validators;
    public OrderValidationService(IEnumerable<IOrderValidator> validators)
        => _validators = validators.ToList();

    public void ValidateAll(Order order)
    {
        foreach (var v in _validators) v.Validate(order);
    }
}
```

## 13. Đáp án thay thế

Dùng `Dictionary<ShippingKind, IShipping>` thay inheritance sâu. Hoặc `Func<Order, decimal>` đăng ký — vẫn OCP nếu registry nằm ngoài lõi tính toán.

## 14. Thử thách

Hệ thống quest reward: XP, Item, Currency — thêm `TitleReward` không sửa `RewardGrantor` lõi.

## 15. Ứng dụng thực tế

- Plugin pricing, tax theo quốc gia
- Middleware ASP.NET (pipeline)
- Rule engine khuyến mãi

## 16. Liên hệ Unity

- Weapon/skill/buff qua ScriptableObject + interface
- Tránh `switch (enemyAIType)` trong một `Enemy.cs` 2000 dòng
- Object Pool + factory (L17) cũng phục vụ mở rộng loại projectile

## 17. Kiểm tra kiến thức

1. “Đóng để sửa” nghĩa là gì?  
   **Đáp án:** Lõi ổn định không phải mở mỗi lần thêm biến thể.

2. OCP luôn cấm sửa file cũ?  
   **Đáp án:** Không tuyệt đối; ưu tiên mở rộng qua abstraction.

3. `switch` theo type xấu ở đâu?  
   **Đáp án:** Phải sửa + retest lõi mỗi loại mới.

4. ScriptableObject giúp OCP thế nào?  
   **Đáp án:** Thêm asset/behavior mới không sửa runner.

5. Quan hệ OCP và YAGNI?  
   **Đáp án:** Chỉ tạo điểm mở rộng khi có biến thể thật / dự kiến rõ.
