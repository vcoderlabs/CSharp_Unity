# Chương 7 — Object Pooling & Unity GC Spikes

## 1. Mục tiêu học

- Hiểu **object pooling**: reuse thay vì `new` / GC liên tục
- Implement pool generic tối giản (`Rent` / `Return`)
- Biết `ArrayPool<T>`, `ObjectPool<T>` (.NET) và hướng Unity
- **Liên hệ trực tiếp Unity GC spikes**: pool = giảm Gen0 alloc trong combat/VFX

## 2. Điều kiện tiên quyết

- Chương 1–2: allocation → Gen0 → GC spike
- Level 5: generics (`Pool<T>`)
- Chương 6: reset state / unsubscribe khi Return

## 3. Khái niệm

### Vấn đề

```text
Mỗi viên đạn: new Bullet()
60 FPS × 30 bullet/s × nhiều skill
→ Gen0 đầy liên tục → GC.Collect → SPIKE → giật
```

### Giải pháp pooling

```text
Prewarm: tạo N bullet lúc load
Shoot:  Rent() từ pool (SetActive true / reset state)
Despawn: Return() về pool (SetActive false) — KHÔNG Destroy/GC
```

### Thuật ngữ

| Thuật ngữ | Nghĩa |
|-----------|--------|
| Rent / Get | Lấy instance từ pool |
| Return / Release | Trả lại pool |
| Prewarm / Prefill | Tạo sẵn trước gameplay |
| Capacity | Số lượng tối đa / ban đầu |
| Reset | Xóa state cũ trước khi reuse |

### BCL hỗ trợ

- `System.Buffers.ArrayPool<T>.Shared` — mảng
- `Microsoft.Extensions.ObjectPool` / `ObjectPool<T>` (NuGet) — object
- Unity: `UnityEngine.Pool.ObjectPool<T>`, `GenericPool<T>`, v.v.

### Không pool mọi thứ

Pool khi: **alloc thường xuyên + cùng kiểu + reset được**.  
Không pool: object tạo 1 lần/scene, state cực phức tạp khó reset, hoặc pool management đắt hơn lợi ích.

## 4. Mô hình tư duy

```text
KHÔNG POOL (mỗi shot):
  new → dùng → quên ref → Gen0 → GC → SPIKE

CÓ POOL:
  [Idle]──Rent──►[Active in world]
     ▲                │
     └────Return──────┘

Heap ổn định: N object sống cả trận (Gen2 OK)
Gameplay: ~0 alloc/frame từ bullet spawn
```

```text
Unity frame budget (ví dụ):
  16.6ms @ 60FPS
  GC pause 8ms  →  MẤT NỬA FRAME → giật rõ
  Pool tránh pause đó từ spawn spam
```

## 5. Cú pháp

```csharp
sealed class ObjectPool<T> where T : class, new()
{
    private readonly Stack<T> _free = new();
    private readonly Action<T>? _reset;

    public ObjectPool(int prewarm = 0, Action<T>? reset = null)
    {
        _reset = reset;
        for (int i = 0; i < prewarm; i++)
            _free.Push(new T());
    }

    public T Rent() => _free.Count > 0 ? _free.Pop() : new T();

    public void Return(T item)
    {
        _reset?.Invoke(item);
        _free.Push(item);
    }
}

// ArrayPool
var buffer = ArrayPool<byte>.Shared.Rent(1024);
try { /* dùng buffer[0..length] */ }
finally { ArrayPool<byte>.Shared.Return(buffer); }
```

## 6. Ví dụ

### Cơ bản — Pool `StringBuilder`

```csharp
var pool = new ObjectPool<StringBuilder>(
    prewarm: 4,
    reset: sb => sb.Clear());

var sb = pool.Rent();
sb.Append("Damage ");
sb.Append(42);
Console.WriteLine(sb);
pool.Return(sb);
```

### Trung cấp — Pool đạn console

```csharp
sealed class Bullet
{
    public int Id;
    public float X, Y;
    public bool Active;
}

var bullets = new ObjectPool<Bullet>(32, b =>
{
    b.Active = false;
    b.X = b.Y = 0;
});

Bullet Shoot(float x, float y)
{
    var b = bullets.Rent();
    b.Active = true;
    b.X = x; b.Y = y;
    return b;
}

void Despawn(Bullet b) => bullets.Return(b);
```

### Nâng cao — Đo alloc trước/sau

```csharp
static void BenchmarkPool(bool usePool, int ops)
{
    ObjectPool<Bullet>? pool = usePool ? new(256, b => b.Active = false) : null;
    long before = GC.GetAllocatedBytesForCurrentThread();
    for (int i = 0; i < ops; i++)
    {
        Bullet b = usePool ? pool!.Rent() : new Bullet();
        b.Active = true;
        if (usePool) pool!.Return(b);
    }
    long after = GC.GetAllocatedBytesForCurrentThread();
    Console.WriteLine($"{(usePool ? "pool" : "new")}: {after - before} bytes");
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Object “ma” còn state cũ | Quên reset | Reset trong Return |
| Double Return | Logic despawn 2 lần | Guard / HashSet rented |
| Pool vẫn alloc | Expand không prewarm đủ | Prewarm đúng peak |
| Leak qua event trên pooled object | Còn subscription | Unsubscribe trong reset |
| Pool quá lớn | Giữ RAM vô ích | Cap max + trim |
| Thread-safety | Nhiều thread Rent | Concurrent bag / lock / per-thread |

## 8. Gỡ lỗi

1. So sánh `GetAllocatedBytes` / Unity **GC Alloc** có/không pool.
2. Log `Rent` count vs `new` fallback count — fallback cao = prewarm yếu.
3. Assert: object Active không nằm trong `_free`.
4. Unity Profiler: spawn VFX trước/sau pool.

## 9. Best practices

- Prewarm lúc **loading**, không lúc combat.
- Reset **mọi** state: transform, velocity, HP, events, trail.
- Giới hạn max size; policy khi đầy (reject / expand / destroy excess).
- Một pool một prefab/type.
- Đo peak concurrent (max bullets on screen) để chọn capacity.
- .NET: ưu tiên `ArrayPool` cho buffer tạm.

## 10. Bài tập

**Bài 1** — Implement `ObjectPool<T> where T : class, new()` với `Rent`/`Return`/`CountFree`.

**Bài 2** — Thêm `prewarm` và `Action<T> reset`.

**Bài 3** — Demo `StringBuilder` pool vs `new StringBuilder` mỗi lần — in bytes allocated.

**Bài 4** — Viết 1 đoạn giải thích (5–8 câu) gửi “đồng đội Unity”: pool giảm GC spike thế nào.

## 11. Gợi ý

- Bài 1: `Stack<T>` hoặc `Queue<T>`.
- Bài 2: ctor loop `Push(new T())`; Return gọi reset.
- Bài 3: dùng `GC.GetAllocatedBytesForCurrentThread`.
- Bài 4: nhắc Gen0, pause, frame budget, prewarm.

## 12. Đáp án

**Bài 1–2** — Xem khung mục 5 (đủ yêu cầu).

**Bài 3**:

```csharp
long N(int n)
{
    long a = GC.GetAllocatedBytesForCurrentThread();
    for (int i = 0; i < n; i++)
    {
        var sb = new StringBuilder();
        sb.Append(i);
    }
    return GC.GetAllocatedBytesForCurrentThread() - a;
}

long P(int n)
{
    var pool = new ObjectPool<StringBuilder>(32, sb => sb.Clear());
    long a = GC.GetAllocatedBytesForCurrentThread();
    for (int i = 0; i < n; i++)
    {
        var sb = pool.Rent();
        sb.Append(i);
        pool.Return(sb);
    }
    return GC.GetAllocatedBytesForCurrentThread() - a;
}
```

**Bài 4** (ý chính): Spawn liên tục `new` làm đầy Gen0; GC dừng frame → spike. Pool tái sử dụng instance đã tạo lúc load; hot path gần như không alloc; GC ít chạy giữa combat → mượt hơn.

## 13. Đáp án thay thế

Dùng `ConcurrentBag<T>` nếu đa luồng. Unity: `new ObjectPool<T>(createFunc, actionOnGet, actionOnRelease, actionOnDestroy, ...)`.

## 14. Thử thách

Thêm `maxSize`: khi Return mà free count ≥ max thì **không** giữ (để GC thu). So sánh RAM vs spike.

## 15. Ứng dụng thực tế

- Network buffer pooling
- DB command pooling
- Particle / projectile / damage text
- Temporary `List<T>` / `HashSet` trong systems (rent list)

## 16. Liên hệ Unity

> **Đây là chương quan trọng nhất Level 10 cho MMORPG.**

- Bullet, hit VFX, damage popup, enemy non-boss, audio source one-shot → **pool**.
- `Instantiate`/`Destroy` mỗi frame = alloc + native cost + GC — đổi thành `SetActive` + pool.
- Unity `ObjectPool<T>` / Addressables pooled release — học concept ở đây rồi map API.
- Prewarm trong loading screen dungeon.
- Kết hợp chương 6: Return phải clear events/`UnityEvent`.
- Incremental GC **không** thay thế pooling — chỉ giảm độ cao spike khi vẫn còn alloc.
- Checklist combat scene: Profiler **GC Alloc ≈ 0** trên script bắn đạn.

## 17. Kiểm tra kiến thức

1. Pool giải quyết vấn đề gì?  
   **Đáp án:** Giảm allocation lặp lại → giảm GC pressure/spike.

2. Prewarm nên làm khi nào?  
   **Đáp án:** Lúc load / idle — không giữa combat nóng.

3. Return mà không reset thì sao?  
   **Đáp án:** Object “ma” mang state cũ (HP, vị trí, event…).

4. `ArrayPool` dùng cho gì?  
   **Đáp án:** Thuê/trả mảng tạm, tránh LOH/Gen alloc lặp.

5. Incremental GC có đủ thay pool không?  
   **Đáp án:** Không.
