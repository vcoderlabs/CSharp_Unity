# Chương 6 — Modular Architecture

## 1. Mục tiêu học

- Chia hệ thống theo **module/feature** (vertical) ngoài/kết hợp layer (horizontal)
- Định nghĩa API module công khai vs nội bộ
- Giảm coupling giữa Inventory / Quest / Combat
- Hướng tới maintain monolith trước khi microservices

## 2. Điều kiện tiên quyết

- Clean / Layered
- ISP, Mediator nhẹ (L16–17)
- Solution multi-project

## 3. Khái niệm

**Modular monolith:** một deploy, nhiều module biên giới rõ (namespace/project/`InternalsVisibleTo`). Giao tiếp qua interface công khai / domain events — không đụng DB table của module khác trực tiếp.

| Hướng cắt | Ý nghĩa |
|-----------|---------|
| Horizontal layers | Kỹ thuật (UI/App/Domain/Infra) |
| Vertical modules | Nghiệp vụ (Shop, Quest, Party) |

Lý tưởng: mỗi module có đủ layer mỏng bên trong, hoặc shared kernel rất mỏng.

## 4. Mô hình tư duy

```text
[Host]
 ├─ Modules.Quest      (public: IQuestApi)
 ├─ Modules.Inventory  (public: IInventoryApi)
 └─ Modules.Combat
Quest không using Inventory.Internal.*; chỉ IInventoryApi.Reserve(...)
```

## 5. Cú pháp

```csharp
// Modules.Inventory.Contracts
public interface IInventoryApi
{
    bool TryConsume(ItemId id, int qty);
}

// Modules.Quest → reference Contracts only
public sealed class TurnInQuest
{
    private readonly IInventoryApi _inventory;
    public void Handle(...)
    {
        if (!_inventory.TryConsume(requiredItem, 1))
            throw new InvalidOperationException("missing item");
        // complete quest...
    }
}
```

## 6. Ví dụ

### Cơ bản — bad

`QuestService` query bảng `Items` SQL trực tiếp copy từ Inventory.

### Trung cấp — good

Gọi `IInventoryApi`; Inventory module sở hữu persistence item.

### Nâng cao

Domain event `QuestCompleted` → Inventory lắng nghe phát reward (async/in-process bus) — giảm phụ thuộc cứng chiều ngược.

## 7. Lỗi thường gặp

| Hiện tượng | Cách xử lý |
|------------|------------|
| Shared Kernel phình | Chỉ primitive/IDS thật sự chung |
| Circular module refs | Events / tách contracts project |
| Module giả (folder) | Enforce project + public API |
| Microservices quá sớm | Modular monolith trước |

## 8. Gỡ lỗi

Architecture test: `Quest` không reference `Inventory.Infrastructure`. Dependency graph tool.

## 9. Best practices

- `*.Contracts` assembly cho API module.  
- Database: schema/table prefix theo module khi share DB.  
- Team ownership theo module.  
- Extract service chỉ khi module + scale đòi hỏi.

## 10. Bài tập

**Bài 1** — Chia app mẫu thành 3 module + contracts.  
**Bài 2** — Liệt kê 5 thứ được/không được share.  
**Bài 3** — Thay reference cứng bằng event `ItemConsumed`.  
**Bài 4** — So modular monolith vs microservices (3 ý).

## 11. Gợi ý

Share: `PlayerId`, clock, logging abstraction. Không share: EF entities nội bộ.

## 12. Đáp án

```csharp
// Contracts
public sealed record QuestCompleted(Guid QuestId, Guid PlayerId);

// Inventory handler
public sealed class GrantQuestRewardHandler
{
    public void Handle(QuestCompleted e) { /* add items */ }
}
```

## 13. Đáp án thay thế

MediatR in-process = bus module. Feature folders trong một project — bước nhẹ trước multi-project.

## 14. Thử thách

Thêm module `Achievement` chỉ listen events — không reference Quest internals.

## 15. Ứng dụng thực tế

- Large .NET monoliths  
- Chuẩn bị tách service  
- Plugin modules

## 16. Liên hệ Unity

- asmdef theo hệ thống (Combat, UI, Core)  
- Capstone nhiều milestone = nhiều module  
- Tránh mọi thứ trong `Assets/_Game` một asmdef khổng lồ

## 17. Kiểm tra kiến thức

1. Modular cắt theo? **Feature/bounded context.**  
2. Contracts để? **API công khai giữa module.**  
3. Vì sao đừng đụng DB module khác? **Phá encapsulation — khó tách sau.**  
4. Shared Kernel nên? **Mỏng.**  
5. Microservices luôn tốt hơn? **Không — chi phí phân tán cao.**
