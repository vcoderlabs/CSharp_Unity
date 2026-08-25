# Chương 5 — xUnit và NUnit

## 1. Mục tiêu học

- Tạo solution 2 project: app + test
- Viết test với **xUnit** (`Fact`, `Theory`, `InlineData`)
- So sánh nhanh với **NUnit** (`Test`, `TestCase`, `SetUp`)
- Chạy `dotnet test` và đọc kết quả fail

## 2. Điều kiện tiên quyết

- Chương 1–4
- .NET SDK 8+ (hoặc 6+)
- Biết `dotnet new`, `dotnet add reference`

## 3. Khái niệm

**xUnit.net** — framework test phổ biến trên .NET hiện đại; mặc định trong nhiều template. Mỗi test class thường **new instance per test** (ít state dùng chung hơn).

**NUnit** — lâu đời, attribute phong phú (`SetUp`, `TearDown`, `OneTimeSetUp`, constraints fluent).

Cả hai đều chạy qua `dotnet test` (VSTest).

| Khái niệm | xUnit | NUnit |
|-----------|-------|-------|
| Test method | `[Fact]` | `[Test]` |
| Data-driven | `[Theory]` + `[InlineData]` | `[TestCase]` |
| Init mỗi test | ctor của test class | `[SetUp]` |
| Assert | `Assert.Equal` | `Assert.That` / `Assert.AreEqual` |

Level này **ưu tiên xUnit** cho project; NUnit học để đọc codebase khác.

## 4. Mô hình tư duy

```text
Solution
├── MyApp/              ← production code
└── MyApp.Tests/        ← chỉ test; reference MyApp
        └── *.cs        ← Fact/Theory

dotnet test
   → build
   → khám phá attribute
   → chạy từng test
   → báo pass/fail
```

## 5. Cú pháp

### Tạo project (xUnit)

```bash
dotnet new sln -n TestingDemo
dotnet new classlib -n TestingDemo -f net8.0 -o src/TestingDemo
dotnet new xunit -n TestingDemo.Tests -f net8.0 -o tests/TestingDemo.Tests
dotnet sln add src/TestingDemo/TestingDemo.csproj
dotnet sln add tests/TestingDemo.Tests/TestingDemo.Tests.csproj
dotnet add tests/TestingDemo.Tests reference src/TestingDemo
```

### xUnit

```csharp
using Xunit;

public class MathTests
{
    [Fact]
    public void Add_2And3_Returns5()
        => Assert.Equal(5, Calculator.Add(2, 3));

    [Theory]
    [InlineData(4, 2, 2)]
    [InlineData(9, 3, 3)]
    public void Divide_Valid_ReturnsQuotient(int a, int b, int expected)
        => Assert.Equal(expected, Calculator.Divide(a, b));
}
```

### NUnit tương đương

```csharp
using NUnit.Framework;

public class MathTests
{
    [Test]
    public void Add_2And3_Returns5()
        => Assert.That(Calculator.Add(2, 3), Is.EqualTo(5));

    [TestCase(4, 2, 2)]
    [TestCase(9, 3, 3)]
    public void Divide_Valid_ReturnsQuotient(int a, int b, int expected)
        => Assert.That(Calculator.Divide(a, b), Is.EqualTo(expected));
}
```

Tạo NUnit project: `dotnet new nunit -n My.Tests`.

## 6. Ví dụ

### Cơ bản — Fact fail có message

```csharp
[Fact]
public void Multiply_3And4_Returns12()
{
    int actual = Calculator.Multiply(3, 4);
    Assert.Equal(12, actual);
}
```

### Trung cấp — Collection + Throws

```csharp
[Fact]
public void ParseTags_SplitsByComma()
{
    var tags = TagParser.Parse("a, b, c");
    Assert.Equal(new[] { "a", "b", "c" }, tags);
}

[Fact]
public void Divide_ByZero_Throws()
{
    Assert.Throws<DivideByZeroException>(() => Calculator.Divide(1, 0));
}
```

### Nâng cao — Shared context xUnit (IClassFixture)

Khi setup đắt (nhẹ integration), dùng fixture — **cẩn thận state**:

```csharp
public class DatabaseFixture : IDisposable
{
    public string ConnectionString { get; } = "Data Source=:memory:";
    public DatabaseFixture() { /* tạo schema */ }
    public void Dispose() { /* cleanup */ }
}

public class OrderIntegrationTests : IClassFixture<DatabaseFixture>
{
    private readonly DatabaseFixture _fx;
    public OrderIntegrationTests(DatabaseFixture fx) => _fx = fx;

    [Fact]
    public void CanConnect()
    {
        Assert.False(string.IsNullOrEmpty(_fx.ConnectionString));
    }
}
```

Với unit thuần: **tránh** fixture — Arrange trong từng test rõ hơn.

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| 0 tests found | Sai SDK / thiếu package / namespace | Kiểm tra `Microsoft.NET.Test.Sdk`, xunit.runner |
| Không thấy type production | Chưa project reference | `dotnet add reference` |
| `Assert` ambiguous | Dùng cả NUnit + xUnit | Một framework / file |
| Theory không chạy đủ case | Sai attribute | `InlineData` đúng kiểu tham số |
| Internal class không test được | Không visible | `InternalsVisibleTo` |

```csharp
// Production.csproj
[assembly: InternalsVisibleTo("TestingDemo.Tests")]
```

## 8. Gỡ lỗi

1. `dotnet test -v n` — xem discovery.
2. Fail: đọc Expected/Actual; mở test trong IDE, debug test method.
3. “Build failed” trước khi chạy test — sửa compile trước.
4. Filter: `dotnet test --filter FullyQualifiedName~Withdraw`.

## 9. Best practices

- Một test project / solution (hoặc theo layer sau này).
- Tên test project: `*.Tests`.
- Không reference test → production ngược chiều.
- Prefer xUnit cho greenfield .NET; giữ NUnit nếu team đang dùng.
- Không share mutable static giữa tests.
- Chạy test trước khi push.

## 10. Bài tập

**Bài 1** — Tạo solution + classlib `Calc` + xunit `Calc.Tests`. Method `Add`. Ít nhất 2 Fact.

**Bài 2** — Thêm `Theory` cho `IsPrime` (bạn tự implement đơn giản) với ≥ 3 InlineData.

**Bài 3** — Viết lại 1 Fact của bài 1 bằng NUnit project riêng (hoặc đọc docs và viết snippet tương đương trong markdown nếu không muốn 2 framework cùng lúc).

**Bài 4** — Dùng `--filter` chạy đúng một test; ghi lệnh vào README nhỏ.

## 11. Gợi ý

- Bài 1: theo đúng lệnh mục 5.
- Bài 2: chỉ check chia hết tới sqrt(n) là đủ học.
- Bài 3: `[Test]` + `Assert.That(..., Is.EqualTo(...))`.
- Bài 4: `dotnet test --filter Add_2And3_Returns5`.

## 12. Đáp án

**Bài 1 (cốt lõi):**

```csharp
namespace Calc;

public static class Calculator
{
    public static int Add(int a, int b) => a + b;
}

// tests
public class CalculatorTests
{
    [Fact]
    public void Add_2And3_Returns5() => Assert.Equal(5, Calculator.Add(2, 3));

    [Fact]
    public void Add_Zero_ReturnsSame() => Assert.Equal(7, Calculator.Add(7, 0));
}
```

**Bài 2:**

```csharp
[Theory]
[InlineData(2, true)]
[InlineData(4, false)]
[InlineData(17, true)]
public void IsPrime_ReturnsExpected(int n, bool expected)
    => Assert.Equal(expected, NumberTheory.IsPrime(n));
```

**Bài 3:** `[Test] public void Add_2And3_Returns5() => Assert.That(Calculator.Add(2, 3), Is.EqualTo(5));`

**Bài 4:** `dotnet test --filter FullyQualifiedName~Add_2And3_Returns5`

## 13. Đáp án thay thế

Dùng MSTest (`[TestMethod]`) — cũng được `dotnet new mstest`; Level này không bắt buộc nhưng gặp nhiều trên codebase cũ.

## 14. Thử thách

Thêm coverlet / `dotnet test --collect:"XPlat Code Coverage"` và mở báo cáo coverage (học công cụ, không cần 100%).

## 15. Ứng dụng thực tế

- CI GitHub Actions: step `dotnet test`
- IDE: Test Explorer
- Mutation testing / coverage gate ở team lớn

## 16. Liên hệ Unity

- Unity Test Framework dựa trên NUnit (Edit/Play Mode)
- Attribute `[Test]`, `[UnityTest]` (coroutine) — khác xUnit thuần
- Logic tách khỏi MonoBehaviour vẫn có thể test bằng xUnit ngoài Unity khi thuần C#

## 17. Kiểm tra kiến thức

1. Attribute test đơn trong xUnit?  
   **Đáp án:** `[Fact]`.

2. Data-driven trong xUnit?  
   **Đáp án:** `[Theory]` + `[InlineData]` (hoặc MemberData…).

3. Lệnh chạy test?  
   **Đáp án:** `dotnet test`.

4. NUnit tương đương Fact?  
   **Đáp án:** `[Test]`.

5. Vì sao tách project Tests riêng?  
   **Đáp án:** Không đưa package test vào production; tách reference rõ ràng.
