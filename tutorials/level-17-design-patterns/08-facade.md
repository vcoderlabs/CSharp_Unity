# Chương 8 — Facade

## 1. Mục tiêu học

- Cung cấp **API đơn giản** trước subsystem phức tạp
- Giảm coupling client ↔ nhiều class nội bộ
- Phân biệt Facade vs Adapter vs Mediator
- Unity: `GameplayFacade` bootstrap combat turn

## 2. Điều kiện tiên quyết

- SRP (facade mỏng, subsystem vẫn tách)
- DIP cơ bản

## 3. Khái niệm

**Facade** là “mặt tiền”: một (vài) method điều phối nhiều service bên trong. Không thay thế subsystem — chỉ **làm dễ dùng**.

## 4. Mô hình tư duy

```text
UI/Controller → GameFacade.StartMatch()
                    ├─ MapLoader
                    ├─ Spawner
                    └─ HudBinder
```

## 5. Cú pháp

```csharp
public sealed class OrderFacade
{
    private readonly Inventory _inv;
    private readonly Pricing _price;
    private readonly Payment _pay;
    public OrderFacade(Inventory inv, Pricing price, Payment pay)
    { _inv = inv; _price = price; _pay = pay; }

    public void Checkout(Cart cart)
    {
        _inv.Reserve(cart);
        var total = _price.Calc(cart);
        _pay.Charge(total);
    }
}
```

## 6. Ví dụ

### Cơ bản

Client gọi `Checkout` — không biết 3 class dưới.

### Trung cấp

Facade trả DTO kết quả; log lỗi tập trung.

### Nâng cao / Unity

```csharp
public class CombatFacade : MonoBehaviour
{
    [SerializeField] private Spawner _spawner;
    [SerializeField] private TurnSystem _turns;
    [SerializeField] private CombatHud _hud;

    public void BeginEncounter(EncounterId id)
    {
        _spawner.Spawn(id);
        _turns.Reset();
        _hud.Show();
    }
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Cách xử lý |
|------------|------------|
| Facade thành God | Chỉ orchestration; logic ở subsystem |
| Expose hết mọi thứ | API hẹp theo use case |
| Nhầm Adapter | Adapter đổi interface 1 bên; Facade gom nhiều bên |

## 8. Gỡ lỗi

Integration test facade; unit test vẫn ở subsystem. Nếu facade > ~50–80 dòng rule → đẩy xuống.

## 9. Best practices

- Method tên theo use case (`CompleteQuest`).  
- Không bypass: team nên đi qua facade ở biên module (khi cần).  
- Có thể có nhiều facade theo bounded context.

## 10. Bài tập

**Bài 1** — Facade cho save: serialize + write + backup.  
**Bài 2** — `ShopFacade.Buy`.  
**Bài 3** — Vẽ diagram trước/sau coupling.  
**Bài 4** — Khác Mediator?

## 11. Gợi ý

Mediator: peer giao tiếp qua hub; Facade: client một chiều vào subsystem.

## 12. Đáp án

```csharp
public sealed class SaveFacade
{
    private readonly ISerializer _ser;
    private readonly IFileStore _files;
    public SaveFacade(ISerializer ser, IFileStore files) { _ser = ser; _files = files; }
    public void Save(GameState s)
    {
        var bytes = _ser.ToBytes(s);
        _files.Write("save.dat", bytes);
        _files.Write("save.bak", bytes);
    }
}
```

## 13. Đáp án thay thế

Application Service / Use Case trong Clean Architecture ≈ Facade có kỷ luật layer.

## 14. Thử thách

Facade `Matchmaking` ghép network + roster + scene load (stub).

## 15. Ứng dụng thực tế

- SDK client libraries  
- Home theater remote metaphor cổ điển

## 16. Liên hệ Unity

- Scene flow facade  
- Giảm `FindObjectOfType` rải rác bằng một cửa

## 17. Kiểm tra kiến thức

1. Facade đơn giản hóa gì? **Subsystem phức tạp.**  
2. Có xóa subsystem không? **Không.**  
3. God facade xấu vì? **Nhồi logic — mất SRP.**  
4. Khác Adapter? **Gom nhiều vs dịch một.**  
5. Khác Mediator? **Một chiều facade vs điều phối peers.**
