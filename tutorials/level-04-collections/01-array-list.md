# Chương 1 — Array và List\<T\>

## 1. Mục tiêu học

- Phân biệt mảng cố định kích thước (`T[]`) và danh sách động (`List<T>`)
- Thêm, xóa, duyệt, tìm kiếm trên cả hai
- Hiểu capacity vs Count của `List<T>`
- Chọn đúng khi nào dùng Array, khi nào dùng List

## 2. Điều kiện tiên quyết

- Level 1: vòng lặp `for` / `foreach`, method
- Level 2: class cơ bản
- Level 3: mảng là reference type; `List<T>` cũng nằm trên heap

## 3. Khái niệm

### Array (`T[]`)

- Dãy phần tử **cùng kiểu**, kích thước **cố định** sau khi tạo
- Truy cập bằng chỉ số `O(1)`: `arr[i]`
- Không thể `Add`/`Remove` thay đổi độ dài (phải tạo mảng mới hoặc dùng `Array.Resize`)

### List\<T\>

- Collection generic trong `System.Collections.Generic`
- Kích thước **thay đổi được**: `Add`, `Remove`, `Insert`
- Bên trong vẫn là mảng; khi đầy thì **tăng capacity** (thường ×2) và copy

| | Array | List\<T\> |
|---|-------|-----------|
| Kích thước | Cố định | Động |
| Truy cập theo index | O(1) | O(1) |
| Thêm cuối | Không (trừ tạo mới) | O(1) amortized |
| Chèn / xóa giữa | Phải tự dịch chuyển | O(n) |
| Khi dùng | Độ dài biết trước, buffer cố định | Danh sách thay đổi thường xuyên |

## 4. Mô hình tư duy

```text
Array (length = 4, đầy đủ):
┌───┬───┬───┬───┐
│ A │ B │ C │ D │
└───┴───┴───┴───┘
  0   1   2   3

List (Count = 3, Capacity = 4):
┌───┬───┬───┬───┐
│ A │ B │ C │   │  ← slot trống vẫn chiếm bộ nhớ (capacity)
└───┴───┴───┴───┘
Count=3        Capacity=4

Add("D") → Count=4. Add("E") → capacity tăng (vd 8), copy sang mảng mới.
```

## 5. Cú pháp

```csharp
using System.Collections.Generic;

// Array
int[] scores = new int[3];           // [0, 0, 0]
int[] fixedScores = { 10, 20, 30 };  // collection initializer (cũ)
string[] names = ["Alice", "Bob"];   // C# 12 collection expression

// List
var players = new List<string>();
players.Add("Warrior");
players.Add("Mage");

var hp = new List<int> { 100, 80, 120 };
hp.Insert(1, 90);     // chèn tại index 1
hp.Remove(80);        // xóa giá trị đầu tiên khớp
hp.RemoveAt(0);       // xóa theo index
int n = hp.Count;
int cap = hp.Capacity;
```

API hay dùng: `Contains`, `IndexOf`, `Clear`, `Sort`, `ToArray`, `AddRange`, `EnsureCapacity` (.NET 6+).

## 6. Ví dụ

### Cơ bản

Duyệt và tính tổng:

```csharp
int[] values = { 1, 2, 3, 4, 5 };
int sum = 0;
for (int i = 0; i < values.Length; i++)
    sum += values[i];

Console.WriteLine(sum); // 15

var list = new List<int> { 1, 2, 3 };
foreach (var x in list)
    Console.WriteLine(x);
```

### Trung cấp

Lọc số chẵn vào List mới (không dùng LINQ — học sau Level 8):

```csharp
static List<int> EvensOnly(int[] source)
{
    var result = new List<int>();
    foreach (int n in source)
    {
        if (n % 2 == 0)
            result.Add(n);
    }
    return result;
}
```

### Nâng cao

Pre-allocate capacity khi biết trước số phần tử (tránh resize nhiều lần):

```csharp
static List<string> LoadNames(int expectedCount)
{
    var names = new List<string>(expectedCount); // capacity ban đầu
    for (int i = 0; i < expectedCount; i++)
        names.Add($"Player_{i}");
    return names;
}

// Sao chép mảng an toàn
int[] original = { 1, 2, 3 };
int[] copy = new int[original.Length];
Array.Copy(original, copy, original.Length);
// hoặc: int[] copy2 = original.ToArray();
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| `IndexOutOfRangeException` | Index &lt; 0 hoặc ≥ Length/Count | Kiểm tra biên trước khi truy cập |
| `ArgumentOutOfRangeException` trên List | `RemoveAt` / indexer sai | `if (i >= 0 && i < list.Count)` |
| Sửa `arr` sau khi `list = arr.ToList()` rồi mong list đổi | Đó là **copy** phần tử | Hiểu List mới độc lập (với value type) |
| Dùng `ArrayList` (non-generic) | Tài liệu cũ | Dùng `List<T>` |
| `list.Count` vs `list.Capacity` nhầm | Capacity là chỗ trống nội bộ | Chỉ quan tâm `Count` khi duyệt |

## 8. Gỡ lỗi

1. In `Length` / `Count` trước vòng lặp nghi ngờ.
2. Breakpoint tại dòng index — xem giá trị `i`.
3. Với List: xem `Capacity` có nhảy lớn bất thường (dấu hiệu Add trong vòng lặp nóng).
4. `Collection was modified; enumeration operation may not execute` → đang `foreach` vừa `Remove` — dùng `for` ngược hoặc thu thập rồi xóa sau.

## 9. Best practices

- Biết trước độ dài cố định → ưu tiên `T[]` hoặc `Span<T>` (sau này).
- Danh sách thay đổi → `List<T>`; set `Capacity` nếu biết gần đúng số phần tử.
- Không expose `List<T>` mutable ra public API nếu caller không được sửa — trả về `IReadOnlyList<T>` (chương 5).
- Tránh `Remove` trong vòng `foreach`.
- Unity: tái sử dụng `List` với `Clear()` thay vì `new List` mỗi frame.

## 10. Bài tập

**Bài 1 — Đảo ngược mảng**  
*Input:* `int[]` bất kỳ.  
*Output:* mảng mới với phần tử đảo ngược (không dùng `Array.Reverse` cho bài này).

**Bài 2 — Max trong List**  
*Input:* `List<int>` không rỗng.  
*Output:* giá trị lớn nhất.

**Bài 3 — Gộp hai list**  
*Input:* hai `List<string>`.  
*Output:* một `List<string>` gồm tất cả phần tử (giữ thứ tự: list1 rồi list2).

**Bài 4 — Xóa trùng liền kề**  
*Input:* `List<int>` đã sắp xếp không giảm.  
*Output:* list mới không còn phần tử trùng **liền kề** (giữ một bản).

## 11. Gợi ý

- Bài 1: tạo mảng cùng `Length`, gán `result[i] = source[Length - 1 - i]`.
- Bài 2: khởi tạo `max = list[0]`, duyệt từ 1.
- Bài 3: `AddRange` hoặc hai vòng `Add`.
- Bài 4: duyệt, chỉ `Add` khi khác phần tử vừa thêm (hoặc index 0).

## 12. Đáp án

**Bài 1** — Tạo mảng mới, map chỉ số từ cuối về đầu:

```csharp
static int[] ReverseCopy(int[] source)
{
    var result = new int[source.Length];
    for (int i = 0; i < source.Length; i++)
        result[i] = source[source.Length - 1 - i];
    return result;
}
```

**Bài 2** — Giả định list không rỗng, cập nhật max khi gặp số lớn hơn:

```csharp
static int FindMax(List<int> list)
{
    int max = list[0];
    for (int i = 1; i < list.Count; i++)
    {
        if (list[i] > max)
            max = list[i];
    }
    return max;
}
```

**Bài 3** — Dùng capacity = tổng Count rồi `AddRange`:

```csharp
static List<string> Concat(List<string> a, List<string> b)
{
    var result = new List<string>(a.Count + b.Count);
    result.AddRange(a);
    result.AddRange(b);
    return result;
}
```

**Bài 4** — Chỉ thêm khi list kết quả rỗng hoặc khác phần tử cuối:

```csharp
static List<int> CollapseAdjacentDuplicates(List<int> sorted)
{
    var result = new List<int>();
    foreach (int n in sorted)
    {
        if (result.Count == 0 || result[^1] != n)
            result.Add(n);
    }
    return result;
}
```

## 13. Đáp án thay thế

Bài 1 dùng hai con trỏ in-place trên bản copy:

```csharp
static int[] ReverseCopyAlt(int[] source)
{
    int[] result = (int[])source.Clone();
    int left = 0, right = result.Length - 1;
    while (left < right)
    {
        (result[left], result[right]) = (result[right], result[left]);
        left++;
        right--;
    }
    return result;
}
```

## 14. Thử thách

Viết `RemoveAllMatching(List<int> list, int value)` **không** dùng `RemoveAll` / LINQ: xóa mọi phần tử bằng `value`, giữ thứ tự phần tử còn lại, độ phức tạp O(n) (một lần duyệt + compact).

## 15. Ứng dụng thực tế

- Buffer điểm số, lịch sử log, danh sách đơn hàng theo thứ tự thời gian
- API config: mảng cố định các mức difficulty
- Batch processing: đọc N dòng vào `List<string>` rồi xử lý

## 16. Liên hệ Unity

- `GameObject[]` / `Transform[]` từ `FindGameObjectsWithTag`
- Inventory slot: thường `List<Item>` hoặc mảng cố định kích thước túi
- **Object pooling:** giữ `List<T>` / `Queue<T>` tái sử dụng thay vì `Instantiate`/`Destroy` liên tục
- Tránh `new List<>()` trong `Update()` — allocate mỗi frame → GC spike

## 17. Kiểm tra kiến thức

1. `Array` khác `List<T>` chỗ nào quan trọng nhất?  
   **Đáp án:** Array kích thước cố định; List thay đổi được (Count/Capacity).

2. Truy cập `list[i]` độ phức tạp?  
   **Đáp án:** O(1).

3. `Count` vs `Capacity`?  
   **Đáp án:** Count = số phần tử đang dùng; Capacity = kích thước mảng nội bộ.

4. `Insert(0, x)` trên List lớn chậm vì sao?  
   **Đáp án:** Phải dịch chuyển mọi phần tử sang phải → O(n).

5. Vì sao không nên `Remove` trong `foreach`?  
   **Đáp án:** Enumerator bị invalidate → exception “Collection was modified”.
