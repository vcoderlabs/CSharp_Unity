# Chương 1 — Class, Object, Instance, Field, Property, Method

## 1. Mục tiêu học

- Phân biệt **class** (bản thiết kế) và **object/instance** (thực thể sống trong bộ nhớ)
- Khai báo **field**, **property**, **method** trong class
- Tạo object bằng `new` và gọi thành viên qua toán tử `.`
- Hiểu auto-property và property có logic get/set đơn giản

## 2. Điều kiện tiên quyết

- Hoàn thành Level 1 (biến, method, điều khiển luồng)
- Biết tạo và chạy project console: `dotnet new console`

## 3. Khái niệm

**Class** là khuôn mẫu mô tả *dữ liệu* và *hành vi* của một loại thực thể. Ví dụ đời thường: bản vẽ “Xe máy” — có biển số, dung tích bình xăng, hành vi `Chay()`, `DoXang()`.

**Object / Instance** là một chiếc xe cụ thể được tạo từ bản vẽ đó: xe của bạn biển `59-A1`, bình còn 3 lít.

Trong game: class `Enemy` mô tả máu, tốc độ, `TakeDamage()`; mỗi quái trên map là một **instance** riêng (`goblin1`, `goblin2`).

| Thành phần | Vai trò ngắn |
|------------|--------------|
| **Field** | Biến thành viên lưu trạng thái (thường `private`) |
| **Property** | Cửa truy cập có kiểm soát (`get`/`set`) tới dữ liệu |
| **Method** | Hành vi / thao tác trên object |

**Auto-property** (`public int Hp { get; set; }`) = compiler tự tạo field ẩn + get/set. Dùng khi chưa cần logic; dễ mở rộng sau.

## 4. Mô hình tư duy

```text
        Class Enemy                          Bộ nhớ (heap)
   ┌─────────────────────┐              ┌──────────────────┐
   │ fields / properties │   new →      │ Instance #1      │
   │ methods             │ ──────────►  │ Hp=50, Speed=2   │
   └─────────────────────┘              └──────────────────┘
                    │                   ┌──────────────────┐
                    └─────────────────► │ Instance #2      │
                                        │ Hp=80, Speed=1   │
                                        └──────────────────┘

Một class → nhiều object độc lập (trạng thái riêng).
```

## 5. Cú pháp

Giải thích: khai báo class với field, property và method; tạo instance bằng `new`.

```csharp
public class Player
{
    // Field (thường private)
    private string _name = "Hero";

    // Auto-property
    public int Hp { get; set; } = 100;

    // Property có logic
    public string Name
    {
        get => _name;
        set => _name = string.IsNullOrWhiteSpace(value) ? "Hero" : value.Trim();
    }

    // Method
    public void TakeDamage(int amount)
    {
        if (amount <= 0) return;
        Hp = Math.Max(0, Hp - amount);
    }

    public bool IsAlive() => Hp > 0;
}

// Dùng:
var p = new Player();
p.Name = "Aria";
p.TakeDamage(30);
Console.WriteLine($"{p.Name}: {p.Hp} HP, sống={p.IsAlive()}");
```

## 6. Ví dụ

### Cơ bản

Giải thích: class `BankAccount` tối giản — số dư và nạp tiền.

```csharp
public class BankAccount
{
    public string Owner { get; set; } = "";
    public decimal Balance { get; private set; }

    public void Deposit(decimal amount)
    {
        if (amount <= 0) return;
        Balance += amount;
    }
}

var acc = new BankAccount { Owner = "Lan" };
acc.Deposit(500);
Console.WriteLine($"{acc.Owner}: {acc.Balance}");
```

### Trung cấp

Giải thích: property `Level` tính từ `Xp`; method `AddXp` tăng điểm kinh nghiệm.

```csharp
public class Character
{
    public string Name { get; set; } = "NPC";
    public int Xp { get; private set; }

    public int Level => Xp / 100 + 1; // computed property

    public void AddXp(int amount)
    {
        if (amount <= 0) return;
        Xp += amount;
    }
}

var c = new Character { Name = "Mage" };
c.AddXp(250);
Console.WriteLine($"{c.Name} Lv{c.Level} (XP={c.Xp})");
```

### Nâng cao

Giải thích: nhiều instance độc lập; thay đổi một object không ảnh hưởng object khác cùng class.

```csharp
public class InventorySlot
{
    public string ItemId { get; set; } = "";
    public int Quantity { get; set; }

    public void Add(int n)
    {
        if (n > 0) Quantity += n;
    }
}

var potion = new InventorySlot { ItemId = "potion", Quantity = 1 };
var sword = new InventorySlot { ItemId = "sword", Quantity = 1 };
potion.Add(2);
Console.WriteLine($"potion={potion.Quantity}, sword={sword.Quantity}");
// potion=3, sword=1 — hai instance tách biệt
```

## 7. Lỗi thường gặp

| Lỗi | Nguyên nhân | Cách xử lý |
|-----|-------------|------------|
| `NullReferenceException` | Gọi method trên biến object chưa `new` | Luôn `new` trước khi dùng |
| Quên `new` | Chỉ khai báo `Player p;` rồi dùng | `var p = new Player();` |
| Field public lung tung | Ai cũng ghi đè trạng thái | Dùng property + `private set` / method |
| Nhầm class với object | Nghĩ sửa class là sửa mọi instance | Class = khuôn; instance = dữ liệu riêng |

## 8. Gỡ lỗi

1. In `GetHashCode()` hoặc Id tùy chỉnh để chắc bạn đang nhìn đúng instance.
2. Breakpoint ngay sau `new` — kiểm tra property mặc định.
3. Nếu “đổi A mà B cũng đổi”: bạn đang dùng chung **một** reference (học sâu ở Level 3).
4. `dotnet build` thường xuyên khi thêm thành viên mới.

## 9. Best practices

- Tên class: **PascalCase**, danh từ (`Player`, `Order`).
- Field private: tiền tố `_` (`_name`) theo convention phổ biến.
- Ưu tiên property thay vì field public.
- Method = động từ rõ (`Deposit`, `TakeDamage`).
- Một class một trách nhiệm chính (sẽ mở rộng ở Clean Code / SOLID).

## 10. Bài tập

**Bài 1 — Book**  
Tạo class `Book` với `Title`, `Author`, `Pages`. Nhập 1 cuốn từ bàn phím, in ra.

**Bài 2 — Counter**  
Class `Counter` có `Value` (chỉ tăng qua `Increment()` / `Add(int)`). In giá trị sau vài lần gọi.

**Bài 3 — Rectangle**  
`Width`, `Height`; property `Area` và `Perimeter` chỉ đọc. Nhập W/H, in diện tích & chu vi.

**Bài 4 — Student điểm**  
`Name`, `Score` (0–10, set phải validate). Method `Grade()` trả `"A"/"B"/"C"/"D"/"F"`.

**Bài 5 — Hai Player so máu**  
Tạo 2 `Player`, trừ máu khác nhau, in ai còn sống / ai nhiều HP hơn.

## 11. Gợi ý

- Bài 1–3: bắt đầu auto-property; chưa cần constructor (Chương 2).
- Bài 4: logic validate nằm trong `set` hoặc method `SetScore`.
- Bài 5: nhắc lại “hai instance độc lập”.

## 12. Đáp án + Giải thích

### Bài 1

Giải thích: đọc Title/Author/Pages, gán vào object `Book`, rồi in.

```csharp
public class Book
{
    public string Title { get; set; } = "";
    public string Author { get; set; } = "";
    public int Pages { get; set; }
}

Console.Write("Title: ");
var title = Console.ReadLine() ?? "";
Console.Write("Author: ");
var author = Console.ReadLine() ?? "";
Console.Write("Pages: ");
int.TryParse(Console.ReadLine(), out int pages);

var book = new Book { Title = title, Author = author, Pages = pages };
Console.WriteLine($"{book.Title} — {book.Author} ({book.Pages} trang)");
```

### Bài 2

Giải thích: `Value` chỉ tăng qua method; `private set` chặn gán trực tiếp từ ngoài.

```csharp
public class Counter
{
    public int Value { get; private set; }
    public void Increment() => Value++;
    public void Add(int n) { if (n > 0) Value += n; }
}

var c = new Counter();
c.Increment();
c.Add(5);
Console.WriteLine(c.Value); // 6
```

### Bài 3

Giải thích: `Area`/`Perimeter` là computed property — không lưu thừa dữ liệu.

```csharp
public class Rectangle
{
    public double Width { get; set; }
    public double Height { get; set; }
    public double Area => Width * Height;
    public double Perimeter => 2 * (Width + Height);
}

Console.Write("W H: ");
var parts = (Console.ReadLine() ?? "").Split(' ', StringSplitOptions.RemoveEmptyEntries);
double.TryParse(parts.ElementAtOrDefault(0), out double w);
double.TryParse(parts.ElementAtOrDefault(1), out double h);
var r = new Rectangle { Width = w, Height = h };
Console.WriteLine($"Area={r.Area}, Perimeter={r.Perimeter}");
```

### Bài 4

Giải thích: validate trong setter; `Grade()` map điểm sang chữ.

```csharp
public class Student
{
    public string Name { get; set; } = "";
    private double _score;
    public double Score
    {
        get => _score;
        set => _score = Math.Clamp(value, 0, 10);
    }

    public string Grade() => Score switch
    {
        >= 8.5 => "A",
        >= 7 => "B",
        >= 5.5 => "C",
        >= 4 => "D",
        _ => "F"
    };
}

var s = new Student { Name = "Minh", Score = 9.2 };
Console.WriteLine($"{s.Name}: {s.Score} → {s.Grade()}");
```

### Bài 5

Giải thích: hai `Player` riêng; so sánh `Hp` sau khi nhận sát thương khác nhau.

```csharp
public class Player
{
    public string Name { get; set; } = "";
    public int Hp { get; set; } = 100;
    public void TakeDamage(int dmg) => Hp = Math.Max(0, Hp - Math.Max(0, dmg));
    public bool IsAlive => Hp > 0;
}

var a = new Player { Name = "A" };
var b = new Player { Name = "B" };
a.TakeDamage(40);
b.TakeDamage(90);
Console.WriteLine($"{a.Name}:{a.Hp} sống={a.IsAlive}");
Console.WriteLine($"{b.Name}:{b.Hp} sống={b.IsAlive}");
Console.WriteLine(a.Hp >= b.Hp ? "A nhiều HP hơn hoặc bằng" : "B nhiều HP hơn");
```

## 13. Đáp án thay thế

- Dùng field `private` + method `GetX`/`SetX` kiểu Java thay property (hợp lệ nhưng kém idiomatic C#).
- `record` C# cho dữ liệu bất biến — học sau; Level 2 ưu tiên `class` cổ điển.
- Top-level statements + class trong cùng file hoặc tách `Book.cs`.

## 14. Thử thách

Viết class `ShoppingCart`: danh sách tên mặt hàng (`List<string>`), method `Add`/`Remove`/`Count`, property `IsEmpty`. In giỏ sau vài thao tác.

## 15. Ứng dụng thực tế

- Domain model: `Order`, `Customer`, `Invoice` trong app nghiệp vụ.
- DTO / entity trong API và ORM.
- Mọi hệ thống lớn đều bắt đầu từ “dữ liệu + hành vi gắn với thực thể”.

## 16. Liên hệ Unity

- Mỗi script gắn GameObject thường là class kế thừa `MonoBehaviour` — **một instance component** trên object đó.
- `GetComponent<T>()` lấy instance component trên cùng GameObject.
- Field trong Inspector ≈ serialize field của instance; đừng nhầm với “static toàn game” (Chương 4).
- Tránh nhồi mọi logic vào một God-class `Player` khổng lồ — tách class/`MonoBehaviour` nhỏ (composition).

## 17. Kiểm tra kiến thức

1. Class khác object chỗ nào?  
2. `new Player()` làm gì?  
3. Field và property khác nhau ra sao?  
4. Hai instance cùng class có chia sẻ field instance không?  
5. Auto-property dùng khi nào?

**Đáp án:**  
1) Class = khuôn mẫu; object = thực thể tạo từ khuôn.  
2) Cấp phát instance mới trên heap (chi tiết Level 3) và gọi constructor.  
3) Field lưu trực tiếp; property là cổng get/set (có thể có logic).  
4) Không — mỗi instance có bộ field riêng (trừ `static`).  
5) Khi chưa cần logic get/set; vẫn giữ API dạng property để mở rộng sau.
