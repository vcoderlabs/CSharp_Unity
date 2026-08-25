# Milestone 03 — Inventory

## Requirements

- Hệ **Inventory** generic-friendly (`ItemId`, stack, slot)
- Định nghĩa item bằng **ScriptableObject** (hoặc Addressable SO)
- Thao tác: Add, Remove, Has, Move/Swap slot
- Tuân **SOLID** vừa đủ: `IInventory`, tách UI
- **Events** khi inventory đổi (C# event hoặc channel SO)
- UI tối thiểu: mở túi, hiện icon/số lượng
- Tích hợp nhặt item trong world (trigger / interact)

**Không yêu cầu:** crafting phức tạp, trade player-player.

---

## Architecture

```text
ItemDefinition : ScriptableObject
  Id, DisplayName, Icon, MaxStack, ItemType

IInventory
  bool TryAdd(ItemId, int qty)
  bool TryRemove(ItemId, int qty)
  IReadOnlyList<Slot> Slots

InventoryService : IInventory
  raises event InventoryChanged

InventoryPresenter ──► InventoryView (MB/UI)
WorldItemPickup (MB) ──► TryAdd + despawn

DIP: UI phụ thuộc IInventory, không class cụ thể cứng nếu có thể
```

```text
Player Entity
  └── InventoryOwner component { InventoryId } 
        hoặc service map EntityId → IInventory
```

---

## Tasks

1. `ItemDefinition` SO + 3 item mẫu (potion, material, quest item).  
2. `Slot` model: itemId nullable + quantity.  
3. `InventoryService` với capacity cố định (ví dụ 20).  
4. Event đổi túi; HUD/bag listen.  
5. Pickup trong scene.  
6. Unit test: add stack, overflow, remove.  
7. Ghi chú SRP/OCP trong ARCHITECTURE (5–10 dòng).

---

## Expected result

- Nhặt item → túi cập nhật UI không `Find` mỗi frame.  
- Stack đúng `MaxStack`.  
- Túi đầy → Add fail (feedback log/UI).  
- Tests xanh cho logic thuần.

---

## Exercises

**E1** — `TryAdd` trả enum `InventoryResult`.  
**E2** — Lọc hiện theo `ItemType`.  
**E3** — Giới hạn quest item không bán được (flag trên definition).  
**E4** — Drag-swap 2 slot (UI).

---

## Hints

- Stack: tìm slot cùng id chưa full trước khi tìm slot trống.  
- Không lưu SO reference trong save (M07) — lưu Id string.  
- UI Toolkit hoặc uGUI đều được.  
- Presenter dễ test hơn logic trong Button OnClick.

---

## Solution outline

```csharp
public readonly struct ItemId : IEquatable<ItemId>
{
    public string Value { get; }
    public ItemId(string v) => Value = v;
}

public sealed class InventoryService : IInventory
{
    readonly Slot[] _slots;
    readonly IItemCatalog _catalog;
    public event Action Changed;

    public bool TryAdd(ItemId id, int qty)
    {
        // 1) stack vào slot hiện có
        // 2) slot trống
        // 3) fail nếu còn dư
        Changed?.Invoke();
        return true;
    }
}
```

`IItemCatalog.Get(ItemId) → ItemDefinition` (load từ SO database).

Pickup:

```text
OnInteract → inventory.TryAdd → if ok Destroy/pool world item → raise feedback
```

---

## Code review checklist

- [ ] Interface `IInventory` tồn tại và UI không phá luật DIP quá nặng  
- [ ] Stacking đúng MaxStack  
- [ ] Event không leak (OnDisable unsubscribe)  
- [ ] Unit tests logic Add/Remove  
- [ ] Item id ổn định (string/guid) — không dựa tên file lung tung  
- [ ] Không sửa shared SO runtime cho số lượng  
- [ ] ARCHITECTURE có mục Inventory  
- [ ] Pickup có feedback khi full  
