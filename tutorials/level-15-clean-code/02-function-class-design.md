# Chương 2 — Thiết kế hàm và class

## 1. Mục tiêu học

- Giữ **hàm nhỏ**, làm **một việc** (ở mức abstraction nhất quán)
- Hạn chế tham số (≤ 3 lý tưởng; dùng object khi nhiều)
- Class có **một lý do chính** để thay đổi (SRP thực dụng)
- Tách “orchestration” khỏi “chi tiết”

## 2. Điều kiện tiên quyết

- Chương 1: naming
- Level 2: class, access modifier
- Nên có L14 để test sau khi tách hàm

## 3. Khái niệm

### Hàm sạch (thực dụng)

- Tên rõ + body ngắn đến mức đọc một hơi
- Một mức trừu tượng: đừng xen `SqlCommand` với rule giảm giá trong cùng 40 dòng không tách
- Ít side-effect bất ngờ: `GetTotal()` không được xóa file

### Class sạch (thực dụng)

- Nhóm hành vi + dữ liệu liên quan
- Không phải “God class” biết mọi feature
- Public API mỏng; chi tiết private

**Trước:**

```csharp
public class GameManager
{
    public void Run(string user, string pass, string item, int qty, decimal price)
    {
        // login + inventory + payment + save + email...
    }
}
```

**Sau:** tách `AuthService`, `InventoryService`, `CheckoutService`; `GameSession` chỉ điều phối.

## 4. Mô hình tư duy

```text
Đọc hàm như mục lục:
  PlaceOrder()
    Validate(order)
    ReserveStock(order)
    Charge(order)
    Publish(orderPlaced)

Mỗi bước = private method hoặc service khác.
Nếu không đặt tên được bước → chưa hiểu việc đang làm.
```

## 5. Cú pháp

```csharp
public sealed class CheckoutService
{
    private readonly IInventory _inventory;
    private readonly IPaymentGateway _payments;

    public CheckoutService(IInventory inventory, IPaymentGateway payments)
    {
        _inventory = inventory;
        _payments = payments;
    }

    public Receipt PlaceOrder(Order order)
    {
        EnsureValid(order);
        _inventory.Reserve(order.Sku, order.Quantity);
        var charge = _payments.Charge(order.CustomerId, order.Total);
        return new Receipt(order.Id, charge.TransactionId);
    }

    private static void EnsureValid(Order order)
    {
        if (order.Quantity <= 0)
            throw new ArgumentOutOfRangeException(nameof(order.Quantity));
    }
}
```

Parameter object:

```csharp
public sealed class CreateUserRequest
{
    public required string Email { get; init; }
    public required string DisplayName { get; init; }
}
```

## 6. Ví dụ

### Cơ bản — Tách hàm dài

**Trước:**

```csharp
public decimal Bill(List<Item> items, bool vip)
{
    decimal t = 0;
    foreach (var i in items) t += i.Price;
    if (vip) t *= 0.9m;
    if (t > 1000) t -= 50;
    return t;
}
```

**Sau:**

```csharp
public decimal CalculateBill(IEnumerable<Item> items, bool isVip)
{
    decimal subtotal = SumPrices(items);
    decimal afterVip = ApplyVipDiscount(subtotal, isVip);
    return ApplyBulkBonus(afterVip);
}

private static decimal SumPrices(IEnumerable<Item> items)
    => items.Sum(i => i.Price);

private static decimal ApplyVipDiscount(decimal subtotal, bool isVip)
    => isVip ? subtotal * 0.9m : subtotal;

private static decimal ApplyBulkBonus(decimal total)
    => total > 1000m ? total - 50m : total;
```

### Trung cấp — Giảm tham số

**Trước:** `void Draw(int x, int y, int w, int h, string color, bool fill)`

**Sau:** `void Draw(Rectangle bounds, DrawStyle style)`

### Nâng cao — Class theo thay đổi

Nếu rule thuế đổi độc lập giá ship → đừng nhét cả hai vào một class `OrderStuff`.  
`TaxCalculator` và `ShippingCalculator` tách; `OrderPricing` gọi cả hai.

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Hàm 200 dòng | Sợ tạo method | Extract method |
| 7 tham số boolean | “Feature flag” tập trung | Split method / flags object |
| Class `*Manager` biết hết | Thiếu ranh giới | Tách theo use case |
| Getter có side-effect | Thiết kế xấu | Đổi tên `Load`/`Fetch` hoặc tách |
| Deep nesting `if` | Thiếu guard clause | Return sớm |

## 8. Gỡ lỗi

1. Không hiểu hàm → viết lại outline 5 dòng comment mục lục, rồi extract.
2. Bug chỉ ở nhánh VIP → tách `ApplyVipDiscount` dễ unit test.
3. Diff PR quá lớn vì “tách hết một lần” → tách từng hàm + test xanh.
4. Công cụ IDE: Extract Method.

## 9. Best practices

- Guard clause đầu hàm.
- Tránh output param (`out`) nếu trả về tuple/record rõ hơn.
- `private` mặc định cho helper.
- Class `sealed` khi không thiết kế kế thừa.
- Command–Query: hỏi (`Get`) không nên đổi state; lệnh (`Place`) có thể.
- Đừng tách hàm 1 dòng vô nghĩa chỉ để “trông clean”.

## 10. Bài tập

**Bài 1** — Refactor hàm in báo cáo 40 dòng giả định (login check + build string + write file) thành 3 method.

**Bài 2** — Thay `Create(string a, string b, string c, int d, bool e)` bằng request object.

**Bài 3** — Viết lại bằng guard clause:

```csharp
decimal? Bonus(Employee e)
{
    if (e != null)
    {
        if (e.IsActive)
        {
            if (e.Years >= 5) return 1000m;
        }
    }
    return null;
}
```

**Bài 4** — Đề xuất 2 class tách từ `UserOrderPaymentEmailService`.

## 11. Gợi ý

- Bài 1: `EnsureAuthenticated`, `BuildReport`, `WriteReport`.
- Bài 2: record `CreateOrderRequest`.
- Bài 3: if null return; if !active return; if years < 5 return; return 1000.
- Bài 4: `OrderService`, `PaymentService`, `EmailNotifier`…

## 12. Đáp án

**Bài 3:**

```csharp
decimal? CalculateLoyaltyBonus(Employee? employee)
{
    if (employee is null) return null;
    if (!employee.IsActive) return null;
    if (employee.Years < 5) return null;
    return 1000m;
}
```

**Bài 2:**

```csharp
public sealed record CreateOrderRequest(
    string CustomerId,
    string Sku,
    int Quantity,
    bool ExpressShipping);

public Order Create(CreateOrderRequest request) { ... }
```

**Bài 4 (ví dụ):** `OrderService` + `PaymentService` + `OrderEmailComposer`.

## 13. Đáp án thay thế

Dùng early exception thay `null` return nếu “không đủ điều kiện” là lỗi. Pipeline/strategy nếu rule bonus phức tạp dần.

## 14. Thử thách

Chọn một MonoBehaviour hoặc class 150+ dòng — extract 3 private methods và (nếu có) 1 class mới mà không đổi hành vi quan sát được.

## 15. Ứng dụng thực tế

- Review PR dễ hơn khi hàm ngắn
- Unit test nhắm đúng nhánh discount/tax
- On-call đọc stack trace: tên method = bản đồ

## 16. Liên hệ Unity

- `Update()` mỏng: gọi `movement.Tick`, `combat.Tick`
- Tránh một `Player.cs` 2000 dòng — tách components
- ScriptableObject cho data; service cho behavior

## 17. Kiểm tra kiến thức

1. SRP thực dụng nói gì về class?  
   **Đáp án:** Một lý do chính để thay đổi / một trách nhiệm rõ.

2. Vì sao nhiều tham số boolean xấu?  
   **Đáp án:** Khó đọc tại call site; dễ nhầm thứ tự; hàm làm nhiều biến thể.

3. Guard clause là gì?  
   **Đáp án:** Return/throw sớm để giảm lồng nhau.

4. `GetTotal()` nên có side-effect xóa cache file không?  
   **Đáp án:** Không — gây bất ngờ; tách lệnh rõ tên.

5. Parameter object giúp gì?  
   **Đáp án:** Gom tham số liên quan, dễ mở rộng, đọc call site rõ hơn.
