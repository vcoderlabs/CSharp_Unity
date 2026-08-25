# Chương 4 — `static`, `readonly`, `const`

## 1. Mục tiêu học

- Phân biệt thành viên **instance** và **static** (thuộc type)
- Dùng `static` class / method / field đúng chỗ (utility, đếm instance…)
- Phân biệt **`const`** (compile-time) và **`readonly`** (runtime, gán trong ctor)
- Tránh lạm dụng static (global state — khó test, khó Unity)

## 2. Điều kiện tiên quyết

- Chương 1–3: class, constructor, encapsulation
- Biết `const` cơ bản từ Level 1

## 3. Khái niệm

**Instance member:** mỗi object một bản (`player.Hp`).  
**Static member:** một bản gắn với **type** (`Math.PI`, `Enemy.CountAlive`).

Đời thường: mỗi xe có nhiên liệu riêng (instance); “quy định tốc độ tối đa toàn quốc” là rule dùng chung (static/const).

Game: `GameConfig.MaxPlayers` (static/const); máu từng hero là instance.

| | `const` | `readonly` | field thường |
|--|---------|------------|--------------|
| Khi gán | Compile-time | Khai báo hoặc ctor | Mọi lúc (nếu không private rule) |
| Thuộc | Luôn static về bản chất giá trị nhúng | Instance hoặc static | Instance/static |
| Ví dụ | `const int Max = 3;` | `readonly DateTime Created;` | `_score` |

`static readonly` thường dùng cho giá trị cố định nhưng không phải literal compile-time (`static readonly Random Rng = new();`).

## 4. Mô hình tư duy

```text
Type Enemy
  static int SpawnedCount ──────────► [ 1 ô nhớ dùng chung ]
  static readonly Random Rng ───────► [ 1 RNG dùng chung ]

Instance e1: Hp, Speed ──► ô riêng
Instance e2: Hp, Speed ──► ô riêng

const MaxLives = 3  →  compiler có thể nhúng số 3 vào chỗ gọi
```

## 5. Cú pháp

Giải thích: đếm số object đã tạo; hằng và readonly khởi tạo khác nhau.

```csharp
public class Enemy
{
    public const int MaxLevel = 50;                 // compile-time
    public static readonly DateTime BootTime = DateTime.UtcNow; // 1 lần khi type load
    public static int Spawned { get; private set; }

    public readonly string Id; // gán trong ctor
    public int Hp { get; private set; }

    public Enemy(string id, int hp)
    {
        Id = id;
        Hp = hp;
        Spawned++;
    }
}

public static class MathUtil
{
    public static int Clamp(int v, int min, int max)
        => v < min ? min : (v > max ? max : v);
}
```

## 6. Ví dụ

### Cơ bản

Giải thích: utility static không cần `new`.

```csharp
public static class TempConvert
{
    public static double CtoF(double c) => c * 9 / 5 + 32;
}

Console.WriteLine(TempConvert.CtoF(100));
```

### Trung cấp

Giải thích: `const` cho ID cấu hình; `readonly` cho thời điểm tạo account.

```csharp
public class Account
{
    public const string BankCode = "ACME";
    public readonly DateTime OpenedAt;
    public string Owner { get; }

    public Account(string owner)
    {
        Owner = owner;
        OpenedAt = DateTime.Now;
    }
}
```

### Nâng cao

Giải thích: static constructor chạy một lần khi type được dùng lần đầu.

```csharp
public class AppSettings
{
    public static string Env { get; private set; } = "dev";

    static AppSettings()
    {
        // giả lập đọc môi trường
        Env = Environment.GetEnvironmentVariable("APP_ENV") ?? "dev";
    }
}

Console.WriteLine(AppSettings.Env);
```

## 7. Lỗi thường gặp

| Lỗi | Nguyên nhân | Cách xử lý |
|-----|-------------|------------|
| Gọi instance từ static không có object | Không có `this` | Truyền instance vào hoặc bỏ static |
| `const` object phức tạp | Chỉ literal/cho phép | Dùng `static readonly` |
| Global mutable static | Race, khó test | Giảm static state; inject dependency |
| Nhầm readonly không đổi được phần tử collection | Reference readonly, nội dung list vẫn sửa | Dùng immutable / expose read-only |

## 8. Gỡ lỗi

1. Lỗi “object reference required”: bạn đang gọi instance member như static hoặc ngược lại.
2. Static field giữ reference object → object không bị GC — kiểm tra leak.
3. Unit test: static mutable làm test ảnh hưởng nhau — reset cẩn thận hoặc tránh.
4. Log khi static ctor chạy để hiểu thứ tự khởi tạo type.

## 9. Best practices

- `static` cho hàm thuần (pure) / utility / extension-like helper.
- Tránh static mutable cho gameplay state lớn.
- `const` cho số/magic rõ ràng; `static readonly` cho object cố định.
- Instance `readonly` cho Id không đổi sau ctor.
- Unity: cẩn thận Singleton static — biết trade-off.

## 10. Bài tập

**Bài 1 — Circle**  
`const double Pi`; method instance `Area()` dùng `Pi` và `Radius`.

**Bài 2 — IdGenerator**  
`static int NextId()` tăng dần từ 1.

**Bài 3 — Config**  
`static readonly` string `Version`; in ra.

**Bài 4 — Counter instances**  
Mỗi `new Player` tăng `static Count`; in Count sau 3 lần `new`.

**Bài 5 — MathUtil**  
Static class: `Max`, `Min`, `Abs` cho `int`; đọc 2 số, in kết quả.

## 11. Gợi ý

- Bài 2–4: static field private + property get.
- Bài 5: không cho tạo instance (`static class`).

## 12. Đáp án + Giải thích

### Bài 1

Giải thích: Pi là const; diện tích theo radius instance.

```csharp
public class Circle
{
    public const double Pi = 3.141592653589793;
    public double Radius { get; }
    public Circle(double r) => Radius = r < 0 ? 0 : r;
    public double Area() => Pi * Radius * Radius;
}

Console.WriteLine(new Circle(2).Area());
```

### Bài 2

Giải thích: một bộ đếm dùng chung cho toàn type.

```csharp
public static class IdGenerator
{
    private static int _n;
    public static int NextId() => ++_n;
}

Console.WriteLine(IdGenerator.NextId());
Console.WriteLine(IdGenerator.NextId());
```

### Bài 3

Giải thích: version gán một lần khi load type.

```csharp
public static class Config
{
    public static readonly string Version = "1.0.0";
}

Console.WriteLine(Config.Version);
```

### Bài 4

Giải thích: constructor tăng `Count`.

```csharp
public class Player
{
    public static int Count { get; private set; }
    public string Name { get; }
    public Player(string name) { Name = name; Count++; }
}

_ = new Player("A");
_ = new Player("B");
_ = new Player("C");
Console.WriteLine(Player.Count); // 3
```

### Bài 5

Giải thích: static helpers thuần.

```csharp
public static class MathUtil
{
    public static int Max(int a, int b) => a > b ? a : b;
    public static int Min(int a, int b) => a < b ? a : b;
    public static int Abs(int x) => x < 0 ? -x : x;
}

Console.Write("a b: ");
var p = (Console.ReadLine() ?? "").Split(' ', StringSplitOptions.RemoveEmptyEntries);
int.TryParse(p.ElementAtOrDefault(0), out int a);
int.TryParse(p.ElementAtOrDefault(1), out int b);
Console.WriteLine($"max={MathUtil.Max(a,b)} min={MathUtil.Min(a,b)}");
```

## 13. Đáp án thay thế

- `Random.Shared` (.NET 6+) thay `static readonly Random` tự quản lý.
- Dependency injection container thay static service locator (kiến trúc sau).

## 14. Thử thách

`static class Logger` với `Info`/`Error` ghi ra console kèm `DateTime.Now`; đếm số lần gọi qua `static int Calls`.

## 15. Ứng dụng thực tế

- `const` protocol version, limit.
- Cache/`static readonly` regex biên dịch sẵn.
- Helper parse/format không giữ state.

## 16. Liên hệ Unity

- `static` event / singleton dễ **domain reload** và leak listener.
- ScriptableObject / service gắn scene thường rõ vòng đời hơn static lung tung.
- `const` cho layer name/key — ổn; gameplay state trên static — hạn chế.
- `Time.deltaTime` là static API engine — khác static mutable của bạn.

## 17. Kiểm tra kiến thức

1. Static field chia sẻ giữa các instance?  
2. `const` khác `readonly`?  
3. Static method có `this` không?  
4. Khi nào dùng `static class`?  
5. Rủi ro của static mutable?

**Đáp án:**  
1) Có — một bản cho type.  
2) `const` compile-time literal; `readonly` gán runtime (ctor/static ctor).  
3) Không.  
4) Chỉ chứa thành viên static (utility).  
5) Global state, khó test, race, vòng đời khó kiểm soát.
