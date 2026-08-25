# Project Level 14 — Tested Application

## 1. Mục tiêu học

- Xây solution **production + tests** hoàn chỉnh
- Viết code **injectable** và phủ unit test cho khái niệm từ **Level 1–8**
- Luyện AAA, Theory, fake/mock, exception assert
- Có calculator/service mẫu đủ dùng làm “hồi cố” kiến thức cũ qua test

## 2. Điều kiện tiên quyết

- Hoàn thành 5 chương Level 14
- Level 1–8: biến, OOP, collections, generics, exceptions, delegates/events, LINQ
- .NET 8 SDK

## 3. Khái niệm / Yêu cầu sản phẩm

### Phần A — Solution

```text
TestedApp/
├── src/TestedApp/           # class library (logic)
│   ├── Calculator.cs
│   ├── Pricing/
│   │   ├── IDiscountPolicy.cs
│   │   ├── PercentDiscount.cs
│   │   └── PricingService.cs
│   ├── Inventory/
│   │   ├── IInventoryStore.cs
│   │   ├── InMemoryInventoryStore.cs
│   │   └── InventoryService.cs
│   └── Analytics/
│       └── SalesAnalytics.cs   # LINQ
└── tests/TestedApp.Tests/   # xUnit
```

### Phần B — Calculator (ôn L1: toán, nhánh, exception)

| Method | Hành vi |
|--------|---------|
| `Add/Subtract/Multiply` | Số học cơ bản (`decimal` khuyến nghị) |
| `Divide(a, b)` | `b == 0` → `DivideByZeroException` |
| `Average(IEnumerable<decimal>)` | rỗng → `InvalidOperationException` |

### Phần C — PricingService (ôn L2 OOP + L4 DI mindset)

```csharp
public interface IDiscountPolicy
{
    decimal Apply(decimal price);
}

public class PricingService
{
    private readonly IDiscountPolicy _discount;
    public PricingService(IDiscountPolicy discount) => _discount = discount;

    public decimal FinalPrice(decimal price)
    {
        if (price < 0) throw new ArgumentOutOfRangeException(nameof(price));
        return _discount.Apply(price);
    }
}
```

`PercentDiscount(percent)`: `price * (1 - percent/100)`.

### Phần D — InventoryService (ôn L4 collections + L6 exceptions)

- `Add(sku, qty)`, `Remove(sku, qty)`, `GetQuantity(sku)`
- Remove quá số lượng → exception rõ ràng
- Store qua `IInventoryStore` để mock/fake

### Phần E — SalesAnalytics (ôn L8 LINQ + L5 generics nhẹ)

```csharp
public record Sale(string Category, decimal Amount);

public static class SalesAnalytics
{
    public static decimal Total(IEnumerable<Sale> sales) => ...
    public static IReadOnlyDictionary<string, decimal> TotalByCategory(IEnumerable<Sale> sales) => ...
    public static IEnumerable<Sale> TopN(IEnumerable<Sale> sales, int n) => ...
}
```

### Phần F — (Bonus) Event (ôn L7)

`InventoryService` raise `event Action<string, int>? StockChanged` khi Add/Remove thành công — test bằng subscribe + flag/list.

## 4. Mô hình tư duy

```text
Mỗi feature:
  1. Viết API production tối giản
  2. Viết test fail (red) hoặc test cùng lúc
  3. Đảm bảo AAA + tên rõ
  4. Chạy dotnet test — xanh hết mới thêm feature

Map Level → test:
  L1  Calculator arithmetic + Divide zero
  L2  PercentDiscount class / interface
  L3  (optional) record Sale value semantics
  L4  Inventory dictionary store
  L5  generic helpers nếu bạn thêm Result<T>
  L6  Throws assertions
  L7  event StockChanged
  L8  SalesAnalytics LINQ
```

## 5. Cú pháp / Skeleton

```bash
mkdir TestedApp && cd TestedApp
dotnet new sln -n TestedApp
dotnet new classlib -n TestedApp -f net8.0 -o src/TestedApp
dotnet new xunit -n TestedApp.Tests -f net8.0 -o tests/TestedApp.Tests
dotnet sln add src/TestedApp/TestedApp.csproj tests/TestedApp.Tests/TestedApp.Tests.csproj
dotnet add tests/TestedApp.Tests reference src/TestedApp
# optional
dotnet add tests/TestedApp.Tests package Moq
```

Skeleton `Calculator`:

```csharp
namespace TestedApp;

public class Calculator
{
    public decimal Add(decimal a, decimal b) => a + b;
    public decimal Subtract(decimal a, decimal b) => a - b;
    public decimal Multiply(decimal a, decimal b) => a * b;

    public decimal Divide(decimal a, decimal b)
    {
        if (b == 0) throw new DivideByZeroException();
        return a / b;
    }

    public decimal Average(IEnumerable<decimal> values)
    {
        var list = values as IList<decimal> ?? values.ToList();
        if (list.Count == 0) throw new InvalidOperationException("empty");
        return list.Average();
    }
}
```

## 6. Ví dụ test mẫu

### Cơ bản

```csharp
public class CalculatorTests
{
    private readonly Calculator _sut = new();

    [Theory]
    [InlineData(1, 2, 3)]
    [InlineData(0, 0, 0)]
    [InlineData(-1, 5, 4)]
    public void Add_ReturnsSum(decimal a, decimal b, decimal expected)
        => Assert.Equal(expected, _sut.Add(a, b));

    [Fact]
    public void Divide_ByZero_Throws()
        => Assert.Throws<DivideByZeroException>(() => _sut.Divide(1, 0));
}
```

### Trung cấp — Pricing + fake policy

```csharp
public class PricingServiceTests
{
    [Fact]
    public void FinalPrice_Applies10PercentDiscount()
    {
        var sut = new PricingService(new PercentDiscount(10));
        Assert.Equal(90m, sut.FinalPrice(100m));
    }

    [Fact]
    public void FinalPrice_Negative_Throws()
    {
        var sut = new PricingService(new PercentDiscount(0));
        Assert.Throws<ArgumentOutOfRangeException>(() => sut.FinalPrice(-1m));
    }
}
```

### Nâng cao — Inventory + mock + LINQ

```csharp
[Fact]
public void Remove_TooMany_Throws_AndDoesNotCallStoreWriteBeyond()
{
    var store = new InMemoryInventoryStore();
    store.Set("sword", 2);
    var sut = new InventoryService(store);

    Assert.Throws<InvalidOperationException>(() => sut.Remove("sword", 5));
    Assert.Equal(2, store.Get("sword"));
}

[Fact]
public void TotalByCategory_GroupsSums()
{
    var sales = new[]
    {
        new Sale("Weapon", 10),
        new Sale("Armor", 5),
        new Sale("Weapon", 7),
    };
    var map = SalesAnalytics.TotalByCategory(sales);
    Assert.Equal(17m, map["Weapon"]);
    Assert.Equal(5m, map["Armor"]);
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Test không thấy class | Sai namespace / chưa reference | Check usings + csproj |
| Decimal assert lệch | Dùng `double` | Prefer `decimal` tiền tệ |
| LINQ test order-dependent | `TopN` không OrderBy | Sort rõ trong implementation |
| Event test flaky | Quên unsubscribe / static | Local handler trong test |
| Inventory âm | Thiếu validate | Throw trước khi ghi store |

## 8. Gỡ lỗi

1. `dotnet test` — sửa fail từng cái, đừng tắt assert.
2. Tách test: nếu một Fact assert 5 thứ, chia nhỏ khi fail khó đọc.
3. Debug test trong IDE trên method đang đỏ.
4. InMemory store: dump dictionary khi assert quantity sai.

## 9. Best practices

- Production library **không** reference xUnit.
- Một thư mục test / feature (`CalculatorTests.cs`, `PricingTests.cs`, …).
- Đặt tên test theo hành vi, không theo số bài.
- Mỗi bug tìm thấy khi demo → thêm một regression test.
- Giữ test < ~100ms mỗi unit test.
- README ngắn trong solution: cách chạy `dotnet test`.

## 10. Bài tập (checklist bắt buộc)

**Bài 1** — Calculator: Add/Sub/Mul/Div + Average; ≥ 8 test methods (kết hợp Fact/Theory).

**Bài 2** — `PercentDiscount` + `PricingService`; test 0%, 10%, 100%; price âm throw.

**Bài 3** — `IInventoryStore` + `InMemoryInventoryStore` + `InventoryService`; test Add/Remove/insufficient.

**Bài 4** — `SalesAnalytics`: Total, TotalByCategory, TopN; dùng LINQ trong production.

**Bài 5 (bonus)** — Event `StockChanged` hoặc Moq verify store được gọi khi Add.

## 11. Gợi ý

- Average: `values.ToList()` rồi check Count.
- PercentDiscount: validate percent 0..100.
- Inventory Remove: if qty > current → throw.
- TotalByCategory: `GroupBy` + `ToDictionary(g => g.Key, g => g.Sum(x => x.Amount))`.
- TopN: `OrderByDescending(s => s.Amount).Take(n)`.

## 12. Đáp án

**PercentDiscount:**

```csharp
public class PercentDiscount : IDiscountPolicy
{
    private readonly decimal _percent;
    public PercentDiscount(decimal percent)
    {
        if (percent < 0 || percent > 100)
            throw new ArgumentOutOfRangeException(nameof(percent));
        _percent = percent;
    }

    public decimal Apply(decimal price) => price * (1 - _percent / 100m);
}
```

**SalesAnalytics:**

```csharp
public static class SalesAnalytics
{
    public static decimal Total(IEnumerable<Sale> sales) => sales.Sum(s => s.Amount);

    public static IReadOnlyDictionary<string, decimal> TotalByCategory(IEnumerable<Sale> sales)
        => sales.GroupBy(s => s.Category)
                .ToDictionary(g => g.Key, g => g.Sum(s => s.Amount));

    public static IEnumerable<Sale> TopN(IEnumerable<Sale> sales, int n)
        => sales.OrderByDescending(s => s.Amount).Take(n);
}
```

**InventoryService (ý chính):**

```csharp
public class InventoryService
{
    private readonly IInventoryStore _store;
    public event Action<string, int>? StockChanged;

    public InventoryService(IInventoryStore store) => _store = store;

    public void Add(string sku, int qty)
    {
        if (qty <= 0) throw new ArgumentOutOfRangeException(nameof(qty));
        var next = _store.Get(sku) + qty;
        _store.Set(sku, next);
        StockChanged?.Invoke(sku, next);
    }

    public void Remove(string sku, int qty)
    {
        if (qty <= 0) throw new ArgumentOutOfRangeException(nameof(qty));
        var current = _store.Get(sku);
        if (qty > current) throw new InvalidOperationException("insufficient stock");
        var next = current - qty;
        _store.Set(sku, next);
        StockChanged?.Invoke(sku, next);
    }
}
```

## 13. Đáp án thay thế

- Dùng NUnit thay xUnit cho toàn bộ tests.
- `Result<T>` generic (L5) thay exception cho Divide — miễn test đúng contract.
- FluentAssertions package cho assert đọc tự nhiên hơn.

## 14. Thử thách

Thêm `IClock` vào mã giảm giá “happy hour” (chỉ discount trong khung giờ) + fake clock; hoặc integration test ghi inventory ra file JSON tạm rồi đọc lại.

## 15. Ứng dụng thực tế

- Mọi thư viện domain/backend bắt đầu bằng pattern App + App.Tests
- Regression suite chạy CI mỗi PR
- Test là tài liệu sống cho calculator/pricing rules

## 16. Liên hệ Unity

- Port `Calculator` / `PricingService` / `SalesAnalytics` sang asmdef Editor tests (NUnit)
- `InventoryService` có thể gắn vào game inventory — logic unit test Edit Mode; UI Play Mode riêng
- Tránh để công thức giá chỉ nằm trong MonoBehaviour

## 17. Kiểm tra kiến thức

1. Vì sao production là classlib thay vì console-only?  
   **Đáp án:** Dễ reference từ test; console chỉ là host (tuỳ chọn).

2. Map nhanh: test `Divide_ByZero` ôn Level nào?  
   **Đáp án:** Exceptions (L6) + fundamentals.

3. Vì sao `IDiscountPolicy` thay hard-code percent trong service?  
   **Đáp án:** Đổi policy / fake khi test; OCP + DI.

4. `TotalByCategory` chủ yếu ôn gì?  
   **Đáp án:** LINQ GroupBy/Sum (L8).

5. Khi nào thêm Moq vào project này?  
   **Đáp án:** Khi cần verify tương tác hoặc stub nhanh thay fake thủ công.

---

## Checklist hoàn thành project

- [ ] `dotnet test` xanh toàn bộ
- [ ] Có test Calculator + Pricing + Inventory + SalesAnalytics
- [ ] Ít nhất một `Theory` và một `Assert.Throws`
- [ ] Ít nhất một fake hoặc mock cho interface
- [ ] (Bonus) Event hoặc file integration nhẹ

Xong project → **Level 15 — Clean Code**.
