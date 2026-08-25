# Chương 6 — Polymorphism: `virtual`, `override`, `abstract`, `sealed`

## 1. Mục tiêu học

- Hiểu **polymorphism**: cùng lời gọi, hành vi khác nhau tùy kiểu thật
- Dùng `virtual` / `override` / `new` (biết khác nhau)
- Thiết kế `abstract` class và method
- Dùng `sealed` để chặn override / kế thừa tiếp

## 2. Điều kiện tiên quyết

- Chương 5: inheritance, `base`, `this`
- Encapsulation và constructor

## 3. Khái niệm

**Polymorphism (đa hình):** thao tác qua kiểu cha / contract, runtime chạy đúng implementation của object thật.

Đời thường: nút “Thanh toán” — thẻ tín dụng, ví điện tử, tiền mặt: cùng lệnh “pay”, khác quy trình.

Game: `Enemy[] wave` — mỗi phần tử `Attack()` khác nhau (goblin đâm, skeleton bắn, slime nhảy).

| Từ khóa | Ý nghĩa |
|---------|---------|
| `virtual` | Cho phép con `override` |
| `override` | Thay thế implementation ảo của cha |
| `abstract` | Chưa có body; con **bắt buộc** implement (class cũng abstract) |
| `sealed` | Chặn class bị kế thừa, hoặc chặn `override` tiếp |
| `new` | Che member (không đa hình theo virtual slot) — thường tránh |

## 4. Mô hình tư duy

```text
Enemy e = new Dragon(...);
e.Attack();   // gọi Dragon.Attack nếu override đúng

Bảng ảo (ý tưởng):
  Attack → &Dragon.Attack

abstract Shape
  + abstract double Area()
     ↑
  Circle.Area / Rectangle.Area

sealed class MathConstants { }  // không ai : MathConstants
```

## 5. Cú pháp

Giải thích: gọi qua kiểu cha vẫn chạy override của con.

```csharp
public abstract class Enemy
{
    public int Hp { get; protected set; }
    protected Enemy(int hp) => Hp = hp;

    public abstract void Attack(Enemy target);
    public virtual void TakeDamage(int dmg)
        => Hp = Math.Max(0, Hp - dmg);
}

public class Goblin : Enemy
{
    public Goblin() : base(30) { }
    public override void Attack(Enemy target) => target.TakeDamage(5);
}

public class Dragon : Enemy
{
    public Dragon() : base(200) { }
    public override void Attack(Enemy target) => target.TakeDamage(40);
    public sealed override void TakeDamage(int dmg)
        => base.TakeDamage(dmg / 2); // giáp; sealed: subclass Dragon không override tiếp
}

Enemy a = new Goblin();
Enemy b = new Dragon();
a.Attack(b);
```

## 6. Ví dụ

### Cơ bản

Giải thích: `Speak` khác nhau ở Dog/Cat.

```csharp
public class Animal
{
    public virtual string Speak() => "...";
}
public class Dog : Animal
{
    public override string Speak() => "Gâu";
}
public class Cat : Animal
{
    public override string Speak() => "Meo";
}

Animal[] arr = { new Dog(), new Cat() };
foreach (var x in arr) Console.WriteLine(x.Speak());
```

### Trung cấp

Giải thích: abstract `Shape.Area()`; tính tổng diện tích danh sách.

```csharp
public abstract class Shape
{
    public abstract double Area();
}
public class Circle : Shape
{
    public double R { get; }
    public Circle(double r) => R = r;
    public override double Area() => Math.PI * R * R;
}
public class Rect : Shape
{
    public double W { get; }
    public double H { get; }
    public Rect(double w, double h) { W = w; H = h; }
    public override double Area() => W * H;
}

Shape[] shapes = { new Circle(2), new Rect(3, 4) };
Console.WriteLine(shapes.Sum(s => s.Area()));
```

### Nâng cao

Giải thích: template method — cha định nghĩa khung, con điền bước abstract/`virtual`.

```csharp
public abstract class DataExporter
{
    public void Export()
    {
        var data = Load();
        var text = Format(data);
        Save(text);
    }

    protected abstract string Load();
    protected virtual string Format(string data) => data.Trim();
    protected abstract void Save(string text);
}

public class ConsoleExporter : DataExporter
{
    protected override string Load() => "  hello  ";
    protected override void Save(string text) => Console.WriteLine(text);
}

new ConsoleExporter().Export();
```

## 7. Lỗi thường gặp

| Lỗi | Nguyên nhân | Cách xử lý |
|-----|-------------|------------|
| Quên `override` → dùng `new` | Che member, mất đa hình | Thêm `virtual` ở cha + `override` |
| `new AbstractClass()` | Abstract không tạo trực tiếp | Tạo class con cụ thể |
| Abstract thiếu implement | Con chưa override đủ | Implement mọi abstract member |
| Gọi virtual trong ctor | Object con chưa init xong | Tránh; dùng init method |

## 8. Gỡ lỗi

1. In `GetType().Name` trước khi gọi method ảo.
2. Nếu hành vi “của cha” chạy: kiểm tra có `override` thật không.
3. Breakpoint trong từng override.
4. Tránh `new` keyword trừ khi cố ý che và hiểu hậu quả.

## 9. Best practices

- API đa hình qua abstract/`virtual` có chủ đích — không virtual hóa mọi thứ.
- Abstract class = khung + code dùng chung; interface = chỉ contract (Chương 7–8).
- `sealed` class/method khi không muốn mở rộng thêm (bảo mật hành vi, tối ưu).
- Liskov: con không được phá kỳ vọng của cha (ví dụ `TakeDamage` không được hồi máu trừ khi contract cho phép).

## 10. Bài tập

**Bài 1 — Notify**  
`Notification` virtual `Send`; `EmailNotification`, `SmsNotification` override — gọi qua mảng base.

**Bài 2 — Account lãi**  
Abstract `Account` với `ApplyMonthly()`; `Savings` cộng %; `Checking` trừ phí cố định.

**Bài 3 — sealed**  
`sealed class Uuid` không cho kế thừa — minh họa lỗi nếu cố `: Uuid`.

**Bài 4 — Weapon**  
Abstract `Weapon.Use(target)`; `Sword`, `Bow`.

**Bài 5 — Template**  
Abstract `MealRecipe` với `Cook()` gọi `Prepare`/`Main`/`Serve` (abstract/virtual).

## 11. Gợi ý

- Luôn giữ reference kiểu cha khi demo đa hình.
- Bài 3: comment dòng lỗi biên dịch.

## 12. Đáp án + Giải thích

### Bài 1

Giải thích: cùng `Send()`, khác kênh.

```csharp
public class Notification
{
    public virtual void Send(string msg) => Console.WriteLine(msg);
}
public class EmailNotification : Notification
{
    public override void Send(string msg) => Console.WriteLine($"EMAIL: {msg}");
}
public class SmsNotification : Notification
{
    public override void Send(string msg) => Console.WriteLine($"SMS: {msg}");
}

Notification[] n = { new EmailNotification(), new SmsNotification() };
foreach (var x in n) x.Send("Hi");
```

### Bài 2

Giải thích: áp dụng cuối tháng khác nhau.

```csharp
public abstract class Account
{
    public decimal Balance { get; protected set; }
    protected Account(decimal b) => Balance = b;
    public abstract void ApplyMonthly();
}
public class Savings : Account
{
    public Savings(decimal b) : base(b) { }
    public override void ApplyMonthly() => Balance *= 1.01m;
}
public class Checking : Account
{
    public Checking(decimal b) : base(b) { }
    public override void ApplyMonthly() => Balance -= 5m;
}

Account[] a = { new Savings(100), new Checking(100) };
foreach (var x in a) { x.ApplyMonthly(); Console.WriteLine(x.Balance); }
```

### Bài 3

Giải thích: `sealed` chặn kế thừa.

```csharp
public sealed class Uuid
{
    public string Value { get; }
    public Uuid(string v) => Value = v;
}
// public class Bad : Uuid { } // lỗi biên dịch
Console.WriteLine(new Uuid("abc").Value);
```

### Bài 4

Giải thích: vũ khí abstract.

```csharp
public abstract class Weapon
{
    public abstract void Use(string target);
}
public class Sword : Weapon
{
    public override void Use(string target) => Console.WriteLine($"Chém {target}");
}
public class Bow : Weapon
{
    public override void Use(string target) => Console.WriteLine($"Bắn {target}");
}

Weapon w = new Bow();
w.Use("slime");
```

### Bài 5

Giải thích: khung `Cook` cố định.

```csharp
public abstract class MealRecipe
{
    public void Cook()
    {
        Prepare();
        Main();
        Serve();
    }
    protected abstract void Prepare();
    protected abstract void Main();
    protected virtual void Serve() => Console.WriteLine("Thưởng thức!");
}
public class Pho : MealRecipe
{
    protected override void Prepare() => Console.WriteLine("Ninh xương");
    protected override void Main() => Console.WriteLine("Trần bánh + thịt");
}
new Pho().Cook();
```

## 13. Đáp án thay thế

- Strategy pattern bằng delegate/interface thay hierarchy sâu.
- `record` + interface cho dữ liệu + hành vi tách.

## 14. Thử thách

Hệ `Ability` abstract với `Cooldown`; `Fireball`/`Heal` override `Activate(Character)`. Quản lý list ability và tick cooldown.

## 15. Ứng dụng thực tế

- Plugin payment / shipping providers.
- Renderers, serializers, exporters.
- Rule engine: mỗi rule một subclass hoặc strategy.

## 16. Liên hệ Unity

- `Update`/`FixedUpdate` trên MB là đa hình theo engine — đừng tạo chuỗi override `Update` sâu.
- Prefer nhiều component nhỏ thay abstract `Entity` khổng lồ.
- `ScriptableObject` + strategy thường sạch hơn inheritance enemy 10 tầng.
- `sealed` trên class utility tránh ai đó `:` nhầm trong project lớn.

## 17. Kiểm tra kiến thức

1. Polymorphism giúp gì?  
2. `abstract` method khác `virtual`?  
3. `override` vs `new`?  
4. Tạo được instance abstract class không?  
5. `sealed override` nghĩa là gì?

**Đáp án:**  
1) Cùng API, khác hành vi theo kiểu thật.  
2) Abstract bắt buộc override, không body; virtual có body mặc định.  
3) `override` đa hình đúng; `new` che, dễ gọi nhầm bản cha.  
4) Không.  
5) Chặn lớp cháu override tiếp method đó.
