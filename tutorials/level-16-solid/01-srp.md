# Chương 1 — SRP (Single Responsibility Principle)

## 1. Mục tiêu học

- Hiểu SRP: một module/class có **một lý do để thay đổi**
- Nhận diện God class / mixed concerns
- Refactor: tách trách nhiệm, giữ API ổn định
- Áp dụng trên Unity (logic vs presentation vs persistence)

## 2. Điều kiện tiên quyết

- Level 2: class, method, encapsulation
- Level 15: naming, function/class design, coupling/cohesion
- Biết interface cơ bản

## 3. Khái niệm

**SRP (Single Responsibility Principle):** Một class nên có **một trách nhiệm** — hay nói cách khác, **một lý do để đổi** (Robert C. Martin).

Không có nghĩa “một class chỉ một method”. Nghĩa là các method trong class phục vụ **cùng một actor / cùng một thay đổi nghiệp vụ**.

| Vi phạm điển hình | Lý do đổi lẫn nhau |
|-------------------|--------------------|
| `PlayerManager` vừa combat vừa save file vừa UI | Designer đổi UI ≠ đổi combat |
| `OrderService` vừa tính giá vừa gửi email vừa ghi DB | Marketing đổi template email ≠ đổi pricing |
| `MonoBehaviour` vừa input vừa network vừa inventory | Mỗi hệ thống đổi theo nhịp khác |

## 4. Mô hình tư duy

```text
Hỏi: “Ai / điều gì khiến class này phải sửa?”
→ 1 câu trả lời rõ ràng  → OK (SRP)
→ 2+ lý do độc lập       → Cần tách

GodClass
  ├─ tính damage        → CombatCalculator
  ├─ lưu file           → SaveService
  └─ cập nhật HUD       → HudPresenter
```

## 5. Cú pháp

Không có keyword “SRP”. Dấu hiệu thiết kế:

```csharp
// Tách theo trách nhiệm + phụ thuộc rõ
public sealed class DamageCalculator
{
    public int Calculate(AttackContext ctx) { /* chỉ combat math */ }
}

public sealed class PlayerSaveService
{
    public void Save(PlayerState state, IFileStore store) { /* chỉ I/O */ }
}
```

## 6. Ví dụ

### Bad code

```csharp
public class Player
{
    public string Name { get; set; }
    public int Hp { get; set; }
    public int Gold { get; set; }

    public void TakeDamage(int amount)
    {
        Hp -= amount;
        Console.WriteLine($"[UI] HP bar: {Hp}");           // UI
        File.WriteAllText("player.json", $"{Name}:{Hp}"); // I/O
        if (Hp <= 0)
            SmtpClientSend("admin@game.com", "Player died"); // email
    }
}
```

### Problem

- Đổi format HUD → sửa `Player`
- Đổi đường dẫn save → sửa `Player`
- Đổi SMTP → sửa `Player`
- Khó unit test `TakeDamage` (phụ thuộc file + mạng)

### Refactor

1. Giữ `Player` / domain chỉ đổi HP.
2. Tách `IPlayerPresenter`, `IPlayerRepository`, `INotificationService`.
3. Orchestrator (use case) gọi các phụ thuộc sau khi domain đổi.

### Good code

```csharp
public sealed class Player
{
    public string Name { get; }
    public int Hp { get; private set; }

    public Player(string name, int hp) { Name = name; Hp = hp; }

    public void TakeDamage(int amount)
    {
        if (amount < 0) throw new ArgumentOutOfRangeException(nameof(amount));
        Hp = Math.Max(0, Hp - amount);
    }

    public bool IsDead => Hp <= 0;
}

public sealed class ApplyDamageUseCase
{
    private readonly IPlayerPresenter _ui;
    private readonly IPlayerRepository _repo;
    private readonly INotificationService _notify;

    public ApplyDamageUseCase(
        IPlayerPresenter ui,
        IPlayerRepository repo,
        INotificationService notify)
    {
        _ui = ui;
        _repo = repo;
        _notify = notify;
    }

    public void Execute(Player player, int amount)
    {
        player.TakeDamage(amount);
        _ui.ShowHp(player.Hp);
        _repo.Save(player);
        if (player.IsDead)
            _notify.PlayerDied(player.Name);
    }
}
```

### Unity example

```csharp
// Bad: một MonoBehaviour ôm hết
// public class PlayerController : MonoBehaviour { /* input + combat + save + UI */ }

// Better: domain thuần + components mỏng
public sealed class Health
{
    public int Current { get; private set; }
    public Health(int max) => Current = max;
    public void Damage(int n) => Current = Mathf.Max(0, Current - n);
}

public class HealthView : MonoBehaviour
{
    [SerializeField] private Slider _bar;
    public void Render(int hp, int max) => _bar.value = (float)hp / max;
}

public class PlayerCombat : MonoBehaviour
{
    private Health _health = new(100);
    [SerializeField] private HealthView _view;

    public void Hit(int dmg)
    {
        _health.Damage(dmg);
        _view.Render(_health.Current, 100);
    }
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Tách quá vụn (1 method / class) | Hiểu SRP cực đoan | Gộp theo *lý do đổi*, không theo số dòng |
| Vẫn God class sau “refactor” | Chỉ đổi tên method | Vẽ danh sách lý do đổi trước khi tách |
| Circular dependency sau tách | Tách sai hướng phụ thuộc | Dùng interface + use case ở giữa |
| Test vẫn chậm | I/O còn nằm trong domain | Domain thuần; I/O ở adapter |

## 8. Gỡ lỗi

1. Liệt kê 3 thay đổi gần nhất từng chạm class — nếu khác “chủ đề” → vi phạm SRP.
2. Đặt breakpoint: một method gọi UI + File + Network → mùi.
3. Viết unit test không mock file: nếu không viết được → trách nhiệm lẫn.

## 9. Best practices

- Đặt tên class theo trách nhiệm: `DamageCalculator`, không `Manager2`.
- Prefer nhiều class nhỏ + orchestration rõ hơn 1 file “ tiện”.
- `MonoBehaviour` = adapter Unity; logic game nên test được ngoài Play Mode.
- SRP đi đôi với cohesion cao (L15).

## 10. Bài tập

**Bài 1** — Tách class `ReportExporter` đang vừa query data, format CSV, vừa gửi email thành ≥ 3 phần.

**Bài 2** — `Inventory` vừa add item vừa ghi log file — tách logging.

**Bài 3** — Viết `CheckoutUseCase` orchestrate `Pricing`, `Payment`, `Receipt` (stub interface).

**Bài 4** — Liệt kê ≥ 3 lý do đổi của một God class tự đặt tên; đề xuất tên class mới.

## 11. Gợi ý

- Bài 1: `IReportQuery`, `ICsvFormatter`, `IEmailSender`.
- Bài 2: `ILogger` inject ctor.
- Bài 3: use case không biết SMTP/SQL cụ thể.
- Bài 4: mỗi lý do đổi ≈ một class hoặc module.

## 12. Đáp án

**Bài 2 (rút gọn):**

```csharp
public interface ILogger { void Info(string message); }

public sealed class Inventory
{
    private readonly List<string> _items = new();
    private readonly ILogger _logger;

    public Inventory(ILogger logger) => _logger = logger;

    public void Add(string item)
    {
        _items.Add(item);
        _logger.Info($"Added {item}");
    }
}
```

**Bài 3:**

```csharp
public sealed class CheckoutUseCase
{
    private readonly IPricing _pricing;
    private readonly IPaymentGateway _pay;
    private readonly IReceiptPrinter _receipt;

    public CheckoutUseCase(IPricing pricing, IPaymentGateway pay, IReceiptPrinter receipt)
    {
        _pricing = pricing;
        _pay = pay;
        _receipt = receipt;
    }

    public void Execute(Cart cart)
    {
        var total = _pricing.Total(cart);
        _pay.Charge(total);
        _receipt.Print(cart, total);
    }
}
```

## 13. Đáp án thay thế

Có thể dùng event (`PlayerDamaged`) thay use case tập trung — vẫn SRP nếu mỗi handler một trách nhiệm. Tránh event bus khổng lồ không kiểm soát (xem L17 Mediator/Observer).

## 14. Thử thách

Refactor một `GameManager` giả (combat + quest + save) thành 3 service + 1 facade mỏng — không đổi hành vi console output.

## 15. Ứng dụng thực tế

- Backend: Controller mỏng, Service theo use case, Repository I/O
- Tooling: CLI tách parse args / business / render
- Team: giảm conflict git khi nhiều người sửa cùng God file

## 16. Liên hệ Unity

- Tách `ScriptableObject` data, `MonoBehaviour` view, C# thuần rules
- Tránh `GameManager` làm network + UI + spawn + economy
- Project chung L15–18: mỗi feature (Quest, Inventory) một cụm class rõ trách nhiệm

## 17. Kiểm tra kiến thức

1. SRP nói về gì?  
   **Đáp án:** Một lý do chính để thay đổi / một trách nhiệm gắn actor.

2. Class một method có luôn đúng SRP?  
   **Đáp án:** Không bắt buộc; SRP không = một method.

3. Vì sao God class khó test?  
   **Đáp án:** Nhiều phụ thuộc phụ (I/O, UI) trộn với logic.

4. `MonoBehaviour` nên giữ logic nặng không?  
   **Đáp án:** Nên mỏng; logic thuần tách ra để test/reuse.

5. SRP liên hệ cohesion thế nào?  
   **Đáp án:** Class SRP thường cohesion cao — các phần cùng phục vụ một mục tiêu.
