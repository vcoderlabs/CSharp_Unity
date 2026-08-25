# Chương 7 — Interface: thiết kế contract

## 1. Mục tiêu học

- Hiểu **interface** như hợp đồng: *có thể làm gì*, không phải *là gì*
- Khai báo và implement một / nhiều interface
- Dùng interface như kiểu tham số / collection đa hình
- Biết default interface methods (nhận biết) và khi nào tách interface nhỏ

## 2. Điều kiện tiên quyết

- Chương 5–6: inheritance, polymorphism
- Encapsulation cơ bản

## 3. Khái niệm

**Interface** mô tả **capability** (khả năng). Class **ký hợp đồng** bằng cách implement đủ thành viên.

Đời thường: ổ cắm USB — thiết bị “tuân thủ chuẩn USB” thì máy tính nói chuyện được, không cần biết bên trong là USB nào.

Game: `IDamageable` — Player, Barrel, Enemy đều `TakeDamage`; hệ thống combat không cần biết cụ thể class.

Class chỉ kế thừa **một** class nhưng implement **nhiều** interface: `class Hero : Unit, IDamageable, ISaveable`.

Khác abstract class: interface thường không mang trạng thái field instance cổ điển; tập trung API. (Default methods có thể có body — dùng có chừng.)

## 4. Mô hình tư duy

```text
        IDamageable                 IMovable
        + TakeDamage()              + Move()
              △                          △
              │                          │
         ┌────┴────┐                    │
      Player     Barrel              Player
      (ký cả hai hợp đồng)

CombatSystem.Hit(IDamageable target)
  → chỉ cần TakeDamage — không phụ thuộc Player/Barrel
```

## 5. Cú pháp

Giải thích: định nghĩa contract và class ký kết; gọi qua kiểu interface.

```csharp
public interface IDamageable
{
    int Hp { get; }
    void TakeDamage(int amount);
}

public interface IHealable
{
    void Heal(int amount);
}

public class Player : IDamageable, IHealable
{
    public int Hp { get; private set; } = 100;
    public void TakeDamage(int amount) => Hp = Math.Max(0, Hp - amount);
    public void Heal(int amount) => Hp = Math.Min(100, Hp + amount);
}

void Hit(IDamageable t) => t.TakeDamage(10);

IDamageable p = new Player();
Hit(p);
```

## 6. Ví dụ

### Cơ bản

Giải thích: `ILogger` với 2 implementation.

```csharp
public interface ILogger
{
    void Info(string message);
}

public class ConsoleLogger : ILogger
{
    public void Info(string message) => Console.WriteLine(message);
}

public class PrefixLogger : ILogger
{
    private readonly string _prefix;
    public PrefixLogger(string prefix) => _prefix = prefix;
    public void Info(string message) => Console.WriteLine($"{_prefix}{message}");
}

ILogger log = new PrefixLogger("[APP] ");
log.Info("started");
```

### Trung cấp

Giải thích: repository ảo — dễ thay đổi lưu trữ.

```csharp
public interface IUserRepository
{
    void Add(string user);
    IReadOnlyList<string> All();
}

public class InMemoryUserRepository : IUserRepository
{
    private readonly List<string> _users = new();
    public void Add(string user) => _users.Add(user);
    public IReadOnlyList<string> All() => _users;
}

void Seed(IUserRepository repo) => repo.Add("admin");
```

### Nâng cao

Giải thích: explicit interface implementation khi trùng tên member.

```csharp
public interface IPrintable
{
    void Print();
}
public interface IExportable
{
    void Print(); // cùng tên
}

public class Report : IPrintable, IExportable
{
    void IPrintable.Print() => Console.WriteLine("print UI");
    void IExportable.Print() => Console.WriteLine("print file");
}

var r = new Report();
((IPrintable)r).Print();
((IExportable)r).Print();
```

## 7. Lỗi thường gặp

| Lỗi | Nguyên nhân | Cách xử lý |
|-----|-------------|------------|
| Class thiếu member | Chưa implement đủ | Implement tất cả |
| Interface có field instance | Không khai báo field kiểu class | Dùng property trong contract |
| Interface “béo” | Quá nhiều method không liên quan | Tách interface nhỏ (ISP — SOLID sau) |
| New concrete khắp nơi | Không phụ thuộc abstraction | Nhận `ILogger` thay `ConsoleLogger` |

## 8. Gỡ lỗi

1. Compiler liệt kê member còn thiếu — làm từng cái.
2. Nếu gọi không ra method: đang giữ sai kiểu (explicit impl cần cast).
3. Đặt tên `I` + tính từ/danh từ khả năng: `IComparable`, `IReadOnlyRepository`.
4. Viết test với fake `IUserRepository`.

## 9. Best practices

- Interface nhỏ, đúng một vai trò.
- Phụ thuộc vào interface ở API công khai (parameter/return khi hợp lý).
- Đừng tạo interface cho mọi class một-một nếu chỉ một impl mãi mãi — YAGNI.
- Tên thể hiện capability: `IDamageable` hơn `IPlayerStuff`.

## 10. Bài tập

**Bài 1 — IShape**  
`Area()`; `Circle`, `Rect` implement; in diện tích.

**Bài 2 — IPayment**  
`Pay(decimal)`; `CashPayment`, `CardPayment`.

**Bài 3 — Nhiều interface**  
`Robot : IMovable, ISpeakable`.

**Bài 4 — Service**  
Method `Process(IEnumerable<IShape>)` tổng diện tích.

**Bài 5 — Fake**  
`IClock` với `Now`; `FixedClock` cho test in giờ cố định.

## 11. Gợi ý

- Bài 4: `foreach` và cộng `Area()`.
- Bài 5: production dùng `SystemClock : IClock => DateTime.Now`.

## 12. Đáp án + Giải thích

### Bài 1

Giải thích: contract diện tích.

```csharp
public interface IShape { double Area(); }
public class Circle : IShape
{
    public double R { get; }
    public Circle(double r) => R = r;
    public double Area() => Math.PI * R * R;
}
public class Rect : IShape
{
    public double W { get; }
    public double H { get; }
    public Rect(double w, double h) { W = w; H = h; }
    public double Area() => W * H;
}

IShape s = new Circle(2);
Console.WriteLine(s.Area());
```

### Bài 2

Giải thích: thanh toán đa hình qua interface.

```csharp
public interface IPayment { void Pay(decimal amount); }
public class CashPayment : IPayment
{
    public void Pay(decimal amount) => Console.WriteLine($"Cash {amount}");
}
public class CardPayment : IPayment
{
    public void Pay(decimal amount) => Console.WriteLine($"Card {amount}");
}

IPayment p = new CardPayment();
p.Pay(50);
```

### Bài 3

Giải thích: một class, hai contract.

```csharp
public interface IMovable { void Move(int dx, int dy); }
public interface ISpeakable { void Speak(string text); }

public class Robot : IMovable, ISpeakable
{
    public int X { get; private set; }
    public int Y { get; private set; }
    public void Move(int dx, int dy) { X += dx; Y += dy; }
    public void Speak(string text) => Console.WriteLine($"Robot: {text}");
}

var bot = new Robot();
bot.Move(1, 2);
bot.Speak("Beep");
```

### Bài 4

Giải thích: xử lý theo contract, không theo class cụ thể.

```csharp
static double TotalArea(IEnumerable<IShape> shapes)
{
    double sum = 0;
    foreach (var s in shapes) sum += s.Area();
    return sum;
}

var list = new List<IShape> { new Circle(1), new Rect(2, 3) };
Console.WriteLine(TotalArea(list));
```

### Bài 5

Giải thích: clock giả lập — test ổn định.

```csharp
public interface IClock { DateTime Now { get; } }
public class SystemClock : IClock { public DateTime Now => DateTime.Now; }
public class FixedClock : IClock
{
    public DateTime Now { get; }
    public FixedClock(DateTime now) => Now = now;
}

IClock clock = new FixedClock(new DateTime(2026, 1, 1));
Console.WriteLine(clock.Now);
```

## 13. Đáp án thay thế

- Default interface method cho versioning API (thêm method không phá impl cũ — cân nhắc).
- Generic interface `IRepository<T>` (Level Generics).

## 14. Thử thách

Thiết kế `IInventory` (`TryAdd`, `TryRemove`, `Count`) và 2 impl: unlimited bag vs bag có `capacity`.

## 15. Ứng dụng thực tế

- DI containers đăng ký `IEmailSender`.
- Plugin / driver thay thế.
- Mock trong unit test.

## 16. Liên hệ Unity

- Interface trên component: `IInteractable` — player raycast tìm `GetComponent<IInteractable>()`.
- Tránh mọi collsion phụ thuộc class cụ thể.
- C# interface ≠ Unity `SendMessage` string — type-safe hơn.
- Nhiều capability = nhiều interface nhỏ trên cùng MonoBehaviour hoặc tách component.

## 17. Kiểm tra kiến thức

1. Interface mô tả gì?  
2. Class implement được bao nhiêu interface?  
3. Vì sao combat nên nhận `IDamageable`?  
4. Interface khác abstract class (ý chính)?  
5. Explicit implementation khi nào?

**Đáp án:**  
1) Hợp đồng khả năng / API.  
2) Nhiều.  
3) Không phụ thuộc class cụ thể; mở rộng dễ.  
4) Abstract có thể giữ state + code chung mạnh; interface tập trung contract, đa kế thừa khả năng.  
5) Trùng chữ ký từ nhiều interface hoặc muốn ẩn method khỏi API class.
