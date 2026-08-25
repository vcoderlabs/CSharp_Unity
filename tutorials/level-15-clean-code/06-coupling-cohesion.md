# Chương 6 — Coupling và Cohesion

## 1. Mục tiêu học

- Định nghĩa **coupling** (độ phụ thuộc) và **cohesion** (độ gắn kết)
- Nhắm **low coupling, high cohesion**
- Nhận diện coupling xấu: cứng `new`, static toàn cục, biết quá chi tiết nội bộ
- Liên hệ outline **Clean Architecture Application** (project chung L15–18)

## 2. Điều kiện tiên quyết

- Chương 1–5
- Level 14: DI / interface
- Đọc README Level 15 phần project chung

## 3. Khái niệm

**Coupling** = mức độ một module phụ thuộc module khác.  
Cao → đổi một chỗ dễ vỡ chỗ khác.

**Cohesion** = mức độ các phần trong một module thuộc về **cùng một chủ đề**.  
Cao → class/file làm một việc rõ.

| Mục tiêu | Ý nghĩa |
|----------|---------|
| Low coupling | Phụ thuộc ít, qua abstraction ổn định |
| High cohesion | Thành viên class cùng một lý do tồn tại |

### Trước — coupling cao, cohesion thấp

```csharp
public class Player
{
    public void Attack()
    {
        var db = new SqlConnection("..."); // cứng infra
        db.Open();
        // combat + SQL + UI text + audio...
        GameObject.Find("HUD").GetComponent<Text>().text = "Hit";
    }
}
```

### Sau — hướng tốt hơn

```csharp
public sealed class CombatService
{
    private readonly ICombatPresenter _presenter;
    public CombatService(ICombatPresenter presenter) => _presenter = presenter;

    public HitResult Attack(Attacker attacker, Target target)
    {
        var result = ResolveHit(attacker, target); // pure-ish domain
        _presenter.ShowHit(result);
        return result;
    }
}
```

Domain không biết SQL/UI cụ thể → coupling thấp hơn; `CombatService` cohesive hơn.

## 4. Mô hình tư duy

```text
        High cohesion
              ▲
              │  ★ đích
              │
              │
Low ◄─────────┼─────────► High
coupling      │         coupling
              │
              ▼
        Low cohesion  (tránh)
```

```text
UI / Controllers
      │  phụ thuộc xuống (abstraction)
Use Cases / Application
      │
Domain (entities, rules)     ← ít phụ thuộc ra ngoài
      ▲
Infrastructure (SQL, HTTP) implement interface của trong
```

Đây là hạt giống **Clean Architecture**: phụ thuộc hướng **vào trong**.

## 5. Cú pháp

Giảm coupling bằng interface + DI (ôn L14):

```csharp
public interface IOrderRepository
{
    void Save(Order order);
}

public sealed class PlaceOrderHandler
{
    private readonly IOrderRepository _orders;
    public PlaceOrderHandler(IOrderRepository orders) => _orders = orders;

    public void Handle(PlaceOrderCommand cmd)
    {
        var order = Order.Create(cmd.CustomerId, cmd.Lines);
        _orders.Save(order);
    }
}
```

Infrastructure:

```csharp
public sealed class SqlOrderRepository : IOrderRepository
{
    public void Save(Order order) { /* EF/SQL */ }
}
```

`PlaceOrderHandler` **không** reference SQL.

## 6. Ví dụ

### Cơ bản — Cohesion

**Thấp:** class `Utilities` có `ParseDate`, `SendEmail`, `HashPassword`.  
**Cao:** `DateParser`, `SmtpEmailSender`, `PasswordHasher` tách.

### Trung cấp — Loại coupling

| Loại (ý) | Ví dụ | Hướng giảm |
|----------|--------|------------|
| Content | Sửa field private class khác | API public rõ |
| Common / global | Static `App.State` mọi nơi | Inject context |
| Control | Flag bảo module khác làm gì | Polymorphism / tách API |
| Transient | A→B→C→D sâu | Facade; đảo phụ thuộc |

### Nâng cao — Outline Clean Architecture Application (L15–18)

Domain ví dụ: **Quest Board** hoặc **Shop Orders** (chọn một).

```text
src/
  Domain/           # Quest, Reward rules — zero infra
  Application/      # AcceptQuest, CompleteQuest handlers + interfaces
  Infrastructure/   # EF, files, email — implement interfaces
  ApiOrConsole/     # composition root, UI/API
tests/
  Domain.Tests/
  Application.Tests/  # fake repos
```

**Việc Level 15:** đặt tên rõ, tách hàm/class, viết 1 use case `AcceptQuest` cohesive; ghi outline.  
**L16–18:** patterns, boundary cứng, hoàn thiện.

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Interface cho mọi class 1-1 vô ích | “Giảm coupling” máy móc | Chỉ trừu tượng hóa chỗ cần swap/test |
| Circular reference project | Coupling vòng | Hướng phụ thuộc một chiều |
| Domain reference EF | Sai chiều phụ thuộc | Interface ở Application/Domain |
| Cohesion thấp vì “folder theo tầng kỹ thuật quá sớm” | Chưa có code | Bắt đầu theo feature rồi tách |
| Static service locator | Coupling ẩn | Constructor injection |

## 8. Gỡ lỗi

1. Đổi schema DB phải sửa UI → coupling xuyên tầng; thiếu abstraction.
2. Không unit test được vì `new Sql...` → giảm coupling bằng inject.
3. Class không đặt được tên → cohesion thấp.
4. Vẽ sơ đồ reference project (`Domain` không reference `Infrastructure`).

## 9. Best practices

- Phụ thuộc vào abstraction ổn định hơn concrete dễ đổi.
- Giữ domain thuần khi bắt đầu Clean Architecture.
- High cohesion trước: tách đúng trách nhiệm rồi mới “vẽ tầng”.
- Cân nhắc: low coupling tuyệt đối có giá (nhiều interface) — đủ dùng là được.
- Composition root duy nhất ở mép app.

## 10. Bài tập

**Bài 1** — Phân loại: `ReportExporter` gọi thẳng `File.WriteAllText` và `SmtpClient` — coupling/cohesion thế nào? Đề xuất tách.

**Bài 2** — Viết interface tối thiểu để `AcceptQuestHandler` không biết lưu file hay SQL.

**Bài 3** — Trong outline Quest Board, liệt kê 3 class Domain cohesive.

**Bài 4** — Giải thích vì sao `Domain` reference `Infrastructure` là xấu (3–5 câu).

## 11. Gợi ý

- Bài 1: cohesion thấp (file+email); inject `IFileStorage`, `IEmailSender`.
- Bài 2: `IQuestRepository.Save/Get`.
- Bài 3: `Quest`, `QuestId`, `Reward`, `QuestProgress`…
- Bài 4: domain bị dính chi tiết kỹ thuật; không tái sử dụng/test độc lập; đảo chiều phụ thuộc.

## 12. Đáp án

**Bài 2:**

```csharp
public interface IQuestRepository
{
    Quest? Get(string questId);
    void Save(Quest quest);
}

public sealed class AcceptQuestHandler
{
    private readonly IQuestRepository _quests;
    public AcceptQuestHandler(IQuestRepository quests) => _quests = quests;

    public void Handle(string playerId, string questId)
    {
        var quest = _quests.Get(questId) ?? throw new InvalidOperationException("missing");
        quest.Accept(playerId);
        _quests.Save(quest);
    }
}
```

**Bài 4:** Domain phải ổn định và độc lập công nghệ lưu trữ/UI. Reference Infrastructure buộc domain biết SQL/HTTP, tăng coupling, phá test thuần, làm Clean Architecture mất ý nghĩa (phụ thuộc phải hướng vào trong).

## 13. Đáp án thay thế

Dùng mediator (`IRequestHandler`) ở Application — vẫn low coupling nếu handler không đụng SQL concrete. Hoặc modular monolith theo feature folder trước khi tách assembly.

## 14. Thử thách

Tạo skeleton solution 3 project `Domain`, `Application`, `Infrastructure` + test Application với fake repo — chưa cần API. Đây là khởi động Clean Architecture Application (L15–18).

## 15. Ứng dụng thực tế

- Microservices: coupling qua contract/API versioning
- Monolith: module boundary + cohesion theo feature
- Đội lớn: ownership theo cohesive module

## 16. Liên hệ Unity

- Assembly definitions: `Game.Domain` không reference `UnityEngine.UI`
- Presentation (MonoBehaviour) mỏng; logic combat cohesive trong service
- Tránh mọi script `FindObjectOfType` lung tung — coupling ẩn vào scene

## 17. Kiểm tra kiến thức

1. Coupling cao nghĩa là gì?  
   **Đáp án:** Module phụ thuộc chặt / nhiều vào module khác — khó đổi độc lập.

2. High cohesion nghĩa là gì?  
   **Đáp án:** Các phần trong module cùng một trách nhiệm/chủ đề.

3. Mục tiêu phổ biến?  
   **Đáp án:** Low coupling, high cohesion.

4. DI giúp coupling thế nào?  
   **Đáp án:** Phụ thuộc abstraction; thay implementation mà không sửa consumer.

5. Clean Architecture nhấn hướng phụ thuộc nào?  
   **Đáp án:** Vào trong — infrastructure phụ thuộc domain/application, không ngược lại.

---

## Bước nhẹ cho project chung L15–18

Trước khi sang Level 16:

- [ ] Chọn domain (Quest / Shop / …)
- [ ] Viết outline thư mục + 1 use case
- [ ] Đặt tên class theo chương 1–2
- [ ] (Thử thách) Skeleton `Domain` + `Application` + fake test
