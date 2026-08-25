# Chương 3 — Encapsulation và Access Modifiers

## 1. Mục tiêu học

- Hiểu **encapsulation**: giấu chi tiết, chỉ mở API cần thiết
- Dùng đúng `public`, `private`, `protected`, `internal`, `protected internal`, `private protected`
- Bảo vệ invariant bằng property/`private set` và method
- Tránh “god object” lộ hết field

## 2. Điều kiện tiên quyết

- Chương 1–2: class, property, constructor
- Biết project có thể có nhiều file / namespace (Level 1)

## 3. Khái niệm

**Encapsulation** = đóng gói dữ liệu + quy tắc đi kèm. Bên ngoài không được “séc” trực tiếp vào nội thất nếu điều đó phá luật.

Đời thường: két ATM — bạn không chỉnh số dư trong DB; bạn gọi `Rút tiền` với PIN và hạn mức.

Game: không cho UI set `Hp = -999`; chỉ `TakeDamage` / `Heal` với clamp.

**Access modifier** kiểm soát *ai được thấy* thành viên:

| Modifier | Ai thấy |
|----------|---------|
| `private` | Chỉ trong class (mặc định của thành viên class) |
| `public` | Mọi nơi |
| `protected` | Class + class con |
| `internal` | Trong cùng assembly |
| `protected internal` | Assembly **hoặc** class con |
| `private protected` | Class con **và** cùng assembly |

API công khai nên mỏng; phần còn lại `private`.

## 4. Mô hình tư duy

```text
        ┌─────────────────────────────┐
        │         public API          │  ← Deposit, Withdraw, Balance (get)
        │  ┌───────────────────────┐  │
        │  │  private fields/rules │  │  ← _balance, validate amount
        │  └───────────────────────┘  │
        └─────────────────────────────┘

Bên ngoài chỉ nói chuyện qua cửa công khai.
Invariant: Balance không âm (trừ khi rule overdraft cho phép).
```

## 5. Cú pháp

Giải thích: số dư ẩn; chỉ thay đổi qua method có kiểm tra.

```csharp
public class Wallet
{
    private decimal _balance;

    public decimal Balance => _balance;

    public bool Deposit(decimal amount)
    {
        if (amount <= 0) return false;
        _balance += amount;
        return true;
    }

    public bool Withdraw(decimal amount)
    {
        if (amount <= 0 || amount > _balance) return false;
        _balance -= amount;
        return true;
    }
}

var w = new Wallet();
w.Deposit(100);
Console.WriteLine(w.Withdraw(30));  // True
Console.WriteLine(w.Balance);       // 70
// w._balance = 999; // lỗi biên dịch nếu gọi từ ngoài
```

## 6. Ví dụ

### Cơ bản

Giải thích: `Age` chỉ nhận 0–150.

```csharp
public class Person
{
    private int _age;
    public string Name { get; set; } = "";

    public int Age
    {
        get => _age;
        set => _age = Math.Clamp(value, 0, 150);
    }
}
```

### Trung cấp

Giải thích: `internal` helper chỉ dùng trong assembly; API public gọi helper.

```csharp
public class Order
{
    private readonly List<string> _items = new();
    public IReadOnlyList<string> Items => _items;

    public void AddItem(string sku)
    {
        if (IsValidSku(sku))
            _items.Add(sku.Trim().ToUpperInvariant());
    }

    internal static bool IsValidSku(string sku)
        => !string.IsNullOrWhiteSpace(sku) && sku.Length <= 32;
}
```

### Nâng cao

Giải thích: bảo vệ collection — không trả `List` mutable ra ngoài.

```csharp
public class Party
{
    private readonly List<string> _members = new();

    public IReadOnlyCollection<string> Members => _members;

    public bool Invite(string name)
    {
        if (string.IsNullOrWhiteSpace(name)) return false;
        if (_members.Contains(name)) return false;
        _members.Add(name);
        return true;
    }
}
```

## 7. Lỗi thường gặp

| Lỗi | Nguyên nhân | Cách xử lý |
|-----|-------------|------------|
| Mọi field `public` | Tiện ngắn hạn | Property + method |
| Trả `List<T>` nội bộ | Bên ngoài `Clear()` phá dữ liệu | `IReadOnlyList` / copy |
| `protected` lạm dụng | Cây inheritance sâu | Prefer private + composition |
| Quên invariant | Set trực tiếp phá rule | Một đường vào duy nhất |

## 8. Gỡ lỗi

1. Nếu bug “số dư âm”: tìm mọi chỗ gán — chỉ được 1–2 method.
2. IDE: Find Usages trên property để xem ai ghi.
3. Viết unit test nhỏ cho rule (Deposit/Withdraw).
4. Review: thành viên nào `public` mà không cần?

## 9. Best practices

- Mặc định `private`; mở dần khi cần.
- `public` = cam kết ổn định API.
- Validate ở biên (constructor/method/property set).
- Không expose mutable internals.
- `internal` cho chi tiết assembly; tốt khi tách library.

## 10. Bài tập

**Bài 1 — Temperature**  
Class lưu Celsius private; property `Celsius` và `Fahrenheit` (get/set đổi qua công thức).

**Bài 2 — PasswordBox**  
`SetPassword`, `Verify(string)` — không có getter password.

**Bài 3 — ScoreBoard**  
Chỉ `AddPoints(int)`; điểm không giảm; in điểm.

**Bài 4 — BankAccount rút**  
`Withdraw` trả `bool`; không âm. Nhập lệnh D/W từ console.

**Bài 5 — Readonly view**  
`Playlist` thêm bài hát; expose `IReadOnlyList<string>`.

## 11. Gợi ý

- Bài 1: set Fahrenheit → đổi `_celsius` bên trong.
- Bài 2: so sánh chuỗi đơn giản (chưa hash — Level bảo mật sau).

## 12. Đáp án + Giải thích

### Bài 1

Giải thích: một field gốc Celsius; Fahrenheit quy đổi.

```csharp
public class Temperature
{
    private double _celsius;
    public double Celsius
    {
        get => _celsius;
        set => _celsius = value;
    }
    public double Fahrenheit
    {
        get => _celsius * 9 / 5 + 32;
        set => _celsius = (value - 32) * 5 / 9;
    }
}

var t = new Temperature { Fahrenheit = 212 };
Console.WriteLine(t.Celsius); // 100
```

### Bài 2

Giải thích: không lộ password; chỉ verify.

```csharp
public class PasswordBox
{
    private string _password = "";
    public void SetPassword(string p) => _password = p ?? "";
    public bool Verify(string p) => _password == (p ?? "");
}
```

### Bài 3

Giải thích: điểm chỉ tăng qua `AddPoints`.

```csharp
public class ScoreBoard
{
    public int Score { get; private set; }
    public void AddPoints(int p) { if (p > 0) Score += p; }
}
```

### Bài 4

Giải thích: menu nạp/rút với encapsulation.

```csharp
public class BankAccount
{
    public decimal Balance { get; private set; }
    public void Deposit(decimal a) { if (a > 0) Balance += a; }
    public bool Withdraw(decimal a)
    {
        if (a <= 0 || a > Balance) return false;
        Balance -= a;
        return true;
    }
}

var acc = new BankAccount();
while (true)
{
    Console.Write("D/W/Q amount?: ");
    var line = Console.ReadLine() ?? "";
    var p = line.Split(' ', StringSplitOptions.RemoveEmptyEntries);
    if (p.Length == 0) continue;
    if (p[0].Equals("Q", StringComparison.OrdinalIgnoreCase)) break;
    if (p.Length < 2 || !decimal.TryParse(p[1], out var amt)) { Console.WriteLine("Bad"); continue; }
    if (p[0].Equals("D", StringComparison.OrdinalIgnoreCase)) acc.Deposit(amt);
    else if (p[0].Equals("W", StringComparison.OrdinalIgnoreCase))
        Console.WriteLine(acc.Withdraw(amt) ? "OK" : "Fail");
    Console.WriteLine($"Balance={acc.Balance}");
}
```

### Bài 5

Giải thích: list nội bộ; bên ngoài chỉ đọc.

```csharp
public class Playlist
{
    private readonly List<string> _songs = new();
    public IReadOnlyList<string> Songs => _songs;
    public void Add(string s)
    {
        if (!string.IsNullOrWhiteSpace(s)) _songs.Add(s.Trim());
    }
}
```

## 13. Đáp án thay thế

- Dùng `init` accessor (C# 9+) cho set-một-lần từ object initializer.
- Record với private ctor cho value object bất biến.

## 14. Thử thách

`VendingMachine`: tồn kho private; `InsertCoin`, `Select(code)` trả item hoặc lỗi; không cho âm tồn.

## 15. Ứng dụng thực tế

- API library: chỉ public những gì client cần.
- Bảo mật nghiệp vụ: số dư, quyền, mật khẩu không lộ field.
- Refactor: đổi cấu trúc private mà không phá caller.

## 16. Liên hệ Unity

- `[SerializeField] private` = Inspector thấy, code ngoài không (encapsulation tốt).
- Tránh `public` field chỉ vì “kéo thả Inspector”.
- UI không ghi thẳng `player.hp` — gọi method trên `Health` component.
- `internal` ít gặp trong script Unity thông thường; vẫn hữu ích trong package assembly.

## 17. Kiểm tra kiến thức

1. Encapsulation giải quyết vấn đề gì?  
2. `private` khác `protected`?  
3. Vì sao không return `List<T>` nội bộ?  
4. `internal` giới hạn phạm vi nào?  
5. Invariant là gì (ví dụ)?

**Đáp án:**  
1) Giữ rule/data an toàn; giảm phụ thuộc chi tiết.  
2) `private` chỉ class đó; `protected` thêm class con.  
3) Caller có thể sửa phá invariant.  
4) Cùng assembly.  
5) Điều luôn đúng, ví dụ Balance ≥ 0.
