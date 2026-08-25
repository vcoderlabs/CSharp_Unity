# Milestone 02 — Entity System

## Requirements

- Có mô hình **Entity** (id + tập component/data) theo hướng component-based hoặc ECS-style nhẹ (không bắt buộc DOTS)
- Dùng **generics + interfaces** cho API kiểu `GetComponent` của bạn (không chỉ Unity `GetComponent`)
- Spawn entity từ prefab/definition SO
- Ít nhất 3 component: `Health`, `ViewLink` (gắn facade MB), `Team` / `Faction`
- Query được “mọi entity có Health + Team”
- Tương thích DI từ M01

**Không yêu cầu:** inventory, skill tree đầy đủ.

---

## Architecture

```text
IEntityRegistry
  ├── Create(EntityId, components...)
  ├── Destroy(EntityId)
  └── Query<T>() / Query<T1,T2>()

EntityId (ulong / Guid)

Components (data):
  Health { int Current, Max }
  Team { int Id }
  ViewLink { EntityView mb }

Systems:
  HealthSystem.ApplyDamage(EntityId, int)
  DeathSystem — Health <= 0 → event

Unity:
  EntityView : MonoBehaviour — giữ EntityId, sync nếu cần
  CharacterFacade prefab ──spawn──► Registry.Create + bind ViewLink
```

Hybrid chấp nhận được: GameObject vẫn tồn tại; **gameplay truth** nằm registry.

---

## Tasks

1. Định nghĩa `EntityId`, `IEntityRegistry`.  
2. Implement registry in-memory (store theo type).  
3. Thêm `Health` / `Team` / `ViewLink`.  
4. API `TryGet<T>`, `Add`/`Replace`, `Query`.  
5. Spawner: tạo GO + đăng ký entity.  
6. Smoke: 2 enemy + 1 player; damage qua system.  
7. Cập nhật `ARCHITECTURE.md`.

---

## Expected result

- Spawn entity không chỉ dựa `FindObjectsOfType`.  
- Damage đi qua `HealthSystem`.  
- Destroy entity gỡ data + optionally Destroy GO.  
- Query team filter hoạt động.

---

## Exercises

**E1** — Generic `ComponentStore<T>` nội bộ registry.  
**E2** — `IEntityFactory` đọc `EntityDefinition` SO.  
**E3** — Event `EntityDied`.  
**E4** — Note hiệu năng query 1k–10k entity (Editor).

---

## Hints

- `Dictionary<Type, object>` chứa `Dictionary<EntityId, T>`.  
- Struct component: cẩn thận copy — `Replace` cả giá trị.  
- Không cần DOTS đầy đủ.  
- EntityId tăng dần.

---

## Solution outline

```csharp
public readonly struct EntityId : IEquatable<EntityId>
{
    public readonly ulong Value;
    public EntityId(ulong v) => Value = v;
}

public interface IEntityRegistry
{
    EntityId Create();
    void Destroy(EntityId id);
    void Add<T>(EntityId id, T component);
    bool TryGet<T>(EntityId id, out T component);
    void Replace<T>(EntityId id, T component);
    IEnumerable<EntityId> Query<T>();
}
```

`HealthSystem`: lấy Health → trừ → Replace → nếu ≤0 raise die bus.  
Spawner inject registry + prefab → Create → Add components → gán `EntityView.Id`.

---

## Code review checklist

- [ ] HP gameplay không chỉ public field MB tùy tiện  
- [ ] Ghi chú “main thread only” nếu không thread-safe  
- [ ] Destroy không để dangling ViewLink  
- [ ] Generic API không lộ `object` lung tung  
- [ ] Smoke scene hoặc test verify spawn  
- [ ] ARCHITECTURE cập nhật  
- [ ] Không dùng `FindObjectsOfType` làm target list chính  
- [ ] Phân biệt Entity vs GameObject trong docs  
