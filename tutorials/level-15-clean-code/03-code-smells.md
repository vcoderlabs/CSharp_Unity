# Chương 3 — Code Smells

## 1. Mục tiêu học

- Nhận diện **code smell** phổ biến trong C#
- Phân biệt smell (gợi ý) với bug (sai hành vi)
- Biết hướng xử lý tương ứng (chi tiết refactor ở chương 4)
- Ưu tiên smell gây đau thật sự, không “săn phù thủy”

## 2. Điều kiện tiên quyết

- Chương 1–2
- Đã từng sửa code “sợ đụng vào”

## 3. Khái niệm

**Code smell** = dấu hiệu thiết kế/cấu trúc có thể gây khó bảo trì. Không phải lúc nào cũng phải sửa ngay — nhưng cần biết nhận ra.

| Smell | Dấu hiệu | Hướng xử lý |
|-------|----------|-------------|
| Long Method | Hàm quá dài / nhiều việc | Extract Method |
| Large Class | Class quá nhiều field/method | Extract Class |
| Long Parameter List | Nhiều tham số | Parameter Object |
| Duplicated Code | Copy-paste | Extract / template |
| Feature Envy | Method dùng data class khác nhiều hơn của mình | Move Method |
| Data Clumps | Nhóm field luôn đi cùng nhau | Object gộp |
| Switch / If hung thần | Phân nhánh theo type xuyên file | Polymorphism / strategy |
| Primitive Obsession | `string`/`int` cho mọi khái niệm | Tiny type / value object |
| Dead Code | Không gọi tới | Xóa |
| Comments che code xấu | Comment giải thích rối | Đổi tên / tách hàm |
| God Class | Biết và làm mọi thứ | Tách module |
| Shotgun Surgery | Một thay đổi sửa nhiều chỗ rải rác | Gộp trách nhiệm |

## 4. Mô hình tư duy

```text
Ngửi thấy mùi → hỏi:
  1. Có đang làm chậm team / gây bug không?
  2. Có test bao quanh không?
  3. Refactor nhỏ nhất giảm mùi là gì?

Smell ≠ bắt buộc pattern nặng.
```

## 5. Cú pháp

Ví dụ **Primitive Obsession → Value object nhẹ**:

```csharp
// Trước: string email khắp nơi, validate copy-paste
public void Register(string email) { ... }

// Sau
public readonly record struct EmailAddress
{
    public string Value { get; }
    public EmailAddress(string value)
    {
        if (string.IsNullOrWhiteSpace(value) || !value.Contains('@'))
            throw new ArgumentException("Invalid email", nameof(value));
        Value = value.Trim();
    }
    public override string ToString() => Value;
}
```

## 6. Ví dụ

### Cơ bản — Duplicated Code

**Trước:**

```csharp
decimal weaponTotal = weapons.Sum(w => w.Price * 1.1m);
decimal armorTotal = armors.Sum(a => a.Price * 1.1m);
```

**Sau:**

```csharp
decimal WithTax(IEnumerable<Item> items) => items.Sum(i => i.Price * 1.1m);
```

### Trung cấp — Feature Envy

**Trước:**

```csharp
public class OrderPrinter
{
    public string Print(Order o)
        => $"{o.Customer.FirstName} {o.Customer.LastName} - {o.Customer.Email}";
}
```

`OrderPrinter` ghen với `Customer`.

**Sau:**

```csharp
public sealed class Customer
{
    public string FormatLabel() => $"{FirstName} {LastName} - {Email}";
}
```

### Nâng cao — Switch theo type lặp lại

**Trước:**

```csharp
decimal Damage(Enemy e) => e.Type switch
{
    "slime" => 1,
    "orc" => 5,
    "dragon" => 20,
    _ => 0
};
```

Thêm enemy mới = sửa nhiều switch.

**Sau (hướng):**

```csharp
public abstract class Enemy
{
    public abstract decimal BaseDamage { get; }
}
public sealed class Slime : Enemy { public override decimal BaseDamage => 1; }
```

Hoặc `IDamageProfile` strategy nếu không muốn hierarchy sâu.

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Xóa dead code… đang dùng reflection | Không search kỹ | Tìm reference + test |
| “Abstract mọi thứ” vì sợ duplication | Over-react DRY | Chờ trùng **cùng ý nghĩa** |
| Comment `// tăng i` | Vô ích | Xóa; giữ comment *vì sao* |
| Smell ignore vì deadline | Nợ kỹ thuật | Ghi TODO có ticket; trả dần |
| Đổi smell thành pattern khổng lồ | Over-engineer | Refactor tối thiểu |

## 8. Gỡ lỗi

1. Bug chỉ sửa được bằng copy thêm một chỗ nữa → đang có duplication/shotgun.
2. Không biết sửa file nào → Large Class / kém cohesion.
3. Dùng quantifiers: số dòng method, số tham số, số `switch` trùng — ưu tiên hotspot.
4. Code review checklist smell 5 phút trước khi merge.

## 9. Best practices

- Ưu tiên smell ở **hot path** / module hay đổi.
- Kết hợp đổi tên trước extract.
- Giữ comment giải thích *lý do nghiệp vụ / ràng buộc*, không kể lại code.
- Đo bằng đau thật (bug, thời gian onboard), không bằng cảm tính “xấu”.
- Học tên smell để giao tiếp team nhanh.

## 10. Bài tập

**Bài 1** — Chỉ ra ≥ 3 smell trong đoạn:

```csharp
public class Do
{
    public string X(string a, string b, int c, int d, bool f)
    {
        // check
        if (a != null)
        {
            if (b != null)
            {
                return a + b + c + d + (f ? "1" : "0");
            }
        }
        return "";
    }
}
```

**Bài 2** — Viết lại đoạn bài 1 sạch hơn (tên + guard + ít tham số hơn nếu hợp lý).

**Bài 3** — Tìm Feature Envy trong code của bạn (hoặc tự viết 1 ví dụ) và Move Method.

**Bài 4** — Cho ví dụ Shotgun Surgery trong game (1–2 câu).

## 11. Gợi ý

- Bài 1: tên xấu, long params, nested if, class `Do`, comment vô ích.
- Bài 2: `FormatCustomerCode(...)` hoặc object request + early return.
- Bài 4: đổi format save → sửa 12 chỗ đọc/ghi rải rác.

## 12. Đáp án

**Bài 1:** Long Parameter List; Poor Naming (`Do`, `X`, `a`…); Nested conditionals; Possibly Primitive Obsession; Meaningless comment.

**Bài 2 (ví dụ):**

```csharp
public sealed class CustomerCodeFormatter
{
    public string Format(CustomerCodeParts parts)
    {
        if (parts.Prefix is null || parts.Suffix is null) return string.Empty;
        string flag = parts.IsPriority ? "1" : "0";
        return $"{parts.Prefix}{parts.Suffix}{parts.RegionId}{parts.BranchId}{flag}";
    }
}
```

**Bài 4:** Thêm field `Quest.Difficulty` phải sửa UI, save, network DTO, analytics — nếu không có một model trung tâm.

## 13. Đáp án thay thế

Giữ primitive nhưng extract validation `EnsureNotNull`. Không bắt buộc value object mọi lúc.

## 14. Thử thách

Lấy 1 file > 200 dòng, gắn một bảng “smell → mức độ đau (1–5) → hành động”. Chỉ refactor mục đau ≥ 4.

## 15. Ứng dụng thực tế

- Tech debt backlog gắn smell cụ thể
- Boy Scout Rule: để lại file sạch hơn một chút
- Phỏng vấn / review: nói chuyện bằng vocabulary chung

## 16. Liên hệ Unity

- God `GameManager` cực phổ biến — tách systems
- Duplicate `GetComponent` pattern → cache / inject
- `Update` phình to = Long Method classic

## 17. Kiểm tra kiến thức

1. Code smell khác bug thế nào?  
   **Đáp án:** Smell là dấu hiệu thiết kế; bug là sai hành vi.

2. Feature Envy là gì?  
   **Đáp án:** Method quan tâm dữ liệu class khác hơn của mình.

3. Shotgun Surgery?  
   **Đáp án:** Một thay đổi buộc sửa nhiều chỗ rải rác.

4. Comment tốt nên nói gì?  
   **Đáp án:** *Vì sao* / ràng buộc — không kể lại từng dòng.

5. Có phải lúc nào cũng xóa duplication ngay?  
   **Đáp án:** Không — chỉ khi trùng *ý nghĩa*; tránh trừu tượng sớm (xem DRY/YAGNI).
