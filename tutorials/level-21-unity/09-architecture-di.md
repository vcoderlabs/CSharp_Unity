# Chương 9 — Architecture & DI (MVC/MVP/ECS, VContainer/Zenject)

## 1. Mục tiêu học

- Phác **MVC / MVP / ECS** trong ngữ cảnh Unity
- Hiểu vì sao MonoBehaviour “mập” chết project
- Dùng **DI** (VContainer hoặc Zenject) đăng ký service
- Giữ MB mỏng: glue scene ↔ logic thuần C#

## 2. Điều kiện tiên quyết

- Level 16–18 khái niệm SOLID/Architecture (đọc lại nếu cần)
- Events, pooling, lifecycle
- Package Manager

## 3. Khái niệm

### MVC / MVP (tóm tắt)

| | Vai trò |
|--|---------|
| Model | Data + rules |
| View | UI / mesh / animation trình bày |
| Controller / Presenter | Điều phối input → model → cập nhật view |

Unity dễ trộn cả ba trong một MB — **tách dần**.

### ECS (overview)

**Entity** = id; **Component** = data thuần; **System** = logic duyệt data.  
Unity DOTS ECS khác component MonoBehaviour cổ điển. Ở level này chỉ cần hiểu **data-oriented** vs OOP object lớn.

### DI

Thay `FindObjectOfType` / singleton cứng bằng **constructor/method injection** từ container.

**VContainer** và **Zenject/Extenject** phổ biến. Chọn **một**.

## 4. Mô hình tư duy

```text
LifetimeLifetimeLifetimeLifetime LifetimeScope (VContainer)
  đăng ký: IInventory, IEventBus, BulletPool, ...
        │
        ▼
PlayerFacade (MB mỏng) ──inject──► PlayerAttackService (pure C#)
        │                              │
        └──── View / Animator ──────────┘

ECS mentally:
  Entities → Components (data) → Systems (logic) — parallel friendly
```

## 5. Cú pháp (VContainer)

```csharp
// Package: jp.hadashikick.vcontainer
using VContainer;
using VContainer.Unity;

public class GameLifetimeScope : LifetimeScope
{
    protected override void Configure(IContainerBuilder builder)
    {
        builder.Register<IInventory, InventoryService>(Lifetime.Singleton);
        builder.RegisterEntryPoint<GameRunner>();
        builder.RegisterComponentInHierarchy<PlayerView>();
    }
}

public class PlayerAttackService
{
    readonly BulletPool _pool;
    public PlayerAttackService(BulletPool pool) => _pool = pool;
    public void Fire(Vector3 pos, Vector3 dir) { /* ... */ }
}
```

Zenject tương đương: `GameInstaller : MonoInstaller` + `[Inject]`.

## 6. Ví dụ

### Cơ bản — bỏ singleton cứng

```csharp
// Bad
public class GameServices { public static Inventory I; }

// Better: inject IInventory
```

### Trung cấp — MVP UI

```csharp
public class InventoryPresenter
{
    readonly IInventory _model;
    readonly InventoryView _view;
    public InventoryPresenter(IInventory model, InventoryView view) { ... }
    public void OnOpen() => _view.Render(_model.Items);
}
```

### Nâng cao — ECS overview chỉ đọc

Combat hit: thay vì `enemy.TakeDamage`, hệ thống `DamageSystem` đọc `Health`+`DamageEvent` buffers — Capstone có thể hybrid OOP.

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Resolve null | Quên Register / scope sai | Kiểm tra LifetimeScope parent |
| MB vẫn Find | Nửa DI nửa cũ | Quyết liệt inject |
| God container | Đăng ký 200 type lung tung | Module installers |
| Test khó | Logic trong MB | Tách pure C# |

## 8. Gỡ lỗi

1. VContainer diagnostics / Zenject validate.  
2. Log khi EntryPoint Start.  
3. Scene thiếu LifetimeScope.  
4. DontDestroyOnLoad scope vs scene scope — hiểu lifetime.

## 9. Best practices

- Pure C# services + thin MB/views.  
- Interface tại biên.  
- Một composition root rõ.  
- Không DI mọi class 10 dòng.  
- Document: team chọn VContainer **hoặc** Zenject.

## 10. Bài tập

**Bài 1** — Cài VContainer hoặc Zenject; LifetimeScope trống chạy được.  
**Bài 2** — Register `IScoreService`; MB HUD inject và hiện điểm.  
**Bài 3** — Tách `AttackService` khỏi `PlayerInput` MB.  
**Bài 4** — Viết bảng so MVC vs MVP 5 dòng cho UI inventory.

## 11. Gợi ý

- Bài 2: Singleton lifetime.  
- Bài 4: Presenter dễ unit test hơn Controller dày UI.

## 12. Đáp án

**Bài 4:** MVP: View dumb, Presenter cập nhật View từ Model — test Presenter không cần Scene. MVC: Controller có thể dày hơn tùy biến thể Unity.

**Bài 2–3:** theo cú pháp Register + constructor inject.

## 13. Đáp án thay thế

StrangeIoC, Reflex, tự viết ServiceLocator nhẹ (không khuyến khích lâu dài). Pure MVC không package.

## 14. Thử thách

EntryPoint load Addressables rồi đăng ký runtime factory đạn vào container.

## 15. Ứng dụng thực tế

Project > 1 tháng: DI + layer = onboarding và test khả thi. MMORPG bắt buộc kỷ luật này.

## 16. Liên hệ C# thuần

Giống ASP.NET DI `IServiceCollection`. LifetimeScope ≈ scope request/scene.

## 17. Kiểm tra kiến thức

1. MB mập gây gì?  
   **Đáp án:** Khó test, khó tái sử dụng, circular ref scene, conflict merge.

2. DI giúp gì?  
   **Đáp án:** Cấp dependency tường minh, giảm singleton/Find, dễ fake khi test.

3. ECS khác OOP component Unity cổ điển?  
   **Đáp án:** Nhấn data layout + systems; entity chỉ id — không phải class “thông minh”.

4. Composition root là gì?  
   **Đáp án:** Nơi duy nhất (hoặc ít) wire đăng ký DI.

5. View trong MVP nên chứa business rule không?  
   **Đáp án:** Không — chỉ hiển thị và báo input.
