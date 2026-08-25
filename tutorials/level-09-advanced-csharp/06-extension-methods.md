# Chương 6 — Extension methods

## 1. Mục tiêu học

- Viết **extension method** đúng cú pháp (`this` modifier)
- Tổ chức trong **static class** + namespace
- Hiểu thứ tự ưu tiên (instance method thắng extension)
- Áp dụng cho fluent API và helper trên kiểu có sẵn / không sửa được

## 2. Điều kiện tiên quyết

- Level 1–2: method, static class
- Level 8: đã *dùng* LINQ — LINQ là extension methods trên `IEnumerable<T>`

## 3. Khái niệm

**Extension method** = static method “giả trang” thành instance method nhờ tham số đầu `this TypeName self`.

```csharp
public static class StringExtensions
{
    public static bool IsEmpty(this string? s) => string.IsNullOrEmpty(s);
}

// Gọi:
"".IsEmpty();
StringExtensions.IsEmpty(""); // cũng được
```

Điều kiện:

- Class chứa **static**
- Method **static**
- Tham số đầu có `this`
- Caller **import namespace** chứa class (using)

Không thể: override method thật, truy cập `private` của kiểu đang extend (trừ internals cùng assembly với `InternalsVisibleTo` — không phải mục tiêu chính).

## 4. Mô hình tư duy

```text
Bạn không sửa được string / List / Type thư viện
    → viết extension trong project bạn
    → using Namespace → gọi như member

LINQ: Enumerable.Where(this IEnumerable<T> …)
```

## 5. Cú pháp

```csharp
namespace MyApp.Extensions;

public static class EnumerableExtras
{
    public static bool None<T>(this IEnumerable<T> source) => !source.Any();

    public static IEnumerable<T> WhereNotNull<T>(this IEnumerable<T?> source)
        where T : class => source.Where(x => x is not null)!;
}
```

## 6. Ví dụ

### Cơ bản

```csharp
public static class IntExtensions
{
    public static bool IsEven(this int n) => n % 2 == 0;
}

Console.WriteLine(4.IsEven()); // True
```

### Trung cấp — chuỗi fluent

```csharp
public static class StringExt
{
    public static string Truncate(this string s, int max) =>
        s.Length <= max ? s : s[..max] + "…";

    public static string OrDefault(this string? s, string fallback) =>
        string.IsNullOrWhiteSpace(s) ? fallback : s;
}

var title = maybeTitle.OrDefault("Untitled").Truncate(20);
```

### Nâng cao — generic + tránh xung đột

```csharp
public static class CollectionExt
{
    public static void AddRange<T>(this ICollection<T> target, IEnumerable<T> items)
    {
        foreach (var i in items)
            target.Add(i);
    }
}

// Nếu tự viết Where trên MyList có instance Where — instance thắng
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Không hiện IntelliSense | Thiếu `using` namespace | Import đúng |
| Không compile `this` | Method/class không static | Thêm `static` |
| Extension trên `null` | Vẫn gọi được về lý thuyết | Null-check trong method |
| Trùng tên LINQ | Ambiguous | Namespace / gọi static rõ |
| “Override” ToString bằng extension | Không được — instance thắng | Đặt tên khác |

## 8. Gỡ lỗi

1. F12 vào method — nhảy tới extension nào.
2. Ambiguous: chỉ định `MyExt.Method(x)` đầy đủ.
3. Test null: ` ((string?)null).IsEmpty()`.
4. Đảm bảo file cùng ngôn ngữ C# (không nhầm file khác).

## 9. Best practices

- Đặt tên class `XxxExtensions`.
- Namespace `Company.Product.Extensions`.
- Không tạo extension “thần thánh” cho mọi thứ — chỉ khi tái sử dụng thật.
- Document null behavior.
- Tránh extension làm side-effect nặng khó thấy khi đọc call site.

## 10. Bài tập

**Bài 1** — `IsBlank(this string? s)` — null/whitespace → true.

**Bài 2** — `Clamp(this int n, int min, int max)`.

**Bài 3** — `ForEach<T>(this IEnumerable<T> src, Action<T> action)`.

**Bài 4** — `ToCsv(this IEnumerable<string> parts)` join bằng dấu phẩy.

## 11. Gợi ý

- Bài 1: `string.IsNullOrWhiteSpace`.
- Bài 2: `Math.Clamp` (.NET) hoặc if.
- Bài 3: foreach gọi action — hiểu trade-off vs foreach tường minh.
- Bài 4: `string.Join(',', parts)`.

## 12. Đáp án

```csharp
public static class BasicExt
{
    public static bool IsBlank(this string? s) => string.IsNullOrWhiteSpace(s);

    public static int Clamp(this int n, int min, int max) => Math.Clamp(n, min, max);

    public static void ForEach<T>(this IEnumerable<T> src, Action<T> action)
    {
        foreach (var item in src)
            action(item);
    }

    public static string ToCsv(this IEnumerable<string> parts) => string.Join(',', parts);
}
```

## 13. Đáp án thay thế

Bài 3 nhiều style guide **không** khuyến khích ForEach extension (side-effect che giấu) — dùng foreach statement. Clamp tự viết if cho mục đích học.

## 14. Thử thách

Viết `Pipe<TIn, TOut>(this TIn input, Func<TIn, TOut> f)` fluent: `x.Pipe(a => …).Pipe(b => …)`.

## 15. Ứng dụng thực tế

- Toàn bộ LINQ
- FluentValidation-style helpers
- Domain extensions trên entity không sửa được (generated code)
- Unity extension cho `Vector3`, `Transform` (cẩn thận allocate)

## 16. Liên hệ Unity

- `transform.ResetLocal()` extension
- `GameObject.GetOrAddComponent<T>()`
- Tránh extension gọi `Find` mỗi frame
- Đặt file Extensions trong asmdef rõ ràng

## 17. Kiểm tra kiến thức

1. Tham số `this` đứng ở đâu?  
   **Đáp án:** Tham số đầu tiên của static method trong static class.

2. Có cần `using` không?  
   **Đáp án:** Có — namespace chứa extension class phải được import (cùng namespace thì không).

3. Instance method và extension trùng tên — cái nào thắng?  
   **Đáp án:** Instance method.

4. LINQ `Where` là gì về bản chất?  
   **Đáp án:** Extension method trên `IEnumerable<T>` (và `IQueryable`).

5. Extension có truy cập private của kiểu gốc không?  
   **Đáp án:** Không (như static method ngoài class đó).
