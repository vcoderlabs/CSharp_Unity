# Chương 2 — `struct` vs `class`

## 1. Mục tiêu học

- So sánh `struct` và `class` về semantics, bộ nhớ, khả năng kế thừa
- Chọn đúng khi nào dùng struct, khi nào dùng class
- Tránh bug sửa field qua property struct (Unity-style)
- Hiểu `readonly struct` và giới hạn mặc định của struct

## 2. Điều kiện tiên quyết

- Chương 1 — Stack vs Heap
- Level 2 — OOP: class, constructor, property, inheritance cơ bản

## 3. Khái niệm

### `class` (reference type)

- Object trên **heap**; biến giữ **reference**
- Hỗ trợ kế thừa class, interface; có thể `null`
- Phù hợp thực thể có **identity** (Player, Inventory, Transform)

### `struct` (value type)

- Copy **theo giá trị**; thường nhỏ, immutable hoặc gần immutable
- **Không** kế thừa class khác (chỉ implement interface)
- Không thể là `null` (trừ `Nullable<T>` / `T?`)
- Có constructor không tham số ngầm (field = default) — hành vi theo phiên bản C#/runtime cần lưu ý khi học sâu

### Bảng so sánh nhanh

| Tiêu chí | `struct` | `class` |
|----------|----------|---------|
| Semantics khi gán | Copy dữ liệu | Copy reference |
| `null` | Không (trừ `T?`) | Có |
| Kế thừa | Chỉ interface | Class + interface |
| Kích thước khuyến nghị | Nhỏ (thường ≤ ~16–24 bytes hướng dẫn thực dụng) | Tùy ý |
| Equality mặc định | So sánh field (value) | So sánh reference (`==` thường) |
| Dùng khi | Tọa độ, màu, slot item nhẹ, ID nhỏ | Hệ thống phức tạp, lifetime dài |

> **Cảnh báo Unity:** `Vector3` là struct. Lấy `transform.position` rồi sửa `.x` trên bản sao **không** cập nhật Transform.

## 4. Mô hình tư duy

### Gán class vs gán struct

```text
// CLASS
Player a = new Player { Name = "A" };
Player b = a;
b.Name = "B";
// a.Name == "B"

STACK          HEAP
a ──┐       ┌────────────┐
b ──┴──────►│ Name = "B" │
            └────────────┘


// STRUCT
ItemSlot s1 = new ItemSlot { ItemId = 1, Count = 5 };
ItemSlot s2 = s1;
s2.Count = 99;
// s1.Count vẫn 5

STACK
┌──────────────────┐
│ s1: Id=1 Count=5 │
│ s2: Id=1 Count=99│  ← bản sao độc lập
└──────────────────┘
```

### Property trả struct = bản sao

```text
class Body
{
    public Vector3 Pos { get; set; }  // get trả COPY
}

body.Pos.x = 3;
// tương đương:
//   Vector3 tmp = body.Pos;  // copy
//   tmp.x = 3;               // sửa copy
//   // không set lại Pos!
```

## 5. Cú pháp

```csharp
struct ItemSlot
{
    public int ItemId;
    public int Count;

    public ItemSlot(int itemId, int count)
    {
        ItemId = itemId;
        Count = count;
    }

    public bool IsEmpty => ItemId == 0 || Count <= 0;
}

class Inventory
{
    public ItemSlot[] Slots = new ItemSlot[20];
}

// readonly struct — khuyến khích không mutate field
readonly struct Damage
{
    public readonly int Amount;
    public Damage(int amount) => Amount = amount;
}
```

Implement interface:

```csharp
interface IDamageable { void TakeHit(int dmg); }

struct Crystal : IDamageable  // OK — nhưng boxing nếu gọi qua interface (xem Ch.3)
{
    public int Hp;
    public void TakeHit(int dmg) => Hp -= dmg;
}
```

## 6. Ví dụ

### Cơ bản — định nghĩa struct nhỏ

```csharp
struct ColorRgb
{
    public byte R, G, B;
}

var red = new ColorRgb { R = 255, G = 0, B = 0 };
var copy = red;
copy.G = 128;
Console.WriteLine($"{red.G} vs {copy.G}"); // 0 vs 128
```

### Trung cấp — class chứa mảng struct

```csharp
struct ItemSlot
{
    public int ItemId;
    public int Count;
}

class Inventory
{
    public ItemSlot[] Slots = new ItemSlot[3];

    public void AddToFirst(int itemId, int count)
    {
        // ĐÚNG: sửa phần tử mảng tại chỗ (mảng chứa value inline)
        Slots[0].ItemId = itemId;
        Slots[0].Count += count;
    }
}

var inv = new Inventory();
inv.AddToFirst(1001, 5);
Console.WriteLine(inv.Slots[0].Count); // 5
```

### Nâng cao — bug property struct vs sửa mảng

```csharp
struct Stats { public int Atk; public int Def; }

class Hero
{
    private Stats _stats;
    public Stats Stats
    {
        get => _stats;
        set => _stats = value;
    }
}

var hero = new Hero();
// hero.Stats.Atk = 10; // lỗi biên dịch với property (không gán được vào member của copy)
// Cách đúng:
var s = hero.Stats;
s.Atk = 10;
hero.Stats = s;
```

Với mảng/field public struct, mutation tại chỗ **có thể** được — đừng nhầm với property.

## 7. Lỗi thường gặp

| Lỗi | Nguyên nhân | Cách xử lý |
|-----|-------------|------------|
| Sửa `transform.position.x` không ăn | Property trả copy | Đọc–sửa–gán lại |
| Struct quá lớn, copy chậm | Copy cả khối bytes mỗi lần gán/truyền | Thu nhỏ / dùng class / `ref` |
| Mutable struct + property | Dễ sửa nhầm bản sao | Ưu tiên immutable / `readonly struct` |
| Kỳ vọng kế thừa từ struct | Struct không inherit class | Dùng class hoặc composition |
| `==` giữa hai class | So reference, không phải field | Override `Equals` / dùng record |

## 8. Gỡ lỗi

Checklist khi “set field không đổi”:

1. Kiểu đang cầm là `struct` hay `class`?
2. Lấy từ **property** hay **field/array element**?
3. Đã **gán lại** vào chỗ lưu trữ chưa?

```csharp
// Unity pattern
var pos = transform.position;
pos.y += 2f;
transform.position = pos;
```

## 9. Best practices

- Struct: **nhỏ**, thường **immutable**, biểu diễn “giá trị” (điểm, màu, slot)
- Class: hành vi phức tạp, identity, lifecycle, inheritance
- Tránh mutable struct public trừ khi bạn nắm chắc copy semantics (Unity legacy API vẫn mutable — cẩn thận)
- `readonly struct` + method không mutate giúp giảm bug copy
- Đừng chuyển mọi thứ sang struct “cho nhanh” — struct lớn copy **đắt hơn** truyền reference

## 10. Bài tập

**Bài 1.** Viết `struct DamageInfo { int Amount; bool IsCrit; }` và class `CombatLog` giữ `DamageInfo Last`. Chứng minh gán `Last` ra biến rồi sửa không đổi `CombatLog` cho đến khi gán lại.

**Bài 2.** Giải thích vì sao `Slots[i].Count++` trên `ItemSlot[]` có thể đổi inventory, trong khi `hero.Stats.Atk++` qua property thì không.

**Bài 3.** Chọn struct hay class cho: (a) tọa độ 2D, (b) người chơi online, (c) mã màu RGB, (d) hệ thống quest có 20 method.

**Bài 4.** Tái hiện bug Unity bằng class giả:

```csharp
class FakeTransform
{
    public Vector3Fake Position { get; set; }
}
struct Vector3Fake { public float X, Y, Z; }
```

Thử “`t.Position.X = 1`” và sửa đúng cách.

## 11. Gợi ý

- Bài 1: copy value từ property/field; chỉ `Last = modified` mới ghi.
- Bài 2: phần tử mảng là **storage thật**; property get trả **temporary**.
- Bài 3: (a)(c) struct; (b)(d) class.
- Bài 4: cần biến tạm + set lại property.

## 12. Đáp án + Giải thích

**Bài 1:**

```csharp
struct DamageInfo { public int Amount; public bool IsCrit; }

class CombatLog
{
    public DamageInfo Last;
}

var log = new CombatLog();
log.Last = new DamageInfo { Amount = 10, IsCrit = false };

var tmp = log.Last;
tmp.Amount = 999;
Console.WriteLine(log.Last.Amount); // 10

log.Last = tmp;
Console.WriteLine(log.Last.Amount); // 999
```

**Bài 2:** Mảng lưu struct **inline** — `Slots[i].Count++` sửa ô nhớ trong mảng. Property `get` tạo **bản sao**; tăng field trên bản sao rồi bỏ → storage gốc nguyên.

**Bài 3:** Tọa độ/màu = value nhỏ → struct. Player/quest system = identity + hành vi → class.

**Bài 4:**

```csharp
// Sai (thường lỗi biên dịch với property struct):
// t.Position.X = 1;

// Đúng:
var p = t.Position;
p.X = 1;
t.Position = p;
```

## 13. Đáp án thay thế

Bài 1 có thể dùng auto-property `DamageInfo Last { get; set; }` — kết luận copy vẫn giống.

Bài 3: nếu “màu” cần identity/palette object chia sẻ mutable → class; còn RGB thuần túy → struct.

## 14. Thử thách

Đo (tạm thời bằng `Stopwatch`) copy 1_000_000 lần một `struct` 4 field `int` vs một `class` cùng field (chỉ copy reference). Quan sát: copy struct lớn hơn nhiều field sẽ chậm dần — rút ra quy tắc kích thước.

## 15. Ứng dụng thực tế

- DTO nhỏ, key, tọa độ, money amount → struct
- Service, repository, entity game phức tạp → class
- API public: tránh mutable struct property nếu team dễ quên gán lại

## 16. Liên hệ Unity

- `Vector3`, `Vector2`, `Quaternion`, `Color`, `Ray`, `Bounds` → **struct**
- `Transform`, `GameObject`, `Component`, `MonoBehaviour` → **class**
- Performance: struct nhỏ giảm GC; nhưng **copy** mỗi frame trong vòng lặp nóng vẫn tốn CPU
- DOTS/`IComponentData` dựa trên mindset value type — Level này là nền

> **Nhắc lại:** Mutating struct copy (`transform.position.x = …`) **không** đổi bản gốc. Đây là bug Unity số 1 liên quan value type.

## 17. Kiểm tra kiến thức

1. `struct` hỗ trợ kế thừa class khác? (A) Có (B) Không  
2. Gán `struct` copy gì? (A) Reference (B) Giá trị các field  
3. `Vector3` trong Unity là? (A) class (B) struct  
4. `transform.position.x = 5` đổi vị trí? (A) Có (B) Không  
5. Struct rất lớn truyền by-value mỗi lần gọi method thường? (A) Rẻ (B) Có thể đắt vì copy  

**Đáp án:** 1B, 2B, 3B, 4B, 5B
