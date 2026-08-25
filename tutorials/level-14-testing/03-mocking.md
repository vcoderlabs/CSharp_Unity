# Chương 3 — Mocking

## 1. Mục tiêu học

- Phân biệt **fake**, **stub**, **mock** (đủ dùng thực tế)
- Viết **fake thủ công** bằng class implement interface
- Dùng thư viện mock (Moq hoặc NSubstitute) để giả dependency
- Biết khi nào **verify** tương tác (gọi method bao nhiêu lần) là hợp lý

## 2. Điều kiện tiên quyết

- Chương 1–2: unit test + AAA
- Level 2: interface và dependency qua constructor
- NuGet: biết `dotnet add package`

## 3. Khái niệm

Khi unit test `OrderService`, bạn **không** muốn gửi email thật hay ghi SQL thật. Bạn thay dependency bằng bản giả.

| Thuật ngữ | Ý đơn giản | Ví dụ |
|-----------|-------------|--------|
| **Fake** | Implementation nhẹ thay thế | `InMemoryRepository` |
| **Stub** | Trả dữ liệu cố định cho SUT dùng | `GetUser` luôn trả user X |
| **Mock** | Đối tượng giả + **kiểm tra đã gọi đúng** | Verify `Send` được gọi 1 lần |

Trong thực tế nhiều người gọi chung là “mock”. Quan trọng: **bạn đang thay dependency để cô lập SUT**.

### Trước / sau tư duy

**Trước** — khó test:

```csharp
public class Notifier
{
    public void Notify(string email, string msg)
    {
        new SmtpClient().Send(email, msg); // cứng, chậm, side-effect
    }
}
```

**Sau** — test được:

```csharp
public interface IEmailSender
{
    void Send(string email, string message);
}

public class Notifier
{
    private readonly IEmailSender _sender;
    public Notifier(IEmailSender sender) => _sender = sender;

    public void Notify(string email, string msg) => _sender.Send(email, msg);
}
```

## 4. Mô hình tư duy

```text
SUT ──gọi──► IDependency
                 ▲
                 │
         Fake / Mock trong test

Test Arrange: tạo mock cấu hình “khi gọi X thì trả Y”
Test Act:     gọi SUT
Test Assert:  kết quả SUT  (± verify mock đã được gọi)
```

**Chỉ verify interaction khi interaction là yêu cầu nghiệp vụ** (ví dụ: “phải gửi email 1 lần”). Đừng verify mọi getter.

## 5. Cú pháp

### Fake thủ công

```csharp
public class FakeEmailSender : IEmailSender
{
    public List<(string Email, string Message)> Sent { get; } = new();
    public void Send(string email, string message) => Sent.Add((email, message));
}
```

### Moq

```bash
dotnet add package Moq
```

```csharp
using Moq;

var mock = new Mock<IEmailSender>();
mock.Setup(s => s.Send(It.IsAny<string>(), It.IsAny<string>()));

var sut = new Notifier(mock.Object);
sut.Notify("a@b.com", "hi");

mock.Verify(s => s.Send("a@b.com", "hi"), Times.Once);
```

### NSubstitute (cú pháp đọc gần C#)

```bash
dotnet add package NSubstitute
```

```csharp
using NSubstitute;

var sender = Substitute.For<IEmailSender>();
var sut = new Notifier(sender);
sut.Notify("a@b.com", "hi");
sender.Received(1).Send("a@b.com", "hi");
```

## 6. Ví dụ

### Cơ bản — Fake thủ công + assert state

```csharp
public interface IClock
{
    DateTime UtcNow { get; }
}

public class CouponService
{
    private readonly IClock _clock;
    public CouponService(IClock clock) => _clock = clock;

    public bool IsValid(DateTime expiresAt) => _clock.UtcNow < expiresAt;
}

public class FixedClock : IClock
{
    public FixedClock(DateTime utcNow) => UtcNow = utcNow;
    public DateTime UtcNow { get; }
}

[Fact]
public void IsValid_WhenBeforeExpiry_ReturnsTrue()
{
    var clock = new FixedClock(new DateTime(2026, 1, 1));
    var sut = new CouponService(clock);
    Assert.True(sut.IsValid(new DateTime(2026, 1, 2)));
}
```

### Trung cấp — Stub trả giá trị + Moq

```csharp
public interface IUserRepository
{
    User? FindById(int id);
}

public class UserGreeting
{
    private readonly IUserRepository _repo;
    public UserGreeting(IUserRepository repo) => _repo = repo;

    public string Greet(int id)
    {
        var user = _repo.FindById(id);
        return user is null ? "Hello, guest" : $"Hello, {user.Name}";
    }
}

[Fact]
public void Greet_ExistingUser_ReturnsName()
{
    var mock = new Mock<IUserRepository>();
    mock.Setup(r => r.FindById(7)).Returns(new User { Id = 7, Name = "Ada" });

    var sut = new UserGreeting(mock.Object);
    Assert.Equal("Hello, Ada", sut.Greet(7));
}
```

### Nâng cao — Verify + callback

```csharp
[Fact]
public void PlaceOrder_Valid_SendsConfirmationEmail()
{
    var emails = new Mock<IEmailSender>();
    var repo = new Mock<IOrderRepository>();
    var sut = new OrderService(repo.Object, emails.Object);

    sut.Place("a@b.com", "Shield", 50m);

    emails.Verify(
        e => e.Send("a@b.com", It.Is<string>(m => m.Contains("Shield"))),
        Times.Once);
    repo.Verify(r => r.Save(It.IsAny<Order>()), Times.Once);
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Mock trả `null` mặc định | Chưa `Setup` | Setup Returns |
| Over-verify | Assert mọi lời gọi | Chỉ verify hành vi quan trọng |
| Mock class concrete sealed | Moq hạn chế | Extract interface |
| Test vỡ khi rename param | Verify quá chặt | `It.IsAny` khi không quan trọng |
| Logic trong mock phức tạp | Mock thành mini-app | Viết fake class rõ ràng |

## 8. Gỡ lỗi

1. `MockException: Expected invocation...` — SUT không gọi như bạn Verify, hoặc Setup sai.
2. NullReference trong SUT — dependency trả null; Setup `Returns(...)`.
3. In debug: xem `mock.Invocations` (Moq) để biết đã gọi gì.
4. Nếu fake thủ công dễ hơn mock library cho state phức tạp — **cứ dùng fake**.

## 9. Best practices

- Ưu tiên **interface nhỏ** (ISP) — dễ mock.
- Fake thủ công tốt cho collection/in-memory DB.
- Mock library tốt cho verify tương tác và stub nhanh.
- Đừng mock SUT — mock **dependency của SUT**.
- Tránh mock mọi thứ; test vẫn cần đọc được.
- `DateTime.Now` → inject `IClock` (như ví dụ) để hết flaky.

## 10. Bài tập

**Bài 1** — `ILogger` với `void Info(string msg)`. Class `Checkout` gọi `Info` khi thanh toán thành công. Fake ghi lại messages; assert có chứa `"paid"`.

**Bài 2** — Dùng Moq hoặc NSubstitute stub `IRateProvider.GetRate("USD")` trả `1.1m`; test `CurrencyConverter.ToEur`.

**Bài 3** — Verify `IEmailSender.Send` **không** được gọi khi order invalid.

**Bài 4** — Viết `IFileStore` fake dictionary path→content; test `ConfigLoader.Load` đọc key.

## 11. Gợi ý

- Bài 1: `List<string> Messages` trong fake.
- Bài 2: `amount / rate` hoặc `* rate` — chọn và test đúng công thức.
- Bài 3: `Times.Never` / `DidNotReceive()`.
- Bài 4: `Dictionary<string,string>` implement `ReadAllText`.

## 12. Đáp án

**Bài 1:**

```csharp
public interface ILogger { void Info(string msg); }

public class FakeLogger : ILogger
{
    public List<string> Messages { get; } = new();
    public void Info(string msg) => Messages.Add(msg);
}

public class Checkout
{
    private readonly ILogger _log;
    public Checkout(ILogger log) => _log = log;
    public void Pay(decimal amount)
    {
        if (amount <= 0) throw new ArgumentOutOfRangeException(nameof(amount));
        _log.Info($"paid:{amount}");
    }
}

[Fact]
public void Pay_Valid_LogsPaid()
{
    var log = new FakeLogger();
    new Checkout(log).Pay(10m);
    Assert.Contains(log.Messages, m => m.Contains("paid"));
}
```

**Bài 2 (Moq):**

```csharp
var rates = new Mock<IRateProvider>();
rates.Setup(r => r.GetRate("USD")).Returns(1.1m);
var sut = new CurrencyConverter(rates.Object);
Assert.Equal(110m, sut.ToEur(100m, "USD")); // giả sử * rate
```

**Bài 3:**

```csharp
emails.Verify(e => e.Send(It.IsAny<string>(), It.IsAny<string>()), Times.Never);
```

**Bài 4:** Fake `Dictionary` + `Assert.Equal(expected, loader.Get("key"))`.

## 13. Đáp án thay thế

Dùng NSubstitute thay Moq. Hoặc chỉ fake thủ công cả bài 2–3 nếu chưa muốn thêm package — vẫn đạt mục tiêu học.

## 14. Thử thách

Refactor một class đang `new HttpClient()` bên trong method → `IHttpClient`/`IApiClient` + mock trả JSON giả; viết 2 unit test success/fail.

## 15. Ứng dụng thực tế

- Không gửi email/SMS thật trong CI
- Không trừ tiền payment gateway trong unit test
- Contract test / integration bổ sung chỗ mock không cover được

## 16. Liên hệ Unity

- Inject `IAudioService` thay `AudioSource.Play` trực tiếp → Edit Mode test được
- Fake `ISaveSystem` in-memory thay ghi `Application.persistentDataPath`
- Play Mode vẫn cần integration cho Physics — mock không thay hết

## 17. Kiểm tra kiến thức

1. Vì sao cần mock/fake?  
   **Đáp án:** Cô lập SUT, tránh side-effect, test nhanh/ổn định.

2. Fake khác mock (theo nghĩa verify) thế nào?  
   **Đáp án:** Fake là implementation nhẹ; mock thường nhấn kiểm tra tương tác.

3. Nên mock SUT không?  
   **Đáp án:** Không — mock dependency của SUT.

4. `DateTime.Now` trong logic nghiệp vụ gây gì?  
   **Đáp án:** Test flaky / khó kiểm soát thời gian → inject clock.

5. Over-verify là gì?  
   **Đáp án:** Assert quá nhiều lời gọi chi tiết → test dễ vỡ khi refactor vô hại.
