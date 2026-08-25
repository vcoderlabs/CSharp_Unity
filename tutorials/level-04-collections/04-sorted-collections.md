# Chương 4 — SortedDictionary và SortedSet

## 1. Mục tiêu học

- Dùng `SortedDictionary<TKey, TValue>` khi cần map **luôn sắp xếp theo key**
- Dùng `SortedSet<T>` khi cần tập hợp **đã sort + unique**
- So sánh với `Dictionary` / `HashSet` về tốc độ và thứ tự
- Biết chi phí O(log n) của cây đỏ-đen

## 2. Điều kiện tiên quyết

- Chương 2: Dictionary, HashSet
- Hiểu so sánh (`IComparable` / `IComparer`)

## 3. Khái niệm

### SortedDictionary\<TKey, TValue\>

- Giống Dictionary về API (key → value) nhưng **duyệt theo thứ tự key tăng dần**
- Cài đặt bằng cây cân bằng → thêm/tìm/xóa **O(log n)**
- Không dựa hash; key phải comparable

### SortedSet\<T\>

- Tập unique **đã sắp xếp**
- `Add`/`Contains`/`Remove`: O(log n)
- Hỗ trợ view theo khoảng: `GetViewBetween`, `Min`, `Max`

| | Dictionary / HashSet | Sorted* |
|---|----------------------|---------|
| Thứ tự duyệt | Không đảm bảo (hash) | Theo key/phần tử tăng |
| Thêm / tìm | O(1) avg | O(log n) |
| Khi chọn | Tra cứu thuần túy | Cần sorted + range query |

> Còn `SortedList<TKey,TValue>`: mảng sort, lookup O(log n), insert O(n). Ít linh hoạt hơn SortedDictionary khi chèn nhiều; tiết kiệm bộ nhớ hơn một chút.

## 4. Mô hình tư duy

```text
Dictionary (hash):   duyệt có thể:  C, A, B
SortedDictionary:    duyệt luôn:    A, B, C

SortedSet scores: { 10, 20, 50, 80 }
Min=10, Max=80
GetViewBetween(20, 50) → { 20, 50 }
```

## 5. Cú pháp

```csharp
var byName = new SortedDictionary<string, int>
{
    ["Zoe"] = 1,
    ["Anna"] = 2,
    ["Mike"] = 3
};
foreach (var kv in byName) // Anna, Mike, Zoe
    Console.WriteLine(kv.Key);

var set = new SortedSet<int> { 5, 1, 3, 3 }; // {1,3,5}
int min = set.Min;
var mid = set.GetViewBetween(2, 4); // {3}

// Comparer tùy chỉnh (giảm dần)
var desc = new SortedSet<int>(Comparer<int>.Create((a, b) => b.CompareTo(a)));
```

## 6. Ví dụ

### Cơ bản

In leaderboard theo tên alphabet:

```csharp
var scores = new SortedDictionary<string, int>();
scores["Bob"] = 1200;
scores["Alice"] = 1500;
scores["Carol"] = 900;

foreach (var (name, score) in scores)
    Console.WriteLine($"{name}: {score}");
```

### Trung cấp

Top điểm bằng SortedSet với comparer tùy chỉnh (điểm giảm, tên tăng khi hòa):

```csharp
record Entry(string Name, int Score);

var comparer = Comparer<Entry>.Create((a, b) =>
{
    int c = b.Score.CompareTo(a.Score);
    return c != 0 ? c : string.Compare(a.Name, b.Name, StringComparison.Ordinal);
});

var board = new SortedSet<Entry>(comparer)
{
    new("Alice", 100),
    new("Bob", 100),
    new("Carol", 120)
};
```

### Nâng cao

Đếm số phần tử trong khoảng trên SortedSet:

```csharp
static int CountInRange(SortedSet<int> set, int lo, int hi)
{
    if (lo > hi) return 0;
    return set.GetViewBetween(lo, hi).Count;
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Chậm hơn Dictionary rõ rệt | Dùng Sorted* khi không cần sort | Quay lại Dictionary |
| Key không comparable | Thiếu `IComparable` / comparer | Truyền `IComparer<TKey>` vào ctor |
| Sửa field ảnh hưởng thứ tự sort | Object mutable trong SortedSet | Dùng immutable / không sửa field so sánh |
| Nhầm SortedList vs SortedDictionary | Insert nhiều trên SortedList O(n) | Chèn nhiều → SortedDictionary |

## 8. Gỡ lỗi

1. In từng key khi duyệt — xác nhận thứ tự.
2. Kiểm tra comparer: viết unit test `Compare(a,b)` và `Compare(b,a)` đối xứng.
3. Đo thời gian với `Stopwatch` nếu nghi chọn sai cấu trúc (chương 7).

## 9. Best practices

- Cần O(1) lookup, không cần thứ tự → `Dictionary`/`HashSet`
- Cần luôn sorted hoặc range → `SortedDictionary`/`SortedSet`
- Sort một lần rồi dùng: có thể `List` + `Sort` + binary search thay vì Sorted* sống lâu
- Comparer phải **ổn định và nhất quán** với Equals (với set)

## 10. Bài tập

**Bài 1 — Word index**  
*Input:* các từ.  
*Output:* in từ theo alphabet kèm số lần xuất hiện (`SortedDictionary<string,int>`).

**Bài 2 — Unique sorted**  
*Input:* `int[]` lộn xộn có trùng.  
*Output:* mảng tăng dần không trùng (`SortedSet`).

**Bài 3 — Floor / Ceiling**  
*Input:* `SortedSet<int>`, giá trị `x`.  
*Output:* phần tử lớn nhất ≤ x và nhỏ nhất ≥ x (nếu có).

**Bài 4 — Merge two sorted maps**  
*Input:* hai `SortedDictionary<string,int>` (cộng value nếu trùng key).  
*Output:* dictionary đã merge, vẫn sorted.

## 11. Gợi ý

- Bài 1: giống word count nhưng dùng SortedDictionary.
- Bài 2: `new SortedSet<int>(arr).ToArray()`.
- Bài 3: `GetViewBetween(set.Min, x).Max` / `GetViewBetween(x, set.Max).Min` — cẩn thận set rỗng / ngoài khoảng.
- Bài 4: copy dict1, duyệt dict2 cộng dồn.

## 12. Đáp án

**Bài 1** — SortedDictionary tự sắp key khi duyệt:

```csharp
static void PrintWordIndex(IEnumerable<string> words)
{
    var map = new SortedDictionary<string, int>(StringComparer.OrdinalIgnoreCase);
    foreach (string w in words)
        map[w] = map.GetValueOrDefault(w) + 1;
    foreach (var kv in map)
        Console.WriteLine($"{kv.Key}: {kv.Value}");
}
```

**Bài 2** — SortedSet vừa unique vừa sort:

```csharp
static int[] UniqueSorted(int[] nums) => new SortedSet<int>(nums).ToArray();
```

**Bài 3** — Dùng view theo khoảng (xử lý biên):

```csharp
static (int? floor, int? ceil) FloorCeil(SortedSet<int> set, int x)
{
    if (set.Count == 0) return (null, null);
    int? floor = set.GetViewBetween(set.Min, x).Count > 0
        ? set.GetViewBetween(set.Min, x).Max
        : null;
    int? ceil = set.GetViewBetween(x, set.Max).Count > 0
        ? set.GetViewBetween(x, set.Max).Min
        : null;
    return (floor, ceil);
}
```

**Bài 4** — Merge cộng value:

```csharp
static SortedDictionary<string, int> Merge(
    SortedDictionary<string, int> a,
    SortedDictionary<string, int> b)
{
    var result = new SortedDictionary<string, int>(a);
    foreach (var kv in b)
        result[kv.Key] = result.GetValueOrDefault(kv.Key) + kv.Value;
    return result;
}
```

## 13. Đáp án thay thế

Bài 2: `HashSet` rồi copy sang mảng + `Array.Sort` — thường **nhanh hơn** SortedSet nếu chỉ cần kết quả cuối, không cần cấu trúc sống sorted.

## 14. Thử thách

Cho stream điểm số online, luôn trả lời “điểm cao nhất hiện tại” và “có bao nhiêu người trong khoảng [L,R]”. Chọn cấu trúc và implement API `Add(score)`, `Max()`, `CountRange(L,R)`.

## 15. Ứng dụng thực tế

- Leaderboard theo tên / theo rank
- Index từ điển, autocomplete prefix (kết hợp cấu trúc khác)
- Time-series key sorted (timestamp → event) — cân nhắc thêm thư viện chuyên dụng

## 16. Liên hệ Unity

- Ít dùng Sorted* trong hot path gameplay (O(log n) + allocation)
- Phù hợp tool/editor, bảng xếp hạng UI cập nhật thưa
- Runtime combat: thường `List` sort khi cần hiển thị, hoặc giữ top-N thủ công

## 17. Kiểm tra kiến thức

1. SortedDictionary thêm phần tử độ phức tạp?  
   **Đáp án:** O(log n).

2. Khi nào chọn Dictionary thay SortedDictionary?  
   **Đáp án:** Chỉ cần tra cứu nhanh, không cần thứ tự key.

3. SortedSet khác HashSet?  
   **Đáp án:** Có thứ tự + O(log n); HashSet O(1) avg, không sort.

4. `GetViewBetween` dùng để làm gì?  
   **Đáp án:** Lấy tập con theo khoảng giá trị.

5. Key trong SortedDictionary cần gì?  
   **Đáp án:** So sánh được (`IComparable` hoặc `IComparer`).
