# Chương 8 — Class vs Interface vs Abstract; Composition over Inheritance

## 1. Mục tiêu học

- Chọn đúng công cụ: **class cụ thể**, **abstract class**, **interface**
- Hiểu nguyên tắc **composition over inheritance**
- Nhận diện anti-pattern inheritance trong game/Unity
- Thiết kế lại ví dụ “kế thừa tệ” thành composition

## 2. Điều kiện tiên quyết

- Chương 5–7: inheritance, polymorphism, interface
- Biết encapsulation

## 3. Khái niệm

Ba lớp công cụ:

| Công cụ | Dùng khi |
|---------|----------|
| **Concrete class** | Thực thể đủ dùng, ít biến thể |
| **Abstract class** | Cùng họ is-a, **chia sẻ code/state**, một phần bắt buộc override |
| **Interface** | Capability ngang hàng, nhiều nguồn, không cần base state chung |

**Composition:** object **có** thành phần (`Player` có `Health`, `Inventory`) thay vì **là** mọi thứ qua cây thừa kế.

Đời thường: máy tính *có* CPU, RAM, SSD — không phải “Laptop : CPU : RAM”.

Game/Unity: GameObject *có* nhiều component — đúng tinh thần composition. Cây `Enemy → FlyingEnemy → FastFlyingFireEnemy` là mùi thiết kế xấu khi tổ hợp feature bùng nổ.

## 4. Mô hình tư duy

```text
INHERITANCE (is-a)              COMPOSITION (has-a)
Player : Entity                 Player
  └─ quá nhiều tầng               ├─ Health
                                ├─ Mover
                                └─ Inventory

Feature combo:
Fly + Swim + Fire
Inheritance → class nổ tổ hợp
Composition → gắn behavior cần dùng

Quy tắc nhanh:
  "X là một Y?" rõ và ổn định → có thể inheritance
  "X có Y?" / "X dùng Y?" → composition
  "X có thể làm Y?" → interface
```

## 5. Cú pháp

Giải thích: cùng năng lực di chuyển — composition + interface thay subclass explosion.

```csharp
public interface IMover
{
    void Move(int dx, int dy);
}

public class WalkMover : IMover
{
    private readonly Actor _actor;
    public WalkMover(Actor actor) => _actor = actor;
    public void Move(int dx, int dy) { _actor.X += dx; _actor.Y += dy; }
}

public class Actor
{
    public int X { get; set; }
    public int Y { get; set; }
    public IMover Mover { get; set; }

    public Actor(IMover mover) => Mover = mover;
    public void Move(int dx, int dy) => Mover.Move(dx, dy);
}

// Gắn hành vi lúc tạo / runtime đổi Mover (fly vs walk)
var hero = new Actor(null!);
hero.Mover = new WalkMover(hero);
hero.Move(1, 0);
```

## 6. Ví dụ

### Cơ bản — chọn công cụ

Giải thích: logging capability → interface; không cần abstract.

```csharp
public interface ILogger { void Log(string msg); }
public class ConsoleLogger : ILogger
{
    public void Log(string msg) => Console.WriteLine(msg);
}
```

### Trung cấp — abstract khi có code chung

Giải thích: mọi account đều có balance + deposit; khác nhau ở phí rút.

```csharp
public abstract class Account
{
    public decimal Balance { get; protected set; }
    public void Deposit(decimal a) { if (a > 0) Balance += a; }
    public abstract bool Withdraw(decimal a);
}
```

### Nâng cao — refactor inheritance xấu

Giải thích: thay `FlyingFireEnemy : FireEnemy : Enemy` bằng components.

```csharp
public class Health
{
    public int Hp { get; private set; }
    public Health(int hp) => Hp = hp;
    public void Damage(int d) => Hp = Math.Max(0, Hp - d);
}

public class FireBreath
{
    public int Power { get; }
    public FireBreath(int power) => Power = power;
    public void Apply(Health target) => target.Damage(Power);
}

public class Enemy
{
    public Health Health { get; } = new(50);
    public FireBreath? Fire { get; set; } // null = không phun lửa
}

var dragon = new Enemy { Fire = new FireBreath(40) };
var slime = new Enemy(); // không Fire
dragon.Fire!.Apply(slime.Health);
```

## 7. Lỗi thường gặp

| Lỗi | Nguyên nhân | Cách xử lý |
|-----|-------------|------------|
| Interface + abstract trùng vô nghĩa | Sinh abstraction thừa | Chỉ giữ lớp cần thiết |
| Inheritance để reuse 5 dòng | Lười copy | Helper / composition |
| God base class | Nhét hết vào Entity | Tách component |
| “Mọi thứ là interface” | Over-engineering | Concrete đến khi cần abstraction |

## 8. Gỡ lỗi

1. Viết câu is-a/has-a trước khi mở IDE.
2. Đếm số subclass chỉ khác 1 flag → nên composition/strategy.
3. Nếu sửa base làm vỡ 10 con → coupling inheritance quá chặt.
4. Review Unity: script có >3 trách nhiệm? Tách component.

## 9. Best practices

- Bắt đầu concrete; extract interface khi có ≥2 impl hoặc cần test fake.
- Abstract class khi họ hàng gần và chia sẻ implementation thật.
- Composition mặc định cho tổ hợp hành vi.
- Ưu tiên sâu cây thấp; ngang bằng interface + components.
- Document quyết định (“vì sao không : X”).

## 10. Bài tập

**Bài 1 — Chọn công cụ**  
Với từng case, ghi `class` / `abstract` / `interface` + lý do ngắn: (a) Math.Max helper (b) nhiều cổng thanh toán (c) Account savings/checking chia sẻ balance.

**Bài 2 — Refactor**  
Code giả: `class Player : Weapon` — viết lại composition.

**Bài 3 — Bird problem**  
`Bird` có `Fly()`; thêm `Penguin` — thiết kế lại cho đúng Liskov (interface `IFlyable`).

**Bài 4 — GameObject mini**  
Class `Entity` chứa `List<object>` components; method `Add`/`Get<T>` đơn giản (cast).

**Bài 5 — Banking preview**  
Phác thảo (skeleton) `Account` abstract + `IInterestBearing` — chuẩn bị project cuối.

## 11. Gợi ý

- Bài 3: Penguin không implement `IFlyable`.
- Bài 4: dùng `list.OfType<T>().FirstOrDefault()`.

## 12. Đáp án + Giải thích

### Bài 1

Giải thích: trả lời ngắn theo tiêu chí chương.

```text
(a) static class / concrete utility — không cần interface
(b) interface IPayment — nhiều provider
(c) abstract class Account — state + hành vi chung, Withdraw khác
```

### Bài 2

Giải thích: Player có Weapon, không phải là Weapon.

```csharp
public class Weapon
{
    public int Damage { get; }
    public Weapon(int dmg) => Damage = dmg;
}

public class Player
{
    public Weapon? Weapon { get; set; }
    public void Attack(Health target)
    {
        if (Weapon is null) return;
        target.Damage(Weapon.Damage);
    }
}

public class Health
{
    public int Hp { get; private set; } = 100;
    public void Damage(int d) => Hp = Math.Max(0, Hp - d);
}
```

### Bài 3

Giải thích: bay là capability — không phải mọi Bird.

```csharp
public abstract class Bird
{
    public string Name { get; }
    protected Bird(string name) => Name = name;
}

public interface IFlyable { void Fly(); }

public class Eagle : Bird, IFlyable
{
    public Eagle() : base("Eagle") { }
    public void Fly() => Console.WriteLine("Bay cao");
}

public class Penguin : Bird
{
    public Penguin() : base("Penguin") { }
    public void Swim() => Console.WriteLine("Bơi");
}
```

### Bài 4

Giải thích: túi component tối giản.

```csharp
public class Entity
{
    private readonly List<object> _components = new();
    public void Add(object c) => _components.Add(c);
    public T? Get<T>() where T : class
        => _components.OfType<T>().FirstOrDefault();
}

var e = new Entity();
e.Add(new Health());
Console.WriteLine(e.Get<Health>()?.Hp);
```

### Bài 5

Giải thích: skeleton cho Banking System.

```csharp
public interface IInterestBearing
{
    decimal InterestRate { get; }
    void ApplyInterest();
}

public abstract class Account
{
    public string Owner { get; }
    public decimal Balance { get; protected set; }
    protected Account(string owner, decimal balance)
    {
        Owner = owner;
        Balance = balance;
    }
    public abstract void Describe();
}

public class SavingsAccount : Account, IInterestBearing
{
    public decimal InterestRate { get; }
    public SavingsAccount(string owner, decimal balance, decimal rate)
        : base(owner, balance) => InterestRate = rate;
    public void ApplyInterest() => Balance += Balance * InterestRate;
    public override void Describe()
        => Console.WriteLine($"Savings {Owner}: {Balance} rate={InterestRate}");
}
```

## 13. Đáp án thay thế

- ECS-style: data components thuần + systems (hướng Unity DOTS — chỉ cần biết tồn tại).
- Strategy inject qua constructor thay field settable.

## 14. Thử thách

Liệt kê 5 feature enemy (bay, khiên, độc, triệu hồi, biến hình). Ước lượng số class nếu dùng inheritance tổ hợp vs số component nếu composition — viết bảng ngắn.

## 15. Ứng dụng thực tế

- Clean architecture: phụ thuộc interface ở biên.
- Microservice clients: composition của HTTP clients.
- UI: panel có toolbar/components thay vì kế thừa sâu WinForms/WPF (tùy framework).

## 16. Liên hệ Unity — pitfalls inheritance

- **Pitfall 1:** `EnemyBase` 2000 dòng, mọi enemy `: EnemyBase` → merge conflict, sợ sửa.
- **Pitfall 2:** Override `Update` 4 tầng, quên `base.Update()` → bug lặn.
- **Pitfall 3:** Dùng inheritance để share prefab logic thay vì nested prefab / component.
- **Nên:** `Health`, `NavAgentMover`, `AttackRange`, `LootDrop` là component riêng; bật/tắt theo prefab.
- MonoBehaviour inheritance nông (1 tầng script cụ thể `: MonoBehaviour`) thường đủ.

## 17. Kiểm tra kiến thức

1. Khi nào abstract class hơn interface?  
2. Composition over inheritance nghĩa là gì?  
3. Dấu hiệu inheritance đang bị lạm dụng?  
4. `IFlyable` giúp Penguin thế nào?  
5. Trong Unity, analogue của composition là gì?

**Đáp án:**  
1) Cần code/state dùng chung mạnh trong một họ is-a.  
2) Ưu tiên “có thành phần” hơn “là subclass”.  
3) Cây sâu, tổ hợp feature nổ class, base mong manh.  
4) Không ép penguin bay — chỉ loài bay mới ký `IFlyable`.  
5) Gắn nhiều component trên GameObject.
