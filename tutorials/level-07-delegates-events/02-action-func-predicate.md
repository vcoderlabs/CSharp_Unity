# Chương 2 — Action, Func, Predicate

## 1. Mục tiêu học

- Dùng `Action`, `Action<T...>`, `Func<T...,TResult>`, `Predicate<T>` thành thạo
- Chọn đúng generic theo chữ ký (void vs có return, bool)
- Thay delegate tự viết bằng BCL khi phù hợp
- Nhận ra các API nhận Action/Func (List, Task, LINQ sau này)

## 2. Điều kiện tiên quyết

- Chương 1: delegate, method group, Invoke
- Level 5: generic type parameters

## 3. Khái niệm

| Kiểu | Ý nghĩa | Ví dụ chữ ký |
|------|---------|--------------|
| `Action` | void, 0 tham số | `() => ...` |
| `Action<T>` | void, 1 tham số | `(x) => ...` |
| `Action<T1,T2,...>` | void, nhiều tham số (tới 16) | |
| `Func<TResult>` | trả về TResult, 0 tham số | |
| `Func<T,TResult>` | 1 in + return | |
| `Func<...,TResult>` | **Tham số cuối là kiểu trả về** | |
| `Predicate<T>` | `bool` từ `T` — tương đương `Func<T,bool>` | |

```csharp
Action<string> print = Console.WriteLine;
Func<int, int, int> add = (a, b) => a + b;
Predicate<int> even = n => n % 2 == 0;
```

## 4. Mô hình tư duy

```text
Có trả về không?
  Không → Action<...>
  Có → Func<..., TResult>   (TResult luôn cuối)
Chỉ cần bool từ T?
  → Predicate<T> hoặc Func<T,bool> (cùng ý)

List.Find(Predicate<T>)
Task.Run(Action) / Task.Run(Func<T>)
```

## 5. Cú pháp

```csharp
Action greet = () => Console.WriteLine("Hi");
Action<int> show = x => Console.WriteLine(x);
Action<string, int> repeat = (s, n) =>
{
    for (int i = 0; i < n; i++) Console.WriteLine(s);
};

Func<DateTime> now = () => DateTime.UtcNow;
Func<int, int> square = x => x * x;
Func<int, int, string> format = (a, b) => $"{a}+{b}={a + b}";

Predicate<string> longName = s => s.Length > 5;

greet();
show(3);
Console.WriteLine(square(4));
```

Truyền vào method:

```csharp
static void Process(int[] data, Action<int> sink)
{
    foreach (var x in data) sink(x);
}
```

## 6. Ví dụ

### Cơ bản

```csharp
Action tick = () => Console.WriteLine("tick");
tick();

Func<int, bool> positive = n => n > 0;
Console.WriteLine(positive(-1)); // False
```

### Trung cấp

```csharp
var list = new List<int> { 1, 2, 3, 4, 5 };
Predicate<int> odd = n => n % 2 != 0;
int firstOdd = list.Find(odd);
List<int> odds = list.FindAll(odd);
list.ForEach(x => Console.WriteLine(x)); // Action<int>
```

### Nâng cao

Higher-order: trả về Func:

```csharp
static Func<int, int> Multiplier(int k) => x => x * k;

var times3 = Multiplier(3);
Console.WriteLine(times3(10)); // 30
```

(Closure — chi tiết chương 4.)

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Nhầm `Func<int,int,int>` thứ tự | Nghĩ tham số đầu là return | Return **cuối** |
| Dùng Action khi cần giá trị | Action luôn void | Đổi Func |
| `Predicate` vs `Func<T,bool>` không gán được? | Kiểu danh nghĩa khác | Có thể tương thích qua method group/lambda mới |
| Quá tải Action 0–16 | Truyền thừa/thiếu generic | Đếm lại type args |

## 8. Gỡ lỗi

1. Đọc tooltip chữ ký `Func<T1,T2,TResult>`.  
2. Nếu compile “cannot convert”: so return type void vs non-void.  
3. Viết tạm biến trung gian `Func<...> f = ...;` để IDE báo rõ.

## 9. Best practices

- Public API: `Action`/`Func` dễ dùng hơn custom delegate (trừ khi domain cần tên riêng / event args).
- Đặt tên biến mô tả hành vi: `onComplete`, `selector`, `predicate`.
- Prefer method group khi method đã có sẵn: `list.Find(IsValid)`.
- Không nhồi logic khổng lồ vào lambda — tách method.

## 10. Bài tập

**Bài 1** — `Action<string, string>` ghép và in hai chuỗi.

**Bài 2** — `Func<double, double, double>` tính hypot (`Math.Sqrt(a*a+b*b)`).

**Bài 3** — Method `CountWhere(int[] data, Predicate<int> match)`.

**Bài 4** — `Apply(int x, Func<int,int> f)` gọi `f` ba lần liên tiếp (compose tự áp).

## 11. Gợi ý

- Bài 3: biến đếm, foreach, `if (match(x)) count++`.
- Bài 4: `x = f(x);` lặp 3 lần rồi return.

## 12. Đáp án

**Bài 1** — Action hai tham số:

```csharp
Action<string, string> joinPrint = (a, b) => Console.WriteLine(a + b);
joinPrint("Hello, ", "World");
```

**Bài 2** — Func hypot:

```csharp
Func<double, double, double> hypot =
    (a, b) => Math.Sqrt(a * a + b * b);
Console.WriteLine(hypot(3, 4)); // 5
```

**Bài 3** — Đếm theo Predicate:

```csharp
static int CountWhere(int[] data, Predicate<int> match)
{
    int count = 0;
    foreach (int x in data)
        if (match(x)) count++;
    return count;
}

Console.WriteLine(CountWhere(new[] { 1, 2, 3, 4 }, n => n > 2)); // 2
```

**Bài 4** — Áp hàm ba lần:

```csharp
static int ApplyThrice(int x, Func<int, int> f)
{
    x = f(x);
    x = f(x);
    x = f(x);
    return x;
}

Console.WriteLine(ApplyThrice(2, n => n + 1)); // 5
```

## 13. Đáp án thay thế

Bài 3 dùng `Func<int, bool>` thay `Predicate<int>`. Bài 4 vòng `for (int i = 0; i < 3; i++)`.

## 14. Thử thách

Viết `Map` và `Filter` trên `List<T>` nhận `Func<T,TOut>` và `Predicate<T>` — không dùng LINQ.

## 15. Ứng dụng thực tế

- DI callback, options builders
- `HttpClient` handlers / Polly policies
- LINQ: `Where(Func)`, `Select(Func)`
- Parallel: `Parallel.ForEach(..., Action<T>)`

## 16. Liên hệ Unity

- `UnityEvent` ≈ danh sách `UnityAction` (gần `Action`) gọi tuần tự
- `UnityEvent<T>` ≈ `Action<T>` có serialize trên Inspector
- Code thuần: `public Action OnDied;` vs UnityEvent — chọn UnityEvent khi Designer cần gán trong Editor
- `StartCoroutine` không phải Action — đừng nhầm; nhưng callback hoàn thành thường là Action

## 17. Kiểm tra kiến thức

1. `Func<int, string>` nhận gì, trả gì?  
   **Đáp án:** Nhận `int`, trả `string`.

2. `Action<int>` trả về gì?  
   **Đáp án:** Không trả về (void).

3. `Predicate<T>` tương đương Func nào?  
   **Đáp án:** `Func<T, bool>`.

4. Tham số cuối của `Func` là gì?  
   **Đáp án:** Kiểu kết quả `TResult`.

5. `List<T>.ForEach` nhận kiểu gì?  
   **Đáp án:** `Action<T>`.
