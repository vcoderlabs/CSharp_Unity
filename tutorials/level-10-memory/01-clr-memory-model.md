# Chương 1 — CLR Memory Model

## 1. Mục tiêu học

- Hiểu **CLR** quản lý bộ nhớ như thế nào trên .NET 8+
- Phân biệt **stack**, **managed heap**, **unmanaged memory**
- Biết object header, method table, và reference trỏ tới đâu
- Liên hệ: mỗi `new` trong Unity đều đụng managed heap → GC

## 2. Điều kiện tiên quyết

- Level 3: stack vs heap, value vs reference, boxing
- Level 1–2: class, `new`, method
- Biết Unity có C#/.NET (Mono / IL2CPP) — khái niệm GC giống hướng

## 3. Khái niệm

### CLR là gì?

**CLR (Common Language Runtime)** chạy IL của bạn: JIT (hoặc AOT), type safety, **Garbage Collector**, exception, threading.

Bạn **không** `malloc`/`free` cho object C# thông thường — CLR cấp phát trên **managed heap** và GC thu hồi khi không còn root trỏ tới.

### Ba vùng nhớ cần nhớ

| Vùng | Ai quản lý | Ví dụ |
|------|------------|--------|
| **Stack** | Runtime (per-thread) | Tham số, biến cục bộ, return address |
| **Managed heap** | GC | `new Player()`, `new byte[100]`, boxing |
| **Unmanaged** | Bạn (hoặc OS) | File handle, `IntPtr`, native plugin, một phần Unity native |

### Object trên managed heap gồm gì?

Mỗi reference-type instance thường có:

```text
┌──────────────────────────────┐
│ Object header (sync, GC info)│
│ Method table / type pointer  │  ← biết đây là Player, không phải Enemy
│ Instance fields...           │
└──────────────────────────────┘
```

Biến reference trên stack chỉ giữ **địa chỉ** (hoặc null), không giữ toàn bộ object.

### Allocation = tăng “allocation pointer”

CLR giữ con trỏ bump-allocate trên Gen0: `new` gần như chỉ tăng pointer + khởi tạo field — **nhanh**. Đắt nằm ở chỗ: object chết hàng loạt → GC phải **đánh dấu / quét / compact** → pause.

### Value type vẫn quan trọng

- `int` cục bộ: trên stack frame (thường)
- `int` field trong class: nằm **trong** object trên heap
- Boxing `object o = 42`: **object mới** trên heap — allocation ẩn, hay gặp trong Unity API cũ / non-generic

## 4. Mô hình tư duy

```text
THREAD STACK                 MANAGED HEAP (GC)
┌─────────────────┐          ┌─────────────────────────────┐
│ Main frame      │          │  [Gen0 nursery - nóng]      │
│  player ────────┼─────────►│   Player { Hp, Name... }    │
│  dmg = 10       │          │   string "Hero"             │
│                 │          │  [Gen1]                     │
│ Call Update()   │          │   long-lived Manager        │
│  tmp list ──────┼─────────►│  [Gen2]                     │
└─────────────────┘          │   static caches, singletons │
                             │  [LOH > ~85KB]              │
                             │   large arrays, textures*?  │
                             └─────────────────────────────┘
* Texture pixel data thường native; managed side vẫn có wrapper object.
```

**Unity góc nhìn:**

```text
C# script (managed)  ←→  Unity Engine (native C++)
     new GameObject() cũng tạo managed wrapper + native object
     Destroy() ≠ GC ngay — managed wrapper có thể sống tới khi không còn ref
```

## 5. Cú pháp

```csharp
// Allocation rõ ràng
var p = new Player("Ada", 100);

// Allocation ẩn (boxing)
object boxed = 42;
IComparable c = 10; // boxing nếu không generic

// Mảng trên heap
int[] scores = new int[16];

// Stackalloc: bộ nhớ stack tạm (advanced, unsafe / Span)
Span<int> buf = stackalloc int[8];

// Đo allocation thô (diagnostic)
long before = GC.GetAllocatedBytesForCurrentThread();
_ = new byte[1024];
long after = GC.GetAllocatedBytesForCurrentThread();
Console.WriteLine($"Allocated ~{after - before} bytes");
```

## 6. Ví dụ

### Cơ bản — Reference trên stack, object trên heap

```csharp
class Player
{
    public string Name { get; set; } = "";
    public int Hp { get; set; }
}

static void Demo()
{
    Player a = new Player { Name = "A", Hp = 100 };
    Player b = a;      // copy reference
    b.Hp = 50;
    Console.WriteLine(a.Hp); // 50 — cùng một object trên heap
}
```

### Trung cấp — Đo allocation trong vòng lặp “giống Update”

```csharp
static void FakeUpdateAllocating(int frames)
{
    long start = GC.GetAllocatedBytesForCurrentThread();
    for (int i = 0; i < frames; i++)
    {
        // Giống antipattern Unity: new mỗi frame
        var evt = new DamageEvent(i, 10);
        _ = evt.ToString(); // có thể thêm allocation string
    }
    long end = GC.GetAllocatedBytesForCurrentThread();
    Console.WriteLine($"~{end - start} bytes / {frames} frames");
}

readonly record struct DamageEvent(int TargetId, int Amount);
// Dùng struct + reuse → ít/no heap alloc nếu không box
```

### Nâng cao — Phân biệt managed vs cần Dispose

```csharp
// Managed only — GC lo (nhưng vẫn tốn GC nếu spam)
var list = new List<int>(1024);

// Wrap resource OS — cần IDisposable (chương 4–5)
using var fs = File.OpenRead("data.bin");

// IntPtr / native — bạn chịu trách nhiệm (hoặc SafeHandle)
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Nghĩ `int` luôn trên stack | Field trong class nằm trên heap | Nhớ: “nơi chứa” quyết định |
| Nghĩ `Destroy(go)` = free managed ngay | Unity native destroy ≠ GC collect | Null ref + tránh giữ list chết |
| Sợ mọi allocation | Allocation nhỏ Gen0 thường rẻ | Tránh **allocation/frame** liên tục |
| Dùng `GC.Collect()` “cho chắc” | Gây spike nhân tạo | Chỉ diagnostic / rất hiếm production |
| Quên boxing trong interface non-generic | `IEnumerable` cũ, `ArrayList` | Generic + struct cẩn thận |

## 8. Gỡ lỗi

1. `GC.GetTotalMemory(false)` / `GetAllocatedBytesForCurrentThread()` — xem xu hướng tăng.
2. Rider / VS / Unity Profiler → **CPU Usage** + **Memory** / **GC Alloc**.
3. Trong Unity: cột **GC Alloc** trên Timeline — bất kỳ script `Update` nào > 0B đáng nghi.
4. Đặt breakpoint sau vòng lặp nghi ngờ; so sánh bytes trước/sau.

## 9. Best practices

- Coi **allocation trong hot path** (Update, FixedUpdate, network tick) là nợ kỹ thuật.
- Prefer `struct` + generic khi dữ liệu nhỏ, ngắn sống (nhưng tránh copy lớn / boxing).
- Cache `List`/`StringBuilder`; `Clear()` tái sử dụng capacity.
- Tách rõ: memory **managed** (GC) vs **native/unmanaged** (Dispose / Unity Destroy).
- Đo trước khi tối ưu — đừng pool mọi thứ mù quáng.

## 10. Bài tập

**Bài 1** — Vẽ ASCII: biến `Player p = new Player()` sau khi method return (p là local). Object còn sống không? Giải thích.

**Bài 2** — Viết method so sánh allocation: tạo `10000` lần `new string("x")` vs tái sử dụng một `string` literal `"x"`.

**Bài 3** — Giải thích vì sao `List<int>` không boxing từng phần tử, còn `ArrayList` thì có.

**Bài 4** — Dùng `GC.GetAllocatedBytesForCurrentThread()` đo một vòng `for` tạo `new object()` 1000 lần.

## 11. Gợi ý

- Bài 1: local hết scope → không còn root → object **eligible** cho GC (chưa chắc đã collect ngay).
- Bài 2: literal intern/reuse; `new string` buộc object mới.
- Bài 3: `List<int>` store inline ints trong array `int[]`; `ArrayList` store `object` → box.
- Bài 4: bọc đo before/after; tránh `Console.Write` trong đoạn đo.

## 12. Đáp án

**Bài 1** — Khi `Demo()` return, local `p` biến mất. Nếu không ai giữ reference, object trên heap trở thành garbage; GC sẽ thu **sau này**, không phải ngay lúc return.

**Bài 2** — Minh họa:

```csharp
static void CompareStringAlloc()
{
    long a0 = GC.GetAllocatedBytesForCurrentThread();
    for (int i = 0; i < 10_000; i++)
        _ = new string('x', 1);
    long a1 = GC.GetAllocatedBytesForCurrentThread();

    long b0 = GC.GetAllocatedBytesForCurrentThread();
    for (int i = 0; i < 10_000; i++)
        _ = "x";
    long b1 = GC.GetAllocatedBytesForCurrentThread();

    Console.WriteLine($"new string: {a1 - a0}, literal: {b1 - b0}");
}
```

**Bài 3** — `List<T>` với `T=int` dùng mảng `int[]` (value liền). `ArrayList.Add(1)` nhận `object` → boxing từng `int` thành object trên heap.

**Bài 4**:

```csharp
long before = GC.GetAllocatedBytesForCurrentThread();
for (int i = 0; i < 1000; i++)
    _ = new object();
long after = GC.GetAllocatedBytesForCurrentThread();
Console.WriteLine(after - before);
```

## 13. Đáp án thay thế

Bài 2 có thể dùng BenchmarkDotNet (`MemoryDiagnoser`) thay vì `GetAllocatedBytes` — chính xác hơn khi JIT/GC nhiễu. Bài 1 có thể mở rộng: static field giữ `p` → object **không** eligible.

## 14. Thử thách

Viết chương trình in sơ đồ text: sau 3 lần `new Player()`, gán `p1 = p2 = null`, đoán object nào còn sống. Dùng static list giữ một player để minh họa root.

## 15. Ứng dụng thực tế

- Backend: giảm allocation trên request path nóng
- Desktop: tránh leak handle file/DB (unmanaged side)
- Game server C#: entity spam → Gen2 pressure
- Library design: API trả `struct` / `Span` / pool để caller kiểm soát alloc

## 16. Liên hệ Unity

- Mỗi `Update()`: **0 B GC Alloc** là mục tiêu hot path.
- `GetComponent` cache; tránh `Find` + string nối mỗi frame.
- IL2CPP vẫn có **Boehm / incremental GC** (tùy version) — nguyên tắc “ít alloc” vẫn đúng.
- Native memory (mesh, texture) ≠ managed heap nhưng vẫn làm spike hitch khác — đừng nhầm với GC spike.
- Level này là nền cho **Object Pool** (chương 7) chống bullet/VFX spam.

## 17. Kiểm tra kiến thức

1. Ai thu hồi object `new Player()` thông thường?  
   **Đáp án:** Garbage Collector trên managed heap.

2. Biến reference chứa gì?  
   **Đáp án:** Địa chỉ (reference) tới object trên heap, hoặc null.

3. Vì sao `new` mỗi frame nguy hiểm trong Unity?  
   **Đáp án:** Làm đầy Gen0 → GC chạy → GC spike / frame hitch.

4. `GC.Collect()` nên dùng khi nào?  
   **Đáp án:** Chủ yếu diagnostic; tránh thói quen gọi trong gameplay.

5. Boxing tạo object ở đâu?  
   **Đáp án:** Trên managed heap (allocation ẩn).
