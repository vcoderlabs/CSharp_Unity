# Chương 5 — Truyền tham số: `ref`, `out`, `in`

## 1. Mục tiêu học

- Phân biệt truyền by-value mặc định với `ref` / `out` / `in`
- Biết khi nào **cần** `ref` với class (đổi biến reference) vs khi nào không
- Dùng `out` cho API trả nhiều kết quả / Try-pattern
- Dùng `in` để tránh copy struct lớn (readonly-ish)

## 2. Điều kiện tiên quyết

- Chương 4 — Copying semantics
- Method, return value (Level 1)

## 3. Khái niệm

### Mặc định (by-value)

Method nhận **bản sao** của đối số. Với class: bản sao của **reference**.

### `ref`

- Truyền **alias** tới biến của caller
- Phải **khởi tạo trước** khi truyền
- Method có thể đọc/ghi; có thể cho biến trỏ object khác

### `out`

- Giống `ref` về “truyền biến thật”, nhưng dùng để **output**
- Caller **không cần** khởi tạo trước
- Method **phải** gán trước khi return (theo quy tắc compiler)

### `in` (C# 7.2+)

- Truyền by-reference nhưng **readonly** (không gán lại biến; hạn chế mutate trong nhiều case)
- Mục tiêu: tránh copy struct **lớn**, vẫn an toàn hơn `ref` mutable
- Với struct nhỏ (`int`), `in` thường **không** cần — đôi khi còn chậm hơn do indirection

### Try-pattern

```csharp
bool TryParse(string s, out int value);
```

Trả `bool` thành công + `out` kết quả — rất phổ biến trong .NET.

## 4. Mô hình tư duy

### `ref` với value type

```text
void AddOne(ref int x) => x++;

STACK caller
n = 10

AddOne(ref n)  →  x là tên khác của n
sau: n = 11
```

### `ref` với class — đổi reference

```text
void Replace(ref Player p) => p = new Player();

Trước: caller.player ──► ObjectA
Sau:   caller.player ──► ObjectB (mới)

Không có ref: chỉ local param đổi; caller vẫn ObjectA
```

### `in` với struct lớn

```text
void Draw(in Matrix4x4 m) { ... dùng m ... }

Không copy 64 bytes (ví dụ) lên stack frame như by-value;
truyền địa chỉ readonly tới bản gốc.
```

### Nhầm lẫn nguy hiểm với struct property (Unity)

```text
// KHÔNG liên quan ref/out/in:
transform.position.x = 1; // vẫn sai — property trả copy

// ref không “sửa giúp” property getter trả temporary theo cách bạn kỳ vọng ở đây.
```

## 5. Cú pháp

```csharp
void Scale(ref int value, int factor) => value *= factor;

void CreateDefault(out ItemSlot slot)
{
    slot = new ItemSlot { ItemId = 0, Count = 0 };
}

double LengthSq(in Vector3Fake v) => v.X * v.X + v.Y * v.Y + v.Z * v.Z;

// Gọi
int hp = 10;
Scale(ref hp, 2);

CreateDefault(out ItemSlot empty); // out var empty cũng được
CreateDefault(out var empty2);

var v = new Vector3Fake(1, 2, 3);
Console.WriteLine(LengthSq(in v));
```

Try-pattern:

```csharp
static bool TryGetSlot(Inventory inv, int index, out ItemSlot slot)
{
    if (index < 0 || index >= inv.Slots.Length)
    {
        slot = default;
        return false;
    }
    slot = inv.Slots[index];
    return true;
}
```

## 6. Ví dụ

### Cơ bản — `ref` đổi `int`

```csharp
static void Triple(ref int n) => n *= 3;

int score = 4;
Triple(ref score);
Console.WriteLine(score); // 12
```

### Trung cấp — `out` nhiều kết quả

```csharp
static bool TryDivide(int a, int b, out int quotient, out int remainder)
{
    if (b == 0)
    {
        quotient = 0;
        remainder = 0;
        return false;
    }
    quotient = a / b;
    remainder = a % b;
    return true;
}

if (TryDivide(17, 5, out int q, out int r))
    Console.WriteLine($"{q} dư {r}"); // 3 dư 2
```

### Nâng cao — class: mutate object vs `ref` thay reference

```csharp
class Bag
{
    public int Gold;
}

static void AddGold(Bag bag) => bag.Gold += 5;           // không cần ref
static void NewBag(ref Bag bag) => bag = new Bag { Gold = 0 }; // cần ref

var b = new Bag { Gold = 10 };
AddGold(b);
Console.WriteLine(b.Gold); // 15

NewBag(ref b);
Console.WriteLine(b.Gold); // 0
```

`in` + struct:

```csharp
readonly struct Mat2
{
    public readonly float M00, M01, M10, M11;
    public Mat2(float m00, float m01, float m10, float m11)
    {
        M00 = m00; M01 = m01; M10 = m10; M11 = m11;
    }
}

static float Trace(in Mat2 m) => m.M00 + m.M11;
```

## 7. Lỗi thường gặp

| Lỗi | Nguyên nhân | Cách xử lý |
|-----|-------------|------------|
| `ref` mà biến chưa gán | Quy tắc `ref` | Khởi tạo trước / dùng `out` |
| Quên gán `out` trong mọi nhánh | Compiler bắt buộc | Gán `default` trên nhánh lỗi |
| Dùng `ref` chỉ để sửa field class | Không cần | Bỏ `ref` cho rõ nghĩa |
| `in` với struct nhỏ mọi chỗ | Indirection overhead | Chỉ struct lớn / hot path đo được |
| Kỳ vọng `ref` sửa `transform.position.x` | Property copy | Đọc–sửa–gán; API Unity không expose `ref` position kiểu đó |

## 8. Gỡ lỗi

1. Đọc thông báo compiler: “must be assigned before…” → `out`/`ref` rules.
2. Nếu method không đổi caller như kỳ vọng: bạn có đang sửa **copy** struct không?
3. Với class: in `ReferenceEquals` trước/sau nếu nghi `ref` thay instance.
4. Đừng thêm `ref` để “fix Unity position bug” — đó là **copy semantics**, không thiếu `ref`.

## 9. Best practices

- Prefer **return value** rõ ràng khi có thể (một kết quả)
- `out` + `bool` cho Try-pattern
- `ref` khi cần đổi biến caller hoặc tránh copy struct lớn **và** cho phép mutate
- `in` cho struct lớn readonly trên hot path (đo trước khi áp dụng mù quáng)
- Đặt tên method phản ánh side-effect (`Try…`, `Replace…`, `Init…`)

## 10. Bài tập

**Bài 1.** Viết `Swap(ref int a, ref int b)`.

**Bài 2.** Viết `TryParsePositive(string s, out int value)` — true chỉ khi parse được và `> 0`.

**Bài 3.** Cho `class Player { public int Hp; }`. Method `Kill` đặt `Hp = 0` — có cần `ref`? Method `Respawn` gán `player = new Player { Hp = 100 }` cho caller — cần gì?

**Bài 4.** Giải thích ngắn: tại sao `in Vector3` hữu ích hơn `in int` trong hot path?

## 11. Gợi ý

- Bài 1: biến tạm.
- Bài 2: `int.TryParse` bên trong + kiểm tra `> 0`.
- Bài 3: Kill không cần; Respawn cần `ref` (hoặc return).
- Bài 4: tránh copy nhiều byte; `int` copy đã cực rẻ.

## 12. Đáp án + Giải thích

**Bài 1:**

```csharp
static void Swap(ref int a, ref int b)
{
    int t = a;
    a = b;
    b = t;
}
```

**Bài 2:**

```csharp
static bool TryParsePositive(string s, out int value)
{
    if (int.TryParse(s, out value) && value > 0)
        return true;
    value = 0;
    return false;
}
```

**Bài 3:**

```csharp
static void Kill(Player p) => p.Hp = 0;

static void Respawn(ref Player p) => p = new Player { Hp = 100 };
```

**Bài 4:** `Vector3` (12 bytes+) copy lặp lại tốn hơn; `in` tránh copy. `int` (4 bytes) copy rẻ — `in` ít lợi, đôi khi hại.

## 13. Đáp án thay thế

Bài 1 có thể generic `Swap<T>(ref T a, ref T b)`.

Bài 3 Respawn: `static Player Respawn() => new Player { Hp = 100 };` rồi `p = Respawn();`.

## 14. Thử thách

Implement `TryCombineStacks(ItemSlot a, ItemSlot b, out ItemSlot result)`:

- Cùng `ItemId` và không tràn `MaxStack` → gộp, return true
- Ngược lại `result = default`, false  
Viết unit-style tests bằng `Console.Assert` hoặc if + throw.

## 15. Ứng dụng thực tế

- `Dictionary.TryGetValue`, `int.TryParse` — mẫu chuẩn production
- Math library: `in Matrix4x4` / `in Quaternion`
- Game inventory: `TryAddItem(..., out int remaining)` 

## 16. Liên hệ Unity

- Nhiều API Unity dùng **property struct** thay vì `ref` trả về — vì thế pattern đọc–sửa–gán là bắt buộc với `position`/`rotation`
- `Physics.Raycast(..., out RaycastHit hit)` — `out` rất quen thuộc
- Performance: `in` với `Matrix4x4` trong vòng render/update có thể giảm copy; vẫn đo Profiler
- GC: `ref`/`in`/`out` **không** cấp phát heap chỉ vì dùng modifier — allocation đến từ `new` class / boxing

> **Nhắc lại:** Dù thành thạo `ref`/`out`/`in`, bug `transform.position.x = …` vẫn xảy ra vì bạn mutate **struct copy** từ property — hãy gán lại cả `Vector3`.

## 17. Kiểm tra kiến thức

1. `out` bắt buộc gán trong method trước khi return? (A) Có (B) Không  
2. Sửa `player.Hp` trong method — cần `ref`? (A) Không (B) Luôn cần  
3. Muốn caller nhận object mới hoàn toàn qua tham số — dùng? (A) by-value thường (B) `ref`/`out` hoặc return  
4. `in` chủ yếu giúp? (A) Tránh copy struct lớn (B) Cho phép gán null vào class dễ hơn  
5. `transform.position.x = 1` sửa bằng cách thêm `ref`? (A) Đúng (B) Sai — phải đọc–sửa–gán  

**Đáp án:** 1A, 2A, 3B, 4A, 5B
