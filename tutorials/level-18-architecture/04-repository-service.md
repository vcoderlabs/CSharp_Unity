# Chương 4 — Repository & Service

## 1. Mục tiêu học

- **Repository:** trừu tượng hóa persistence (collection-like)
- **Application Service / Domain Service:** orchestration vs domain rule
- Tránh Repository God và Service chỉ pass-through vô nghĩa
- Unit of Work khái niệm nhẹ

## 2. Điều kiện tiên quyết

- Clean Architecture ports
- Generics (L5) — `IRepository<T>`

## 3. Khái niệm

| Thành phần | Vai trò |
|------------|---------|
| Repository | Load/Save aggregate/entity; giấu SQL/file |
| Application Service / Use Case | Điều phối 1 luồng app; transaction biên |
| Domain Service | Rule không tự nhiên thuộc 1 entity |

Anti-pattern: `UserService` 50 method CRUD mỏng + business rải controller.

## 4. Mô hình tư duy

```text
UseCase
  ├─ đọc/ghi qua IRepository
  ├─ gọi entity.Method() / DomainService
  └─ (optional) IUnitOfWork.SaveChanges
```

## 5. Cú pháp

```csharp
public interface IQuestRepository
{
    Quest? GetById(QuestId id);
    void Add(Quest quest);
    void Update(Quest quest);
}

public sealed class QuestAppService
{
    private readonly IQuestRepository _repo;
    public QuestAppService(IQuestRepository repo) => _repo = repo;

    public void Complete(QuestId id)
    {
        var q = _repo.GetById(id) ?? throw new KeyNotFoundException();
        q.Complete();
        _repo.Update(q);
    }
}
```

## 6. Ví dụ

### Cơ bản — bad

```csharp
public class QuestService
{
    public Quest Get(Guid id) => _db.Quests.Find(id); // lộ EF + không rule
}
```

### Trung cấp — good

Như mục 5 với entity có behavior.

### Nâng cao — UoW

```csharp
public interface IUnitOfWork
{
    Task CommitAsync();
}
// Repositories share same DbContext scope; use case cuối Commit
```

## 7. Lỗi thường gặp

| Hiện tượng | Cách xử lý |
|------------|------------|
| Generic repo lộ `IQueryable` | Không leak query ra Application nếu muốn strict |
| Service trùng repo 1:1 | Gộp use case có nghĩa |
| Repo biết use case khác | Giữ persistence only |

## 8. Gỡ lỗi

Test use case với fake repo. Integration test một repo với DB testcontainer/sqlite.

## 9. Best practices

- Một repo per aggregate gốc (DDD nhẹ).  
- Method repo đặt tên domain (`GetActiveForPlayer`) khi hữu ích.  
- Không để business SQL string trong use case.

## 10. Bài tập

**Bài 1** — `IInventoryRepository` + `AddItem` use case.  
**Bài 2** — Fake in-memory.  
**Bài 3** — Domain service `PricingService` tính tổng.  
**Bài 4** — Khi nào *không* cần repo? (dict in-memory app học)

## 11. Gợi ý

Bài 4: prototype throwaway — YAGNI; vẫn tách nếu muốn luyện.

## 12. Đáp án

```csharp
public sealed class AddItem
{
    private readonly IInventoryRepository _repo;
    public AddItem(IInventoryRepository repo) => _repo = repo;
    public void Handle(PlayerId player, ItemId item, int qty)
    {
        var inv = _repo.Get(player) ?? Inventory.Empty(player);
        inv.Add(item, qty);
        _repo.Save(inv);
    }
}
```

## 13. Đáp án thay thế

Specification pattern cho query phức tạp — chỉ khi thật sự cần.

## 14. Thử thách

Hai repo trong một use case + UoW commit một lần (giả lập).

## 15. Ứng dụng thực tế

- ORM behind repo  
- Microservice persistence adapters

## 16. Liên hệ Unity

- Save repository (JSON/file)  
- ScriptableObject “repo” data read-only  
- Online: API gateway như remote repository

## 17. Kiểm tra kiến thức

1. Repository giấu gì? **Chi tiết persistence.**  
2. Application service khác domain service? **Orchestration app vs rule domain ngang entity.**  
3. Leak `IQueryable` rủi ro? **Coupling infra + logic query lan.**  
4. UoW? **Gom commit transaction.**  
5. Service pass-through xấu vì? **Thêm tầng không giá trị.**
