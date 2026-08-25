# Chương 4 — Kiểu dữ liệu nguyên thủy: số, bool, char, string, enum

## 1. Mục tiêu học

- Chọn đúng kiểu số (`int`, `long`, `float`, `double`, `decimal`, …)
- Dùng `bool`, `char`, `string` và hiểu khác biệt `char` vs chuỗi 1 ký tự
- Khai báo và dùng `enum` cho tập giá trị có tên

## 2. Điều kiện tiên quyết

- Thành thạo khai báo biến / `const` / `var` (Chương 3)
- Biết chương trình console đọc/ghi cơ bản

## 3. Khái niệm

C# là ngôn ngữ **strong typing**: mọi giá trị thuộc một kiểu. Các kiểu số nguyên và dấu phẩy là **value types** (lưu trực tiếp giá trị).

| Nhóm | Kiểu hay dùng | Ghi chú |
|------|---------------|---------|
| Nguyên | `int`, `long`, `byte`, `short` | `int` đủ cho hầu hết đếm |
| Thực | `float`, `double` | `double` mặc định cho số `3.14` |
| Tiền tệ / độ chính xác cao | `decimal` | Chậm hơn nhưng chính xác thập phân |
| Logic | `bool` | chỉ `true` / `false` |
| Ký tự | `char` | 1 Unicode character, literal `'A'` |
| Chuỗi | `string` | dãy ký tự, literal `"hello"` (reference type) |
| Liệt kê | `enum` | tập tên có thứ tự/ngầm số nguyên |

**Ép kiểu (cast):** ` (int)3.9` cắt phần thập phân. Chuyển an toàn hơn: `Convert`, `Parse`, `TryParse`.

## 4. Mô hình tư duy

```text
Giá trị trong đầu bạn  →  chọn “hộp” đúng kích thước/kiểu
  tuổi          → int
  tiền VNĐ lẻ   → decimal (hoặc long xu)
  bật/tắt       → bool
  phím 'Y'      → char
  họ tên        → string
  trạng thái    → enum (Idle, Run, Jump)
```

`enum` = “số có tên” — dễ đọc hơn `if (state == 3)`.

## 5. Cú pháp (C# thật)

```csharp
int lives = 3;
long population = 8_000_000_000L;
float speed = 3.5f;
double pi = 3.1415926535;
decimal price = 199_000.50m;

bool isReady = true;
char grade = 'A';
string name = "An";

enum Direction { North, East, South, West }
Direction d = Direction.North;

enum ItemRarity : byte { Common = 1, Rare = 2, Epic = 5 }
```

## 6. Ví dụ

### Cơ bản

In kích thước và min/max của `int`:

```csharp
Console.WriteLine($"sizeof(int) via sizeof: {sizeof(int)} bytes");
Console.WriteLine($"int min={int.MinValue}, max={int.MaxValue}");
Console.WriteLine($"double.Epsilon (rất nhỏ) ≈ {double.Epsilon}");
```

### Trung cấp

Chuyển chuỗi → số an toàn và dùng `bool` quyết định:

```csharp
Console.Write("Nhập tuổi: ");
bool ok = int.TryParse(Console.ReadLine(), out int age);
bool canVote = ok && age >= 18;
Console.WriteLine(canVote ? "Đủ tuổi bầu cử (minh họa)" : "Không hợp lệ hoặc chưa đủ tuổi");
```

### Nâng cao

`enum` + `switch` (xem thêm Chương 6) mô phỏng hướng nhân vật:

```csharp
enum Dir { North, East, South, West }

static (int x, int y) Step(Dir dir, int x, int y) => dir switch
{
    Dir.North => (x, y + 1),
    Dir.East  => (x + 1, y),
    Dir.South => (x, y - 1),
    Dir.West  => (x - 1, y),
    _ => (x, y)
};

var pos = Step(Dir.East, 0, 0);
Console.WriteLine($"Pos = ({pos.x}, {pos.y})");
```

## 7. Lỗi thường gặp

| Lỗi | Nguyên nhân | Cách xử lý |
|-----|-------------|------------|
| Literal `3.14` gán `float` | Mặc định là `double` | Thêm `f`: `3.14f` |
| `decimal` thiếu `m` | Sai kiểu literal | `10.5m` |
| So sánh `float` bằng `==` | Sai số học dấu phẩy | So sánh với epsilon / redesign |
| `char` dùng nháy kép | `"A"` là `string` | Dùng `'A'` |
| `enum` không parse từ input | Nhập chữ thô | `Enum.TryParse` |

## 8. Gỡ lỗi

1. In `value.GetType().Name` khi nghi sai kiểu.
2. Overflow `int`: bật `checked { }` khi học để thấy exception.
3. Với tiền tệ: thử `0.1 + 0.2` bằng `double` vs `decimal` để thấy khác biệt.
4. `enum`: in `(int)myEnum` để xem giá trị số ngầm.

## 9. Best practices

- Mặc định: `int`, `double`, `bool`, `string`; `decimal` cho tiền; `long` khi vượt `int`.
- Tránh magic number — dùng `enum` hoặc `const`.
- Prefer `TryParse` hơn `Parse` với input người dùng.
- Đặt tên `enum` PascalCase, giá trị PascalCase (`OrderStatus.Pending`).

## 10. Bài tập

**Bài 1 — Phân loại số**  
*Input:* một `int`.  
*Output:* in `âm` / `không` / `dương`.

**Bài 2 — Điểm chữ**  
*Input:* điểm `double` 0–10.  
*Output:* `char` xếp loại thô: `>=8 → 'A'`, `>=6.5 → 'B'`, `>=5 → 'C'`, else `'D'`. Sai input → thông báo.

**Bài 3 — Tiền tệ `decimal`**  
*Input:* đơn giá và số lượng.  
*Output:* thành tiền = giá * lượng, format tiền Việt giả lập (`F0` hoặc `N0`).

**Bài 4 — Enum mùa**  
*Input:* số 1–4.  
*Output:* map sang `enum Season { Spring=1, Summer, Autumn, Winter }` và in tên mùa (tiếng Việt tùy bạn).

**Bài 5 — Đếm nguyên âm**  
*Input:* một chuỗi.  
*Output:* số lượng ký tự thuộc `aeiouAEIOU` (duyệt `char` trong `string`).

## 11. Gợi ý

- Bài 2: nhánh `if/else if` hoặc biểu thức điều kiện lồng.
- Bài 4: `Enum.IsDefined(typeof(Season), number)`.
- Bài 5: `foreach (char c in text)`.

## 12. Đáp án

**Bài 1** — Phân loại dấu:

```csharp
Console.Write("n = ");
int n = int.Parse(Console.ReadLine()!);
if (n < 0) Console.WriteLine("âm");
else if (n == 0) Console.WriteLine("không");
else Console.WriteLine("dương");
```

**Bài 2** — Điểm → chữ:

```csharp
Console.Write("Điểm: ");
if (!double.TryParse(Console.ReadLine(), out double score) || score < 0 || score > 10)
{
    Console.WriteLine("Điểm không hợp lệ");
    return;
}

char grade = score >= 8 ? 'A' : score >= 6.5 ? 'B' : score >= 5 ? 'C' : 'D';
Console.WriteLine($"Xếp loại: {grade}");
```

**Bài 3** — Thành tiền:

```csharp
Console.Write("Đơn giá: ");
decimal price = decimal.Parse(Console.ReadLine()!);
Console.Write("Số lượng: ");
int qty = int.Parse(Console.ReadLine()!);
decimal total = price * qty;
Console.WriteLine($"Thành tiền: {total:N0}");
```

**Bài 4** — Mùa:

```csharp
enum Season { Spring = 1, Summer, Autumn, Winter }

Console.Write("1-4: ");
if (!int.TryParse(Console.ReadLine(), out int v) || !Enum.IsDefined(typeof(Season), v))
{
    Console.WriteLine("Không hợp lệ");
    return;
}

Season s = (Season)v;
string vi = s switch
{
    Season.Spring => "Xuân",
    Season.Summer => "Hạ",
    Season.Autumn => "Thu",
    Season.Winter => "Đông",
    _ => "?"
};
Console.WriteLine(vi);
```

**Bài 5** — Đếm nguyên âm:

```csharp
Console.Write("Chuỗi: ");
string text = Console.ReadLine() ?? "";
const string vowels = "aeiouAEIOU";
int count = 0;
foreach (char c in text)
{
    if (vowels.Contains(c)) count++;
}
Console.WriteLine($"Số nguyên âm: {count}");
```

## 13. Đáp án thay thế

Bài 5 dùng LINQ (xem trước Level 8 — chỉ tham khảo):

```csharp
int count = text.Count(c => "aeiouAEIOU".Contains(c));
```

## 14. Thử thách

Tạo `enum WeaponType { Sword, Bow, Staff }` và in sát thương cơ bản theo từng loại (dùng `switch`). Nhập tên enum từ bàn phím bằng `Enum.TryParse` (ignoreCase).

## 15. Ứng dụng thực tế

API, DTO, cấu hình feature flag (`bool`), mã tiền (`decimal`), trạng thái đơn hàng (`enum`) — chọn sai kiểu (ví dụ `double` cho tiền) gây lỗi làm tròn khó tìm.

## 16. Liên hệ Unity

Unity hay dùng `float` (engine theo single precision), `bool` cho flag, `string` cho UI tạm, `enum` cho trạng thái animation / layer. `decimal` gần như không dùng trong gameplay realtime. `char` ít gặp hơn `string` / KeyCode.

## 17. Kiểm tra kiến thức

1. Literal `3.14` mặc định thuộc kiểu nào?  
   **Đáp án:** `double`

2. Vì sao tiền nên dùng `decimal`?  
   **Đáp án:** Chính xác thập phân hơn `float`/`double`, tránh sai số làm tròn tiền.

3. `'A'` và `"A"` khác gì?  
   **Đáp án:** `char` vs `string`.

4. `enum` giúp gì?  
   **Đáp án:** Đặt tên cho tập giá trị cố định, code dễ đọc/an toàn hơn số thô.

5. `TryParse` khác `Parse` chỗ nào?  
   **Đáp án:** `TryParse` không ném exception khi sai định dạng; trả `bool` thành công/thất bại.
