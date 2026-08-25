# Chương 7 — Performance và Big O của Collections

## 1. Mục tiêu học

- Đọc và dùng bảng Big O cho thao tác phổ biến trên từng collection
- Cân nhắc thêm chi phí bộ nhớ / GC, không chỉ thời gian
- Ra quyết định “chọn cấu trúc nào” theo use-case
- Đo nhanh bằng `Stopwatch` khi nghi ngờ

## 2. Điều kiện tiên quyết

- Chương 1–6: đã dùng các collection chính
- Level 0: trực giác Big O (O(1), O(n), O(log n))

## 3. Khái niệm

**Big O** mô tả tốc độ tăng chi phí khi dữ liệu lớn dần (n = số phần tử). Không phải số millisecond tuyệt đối.

Ký hiệu thường gặp:

| Ký hiệu | Ý nghĩa trực giác |
|---------|-------------------|
| O(1) | Không phụ thuộc n (hoặc trung bình gần như vậy) |
| O(log n) | Tăng chậm (cây cân bằng) |
| O(n) | Tỷ lệ thuận n (duyệt / dịch chuyển) |
| O(n log n) | Sort điển hình |
| Amortized O(1) | Trung bình rẻ; đôi khi một lần đắt (List resize) |

**Bộ nhớ:** Dictionary/HashSet tốn hơn List vì bucket + entry. Sorted* tốn node cây.

## 4. Mô hình tư duy

```text
Câu hỏi chọn collection:
1. Cần tra cứu theo key/id?     → Dictionary / HashSet
2. Cần thứ tự chèn + index?     → List / Array
3. FIFO / LIFO?                 → Queue / Stack
4. Luôn sorted / range?         → Sorted*
5. Chỉ duyệt một lần?           → IEnumerable / mảng

Đo khi: n lớn (10^5–10^7) hoặc hot path (mỗi frame / mỗi request).
```

## 5. Cú pháp (đo nhanh)

```csharp
using System.Diagnostics;

var sw = Stopwatch.StartNew();
// ... thao tác ...
sw.Stop();
Console.WriteLine($"{sw.ElapsedMilliseconds} ms ({sw.Elapsed.TotalMicroseconds} µs)");
```

Warm-up 1 lần trước khi đo (JIT); đo nhiều lần lấy trung bình nếu nghiêm túc.

## 6. Ví dụ

### Cơ bản — Bảng Big O (trung bình / điển hình)

| Thao tác | Array | List\<T\> | Dictionary | HashSet | Queue/Stack | SortedDict/Set | LinkedList\* |
|----------|-------|-----------|------------|---------|-------------|----------------|--------------|
| Access by index | O(1) | O(1) | — | — | — | — | O(n) |
| Search by value | O(n) | O(n) | — | O(1) | O(n) | O(log n) | O(n) |
| Lookup by key | — | — | O(1) | — | — | O(log n) | — |
| Add cuối / Push/Enqueue | — | O(1)* | O(1) | O(1) | O(1) | O(log n) | O(1) đầu/cuối |
| Insert đầu / giữa | O(n) | O(n) | — | — | — | — | O(1) nếu có node |
| Remove by key/value | O(n) | O(n) | O(1) | O(1) | — | O(log n) | O(n) tìm + O(1) xóa node |
| Duyệt toàn bộ | O(n) | O(n) | O(n) | O(n) | O(n) | O(n) | O(n) |

\*List Add: amortized O(1). Dictionary/HashSet: average O(1), worst O(n) nếu hash xấu.  
\*LinkedList: xóa O(1) **chỉ khi đã có `LinkedListNode`**.

### Trung cấp — Case study chọn sai

```csharp
// SAI: tìm player theo id trong List mỗi lần → O(n)
Player? Find(List<Player> all, int id)
{
    foreach (var p in all)
        if (p.Id == id) return p;
    return null;
}

// ĐÚNG: index hóa một lần
Dictionary<int, Player> byId = ...;
byId.TryGetValue(id, out var player);
```

### Nâng cao — Đo List.Contains vs HashSet.Contains

```csharp
const int N = 100_000;
var list = Enumerable.Range(0, N).ToList();
var set = new HashSet<int>(list);
int target = N - 1;

var sw = Stopwatch.StartNew();
bool a = list.Contains(target);
sw.Stop();
Console.WriteLine($"List: {sw.Elapsed.TotalMilliseconds} ms");

sw.Restart();
bool b = set.Contains(target);
sw.Stop();
Console.WriteLine($"HashSet: {sw.Elapsed.TotalMilliseconds} ms");
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| “Dictionary chậm hơn List” trên n nhỏ | Hằng số hash + overhead | N nhỏ: List có thể thắng; chọn theo n thực tế |
| Quên amortized | Một lần Add đắt khi resize | `EnsureCapacity` |
| Chỉ nhìn CPU, quên GC | `new` Dictionary mỗi frame | Tái sử dụng / pool / Clear |
| Sort mỗi lần lookup | O(n log n) lặp lại | Sort một lần hoặc Sorted* / Dictionary |

## 8. Gỡ lỗi (performance)

1. Xác định thao tác hot: Add? Lookup? Insert đầu?
2. Ước n tối đa (100? 100k? 10M?).
3. Chọn cấu trúc theo bảng; viết micro-benchmark nếu cần.
4. Profile allocation (dotMemory / `dotNet-trace` / Unity Profiler) — không chỉ Stopwatch.

## 9. Best practices

- Tối ưu theo **đo lường** và use-case, không theo cảm tính.
- Default lành mạnh: `List` cho dãy; `Dictionary` cho id→entity; `HashSet` cho membership.
- Pre-size capacity khi biết n.
- Hot path Unity: tránh allocation; cân nhắc array cố định / NativeArray (DOTS) sau này.
- Document giả định: “n &lt; 100 nên linear OK”.

## 10. Bài tập

**Bài 1** — Với yêu cầu “kiểm tra userId đã online chưa, n≈50_000, rất nhiều Contains”, chọn cấu trúc và giải thích Big O.

**Bài 2** — “Danh sách skill theo thứ tự cố định, truy cập skill[i], ít khi thêm giữa”, chọn Array hay LinkedList? Vì sao?

**Bài 3** — Viết đoạn đo: thêm 100_000 phần tử vào `List` không capacity vs `List(capacity)`.

**Bài 4** — Cho API: thêm log theo thời gian, thỉnh thoảng lấy mọi log trong khoảng thời gian [t1,t2]. Gợi ý cấu trúc (có thể kết hợp).

## 11. Gợi ý

- Bài 1: HashSet&lt;int&gt; hoặc Dictionary — Contains O(1) avg.
- Bài 2: Array hoặc List — index O(1); LinkedList index O(n).
- Bài 3: Stopwatch + hai list.
- Bài 4: `SortedDictionary&lt;DateTime, List&lt;Log&gt;&gt;` hoặc List sort + binary search; với volume lớn cần thiết kế kỹ hơn.

## 12. Đáp án

**Bài 1** — Giải thích: membership thuần túy → HashSet:

```csharp
var online = new HashSet<int>(capacity: 50_000);
online.Add(userId);
bool isOnline = online.Contains(userId); // O(1) average
```

**Bài 2** — Array/List: cần index O(1). LinkedList tệ cho `skill[i]`.

**Bài 3** — Đo capacity:

```csharp
static void BenchListCapacity()
{
    const int N = 100_000;
    var sw = Stopwatch.StartNew();
    var a = new List<int>();
    for (int i = 0; i < N; i++) a.Add(i);
    sw.Stop();
    Console.WriteLine($"No cap: {sw.Elapsed.TotalMilliseconds} ms");

    sw.Restart();
    var b = new List<int>(N);
    for (int i = 0; i < N; i++) b.Add(i);
    sw.Stop();
    Console.WriteLine($"With cap: {sw.Elapsed.TotalMilliseconds} ms");
}
```

**Bài 4** — Hướng thiết kế: lưu `List<(DateTime t, string msg)>` append O(1); khi query, nếu list đã sort theo thời gian → binary search biên; hoặc `SortedDictionary&lt;DateTime, List&lt;string&gt;&gt;` cho insert O(log n) + lấy khoảng bằng view/keys.

## 13. Đáp án thay thế

Bài 1 dùng `Dictionary<int, byte>` hoặc `ConcurrentDictionary` nếu đa luồng (Level 11). Bài 4 với dữ liệu cực lớn: database / time-series store thay vì in-memory thuần.

## 14. Thử thách

Tự lập bảng đo thực nghiệm trên máy bạn cho n = 10³, 10⁵, 10⁶: List.Contains vs HashSet.Contains vs Dictionary.ContainsKey. Xuất CSV kết quả.

## 15. Ứng dụng thực tế

- API gateway rate-limit set
- Game server entity index
- ETL: chọn cấu trúc ảnh hưởng hàng giờ chạy batch
- Tránh regression: PR review hỏi “đây là O gì khi n tăng?”

## 16. Liên hệ Unity

- `Update` 60fps: O(n²) với n = số enemy dễ drop FPS
- GC spike: tạo List/Dictionary tạm mỗi frame nguy hiểm hơn vài µs CPU
- Unity Profiler: CPU + Memory + GC Alloc columns
- Spatial partition (grid/hash) = “Dictionary hóa” không gian — cùng tư duy Level 4

## 17. Kiểm tra kiến thức

1. List truy cập index?  
   **Đáp án:** O(1).

2. Dictionary lookup trung bình?  
   **Đáp án:** O(1).

3. SortedDictionary Add?  
   **Đáp án:** O(log n).

4. Insert đầu List?  
   **Đáp án:** O(n) — dịch phần tử.

5. Amortized O(1) của List.Add nghĩa là gì?  
   **Đáp án:** Hầu hết lần Add O(1); thỉnh thoảng resize O(n), trung bình vẫn O(1).
