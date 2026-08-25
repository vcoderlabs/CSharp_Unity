# Chương 2 — Constructor và Finalizer (Destructor)

## 1. Mục tiêu học

- Viết **constructor** để khởi tạo object đúng trạng thái ngay khi `new`
- Dùng constructor overload và **constructor chaining** (`this(...)`)
- Biết **object initializer** và primary constructor (C# 12) ở mức nhận biết
- Hiểu **finalizer** (`~Class`) là gì — và vì sao hầu như **không** dùng trong app thường / Unity gameplay

## 2. Điều kiện tiên quyết

- Chương 1: class, field, property, method
- Biết method overload từ Level 1

## 3. Khái niệm

**Constructor** chạy tự động khi `new Type(...)`. Nhiệm vụ: đưa object về trạng thái hợp lệ (không “nửa đời”).

Đời thường: mua điện thoại mới → cửa hàng gắn SIM, cài ngôn ngữ, bật bảo mật — đó là “constructor” của máy.

Game: spawn `Enemy(100, 2.5f)` — máu và tốc độ đã set; không để enemy máu 0 rồi mới nhớ gán.

**Default constructor:** nếu bạn không viết constructor nào, compiler tạo constructor không tham số rỗng (với class thường). Khi bạn viết constructor có tham số, default biến mất trừ khi bạn tự thêm.

**Finalizer (`~MyClass()`):** được GC gọi (không chắc khi nào) trước khi thu hồi object có finalizer. Dùng cho dọn tài nguyên không quản lý (hiếm). Với file/socket dùng `IDisposable` + `using` (Level sau). **Không** dùng finalizer để “hủy enemy khi chết” trong game loop.

## 4. Mô hình tư duy

```text
new Player("Aria", 100)
        │
        ▼
┌───────────────────────────┐
│ 1. Cấp phát bộ nhớ        │
│ 2. Gán field mặc định     │
│ 3. Chạy constructor body  │
│ 4. Trả reference cho bạn  │
└───────────────────────────┘

Finalizer (hiếm):
  object chết (không còn ai giữ) → GC → (~Finalizer?) → thu hồi
  Thời điểm: KHÔNG đoán được → không dùng cho gameplay
```

## 5. Cú pháp

Giải thích: nhiều constructor; chaining bằng `this(...)` để tránh lặp code khởi tạo.

```csharp
public class Player
{
    public string Name { get; }
    public int MaxHp { get; }
    public int Hp { get; private set; }

    public Player(string name) : this(name, 100) { }

    public Player(string name, int maxHp)
    {
        Name = string.IsNullOrWhiteSpace(name) ? "Hero" : name.Trim();
        MaxHp = maxHp < 1 ? 1 : maxHp;
        Hp = MaxHp;
    }

    // Finalizer — chỉ minh họa; hầu như không viết trong bài tập thường
    // ~Player() { /* cleanup unmanaged — tránh dùng cho logic game */ }
}

var a = new Player("Aria");
var b = new Player("Tank", 200);
```

Object initializer (sau khi constructor chạy):

```csharp
var p = new Player("Mage") { /* chỉ set được property có setter */ };
```

## 6. Ví dụ

### Cơ bản

Giải thích: bắt buộc truyền `owner` khi tạo tài khoản; số dư khởi tạo ≥ 0.

```csharp
public class Account
{
    public string Owner { get; }
    public decimal Balance { get; private set; }

    public Account(string owner, decimal opening = 0)
    {
        if (string.IsNullOrWhiteSpace(owner))
            throw new ArgumentException("Owner required", nameof(owner));
        Owner = owner.Trim();
        Balance = opening < 0 ? 0 : opening;
    }
}

var acc = new Account("Lan", 1000);
Console.WriteLine($"{acc.Owner}: {acc.Balance}");
```

### Trung cấp

Giải thích: overload constructor + chaining; một nơi validate chung.

```csharp
public class Bullet
{
    public float Speed { get; }
    public int Damage { get; }
    public string Tag { get; }

    public Bullet(float speed, int damage) : this(speed, damage, "player") { }

    public Bullet(float speed, int damage, string tag)
    {
        Speed = speed < 0 ? 0 : speed;
        Damage = Math.Max(0, damage);
        Tag = tag;
    }
}

var b1 = new Bullet(12f, 5);
var b2 = new Bullet(8f, 20, "enemy");
```

### Nâng cao

Giải thích: static factory method tạo object với tên rõ nghĩa hơn overload trần.

```csharp
public class ColorRgb
{
    public byte R { get; }
    public byte G { get; }
    public byte B { get; }

    private ColorRgb(byte r, byte g, byte b) { R = r; G = g; B = b; }

    public static ColorRgb FromRgb(byte r, byte g, byte b) => new(r, g, b);
    public static ColorRgb Red() => new(255, 0, 0);
    public static ColorRgb Gray(byte v) => new(v, v, v);
}

var c = ColorRgb.Red();
Console.WriteLine($"{c.R},{c.G},{c.B}");
```

## 7. Lỗi thường gặp

| Lỗi | Nguyên nhân | Cách xử lý |
|-----|-------------|------------|
| Không còn default ctor | Đã viết ctor có tham số | Thêm `public T() { }` nếu cần |
| Field chưa gán | Logic phụ thuộc field trong ctor sai thứ tự | Gán hết dependency trước khi dùng |
| Ném exception mơ hồ trong ctor | Input xấu | Validate rõ; `ArgumentException` |
| Dùng finalizer “hủy object game” | Hiểu nhầm destructor C++ | Dùng method `Die()` / pool / `Destroy` Unity |

## 8. Gỡ lỗi

1. Đặt breakpoint dòng đầu constructor — xem tham số vào.
2. Nếu object “sai mặc định”: kiểm tra bạn gọi đúng overload.
3. Tránh gọi method `virtual` trong constructor (polymorphism Chương 6 — nguy hiểm).
4. Không dựa vào thứ tự finalizer giữa các object.

## 9. Best practices

- Object phải **hợp lệ** ngay sau constructor.
- Ít tham số; nhóm thành options object nếu quá nhiều.
- Chaining `this(...)` để một nơi validate.
- Prefer `IDisposable` hơn finalizer cho tài nguyên.
- Trong Unity: khởi tạo gameplay thường ở `Awake`/`Start` hoặc method `Init` — cẩn thận vì `new MonoBehaviour` không hợp lệ như class thường.

## 10. Bài tập

**Bài 1 — Product**  
Constructor `(string name, decimal price)`; `price < 0` → 0. In sản phẩm.

**Bài 2 — Timer**  
Ctor `(int seconds)`; overload không tham số = 60. Method `Tick()` giảm 1 nếu > 0.

**Bài 3 — Character chaining**  
`Character(name)` gọi `Character(name, level: 1)`. In name + level.

**Bài 4 — Validate Email nhẹ**  
Ctor `User(email)`: nếu không chứa `@` thì gán `"invalid@local"`. In email.

**Bài 5 — Factory**  
Class `Dice` private ctor; `Dice.D6()`, `Dice.D20()`; method `Roll()` dùng `Random`.

## 11. Gợi ý

- Bài 1–2: một file, top-level gọi ctor.
- Bài 5: `private Dice(int sides)` + static factory.

## 12. Đáp án + Giải thích

### Bài 1

Giải thích: chuẩn hóa giá trong constructor.

```csharp
public class Product
{
    public string Name { get; }
    public decimal Price { get; }

    public Product(string name, decimal price)
    {
        Name = name?.Trim() ?? "";
        Price = price < 0 ? 0 : price;
    }
}

var p = new Product("USB", -5);
Console.WriteLine($"{p.Name}: {p.Price}"); // 0
```

### Bài 2

Giải thích: overload mặc định 60 giây; `Tick` không cho âm.

```csharp
public class Timer
{
    public int Remaining { get; private set; }
    public Timer() : this(60) { }
    public Timer(int seconds) => Remaining = Math.Max(0, seconds);
    public void Tick() { if (Remaining > 0) Remaining--; }
}

var t = new Timer(3);
t.Tick(); t.Tick();
Console.WriteLine(t.Remaining); // 1
```

### Bài 3

Giải thích: chaining đảm bảo một đường khởi tạo.

```csharp
public class Character
{
    public string Name { get; }
    public int Level { get; }

    public Character(string name) : this(name, 1) { }
    public Character(string name, int level)
    {
        Name = string.IsNullOrWhiteSpace(name) ? "NPC" : name;
        Level = level < 1 ? 1 : level;
    }
}

Console.WriteLine($"{new Character("Aria").Name} Lv{new Character("Aria").Level}");
```

### Bài 4

Giải thích: validate tối thiểu chuỗi email trong ctor.

```csharp
public class User
{
    public string Email { get; }
    public User(string email)
    {
        Email = !string.IsNullOrWhiteSpace(email) && email.Contains('@')
            ? email.Trim()
            : "invalid@local";
    }
}

Console.WriteLine(new User("a@b.com").Email);
Console.WriteLine(new User("bad").Email);
```

### Bài 5

Giải thích: factory đặt tên rõ; `Roll` trả 1..Sides.

```csharp
public class Dice
{
    public int Sides { get; }
    private static readonly Random Rng = new();
    private Dice(int sides) => Sides = sides < 2 ? 2 : sides;

    public static Dice D6() => new(6);
    public static Dice D20() => new(20);
    public int Roll() => Rng.Next(1, Sides + 1);
}

var d = Dice.D20();
Console.WriteLine(d.Roll());
```

## 13. Đáp án thay thế

- Primary constructor C# 12: `public class Product(string name, decimal price)` — gọn, học thêm khi quen class cổ điển.
- `required` property + object initializer thay một phần ctor (C# 11+).

## 14. Thử thách

Class `DateRange(start, end)`: nếu `end < start` thì đổi chỗ. Property `Days` = số ngày (dùng `TimeSpan`).

## 15. Ứng dụng thực tế

- Entity luôn có Id/ngày tạo hợp lệ từ lúc insert.
- Config object: fail-fast nếu thiếu connection string.
- Domain-driven: factory `Order.Create(...)` thay `new` lộ liễu.

## 16. Liên hệ Unity

- **Không** `new MonoBehaviour()` — dùng `AddComponent` / prefab instantiate.
- Constructor của C# trên MB bị hạn chế; init phổ biến: `Awake`, `Start`, `OnEnable`, hoặc `Init(data)` sau spawn.
- Finalizer **không** thay `OnDestroy`. Dọn event subscription trong `OnDestroy` để tránh leak.
- Object pooling: “tái sử dụng” thay tạo/hủy liên tục — khác hẳn finalizer.

## 17. Kiểm tra kiến thức

1. Constructor chạy khi nào?  
2. Viết ctor có tham số thì default ctor còn không?  
3. `this(...)` trong ctor để làm gì?  
4. Có nên dùng finalizer để trừ máu enemy không?  
5. Object hợp lệ ngay sau ctor nghĩa là gì?

**Đáp án:**  
1) Khi `new` tạo instance.  
2) Không — trừ khi bạn tự khai báo thêm.  
3) Gọi overload khác, tránh lặp khởi tạo.  
4) Không — thời điểm GC không đoán được; dùng logic game tường minh.  
5) Mọi invariant quan trọng (tên, máu > 0, …) đã đúng, sẵn sàng dùng.
