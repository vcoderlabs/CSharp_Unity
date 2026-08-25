# Chương 4 — Copying Semantics (Value vs Reference)

## 1. Mục tiêu học

- Dự đoán chính xác kết quả sau phép gán `a = b`
- Phân biệt **shallow copy** của reference vs **value copy** của struct
- Hiểu copy khi truyền tham số (by value mặc định)
- Nhận diện và sửa bug “sửa copy struct / property”

## 2. Điều kiện tiên quyết

- Chương 1–3 (Stack/Heap, struct/class, boxing cơ bản)
- Method parameters Level 1

## 3. Khái niệm

### Value semantics (value type)

```csharp
A = B; // copy toàn bộ dữ liệu của B vào A (độc lập)
```

Đổi `A` sau đó **không** đổi `B`.

### Reference semantics (reference type)

```csharp
A = B; // copy địa chỉ; A và B cùng object
```

Đổi **nội dung object** qua `A` → thấy qua `B`.  
Nhưng `A = somethingElse` chỉ đổi biến `A`, không đổi `B`.

### Truyền tham số mặc định = truyền **bản sao của biến**

- Với `int`/`struct`: method nhận **bản sao giá trị**
- Với `class`: method nhận **bản sao reference** (vẫn trỏ cùng object!) → có thể side-effect lên object

Đây là chỗ gây nhầm: “C# luôn pass-by-value” **đúng** — nhưng với class, value được copy là **reference**.

## 4. Mô hình tư duy

### 4.1. Copy value type

```text
ItemSlot a = { Id=1, Count=5 };
ItemSlot b = a;     // copy bytes
b.Count = 0;

STACK
a: Id=1 Count=5
b: Id=1 Count=0     ← độc lập
```

### 4.2. Copy reference type

```text
Inventory a = new Inventory(...);
Inventory b = a;    // copy pointer
b.Gold = 0;         // a.Gold cũng 0

STACK              HEAP
a ──┐           ┌─────────────┐
b ──┴──────────►│ Gold = 0    │
                └─────────────┘
```

### 4.3. Pass-by-value với class (side-effect)

```text
void Hurt(Player p) { p.Hp -= 10; }

STACK (caller)         HEAP
player ───────────────► Hp=90   (sau Hurt)

STACK (Hurt frame)
p ─────────────────────► (cùng object)
```

`Hurt` không cần `ref` để sửa **field object**.  
`ref` chỉ cần khi muốn đổi **biến reference** của caller (trỏ object khác / null).

### 4.4. Unity: copy từ property

```text
transform.position   →  trả về Vector3 COPY
gán .x trên copy     →  bản gốc Transform không đổi

ĐÚNG:
  tmp = transform.position
  tmp.x = ...
  transform.position = tmp   // copy ngược lại
```

## 5. Cú pháp

```csharp
// Value copy
var s1 = new ItemSlot(1, 5);
var s2 = s1;

// Reference copy
var inv1 = new Inventory();
var inv2 = inv1;
inv2 = new Inventory(); // chỉ inv2 trỏ object mới; inv1 giữ object cũ

// Method
void ResetSlot(ItemSlot slot) { slot.Count = 0; }          // không ảnh hưởng caller
void ResetInv(Inventory inv) { inv.Clear(); }              // ảnh hưởng object caller
void ReplaceInv(ref Inventory inv) { inv = new Inventory(); } // đổi biến caller
```

Clone thủ công (minh họa — chưa sâu ICloneable):

```csharp
Inventory CloneShallow(Inventory src)
{
    return src; // KHÔNG phải clone — cùng reference!
}

ItemSlot CloneSlot(ItemSlot src) => src; // với struct: đây là copy value hợp lệ
```

## 6. Ví dụ

### Cơ bản — dự đoán output

```csharp
int x = 1;
int y = x;
y++;
Console.WriteLine(x); // 1

var p1 = new Player { Hp = 1 };
var p2 = p1;
p2.Hp++;
Console.WriteLine(p1.Hp); // 2
```

### Trung cấp — method và side-effect

```csharp
struct ItemSlot
{
    public int ItemId;
    public int Count;
}

class Inventory
{
    public int Gold;
    public ItemSlot MainHand;
}

static void TryEmptySlot(ItemSlot slot) => slot.Count = 0;

static void TryEmptyMainHand(Inventory inv)
{
    var s = inv.MainHand; // COPY struct ra
    s.Count = 0;
    // quên: inv.MainHand = s;  → MainHand gốc không đổi!
}

static void EmptyMainHandCorrect(Inventory inv)
{
    var s = inv.MainHand;
    s.Count = 0;
    inv.MainHand = s;
}

var inv = new Inventory
{
    Gold = 10,
    MainHand = new ItemSlot { ItemId = 7, Count = 3 }
};

TryEmptySlot(inv.MainHand);
Console.WriteLine(inv.MainHand.Count); // 3 — method nhận copy

TryEmptyMainHand(inv);
Console.WriteLine(inv.MainHand.Count); // 3 — quên gán lại

EmptyMainHandCorrect(inv);
Console.WriteLine(inv.MainHand.Count); // 0
```

### Nâng cao — đổi reference vs đổi object

```csharp
static void RecreateWrong(Inventory inv) => inv = new Inventory { Gold = 0 };
static void RecreateRight(ref Inventory inv) => inv = new Inventory { Gold = 0 };

var bag = new Inventory { Gold = 99 };
RecreateWrong(bag);
Console.WriteLine(bag.Gold); // 99 — chỉ local param đổi

RecreateRight(ref bag);
Console.WriteLine(bag.Gold); // 0
```

## 7. Lỗi thường gặp

| Lỗi | Nguyên nhân | Cách xử lý |
|-----|-------------|------------|
| Tưởng truyền class là copy sâu | Chỉ copy reference | Clone tường minh nếu cần độc lập |
| Method “xóa struct” không ăn | Pass-by-value copy | Trả về struct mới / dùng `ref` / gán lại field |
| `transform.position.x = …` | Property copy | Đọc–sửa–gán |
| Nhầm `a = b` với clone | Hai tên một object | `new` + copy field thủ công |
| Sửa element struct qua property list wrapper | Getter copy | API trả `ref` / sửa mảng trực tiếp |

## 8. Gỡ lỗi

Hỏi 3 câu:

1. Biến chứa **value** hay **reference**?
2. Tôi đang sửa **biến** hay **object**?
3. Dữ liệu lấy từ **property** (copy) hay **ô nhớ thật** (field/array)?

Thêm log:

```csharp
Console.WriteLine(ReferenceEquals(a, b));
Console.WriteLine(a.GetHashCode()); // cẩn thận: có thể override
```

## 9. Best practices

- Struct: ưu tiên immutable → trả instance mới thay vì mutate copy
- Class: document method nào **mutate** object
- Cần độc lập: copy tường minh (memberwise / constructor copy)
- Unity: luôn đọc `position`/`rotation`/`localScale` ra biến tạm rồi gán lại
- Đừng dùng `ref` khắp nơi để “cho chắc” — chỉ khi cần đổi biến của caller

## 10. Bài tập

**Bài 1.** Viết bảng dự đoán output cho:

```csharp
var a = new Counter { N = 1 };
var b = a;
var c = new Counter { N = 1 };
b.N = 5;
Console.WriteLine($"{a.N} {b.N} {c.N} {ReferenceEquals(a,b)} {ReferenceEquals(a,c)}");
```

**Bài 2.** Method `void AddGold(Inventory inv, int x) => inv.Gold += x;` — có cần `ref` không? Vì sao?

**Bài 3.** Method muốn đặt `inv = null` cho caller — cần gì?

**Bài 4.** Mô phỏng bug Unity: class `Body` có property `Vector3 Pos`. Viết đoạn sai và đoạn đúng để tăng `Pos.Y`.

## 11. Gợi ý

- Bài 1: `a`/`b` chung object; `c` khác.
- Bài 2: không cần `ref` để sửa `Gold`.
- Bài 3: `ref` (hoặc trả về reference mới).
- Bài 4: biến tạm + gán lại property.

## 12. Đáp án + Giải thích

**Bài 1:** `5 5 1 True False`

**Bài 2:** Không cần `ref`. `inv` là bản sao **reference**, vẫn trỏ cùng `Inventory` → `Gold += x` mutate object.

**Bài 3:**

```csharp
void ClearRef(ref Inventory inv) => inv = null;
```

**Bài 4:**

```csharp
// Sai / không hiệu lực (thường lỗi biên dịch):
// body.Pos.Y += 1;

var p = body.Pos;
p.Y += 1;
body.Pos = p;
```

## 13. Đáp án thay thế

Bài 3 có thể `Inventory Clear(Inventory _) => null;` rồi `inv = Clear(inv);` — đổi bằng giá trị trả về thay vì `ref`.

Bài 2: nếu `Inventory` là **struct** (hiếm với hệ lớn) thì **cần** `ref` hoặc return lại — đó là lý do Inventory nên là class.

## 14. Thử thách

Viết `struct Matrix4` giả (16 `float`). Truyền vào method by-value trong loop 1e5 lần vs truyền `ref`. So sánh thời gian và giải thích liên hệ copy semantics + performance.

## 15. Ứng dụng thực tế

- Undo/redo: cần **bản sao độc lập** state — reference share sẽ làm hỏng history
- Multiplayer snapshot: serialize value rõ ràng; đừng share reference bất ngờ
- API: trả struct nhỏ thay vì expose field mutable qua property sai cách

## 16. Liên hệ Unity

| Thao tác | Semantics |
|----------|-----------|
| `var t = transform;` | Copy **reference** tới cùng Transform |
| `var p = transform.position;` | Copy **Vector3 value** |
| `t.position = p;` | Gán value vào property (setter) |
| `GetComponent<T>()` | Reference tới component trên heap |

**GC:** tạo `new class` tạm để “copy” mỗi frame → allocation. Struct copy không tạo object mới (nhưng tốn CPU theo kích thước).

> **Cảnh báo lặp lại:** Mutating struct copy không đổi bản gốc. `transform.position.x = …` là case điển hình.

## 17. Kiểm tra kiến thức

1. C# mặc định pass parameter theo? (A) by-reference luôn (B) by-value (copy biến)  
2. Pass class cho method, sửa field — caller thấy? (A) Thường có (B) Không bao giờ  
3. Pass struct, sửa field trong method — caller thấy? (A) Có (B) Không (trừ `ref`)  
4. `a = b` với class là clone sâu? (A) Đúng (B) Sai  
5. Muốn `transform` đổi vị trí sau khi sửa x? (A) Gán `.x` trực tiếp (B) Đọc–sửa–gán lại `position`  

**Đáp án:** 1B, 2A, 3B, 4B, 5B
