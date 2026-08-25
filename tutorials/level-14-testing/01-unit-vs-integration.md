# Chương 1 — Unit vs Integration Testing

## 1. Mục tiêu học

- Phân biệt **unit test** và **integration test**
- Hiểu **test pyramid**: nhiều unit, ít integration, rất ít E2E
- Biết phạm vi “unit” trong C# (thường là class / method, không phải cả hệ thống)
- Chọn đúng loại test cho từng rủi ro

## 2. Điều kiện tiên quyết

- Level 1–2: class, method, interface
- Biết chạy `dotnet new` / `dotnet build`
- Đã từng “chạy thử” logic trong `Main` — giờ thay bằng test

## 3. Khái niệm

**Unit test** kiểm tra một đơn vị nhỏ **cô lập**: thường một class/method, dependency giả (fake/mock), chạy nhanh, không cần mạng/DB/file thật.

**Integration test** kiểm tra **nhiều phần thật cùng chạy**: DB, HTTP, file system, message queue — chậm hơn, dễ flaky hơn, nhưng bắt lỗi “ghép nối”.

| Tiêu chí | Unit | Integration |
|----------|------|-------------|
| Phạm vi | 1 class/method (+ fake) | Nhiều component thật |
| Tốc độ | ms | thường chậm hơn |
| Dependency ngoài | mock/fake | thật hoặc test container |
| Phát hiện lỗi | logic sai, edge case | cấu hình, wire-up, SQL, schema |
| Số lượng khuyến nghị | Nhiều | Ít hơn |

### Test pyramid (tư duy)

```text
        /\
       /E2E\          ← ít: UI + server + DB thật
      /------\
     / Integr.\       ← vừa: API + DB test, file thật
    /----------\
   /   Unit     \     ← nhiều: logic thuần, nhanh
  /--------------\
```

Không phải mọi thứ đều “unit”. Nếu test mở SQL Server thật → đó là integration dù bạn đặt tên file `SomethingUnitTests.cs`.

## 4. Mô hình tư duy

```text
Code cần kiểm chứng
        │
        ├─ Chỉ cần đúng công thức / nhánh if?
        │     → Unit test (cô lập)
        │
        ├─ Cần chắc “lưu DB rồi đọc lại đúng”?
        │     → Integration test
        │
        └─ Cần chắc “user bấm nút → thấy kết quả trên UI”?
              → E2E (Level sau / tool riêng; Level 14 tập trung unit + integration cơ bản)
```

**Quy tắc vàng:** Unit test fail → bug trong logic đơn vị. Integration fail → bug ở ranh giới giữa các phần (hoặc môi trường).

## 5. Cú pháp

Với xUnit (chi tiết chương 5), khung tối thiểu:

```csharp
using Xunit;

public class CalculatorTests
{
    [Fact]
    public void Add_TwoPositiveNumbers_ReturnsSum()
    {
        // Arrange
        var calc = new Calculator();

        // Act
        int result = calc.Add(2, 3);

        // Assert
        Assert.Equal(5, result);
    }
}
```

Chạy:

```bash
dotnet test
```

## 6. Ví dụ

### Cơ bản — Unit test thuần logic

```csharp
public class PriceCalculator
{
    public decimal ApplyDiscount(decimal price, decimal percent)
    {
        if (price < 0) throw new ArgumentOutOfRangeException(nameof(price));
        if (percent < 0 || percent > 100) throw new ArgumentOutOfRangeException(nameof(percent));
        return price * (1 - percent / 100m);
    }
}

public class PriceCalculatorTests
{
    [Fact]
    public void ApplyDiscount_10Percent_On100_Returns90()
    {
        var sut = new PriceCalculator(); // SUT = System Under Test
        var result = sut.ApplyDiscount(100m, 10m);
        Assert.Equal(90m, result);
    }
}
```

Không cần DB, không cần web — đây là **unit**.

### Trung cấp — Cùng một feature, hai loại test

```csharp
public interface IOrderRepository
{
    void Save(Order order);
    Order? GetById(int id);
}

public class OrderService
{
    private readonly IOrderRepository _repo;
    public OrderService(IOrderRepository repo) => _repo = repo;

    public Order Place(string product, decimal price)
    {
        var order = new Order { Product = product, Price = price, Status = "Placed" };
        _repo.Save(order);
        return order;
    }
}
```

**Unit** — fake repo trong memory:

```csharp
public class InMemoryOrderRepository : IOrderRepository
{
    private readonly Dictionary<int, Order> _data = new();
    private int _nextId = 1;

    public void Save(Order order)
    {
        if (order.Id == 0) order.Id = _nextId++;
        _data[order.Id] = order;
    }

    public Order? GetById(int id) => _data.GetValueOrDefault(id);
}

[Fact]
public void Place_ValidOrder_SavesAndReturnsPlaced()
{
    var repo = new InMemoryOrderRepository();
    var sut = new OrderService(repo);

    var order = sut.Place("Sword", 99m);

    Assert.Equal("Placed", order.Status);
    Assert.NotNull(repo.GetById(order.Id));
}
```

**Integration** — (ý tưởng) dùng SQL thật / file thật:

```csharp
[Fact]
public void Place_ThenGetById_FromSqlite_RoundTrips()
{
    // Mở connection SQLite thật, tạo bảng, wire SqlOrderRepository
    // Assert đọc lại đúng từ DB
}
```

### Nâng cao — Ranh giới xám (gray area)

Test gọi `HttpClient` tới server local bạn spin up trong test = integration (hoặc component test).  
Test chỉ kiểm tra `PriceCalculator` với số liệu = unit.  
**Đừng tranh tên** — quan trọng hơn: tốc độ, độ cô lập, và bạn đang kiểm chứng **cái gì**.

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| “Unit test” chậm 5 giây | Gọi mạng/DB/file | Tách dependency, mock |
| Test pass local, fail CI | Phụ thuộc máy (path, timezone, culture) | Cô lập; set culture; dùng path tạm |
| Test phụ thuộc thứ tự chạy | State dùng chung (static, DB bẩn) | Mỗi test tự Arrange; cleanup |
| Chỉ test “happy path” | Sợ viết nhiều | Thêm biên: null, 0, âm, empty |
| Đặt tên `Test1` | Vội | Đặt tên mô tả hành vi |

## 8. Gỡ lỗi

1. `dotnet test --logger "console;verbosity=detailed"` — xem test nào fail và message assert.
2. Fail “Expected 90, Actual 89.999…” → so sánh `decimal` / floating; dùng `Assert.Equal(expected, actual, precision)`.
3. Integration fail ngẫu nhiên → nghi race / dữ liệu còn sót từ test trước.
4. Hỏi: “Test này fail vì **logic** hay vì **môi trường**?” → phân loại lại unit/integration.

## 9. Best practices

- Unit test **nhanh + ổn định** là nền tảng — đừng thay bằng toàn integration.
- Mỗi test nên kiểm tra **một hành vi** chính (một lý do fail rõ ràng).
- Đặt tên: `Method_Scenario_Expected` hoặc `Given_When_Then`.
- Không assert mọi field nếu không liên quan — noise.
- Integration: dùng DB test riêng, không đụng production.
- Giữ pyramid: nhiều unit, ít integration có chủ đích.

## 10. Bài tập

**Bài 1** — Viết `TaxCalculator.Calculate(decimal amount, decimal rate)` và 2 unit test: rate 10% trên 200 → 20; amount âm → throw.

**Bài 2** — Liệt kê 3 scenario của feature “đăng nhập” nên là unit, 2 scenario nên là integration. Viết ngắn trong comment/markdown.

**Bài 3** — Class `GreetingService.Greet(string name)` trả `$"Hello, {name}!"`. Unit test với `"Ada"` và `""` (quy ước: empty → `"Hello, guest!"`).

**Bài 4** — Phân loại: test đọc `appsettings.json` từ disk rồi parse — unit hay integration? Giải thích 2 câu.

## 11. Gợi ý

- Bài 1: `amount * rate`, validate `amount >= 0`.
- Bài 2: unit = hash password logic; integration = gọi Identity DB.
- Bài 3: `string.IsNullOrWhiteSpace(name)`.
- Bài 4: đụng file system → thiên về integration (hoặc “unit với file” nhưng không còn cô lập tuyệt đối).

## 12. Đáp án

**Bài 1:**

```csharp
public class TaxCalculator
{
    public decimal Calculate(decimal amount, decimal rate)
    {
        if (amount < 0) throw new ArgumentOutOfRangeException(nameof(amount));
        return amount * rate;
    }
}

public class TaxCalculatorTests
{
    [Fact]
    public void Calculate_10Percent_On200_Returns20()
    {
        var sut = new TaxCalculator();
        Assert.Equal(20m, sut.Calculate(200m, 0.10m));
    }

    [Fact]
    public void Calculate_NegativeAmount_Throws()
    {
        var sut = new TaxCalculator();
        Assert.Throws<ArgumentOutOfRangeException>(() => sut.Calculate(-1m, 0.1m));
    }
}
```

**Bài 2 (ví dụ):**

- Unit: validate email format; so sánh hash; rule “password ≥ 8 ký tự”.
- Integration: lưu user vào SQL rồi login lại; gọi API `/login` end-to-end với test server.

**Bài 3:**

```csharp
public class GreetingService
{
    public string Greet(string name)
        => string.IsNullOrWhiteSpace(name) ? "Hello, guest!" : $"Hello, {name}!";
}
```

**Bài 4:** Integration (hoặc file-based test) vì phụ thuộc I/O thật; muốn unit thì inject `IConfiguration` / abstract reader.

## 13. Đáp án thay thế

Bài 1 dùng `Theory` + `InlineData` thay nhiều `Fact`. Bài 3 có thể throw khi `name` null thay vì guest — miễn document và test đúng contract.

## 14. Thử thách

Vẽ test pyramid cho một game inventory service (add item, save to disk, sync server). Ghi chú mỗi tầng test gì — tối đa 1 trang.

## 15. Ứng dụng thực tế

- Backend API: unit cho domain; integration cho EF Core + SQL
- Thư viện NuGet: gần như toàn unit
- CI: `dotnet test` gate trước merge
- Regression: bug production → thêm test tái hiện trước khi fix

## 16. Liên hệ Unity

- **Edit Mode tests**: gần unit — logic thuần C#, không cần Play Mode
- **Play Mode tests**: gần integration — scene, Physics, Coroutine chạy thật
- Logic damage/inventory nên tách khỏi `MonoBehaviour` để unit test Edit Mode được
- Tránh “chỉ Play trong Editor” làm nguồn kiểm chứng duy nhất

## 17. Kiểm tra kiến thức

1. Unit test khác integration test ở điểm nào quan trọng nhất?  
   **Đáp án:** Độ cô lập / có dùng dependency ngoài thật hay không.

2. Vì sao cần nhiều unit hơn integration?  
   **Đáp án:** Nhanh, ổn định, feedback sớm, rẻ hơn khi chạy CI.

3. Test gọi SQL Server thật thường là gì?  
   **Đáp án:** Integration test.

4. SUT nghĩa là gì?  
   **Đáp án:** System Under Test — đối tượng đang được kiểm tra.

5. Test pyramid nói gì về E2E?  
   **Đáp án:** Ít nhất — chậm và dễ vỡ; dùng có chọn lọc.
