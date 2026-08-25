# Chương 3 — Nullable reference types (NRT)

## 1. Mục tiêu học

- Bật **nullable reference types** (`#nullable enable` / project setting)
- Phân biệt `string` vs `string?`, warning CS8600–CS8625 cơ bản
- Dùng `!` (null-forgiving), `?`, `??`, `??=`, null-conditional an toàn
- Viết API rõ ràng về “có thể null” vs “không null” (.NET 8+)

## 2. Điều kiện tiên quyết

- Level 1: reference type, null
- Level 3: value vs reference
- Quen warning compiler

## 3. Khái niệm

Trước NRT, `string name` vẫn có thể là `null` — compiler không bắt. **NRT** thêm annotation:

| Khai báo | Ý nghĩa khi NRT bật |
|----------|---------------------|
| `string s` | Không nên null |
| `string? s` | Có thể null |
| `T?` với class | Nullable reference |
| `T?` với struct | `Nullable<T>` (cũ) |

Đây là **static analysis** — runtime vẫn cho phép null nếu bạn ép/`!`. Mục tiêu: giảm `NullReferenceException`.

Bật toàn project (SDK style):

```xml
<Nullable>enable</Nullable>
```

Hoặc từng file: `#nullable enable`.

## 4. Mô hình tư duy

```text
API boundary: ghi rõ nullability
    │
    ├─ string Name     → caller không truyền null; bạn không cần check mỗi chỗ
    └─ string? Note    → phải xử lý null trước khi dùng

Flow analysis:
  if (s is not null) { s.Length }  // trong block: s là string
  s?.Length                        // int?
  s ?? "default"                   // string
```

## 5. Cú pháp

```csharp
#nullable enable

string name = "Ada";
string? maybe = null;

int? len = maybe?.Length;
string shown = maybe ?? "(none)";
maybe ??= "default";

// Null-forgiving — “tôi chắc không null”
string sure = maybe!; // cẩn thận

static string FirstChar(string? s) =>
    s is { Length: > 0 } ? s[0].ToString() : "";
```

Attributes hữu ích: `[NotNullIfNotNull]`, `[MaybeNull]`, `ArgumentNullException.ThrowIfNull` (.NET).

## 6. Ví dụ

### Cơ bản

```csharp
#nullable enable

static void PrintUpper(string? text)
{
    if (text is null)
    {
        Console.WriteLine("(null)");
        return;
    }
    Console.WriteLine(text.ToUpperInvariant());
}
```

### Trung cấp — property và constructor

```csharp
#nullable enable

class User
{
    public string Name { get; }
    public string? Email { get; set; }

    public User(string name, string? email = null)
    {
        ArgumentNullException.ThrowIfNull(name);
        Name = name;
        Email = email;
    }
}
```

### Nâng cao — generic và forgiving có chủ đích

```csharp
#nullable enable

static TValue GetOrAdd<TKey, TValue>(
    Dictionary<TKey, TValue> map,
    TKey key,
    Func<TValue> factory) where TKey : notnull
{
    if (map.TryGetValue(key, out TValue? existing))
        return existing!; // TryGetValue: khi true, existing có giá trị
    var created = factory();
    map[key] = created;
    return created;
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| CS8600 converting null | Gán null vào non-nullable | Đổi thành `T?` hoặc không gán null |
| CS8602 dereference | Dùng `T?` chưa check | `if`, `?`, `??` |
| Lạm dụng `!` | Tắt warning giả | Fix luồng null thật |
| NRT tắt một nửa solution | Inconsistent | `Nullable>enable</Nullable` toàn project |
| `default` trên class | `default` = null | Trả empty / throw / `T?` |

## 8. Gỡ lỗi

1. Đọc mã warning CS86xx — IDE thường gợi ý fix.
2. Hover kiểu: xem `string` hay `string?`.
3. Nếu chắc từ DB/JSON: validate biên (`ThrowIfNull`) thay vì `!` lan tràn.
4. Bật TreatWarningsAsErrors cho nullable dần dần theo folder.

## 9. Best practices

- Bật NRT cho project mới ngay từ đầu.
- Public API: annotate đúng; đừng để `string` nhưng vẫn nhận null.
- Prefer `is null` / pattern hơn `!`.
- `??` và `??=` cho default rõ ràng.
- Document khi `!` bắt buộc (interop, framework cũ).

## 10. Bài tập

**Bài 1** — Method `int Len(string? s)` trả 0 nếu null.

**Bài 2** — Class `Profile` với `string DisplayName` non-null và `string? Bio`.

**Bài 3** — Sửa đoạn: `string s = Get(); Console.WriteLine(s.Length);` với `Get()` trả `string?`.

**Bài 4** — Dùng `ThrowIfNull` trong constructor.

## 11. Gợi ý

- Bài 1: `s?.Length ?? 0`.
- Bài 3: `if (s is null) return;` hoặc `s?.Length`.
- Bài 4: `ArgumentNullException.ThrowIfNull(name);`.

## 12. Đáp án

```csharp
#nullable enable

static int Len(string? s) => s?.Length ?? 0;

class Profile
{
    public string DisplayName { get; }
    public string? Bio { get; set; }
    public Profile(string displayName, string? bio = null)
    {
        ArgumentNullException.ThrowIfNull(displayName);
        DisplayName = displayName;
        Bio = bio;
    }
}

static void Print(string? s)
{
    if (s is null) return;
    Console.WriteLine(s.Length);
}
```

## 13. Đáp án thay thế

`Len` bằng if cổ điển. `Bio` dùng `string Bio { get; set; } = ""` nếu muốn luôn non-null thay vì nullable.

## 14. Thử thách

Bật `<Nullable>enable</Nullable>` trên một console nhỏ 3 file; sửa hết warning không dùng `!` trừ một chỗ interop có comment giải thích.

## 15. Ứng dụng thực tế

- ASP.NET / minimal APIs model binding
- Library public surface
- Giảm NRE production
- Code review dựa trên annotation

## 16. Liên hệ Unity

- Unity/C# version và NRT: tuỳ phiên bản editor — có thể bật theo asmdef
- Nhiều API Unity trả null (`GetComponent` → dùng try-get / null check)
- Serialize field: null trên prefab thiếu reference — `?` phù hợp
- Đừng `!` trên reference inspector chưa gán

## 17. Kiểm tra kiến thức

1. `string?` nghĩa là gì khi NRT bật?  
   **Đáp án:** Reference có thể null; compiler cảnh báo khi dùng thiếu check.

2. NRT có chặn null lúc runtime không?  
   **Đáp án:** Không — chủ yếu phân tích tĩnh (trừ khi bạn tự validate).

3. `!` (null-forgiving) dùng khi nào?  
   **Đáp án:** Khi bạn *chắc* không null mà analyzer không chứng minh được — dùng sparingly.

4. `s ?? "x"` làm gì?  
   **Đáp án:** Nếu `s` null thì lấy `"x"`.

5. Bật NRT toàn project bằng gì?  
   **Đáp án:** `<Nullable>enable</Nullable>` trong csproj.
