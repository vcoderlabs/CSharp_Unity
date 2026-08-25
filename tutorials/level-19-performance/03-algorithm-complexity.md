# Chương 3 — Algorithm complexity thực chiến

## 1. Mục tiêu học

- Ôn Big-O và gắn với **cấu trúc dữ liệu C#**
- Chọn `List` / `Dictionary` / `HashSet` / sort đúng bài toán
- Nhận “O(n²) ẩn” trong nested loop + lookup chậm
- Cân bằng complexity lý thuyết vs constant factor / allocation

## 2. Điều kiện tiên quyết

- Level 0/4: Big-O trực giác, collections
- Level 19 chương 1–2: đo allocation

## 3. Khái niệm

**Complexity** mô tả tăng chi phí khi input lớn lên — không phải thời gian tuyệt đối trên máy bạn.

| Ký hiệu | Cảm nhận | Ví dụ |
|---------|----------|--------|
| O(1) | Gần như không đổi | `dict[key]`, index mảng |
| O(log n) | Tăng chậm | Binary search trên sorted |
| O(n) | Tỷ lệ thuận | Duyệt list |
| O(n log n) | Sort điển hình | `List.Sort` |
| O(n²) | Nested duyệt | Mỗi phần tử so mọi phần tử |

Thực chiến: O(n) với constant lớn + alloc có thể thua O(n log n) trên n nhỏ. **Đo với kích thước thật.**

## 4. Mô hình tư duy

```text
Bài toán: "Player có item Id X không?"

Sai:  foreach inventory  → O(n) mỗi lần check; 1000 check = O(n·k)
Tốt:  HashSet<ItemId> / Dictionary  → O(1) trung bình mỗi check

Bài toán: top 10 damage trong 100000 events
Sai:  Sort toàn bộ mỗi lần UI refresh → O(n log n) lặp
Tốt:  giữ heap/partial sort / cập nhật incremental
```

## 5. Cú pháp

```csharp
var list = new List<Player>(capacity);
var map = new Dictionary<Guid, Player>();
var set = new HashSet<int>();

list.Sort((a, b) => b.Score.CompareTo(a.Score)); // O(n log n)
bool found = set.Contains(id); // O(1) avg
map.TryGetValue(id, out var p);
```

## 6. Ví dụ

### Cơ bản — lookup

```csharp
// O(n)
static bool HasItem(List<Item> items, int id)
{
    for (int i = 0; i < items.Count; i++)
        if (items[i].Id == id) return true;
    return false;
}

// O(1) avg sau khi build set O(n)
static bool HasItem(HashSet<int> ids, int id) => ids.Contains(id);
```

### Trung cấp — nested ẩn

```csharp
// O(n·m) — với mỗi order, tìm user bằng list
foreach (var order in orders)
{
    var user = users.First(u => u.Id == order.UserId); // O(m) mỗi lần
}

// O(n + m): dictionary trước
var byId = users.ToDictionary(u => u.Id);
foreach (var order in orders)
{
    var user = byId[order.UserId];
}
```

### Nâng cao — chọn cấu trúc

```csharp
// Cần: thêm/xóa đầu thường xuyên? → LinkedList (cẩn thận cache miss) hoặc Queue
// Cần: random access? → List
// Cần: ưu tiên theo điểm? → PriorityQueue<TElement, TPriority> (.NET)
// Cần: khoảng sorted? → sort + binary search / SortedDictionary
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Chậm dần khi data lớn | O(n²) không nhận ra | Profile + đếm nested |
| `List.Contains` nóng | O(n) | `HashSet` |
| Sort mỗi frame | Quá tay | Cache thứ tự / dirty flag |
| Dictionary key xấu | Hash distribution / alloc key string | Key struct ổn định |

## 8. Gỡ lỗi

1. Tăng N ×10: thời gian ×10 → nghi O(n); ×100 → nghi O(n²).
2. Breakpoint đếm số lần so sánh / lookup.
3. Complexity trên giấy trước khi code tối ưu nhỏ.

## 9. Best practices

- Model dữ liệu theo **câu hỏi thường gặp** (lookup by id? range? top-k?).
- Prefill capacity khi biết n.
- Đừng thay O(n) bằng cấu trúc phức tạp khi n < 20 trừ khi hot cực đoan.
- Ghi chú trong PR: “trước O(n²) trên 5k phần tử, sau O(n)”.
- Concurrent: `ConcurrentDictionary` ≠ miễn phí — biết trade-off.

## 10. Bài tập

**Bài 1** — Cho N player, mỗi player M skill id. Kiểm tra “có ai có skill X?” — so naive vs inverted index `Dictionary<skillId, List<playerId>>`.

**Bài 2** — Đếm tần suất từ trong list string: `Dictionary<string, int>`.

**Bài 3** — Tìm giao hai list id (size lớn): `HashSet` + duyệt list kia.

**Bài 4** — Giải thích complexity `List.Insert(0, x)` vs `Add`.

## 11. Gợi ý

- Bài 1: đảo chiều map skill → players.
- Bài 3: put list ngắn hơn vào set nếu lệch size.
- Bài 4: Insert đầu = O(n) shift; Add amortised O(1).

## 12. Đáp án

**Bài 2**:

```csharp
static Dictionary<string, int> CountWords(IEnumerable<string> words)
{
    var map = new Dictionary<string, int>();
    foreach (var w in words)
    {
        if (map.TryGetValue(w, out int c)) map[w] = c + 1;
        else map[w] = 1;
    }
    return map;
}
```

**Bài 3**:

```csharp
static List<int> Intersect(List<int> a, List<int> b)
{
    var set = new HashSet<int>(a);
    var result = new List<int>();
    foreach (var x in b)
        if (set.Remove(x)) result.Add(x); // Remove để unique
    return result;
}
```

**Bài 4** — Insert(0): O(n); Add cuối: amortised O(1).

**Bài 1** — Build `Dictionary<int, List<int>> skillToPlayers` một lần O(total skills), query O(1) + liệt kê.

## 13. Đáp án thay thế

Dùng `Span` sort / `CollectionsMarshal.AsSpan` cho sort in-place không copy. Top-k: `PriorityQueue` thay full sort.

## 14. Thử thách

Sim combat 10k entity: mỗi tick tìm target gần nhất O(n²). Đề xuất spatial hash / grid và ước lượng complexity sau khi lưới hóa.

## 15. Ứng dụng thực tế

- Inventory, quest flag, cooldown map → Dictionary/HashSet
- Leaderboard → sort định kỳ hoặc structure thứ tự
- Pathfinding: complexity heuristic quyết định server CPU

## 16. Liên hệ Unity

- `FindObjectsOfType` mỗi frame = O(scene) thảm họa
- Physics overlap có thể đắt — cache query
- UI rebuild full list thay vì dirty item
- ECS (DOTS): data layout + job — complexity + cache locality

## 17. Kiểm tra kiến thức

1. `Dictionary` lookup trung bình?  
   **Đáp án:** O(1) amortized average.

2. `List.Contains`?  
   **Đáp án:** O(n).

3. Vì sao O(n²) nguy hiểm khi scale?  
   **Đáp án:** Thời gian tăng nhanh theo bình phương kích thước dữ liệu.

4. Sort rồi binary search khi nào hợp lý?  
   **Đáp án:** Nhiều lần tìm trên tập ít thay đổi.

5. Complexity nhỏ có luôn nhanh hơn không?  
   **Đáp án:** Không tuyệt đối — constant, alloc, cache, n thực tế quan trọng; cần đo.
