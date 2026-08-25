# Chương 9 — String và StringBuilder

## 1. Mục tiêu học

- Thành thạo thao tác `string` cơ bản: tìm, cắt, thay, tách, format
- Hiểu tính **bất biến (immutable)** của `string` và hệ quả hiệu năng
- Dùng `StringBuilder` khi nối chuỗi nhiều lần trong vòng lặp

## 2. Điều kiện tiên quyết

- Biết vòng lặp và method (Chương 6–7)
- Biết `char` và duyệt chuỗi (Chương 4)

## 3. Khái niệm

`string` trong .NET là dãy `char` Unicode, **immutable**: mọi thao tác “sửa” thực chất tạo chuỗi mới.

Hệ quả: trong vòng lặp `s += x` lặp N lần có thể tạo nhiều object trung gian → chậm và tốn bộ nhớ khi N lớn.

**`StringBuilder`** là bộ đệm ký tự **mutable** trong `System.Text`: `Append`, `Insert`, `Remove`, `Replace`, rồi `ToString()` một lần ở cuối.

Công cụ hay dùng trên `string`:

- `Length`, index `s[i]`
- `Contains`, `StartsWith`, `EndsWith`, `IndexOf`
- `Substring`, `Trim`, `ToLower` / `ToUpper` / `ToLowerInvariant`
- `Split`, `Join`, `Replace`
- Interpolation `$"..."`, `string.Format`, raw string `"""..."""` (C# 11+)

## 4. Mô hình tư duy

```text
string:        [H][e][l][l][o]   ← không sửa tại chỗ
s += "!"  ⇒    tạo object mới "Hello!"

StringBuilder: buffer có thể phình
  Append...Append... → ToString() một lần
```

Quy tắc ngón tay cái: nối **vài** chuỗi → `+` / `$"..."` ổn; nối **trong loop nhiều lần** → `StringBuilder`.

## 5. Cú pháp (C# thật)

```csharp
using System.Text;

string name = "  Lan  ";
string cleaned = name.Trim();
bool ok = cleaned.StartsWith("L");

string[] parts = "a,b,c".Split(',');
string joined = string.Join("-", parts);

var sb = new StringBuilder();
sb.Append("Hello");
sb.Append(' ');
sb.Append("World");
string result = sb.ToString();

string path = $@"C:\Users\{cleaned}\docs"; // hoặc raw string
```

## 6. Ví dụ

### Cơ bản

Đếm từ (tách theo khoảng trắng):

```csharp
Console.Write("Câu: ");
string sentence = Console.ReadLine() ?? "";
string[] words = sentence.Split(' ', StringSplitOptions.RemoveEmptyEntries);
Console.WriteLine($"Số từ: {words.Length}");
```

### Trung cấp

Chuẩn hóa email đơn giản:

```csharp
Console.Write("Email: ");
string email = (Console.ReadLine() ?? "").Trim().ToLowerInvariant();
bool valid = email.Contains('@') && email.Contains('.') && !email.Contains(' ');
Console.WriteLine(valid ? email : "Email không hợp lệ");
```

### Nâng cao

Xây bảng số bằng `StringBuilder` (tránh `+=` trong loop):

```csharp
using System.Text;

Console.Write("N = ");
int n = int.Parse(Console.ReadLine()!);
var sb = new StringBuilder();
for (int i = 1; i <= n; i++)
{
    if (i > 1) sb.Append(", ");
    sb.Append(i);
}
Console.WriteLine(sb.ToString());
```

## 7. Lỗi thường gặp

| Lỗi | Nguyên nhân | Cách xử lý |
|-----|-------------|------------|
| `NullReferenceException` | Chuỗi null gọi method | `?? ""` / `string.IsNullOrEmpty` |
| So sánh sai văn hóa | `ToLower()` phụ thuộc culture | Prefer `ToLowerInvariant` / `StringComparison.OrdinalIgnoreCase` |
| `Substring` out of range | Index/length sai | Kiểm `Length` trước |
| Chậm với `+=` loop lớn | Immutable | Đổi sang `StringBuilder` |
| `Split` ra phần tử trống | Nhiều delimiter liên tiếp | `RemoveEmptyEntries` |

## 8. Gỡ lỗi

1. In `s.Length` và `s` trong dấu ngoặc kép để thấy space ẩn: `$"[{s}]"`.
2. Với bug so sánh: thử `StringComparison.Ordinal`.
3. Đo nhanh: so thời gian `+=` vs `StringBuilder` với N = 100_000 (Stopwatch — Chương 10 liên quan thời gian).
4. Nhớ `Trim()` khi đọc input người dùng.

## 9. Best practices

- Dùng `$"..."` cho format dễ đọc.
- So sánh không phân biệt hoa thường: `string.Equals(a, b, StringComparison.OrdinalIgnoreCase)`.
- `string.IsNullOrWhiteSpace` trước khi xử lý input.
- `StringBuilder` khi xây chuỗi lớn / loop; không cần cho mọi chỗ chỉ nối 2–3 mảnh.

## 10. Bài tập

**Bài 1 — Đảo từ**  
*Input:* một câu.  
*Output:* đảo thứ tự các từ (`"one two three"` → `"three two one"`).

**Bài 2 — Đếm ký tự không space**  
*Input:* chuỗi.  
*Output:* số ký tự sau khi loại mọi whitespace.

**Bài 3 — Ẩn email**  
*Input:* email dạng `local@domain`.  
*Output:* `l***@domain` (giữ ký tự đầu của local, còn lại `*`). Input sai → thông báo.

**Bài 4 — StringBuilder bảng cửu chương**  
*Input:* `n` (2–9).  
*Output:* in bảng `n x 1..10` xây bằng `StringBuilder` (có xuống dòng).

**Bài 5 — Palindrome**  
*Input:* chuỗi.  
*Output:* `true`/`false` sau khi bỏ space và không phân biệt hoa thường.

## 11. Gợi ý

- Bài 1: `Split` → duyệt ngược → `string.Join`.
- Bài 3: `Split('@')` phải đúng 2 phần.
- Bài 5: hai con trỏ `i`, `j` hoặc so với chuỗi đảo.

## 12. Đáp án

**Bài 1** — Đảo từ:

```csharp
Console.Write("Câu: ");
string[] words = (Console.ReadLine() ?? "")
    .Split(' ', StringSplitOptions.RemoveEmptyEntries);
Array.Reverse(words);
Console.WriteLine(string.Join(" ", words));
```

**Bài 2** — Đếm không whitespace:

```csharp
Console.Write("Chuỗi: ");
string s = Console.ReadLine() ?? "";
int count = 0;
foreach (char c in s)
{
    if (!char.IsWhiteSpace(c)) count++;
}
Console.WriteLine(count);
```

**Bài 3** — Ẩn email:

```csharp
Console.Write("Email: ");
string email = (Console.ReadLine() ?? "").Trim();
string[] parts = email.Split('@');
if (parts.Length != 2 || parts[0].Length == 0 || parts[1].Length == 0)
{
    Console.WriteLine("Email không hợp lệ");
    return;
}
string local = parts[0];
string masked = local[0] + new string('*', Math.Max(0, local.Length - 1));
Console.WriteLine($"{masked}@{parts[1]}");
```

**Bài 4** — Bảng cửu chương:

```csharp
using System.Text;

Console.Write("n = ");
int n = int.Parse(Console.ReadLine()!);
var sb = new StringBuilder();
for (int i = 1; i <= 10; i++)
{
    sb.AppendLine($"{n} x {i} = {n * i}");
}
Console.Write(sb.ToString());
```

**Bài 5** — Palindrome:

```csharp
Console.Write("Chuỗi: ");
string raw = Console.ReadLine() ?? "";
var sb = new System.Text.StringBuilder();
foreach (char c in raw)
{
    if (!char.IsWhiteSpace(c)) sb.Append(char.ToLowerInvariant(c));
}
string s = sb.ToString();
bool ok = true;
for (int i = 0, j = s.Length - 1; i < j; i++, j--)
{
    if (s[i] != s[j]) { ok = false; break; }
}
Console.WriteLine(ok);
```

## 13. Đáp án thay thế

Palindrome một dòng ý tưởng:

```csharp
string s = new string((Console.ReadLine() ?? "")
    .Where(c => !char.IsWhiteSpace(c))
    .Select(char.ToLowerInvariant)
    .ToArray());
string rev = new string(s.Reverse().ToArray());
Console.WriteLine(s == rev);
```

(Cần LINQ — xem trước Level 8.)

## 14. Thử thách

Viết `string Compress(string s)` kiểu run-length đơn giản: `"aaabb"` → `"a3b2"`. Nếu chuỗi nén không ngắn hơn thì trả nguyên gốc. Dùng `StringBuilder`.

## 15. Ứng dụng thực tế

Log message, template email, CSV/JSON thủ công, path URL, sanitize input — xử lý string chiếm phần lớn code business. Hiểu immutable giúp tránh lag khi build báo cáo lớn.

## 16. Liên hệ Unity

UI Text, tên asset, serialize nhẹ — `string` everywhere. Tránh `+=` trong `Update` mỗi frame. Với build string thường xuyên (combat log), cân nhắc pool/`StringBuilder` hoặc không allocate mỗi frame. So sánh tên: cẩn thận culture khi làm multi-language.

## 17. Kiểm tra kiến thức

1. `string` có mutable không?  
   **Đáp án:** Không — immutable.

2. Khi nào ưu tiên `StringBuilder`?  
   **Đáp án:** Khi nối/biến đổi chuỗi nhiều lần, đặc biệt trong vòng lặp.

3. `Trim()` làm gì?  
   **Đáp án:** Gỡ khoảng trắng đầu/cuối (trả về chuỗi mới).

4. Vì sao `ToLowerInvariant` thường an toàn hơn `ToLower` cho logic?  
   **Đáp án:** Không phụ thuộc culture máy người dùng.

5. `string.IsNullOrWhiteSpace` khác `IsNullOrEmpty` chỗ nào?  
   **Đáp án:** Còn coi chuỗi chỉ gồm whitespace như “trống”.
