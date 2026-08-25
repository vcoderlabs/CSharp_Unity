# Chương 4 — Dependency Injection cơ bản (cho Testing)

## 1. Mục tiêu học

- Hiểu **Dependency Injection (DI)** giúp gì cho testability
- Áp dụng **constructor injection** (cách phổ biến nhất)
- Phân biệt “new bên trong” vs “nhận từ ngoài”
- Biết DI container (Microsoft.Extensions.DependencyInjection) ở mức tối thiểu — đủ wire app; test vẫn có thể new thủ công

## 2. Điều kiện tiên quyết

- Chương 3: interface + mock/fake
- Level 2: constructor, interface segregation đơn giản

## 3. Khái niệm

**Dependency** = thứ class cần để làm việc (`IEmailSender`, `IRepository`, `IClock`).

**Injection** = đưa dependency **từ bên ngoài** vào (thường qua constructor), thay vì class tự `new` cứng.

```csharp
// ❌ Hard-coded — khó test
class ReportService
{
    private readonly SqlStore _store = new SqlStore("Server=...");
}

// ✅ Inject — test thay SqlStore bằng fake
class ReportService
{
    private readonly IStore _store;
    public ReportService(IStore store) => _store = store;
}
```

### Ba cách inject (biết tên)

| Cách | Ví dụ | Ghi chú |
|------|--------|---------|
| Constructor | `new Service(dep)` | Khuyến nghị mặc định |
| Property | `service.Dep = x` | Dễ quên set; optional deps |
| Method | `service.Run(dep)` | Ít gặp cho dep dài hạn |

Level này tập trung **constructor injection**.

## 4. Mô hình tư duy

```text
Composition Root (Program.cs / Startup)
    tạo Fake/Real dependencies
    new Service(deps)  hoặc  container.Resolve

Service chỉ nhận abstraction (interface)
    → không biết đang chạy SQL hay InMemory
    → unit test nhét Fake vào constructor
```

**DI ≠ bắt buộc dùng container.** Container chỉ tiện khi app lớn. Trong unit test: `new SUT(fake)` là đủ.

## 5. Cú pháp

### Manual DI

```csharp
IEmailSender sender = new SmtpEmailSender(config);
IOrderRepository repo = new SqlOrderRepository(conn);
var service = new OrderService(repo, sender);
```

### Microsoft DI (tối thiểu)

```bash
dotnet add package Microsoft.Extensions.DependencyInjection
```

```csharp
using Microsoft.Extensions.DependencyInjection;

var services = new ServiceCollection();
services.AddSingleton<IClock, SystemClock>();
services.AddScoped<IOrderRepository, SqlOrderRepository>();
services.AddTransient<OrderService>();

using var provider = services.BuildServiceProvider();
var sut = provider.GetRequiredService<OrderService>();
```

Trong **unit test**, thường bỏ container:

```csharp
var sut = new OrderService(new InMemoryOrderRepository(), new FakeEmailSender());
```

## 6. Ví dụ

### Cơ bản — Refactor trước/sau

**Trước:**

```csharp
public class InvoiceService
{
    public string Create(string customer, decimal total)
    {
        var number = $"INV-{DateTime.Now:yyyyMMdd-HHmmss}";
        File.WriteAllText($"{number}.txt", $"{customer}:{total}");
        return number;
    }
}
```

**Sau:**

```csharp
public interface IInvoiceNumberGenerator
{
    string Next();
}

public interface IInvoiceStore
{
    void Save(string number, string customer, decimal total);
}

public class InvoiceService
{
    private readonly IInvoiceNumberGenerator _numbers;
    private readonly IInvoiceStore _store;

    public InvoiceService(IInvoiceNumberGenerator numbers, IInvoiceStore store)
    {
        _numbers = numbers;
        _store = store;
    }

    public string Create(string customer, decimal total)
    {
        var number = _numbers.Next();
        _store.Save(number, customer, total);
        return number;
    }
}
```

Test:

```csharp
[Fact]
public void Create_Valid_SavesWithGeneratedNumber()
{
    var numbers = new Mock<IInvoiceNumberGenerator>();
    numbers.Setup(n => n.Next()).Returns("INV-1");
    var store = new FakeInvoiceStore();

    var id = new InvoiceService(numbers.Object, store).Create("Ada", 10m);

    Assert.Equal("INV-1", id);
    Assert.True(store.Contains("INV-1"));
}
```

### Trung cấp — Nhiều dependency + null guard

```csharp
public class OrderService
{
    private readonly IOrderRepository _repo;
    private readonly IEmailSender _email;
    private readonly IClock _clock;

    public OrderService(IOrderRepository repo, IEmailSender email, IClock clock)
    {
        _repo = repo ?? throw new ArgumentNullException(nameof(repo));
        _email = email ?? throw new ArgumentNullException(nameof(email));
        _clock = clock ?? throw new ArgumentNullException(nameof(clock));
    }
}
```

### Nâng cao — Composition root tách khỏi domain

```csharp
// Program.cs — composition root
var services = new ServiceCollection();
services.AddSingleton<IClock, SystemClock>();
services.AddSingleton<IEmailSender, SmtpEmailSender>();
services.AddSingleton<IOrderRepository, InMemoryOrderRepository>(); // dev
services.AddTransient<OrderService>();
```

Domain/services **không** reference `ServiceCollection` — chỉ nhận interface.

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Constructor 8+ deps | Class làm quá nhiều | Tách class / facade |
| `new` vẫn nằm sâu trong method | Refactor dở | Inject tiếp abstraction |
| Service Locator anti-pattern | `provider.GetService` khắp nơi | Chỉ resolve ở composition root |
| Test dùng container phức tạp | Setup nặng | `new SUT(fakes)` |
| Interface quá to | Khó fake | Tách interface nhỏ |

## 8. Gỡ lỗi

1. NullReference trong production — quên đăng ký DI / quên truyền ctor.
2. Test pass, app fail — composition root đăng ký implementation sai lifetime.
3. Circular dependency — A cần B cần A; redesign.
4. Đọc stack trace: object nào `null` → thiếu inject.

## 9. Best practices

- Depend on **abstractions** (interface), không depend concrete khó thay.
- Constructor injection mặc định; readonly field.
- Composition root ở mép app (`Program.cs`).
- Unit test: manual inject fakes — đơn giản hơn container.
- Đừng tạo interface cho mọi class một cách máy móc — chỉ khi cần thay thế/test/swap.
- Tránh static stateful service khó inject.

## 10. Bài tập

**Bài 1** — Refactor `class Mailer { public void Send(string m) => File.AppendAllText("log.txt", m); }` thành `IMessageSink` + inject.

**Bài 2** — Viết unit test cho class sau refactor (fake sink ghi list).

**Bài 3** — `PriceService` cần `ITaxCalculator` và `IDiscountPolicy`. Constructor inject cả hai; test một scenario có tax + discount.

**Bài 4** — Giải thích vì sao `public static IServiceProvider Provider { get; set; }` dùng khắp app là xấu cho test (3–5 câu).

## 11. Gợi ý

- Bài 1: `void Write(string message)`.
- Bài 2: `List<string>` trong fake.
- Bài 3: `final = discount.Apply(price) + tax.Calculate(...)` — chọn công thức rõ.
- Bài 4: ẩn dependency, khó thay per-test, khuyến khích coupling toàn cục.

## 12. Đáp án

**Bài 1–2:**

```csharp
public interface IMessageSink { void Write(string message); }

public class Mailer
{
    private readonly IMessageSink _sink;
    public Mailer(IMessageSink sink) => _sink = sink;
    public void Send(string m) => _sink.Write(m);
}

public class ListSink : IMessageSink
{
    public List<string> Items { get; } = new();
    public void Write(string message) => Items.Add(message);
}

[Fact]
public void Send_WritesToSink()
{
    var sink = new ListSink();
    new Mailer(sink).Send("hi");
    Assert.Equal(new[] { "hi" }, sink.Items);
}
```

**Bài 3:** Inject 2 interface; Arrange mock tax/discount; Assert tổng.

**Bài 4:** Service locator toàn cục che giấu deps, test khó cô lập, lifetime rối, khuyến khích mọi chỗ resolve lung tung thay vì inject rõ ràng.

## 13. Đáp án thay thế

Dùng record primary constructor (.NET 8+): `public class Mailer(IMessageSink sink) { public void Send(string m) => sink.Write(m); }`.

## 14. Thử thách

Lấy một “God class” tự viết (hoặc skeleton) có 3 `new` bên trong — extract 3 interface, wire manual DI trong `Program`, viết 3 unit test.

## 15. Ứng dụng thực tế

- ASP.NET Core: DI built-in cho controllers/services
- Worker / console: `Host.CreateDefaultBuilder` + `ConfigureServices`
- Plugin architecture: đăng ký implementation theo config

## 16. Liên hệ Unity

- Unity không phải MS DI mặc định — dùng Zenject/VContainer/Reflex hoặc **manual inject** qua serialized fields / Compose root bootstrap
- `new` trong `Update` = khó test; inject service vào MonoBehaviour mỏng
- ScriptableObject có thể đóng vai “strategy” inject được

## 17. Kiểm tra kiến thức

1. Constructor injection là gì?  
   **Đáp án:** Truyền dependency qua constructor khi tạo object.

2. Vì sao DI giúp unit test?  
   **Đáp án:** Có thể thay dependency thật bằng fake/mock.

3. Composition root nằm đâu?  
   **Đáp án:** Mép ứng dụng (Program/Startup) — nơi ráp đồ thật.

4. Unit test có bắt buộc dùng DI container không?  
   **Đáp án:** Không — thường `new SUT(fake)` là đủ.

5. Hard-coded `new SqlConnection()` trong service gây gì?  
   **Đáp án:** Khó cô lập, chậm, phụ thuộc môi trường — kém testable.
