# Chương 7 — Iterators & yield

## 1. Mục tiêu học

- Viết **iterator method** trả `IEnumerable<T>` / `IEnumerator<T>` bằng **`yield return`**
- Dùng **`yield break`** để dừng sớm
- Hiểu iterator là **state machine** trì hoãn — liên hệ deferred LINQ
- Biết hạn chế (`yield` trong try-catch, ref struct, …) mức thực dụng

## 2. Điều kiện tiên quyết

- Level 4: `IEnumerable`, foreach
- Level 8: deferred execution
- Chương 6: có thể viết extension iterator

## 3. Khái niệm

Thay vì xây `List` rồi trả về, iterator **sinh phần tử khi được duyệt**:

```csharp
IEnumerable<int> OneToThree()
{
    yield return 1;
    yield return 2;
    yield return 3;
}
```

Compiler tạo class state machine triển khai `IEnumerator`. Mỗi `MoveNext` chạy đến `yield` kế tiếp.

| Thành phần | Ý nghĩa |
|------------|---------|
| `yield return x` | Đưa x ra; tạm dừng method |
| `yield break` | Kết thúc dãy |
| `IEnumerable<T>` | Có thể GetEnumerator nhiều lần (factory) |
| `IAsyncEnumerable<T>` | async stream (`await foreach`) — preview Level 11 |

## 4. Mô hình tư duy

```text
Caller foreach
    → GetEnumerator()
    → MoveNext() → chạy đến yield return → Current
    → MoveNext() → tiếp tục sau yield trước
    → MoveNext() false → xong

Chưa foreach → thân iterator gần như chưa chạy (giống deferred)
```

## 5. Cú pháp

```csharp
public static IEnumerable<int> Range(int start, int count)
{
    for (int i = 0; i < count; i++)
        yield return start + i;
}

public static IEnumerable<T> TakeUntil<T>(this IEnumerable<T> src, Func<T, bool> stop)
{
    foreach (var item in src)
    {
        if (stop(item))
            yield break;
        yield return item;
    }
}
```

## 6. Ví dụ

### Cơ bản

```csharp
foreach (var n in Range(10, 3))
    Console.WriteLine(n); // 10 11 12
```

### Trung cấp — filter không LINQ

```csharp
static IEnumerable<string> NonBlank(IEnumerable<string?> lines)
{
    foreach (var line in lines)
    {
        if (string.IsNullOrWhiteSpace(line))
            continue;
        yield return line.Trim();
    }
}
```

### Nâng cao — infinite (cẩn thận) + composition

```csharp
static IEnumerable<int> Infinite()
{
    int n = 0;
    while (true)
        yield return n++;
}

// Chỉ an toàn khi kết hợp Take:
foreach (var n in Infinite().Take(5))
    Console.WriteLine(n);

static IEnumerable<(T Item, int Index)> WithIndex<T>(IEnumerable<T> src)
{
    int i = 0;
    foreach (var item in src)
        yield return (item, i++);
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Vòng vô hạn treo | Iterator vô hạn không Take | `Take` / `yield break` |
| Exception trong iterator muộn | Deferred — lỗi lúc foreach | Hiểu timing; validate sớm nếu cần |
| `yield` trong catch | Không cho phép một số cấu trúc | Tái cấu trúc try |
| Multiple enum chạy lại | Mỗi foreach = chạy lại từ đầu | Materialize nếu đắt |
| Side-effect bất ngờ | Log trong iterator | Tránh / document |

## 8. Gỡ lỗi

1. Breakpoint trên `yield return` — thấy nhịp MoveNext.
2. Log “start iterator” đầu method — chỉ in khi bắt đầu duyệt.
3. So với `ToList()` bên trong — thấy khác deferred.
4. Test `yield break` với điều kiện biên.

## 9. Best practices

- Iterator cho pipeline / stream lớn — không giữ hết RAM.
- Đặt tên rõ `EnumerateX`.
- Document nếu đắt khi duyệt lại.
- Prefer LINQ có sẵn nếu đủ; tự `yield` khi logic đặc thù.
- Không iterator vô hạn public không cảnh báo.

## 10. Bài tập

**Bài 1** — `IEnumerable<char> Letters(string s)` yield từng ký tự.

**Bài 2** — `UpTo(int n)` yield 1..n; n < 1 → không phần tử.

**Bài 3** — Extension `SkipNulls<T>` where T : class.

**Bài 4** — `RepeatForever<T>(T item)` + dùng với `Take(3)` trong demo.

## 11. Gợi ý

- Bài 1: foreach char yield.
- Bài 2: for + yield; hoặc n < 1 thì yield break ngay.
- Bài 3: if item is null continue.
- Bài 4: while(true) yield return item.

## 12. Đáp án

```csharp
static IEnumerable<char> Letters(string s)
{
    foreach (var c in s)
        yield return c;
}

static IEnumerable<int> UpTo(int n)
{
    for (int i = 1; i <= n; i++)
        yield return i;
}

static IEnumerable<T> SkipNulls<T>(this IEnumerable<T?> src) where T : class
{
    foreach (var item in src)
    {
        if (item is null) continue;
        yield return item;
    }
}

static IEnumerable<T> RepeatForever<T>(T item)
{
    while (true)
        yield return item;
}
```

## 13. Đáp án thay thế

Trả `List` built sẵn — đơn giản nhưng mất deferred. `UpTo` có thể `Enumerable.Range(1, Math.Max(0, n))`.

## 14. Thử thách

Viết lại `Select` và `Where` tối giản bằng `yield` (signature giống LINQ); so sánh với LINQ BCL.

## 15. Ứng dụng thực tế

- Đọc file từng dòng (`File.ReadLines` kiểu iterator)
- Generate permutation / graph walk
- Custom LINQ operators
- Streaming parse CSV

## 16. Liên hệ Unity

- Spawn theo frame: coroutine Unity khác `yield` C# iterator một chút nhưng ý niệm tạm dừng tương tự
- Editor: enumerate assets
- Tránh iterator allocate nặng trong Update — cache
- `IEnumerable` custom pathfinding step-by-step

## 17. Kiểm tra kiến thức

1. `yield return` làm gì?  
   **Đáp án:** Trả phần tử hiện tại và tạm dừng iterator đến lần MoveNext sau.

2. Chưa foreach thì thân iterator chạy chưa?  
   **Đáp án:** Về cơ bản chưa (deferred) — đến khi bắt đầu enumerate.

3. `yield break`?  
   **Đáp án:** Kết thúc dãy sớm.

4. Iterator vô hạn nguy hiểm thế nào?  
   **Đáp án:** foreach không Take sẽ treo / không kết thúc.

5. Liên hệ LINQ deferred?  
   **Đáp án:** Cùng mô hình “chạy khi duyệt”; nhiều toán tử LINQ triển khai bằng iterator.
