# Chương 1 — Layered Architecture

## 1. Mục tiêu học

- Tổ chức code theo **tầng** trách nhiệm (Presentation → Application → Domain → Infrastructure)
- Hiểu quy tắc phụ thuộc “chỉ gọi xuống”
- Nhận diện leak: UI đụng SQL, Domain đụng HttpClient
- So sánh với big ball of mud

## 2. Điều kiện tiên quyết

- SRP, DIP (L16)
- Solution nhiều project cơ bản

## 3. Khái niệm

**Layered architecture** chia ứng dụng thành lớp xếp chồng. Mỗi lớp có API cho lớp trên; chi tiết dưới bị che.

| Layer điển hình | Trách nhiệm |
|-----------------|-------------|
| Presentation | UI, Controller, CLI |
| Application | Use cases / orchestration |
| Domain | Entity, rule thuần |
| Infrastructure | DB, file, email, HTTP |

Classic: trên phụ thuộc dưới. (Clean Architecture *đảo* hướng phụ thuộc — chương 2.)

## 4. Mô hình tư duy

```text
[UI] → [Application] → [Domain]
                ↘         ↘
                 [Infrastructure]
(ở layered cổ điển Infra thường bị Application gọi trực tiếp)
```

## 5. Cú pháp

Cấu trúc solution ví dụ:

```text
MyApp.Presentation/
MyApp.Application/
MyApp.Domain/
MyApp.Infrastructure/
```

Project reference: Presentation → Application → Domain; Infrastructure → Domain (và được Application dùng qua interface — tốt hơn).

## 6. Ví dụ

### Cơ bản — vi phạm

```csharp
// Controller gọi SqlConnection trực tiếp → bỏ qua layer
public class OrdersController
{
    public Order Get(Guid id)
    {
        using var c = new SqlConnection("...");
        // ...
        return order;
    }
}
```

### Trung cấp — đúng hướng hơn

```csharp
public class OrdersController
{
    private readonly IOrderService _orders;
    public OrdersController(IOrderService orders) => _orders = orders;
    public OrderDto Get(Guid id) => _orders.Get(id);
}
```

### Nâng cao

Application chỉ phụ thuộc `IOrderRepository` ( Domainside port ); Infrastructure implement. Đây đã nghiêng về Clean.

## 7. Lỗi thường gặp

| Hiện tượng | Cách xử lý |
|------------|------------|
| Domain reference EF | Đảo: interface ở Domain/Application |
| “Utility” mọi tầng dùng lung tung | Đặt đúng layer hoặc Shared Kernel mỏng |
| Layer giả (folder only) | Enforce bằng csproj references |

## 8. Gỡ lỗi

Thêm reference Domain → Infrastructure: nếu compile được theo chiều cấm → sai. Dùng test kiến trúc (NetArchTest) nếu muốn tự động.

## 9. Best practices

- Tên layer theo team thống nhất.  
- Đừng tạo layer trống “cho đủ bộ”.  
- Logging/cross-cutting: cẩn thận không thành God helper.

## 10. Bài tập

**Bài 1** — Vẽ 4 layer cho app Quest.  
**Bài 2** — Liệt kê 5 class mẫu thuộc layer nào.  
**Bài 3** — Chỉ ra 1 hướng reference bị cấm.  
**Bài 4** — So Big Ball of Mud.

## 11. Gợi ý

`Quest` entity → Domain; `SqlQuestRepository` → Infra; `CompleteQuest` → Application; `QuestController` → Presentation.

## 12. Đáp án

Cấm: `Domain` → `Infrastructure`. Cho phép: `Infrastructure` → `Domain` (implement interface).

## 13. Đáp án thay thế

3 layer gộp Application+Domain khi app rất nhỏ — ghi chú trade-off.

## 14. Thử thách

Tạo 4 csproj trống + references đúng; cố tình reference sai để xem compile.

## 15. Ứng dụng thực tế

- Doanh nghiệp N-tier cổ điển  
- Monolith có folder layers

## 16. Liên hệ Unity

- Presentation = MonoBehaviour/UI  
- Domain = pure C# assemblies  
- Infra = PlayerPrefs, Addressables, network  
- Tránh mọi script reference mọi thứ

## 17. Kiểm tra kiến thức

1. Layered tổ chức theo? **Tầng trách nhiệm.**  
2. Quy tắc gọi? **Thường chỉ phụ thuộc chiều xuống / vào trong.**  
3. Domain đụng SQL xấu vì? **Lẫn rule với I/O — khó test/đổi.**  
4. Folder đủ thành layer? **Không — cần kỷ luật reference.**  
5. Clean khác điểm chính? **Hướng phụ thuộc vào Domain (chương 2).**
