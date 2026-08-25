# Chương 12 — Strategy

## 1. Mục tiêu học

- Thay thuật toán **lúc runtime** qua interface
- Thay `switch` behavior (OCP)
- Unity: AI hành vi, damage formula, targeting

## 2. Điều kiện tiên quyết

- OCP (L16)
- Delegates (có thể dùng `Func` như strategy nhẹ)

## 3. Khái niệm

**Strategy:** định nghĩa họ thuật toán, đóng gói từng cái, thay thế được. Context giữ `IStrategy` và ủy thác.

## 4. Mô hình tư duy

```text
Context.SetStrategy(A) → Execute()
                 (B)
```

## 5. Cú pháp

```csharp
public interface IMoveStrategy
{
    Vector3 Next(Vector3 current, Vector3 target);
}

public sealed class Mover
{
    private IMoveStrategy _strategy;
    public Mover(IMoveStrategy s) => _strategy = s;
    public void SetStrategy(IMoveStrategy s) => _strategy = s;
    public Vector3 Step(Vector3 cur, Vector3 tgt) => _strategy.Next(cur, tgt);
}
```

## 6. Ví dụ

### Cơ bản

```csharp
public sealed class LinearMove : IMoveStrategy
{
    public Vector3 Next(Vector3 c, Vector3 t) => Vector3.MoveTowards(c, t, 1f);
}
```

### Trung cấp — combat

```csharp
public interface IDamageFormula
{
    int Compute(Stats atk, Stats def);
}

public sealed class PhysicalFormula : IDamageFormula { /* ... */ }
public sealed class MagicFormula : IDamageFormula { /* ... */ }

public sealed class AttackService
{
    public int Hit(IDamageFormula formula, Stats a, Stats d) => formula.Compute(a, d);
}
```

### Nâng cao / Unity

Đổi AI: `PatrolStrategy` → `ChaseStrategy` khi detect player (kết hợp State chương 14).

## 7. Lỗi thường gặp

| Hiện tượng | Cách xử lý |
|------------|------------|
| Strategy biết quá nhiều Context | Truyền data cần thiết qua args |
| Quá nhiều strategy 1 dòng | `Func<>` đủ |
| Null strategy | Null object / require ctor |

## 8. Gỡ lỗi

Unit test từng strategy độc lập; context chỉ test ủy thác.

## 9. Best practices

- Strategy thuần, không side-effect ẩn nếu có thể.  
- Inject mặc định; đổi runtime khi gameplay cần.  
- Data-driven: SO giữ params, strategy giữ công thức.

## 10. Bài tập

**Bài 1** — Sort strategy Bubble/Quick (stub).  
**Bài 2** — Payment strategy Card/Wallet.  
**Bài 3** — Enemy: Flee vs Aggressive.  
**Bài 4** — So sánh vs State.

## 11. Gợi ý

State mang *chu trình trạng thái*; Strategy mang *thuật toán thay thế* không nhất thiết có transition table.

## 12. Đáp án

```csharp
public interface IPaymentStrategy { void Pay(decimal amount); }
public sealed class Checkout
{
    private IPaymentStrategy _pay;
    public Checkout(IPaymentStrategy pay) => _pay = pay;
    public void SetPayment(IPaymentStrategy pay) => _pay = pay;
    public void Complete(decimal amount) => _pay.Pay(amount);
}
```

## 13. Đáp án thay thế

Dictionary enum → delegate; vẫn là Strategy kiểu functional.

## 14. Thử thách

Targeting: nearest / lowest HP / random — đổi theo skill.

## 15. Ứng dụng thực tế

- Compression algorithms  
- Pricing rules  
- Validation strategies

## 16. Liên hệ Unity

- Capstone combat formula  
- Steering behaviors  
- Difficulty modifiers

## 17. Kiểm tra kiến thức

1. Strategy thay gì runtime? **Thuật toán.**  
2. Liên OCP? **Thêm strategy class mới.**  
3. Khác State? **State nhấn transition; Strategy nhấn swap algorithm.**  
4. `Func` có là Strategy? **Có, dạng nhẹ.**  
5. Context nên `switch` type strategy? **Không — polymorphism.**
