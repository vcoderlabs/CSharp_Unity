# Chương 3 — LSP (Liskov Substitution Principle)

## 1. Mục tiêu học

- Hiểu LSP: subtype phải **thay thế được** base mà không phá kỳ vọng
- Nhận diện inheritance “is-a” giả (Square/Rectangle, Penguin/Bird)
- Refactor bằng composition, interface hẹp hơn
- Tránh override làm “ném NotSupported” lung tung

## 2. Điều kiện tiên quyết

- Level 2: inheritance, polymorphism
- Level 16: SRP, OCP cơ bản
- Hiểu contract / precondition / postcondition (trực giác)

## 3. Khái niệm

**LSP (Liskov Substitution Principle):** Nếu `S` là subtype của `T`, mọi chỗ dùng `T` phải hoạt động đúng khi thay bằng `S` — **không bất ngờ**.

Vi phạm thường gặp:

| Pattern xấu | Vấn đề |
|-------------|--------|
| Override ném `NotSupportedException` | Client gọi base API → crash |
| Siết precondition (con đòi hơn cha) | Client hợp lệ với base lại fail với derived |
| Nới postcondition sai / làm yếu đảm bảo | Trả về null khi base hứa non-null |
| “Square kế thừa Rectangle” cổ điển | Đổi width làm height đổi — phá giả định |

## 4. Mô hình tư duy

```text
Client code viết cho Base
        │
        ▼
   Có thay Derived vào
   mà test/behavior vẫn đúng?
        │
   Có → LSP OK
   Không → đừng inherit; dùng interface riêng / composition
```

## 5. Cú pháp

```csharp
// Contract rõ trên abstraction đúng nghĩa
public interface IBird
{
    void Move();
}

public interface IFlyable
{
    void Fly();
}

public sealed class Sparrow : IBird, IFlyable
{
    public void Move() => Fly();
    public void Fly() { /* ... */ }
}

public sealed class Penguin : IBird
{
    public void Move() { /* swim/walk — không Fly */ }
}
```

## 6. Ví dụ

### Bad code

```csharp
public class Bird
{
    public virtual void Fly() => Console.WriteLine("flying");
}

public class Penguin : Bird
{
    public override void Fly()
        => throw new NotSupportedException("penguins can't fly");
}

void Migrate(Bird bird) => bird.Fly(); // Penguin → boom
```

### Problem

- `Migrate` tin mọi `Bird` bay được
- Thay `Penguin` phá hành vi — LSP vỡ
- Inheritance mô hình hóa sai domain

### Refactor

1. Tách khả năng bay khỏi “chim”.
2. `Migrate` nhận `IFlyable`, không nhận mọi `Bird`.
3. Hoặc composition: `FlightBehavior`.

### Good code

```csharp
public abstract class Bird
{
    public abstract void Move();
}

public sealed class Sparrow : Bird, IFlyable
{
    public override void Move() => Fly();
    public void Fly() => Console.WriteLine("sparrow flies");
}

public sealed class Penguin : Bird
{
    public override void Move() => Console.WriteLine("penguin swims");
}

void Migrate(IFlyable flyer) => flyer.Fly();
```

### Rectangle/Square (classic)

```csharp
// Bad: Square : Rectangle với set Width cũng set Height
// Client: r.Width=5; r.Height=4; expect Area=20 → Square cho 16

public interface IShape
{
    int Area { get; }
}

public sealed class RectangleShape : IShape
{
    public int Width { get; }
    public int Height { get; }
    public RectangleShape(int w, int h) { Width = w; Height = h; }
    public int Area => Width * Height;
}

public sealed class SquareShape : IShape
{
    public int Side { get; }
    public SquareShape(int side) => Side = side;
    public int Area => Side * Side;
}
```

### Unity example

```csharp
public abstract class EnemyBrain : MonoBehaviour
{
    public abstract void Tick();
}

// Bad: Turret : EnemyBrain nhưng override Tick empty +
//       code khác gọi GetComponent<EnemyBrain>().Tick() expect di chuyển

public class PatrolEnemy : EnemyBrain
{
    public override void Tick() { /* move along path */ }
}

public class TurretEnemy : EnemyBrain
{
    public override void Tick() { /* aim & shoot — vẫn là “tick AI” hợp lệ */ }
}

// Tách rõ capability
public interface IMovable { void Move(Vector3 delta); }
// Turret không implement IMovable → client không gọi Move nhầm
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| `NotSupported` trong override | API base quá rộng | Tách interface |
| `is`/`as` check derived khắp nơi | Polymorphism giả | Redesign hierarchy |
| Protected field bị derived phá invariant | Encapsulation yếu | Giữ invariant trong base; hạn chế protected mutable |
| Unity override `Update` rỗng để “tắt” | Dùng inheritance tắt behavior | Component enable/disable hoặc strategy |

## 8. Gỡ lỗi

1. Viết test cho base contract; chạy với mọi subtype.
2. Tìm `throw new NotSupportedException` trong override → cờ đỏ LSP.
3. Tìm `if (x is Special)` trong client dùng base → hierarchy sai hoặc thiếu type.

## 9. Best practices

- Prefer **composition** khi quan hệ không phải thay thế được.
- Interface nhỏ đúng khả năng (chuẩn bị ISP).
- Document invariant: “sau `Deposit`, balance không giảm”.
- Unity: component = khả năng; không nhét mọi enemy vào một cây kế thừa sâu.

## 10. Bài tập

**Bài 1** — Sửa `ReadOnlyList : List` (override Add throw) bằng thiết kế đúng.

**Bài 2** — Domain `Account` / `FrozenAccount` — tránh override `Withdraw` throw; đề xuất API.

**Bài 3** — Viết test chứng minh Square/Rectangle vi phạm nếu inherit sai.

**Bài 4** — Chọn: `NpcEnemy` có `Fly()` — nên inherit `Enemy` có `Walk()` không? Giải thích.

## 11. Gợi ý

- Bài 1: `IReadOnlyList` / trả về interface chỉ đọc, không kế thừa list mutable.
- Bài 2: state pattern hoặc `CanWithdraw` + result, không subtype phá API.
- Bài 4: composition `IFlightMovement` / `IGroundMovement`.

## 12. Đáp án

**Bài 1:**

```csharp
public interface IReadOnlyItems<T>
{
    int Count { get; }
    T this[int index] { get; }
}

public sealed class Items<T> : IReadOnlyItems<T>
{
    private readonly List<T> _items = new();
    public void Add(T item) => _items.Add(item);
    public int Count => _items.Count;
    public T this[int index] => _items[index];
    public IReadOnlyItems<T> AsReadOnly() => this;
}
```

**Bài 3 (ý tưởng test):** set width/height độc lập trên `Rectangle` reference trỏ `Square` → area không khớp kỳ vọng → fail LSP.

## 13. Đáp án thay thế

Dùng sealed class + factory thay hierarchy sâu. Hoặc discriminated union / pattern matching trên record thay inheritance.

## 14. Thử thách

Thiết kế `IWeapon` sao cho `BrokenWeapon` không phá client (không throw khi `Attack`); dùng Null Object hoặc trạng thái rõ ràng.

## 15. Ứng dụng thực tế

- Collection BCL tách `IEnumerable` / `ICollection` / `IList`
- Tránh “Fake” test double phá contract production
- API library: derived không được làm yếu đảm bảo versioning

## 16. Liên hệ Unity

- `MonoBehaviour` subclass không được làm vỡ kỳ vọng base custom của bạn
- AI enemy: capability interfaces thay vì `Enemy` siêu class
- Project chung: entity domain không inherit infrastructure type

## 17. Kiểm tra kiến thức

1. LSP yêu cầu gì khi thay subtype?  
   **Đáp án:** Hành vi vẫn đúng theo contract của base/abstraction.

2. Vì sao `NotSupported` trong override xấu?  
   **Đáp án:** Client hợp lệ với base bị crash với derived.

3. Square : Rectangle thường sai ở đâu?  
   **Đáp án:** Thay đổi kích thước phá giả định độc lập width/height.

4. Cách sửa Penguin/Bird?  
   **Đáp án:** Tách `IFlyable`; không mọi Bird đều Fly.

5. LSP liên hệ OCP?  
   **Đáp án:** Mở rộng bằng subtype chỉ an toàn nếu subtype thay thế được (LSP).
