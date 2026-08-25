# Project Level 2 — Banking System

Hệ thống ngân hàng console: **Account**, **SavingsAccount**, **CheckingAccount**, **TransactionHistory**, **tính lãi** — tổng hợp OOP Level 2 (encapsulation, inheritance, polymorphism, interface, composition).

**Thời lượng đề xuất:** 6–8 giờ  
**Target:** .NET 8+ console app  
**Độ khó:** Trung cấp–nâng cao (cuối Level 2)

---

## 1. Mục tiêu học

- Thiết kế phân cấp account hợp lý (abstract + subclass) **và** biết chỗ dùng composition (`TransactionHistory`)
- Áp dụng encapsulation cho số dư / giao dịch
- Đa hình: xử lý danh sách `Account` (áp lãi, in sao kê)
- Hoàn thành app menu: mở tài khoản, nạp/rút, lãi, lịch sử, liệt kê

## 2. Điều kiện tiên quyết

- Đã học (hoặc đọc kèm) chương 1–8 Level 2
- Thành thạo `dotnet new console`, property, constructor, `virtual`/`abstract`, interface cơ bản
- Biết `List<T>`, `DateTime` (Level 1)

## 3. Khái niệm / Đặc tả sản phẩm

### Chức năng bắt buộc (MVP)

1. **Account (abstract hoặc base rõ ràng)**
   - `AccountNumber`, `Owner`, `Balance` (không set công khai lung tung)
   - `Deposit(amount)`, `Withdraw(amount)` → thành công/thất bại
2. **SavingsAccount**
   - `InterestRate` (ví dụ 0.5%/tháng)
   - `ApplyInterest()` cộng lãi vào số dư + ghi lịch sử
3. **CheckingAccount**
   - Phí rút (`WithdrawalFee`) hoặc overdraft limit đơn giản
   - Rule khác Savings (đa hình `Withdraw` / `ApplyMonthly`)
4. **TransactionHistory** (composition — **không** phải base class của Account)
   - Lưu các bản ghi: thời gian, loại (`Deposit`/`Withdraw`/`Interest`/`Fee`), số tiền, số dư sau
   - In N giao dịch gần nhất
5. **Menu console**
   - Tạo Savings / Checking
   - Nạp / Rút theo số tài khoản
   - Áp lãi (tất cả Savings hoặc một account)
   - Xem lịch sử / liệt kê tài khoản
   - Thoát

### Chức năng khuyến khích

- Interface `IInterestBearing` cho account có lãi
- Mã tài khoản tự tăng (`static` IdGenerator)
- Xuất lịch sử ra file text (Level 12 sẽ sâu hơn — optional)
- Validation input thân thiện (không crash)

### Không bắt buộc

- Đa user login, database, encryption PIN — vượt Level 2

## 4. Mô hình tư duy

```text
┌──────────────────────────────────────────────┐
│                   BankApp                    │
│  Dictionary/List Account + menu loop         │
└──────────────────────┬───────────────────────┘
                       │ owns
                       ▼
              ┌────────────────┐
              │ Account (abs)  │  has-a  ┌─────────────────────┐
              │ Deposit/Withdraw│────────►│ TransactionHistory │
              └───────┬────────┘         └─────────────────────┘
                 ┌────┴─────┐
                 ▼          ▼
          SavingsAccount  CheckingAccount
          + Interest      + Fee / overdraft

is-a: SavingsAccount is Account ✓
has-a: Account has TransactionHistory ✓
KHÔNG: Account : TransactionHistory ✗
```

## 5. Cú pháp / Skeleton gợi ý

Tạo project:

```bash
dotnet new console -n BankingSystem
cd BankingSystem
dotnet run
```

Giải thích: khung abstract Account gắn history bằng composition.

```csharp
public sealed class Transaction
{
    public DateTime At { get; init; }
    public string Type { get; init; } = "";
    public decimal Amount { get; init; }
    public decimal BalanceAfter { get; init; }
    public override string ToString()
        => $"{At:yyyy-MM-dd HH:mm} {Type,-10} {Amount,10:F2} → {BalanceAfter:F2}";
}

public sealed class TransactionHistory
{
    private readonly List<Transaction> _items = new();
    public IReadOnlyList<Transaction> Items => _items;

    public void Add(string type, decimal amount, decimal balanceAfter)
    {
        _items.Add(new Transaction
        {
            At = DateTime.Now,
            Type = type,
            Amount = amount,
            BalanceAfter = balanceAfter
        });
    }

    public void PrintLast(int n)
    {
        foreach (var t in _items.TakeLast(n))
            Console.WriteLine(t);
    }
}

public abstract class Account
{
    private static int _next = 1000;
    public string AccountNumber { get; }
    public string Owner { get; }
    public decimal Balance { get; protected set; }
    public TransactionHistory History { get; } = new();

    protected Account(string owner, decimal opening)
    {
        AccountNumber = (++_next).ToString();
        Owner = owner;
        Balance = opening < 0 ? 0 : opening;
        if (opening > 0)
            History.Add("Open", opening, Balance);
    }

    public virtual bool Deposit(decimal amount)
    {
        if (amount <= 0) return false;
        Balance += amount;
        History.Add("Deposit", amount, Balance);
        return true;
    }

    public abstract bool Withdraw(decimal amount);
    public abstract string Summary();
}
```

Giải thích: Savings có lãi; Checking trừ phí khi rút.

```csharp
public interface IInterestBearing
{
    decimal InterestRate { get; }
    void ApplyInterest();
}

public class SavingsAccount : Account, IInterestBearing
{
    public decimal InterestRate { get; }
    public SavingsAccount(string owner, decimal opening, decimal rate)
        : base(owner, opening) => InterestRate = rate;

    public override bool Withdraw(decimal amount)
    {
        if (amount <= 0 || amount > Balance) return false;
        Balance -= amount;
        History.Add("Withdraw", -amount, Balance);
        return true;
    }

    public void ApplyInterest()
    {
        var interest = decimal.Round(Balance * InterestRate, 2);
        if (interest <= 0) return;
        Balance += interest;
        History.Add("Interest", interest, Balance);
    }

    public override string Summary()
        => $"SAV {AccountNumber} {Owner} bal={Balance:F2} rate={InterestRate:P2}";
}

public class CheckingAccount : Account
{
    public decimal WithdrawalFee { get; }
    public CheckingAccount(string owner, decimal opening, decimal fee)
        : base(owner, opening) => WithdrawalFee = fee;

    public override bool Withdraw(decimal amount)
    {
        var total = amount + WithdrawalFee;
        if (amount <= 0 || total > Balance) return false;
        Balance -= total;
        History.Add("Withdraw", -amount, Balance);
        if (WithdrawalFee > 0)
            History.Add("Fee", -WithdrawalFee, Balance);
        return true;
    }

    public override string Summary()
        => $"CHK {AccountNumber} {Owner} bal={Balance:F2} fee={WithdrawalFee:F2}";
}
```

## 6. Ví dụ luồng chạy (Cơ bản / Trung cấp / Nâng cao)

### Cơ bản

Giải thích: tạo 1 savings, nạp, áp lãi, in lịch sử.

```text
> open savings Lan 1000 0.005
Created 1001
> deposit 1001 200
OK
> interest 1001
OK
> history 1001 5
...
```

### Trung cấp

Nhiều account; `list` in `Summary()` đa hình; rút checking có fee.

### Nâng cao

Lệnh `interest-all` duyệt tất cả `IInterestBearing`; từ chối rút khi không đủ tiền + fee; unit test thủ công các rule.

## 7. Lỗi thường gặp

| Lỗi | Nguyên nhân | Cách xử lý |
|-----|-------------|------------|
| History kế thừa Account | Nhầm has-a thành is-a | Composition field |
| Balance public set | Phá invariant | `protected set` + method |
| Lãi không ghi history | Quên Add | Ghi mọi thay đổi số dư |
| So sánh reference account sai | Tìm nhầm key | Dictionary theo AccountNumber |
| `decimal` vs `double` tiền | Sai kiểu | Dùng `decimal` |

## 8. Gỡ lỗi

1. In `Summary()` sau mỗi giao dịch khi dev.
2. Breakpoint trong `Withdraw` — kiểm tra fee + điều kiện.
3. Đảm bảo mọi đường đổi `Balance` đều `History.Add`.
4. Test case: rút đúng bằng balance (checking phải tính fee).

## 9. Best practices

- Tiền: `decimal`.
- Một đường vào thay đổi số dư.
- Interface cho lãi — Checking không bị ép `ApplyInterest`.
- Menu mỏng; logic trong domain class.
- Không deep inheritance thêm `VipSuperSavingsMegaAccount` — mở rộng bằng strategy/flag/composition nếu cần.

## 10. Bài tập (chia nhỏ trước khi full app)

**Bài 1** — Chỉ `TransactionHistory` + in 3 giao dịch giả.  
**Bài 2** — `SavingsAccount` deposit/withdraw/interest.  
**Bài 3** — `CheckingAccount` withdraw có fee.  
**Bài 4** — `Bank` chứa `List<Account>`, tìm theo số TK.  
**Bài 5** — Menu đầy đủ MVP.

## 11. Gợi ý

- Parse lệnh: `Split` + `decimal.TryParse`.
- Lưu account trong `Dictionary<string, Account>`.
- `interest-all`: `foreach` + pattern `if (acc is IInterestBearing ib) ib.ApplyInterest();`

## 12. Đáp án + Giải thích (Bank service + menu rút gọn)

Giải thích: lớp `Bank` quản lý danh sách; menu gọi method bank.

```csharp
public class Bank
{
    private readonly Dictionary<string, Account> _accounts = new();

    public Account OpenSavings(string owner, decimal opening, decimal rate)
    {
        var a = new SavingsAccount(owner, opening, rate);
        _accounts[a.AccountNumber] = a;
        return a;
    }

    public Account OpenChecking(string owner, decimal opening, decimal fee)
    {
        var a = new CheckingAccount(owner, opening, fee);
        _accounts[a.AccountNumber] = a;
        return a;
    }

    public Account? Find(string number)
        => _accounts.TryGetValue(number, out var a) ? a : null;

    public IEnumerable<Account> All() => _accounts.Values;

    public void ApplyAllInterest()
    {
        foreach (var a in _accounts.Values)
            if (a is IInterestBearing ib)
                ib.ApplyInterest();
    }
}
```

Giải thích: vòng lặp menu tối giản — bạn có thể mở rộng lệnh.

```csharp
var bank = new Bank();
Console.WriteLine("Banking System — gõ help");

while (true)
{
    Console.Write("> ");
    var line = Console.ReadLine();
    if (string.IsNullOrWhiteSpace(line)) continue;
    var p = line.Split(' ', StringSplitOptions.RemoveEmptyEntries);
    var cmd = p[0].ToLowerInvariant();

    if (cmd is "quit" or "exit") break;
    if (cmd == "help")
    {
        Console.WriteLine("open savings|checking <owner> <opening> <rateOrFee>");
        Console.WriteLine("deposit <acc> <amount> | withdraw <acc> <amount>");
        Console.WriteLine("interest <acc> | interest-all | list | history <acc> <n>");
        continue;
    }

    if (cmd == "open" && p.Length >= 5)
    {
        decimal.TryParse(p[3], out var opening);
        decimal.TryParse(p[4], out var x);
        Account acc = p[1].Equals("savings", StringComparison.OrdinalIgnoreCase)
            ? bank.OpenSavings(p[2], opening, x)
            : bank.OpenChecking(p[2], opening, x);
        Console.WriteLine($"Created {acc.AccountNumber}");
        continue;
    }

    if (cmd == "list")
    {
        foreach (var a in bank.All()) Console.WriteLine(a.Summary());
        continue;
    }

    if (cmd == "interest-all")
    {
        bank.ApplyAllInterest();
        Console.WriteLine("OK");
        continue;
    }

    if (p.Length >= 3 && cmd is "deposit" or "withdraw" or "interest" or "history")
    {
        var acc = bank.Find(p[1]);
        if (acc is null) { Console.WriteLine("Not found"); continue; }

        if (cmd == "deposit" && decimal.TryParse(p[2], out var d))
            Console.WriteLine(acc.Deposit(d) ? "OK" : "Fail");
        else if (cmd == "withdraw" && decimal.TryParse(p[2], out var w))
            Console.WriteLine(acc.Withdraw(w) ? "OK" : "Fail");
        else if (cmd == "interest")
        {
            if (acc is IInterestBearing ib) { ib.ApplyInterest(); Console.WriteLine("OK"); }
            else Console.WriteLine("No interest");
        }
        else if (cmd == "history" && int.TryParse(p[2], out var n))
            acc.History.PrintLast(n);
        continue;
    }

    Console.WriteLine("Unknown — gõ help");
}
```

*(Gộp các class ở mục 5 + đoạn này vào project là một đáp án MVP hoàn chỉnh.)*

## 13. Đáp án thay thế

- Không dùng abstract: interface `IAccount` + composition — vẫn chấp nhận nếu rule rõ.
- Checking overdraft: cho phép Balance âm tới hạn mức thay vì fee.
- Primary constructor C# 12 cho `Transaction`.
- Tách file: `Account.cs`, `Bank.cs`, `Program.cs`.

## 14. Thử thách

- Thêm `Transfer(from, to, amount)` atomic (nếu rút fail thì không nạp).
- Kỳ hạn Savings: chỉ `ApplyInterest` nếu đã mở ≥ N ngày (`readonly DateTime OpenedAt`).
- Command `export <acc> path` ghi history ra `.txt`.

## 15. Ứng dụng thực tế

- Core banking / ví điện tử (đơn giản hóa).
- Ledger kế toán: mọi thay đổi số dư có nhật ký.
- Pattern “aggregate + domain events” (mức giới thiệu).

## 16. Liên hệ Unity

- Account ≈ dữ liệu tiến trình; không nhét vào `MonoBehaviour` trừ khi UI.
- `TransactionHistory` giống combat log / quest log component — **has-a**.
- Đừng `Player : Inventory : Wallet` — gắn components/`ScriptableObject` config lãi suất.
- Save game: serialize Balance + history entries (Level serialization).
- Pitfall: static `Bank.Instance` kiểu singleton — biết vòng đời scene; prefer service gắn bootstrap object.

## 17. Kiểm tra kiến thức

1. Vì sao `TransactionHistory` không nên là base của `Account`?  
2. `ApplyInterest` nên nằm interface nào / class nào?  
3. Vì sao dùng `decimal` cho tiền?  
4. `Withdraw` abstract/virtual giúp gì?  
5. Một invariant quan trọng của Account?

**Đáp án:**  
1) Đó là quan hệ has-a (composition), không phải is-a.  
2) `IInterestBearing` / `SavingsAccount` — Checking không bắt buộc.  
3) Tránh lỗi làm tròn binary của `double`.  
4) Đa hình: từng loại account rule rút khác nhau, gọi thống nhất.  
5) Ví dụ: không âm Balance (trừ khi overdraft được định nghĩa rõ); mọi đổi số dư có lịch sử.

---

## Checklist hoàn thành project

- [ ] Tạo được Savings và Checking
- [ ] Deposit / Withdraw đúng rule + không crash input sai
- [ ] Interest cập nhật số dư và history
- [ ] `list` / `history` hoạt động
- [ ] Giải thích được chỗ dùng inheritance vs composition trong code của bạn

Khi xong → sẵn sàng [Level 3 — Value vs Reference](../level-03-value-vs-reference/).
