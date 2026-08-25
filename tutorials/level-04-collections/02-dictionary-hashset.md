# Chương 2 — Dictionary và HashSet

## 1. Mục tiêu học

- Dùng `Dictionary<TKey, TValue>` để tra cứu theo khóa O(1) trung bình
- Dùng `HashSet<T>` để tập hợp không trùng, kiểm tra thuộc tập nhanh
- Hiểu yêu cầu `GetHashCode` / `Equals` với key tùy chỉnh
- Tránh lỗi key trùng và lookup thiếu key

## 2. Điều kiện tiên quyết

- Chương 1: `List<T>`, vòng lặp
- Level 2: class, override cơ bản (sẽ nhắc `Equals`/`GetHashCode`)
- Level 3: reference vs value khi dùng object làm key

## 3. Khái niệm

### Dictionary\<TKey, TValue\>

- Lưu cặp **khóa → giá trị**
- Tra cứu, thêm, xóa theo key: **O(1) trung bình** (hash table)
- Key phải **duy nhất**; không được `null` (với reference type key — trừ nullable đặc biệt)
- Duyệt không đảm bảo thứ tự chèn (trừ khi bạn dựa vào implementation — đừng phụ thuộc)

### HashSet\<T\>

- Tập hợp phần tử **không trùng**
- `Add`, `Contains`, `Remove`: O(1) trung bình
- Phép tập hợp: `UnionWith`, `IntersectWith`, `ExceptWith`, `IsSubsetOf`…

| Nhu cầu | Chọn |
|---------|------|
| Tra cứu giá trị theo ID/tên | `Dictionary` |
| Chỉ cần biết “đã thấy chưa?” / unique | `HashSet` |
| Danh sách có thứ tự, truy cập index | `List` |

## 4. Mô hình tư duy

```text
Dictionary playerById:
  hash(1001) → bucket → (1001, "Alice")
  hash(1002) → bucket → (1002, "Bob")

Lookup 1002: hash → bucket → so sánh Equals → "Bob"   ~O(1)

HashSet onlineIds: { 1001, 1002, 1005 }
Contains(1002) → true mà không cần duyệt cả list
```

## 5. Cú pháp

```csharp
using System.Collections.Generic;

var hpByName = new Dictionary<string, int>
{
    ["Warrior"] = 100,
    ["Mage"] = 60
};

hpByName["Rogue"] = 80;              // thêm hoặc ghi đè
hpByName.Add("Tank", 150);           // thêm; trùng key → exception

if (hpByName.TryGetValue("Mage", out int hp))
    Console.WriteLine(hp);

hpByName.Remove("Rogue");
bool has = hpByName.ContainsKey("Tank");

foreach (var kv in hpByName)
    Console.WriteLine($"{kv.Key}: {kv.Value}");

var tags = new HashSet<string>(StringComparer.OrdinalIgnoreCase)
{
    "Fire", "Ice"
};
tags.Add("fire"); // false — đã có (ignore case)
```

## 6. Ví dụ

### Cơ bản

Đếm tần suất từ:

```csharp
static Dictionary<string, int> WordCount(string[] words)
{
    var map = new Dictionary<string, int>(StringComparer.OrdinalIgnoreCase);
    foreach (string w in words)
    {
        if (map.ContainsKey(w))
            map[w]++;
        else
            map[w] = 1;
        // hoặc: map[w] = map.GetValueOrDefault(w) + 1;
    }
    return map;
}
```

### Trung cấp

Index hóa list player theo Id:

```csharp
record Player(int Id, string Name);

static Dictionary<int, Player> IndexById(List<Player> players)
{
    var dict = new Dictionary<int, Player>(players.Count);
    foreach (var p in players)
        dict[p.Id] = p; // Id trùng → giữ bản sau
    return dict;
}
```

### Nâng cao

HashSet cho unique + phép giao:

```csharp
var questA = new HashSet<int> { 1, 2, 3, 5 };
var questB = new HashSet<int> { 2, 5, 8 };

var both = new HashSet<int>(questA);
both.IntersectWith(questB); // { 2, 5 }

var onlyA = new HashSet<int>(questA);
onlyA.ExceptWith(questB);   // { 1, 3 }
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| `KeyNotFoundException` | `dict[key]` khi key không có | `TryGetValue` / `ContainsKey` |
| `ArgumentException` key đã tồn tại | `Add` trùng | Dùng indexer `dict[k]=v` hoặc kiểm tra trước |
| Key class tự viết: `Contains` luôn false | Không override `Equals`/`GetHashCode` nhất quán | Implement đúng cặp Equals + GetHashCode |
| Dùng `List` + `Find` thay Dictionary | O(n) mỗi lần | Index hóa bằng Dictionary |
| Sửa object key sau khi Add | Hash thay đổi → “mất” entry | Key nên immutable |

## 8. Gỡ lỗi

1. In `dict.Count` và `dict.Keys` khi nghi thiếu dữ liệu.
2. Breakpoint tại `TryGetValue` — xem key thực tế (khoảng trắng, hoa thường).
3. Với string key: thử `StringComparer.OrdinalIgnoreCase`.
4. Unit test nhỏ: Add rồi Contains ngay — nếu fail, nghi Equals/GetHashCode.

## 9. Best practices

- Luôn ưu tiên `TryGetValue` thay vì `ContainsKey` + indexer (một lần hash).
- Chọn `IEqualityComparer<T>` phù hợp lúc tạo Dictionary/HashSet.
- Key nên bất biến (record/struct readonly hoặc string/int).
- Ước lượng `capacity` khi biết số phần tử: `new Dictionary<int, T>(count)`.
- Unity: cache `Dictionary` tra cứu prefab/id; đừng tìm trong list mỗi frame.

## 10. Bài tập

**Bài 1 — Phone book**  
*Input:* các cặp tên → SĐT (string).  
*Output:* tra cứu theo tên; in “Not found” nếu thiếu.

**Bài 2 — Unique numbers**  
*Input:* `int[]` có thể trùng.  
*Output:* số lượng giá trị **phân biệt** (dùng HashSet).

**Bài 3 — Two Sum index**  
*Input:* `int[] nums`, `int target`.  
*Output:* hai index `i < j` sao cho `nums[i] + nums[j] == target` (giả sử luôn có đúng một cặp). Dùng Dictionary.

**Bài 4 — Anagram check**  
*Input:* hai chuỗi.  
*Output:* `true` nếu là anagram (cùng tần suất ký tự), bỏ qua hoa thường.

## 11. Gợi ý

- Bài 1: `Dictionary<string, string>` + `TryGetValue`.
- Bài 2: `new HashSet<int>(array).Count`.
- Bài 3: duyệt một lần; với mỗi `x`, tìm `target - x` đã thấy chưa.
- Bài 4: đếm tần suất char bằng Dictionary, so sánh hai map; hoặc sort rồi so chuỗi.

## 12. Đáp án

**Bài 1** — Tra cứu an toàn bằng TryGetValue:

```csharp
static void LookupPhone(Dictionary<string, string> book, string name)
{
    if (book.TryGetValue(name, out string? phone))
        Console.WriteLine(phone);
    else
        Console.WriteLine("Not found");
}
```

**Bài 2** — HashSet tự loại trùng:

```csharp
static int CountUnique(int[] nums) => new HashSet<int>(nums).Count;
```

**Bài 3** — Một lần duyệt: lưu giá trị → index đã gặp:

```csharp
static (int i, int j) TwoSum(int[] nums, int target)
{
    var seen = new Dictionary<int, int>(); // value → index
    for (int i = 0; i < nums.Length; i++)
    {
        int need = target - nums[i];
        if (seen.TryGetValue(need, out int j))
            return (j, i);
        seen[nums[i]] = i;
    }
    throw new InvalidOperationException("No pair");
}
```

**Bài 4** — Đếm ký tự (bỏ khoảng trắng tùy chọn — ở đây so mọi char, ignore case):

```csharp
static bool AreAnagrams(string a, string b)
{
    if (a.Length != b.Length) return false;
    var count = new Dictionary<char, int>();
    foreach (char c in a.ToLowerInvariant())
        count[c] = count.GetValueOrDefault(c) + 1;
    foreach (char c in b.ToLowerInvariant())
    {
        if (!count.TryGetValue(c, out int n) || n == 0) return false;
        count[c] = n - 1;
    }
    return true;
}
```

## 13. Đáp án thay thế

Bài 4 bằng sort:

```csharp
static bool AreAnagramsSort(string a, string b)
{
    if (a.Length != b.Length) return false;
    var ca = a.ToLowerInvariant().ToCharArray();
    var cb = b.ToLowerInvariant().ToCharArray();
    Array.Sort(ca);
    Array.Sort(cb);
    return ca.SequenceEqual(cb); // cần using System.Linq; hoặc so tay
}
```

(Nếu chưa học LINQ: so từng phần tử sau `Sort` bằng vòng `for`.)

## 14. Thử thách

Viết `FirstUniqueChar(string s)` trả về ký tự đầu tiên chỉ xuất hiện một lần; nếu không có trả `'\0'`. Gợi ý: Dictionary đếm + duyệt lại chuỗi.

## 15. Ứng dụng thực tế

- Cache kết quả API theo key
- Index database in-memory (Id → Entity)
- Session / permission set (`HashSet` role names)
- Đếm metric, histogram

## 16. Liên hệ Unity

- `Dictionary<int, GameObject>` map networkId → object
- `HashSet<Collider>` object đang overlap trigger
- ScriptableObject catalog: load một lần vào Dictionary theo itemId
- Tránh `FindObjectsOfType` mỗi frame — cache vào Dictionary lúc Start

## 17. Kiểm tra kiến thức

1. Vì sao Dictionary tra cứu nhanh hơn List?  
   **Đáp án:** Hash table → O(1) trung bình thay vì duyệt O(n).

2. `dict[key]` khi thiếu key thì sao?  
   **Đáp án:** Ném `KeyNotFoundException`. Nên dùng `TryGetValue`.

3. HashSet dùng khi nào?  
   **Đáp án:** Cần tập không trùng và kiểm tra thuộc tập nhanh.

4. Key class tự viết cần gì?  
   **Đáp án:** `Equals` và `GetHashCode` nhất quán (hoặc comparer riêng).

5. `Add` trùng key trên Dictionary?  
   **Đáp án:** `ArgumentException`. Indexer thì ghi đè.
