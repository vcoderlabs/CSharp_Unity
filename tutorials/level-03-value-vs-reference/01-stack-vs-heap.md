# Chương 1 — Stack vs Heap

## 1. Mục tiêu học

- Phân biệt **Stack** và **Heap** trong CLR
- Biết value type thường nằm đâu, reference type nằm đâu
- Đọc được sơ đồ ASCII bộ nhớ khi gán biến
- Hiểu tại sao sửa “bản sao” struct không đụng tới bản gốc

## 2. Điều kiện tiên quyết

- Level 1: biến, kiểu nguyên thủy, method
- Level 2: class, object, `new`, field/property

## 3. Khái niệm

### Stack (ngăn xếp)

- Vùng nhớ **nhanh**, quản lý theo **LIFO** (vào sau – ra trước)
- Mỗi lần gọi method → tạo **frame** trên stack (tham số, biến cục bộ)
- Khi method **return** → frame bị hủy (biến cục bộ biến mất)
- Phù hợp dữ liệu **nhỏ, tuổi thọ ngắn** (theo scope method)

### Heap (đống)

- Vùng nhớ lớn hơn, cấp phát bằng `new` (object / mảng / boxing…)
- Object sống đến khi **không còn reference** trỏ tới → GC thu gom
- Chậm hơn stack về cấp phát/thu hồi; có **GC pause** nếu tạo quá nhiều

### Value type vs Reference type (nhìn qua bộ nhớ)

| | Value type | Reference type |
|---|------------|----------------|
| Ví dụ | `int`, `bool`, `struct`, `enum` | `class`, `string`, `array`, `delegate` |
| Biến chứa | **Giá trị thật** | **Địa chỉ** (reference) trỏ tới object trên heap |
| Gán `a = b` | Copy **giá trị** | Copy **địa chỉ** (hai biến cùng trỏ một object) |

> **Cảnh báo:** Value type **có thể** nằm trên heap (field trong class, phần tử mảng, boxing). “Value type luôn trên stack” là **đơn giản hóa sai** — đúng hơn: biến value type **chứa dữ liệu inline**, không chứa reference.

## 4. Mô hình tư duy

### 4.1. Value type cục bộ trên Stack

```text
void Demo()
{
    int hp = 100;
    int mp = hp;   // copy giá trị
    mp = 50;       // chỉ đổi mp
}

STACK (frame Demo)
┌─────────────┐
│ hp = 100    │
│ mp = 50     │   ← độc lập với hp
└─────────────┘

HEAP: (trống với ví dụ này)
```

### 4.2. Reference type: biến trên Stack, object trên Heap

```text
void Demo()
{
    Player a = new Player { Hp = 100 };
    Player b = a;      // copy REFERENCE
    b.Hp = 50;         // sửa object chung → a.Hp cũng 50
}

STACK                      HEAP
┌──────────────┐         ┌──────────────────┐
│ a ───────────┼────────►│ Player           │
│ b ───────────┼────────►│   Hp = 50        │
└──────────────┘         └──────────────────┘
         cùng một object
```

### 4.3. Struct field trong class (value type trên Heap)

```text
class Inventory
{
    public ItemSlot slot;  // struct nằm INSIDE object trên heap
}

STACK                 HEAP (Inventory object)
┌──────────┐         ┌─────────────────────────┐
│ inv ─────┼────────►│ ItemSlot slot           │
└──────────┘         │   ItemId=1 Count=5      │  ← value nằm trong object
                     └─────────────────────────┘
```

### 4.4. Bug kinh điển: sửa bản sao

```text
Vector3 p = transform.position;  // COPY struct từ property
p.x = 5;                         // sửa bản sao trên stack
// transform.position vẫn cũ!    // chưa gán lại

STACK sau p.x = 5:
┌─────────────────────┐
│ p = (5, y, z)       │  ← chỉ bản sao đổi
└─────────────────────┘

HEAP / Transform:
position vẫn là (x_cũ, y, z)  ← bản gốc không đổi
```

## 5. Cú pháp

```csharp
// Value type — không cần new (nhưng struct có thể dùng new để gọi constructor)
int score = 10;
Vector3 local = new Vector3(1, 2, 3); // vẫn là value type

// Reference type — object trên heap
var player = new Player();
Player other = player; // cùng reference
other = null;          // chỉ other thôi; player vẫn trỏ object (nếu còn)
```

Kiểm tra kiểu:

```csharp
typeof(int).IsValueType;     // true
typeof(string).IsValueType;  // false (class)
typeof(Player).IsValueType;  // false
```

## 6. Ví dụ

### Cơ bản — hai `int` độc lập

```csharp
int a = 7;
int b = a;
b = 99;
Console.WriteLine($"a={a}, b={b}"); // a=7, b=99
```

### Trung cấp — class dùng chung object

```csharp
class Counter
{
    public int Value;
}

var c1 = new Counter { Value = 1 };
var c2 = c1;
c2.Value = 100;
Console.WriteLine(c1.Value); // 100 — cùng heap object
```

### Nâng cao — struct trong class + copy field

```csharp
struct Point { public int X; public int Y; }

class Node
{
    public Point Pos;
}

var n = new Node { Pos = new Point { X = 1, Y = 2 } };

Point copy = n.Pos;   // copy VALUE ra stack
copy.X = 999;
Console.WriteLine(n.Pos.X); // vẫn 1

n.Pos = copy;         // gán lại cả struct → mới đổi
Console.WriteLine(n.Pos.X); // 999
```

## 7. Lỗi thường gặp

| Lỗi / Nhầm | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Nghĩ mọi biến đều trên stack | Quên object trên heap | Vẽ sơ đồ: biến chứa gì? |
| `NullReferenceException` | Reference = `null` rồi gọi member | Kiểm tra null / khởi tạo |
| Sửa property struct rồi “không ăn” | Property trả **bản sao** | Đọc ra biến, sửa, gán lại |
| StackOverflowException | Đệ quy vô hạn / stack quá sâu | Kiểm tra đệ quy, giảm độ sâu |
| Tin “struct luôn stack” | Field/array/boxing trên heap | Nhớ: semantics copy, không phải “luôn stack” |

## 8. Gỡ lỗi

1. Với bug “sửa rồi không đổi”: hỏi **đây là value hay reference?**
2. Dùng debugger: xem **địa chỉ/hash** của hai biến class (cùng object?)
3. Với struct: in giá trị trước/sau gán; nếu khác nhau → bạn đang giữ hai bản copy.
4. Trong Unity: nếu sửa `transform.position.x` — đó là **compiler/API design** trả về struct copy; không phải “Unity hỏng”.

```csharp
// Pattern đúng khi sửa Vector3 (Unity)
var p = transform.position;
p.x += 1f;
transform.position = p;
```

## 9. Best practices

- Value type nhỏ (`int`, `bool`, `Vector3`, `ItemSlot` nhẹ) → copy rẻ, tránh surprise share
- Dữ liệu lớn / identity / kế thừa → `class` trên heap
- Đừng tối ưu “stack vs heap” sớm — ưu tiên **đúng semantics**
- Luôn nhớ: **gán value = copy dữ liệu; gán reference = copy địa chỉ**

## 10. Bài tập

**Bài 1.** Vẽ ASCII stack/heap cho đoạn:

```csharp
int x = 3;
string s = "hi";
```

**Bài 2.** Dự đoán output:

```csharp
var a = new Box { N = 1 };
var b = a;
b.N = 2;
Console.WriteLine(a.N);
// class Box { public int N; }
```

**Bài 3.** Viết code `struct Coin { public int Value; }` — tạo `c1`, gán `c2 = c1`, đổi `c2.Value`. In cả hai. Giải thích.

**Bài 4.** Giải thích vì sao đoạn Unity sau **không** đổi vị trí:

```csharp
transform.position.x = 10f;
```

## 11. Gợi ý

- Bài 1: `x` trên stack; `s` trên stack là reference → heap chứa string object.
- Bài 2: `a` và `b` cùng object → `a.N == 2`.
- Bài 3: struct copy → `c1.Value` giữ nguyên.
- Bài 4: `position` getter trả `Vector3` tạm; gán `.x` sửa tạm rồi bỏ.

## 12. Đáp án + Giải thích

**Bài 1 (sơ đồ gợi ý):**

```text
STACK              HEAP
x = 3              ┌──────────┐
s ────────────────►│ "hi"     │
                   └──────────┘
```

**Bài 2:** In `2`. Reference copy → cùng `Box` trên heap.

**Bài 3:**

```csharp
struct Coin { public int Value; }
Coin c1 = new Coin { Value = 10 };
Coin c2 = c1;
c2.Value = 99;
Console.WriteLine($"{c1.Value} {c2.Value}"); // 10 99
```

**Bài 4:** Property `position` trả **bản sao** `Vector3`. Gán field trên bản sao không ghi lại vào `Transform`. Cần đọc–sửa–gán lại.

## 13. Đáp án thay thế

Bài 1 có thể vẽ string intern phức tạp hơn — với mức này, “reference → heap object chứa ký tự” là đủ.

Bài 3 có thể dùng `record struct` (C# 10+) — semantics copy vẫn giống value type.

## 14. Thử thách

Viết method đệ quy đếm xuống từ `n` và giải thích **mỗi lần gọi thêm một frame trên stack**. Thử `n` rất lớn (cẩn thận `StackOverflowException`) và ghi lại quan sát.

## 15. Ứng dụng thực tế

- API trả về struct lớn: mỗi lần gọi getter có thể **copy** → tránh gọi lặp trong vòng hot path.
- Logging/debug: in `ReferenceEquals(a, b)` để xác nhận cùng object.
- Game: HP/`int` trên stack frame nhanh; `Player` state phức tạp trên heap.

## 16. Liên hệ Unity

| Kiểu | Value/Ref | Ghi chú |
|------|-----------|---------|
| `Vector3`, `Quaternion`, `Color` | **struct** | Copy khi gán / lấy từ property |
| `Transform`, `GameObject`, `Rigidbody` | **class** | Reference; `a = b` cùng instance |
| `string` | class (immutable) | Reference nhưng nội dung không sửa tại chỗ |

**GC:** tạo nhiều `new class` tạm mỗi frame → GC spike → hitch. Struct nhỏ giúp giảm allocation trên heap (nhưng copy struct **lớn** cũng tốn CPU).

**Vì sao struct quan trọng performance:** dữ liệu nhỏ nằm inline (cache-friendly), ít object header, ít áp lực GC — nền tảng của DOTS/`NativeArray` sau này.

> Nhắc lại: `transform.position.x = ...` sửa **bản sao** → bug kinh điển value type.

## 17. Kiểm tra kiến thức

1. Biến `int` cục bộ chứa gì? (A) reference (B) giá trị số  
2. `Player a = b` (class) copy gì? (A) toàn bộ field (B) địa chỉ  
3. Value type **luôn** trên stack? (A) Đúng (B) Sai  
4. `transform.position.x = 1` có đổi vị trí? (A) Có (B) Không  
5. Khi method return, biến cục bộ trên stack frame thường ra sao? (A) Bị hủy theo frame (B) Sống đến khi GC  

**Đáp án:** 1B, 2B, 3B, 4B, 5A
