# Chương 2 — Clean Architecture

## 1. Mục tiêu học

- Hiểu vòng tròn: Entities → Use Cases → Adapters → Frameworks
- **Dependency Rule:** phụ thuộc chỉ hướng *vào trong*
- Ports & Adapters (Hexagonal) liên hệ
- Áp vào project chung L15–18

## 2. Điều kiện tiên quyết

- Layered (chương 1)
- DIP, Adapter, Facade (L16–17)

## 3. Khái niệm

**Clean Architecture** (Uncle Bob): business rule ở trung tâm, độc lập UI/DB/framework. Chi tiết bên ngoài *phụ thuộc* abstraction bên trong.

| Vòng | Ví dụ |
|------|--------|
| Entities | `Quest`, `Inventory` + invariants |
| Use Cases | `CompleteQuest`, `BuyItem` |
| Interface Adapters | Controllers, Presenters, Gateways |
| Frameworks & Drivers | EF, Unity, ASP.NET, SMTP |

## 4. Mô hình tư duy

```text
        [ Frameworks ]
     [ Controllers / Gateways ]
   [      Use Cases (App)     ]
  [         Entities           ]

Mũi tên phụ thuộc → luôn hướng TÂM
```

## 5. Cú pháp

```csharp
// Domain
public sealed class Quest
{
    public QuestId Id { get; }
    public bool IsCompleted { get; private set; }
    public void Complete()
    {
        if (IsCompleted) throw new InvalidOperationException();
        IsCompleted = true;
    }
}

// Application port
public interface IQuestRepository
{
    Quest? Get(QuestId id);
    void Save(Quest quest);
}

// Application use case
public sealed class CompleteQuest
{
    private readonly IQuestRepository _repo;
    public CompleteQuest(IQuestRepository repo) => _repo = repo;
    public void Handle(QuestId id)
    {
        var q = _repo.Get(id) ?? throw new KeyNotFoundException();
        q.Complete();
        _repo.Save(q);
    }
}
```

## 6. Ví dụ

### Cơ bản — bad

Use case `using Microsoft.EntityFrameworkCore` + query DbSet trực tiếp.

### Trung cấp — good

Như mục 5; `EfQuestRepository` ở Infrastructure implement `IQuestRepository`.

### Nâng cao

Input/Output DTO (Request/Response) ở Application biên — UI không thấy Entity nếu không cần (chương 5).

## 7. Lỗi thường gặp

| Hiện tượng | Cách xử lý |
|------------|------------|
| “Clean” nhưng Domain rỗng anemia | Đặt rule vào entity/domain service |
| Use case gọi UI API | Presenter/callback abstraction |
| 10 project cho todo app | Giảm scale — vẫn giữ hướng phụ thuộc |

## 8. Gỡ lỗi

Domain project: cấm package EF/ASP.NET. Build fail nếu ai đó thêm — đó là feature.

## 9. Best practices

- Use case một hành động ứng dụng.  
- Đặt interface (port) cạnh chỗ *cần*, thường Application.  
- Test domain/use case không DB.

## 10. Bài tập

**Bài 1** — Viết entity `Wallet` không âm gold.  
**Bài 2** — Use case `SpendGold` + port repo.  
**Bài 3** — Fake repo in-memory + test.  
**Bài 4** — Vẽ vòng cho feature Inventory.

## 11. Gợi ý

Invariant trong entity; use case orchestration + persistence.

## 12. Đáp án

```csharp
public sealed class Wallet
{
    public int Gold { get; private set; }
    public Wallet(int gold) => Gold = gold;
    public void Spend(int amount)
    {
        if (amount < 0 || amount > Gold) throw new InvalidOperationException();
        Gold -= amount;
    }
}
```

## 13. Đáp án thay thế

Hexagonal: Primary adapters (CLI/UI) / Secondary (DB) — cùng dependency rule.

## 14. Thử thách

Thêm adapter CLI và adapter ASP.NET Minimal API cùng một use case.

## 15. Ứng dụng thực tế

- Backend enterprise  
- Mobile/shared domain  
- Game logic assemblies dùng lại ngoài Unity Editor tests

## 16. Liên hệ Unity

- Domain/UseCases = asmdef không reference UnityEngine nếu có thể  
- MonoBehaviour = adapter  
- Capstone: Core Architecture milestone

## 17. Kiểm tra kiến thức

1. Dependency Rule? **Phụ thuộc hướng vào trong.**  
2. Use case là gì? **Application-specific business flow.**  
3. EF thuộc vòng nào? **Frameworks/Drivers (ngoài).**  
4. Vì sao dễ test? **Domain/App không cần UI/DB.**  
5. Khác layered cổ điển? **Không để trong cùng phụ thuộc ra ngoài chi tiết.**
