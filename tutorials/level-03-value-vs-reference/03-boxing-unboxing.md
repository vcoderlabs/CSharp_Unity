# Chương 3 — Boxing và Unboxing

## 1. Mục tiêu học

- Giải thích boxing / unboxing là gì trên heap
- Nhận diện chỗ code **ẩn** boxing (`object`, non-generic collection, interface)
- Hiểu chi phí: allocation + cast + GC
- Tránh boxing không cần thiết trong game loop / hot path

## 2. Điều kiện tiên quyết

- Chương 1–2: Stack/Heap, struct vs class
- Biết `object` là base type của mọi kiểu trong C#

## 3. Khái niệm

### Boxing

Đưa **value type** vào “hộp” `object` (hoặc interface reference) trên **heap**:

1. Cấp phát object trên heap
2. **Copy** giá trị value type vào object đó
3. Biến reference trỏ tới hộp đó

### Unboxing

Lấy giá trị ra khỏi hộp:

1. Kiểm tra kiểu (sai → `InvalidCastException`)
2. **Copy** giá trị từ heap về biến value type

### Vì sao quan trọng?

Mỗi lần box → **allocation** trên heap → thêm việc cho **GC**. Trong Unity, box trong `Update` dễ gây hitch.

> Boxing **không** phải “ magically biến struct thành class mãi mãi” — bạn đang giữ **bản sao** trong hộp. Sửa bản gốc sau khi box **không** đổi hộp (và ngược lại sau unbox là bản sao mới).

## 4. Mô hình tư duy

```text
int x = 42;

object boxed = x;   // BOXING
// 1) cấp phát trên heap
// 2) copy 42 vào object
// 3) boxed giữ reference

STACK                 HEAP
┌──────────┐         ┌─────────────────┐
│ x = 42   │         │ boxed int       │
│ boxed ───┼────────►│   value = 42    │
└──────────┘         └─────────────────┘

x = 100;             // chỉ đổi stack; hộp vẫn 42

int y = (int)boxed;  // UNBOXING — copy 42 ra y
```

### Boxing qua interface

```text
struct Hit : IEquatable<Hit> { ... }

IEquatable<Hit> asIface = new Hit(...);
// struct bị BOX nếu gán vào biến interface (trừ pattern đặc biệt / generic)

STACK                    HEAP
asIface ───────────────► [box chứa Hit]
```

## 5. Cú pháp

```csharp
int n = 10;
object o = n;          // boxing tường minh về mặt semantics
int m = (int)o;        // unboxing — cast đúng kiểu

// Sai kiểu khi unbox:
// double d = (double)o; // InvalidCastException
// Phải: double d = (double)(int)o; hoặc Convert

// Boxing ẩn:
object[] arr = { 1, 2, 3 }; // mỗi int bị box
Console.WriteLine(string.Format("{0}", 5)); // boxing với một số overload cũ
```

Non-generic (tránh trong code mới):

```csharp
// System.Collections.ArrayList — mọi phần tử là object → value type bị box
var list = new System.Collections.ArrayList();
list.Add(42); // boxing
int v = (int)list[0]; // unboxing
```

Generic (không box với `List<int>`):

```csharp
var nums = new List<int>();
nums.Add(42); // không boxing
```

## 6. Ví dụ

### Cơ bản — box và unbox tường minh

```csharp
int hp = 100;
object box = hp;       // box
hp = 1;
Console.WriteLine(box); // 100 — bản sao trong hộp
int restored = (int)box;
Console.WriteLine(restored); // 100
```

### Trung cấp — so sánh `ArrayList` vs `List<int>`

```csharp
using System.Collections;
using System.Collections.Generic;

ArrayList legacy = new ArrayList();
legacy.Add(1); // box
legacy.Add(2); // box

List<int> modern = new List<int>();
modern.Add(1); // không box
modern.Add(2);

Console.WriteLine((int)legacy[0] + modern[0]);
```

### Nâng cao — boxing khi gọi interface trên struct

```csharp
interface IStat { int Value { get; } }

struct Attack : IStat
{
    public int Value { get; set; }
}

void Print(IStat s) // tham số interface → nếu truyền struct: BOX
{
    Console.WriteLine(s.Value);
}

Attack atk = new Attack { Value = 15 };
Print(atk); // boxing

// Tránh bằng generic:
void PrintGeneric<T>(T s) where T : IStat
{
    Console.WriteLine(s.Value); // thường không box
}
PrintGeneric(atk);
```

## 7. Lỗi thường gặp

| Lỗi | Nguyên nhân | Cách xử lý |
|-----|-------------|------------|
| `InvalidCastException` khi unbox | Cast sai kiểu (unbox `int` thành `long` trực tiếp) | Unbox đúng kiểu gốc rồi convert |
| GC spike khó hiểu | Boxing ẩn trong loop | Dùng generic, tránh `object` |
| Sửa struct sau khi box, kỳ vọng hộp đổi | Hộp chứa **bản sao** | Hiểu copy semantics |
| `params object[]` / string format cũ | Value type bị box | Prefer interpolated string / generic API |
| So sánh `object` chứa value | Nhầm reference equality | Unbox rồi so sánh value |

## 8. Gỡ lỗi

1. Nghi ngờ boxing → tìm chỗ gán value type vào `object` / non-generic / interface.
2. Unity Profiler: **GC.Alloc** trên dòng nghi ngờ.
3. Thay `ArrayList`/`Hashtable` bằng `List<T>`/`Dictionary<,>`.
4. Với interface + struct: cân nhắc generic `where T : IInterface`.

```csharp
// Unbox an toàn hơn khi không chắc:
if (box is int i)
{
    Console.WriteLine(i);
}
```

## 9. Best practices

- Ưu tiên **generic** (`List<T>`, `Dictionary<TKey,TValue>`)
- Tránh API nhận `object` trên hot path
- Struct implement interface: cẩn thận khi dùng **như** interface (boxing)
- Đừng micro-optimize mọi boxing hiếm — ưu tiên chỗ chạy mỗi frame
- Logging/debug có thể box — chấp nhận được ngoài hot path

## 10. Bài tập

**Bài 1.** Viết đoạn box `bool` vào `object`, đổi biến gốc, in hộp — giải thích kết quả.

**Bài 2.** Dự đoán có boxing không:

```csharp
object o = 3.14;          // ?
List<double> xs = new();
xs.Add(3.14);             // ?
IComparable c = 10;       // ?
```

**Bài 3.** Sửa code sau để hết boxing không cần thiết:

```csharp
var data = new ArrayList();
for (int i = 0; i < 1000; i++) data.Add(i);
```

**Bài 4.** Giải thích vì sao unbox `(long)(object)42` (với `42` là int) lại lỗi, dù `int`→`long` thường hợp lệ.

## 11. Gợi ý

- Bài 1: hộp giữ bản sao lúc box.
- Bài 2: có / không / có.
- Bài 3: `List<int>`.
- Bài 4: unbox đòi **đúng** kiểu đã box; không convert ngầm lúc unbox.

## 12. Đáp án + Giải thích

**Bài 1:**

```csharp
bool flag = true;
object box = flag;
flag = false;
Console.WriteLine(box); // True
```

**Bài 2:** `object o = 3.14` → **có** boxing. `List<double>.Add` → **không**. `IComparable c = 10` → **có** boxing.

**Bài 3:**

```csharp
var data = new List<int>();
for (int i = 0; i < 1000; i++) data.Add(i);
```

**Bài 4:** Object chứa **boxed int**. Unbox phải `(int)o` rồi mới `(long)i`. Unbox thẳng sang `long` → `InvalidCastException`.

## 13. Đáp án thay thế

Bài 3 có thể dùng `int[]` cấp phát sẵn nếu biết kích thước — cũng không box.

Bài 2: với pattern matching / generic có thể tránh boxing trong vài case nâng cao — ở Level 3 chỉ cần nhận diện case cổ điển.

## 14. Thử thách

Viết benchmark đơn giản (loop 1e6): thêm `int` vào `ArrayList` vs `List<int>`. So thời gian và (nếu có thể) quan sát allocation. Viết kết luận 3–5 câu.

## 15. Ứng dụng thực tế

- Library cũ / reflection / serialization đôi khi bắt buộc `object` → chấp nhận boxing có kiểm soát
- Game server tick / Unity `Update`: săn GC.Alloc từ boxing
- API public: prefer generic constraints hơn `object`

## 16. Liên hệ Unity

- `Object.FindObjectsOfType` kiểu cũ / API non-generic historically dễ dẫn tới pattern kém tối ưu
- So sánh tag, message (`SendMessage`) kiểu cũ — cẩn thận allocation
- `Vector3` là struct: đưa vào `object` hoặc non-generic list → **box** + copy
- GC: boxing tạo heap object ngắn sống → pressure; struct đúng chỗ giúp **tránh** allocation không cần thiết

> Vẫn nhớ: ngay cả khi không box, **sửa copy của `Vector3`** (`transform.position.x = …`) cũng không đổi Transform — đó là copy semantics, không phải boxing.

## 17. Kiểm tra kiến thức

1. Boxing xảy ra khi? (A) int → object (B) object class → object  
2. `List<int>.Add(1)` có box? (A) Có (B) Không  
3. Unbox sai kiểu dẫn tới? (A) null (B) InvalidCastException  
4. Boxing tạo object trên? (A) Stack (B) Heap  
5. Gán struct vào biến interface thường? (A) Không box (B) Box  

**Đáp án:** 1A, 2B, 3B, 4B, 5B
