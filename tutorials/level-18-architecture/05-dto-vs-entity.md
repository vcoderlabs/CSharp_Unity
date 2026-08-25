# Chương 5 — DTO vs Entity

## 1. Mục tiêu học

- Phân biệt **Entity** (domain + identity + behavior) và **DTO** (data transfer)
- Không serialize entity ra API/UI một cách mù quáng
- Mapping thủ công / library; tránh over-map
- ViewModel / Message contract liên hệ

## 2. Điều kiện tiên quyết

- Entity ở Clean Architecture
- Record/class C#

## 3. Khái niệm

| | Entity | DTO |
|---|--------|-----|
| Mục đích | Mô hình nghiệp vụ | Vận chuyển dữ liệu qua biên |
| Behavior | Có (methods, invariants) | Thường không |
| Identity | Có (Id) | Có thể chỉ data phẳng |
| Phụ thuộc | Domain | Presentation/App contracts |

Lộ `PasswordHash`, navigation EF vòng lặp JSON = lý do tách DTO.

## 4. Mô hình tư duy

```text
Entity (trong) —map→ DTO (ra UI/API/file)
DTO —map→ Command/Entity (vào use case)
Đừng để UI sửa field entity bỏ qua invariant
```

## 5. Cú pháp

```csharp
public sealed class Quest // entity
{
    public Guid Id { get; }
    public string Title { get; private set; }
    public bool IsCompleted { get; private set; }
    public void Complete() { /* ... */ }
}

public sealed record QuestListItemDto(Guid Id, string Title, bool IsCompleted);
```

## 6. Ví dụ

### Cơ bản — bad

```csharp
return JsonSerializer.Serialize(questEntity); // có thể lộ field nội bộ / cycle
```

### Trung cấp — good

```csharp
public QuestListItemDto ToDto(Quest q)
    => new(q.Id, q.Title, q.IsCompleted);
```

### Nâng cao

Request DTO `CompleteQuestRequest` → use case; Response DTO riêng. Versioning DTO khi API public.

## 7. Lỗi thường gặp

| Hiện tượng | Cách xử lý |
|------------|------------|
| DTO trùng 100% entity mãi | Vẫn tách ở biên public; nội bộ có thể dùng chung tạm |
| Anemic entity + logic ở DTO | Đẩy rule vào entity |
| AutoMapper che mapping sai | Test map; map tay chỗ critical |
| Entity dùng làm EF + Domain cứng | Persistence model tách nếu phức tạp |

## 8. Gỡ lỗi

Snapshot test JSON DTO. Kiểm tra không có field nhạy cảm. Breakpoint map layer.

## 9. Best practices

- Đặt DTO ở Application.Contracts / Presentation.  
- Tên rõ `XxxDto` / `XxxResponse`.  
- Immutable DTO (`record`) ưu tiên.  
- Không DTO trong trung tâm Domain.

## 10. Bài tập

**Bài 1** — Entity `Player` → `PlayerHudDto` (chỉ name, hp).  
**Bài 2** — `CreateQuestRequest` validate cơ bản ở biên.  
**Bài 3** — Chỉ ra 3 field không nên trả API.  
**Bài 4** — So ViewModel vs DTO.

## 11. Gợi ý

Bài 3: password hash, internal flags, PII thừa. ViewModel = DTO hướng UI state.

## 12. Đáp án

```csharp
public sealed record PlayerHudDto(string Name, int Hp, int MaxHp);

public static PlayerHudDto From(Player p)
    => new(p.Name, p.Health.Current, p.Health.Max);
```

## 13. Đáp án thay thế

CQRS: read model DTO tối ưu query; write side entity — không bắt buộc cùng shape.

## 14. Thử thách

Map cây Inventory entity → DTO phẳng cho UI list (không circular refs).

## 15. Ứng dụng thực tế

- REST/GraphQL contracts  
- Message bus payloads  
- Save file versioning (DTO schema)

## 16. Liên hệ Unity

- UI bind DTO/read model  
- Network packets = DTO  
- ScriptableObject data ≠ runtime entity state

## 17. Kiểm tra kiến thức

1. DTO để làm gì? **Chuyển data qua biên.**  
2. Entity khác? **Identity + behavior + invariants.**  
3. Serialize entity rủi ro? **Lộ field, cycle, bỏ qua rule.**  
4. DTO nên có `Complete()`? **Không — đó là entity/use case.**  
5. Đặt DTO ở Domain? **Không khuyến khích.**
