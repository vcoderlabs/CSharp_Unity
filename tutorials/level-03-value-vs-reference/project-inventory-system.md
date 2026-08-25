# Project — Inventory System

## 1. Mục tiêu học

- Áp dụng `struct` (nhẹ) + `class` (hệ thống) đúng chỗ
- Minh họa **side-effect** khi truyền reference / quên gán lại struct
- Tái hiện bug kiểu Unity (`position.x`) bằng inventory API
- Hoàn thành mini hệ thống túi đồ console chạy được bằng `dotnet run`

## 2. Điều kiện tiên quyết

- Xong chương 1–5 Level 3
- Biết tạo console app (`dotnet new console`)
- OOP cơ bản: class, property, method

## 3. Khái niệm

**Inventory System** trong project này:

| Thành phần | Kiểu | Lý do |
|------------|------|-------|
| `ItemSlot` | `struct` | Nhẹ: `ItemId` + `Count` — value semantics, ít GC |
| `Inventory` | `class` | Nhiều slot, gold, hành vi — identity + mutate tại chỗ |
| `ItemDatabase` (optional) | `class` | Tra cứu tên item |

Bạn sẽ **cố tình viết sai** vài method rồi sửa — để cảm nhận side-effect / copy.

> **Cảnh báo:** Mutating bản sao `ItemSlot` (từ property hoặc tham số by-value) **không** đổi inventory — cùng họ bug với `transform.position.x = …`.

## 4. Mô hình tư duy

```text
STACK                              HEAP
┌─────────────────────┐           ┌──────────────────────────────┐
│ inv ────────────────┼──────────►│ Inventory                    │
└─────────────────────┘           │  Gold = 100                  │
                                  │  Slots: ItemSlot[N]  (inline │
                                  │    values trong mảng trên     │
                                  │    heap object)               │
                                  └──────────────────────────────┘

Khi gọi: void BadClear(ItemSlot s) { s.Count = 0; }
→ chỉ xóa BẢN SAO trên stack frame BadClear
→ mảng Slots không đổi

Khi gọi: void GoodClear(Inventory inv, int i)
{
    var s = inv.Slots[i];
    s.Count = 0;
    inv.Slots[i] = s;   // gán VALUE trở lại ô mảng
}
```

Side-effect với class:

```text
void StealGold(Inventory inv) { inv.Gold = 0; }
→ copy reference, cùng object → Gold caller mất thật!
```

## 5. Cú pháp (khung project)

```bash
dotnet new console -n InventorySystem
cd InventorySystem
```

Gợi ý cấu trúc file (có thể gom 1 file `Program.cs` khi học):

```text
InventorySystem/
  Program.cs
  ItemSlot.cs
  Inventory.cs
```

## 6. Ví dụ / Đặc tả triển khai

### Cơ bản — `ItemSlot` (struct)

```csharp
public readonly struct ItemSlot
{
    public int ItemId { get; }
    public int Count { get; }

    public ItemSlot(int itemId, int count)
    {
        ItemId = itemId;
        Count = count;
    }

    public bool IsEmpty => ItemId == 0 || Count <= 0;

    public ItemSlot WithCount(int count) => new ItemSlot(ItemId, count);

    public override string ToString() =>
        IsEmpty ? "[empty]" : $"#{ItemId} x{Count}";
}
```

> Dùng `readonly struct` + `WithCount` để giảm bug mutate copy. Nếu bạn cố tình dùng mutable field `public int Count;` để demo lỗi — OK cho phần side-effect, rồi refactor lại.

### Trung cấp — `Inventory` (class)

```csharp
public class Inventory
{
    public int Gold { get; private set; }
    private readonly ItemSlot[] _slots;

    public int Capacity => _slots.Length;

    public Inventory(int capacity, int gold = 0)
    {
        _slots = new ItemSlot[capacity];
        Gold = gold;
    }

    public ItemSlot GetSlot(int index) => _slots[index]; // trả COPY

    public void SetSlot(int index, ItemSlot slot) => _slots[index] = slot;

    public bool TryAdd(int itemId, int count, int maxStack = 99)
    {
        // 1) cộng dồn stack cùng ItemId
        for (int i = 0; i < _slots.Length; i++)
        {
            if (_slots[i].ItemId == itemId && _slots[i].Count < maxStack)
            {
                int can = maxStack - _slots[i].Count;
                int add = Math.Min(can, count);
                _slots[i] = _slots[i].WithCount(_slots[i].Count + add);
                count -= add;
                if (count == 0) return true;
            }
        }
        // 2) tìm ô trống
        for (int i = 0; i < _slots.Length && count > 0; i++)
        {
            if (_slots[i].IsEmpty)
            {
                int add = Math.Min(maxStack, count);
                _slots[i] = new ItemSlot(itemId, add);
                count -= add;
            }
        }
        return count == 0;
    }

    public void AddGold(int amount) => Gold += amount;

    public void Print()
    {
        Console.WriteLine($"Gold: {Gold}");
        for (int i = 0; i < _slots.Length; i++)
            Console.WriteLine($"  [{i}] {_slots[i]}");
    }
}
```

### Nâng cao — Demo side-effect (BẮT BUỘC làm)

Thêm các method demo trong `Program.cs`:

```csharp
static void Demo_StructCopyDoesNothing(Inventory inv)
{
    Console.WriteLine("=== Demo: sửa copy struct ===");
    inv.SetSlot(0, new ItemSlot(1001, 5));

    ItemSlot copy = inv.GetSlot(0);
    // Giả sử ItemSlot mutable — nếu readonly, minh họa bằng biến tạm + KHÔNG SetSlot:
    copy = copy.WithCount(0); // chỉ đổi local
    // quên inv.SetSlot(0, copy);
    Console.WriteLine("Sau khi 'xóa' copy, slot thật: " + inv.GetSlot(0));
    // Vẫn x5 — GIỐNG transform.position.x = ...
}

static void Demo_ClassSideEffect(Inventory inv)
{
    Console.WriteLine("=== Demo: side-effect qua reference ===");
    StealAllGold(inv);
    Console.WriteLine("Gold sau StealAllGold: " + inv.Gold);
}

static void StealAllGold(Inventory inv) => inv.AddGold(-inv.Gold); // hoặc property setter

static void Demo_ReplaceReferenceNeedsRef()
{
    Console.WriteLine("=== Demo: thay reference cần ref ===");
    Inventory bag = new Inventory(5, gold: 50);
    ReplaceWrong(bag);
    Console.WriteLine("Sau ReplaceWrong, gold: " + bag.Gold); // vẫn 50

    ReplaceRight(ref bag);
    Console.WriteLine("Sau ReplaceRight, gold: " + bag.Gold); // 0 (inventory mới)
}

static void ReplaceWrong(Inventory inv) => inv = new Inventory(5, gold: 0);
static void ReplaceRight(ref Inventory inv) => inv = new Inventory(5, gold: 0);

static void Demo_UnityStyleBug()
{
    Console.WriteLine("=== Demo: Unity-style property struct ===");
    var body = new FakeTransform { Position = new Vec3(1, 2, 3) };

    // Không biên dịch được nếu Pos là property struct mutable field assign —
    // minh họa bằng pattern tương đương:
    var tmp = body.Position;
    tmp.X = 99; // sửa copy
    Console.WriteLine("Không gán lại → Position = " + body.Position); // X vẫn 1

    tmp = body.Position;
    tmp.X = 99;
    body.Position = tmp;
    Console.WriteLine("Đã gán lại → Position = " + body.Position); // X = 99
}

struct Vec3
{
    public float X, Y, Z;
    public Vec3(float x, float y, float z) { X = x; Y = y; Z = z; }
    public override string ToString() => $"({X},{Y},{Z})";
}

class FakeTransform
{
    public Vec3 Position { get; set; }
}
```

`Main` gợi ý:

```csharp
var inv = new Inventory(capacity: 8, gold: 100);
inv.TryAdd(1001, 40);
inv.TryAdd(1001, 70); // test stack / tràn stack
inv.TryAdd(2002, 1);
inv.Print();

Demo_StructCopyDoesNothing(inv);
Demo_ClassSideEffect(inv);
Demo_ReplaceReferenceNeedsRef();
Demo_UnityStyleBug();
```

## 7. Lỗi thường gặp

| Lỗi | Nguyên nhân | Cách xử lý |
|-----|-------------|------------|
| `TryAdd` “thêm” nhưng slot không đổi | Sửa copy từ `GetSlot` quên `SetSlot` | Luôn ghi lại `ItemSlot` vào mảng |
| Vàng bị mất sau gọi helper | Class pass reference + mutate | Cố ý / hoặc clone nếu cần độc lập |
| `ReplaceWrong` tưởng tạo túi mới | Thiếu `ref` | Dùng `ref` hoặc return inventory mới |
| Stack tràn max | Quên chia nhiều ô | Loop ô trống như mẫu |
| Nhầm `readonly struct` không sửa được | Đúng thiết kế | Dùng `WithCount` / tạo slot mới |

## 8. Gỡ lỗi

1. In `GetSlot(i)` trước/sau mỗi thao tác.
2. Đặt breakpoint trong `TryAdd` — xem `_slots[i]` sau gán.
3. Với demo Unity: in `Position` trước khi gán lại và sau khi gán lại.
4. Checklist: **Tôi đang giữ copy hay ô nhớ thật?**

## 9. Best practices

- `ItemSlot` nhỏ + immutable/`readonly` → ít surprise
- `Inventory` class → mutate có kiểm soát qua method (`TryAdd`, `SetSlot`)
- Không expose `ItemSlot[]` public nếu sợ caller sửa lung tung (hoặc expose read-only)
- Tách **demo bug** khỏi API production — comment rõ “anti-pattern”
- Mirror Unity: mọi lần “sửa struct từ property” = đọc–sửa–gán

## 10. Bài tập

**Bài 1.** Implement đủ `ItemSlot` + `Inventory` + `TryAdd` + `Print`. Chạy thêm item đến khi full — trả về `false`.

**Bài 2.** Viết `TryRemove(int itemId, int count)` — trừ dần các stack; return false nếu không đủ.

**Bài 3.** Viết đủ 4 demo side-effect ở mục 6; mỗi demo in giải thích 1 dòng tiếng Việt.

**Bài 4.** Thêm `Transfer(Inventory from, Inventory to, int itemId, int count)` — giải thích vì sao không cần `ref Inventory` để đổi nội dung, nhưng cần nếu muốn thay instance.

## 11. Gợi ý

- Bài 1: 2 vòng — merge stack rồi ô trống.
- Bài 2: duyệt slot, giảm `Count`, set empty khi 0.
- Bài 3: copy nguyên mẫu mục 6, chạy quan sát.
- Bài 4: mutate object qua reference; `ref` chỉ khi `to = new …`.

## 12. Đáp án + Giải thích

**Bài 1–2 (phác thảo `TryRemove`):**

```csharp
public bool TryRemove(int itemId, int count)
{
    int have = 0;
    for (int i = 0; i < _slots.Length; i++)
        if (_slots[i].ItemId == itemId) have += _slots[i].Count;
    if (have < count) return false;

    for (int i = 0; i < _slots.Length && count > 0; i++)
    {
        if (_slots[i].ItemId != itemId) continue;
        int take = Math.Min(_slots[i].Count, count);
        int left = _slots[i].Count - take;
        _slots[i] = left == 0 ? default : _slots[i].WithCount(left);
        count -= take;
    }
    return true;
}
```

**Bài 3:** Output kỳ vọng:

- Struct copy: slot vẫn còn item  
- Steal gold: gold về 0  
- ReplaceWrong: gold cũ; ReplaceRight: gold inventory mới  
- Unity-style: chỉ đổi sau khi gán lại `Position`

**Bài 4:**

```csharp
static bool Transfer(Inventory from, Inventory to, int itemId, int count)
{
    if (!from.TryRemove(itemId, count)) return false;
    if (to.TryAdd(itemId, count)) return true;
    // rollback đơn giản:
    from.TryAdd(itemId, count);
    return false;
}
```

Không cần `ref` vì sửa **nội dung** object. Cần `ref` chỉ khi gán lại biến `Inventory` của caller.

## 13. Đáp án thay thế

- `ItemSlot` mutable (`public int Count;`) + sửa `_slots[i].Count += …` trực tiếp trên mảng — hợp lệ với array element, vẫn phải tránh sửa qua property copy.
- Dùng `List<ItemSlot>` thay mảng — nhớ `list[i] = list[i].WithCount(...)` vì indexer get cũng copy.

## 14. Thử thách

Mở rộng:

1. `maxStack` theo từng `itemId` (dictionary cấu hình)
2. Serialization giả: in JSON-like thủ công các slot
3. Đo số lần copy struct trong `TryAdd` (biến đếm static) — tối ưu giảm copy không cần thiết

## 15. Ứng dụng thực tế

- RPG / survival inventory, hotbar, chest
- Server authoritative inventory: class trên heap, slot value trong mảng
- Tránh GC: không `new class` cho mỗi ô item mỗi frame — `struct` slot + mảng tái sử dụng

## 16. Liên hệ Unity

| Unity | Project này |
|-------|-------------|
| `Vector3` struct | `ItemSlot` / `Vec3` |
| `Transform` class | `Inventory` / `FakeTransform` |
| `transform.position.x = …` | Sửa copy từ `GetSlot` / `Position` quên gán lại |
| GC từ `new` item object mỗi pickup | Dùng struct slot + pool / mảng cố định |

**Vì sao struct quan trọng performance:** hàng nghìn `ItemSlot` trong mảng = dữ liệu liền mạch, không phải hàng nghìn object header trên heap. Đổi sang `class ItemSlot` cho mỗi ô → nhiều allocation + pressure GC hơn rõ rệt khi scale.

> **Nhắc lần cuối Level 3:** Mutating a struct copy không đổi bản gốc. Trong Unity và trong Inventory, luôn **gán lại** value sau khi sửa — hoặc sửa đúng storage (phần tử mảng / field) một cách có chủ đích.

## 17. Kiểm tra kiến thức

1. `ItemSlot` nên là? (A) class nặng (B) struct nhẹ (trong thiết kế project này)  
2. `Inventory` nên là? (A) struct (B) class  
3. `GetSlot` rồi sửa count local — inventory đổi? (A) Không nếu quên SetSlot (B) Luôn đổi  
4. `StealAllGold(inv)` không `ref` — có thể về 0? (A) Có (B) Không  
5. Bug Unity `position.x` cùng họ với? (A) Sửa copy struct (B) Boxing  

**Đáp án:** 1B, 2B, 3A, 4A, 5A

---

## Checklist hoàn thành project

- [ ] Tạo được project console và chạy `dotnet run`
- [ ] `TryAdd` / `Print` / (khuyến nghị) `TryRemove` hoạt động
- [ ] Chạy 4 demo side-effect và giải thích được từng cái
- [ ] Viết được `Transfer` hoặc tương đương
- [ ] Nói lại được bằng lời: **struct copy ≠ đổi gốc**, **class reference = side-effect**

Xong project → Level 3 hoàn tất → sang [Level 4 — Collections](../level-04-collections/).
