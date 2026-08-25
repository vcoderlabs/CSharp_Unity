# Chương 16 — Mediator

## 1. Mục tiêu học

- Tập trung giao tiếp giữa nhiều object — giảm web reference lẫn nhau
- Phân biệt Mediator vs Observer vs Facade
- Unity: UI screens hub, gameplay systems bus có kiểm soát

## 2. Điều kiện tiên quyết

- Observer
- DIP — mediator phụ thuộc abstraction colleagues

## 3. Khái niệm

**Mediator:** colleagues không gọi nhau trực tiếp; gửi message qua mediator. Mediator quyết định chuyển tiếp tới ai.

## 4. Mô hình tư duy

```text
Trước: A↔B↔C↔D (mesh)
Sau:   A,B,C,D → Mediator (hub)
```

## 5. Cú pháp

```csharp
public interface IMediator
{
    void Notify(object sender, string ev);
}

public abstract class Colleague
{
    protected readonly IMediator Mediator;
    protected Colleague(IMediator m) => Mediator = m;
}
```

## 6. Ví dụ

### Cơ bản — dialog UI

```csharp
public sealed class LoginMediator : IMediator
{
    public ButtonOk Ok { get; set; }
    public TextBox User { get; set; }
    public void Notify(object sender, string ev)
    {
        if (ev == "textChanged")
            Ok.Enabled = !string.IsNullOrEmpty(User.Text);
    }
}
```

### Trung cấp

Gameplay: `CombatMediator` nhận `PlayerDied` → bảo `Ui`, `Loot`, `Match` — không để Player gọi 3 hệ thống.

### Nâng cao / Unity

```csharp
public interface IGameplayMediator
{
    void PlayerDied(PlayerId id);
    void WaveCleared(int wave);
}

// Tránh GlobalEventBus subscribe 50 nơi không kỷ luật
```

Chatty mediator → God Mediator (mùi). Tách theo bounded context.

## 7. Lỗi thường gặp

| Hiện tượng | Cách xử lý |
|------------|------------|
| Mediator biết mọi concrete | Interface colleagues / messages |
| God Mediator | Nhiều mediator theo feature |
| Nhầm Facade | Facade API vào subsystem; Mediator điều phối peers |

## 8. Gỡ lỗi

Log mọi Notify. Sequence diagram khi thêm colleague mới.

## 9. Best practices

- Message/event typed hơn string.  
- Mediator không chứa rule domain nặng — chuyển tới use case.  
- Có thể implement bằng Observer bên trong.

## 10. Bài tập

**Bài 1** — Form: Email + Submit enable.  
**Bài 2** — Tower defense: Gate HP 0 → UI + Spawner stop.  
**Bài 3** — Refactor mesh 4 class sang mediator.  
**Bài 4** — So Observer.

## 11. Gợi ý

Observer: subject→many; Mediator: many↔many có điều phối tập trung.

## 12. Đáp án

```csharp
public sealed class TdMediator : IMediator
{
    private readonly Hud _hud;
    private readonly Spawner _spawner;
    public void Notify(object sender, string ev)
    {
        if (ev == "gateDestroyed")
        {
            _spawner.Stop();
            _hud.ShowDefeat();
        }
    }
}
```

## 13. Đáp án thay thế

Message pipe / Event aggregator có channel — Mediator phân tán có chủ đích.

## 14. Thử thách

Shop UI: currency, inventory, confirm modal — không reference chéo.

## 15. Ứng dụng thực tế

- Chatroom classic  
- Air traffic control metaphor  
- CQRS process managers (họ hàng)

## 16. Liên hệ Unity

- UI Toolkit / uGUI screen flow  
- Giảm `FindObjectOfType` chéo hệ thống  
- Đừng thay toàn bộ architecture bằng một GameMediator

## 17. Kiểm tra kiến thức

1. Mediator giảm gì? **Coupling trực tiếp giữa colleagues.**  
2. Rủi ro? **God mediator.**  
3. Khác Facade? **Peers vs mặt tiền subsystem.**  
4. Khác Observer? **Điều phối nhiều chiều có hub.**  
5. Nên chứa domain lớn? **Không — chuyển use case.**
