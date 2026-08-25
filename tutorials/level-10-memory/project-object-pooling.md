# Project Level 10 — Object Pooling System

## 1. Mục tiêu học

- Xây **hệ thống object pooling** hoàn chỉnh bằng C# (.NET 8 console)
- Đo được **allocation trước/sau** — minh chứng giảm GC pressure
- Áp dụng reset state, prewarm, max capacity, thống kê Rent/Return
- Chuẩn bị tư duy mang sang Unity (bullet / VFX pool) chống **GC spike**

## 2. Điều kiện tiên quyết

- Hoàn thành 7 chương Level 10
- Level 5: generics
- .NET 8 SDK: `dotnet new console`

## 3. Khái niệm / Yêu cầu sản phẩm

### Phần A — `IObjectPool<T>` + `ObjectPool<T>`

| Member | Mô tả |
|--------|--------|
| `T Rent()` | Lấy object; nếu hết thì `create` hoặc throw theo policy |
| `void Return(T item)` | Trả object; gọi reset; tôn trọng `maxSize` |
| `int CountFree { get; }` | Số đang nhàn |
| `int CountRented { get; }` | Đang cho thuê (track) |
| `void Prewarm(int count)` | Tạo sẵn |
| `void Clear()` | Tháo hết free (test / teardown) |

Yêu cầu:

- Generic `T : class`
- Nhận `Func<T> create`, `Action<T>? onRent`, `Action<T>? onReturn`
- **Không** double-return (detect bằng `HashSet<T>` rented hoặc flag trên item)
- Policy khi pool đầy lúc Return: **drop** (không giữ, để GC) *hoặc* expand — chọn 1, ghi README

### Phần B — Domain demo: `Projectile`

```csharp
sealed class Projectile
{
    public int Id { get; set; }
    public float X { get; set; }
    public float Y { get; set; }
    public float Vx { get; set; }
    public float Vy { get; set; }
    public bool IsActive { get; set; }
    public int Damage { get; set; }
}
```

Mô phỏng:

1. Prewarm 100 projectile  
2. Mỗi “frame”: spawn 10, cập nhật vị trí, despawn khi `X > 100`  
3. Chạy 1000 frame  

### Phần C — Benchmark alloc

Hai chế độ:

- `UsePool = false`: mỗi spawn `new Projectile()`  
- `UsePool = true`: Rent/Return  

In:

- Bytes allocated (`GC.GetAllocatedBytesForCurrentThread`)
- `GC.CollectionCount(0/1/2)` delta
- Thời gian `Stopwatch`

### Phần D — (Bonus) `ListPool` / `ArrayPool` wrapper

Thuê `List<int>` hoặc `byte[]` cho “aoe query” giả lập — Return kèm `Clear`.

## 4. Mô hình tư duy

```text
ObjectPool<Projectile>
  _free: Stack<Projectile>
  _rented: HashSet<Projectile>   // chống double return
  create / onRent / onReturn

Frame loop:
  for shots:
    p = pool.Rent()
    setup(p)
    active.Add(p)
  foreach active:
    integrate velocity
    if out of bounds:
      pool.Return(p)
      remove from active

Không pool: active.Add(new Projectile()) — Gen0 cháy
```

```text
Unity mapping (không bắt buộc code Unity trong project):
  Rent     ≈ Get from pool + SetActive(true)
  Return   ≈ SetActive(false) + về list
  Prewarm  ≈ Instantiate ẩn lúc Load
```

## 5. Cú pháp / Skeleton

```bash
dotnet new console -n ObjectPoolingSystem -f net8.0
cd ObjectPoolingSystem
```

Gợi ý file:

- `IObjectPool.cs`
- `ObjectPool.cs`
- `Projectile.cs`
- `Simulation.cs`
- `Program.cs`

Skeleton pool:

```csharp
public sealed class ObjectPool<T> : IObjectPool<T> where T : class
{
    private readonly Stack<T> _free = new();
    private readonly HashSet<T> _rented = new();
    private readonly Func<T> _create;
    private readonly Action<T>? _onRent;
    private readonly Action<T>? _onReturn;
    private readonly int _maxFree;

    public ObjectPool(
        Func<T> create,
        Action<T>? onRent = null,
        Action<T>? onReturn = null,
        int maxFree = 1024)
    {
        _create = create;
        _onRent = onRent;
        _onReturn = onReturn;
        _maxFree = maxFree;
    }

    public int CountFree => _free.Count;
    public int CountRented => _rented.Count;

    public T Rent()
    {
        var item = _free.Count > 0 ? _free.Pop() : _create();
        _rented.Add(item);
        _onRent?.Invoke(item);
        return item;
    }

    public void Return(T item)
    {
        if (!_rented.Remove(item))
            throw new InvalidOperationException("Double return or foreign object");
        _onReturn?.Invoke(item);
        if (_free.Count < _maxFree)
            _free.Push(item);
        // else: drop → GC
    }
}
```

## 6. Ví dụ hướng dẫn

### Cơ bản — Prewarm

```csharp
public void Prewarm(int count)
{
    for (int i = 0; i < count; i++)
        _free.Push(_create());
}
```

### Trung cấp — Reset projectile

```csharp
void ResetProjectile(Projectile p)
{
    p.IsActive = false;
    p.X = p.Y = 0;
    p.Vx = p.Vy = 0;
    p.Damage = 0;
}
```

### Nâng cao — Simulation + đo

```csharp
static void Run(bool usePool, int frames)
{
    ObjectPool<Projectile>? pool = null;
    if (usePool)
    {
        pool = new ObjectPool<Projectile>(
            create: () => new Projectile(),
            onReturn: ResetProjectile,
            maxFree: 256);
        pool.Prewarm(100);
    }

    var active = new List<Projectile>(256);
    int nextId = 1;
    int g0 = GC.CollectionCount(0);
    long alloc0 = GC.GetAllocatedBytesForCurrentThread();
    var sw = Stopwatch.StartNew();

    for (int f = 0; f < frames; f++)
    {
        for (int s = 0; s < 10; s++)
        {
            var p = usePool ? pool!.Rent() : new Projectile();
            p.Id = nextId++;
            p.IsActive = true;
            p.X = 0; p.Y = 0;
            p.Vx = 1 + (s % 3); p.Vy = 0;
            p.Damage = 10;
            active.Add(p);
        }

        for (int i = active.Count - 1; i >= 0; i--)
        {
            var p = active[i];
            p.X += p.Vx;
            if (p.X > 100)
            {
                if (usePool) pool!.Return(p);
                active.RemoveAt(i);
            }
        }
    }

    sw.Stop();
    Console.WriteLine(usePool ? "POOL" : "NEW");
    Console.WriteLine($"  time ms: {sw.ElapsedMilliseconds}");
    Console.WriteLine($"  alloc bytes: {GC.GetAllocatedBytesForCurrentThread() - alloc0}");
    Console.WriteLine($"  gen0 collects: {GC.CollectionCount(0) - g0}");
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Alloc pool vẫn cao | `List.RemoveAt` / lambda / boxing trong loop | Pre-size list; tránh LINQ |
| `InvalidOperationException` Return | Despawn 2 lần | Guard `IsActive` |
| So sánh alloc không công bằng | JIT warm-up | Chạy warm-up 1 lần trước khi đo |
| Pool “thắng” nhưng logic sai | Không Return | Đếm `CountRented` cuối = 0 |
| HashSet overhead | Tracking cost | Chấp nhận học; hoặc flag `PooledNode` |

## 8. Gỡ lỗi

1. Cuối sim: `CountRented == 0`, `active.Count == 0`.  
2. Warm-up: gọi `Run` một lần bỏ kết quả, rồi đo chính thức.  
3. Chạy `dotnet run -c Release`.  
4. Thêm log nếu `Rent` phải `_create()` sau prewarm — capacity thiếu.

## 9. Best practices

- Release build khi benchmark.
- Tách “game logic” và “pool infrastructure”.
- Document policy maxFree/drop.
- Reset **toàn bộ** field public.
- README project: bảng số liệu NEW vs POOL + câu kết nối Unity GC spike.

## 10. Bài tập (checklist deliverable)

**Bài 1** — Hoàn thành `ObjectPool<T>` + test double-return throw.

**Bài 2** — Simulation 1000 frame × 10 spawn như trên.

**Bài 3** — In bảng so sánh NEW vs POOL (alloc, Gen0, ms).

**Bài 4** — Viết `README.md` trong project console (ngắn): cách chạy + kết quả + đoạn “Áp dụng Unity”.

## 11. Gợi ý

- Đừng đo trong lần chạy đầu (JIT).
- `RemoveAt` từ cuối list khi despawn.
- Prewarm ≥ peak concurrent projectiles (ước lượng).
- Nếu alloc pool vẫn lớn: kiểm tra `string`/`Console` trong vòng đo.

## 12. Đáp án

Hướng đáp án chính: skeleton mục 5 + simulation mục 6. Kết quả kỳ vọng (số tuyệt đối máy bạn khác nhau):

```text
NEW : alloc bytes lớn (triệu+), Gen0 collects > 0 nhiều hơn
POOL: alloc bytes thấp hơn rõ (gần prewarm only), Gen0 ít hơn
```

Double-return:

```csharp
var p = pool.Rent();
pool.Return(p);
// pool.Return(p); // → InvalidOperationException
```

`Program.cs` tối thiểu:

```csharp
Console.WriteLine("Warm-up...");
Run(usePool: true, frames: 100);
Run(usePool: false, frames: 100);

Console.WriteLine("Measure...");
Run(usePool: false, frames: 1000);
Run(usePool: true, frames: 1000);
```

## 13. Đáp án thay thế

- Dùng `Queue<T>` thay `Stack<T>`
- Không `HashSet`: thêm `bool _inPool` trên `IPoolable`
- NuGet `Microsoft.Extensions.ObjectPool` — so sánh API với bản tự viết
- Bonus: `ArrayPool<byte>` cho payload giả “network packet”

## 14. Thử thách

1. Đa loại pool: `Projectile` + `DamageNumber` (string builder pool).  
2. Thread-safe pool (`ConcurrentBag` + lock policy) — optional.  
3. Vẽ ASCII report cuối run: peak rented, expand count, drop count.  
4. Viết hướng dẫn 10 dòng “port sang Unity ObjectPool\<GameObject\>”.

## 15. Ứng dụng thực tế

- Combat projectile / hitscan VFX  
- Floating combat text  
- Enemy trash mobs  
- Network packet buffer  
- Temporary collections trong ECS/systems  

## 16. Liên hệ Unity

Mang project này sang Unity:

| Console | Unity |
|---------|--------|
| `create: () => new Projectile()` | `create: () => Object.Instantiate(prefab)` |
| `onRent` set active fields | `go.SetActive(true)` + reset transform |
| `onReturn` | `SetActive(false)` + parent về pool root |
| Prewarm | Lúc load scene / additive dungeon |
| Đo `GetAllocatedBytes` | Profiler **GC Alloc** + Deep Profile |

**Mục tiêu cảm nhận:** bắn skill liên tục **không** còn cột GC Alloc nhảy; không còn hitch đồng bộ với `GC.Collect`.

Nhắc lại cảnh báo Level 10:

> GC spike là nguyên nhân hàng đầu gây giật trong Unity MMORPG. Object pooling là công cụ số một bạn vừa tự xây.

## 17. Kiểm tra kiến thức

1. Vì sao cần track `_rented` / double-return?  
   **Đáp án:** Tránh một object vào free hai lần → corruption / double use.

2. Prewarm 100 nghĩa là gì?  
   **Đáp án:** Tạo sẵn 100 instance trước khi đo/chơi.

3. maxFree + drop giúp gì?  
   **Đáp án:** Giới hạn RAM khi spike spawn bất thường; thừa để GC thu.

4. Kết quả kỳ vọng alloc POOL vs NEW?  
   **Đáp án:** POOL thấp hơn rõ rệt trên hot path.

5. Port Unity: Destroy mỗi viên đạn có pool không?  
   **Đáp án:** Không — Return/SetActive(false), chỉ Destroy khi trim pool.
