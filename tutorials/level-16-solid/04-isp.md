# Chương 4 — ISP (Interface Segregation Principle)

## 1. Mục tiêu học

- Hiểu ISP: **client không bị ép phụ thuộc method không dùng**
- Nhận diện fat interface / “God interface”
- Tách interface theo vai trò client
- Áp dụng Unity (input, save, combat callbacks)

## 2. Điều kiện tiên quyết

- Level 2: interface, explicit implement
- Level 16: SRP + LSP
- Generics cơ bản (L5) hữu ích khi tách `IRepository`

## 3. Khái niệm

**ISP (Interface Segregation Principle):** Nhiều interface **cụ thể theo nhu cầu client** tốt hơn một interface “bự” bắt mọi implementor phải stub method thừa.

| Fat interface | Hệ quả |
|---------------|--------|
| `IWorker` có `Work`, `Eat`, `Sleep` cho robot | Robot implement `Eat` giả |
| `IPlayerServices` 40 method | Class fake test khổng lồ |
| `IUnityServices` Get/Save/Network/UI | Thay đổi UI kéo recompile mọi nơi |

## 4. Mô hình tư duy

```text
Hỏi từng client: “Tôi cần method nào?”
→ Gom method theo cụm client (role interface)
→ Class có thể implement nhiều interface nhỏ

IPlayerReader          IPlayerWriter
     ▲                      ▲
     └──── PlayerRepo ──────┘
```

## 5. Cú pháp

```csharp
public interface IOrderReader
{
    Order? GetById(Guid id);
}

public interface IOrderWriter
{
    void Save(Order order);
}

// Client chỉ đọc → chỉ phụ thuộc IOrderReader (ISP)
public sealed class OrderQueryService
{
    private readonly IOrderReader _reader;
    public OrderQueryService(IOrderReader reader) => _reader = reader;
}
```

## 6. Ví dụ

### Bad code

```csharp
public interface IEntity
{
    void Move(Vector3 d);
    void Attack(IEntity target);
    void OpenShop();
    void SaveToCloud();
    void PlayEmote(string id);
}

public class Tree : IEntity
{
    public void Move(Vector3 d) => throw new NotSupportedException();
    public void Attack(IEntity t) => throw new NotSupportedException();
    public void OpenShop() => throw new NotSupportedException();
    public void SaveToCloud() => throw new NotSupportedException();
    public void PlayEmote(string id) { /* maybe sway */ }
}
```

### Problem

- `Tree` bị ép API không thuộc về nó (LSP cũng đau)
- Client chỉ cần `Move` vẫn compile phụ thuộc cả shop/cloud
- Thêm method vào `IEntity` → mọi class vỡ

### Refactor

1. Liệt kê client: movement system, combat, shop UI, persistence.
2. Tạo interface theo client.
3. Entity implement tổ hợp phù hợp.

### Good code

```csharp
public interface IMovable { void Move(Vector3 delta); }
public interface ICombatant { void Attack(IDamageable target); }
public interface IDamageable { void TakeDamage(int amount); }
public interface IShopHost { void OpenShop(); }
public interface IPersistable { void SaveToCloud(); }

public sealed class PlayerCharacter : IMovable, ICombatant, IDamageable, IPersistable
{
    public void Move(Vector3 delta) { /* ... */ }
    public void Attack(IDamageable target) { /* ... */ }
    public void TakeDamage(int amount) { /* ... */ }
    public void SaveToCloud() { /* ... */ }
}

public sealed class Tree : IDamageable
{
    public void TakeDamage(int amount) { /* chop HP */ }
}

public sealed class MovementSystem
{
    public void Tick(IMovable body, Vector3 input) => body.Move(input);
}
```

### Unity example

```csharp
public interface IInteractable
{
    void Interact(GameObject actor);
}

public interface IFocusable
{
    string Prompt { get; }
}

// UI chỉ cần IFocusable; input chỉ cần IInteractable
public class Chest : MonoBehaviour, IInteractable, IFocusable
{
    public string Prompt => "Open [E]";
    public void Interact(GameObject actor) { /* open loot */ }
}

public class InteractionPromptUI : MonoBehaviour
{
    public void Show(IFocusable focus) => /* text = focus.Prompt */;
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| 50 interface 1 method quá vụn | Segregate cực đoan | Gom theo *cùng client / cùng thay đổi* |
| Vẫn fat vì “tiện” | Copy API class thành interface | Thiết kế từ phía consumer |
| Explicit implement rối | Interface trùng tên member | Đặt tên rõ role |
| Circular refs giữa interface | Tách sai bounded context | Xem L18 modular |

## 8. Gỡ lỗi

1. Đếm method trên interface > ~5–7 và nhiều unused ở implementor → nghi ISP.
2. Tìm `throw NotSupported` → fat interface (và LSP).
3. Fake/test double phải stub 20 method → tách interface.

## 9. Best practices

- Role interface đặt tên theo khả năng: `IReadable`, `IAttacker`.
- Class cụ thể implement nhiều interface — OK.
- ISP + DIP: inject đúng interface nhỏ cho từng service.
- Unity: tránh một `IGameServices` mega cho mọi hệ thống.

## 10. Bài tập

**Bài 1** — Tách `IMachine` (`Print`, `Scan`, `Fax`) cho `OldPrinter` chỉ in.

**Bài 2** — `IRepository<T>` quá đầy — tách `IReadRepository` / `IWriteRepository`.

**Bài 3** — Viết client `ReportBuilder` chỉ phụ thuộc read side.

**Bài 4** — Đánh giá interface 15 method trong project chung: đề xuất 3 interface nhỏ.

## 11. Gợi ý

- Bài 1: `IPrinter`, `IScanner`, `IFax`.
- Bài 2–3: CQRS nhẹ — không cần full CQRS framework.

## 12. Đáp án

```csharp
public interface IPrinter { void Print(Document doc); }
public interface IScanner { Document Scan(); }
public interface IFax { void Fax(Document doc, string number); }

public sealed class OldPrinter : IPrinter
{
    public void Print(Document doc) { /* ... */ }
}

public sealed class ModernAllInOne : IPrinter, IScanner, IFax
{
    public void Print(Document doc) { }
    public Document Scan() => new();
    public void Fax(Document doc, string number) { }
}
```

```csharp
public interface IReadRepository<T>
{
    T? Get(Guid id);
    IReadOnlyList<T> List();
}

public interface IWriteRepository<T>
{
    void Add(T entity);
    void Update(T entity);
}
```

## 13. Đáp án thay thế

Default interface methods có thể giảm breaking change nhưng **không thay** ISP — client vẫn phụ thuộc bề mặt lớn. Prefer tách thật.

## 14. Thử thách

Thiết kế API inventory: UI list, combat consume item, trade — 3 interface không overlap thừa.

## 15. Ứng dụng thực tế

- ASP.NET: tách auth reader vs admin writer
- SDK client: surface nhỏ theo feature package
- Microservices: contract theo consumer (consumer-driven)

## 16. Liên hệ Unity

- Input: `IMoveInput` vs `IUINavigateInput`
- Save system chỉ thấy `ISaveable`, không thấy combat API
- Capstone MMORPG: mỗi hệ thống phụ thuộc interface hẹp → giảm coupling

## 17. Kiểm tra kiến thức

1. ISP chống điều gì?  
   **Đáp án:** Client phụ thuộc method không dùng (fat interface).

2. Quan hệ ISP và LSP?  
   **Đáp án:** Fat interface hay dẫn tới NotSupported — phá LSP.

3. Nhiều interface trên một class có sai không?  
   **Đáp án:** Không — đó là cách làm phổ biến đúng ISP.

4. Dấu hiệu fat interface khi test?  
   **Đáp án:** Fake phải stub hàng loạt method thừa.

5. Vì sao mega `IGameServices` xấu trong Unity?  
   **Đáp án:** Mọi hệ thống phụ thuộc mọi thứ — thay đổi lan rộng.
