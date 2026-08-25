# Chương 5 — Inheritance: `base`, `this`

## 1. Mục tiêu học

- Hiểu quan hệ **is-a** và cú pháp `: BaseType`
- Dùng `base` gọi constructor / member lớp cha; `this` cho instance hiện tại
- Biết kế thừa thành viên và giới hạn (single inheritance class)
- Nhận diện khi inheritance **sai** — chuẩn bị cho composition (Chương 8)

## 2. Điều kiện tiên quyết

- Chương 1–4: class, ctor, encapsulation, static
- Hiểu `protected` (Chương 3)

## 3. Khái niệm

**Inheritance** cho class con kế thừa thành viên của class cha và mở rộng/chỉnh hành vi.

Đời thường: “Xe tải **là một** Xe” — có bánh, động cơ (chung), thêm thùng hàng (riêng).

Game: `Dragon : Enemy` — máu/tốc độ chung, thêm `BreatheFire()`.

**Không phải mọi thứ “có máu” đều nên `: Entity`.** Nếu chỉ muốn dùng lại code, thường **composition** tốt hơn (“có” `Health` component).

C# class: **một** base class. Muốn nhiều khả năng → interface (Chương 7) hoặc gắn nhiều thành phần.

`this` = object hiện tại. `base` = thành viên phía lớp cha (hữu ích khi bị che tên hoặc gọi ctor cha).

## 4. Mô hình tư duy

```text
        Animal
       /      \
    Dog        Cat
     │
  Puppy

new Puppy()
  → Animal ctor
  → Dog ctor
  → Puppy ctor

is-a: Puppy is Dog is Animal  ✓
has-a: Player has Inventory   → không cần Player : Inventory
```

## 5. Cú pháp

Giải thích: class con gọi `base(...)` để khởi tạo phần cha trước.

```csharp
public class Enemy
{
    public int Hp { get; protected set; }
    public Enemy(int hp) => Hp = hp;
    public void TakeDamage(int dmg) => Hp = Math.Max(0, Hp - dmg);
}

public class Dragon : Enemy
{
    public int FirePower { get; }
    public Dragon(int hp, int firePower) : base(hp)
    {
        FirePower = firePower;
    }

    public void BreatheFire(Enemy target)
        => target.TakeDamage(FirePower);
}

var d = new Dragon(200, 40);
var goblin = new Enemy(30);
d.BreatheFire(goblin);
```

## 6. Ví dụ

### Cơ bản

Giải thích: `SavingsAccount` kế thừa `Account`, thêm lãi suất.

```csharp
public class Account
{
    public string Owner { get; }
    public decimal Balance { get; protected set; }
    public Account(string owner, decimal balance)
    {
        Owner = owner;
        Balance = balance;
    }
}

public class SavingsAccount : Account
{
    public decimal Rate { get; }
    public SavingsAccount(string owner, decimal balance, decimal rate)
        : base(owner, balance) => Rate = rate;

    public void ApplyInterest() => Balance += Balance * Rate;
}
```

### Trung cấp

Giải thích: method ở con gọi `base` rồi bổ sung hành vi.

```csharp
public class Logger
{
    public virtual void Log(string msg) => Console.WriteLine(msg);
}

public class TimestampLogger : Logger
{
    public override void Log(string msg)
        => base.Log($"{DateTime.Now:HH:mm:ss} {msg}");
}
```

*(Chi tiết `virtual`/`override` ở Chương 6 — ở đây chỉ thấy `base.Log`.)*

### Nâng cao

Giải thích: `this` phân biệt tham số trùng tên field; chaining inheritance ctor.

```csharp
public class Unit
{
    private readonly string name;
    public Unit(string name) => this.name = name;
    public string Name => name;
}

public class Hero : Unit
{
    public int Level { get; }
    public Hero(string name, int level) : base(name) => Level = level;
}
```

## 7. Lỗi thường gặp

| Lỗi | Nguyên nhân | Cách xử lý |
|-----|-------------|------------|
| Không gọi base ctor phù hợp | Cha không có default ctor | `: base(...)` đúng tham số |
| Cây thừa kế quá sâu | Mọi thứ nhét is-a | Composition / interface |
| Lạm dụng `protected` | Con phá invariant cha | Giữ private; mở API có kiểm soát |
| Nhầm has-a thành is-a | `Player : Gun` | `Player` có `Gun` |

## 8. Gỡ lỗi

1. Đặt breakpoint lần lượt ctor cha → con.
2. Nếu thiếu thành viên: kiểm tra `private` ở cha (con không thấy).
3. Vẽ sơ đồ is-a trước khi code — nếu câu “X là một Y” gượng → dừng.
4. Đọc warning về `new` keyword che member (khác `override`).

## 9. Best practices

- Inheritance cho **phân cấp ổn định, thật sự is-a**.
- Giữ sâu cây ≤ 2–3 tầng khi mới học.
- Gọi `base` ctor rõ ràng.
- Đừng kế thừa chỉ để lấy 2–3 method tiện — tách helper/composition.
- Ưu tiên `protected` hẹp; test qua public API.

## 10. Bài tập

**Bài 1 — Vehicle / Car**  
`Vehicle` có `Speed`; `Car` thêm `Doors`. In thông tin.

**Bài 2 — Employee / Manager**  
`Employee(name, salary)`; `Manager` thêm `Bonus`; method `TotalPay()`.

**Bài 3 — Shape base**  
`Shape` có `Name`; `Circle` / `Rectangle` : `Shape`. In name.

**Bài 4 — base ToString**  
Override `ToString` ở con, gọi `base.ToString()` rồi thêm field (xem Chương 6 nếu cần `override`).

**Bài 5 — Phân tích sai**  
Viết 3 ví dụ has-a mà **không** nên inheritance (chỉ comment/markdown trong code).

## 11. Gợi ý

- Luôn có `: base(...)` khi cha bắt buộc tham số.
- Bài 5: Player-Weapon, Car-Engine, Order-EmailService.

## 12. Đáp án + Giải thích

### Bài 1

Giải thích: Car mở rộng Vehicle.

```csharp
public class Vehicle
{
    public int Speed { get; }
    public Vehicle(int speed) => Speed = speed;
}

public class Car : Vehicle
{
    public int Doors { get; }
    public Car(int speed, int doors) : base(speed) => Doors = doors;
}

var c = new Car(120, 4);
Console.WriteLine($"Speed={c.Speed}, Doors={c.Doors}");
```

### Bài 2

Giải thích: Manager cộng bonus vào lương.

```csharp
public class Employee
{
    public string Name { get; }
    public decimal Salary { get; protected set; }
    public Employee(string name, decimal salary)
    {
        Name = name;
        Salary = salary;
    }
    public virtual decimal TotalPay() => Salary;
}

public class Manager : Employee
{
    public decimal Bonus { get; }
    public Manager(string name, decimal salary, decimal bonus)
        : base(name, salary) => Bonus = bonus;
    public override decimal TotalPay() => Salary + Bonus;
}

Console.WriteLine(new Manager("Lan", 1000, 200).TotalPay());
```

### Bài 3

Giải thích: cùng base `Shape` với tên.

```csharp
public class Shape
{
    public string Name { get; }
    public Shape(string name) => Name = name;
}
public class Circle : Shape
{
    public double R { get; }
    public Circle(double r) : base("Circle") => R = r;
}
public class Rectangle : Shape
{
    public double W { get; }
    public double H { get; }
    public Rectangle(double w, double h) : base("Rectangle") { W = w; H = h; }
}

Console.WriteLine(new Circle(2).Name);
Console.WriteLine(new Rectangle(3, 4).Name);
```

### Bài 4

Giải thích: bổ sung thông tin trên `ToString` của object.

```csharp
public class Item
{
    public string Id { get; }
    public Item(string id) => Id = id;
    public override string ToString() => $"Item({Id})";
}

public class Weapon : Item
{
    public int Dmg { get; }
    public Weapon(string id, int dmg) : base(id) => Dmg = dmg;
    public override string ToString() => $"{base.ToString()} dmg={Dmg}";
}

Console.WriteLine(new Weapon("sword", 10));
```

### Bài 5

Giải thích: liệt kê quan hệ has-a — không dùng `:`.

```csharp
// SAI nếu viết: class Player : Weapon  → Player HAS a Weapon
// SAI nếu viết: class Car : Engine     → Car HAS an Engine
// SAI nếu viết: class Order : Mailer   → Order USES a Mailer
// Đúng hướng: field/property hoặc inject dependency
```

## 13. Đáp án thay thế

- Dùng interface `IPayable` thay hierarchy Employee nếu không chia sẻ implementation.
- Sealed class ngăn kế thừa tiếp (Chương 6).

## 14. Thử thách

Thiết kế `Npc` → `Character` → `PlayableCharacter` tối đa 3 tầng; giải thích vì sao không thêm tầng 4.

## 15. Ứng dụng thực tế

- UI control hierarchy (cẩn thận sâu).
- Exception types: `IOException : Exception`.
- Domain: `SavingsAccount : Account` khi rule nghiệp vụ thật sự chuyên biệt.

## 16. Liên hệ Unity

- Mọi script hay `: MonoBehaviour` — đây là inheritance bắt buộc của engine, **không** có nghĩa bạn nên `: PlayerBase : EntityBase : ...` 8 tầng.
- Prefer nhiều component: `Health`, `Mover`, `Attacker` gắn cùng GameObject (**composition**).
- Prefab variant ≠ C# inheritance — đừng nhầm.
- Pitfall: override `Update` sâu cây MB → khó đoán thứ tự/gọi base.

## 17. Kiểm tra kiến thức

1. C# class kế thừa được bao nhiêu class?  
2. `base` dùng để làm gì?  
3. `this` khác `base`?  
4. is-a vs has-a ví dụ?  
5. Vì sao cây sâu nguy hiểm?

**Đáp án:**  
1) Một base class.  
2) Gọi ctor/member lớp cha.  
3) `this` = hiện tại; `base` = phía cha.  
4) Dog is Animal; Car has Engine.  
5) Phá vỡ, khó đổi, siết chặt phụ thuộc, lạm dụng protected.
