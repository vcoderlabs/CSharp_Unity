# Chương 6 — Collection Expressions (C# 12)

## 1. Mục tiêu học

- Viết collection bằng cú pháp `[...]` (C# 12 / .NET 8+)
- Dùng spread `..` để nối collection
- Biết target type: tạo `List`, array, `Span`, interface…
- Phân biệt với collection initializer cũ `{ ... }`

## 2. Điều kiện tiên quyết

- Chương 1–5: Array, List, interface
- Project dùng **LangVersion** đủ C# 12 (SDK .NET 8 mặc định OK)

## 3. Khái niệm

**Collection expression** là cú pháp ngắn tạo collection:

```csharp
int[] a = [1, 2, 3];
List<string> names = ["Ann", "Bob"];
```

Compiler chọn cách tạo phù hợp **kiểu đích** (target typing).

**Spread element** `..expr` chèn toàn bộ phần tử của collection khác:

```csharp
int[] head = [1, 2];
int[] all = [..head, 3, 4]; // 1,2,3,4
```

So với initializer cũ:

```csharp
var old = new List<int> { 1, 2, 3 }; // cần new + kiểu rõ hơn trong vài ngữ cảnh
List<int> neu = [1, 2, 3];           // gọn, dựa target type
```

## 4. Mô hình tư duy

```text
[ e1, e2, ..other, e3 ]
        │
        ▼
Compiler nhìn kiểu bên trái / tham số method
        │
        ├── int[]          → tạo mảng
        ├── List<T>        → tạo List và Add
        ├── Span<T>        → (stackalloc / tạo phù hợp)
        └── IEnumerable<T> → thường tạo mảng bên dưới
```

## 5. Cú pháp

```csharp
// Empty
int[] empty = [];
List<int> emptyList = [];

// Literals + spread
int[] a = [1, 2, 3];
int[] b = [0, ..a, 4];

// Nested / method argument
static void Take(List<int> xs) { }
Take([1, 2, 3]);

// Với interface
IEnumerable<int> seq = [10, 20, 30];
IReadOnlyList<string> ro = ["x", "y"];
```

Yêu cầu: file dự án C# 12+ (ví dụ `net8.0`).

## 6. Ví dụ

### Cơ bản

```csharp
string[] roles = ["Tank", "Healer", "DPS"];
foreach (var r in roles)
    Console.WriteLine(r);
```

### Trung cấp

Ghép buffer:

```csharp
int[] prefix = [0];
int[] body = [1, 2, 3];
int[] suffix = [99];
int[] packet = [..prefix, ..body, ..suffix];
Console.WriteLine(string.Join(',', packet)); // 0,1,2,3,99
```

### Nâng cao

Trả về list mặc định / fallback:

```csharp
static List<string> GetTags(string? raw)
{
    if (string.IsNullOrWhiteSpace(raw))
        return [];
    return [.. raw.Split(',', StringSplitOptions.RemoveEmptyEntries | StringSplitOptions.TrimEntries)];
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| CS9174 / lỗi cú pháp `[` | LangVersion cũ (&lt; 12) | Đổi `net8.0` hoặc set LangVersion 12 |
| Không suy ra kiểu với `var x = [1,2]` | `var` thiếu target type đủ rõ (tùy phiên bản/rule) | Khai báo kiểu tường minh: `int[] x = [1,2]` |
| Nhầm `{1,2}` với `[1,2]` | `{}` là initializer kèm `new` | Collection expression dùng `[]` |
| Spread kiểu không duyệt được | `..` cần enumerable tương thích | Đảm bảo phần tử cùng kiểu đích |

## 8. Gỡ lỗi

1. `dotnet --version` và kiểm tra `TargetFramework` trong `.csproj`.
2. Đọc lỗi compiler: thường chỉ rõ “collection expression” và kiểu đích.
3. Nếu IDE đỏ nhưng build OK (hoặc ngược lại): reload project / đồng bộ SDK.

## 9. Best practices

- Dùng `[...]` cho mảng/list ngắn, test data, giá trị mặc định.
- Prefer kiểu đích rõ: `List<int> x = [1,2]` thay vì phụ thuộc suy luận mơ hồ.
- Spread tốt khi ghép vài đoạn; với vòng lặp lớn vẫn dùng `Add`/`AddRange` có capacity.
- Không thay thế mọi `new List` — chỗ cần capacity/`EnsureCapacity` thì code tường minh rõ hơn.

## 10. Bài tập

**Bài 1** — Tạo `int[]` gồm số 1..5 bằng collection expression.

**Bài 2** — Cho `int[] a` và `int[] b`, tạo `int[]` = a rồi b (spread).

**Bài 3** — Method `WithDefaults(List<string>? tags)` trả về tags nếu không null/rỗng, ngược lại `["default"]` (kiểu `List<string>`).

**Bài 4** — Tạo `IReadOnlyList<int>` từ nhiều mảng nhỏ bằng một expression có spread.

## 11. Gợi ý

- Bài 1: `int[] x = [1, 2, 3, 4, 5];`
- Bài 2: `int[] c = [..a, ..b];`
- Bài 3: kiểm null/Count rồi `return ["default"];` hoặc `return tags;`
- Bài 4: `IReadOnlyList<int> all = [..a, ..b, ..c];`

## 12. Đáp án

**Bài 1** — Target type là mảng:

```csharp
int[] nums = [1, 2, 3, 4, 5];
```

**Bài 2** — Nối bằng spread:

```csharp
static int[] Concat(int[] a, int[] b) => [..a, ..b];
```

**Bài 3** — Fallback list một phần tử:

```csharp
static List<string> WithDefaults(List<string>? tags)
{
    if (tags is null || tags.Count == 0)
        return ["default"];
    return tags;
}
```

**Bài 4** — Target interface read-only:

```csharp
static IReadOnlyList<int> Merge(int[] a, int[] b, int[] c) => [..a, ..b, ..c];
```

## 13. Đáp án thay thế

Bài 2 kiểu cũ (vẫn đúng, dài hơn):

```csharp
static int[] ConcatOld(int[] a, int[] b)
{
    var result = new int[a.Length + b.Length];
    a.CopyTo(result, 0);
    b.CopyTo(result, a.Length);
    return result;
}
```

## 14. Thử thách

Viết helper `static T[] OnceOrMany<T>(T first, params T[] rest)` bằng collection expression + spread, và so sánh IL/`Benchmark` sơ bộ với `new List` + `ToArray` (tự đo bằng `Stopwatch` cũng được).

## 15. Ứng dụng thực tế

- Test data gọn trong unit test
- Cấu hình mặc định: `allowedRoles = ["Admin", "User"]`
- Ghép header/body/footer buffer khi build message

## 16. Liên hệ Unity

- Unity phiên bản editor/runtime có thể **chưa** C# 12 đầy đủ tùy version — kiểm tra trước khi dùng trong script game
- Vẫn học trên console .NET 8; khi port sang Unity, có thể đổi lại `new List<T> { ... }` nếu compiler Unity chưa hỗ trợ
- Package / tool editor dùng .NET mới hơn gameplay scripts đôi khi dùng được sớm hơn

## 17. Kiểm tra kiến thức

1. Collection expression dùng dấu gì?  
   **Đáp án:** Dấu ngoặc vuông `[...]`.

2. `..collection` nghĩa là gì?  
   **Đáp án:** Spread — chèn tất cả phần tử của collection đó.

3. Cần C# phiên bản nào?  
   **Đáp án:** C# 12 trở lên (.NET 8+ mặc định).

4. `int[] x = []` tạo gì?  
   **Đáp án:** Mảng rỗng độ dài 0.

5. Khác collection initializer `{ }` chỗ nào?  
   **Đáp án:** Initializer đi với `new Type`; collection expression dựa target type và hỗ trợ spread.
