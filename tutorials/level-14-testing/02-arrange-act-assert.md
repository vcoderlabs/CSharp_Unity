# Chương 2 — Arrange–Act–Assert (AAA)

## 1. Mục tiêu học

- Viết test theo cấu trúc **Arrange → Act → Assert**
- Đặt tên test mô tả hành vi
- Dùng assert phổ biến: Equal, True, Throws, NotNull, Collection
- Tránh test “thần thánh” (quá nhiều assert / quá nhiều việc)

## 2. Điều kiện tiên quyết

- Chương 1: unit vs integration
- Biết tạo class C# và method trả về giá trị / throw

## 3. Khái niệm

**AAA** là khuôn mẫu đọc test:

| Pha | Việc làm |
|-----|----------|
| **Arrange** | Chuẩn bị data, SUT, fake/mock |
| **Act** | Gọi **một** hành động chính |
| **Assert** | Kiểm tra kết quả / trạng thái / exception |

Tên khác: **Given–When–Then** (BDD-ish) — cùng ý.

```text
Given calculator với số dư 100
When withdraw 40
Then số dư còn 60
```

### Đặt tên test

Hai style phổ biến:

```csharp
// Style 1
Add_TwoNumbers_ReturnsSum()

// Style 2
GivenPositiveInputs_WhenAdd_ThenReturnsSum()
```

Chọn một style trong project và giữ nhất quán.

## 4. Mô hình tư duy

```text
Đọc test như câu chuyện ngắn:

  Arrange  = bối cảnh (không assert ở đây)
  Act      = 1 dòng / 1 ý chính (gọi method)
  Assert   = kỳ vọng quan sát được

Nếu Act có 5 bước → có thể đang test integration hoặc test quá to.
Nếu Assert 15 điều → tách test hoặc assert quá tham.
```

## 5. Cú pháp

```csharp
[Fact]
public void MethodName_Scenario_Expected()
{
    // Arrange
    var sut = new BankAccount(initialBalance: 100m);

    // Act
    sut.Withdraw(40m);

    // Assert
    Assert.Equal(60m, sut.Balance);
}
```

Assert hay dùng (xUnit):

```csharp
Assert.Equal(expected, actual);
Assert.True(condition);
Assert.False(condition);
Assert.Null(value);
Assert.NotNull(value);
Assert.Throws<ArgumentException>(() => sut.DoBadThing());
Assert.Contains(item, collection);
Assert.Empty(collection);
```

## 6. Ví dụ

### Cơ bản — AAA rõ ràng

**Trước** (khó đọc):

```csharp
[Fact]
public void TestWithdraw()
{
    Assert.Equal(60m, new BankAccount(100m).Also(a => a.Withdraw(40m)).Balance);
}
```

**Sau:**

```csharp
public class BankAccount
{
    public decimal Balance { get; private set; }
    public BankAccount(decimal balance) => Balance = balance;

    public void Withdraw(decimal amount)
    {
        if (amount <= 0) throw new ArgumentOutOfRangeException(nameof(amount));
        if (amount > Balance) throw new InvalidOperationException("insufficient");
        Balance -= amount;
    }
}

[Fact]
public void Withdraw_ValidAmount_DecreasesBalance()
{
    // Arrange
    var account = new BankAccount(100m);

    // Act
    account.Withdraw(40m);

    // Assert
    Assert.Equal(60m, account.Balance);
}
```

### Trung cấp — Assert exception + message

```csharp
[Fact]
public void Withdraw_AmountGreaterThanBalance_Throws()
{
    // Arrange
    var account = new BankAccount(50m);

    // Act + Assert
    var ex = Assert.Throws<InvalidOperationException>(() => account.Withdraw(80m));
    Assert.Contains("insufficient", ex.Message);
}
```

Khi Act chính **là** việc throw, gộp Act+Assert trong `Assert.Throws` là chấp nhận được — vẫn rõ ý.

### Nâng cao — Nhiều case với Theory

```csharp
[Theory]
[InlineData(100, 40, 60)]
[InlineData(100, 100, 0)]
[InlineData(10, 3, 7)]
public void Withdraw_ValidAmounts_UpdatesBalance(decimal initial, decimal take, decimal expected)
{
    var account = new BankAccount(initial);
    account.Withdraw(take);
    Assert.Equal(expected, account.Balance);
}
```

Arrange/Act/Assert vẫn giữ — dữ liệu biến thiên nằm ở `InlineData`.

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Assert ngay trong Arrange | Setup lẫn verify | Chỉ chuẩn bị ở Arrange |
| Nhiều Act | Test quá to | Tách test theo hành vi |
| `Assert.True(a == b)` | Message kém | Dùng `Assert.Equal(a, b)` |
| Test phụ thuộc test khác | State dùng chung | Mỗi test tự Arrange |
| Tên `Test1` | Không mô tả | Đổi theo scenario |

## 8. Gỡ lỗi

1. Đọc **Assert message**: Expected vs Actual.
2. Nếu Actual bất ngờ — đặt breakpoint ở Act, không “đoán”.
3. Exception không đúng type → kiểm tra code path / validate order.
4. Collection fail → in `string.Join(",", list)` tạm trong lúc debug (xóa trước khi commit).

## 9. Best practices

- Một hành vi chính mỗi test.
- AAA cách nhau bằng dòng trống hoặc comment `// Arrange`.
- Prefer `Equal` hơn `True` với so sánh giá trị.
- Đặt biến `expected` / `actual` khi assert phức tạp.
- Đừng test implementation detail (private field) nếu có thể quan sát qua public API.
- Test đọc được như tài liệu — đồng nghiệp đọc tên test hiểu feature.

## 10. Bài tập

**Bài 1** — Viết `StringCounter.CountWords(string text)` (tách bằng khoảng trắng). Test: `"a b c"` → 3; `""` → 0; `"  x  "` → 1.

**Bài 2** — `StackService` với `Push`/`Pop`. Test Pop khi rỗng throw `InvalidOperationException` theo AAA.

**Bài 3** — Đổi tên 3 test xấu: `Test`, `Works`, `Check` thành tên AAA-style cho `Calculator.Divide`.

**Bài 4** — Viết Theory cho `MathOps.IsEven(int n)` với ít nhất 4 InlineData.

## 11. Gợi ý

- Bài 1: `text.Split(' ', StringSplitOptions.RemoveEmptyEntries).Length`.
- Bài 2: Arrange stack rỗng → Act Pop trong Throws.
- Bài 3: `Divide_ByZero_Throws`, `Divide_6By3_Returns2`, …
- Bài 4: `[InlineData(2, true)]`, `[InlineData(3, false)]`, …

## 12. Đáp án

**Bài 1:**

```csharp
public class StringCounter
{
    public int CountWords(string text)
    {
        if (string.IsNullOrWhiteSpace(text)) return 0;
        return text.Split(' ', StringSplitOptions.RemoveEmptyEntries).Length;
    }
}

[Theory]
[InlineData("a b c", 3)]
[InlineData("", 0)]
[InlineData("  x  ", 1)]
public void CountWords_VariousInputs_ReturnsExpected(string text, int expected)
{
    var sut = new StringCounter();
    Assert.Equal(expected, sut.CountWords(text));
}
```

**Bài 2:**

```csharp
[Fact]
public void Pop_EmptyStack_ThrowsInvalidOperation()
{
    var stack = new StackService();
    Assert.Throws<InvalidOperationException>(() => stack.Pop());
}
```

**Bài 3 (ví dụ tên):**

- `Divide_ByZero_ThrowsDivideByZeroException`
- `Divide_SixByThree_ReturnsTwo`
- `Divide_NegativeByPositive_ReturnsNegative`

**Bài 4:**

```csharp
[Theory]
[InlineData(2, true)]
[InlineData(3, false)]
[InlineData(0, true)]
[InlineData(-4, true)]
public void IsEven_ReturnsExpected(int n, bool expected)
    => Assert.Equal(expected, MathOps.IsEven(n));
```

## 13. Đáp án thay thế

Bài 1 dùng regex `\S+` nếu muốn đếm token phức tạp hơn. Bài 2 có thể dùng `TryPop` trả `false` thay vì throw — test theo contract đã chọn.

## 14. Thử thách

Refactor một test “messy” (bạn tự viết cố ý xấu: không AAA, nhiều Act) thành 2–3 test sạch. Commit message giả: `test: clarify withdraw scenarios`.

## 15. Ứng dụng thực tế

- Code review: reviewer đọc tên test trước implementation
- Bug report → viết test AAA tái hiện → rồi fix
- Characterization test: Arrange state legacy, Act, Assert hành vi hiện tại trước khi refactor

## 16. Liên hệ Unity

- Edit Mode: AAA cho damage formula, inventory weight
- Play Mode: Arrange spawn prefab → Act trigger collision → Assert HP
- Tránh `yield return null` lung tung không rõ đang Act bước nào — comment AAA trong coroutine test

## 17. Kiểm tra kiến thức

1. Ba chữ A trong AAA là gì?  
   **Đáp án:** Arrange, Act, Assert.

2. Vì sao nên hạn chế nhiều Act trong một test?  
   **Đáp án:** Khó biết bước nào gây fail; test không còn là tài liệu rõ ràng.

3. `Assert.Throws` dùng khi nào?  
   **Đáp án:** Khi hành vi kỳ vọng là ném exception.

4. `Theory` + `InlineData` giúp gì?  
   **Đáp án:** Chạy cùng logic assert với nhiều bộ dữ liệu.

5. Given–When–Then tương ứng AAA thế nào?  
   **Đáp án:** Given≈Arrange, When≈Act, Then≈Assert.
