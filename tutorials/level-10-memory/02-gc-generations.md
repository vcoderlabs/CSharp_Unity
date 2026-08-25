# Chương 2 — GC Generations (Gen0 / Gen1 / Gen2) & LOH

## 1. Mục tiêu học

- Hiểu **generational GC**: Gen0, Gen1, Gen2
- Biết **LOH (Large Object Heap)** và ngưỡng ~85 KB
- Phân biệt collection “rẻ” vs “đắt” (full GC)
- Liên hệ trực tiếp **Unity GC spikes** khi Gen0 đầy vì spam `new`

## 2. Điều kiện tiên quyết

- Chương 1: CLR memory model, managed heap
- Level 3: reference type sống trên heap

## 3. Khái niệm

### Generational hypothesis

Hầu hết object **chết trẻ**. GC tối ưu: quét vùng “mới sinh” (Gen0) thường xuyên hơn vùng “già”.

| Thế hệ | Ý nghĩa | Chi phí collect |
|--------|---------|-----------------|
| **Gen0** | Object mới / ngắn sống | Rẻ, thường xuyên |
| **Gen1** | Sống sót 1 lần Gen0 | Trung bình |
| **Gen2** | Sống lâu (cache, singleton, static) | Đắt — full GC hay quét rộng |

### Luồng promote

```text
new object ──► Gen0
                 │ sống sót GC Gen0
                 ▼
               Gen1
                 │ sống sót tiếp
                 ▼
               Gen2  (càng lâu càng “đắt” khi phải full collect)
```

### LOH — Large Object Heap

- Object **lớn** (thường **≥ 85,000 bytes**) vào **LOH**
- Ví dụ: `new byte[100_000]`, mảng lớn, string khổng lồ
- LOH **không compact thường xuyên** như ephemeral heap (tùy runtime/setting) → dễ **phân mảnh**
- Collect LOH gắn với Gen2 → **đắt**

### Workstation vs Server GC (console/server .NET)

- **Workstation**: latency tốt hơn cho UI/desktop (và gần với mindset game)
- **Server**: throughput, nhiều heap/heap lớn hơn
- Unity có GC riêng (incremental options) — nguyên tắc generations vẫn hữu ích để suy nghĩ allocation

### GC modes liên quan spike

- **Blocking / stop-the-world**: mọi managed thread tạm dừng → frame spike
- **Incremental / concurrent** (Unity có tùy chọn): rải việc thu gom — vẫn cần **giảm alloc**

## 4. Mô hình tư duy

```text
MANAGED HEAP (rút gọn)
┌────────────────────────────────────────────┐
│ GEN0  ░░░░░░░  ← new liên tục trong Update │  collect thường
│       ░ dead ░  → thu hồi nhanh            │
├────────────────────────────────────────────┤
│ GEN1  ▒▒▒                                  │
├────────────────────────────────────────────┤
│ GEN2  ████ Manager, static cache, singletons│  collect đắt
├────────────────────────────────────────────┤
│ LOH   ████████ large byte[] / buffers      │  ~ Gen2 cost
└────────────────────────────────────────────┘

Unity frame:
  Alloc nhiều ─► Gen0 đầy ─► GC ─► pause ─► SPIKE (giật)
```

**Quy tắc vàng:** Object **ngắn sống + nhiều** → Gen0 OK nếu vừa phải. Object **trung bình sống + cực nhiều/frame** → Gen0 thrash. Object **lớn tạm thời** → LOH + Gen2 pain.

## 5. Cú pháp

```csharp
// Thông tin thế hệ
int gen = GC.GetGeneration(myObject);
Console.WriteLine(gen); // 0, 1, hoặc 2

// Ép collect (CHỈ để học / đo — đừng spam trong game)
GC.Collect(0);                 // cố Gen0
GC.Collect(2, GCCollectionMode.Forced, blocking: true);
GC.WaitForPendingFinalizers();

// Memory pressure hint (hiếm khi cần tay)
GC.AddMemoryPressure(10_000);
GC.RemoveMemoryPressure(10_000);

// .NET: cấu hình GC qua runtimeconfig / env (Server GC, etc.)
// Unity: Project Settings → Player → GC (Incremental)
```

## 6. Ví dụ

### Cơ bản — Object chết ở Gen0

```csharp
static void ShortLived()
{
    for (int i = 0; i < 10_000; i++)
    {
        var tmp = new byte[64]; // chết ngay khi hết vòng
        _ = tmp.Length;
    }
    // Phần lớn được thu ở Gen0 — vẫn tốn CPU GC nếu quá nhiều
}
```

### Trung cấp — Promote lên Gen2

```csharp
static object? Cache;

static void PromoteDemo()
{
    Cache = new byte[1024];
    Console.WriteLine($"After new: gen={GC.GetGeneration(Cache)}");

    GC.Collect(0);
    Console.WriteLine($"After Gen0: gen={GC.GetGeneration(Cache!)}");

    GC.Collect(1);
    Console.WriteLine($"After Gen1: gen={GC.GetGeneration(Cache!)}");

    GC.Collect(2);
    Console.WriteLine($"After Gen2: gen={GC.GetGeneration(Cache!)}");
}
```

### Nâng cao — LOH allocation

```csharp
static void LohDemo()
{
    // >= 85_000 bytes → LOH (ngưỡng có thể khác theo runtime, coi ~85KB)
    var big = new byte[90_000];
    Console.WriteLine($"gen={GC.GetGeneration(big)}");
    // big sống lâu + full GC → đắt; reuse buffer / ArrayPool tốt hơn
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Gọi `GC.Collect()` mỗi frame “cho sạch” | Tự tạo spike | Xóa; giảm alloc |
| Cache mọi thứ vào static | Đẩy Gen2, tăng full GC | Cache có giới hạn / pool |
| `new byte[1<<20]` tạm mỗi request | LOH + fragmentation | `ArrayPool<byte>.Shared` |
| Nghĩ Gen0 collect “miễn phí” | Vẫn pause / CPU | Giảm tần suất alloc |
| Đo sai vì debug alloc | Debugger/IDE nhiễu | Release + profiler |

## 8. Gỡ lỗi

1. In `GC.CollectionCount(0/1/2)` trước/sau scene nặng — số Gen2 tăng = đỏ.
2. Unity Profiler: **GC.Collect** sample trên main thread = spike.
3. So sánh hai build: có/không string concat trong `Update`.
4. Tìm “survived to Gen2” bằng Memory Profiler (objects sống qua nhiều collect).

```csharp
Console.WriteLine(
    $"G0={GC.CollectionCount(0)} G1={GC.CollectionCount(1)} G2={GC.CollectionCount(2)}");
```

## 9. Best practices

- Hot path: **zero / near-zero** allocation.
- Object sống cả game (managers): chấp nhận Gen2 — nhưng **ít** và ổn định.
- Buffer lớn: **pool** (`ArrayPool`, object pool) thay vì alloc/free liên tục trên LOH.
- Tránh giữ reference vô tình (event, static list) khiến object “trẻ” promote lên Gen2.
- Prefill pool lúc load scene — không alloc giữa combat.

## 10. Bài tập

**Bài 1** — Viết vòng lặp tạo `new object()` 50_000 lần; in `CollectionCount` Gen0 trước/sau (không gọi `GC.Collect` tay).

**Bài 2** — Giữ một object trong static; gọi `GC.Collect(0)` rồi `GC.Collect(1)`; in generation sau mỗi bước.

**Bài 3** — So sánh thời gian/`CollectionCount` khi alloc nhiều `byte[64]` vs ít lần `byte[200_000]`.

**Bài 4** — Giải thích bằng lời: vì sao bullet `new` mỗi shot tệ hơn một `List<Bullet>` tái sử dụng slot.

## 11. Gợi ý

- Bài 1: chỉ đọc counter; vòng lặp đủ lớn để Gen0 collect tự chạy.
- Bài 2: static root → object survive → generation tăng.
- Bài 3: LOH đi Gen2 path; nhiều small → Gen0.
- Bài 4: pool/reuse → ít Gen0 pressure → ít spike.

## 12. Đáp án

**Bài 1**:

```csharp
int g0 = GC.CollectionCount(0);
for (int i = 0; i < 50_000; i++)
    _ = new object();
Console.WriteLine($"Gen0 collections delta: {GC.CollectionCount(0) - g0}");
```

**Bài 2**:

```csharp
static object? Hold;
Hold = new object();
Console.WriteLine(GC.GetGeneration(Hold));
GC.Collect(0);
Console.WriteLine(GC.GetGeneration(Hold!));
GC.Collect(1);
Console.WriteLine(GC.GetGeneration(Hold!));
```

**Bài 3** — Small: nhiều Gen0. Large (~200KB): LOH, ít alloc hơn nhưng mỗi collect liên quan Gen2 đắt hơn / dễ fragment. Kết luận phụ thuộc pattern reuse.

**Bài 4** — Mỗi `new Bullet` = Gen0 alloc. Combat 100 bullet/s × nhiều phút → GC thường xuyên → spike. Pool lấy/trả instance sẵn có → gần 0 alloc lúc bắn.

## 13. Đáp án thay thế

Dùng `ArrayPool<byte>.Shared.Rent(90_000)` / `Return` thay LOH tạm. Trong Unity dùng `UnityEngine.Pool.ObjectPool<T>` (chương 7).

## 14. Thử thách

Viết mini report: chạy 3 scenario (short-lived spam, static promote, LOH), ghi bảng `CollectionCount` + `GetTotalMemory`. Đưa ra khuyến nghị cho “skill VFX spawn”.

## 15. Ứng dụng thực tế

- High-frequency trading / game server: Gen2 pause = SLA break
- ASP.NET: pooling `byte[]` cho Kestrel
- Mobile: full GC = thermal + frame drop
- Editor tools: chấp nhận alloc; runtime player thì không

## 16. Liên hệ Unity

- **GC spike** ≈ main thread dừng trong `GC.Collect` (hoặc incremental slice lớn).
- Combat log string, LINQ trong `Update`, boxing damage number → Gen0 thrash.
- Large mesh/array managed tạm → LOH — prefer native / pooled buffers.
- Bật **Incremental GC** giảm độ cao spike nhưng **không thay** việc giảm allocation.
- Object pooling (chương 7 + project) là vũ khí số 1 chống Gen0 trong gameplay.

## 17. Kiểm tra kiến thức

1. Object mới thường vào generation nào?  
   **Đáp án:** Gen0.

2. Vì sao Gen2 collect đắt?  
   **Đáp án:** Quét vùng lớn / full GC; object sống lâu nhiều hơn.

3. LOH roughly khi nào?  
   **Đáp án:** Object lớn (~≥ 85KB).

4. `GC.Collect()` mỗi frame trong Unity thì sao?  
   **Đáp án:** Tự gây spike / hitch.

5. Incremental GC có thay thế pool không?  
   **Đáp án:** Không — chỉ làm mượt hơn; vẫn phải giảm alloc.
