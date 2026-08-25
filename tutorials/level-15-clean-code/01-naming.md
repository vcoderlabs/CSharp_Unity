# Chương 1 — Naming (Đặt tên)

## 1. Mục tiêu học

- Đặt tên biến, hàm, class, tham số **nói lên ý định**
- Tránh tên mơ hồ (`data`, `temp`, `mgr`, `obj`)
- Áp dụng convention C#: PascalCase / camelCase / interface `I...`
- Đổi tên như một bước refactor đầu tiên, rẻ và hiệu quả

## 2. Điều kiện tiên quyết

- Level 1–2: biến, method, class, interface
- Biết đọc code C# cơ bản

## 3. Khái niệm

**Tên tốt** = người đọc hiểu *vai trò* mà không cần đoán từ thân hàm.

| Loại | Convention C# | Ví dụ tốt | Ví dụ xấu |
|------|---------------|-----------|-----------|
| Class / method / property | PascalCase | `OrderService`, `CalculateTotal` | `ord_svc`, `doIt` |
| Local / parameter | camelCase | `itemCount`, `customerId` | `x1`, `p` |
| Interface | `I` + Pascal | `IEmailSender` | `EmailSenderInterface` |
| Private field | `_camel` hoặc không underscore (team) | `_repository` | `r` |
| Constant | PascalCase | `MaxRetryCount` | `MAX_RETRY` (OK nếu team chọn) |

### Trước / sau

```csharp
// Trước
int d; // số ngày?
void Proc(List<int> l) { ... }

// Sau
int overdueDays;
void CloseOverdueInvoices(List<int> invoiceIds) { ... }
```

## 4. Mô hình tư duy

```text
Đọc tên → đoán được trách nhiệm?
  Có → giữ
  Không → đổi tên (hoặc tách vì đang làm nhiều việc)

Tên dài hơn một chút OK nếu rõ.
Tên ngắn OK nếu phạm vi hẹp (i trong for).
```

## 5. Cú pháp

```csharp
public interface IInventoryRepository
{
    Item? FindBySku(string sku);
}

public sealed class InventoryRepository : IInventoryRepository
{
    private readonly List<Item> _items = new();

    public Item? FindBySku(string sku)
        => _items.FirstOrDefault(item => item.Sku == sku);
}

public decimal CalculateDiscountedPrice(decimal unitPrice, decimal discountPercent)
{
    decimal discountFactor = 1m - discountPercent / 100m;
    return unitPrice * discountFactor;
}
```

Boolean: ưu tiên `is`, `has`, `can`, `should`:

```csharp
bool isExpired;
bool hasPermission;
bool canEdit;
```

## 6. Ví dụ

### Cơ bản — Đổi tên biến

**Trước:**

```csharp
var n = users.Where(u => u.A > 18).ToList();
```

**Sau:**

```csharp
var adults = users.Where(user => user.Age > 18).ToList();
```

### Trung cấp — Method nói lên side-effect

**Trước:**

```csharp
void User(int id) { /* xóa user */ }
```

**Sau:**

```csharp
void DeleteUser(int userId) { ... }
```

Tránh động từ mơ hồ `Handle`, `Process`, `Manage` nếu có thể cụ thể hơn: `Process` → `ApplyWeekendPricing`.

### Nâng cao — Tên phản ánh đơn vị / miền

```csharp
// Trước
int Wait(int t);

// Sau
TimeSpan Wait(TimeSpan delay);
// hoặc
void WaitMilliseconds(int delayMilliseconds);
```

Đừng để `int t` mà không biết giây hay ms.

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| `data`, `info`, `manager` | Lười đặt tên | Nói cụ thể domain |
| Tên sai với thân hàm | Đổi code quên đổi tên | Rename cùng lúc |
| Viết tắt khó đoán (`btn`, `ctx` lung tung) | Thói quen | Chỉ viết tắt phổ biến (`id`, `i`) |
| Class `Utils` / `Helpers` khổng lồ | Không có nhà | Tách theo chủ đề `DateTimeUtc`, `PriceMath` |
| Động từ cho class | Nhầm vai trò | Class = danh từ; method = động từ |

## 8. Gỡ lỗi

1. Không hiểu đoạn code? Thử **đặt lại tên** trước khi viết comment dài.
2. IDE Rename (F2) — tránh find-replace thô.
3. Nếu không đặt được tên ngắn gọn → smell: hàm đang làm quá nhiều.
4. Code review: hỏi “tên này có đánh lừa không?”

## 9. Best practices

- Tên theo **ngôn ngữ miền** (game: `Player`, `Quest`; không `Obj1`).
- Tránh pun / meme trong production.
- Độ dài tỉ lệ với scope: biến sống 3 dòng có thể `i`; biến sống cả class cần rõ.
- Đừng encode kiểu trong tên (`strName`) — C# đã có type.
- Interface: `ISomething` mô tả khả năng, không phải `ISomethingImpl`.

## 10. Bài tập

**Bài 1** — Đổi tên: `void Go(string s)`, body gửi email. Đặt lại method + param.

**Bài 2** — Class `Mgr` quản lý danh sách quest. Đặt lại class/method `Add`/`Rm`.

**Bài 3** — Biến `List<decimal> x` là giá các item trong giỏ. Đổi tên + method tính tổng.

**Bài 4** — Boolean `flag` nghĩa là “user đã xác minh email”. Đặt lại.

## 11. Gợi ý

- Bài 1: `SendEmail(string toAddress)` hoặc `SendEmail(string recipientEmail)`.
- Bài 2: `QuestLog` / `QuestRepository`, `Remove`.
- Bài 3: `linePrices`, `CalculateCartTotal`.
- Bài 4: `isEmailVerified`.

## 12. Đáp án

**Bài 1:** `void SendWelcomeEmail(string recipientEmail)`

**Bài 2:**

```csharp
public class QuestLog
{
    public void Add(Quest quest) { ... }
    public bool Remove(string questId) { ... }
}
```

**Bài 3:**

```csharp
decimal CalculateCartTotal(IEnumerable<decimal> linePrices)
    => linePrices.Sum();
```

**Bài 4:** `bool isEmailVerified`

## 13. Đáp án thay thế

Bài 2 có thể `QuestService` nếu có business rules; `Repository` nếu chỉ persistence. Chọn theo trách nhiệm thật.

## 14. Thử thách

Mở một file cũ của bạn (≥ 50 dòng). Rename ít nhất 5 symbol cho rõ — không đổi hành vi. Chạy test nếu có.

## 15. Ứng dụng thực tế

- API public: tên method là hợp đồng với consumer
- Log/metrics: tên rõ giúp ops
- Onboarding: codebase đọc được = onboard nhanh

## 16. Liên hệ Unity

- Tránh `GameObject obj1`; dùng `enemyRoot`, `playerSpawnPoint`
- Method `Update` của Unity giữ tên engine; logic bên trong đặt tên domain (`RegenerateStamina`)
- Script tên file = tên class (`PlayerHealth.cs`)

## 17. Kiểm tra kiến thức

1. Method nên là danh từ hay động từ?  
   **Đáp án:** Động từ / cụm động từ (`CalculateTotal`, `LoadScene`).

2. Vì sao `ProcessData` thường là mùi?  
   **Đáp án:** Không nói xử lý *cái gì* / *thế nào*.

3. Boolean nên đặt thế nào?  
   **Đáp án:** `is`/`has`/`can`… khẳng định được true/false.

4. `I` prefix dùng cho?  
   **Đáp án:** Interface trong convention C# thông dụng.

5. Không đặt nổi tên ngắn gọn thường báo hiệu gì?  
   **Đáp án:** Hàm/class đang mang quá nhiều trách nhiệm.
