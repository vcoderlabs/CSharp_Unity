# Chương 2 — Boxing, LINQ overhead & string allocation

## 1. Mục tiêu học

- Nhận diện **boxing** / unboxing và chi phí
- Hiểu LINQ đẹp nhưng có thể allocate + chậm trên hot path
- Kiểm soát **string** allocation (`+`, interpolation, `Substring`)
- Biết khi nào giữ LINQ/string cho rõ ràng, khi nào viết vòng lặp tay

## 2. Điều kiện tiên quyết

- Level 3: value vs reference
- Level 8: LINQ cơ bản
- Chương 1 Level 19: allocation / GC

## 3. Khái niệm

### Boxing

Đưa **value type** vào ngữ cảnh `object` / interface không generic → CLR bọc thành object trên heap.

```csharp
object o = 42;          // box int
IComparable c = 42;     // box
```

Unboxing: `(int)o` — cast sai → exception.

### LINQ overhead

Mỗi operator thường: delegate, iterator state machine, có thể `ToList` materialize. Deferred execution dễ **enumerate nhiều lần**. Hot path tight loop: `for` thường thắng.

### String allocation

`string` bất biến. Mọi “sửa” → string mới. `Split`, `Substring` (một số API), interpolation đều có thể allocate.

## 4. Mô hình tư duy

```text
Boxing:
  int (stack/inline) ──box──► object trên heap
  Non-generic ArrayList, Hashtable, interface cũ = bẫy

LINQ:
  đẹp, declaratif  →  OK cold path / batch nhỏ
  hot path 10^6 lần/frame  →  cân nhắc vòng lặp / Span

String:
  s += x trong loop  →  O(n²) alloc
  StringBuilder / string.Create  →  kiểm soát buffer
```

## 5. Cú pháp

```csharp
// Tránh boxing với generic
void Print<T>(T value) => Console.WriteLine(value);

// Interface generic
int Compare<T>(T a, T b) where T : IComparable<T> => a.CompareTo(b);

// LINQ → vòng lặp
int count = 0;
foreach (var x in source)
    if (x > 0) count++;

// String
var sb = new StringBuilder();
sb.Append(name).Append(':').Append(value);
string result = sb.ToString();

// Composite format tái sử dụng (ít dùng hơn string.Create hiện đại)
```

## 6. Ví dụ

### Cơ bản — boxing

```csharp
ArrayList legacy = new();
legacy.Add(1);      // box
legacy.Add(2);      // box
int x = (int)legacy[0]; // unbox

List<int> modern = new() { 1, 2 }; // không box
```

### Trung cấp — LINQ vs vòng lặp

```csharp
static int SumPositiveLinq(List<int> data)
    => data.Where(x => x > 0).Sum();

static int SumPositiveLoop(List<int> data)
{
    int sum = 0;
    for (int i = 0; i < data.Count; i++)
        if (data[i] > 0) sum += data[i];
    return sum;
}
```

`Where` + `Sum`: iterators + delegates. Loop: gần như không alloc.

### Nâng cao — string & params boxing

```csharp
// Console.WriteLine với nhiều arg value type có thể box tùy overload
Console.WriteLine("HP={0} MP={1}", hp, mp); // format, có thể box

// Tốt hơn khi hot:
Console.WriteLine($"HP={hp} MP={mp}"); // vẫn allocate string kết quả

// Params object[] cũng box value types:
static void Log(string msg, params object[] args) { /* ... */ }
Log("x={0}", 1); // 1 bị box + mảng args allocate
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| `List<object>` chứa int | Boxing mỗi phần tử | `List<int>` / generic |
| LINQ trong `Update` | Alloc mỗi frame | Cache kết quả hoặc `for` |
| `foreach` trên `IEnumerable` | Enumerator class | `List`/`array` + index |
| `string.Split` mỗi parse | Mảng + substring | Span / split thủ công |
| Multiple enumeration | LINQ query chạy lại | `ToList` một lần hoặc một pass |

## 8. Gỡ lỗi

1. IL / SharpLab: xem instruction `box`.
2. BenchmarkDotNet `MemoryDiagnoser`: so LINQ vs loop.
3. Đặt câu hỏi: API nhận `object` hay `T`?
4. Rider/VS: “Heap allocations” highlight (nếu bật analyzer).

## 9. Best practices

- Public API hot: generic + `IEquatable<T>` / `IComparable<T>`.
- LINQ tuyệt vời cho business logic không hot; đừng dogma “cấm LINQ”.
- String: `StringBuilder` khi nối nhiều; cân nhắc `string.Join`.
- Logging structured (Serilog) tránh `string.Format` nặng trên hot path — hoặc sampling.
- `CollectionsMarshal` / `Span` khi đã đo và cần ép hiệu năng (cẩn thận an toàn).

## 10. Bài tập

**Bài 1** — Viết benchmark tư duy (hoặc code): `IComparable` non-generic vs `IComparable<int>` khi so 1 triệu cặp int.

**Bài 2** — Đổi `items.Where(x => x.IsActive).Select(x => x.Id).ToList()` thành một vòng `for` tương đương trên `List<Item>`.

**Bài 3** — So ba cách build key `"user:" + id` vs `$"user:{id}"` vs `string.Create` / `StringBuilder` (N lớn).

**Bài 4** — Tìm boxing trong: `Hashtable`, `ArrayList`, event `EventHandler` với struct sender custom? (giải thích).

## 11. Gợi ý

- Bài 1: non-generic `CompareTo(object)` box.
- Bài 2: một pass, `if` + `Add` vào `List<int>` pre-size nếu biết gần đúng.
- Bài 3: cả `+` và `$` đều tạo string mới; khác ở số bước trung gian khi nối nhiều mảnh.
- Bài 4: `Hashtable` key/value là object.

## 12. Đáp án

**Bài 2**:

```csharp
static List<int> ActiveIds(List<Item> items)
{
    var ids = new List<int>(items.Count);
    for (int i = 0; i < items.Count; i++)
    {
        var it = items[i];
        if (it.IsActive) ids.Add(it.Id);
    }
    return ids;
}
```

**Bài 1** — Kết luận kỳ vọng: generic không box; non-generic box mỗi lần gọi.

**Bài 3** — Với 2 mảnh ngắn, `$` và `+` tương đương về kết quả alloc cuối; vòng lặp nhiều lần → `StringBuilder`.

**Bài 4** — `Hashtable.Add(1, 2)` box cả key và value.

## 13. Đáp án thay thế

Dùng `Collections.Immutable` / pipeline `IAsyncEnumerable` — trade-off khác. `ValueStringBuilder` (internal / community) cho zero-alloc build string nâng cao.

## 14. Thử thách

Viết `TryParseHpMp(ReadOnlySpan<char> line, out int hp, out int mp)` cho `"HP:10 MP:20"` không dùng `Split`/`Substring` tạo string trung gian; so MemoryDiagnoser với bản `Split`.

## 15. Ứng dụng thực tế

- Game server: combat log string → sampling hoặc binary log
- API: projection DTO — LINQ OK; tight serialize loop — tay
- Dictionary key struct: implement `IEquatable<T>` tránh box khi cast equality comparer cũ

## 16. Liên hệ Unity

- `Debug.Log($"...{x}")` trong Update = alloc + I/O
- LINQ trong gameplay code = dấu hiệu đỏ trên mobile
- Tag/layer so sánh: dùng int id thay vì string khi có thể
- `GetComponents` trả mảng — cache, đừng gọi mỗi frame

## 17. Kiểm tra kiến thức

1. Boxing xảy ra khi nào?  
   **Đáp án:** Value type được dùng như `object`/interface không generic (hoặc tương đương).

2. `List<int>` có box khi Add không?  
   **Đáp án:** Không (generic).

3. Vì sao LINQ có thể chậm hơn `for`?  
   **Đáp án:** Iterator, delegate, alloc, nhiều pass enumeration.

4. `string` nối trong vòng lặp nên dùng gì?  
   **Đáp án:** `StringBuilder` (hoặc API tạo một lần).

5. Có phải luôn cấm LINQ?  
   **Đáp án:** Không — cấm trên hot path đã đo; giữ nơi rõ ràng quan trọng hơn microseconds.
