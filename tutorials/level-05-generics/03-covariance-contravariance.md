# Chương 3 — Covariance và Contravariance

## 1. Mục tiêu học

- Hiểu **covariance** (`out T`) và **contravariance** (`in T`)
- Biết chúng áp dụng cho **generic interface** và **delegate**, không phải class thường
- Giải thích được tại sao `IEnumerable<Dog>` gán được vào `IEnumerable<Animal>` nhưng `List` thì không
- Tránh nhầm lẫn khi thiết kế API generic

## 2. Điều kiện tiên quyết

- Chương 1–2: generic interface, inheritance (Level 2)
- Biết quan hệ kế thừa: `Dog : Animal`

## 3. Khái niệm

### Assignment thông thường với mảng (lịch sử — nguy hiểm)

```csharp
Dog[] dogs = { new Dog() };
Animal[] animals = dogs; // được phép với array — runtime có thể ArrayTypeMismatchException khi ghi
```

### Covariance (`out T`) — “output / producer”

`T` chỉ xuất hiện ở vị trí **trả về** (không nhận `T` làm input an toàn để ghi).

```csharp
interface IProducer<out T>
{
    T Get();
}

IProducer<Dog> dogProd = ...;
IProducer<Animal> animalProd = dogProd; // OK: Dog là Animal
```

`IEnumerable<out T>` là ví dụ kinh điển: chỉ đọc `T` khi duyệt.

### Contravariance (`in T`) — “input / consumer”

`T` chỉ xuất hiện ở vị trí **tham số vào**.

```csharp
interface IConsumer<in T>
{
    void Accept(T item);
}

IConsumer<Animal> animalCons = ...;
IConsumer<Dog> dogCons = animalCons; // OK: chỗ cần nhận Dog, nhận Animal vẫn xử lý được
```

`Action<in T>`: `Action<Animal>` dùng được nơi cần `Action<Dog>`.

### Invariance (mặc định)

`List<T>` **invariant**: `List<Dog>` **không** phải `List<Animal>` — vì nếu gán được, `animals.Add(new Cat())` phá vỡ kiểu.

| Từ khóa | Hướng gán (T cụ thể → T cơ sở) | Vai trò T |
|---------|--------------------------------|-----------|
| `out T` | `G<Derived>` → `G<Base>` | Producer |
| `in T` | `G<Base>` → `G<Derived>` | Consumer |
| (không) | Không chuyển | Đọc + ghi |

## 4. Mô hình tư duy

```text
Dog : Animal

Covariance (out) — “lấy ra”:
  IEnumerable<Dog>  ──►  IEnumerable<Animal>
  (mỗi Dog lấy ra đều là Animal)

Contravariance (in) — “đưa vào”:
  Action<Animal>  ──►  Action<Dog>
  (hàm xử lý mọi Animal ắt xử lý được Dog)

List<Dog>  ✖  List<Animal>
  (vì List vừa đọc vừa Add)
```

## 5. Cú pháp

```csharp
interface IReadOnlyRepository<out T>
{
    T GetById(int id);
    IEnumerable<T> GetAll();
    // void Add(T item); // ILLEGAL với out T
}

interface IWriter<in T>
{
    void Save(T item);
    // T Get(); // ILLEGAL với in T
}

delegate T Factory<out T>();
delegate void Sink<in T>(T value);
```

## 6. Ví dụ

### Cơ bản

```csharp
IEnumerable<string> strings = ["a", "b"];
IEnumerable<object> objects = strings; // covariance của IEnumerable<out T>
```

### Trung cấp

```csharp
Action<object> printObj = o => Console.WriteLine(o);
Action<string> printStr = printObj; // contravariance Action<in T>
printStr("hello");
```

### Nâng cao

Thiết kế API:

```csharp
interface IConverter<in TIn, out TOut>
{
    TOut Convert(TIn input);
}

// string → object: TIn=string (in), TOut=object (out)
IConverter<string, object> c = new BoxConverter();
object boxed = c.Convert("x");
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Không gán `List<Dog>` vào `List<Animal>` | List invariant | Dùng `IEnumerable<Animal>` để đọc |
| CS1961 Invalid variance | `out T` nhưng có method nhận `T` | Tách interface đọc/ghi |
| Nghĩ class `Box<out T>` được | Class không hỗ trợ variant type params như interface | Dùng interface |
| Lẫn `in`/`out` tham số method với variance | Khác nghĩa hoàn toàn | Variance chỉ trên khai báo generic interface/delegate |

## 8. Gỡ lỗi

1. Hỏi: “Tôi chỉ **đọc** hay cũng **ghi** `T`?” — chỉ đọc → cân nhắc `out`.
2. Vẽ mũi tên kế thừa Base/Derived rồi thử gán hai chiều.
3. Nếu compiler cấm `out` vì có `Add(T)` — tách `IReadOnlyRepo` / `IWriteRepo`.

## 9. Best practices

- Public API chỉ đọc: trả `IEnumerable<T>` / `IReadOnlyList<T>` (covariance hữu ích).
- Tách read/write interface khi cần variance.
- Đừng force `out`/`in` nếu `T` vừa vào vừa ra — để invariant.
- Document ví dụ gán hợp lệ trong XML doc nếu API public phức tạp.

## 10. Bài tập

**Bài 1** — Giải thích (viết 3–5 câu) vì sao đoạn sau **không** compile: `List<Animal> a = new List<Dog>();`

**Bài 2** — Khai báo `interface IGetter<out T> { T Get(); }` và chứng minh gán `IGetter<Dog>` → `IGetter<Animal>`.

**Bài 3** — Khai báo `interface ISetter<in T> { void Set(T value); }` và chứng minh gán `ISetter<Animal>` → `ISetter<Dog>`.

**Bài 4** — Chỉ ra method nào làm `out T` không hợp lệ trong interface có cả `T Get()` và `void Add(T item)`.

## 11. Gợi ý

- Bài 1: Add Cat vào List&lt;Animal&gt; nếu alias List&lt;Dog&gt;.
- Bài 2–3: viết class implement rồi gán biến.
- Bài 4: `Add(T)` là vị trí input — xung đột `out`.

## 12. Đáp án

**Bài 1** — Giải thích:

Nếu phép gán được phép, code sau sẽ thêm `Cat` vào danh sách thực chất chỉ chứa `Dog` → phá vỡ type safety. Vì vậy `List<T>` invariant.

**Bài 2** — Covariance:

```csharp
interface IGetter<out T> { T Get(); }

class DogGetter : IGetter<Dog>
{
    public Dog Get() => new Dog();
}

IGetter<Dog> dg = new DogGetter();
IGetter<Animal> ag = dg; // OK với out
Animal a = ag.Get();
```

**Bài 3** — Contravariance:

```csharp
interface ISetter<in T> { void Set(T value); }

class AnimalSetter : ISetter<Animal>
{
    public void Set(Animal value) => Console.WriteLine(value);
}

ISetter<Animal> aSet = new AnimalSetter();
ISetter<Dog> dSet = aSet; // OK với in
dSet.Set(new Dog());
```

**Bài 4** — `void Add(T item)` không hợp lệ trên interface `out T` vì `T` ở vị trí input. Phải bỏ `out` hoặc tách interface.

## 13. Đáp án thay thế

Dùng sẵn BCL: chứng minh bằng `IEnumerable<Dog>` → `IEnumerable<Animal>` và `Action<Animal>` → `Action<Dog>` mà không tự khai báo interface.

## 14. Thử thách

Thiết kế `IBuffer<T>` ban đầu invariant. Tách thành `IReadBuffer<out T>` và `IWriteBuffer<in T>`, class `Buffer<T> : IReadBuffer<T>, IWriteBuffer<T>`. Viết demo gán variance.

## 15. Ứng dụng thực tế

- LINQ nhận `IEnumerable<out T>` — pipeline linh hoạt
- Event handler / callback `Action<in TEventArgs>`
- DI: đăng ký handler cơ sở dùng cho event derived
- API thư viện: read-only covariant surfaces

## 16. Liên hệ Unity

- Ít viết `out`/`in` tay, nhưng hưởng lợi khi truyền `IEnumerable<Component>` / interface
- `UnityEvent<T>` và delegate callback: tư duy consumer/producer giúp thiết kế listener
- Tránh coi `List<Enemy>` như `List<MonoBehaviour>` — phải Cast/OfType (LINQ) có chủ đích

## 17. Kiểm tra kiến thức

1. `out T` nghĩa là gì?  
   **Đáp án:** Covariance — T ở vị trí output; `G<Derived>` → `G<Base>`.

2. `in T` nghĩa là gì?  
   **Đáp án:** Contravariance — T ở vị trí input; `G<Base>` → `G<Derived>`.

3. `List<T>` có variant không?  
   **Đáp án:** Không — invariant.

4. Vì sao `IEnumerable<Dog>` gán được cho `IEnumerable<Animal>`?  
   **Đáp án:** `IEnumerable<out T>` chỉ đọc; mỗi Dog là Animal.

5. Variance áp dụng cho class generic thường?  
   **Đáp án:** Không — chủ yếu interface và delegate.
