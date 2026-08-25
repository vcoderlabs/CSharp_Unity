# Chương 1 — Allocation & GC pressure

## 1. Mục tiêu học

- Hiểu **allocation** (cấp phát heap) tốn gì
- Phân biệt chi phí tạo object với **GC pressure** (áp lực thu gom)
- Nhận diện hot path tạo nhiều Gen0
- Đo sơ bộ bằng `GC.GetTotalMemory` / `GC.CollectionCount`

## 2. Điều kiện tiên quyết

- Level 10: heap, Gen0/1/2, LOH
- Biết class vs struct (Level 3)
- Console app .NET 6+

## 3. Khái niệm

Mỗi lần `new` reference type (hoặc boxing, mảng lớn, closure, …) → CLR cấp phát trên **managed heap**.

| Thuật ngữ | Ý nghĩa |
|-----------|---------|
| Allocation | Việc xin chỗ trên heap |
| GC pressure | Tần suất / khối lượng GC phải chạy vì heap đầy nhanh |
| Hot path | Code chạy rất thường (mỗi request, mỗi frame, mỗi tick) |

**Allocation rẻ từng cái nhỏ** nhưng **hàng triệu lần/giây** → Gen0 đầy → GC pause → latency/jank.

GC pressure cao ≠ chỉ vì object “to”. Nhiều object nhỏ sống ngắn cũng gây pressure (Gen0 thrashing).

## 4. Mô hình tư duy

```text
Hot path mỗi giây:
  new A(); new B(); new byte[64]; ...  × N

→ Heap Gen0 đầy nhanh
→ GC Gen0 (đôi khi promote → Gen1/2)
→ Pause ngắn lặp lại = “giật” hoặc tail latency cao

Giảm pressure:
  1) Ít new hơn trên hot path
  2) Tái sử dụng (pool, buffer)
  3) Stackalloc / Span / struct khi hợp lý
  4) Tránh LOH churn (mảng ≥ ~85KB)
```

## 5. Cú pháp

```csharp
long before = GC.GetTotalMemory(forceFullCollection: false);
// ... code cần đo ...
long after = GC.GetTotalMemory(false);
Console.WriteLine($"Δ bytes (ước lượng): {after - before}");

int gen0 = GC.CollectionCount(0);
int gen1 = GC.CollectionCount(1);
int gen2 = GC.CollectionCount(2);
```

`forceFullCollection: true` chỉ dùng khi đo “sạch” — đừng gọi trong production hot path.

## 6. Ví dụ

### Cơ bản — allocation rõ ràng

```csharp
static List<string> BuildBad(int n)
{
    var list = new List<string>();
    for (int i = 0; i < n; i++)
        list.Add(i.ToString()); // mỗi lần: string mới trên heap
    return list;
}
```

### Trung cấp — ẩn allocation

```csharp
static int SumLengths(IEnumerable<string> items)
{
    // foreach trên IEnumerable có thể allocate enumerator (tùy implementation)
    int sum = 0;
    foreach (var s in items)
        sum += s.Length;
    return sum;
}

static int SumLengthsArray(string[] items)
{
    int sum = 0;
    for (int i = 0; i < items.Length; i++) // không enumerator heap
        sum += items[i].Length;
    return sum;
}
```

### Nâng cao — so Gen0 trước/sau

```csharp
static void Measure(Action action, string label)
{
    GC.Collect();
    GC.WaitForPendingFinalizers();
    GC.Collect();

    int g0 = GC.CollectionCount(0);
    long mem = GC.GetTotalMemory(false);
    action();
    Console.WriteLine($"{label}: ΔGen0={GC.CollectionCount(0) - g0}, mem≈{GC.GetTotalMemory(false) - mem}");
}

// Measure(() => { for (int i = 0; i < 100_000; i++) _ = i.ToString(); }, "ToString loop");
```

Số liệu thô — dùng BenchmarkDotNet ở chương 4 để chính xác hơn.

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| “Tôi không `new` mà vẫn alloc” | Boxing, LINQ, string concat, lambda capture, params array | Profiler / Alloc view |
| Đo memory nhảy loạn | GC xen giữa, thread khác | Warm-up + BenchmarkDotNet |
| Tối ưu chỗ lạnh | Đo sai hot path | Profile theo % CPU / alloc rate |
| `GC.Collect()` khắp nơi | Ép pause, che vấn đề thật | Chỉ đo / shutdown đặc biệt |

## 8. Gỡ lỗi

1. Visual Studio / Rider: **Memory Usage** / Allocation tracking khi chạy scenario.
2. `dotnet-trace` / `dotnet-gcdump` trên server.
3. Đặt breakpoint sau vòng lặp lớn — xem `GC.CollectionCount(0)` tăng bao nhiêu.
4. Hỏi: object này **sống bao lâu**? Ngắn → Gen0; lâu → có thể lên Gen2 (đắt hơn khi collect).

## 9. Best practices

- Hot path: zero / near-zero allocation là mục tiêu khi đã đo thấy vấn đề.
- Prefill `List<T>(capacity)`, `Dictionary` capacity ước lượng.
- Tránh allocate trong vòng lặp chặt; tái sử dụng buffer.
- LOH: tái sử dụng mảng lớn hoặc ArrayPool.
- Document “allocation-free zone” trong code review khi cần.

## 10. Bài tập

**Bài 1** — Viết 2 hàm nối N số thành một chuỗi: (A) `+=` trong vòng lặp, (B) `StringBuilder`. Đo `GC.CollectionCount(0)` với N = 50_000.

**Bài 2** — So `new List<int>()` rồi `Add` 10_000 phần tử vs `new List<int>(10_000)`.

**Bài 3** — Hàm tạo `byte[100_000]` mỗi lần gọi vs lấy từ `ArrayPool<byte>.Shared` — so Gen0/LOH cảm nhận (in CollectionCount).

**Bài 4** — Liệt kê 5 chỗ trong code cũ của bạn (hoặc giả định game loop) có thể allocate mỗi frame/request.

## 11. Gợi ý

- Bài 1: `string` bất biến → `+=` tạo string mới mỗi lần.
- Bài 2: thiếu capacity → resize mảng nội bộ nhiều lần.
- Bài 3: `ArrayPool.Shared.Rent` / `Return`; nhớ clear nếu chứa dữ liệu nhạy cảm.
- Bài 4: `GetComponent`, LINQ trong Update, `$"..."`, boxing enum, `new WaitForSeconds` (Unity — xem L21).

## 12. Đáp án

**Bài 1** — Ý tưởng:

```csharp
static string ConcatPlus(int n)
{
    string s = "";
    for (int i = 0; i < n; i++) s += i;
    return s;
}

static string ConcatSb(int n)
{
    var sb = new System.Text.StringBuilder(n * 2);
    for (int i = 0; i < n; i++) sb.Append(i);
    return sb.ToString();
}
```

`ConcatPlus` gây nhiều Gen0 hơn rõ rệt.

**Bài 2** — `new List<int>(10_000)` giảm số lần grow (thường ~2^k).

**Bài 3**:

```csharp
var pool = System.Buffers.ArrayPool<byte>.Shared;
byte[] buf = pool.Rent(100_000);
try { /* dùng buf, chú ý Length >= size thuê */ }
finally { pool.Return(buf); }
```

**Bài 4** — Ví dụ: tạo DTO mỗi request không cần; `ToList()` thừa; log string nặng; exception dùng để control flow; capture loop variable trong lambda tạo closure.

## 13. Đáp án thay thế

Dùng `string.Create` / `IBufferWriter` / `RecyclableMemoryStream` cho pipeline IO. Đo bằng BenchmarkDotNet `MemoryDiagnoser` thay vì `GC.CollectionCount`.

## 14. Thử thách

Viết parser dòng CSV đơn giản **allocation-friendly**: nhận `ReadOnlySpan<char>`, tránh `Split` tạo mảng string; so benchmark với `line.Split(',')`.

## 15. Ứng dụng thực tế

- API gateway: giảm alloc/request → throughput ↑, GC pause ↓
- Game server tick: allocation/tick quyết định ổn định
- Mobile/.NET MAUI: GC ảnh hưởng pin & jank
- Batch job: LOH churn khi xử lý buffer lớn

## 16. Liên hệ Unity

- Mỗi `Update` allocate → GC spike → frame drop
- Ưu tiên: không `new` trong Update; pool đạn/VFX; cache component reference
- Unity Profiler: **CPU Usage** + **GC Alloc** cột
- Player loop = hot path cực đoan — Level 21 đào sâu

## 17. Kiểm tra kiến thức

1. Allocation là gì?  
   **Đáp án:** Cấp phát bộ nhớ (thường trên managed heap) cho object/mảng mới.

2. GC pressure cao nghĩa là gì?  
   **Đáp án:** Heap đầy nhanh khiến GC chạy thường xuyên / nặng hơn.

3. Vì sao nhiều object nhỏ cũng nguy hiểm trên hot path?  
   **Đáp án:** Gen0 đầy liên tục → pause lặp lại dù từng object “rẻ”.

4. `GC.Collect()` trong production hot path nên không?  
   **Đáp án:** Không — gây pause nhân tạo, che vấn đề allocation thật.

5. Cách giảm pressure phổ biến?  
   **Đáp án:** Ít allocate hơn, tái sử dụng (pool/buffer), chọn struct/Span khi phù hợp, đo trước khi sửa.
