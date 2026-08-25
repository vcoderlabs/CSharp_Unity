# Chương 5 — Caching & pooling

## 1. Mục tiêu học

- Biết **cache** giảm công việc lặp (CPU/IO), kèm rủi ro stale data
- Biết **object pool** giảm allocation, kèm rủi ro dirty state / leak
- Thiết kế API `Get`/`Return`, size limit, TTL
- Kết nối với project production L19–20

## 2. Điều kiện tiên quyết

- Level 10: object pooling khái niệm
- Level 19 chương 1–4
- Biết `IDisposable` / thread-safety cơ bản

## 3. Khái niệm

### Cache

Lưu kết quả đắt (DB, HTTP, compute) để lần sau lấy nhanh.

| Vấn đề | Hướng xử lý |
|--------|-------------|
| Stale | TTL, invalidation theo event |
| Memory blow-up | Size limit, LRU/LFU |
| Thundering herd | Singleflight / lock theo key |

### Pool

Tái sử dụng instance thay vì `new`/`Dispose` liên tục (buffer, connection logic, game objects).

`ArrayPool<T>`, `ObjectPool<T>` (Microsoft.Extensions.ObjectPool), Unity ObjectPool — cùng ý tưởng.

## 4. Mô hình tư duy

```text
Cache:
  request key → hit? return : compute → store → return
  Evict khi đầy hoặc hết hạn

Pool:
  Get() → object sạch (hoặc Reset)
  dùng xong → Return()
  không Return = “rò” logical (pool cạn → new hoặc block)
```

## 5. Cú pháp

```csharp
// MemoryCache đơn giản (.NET)
using Microsoft.Extensions.Caching.Memory;

IMemoryCache cache = new MemoryCache(new MemoryCacheOptions
{
    SizeLimit = 1024
});

cache.Set("user:1", userDto, new MemoryCacheEntryOptions
{
    AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(5),
    Size = 1
});

if (cache.TryGetValue("user:1", out UserDto? u)) { /* hit */ }

// ArrayPool
var pool = ArrayPool<byte>.Shared;
byte[] buffer = pool.Rent(4096);
try { /* ... */ }
finally { pool.Return(buffer); }
```

## 6. Ví dụ

### Cơ bản — cache dictionary thủ công

```csharp
sealed class SimpleCache<T>
{
    private readonly Dictionary<string, (T Value, DateTime Exp)> _map = new();
    private readonly TimeSpan _ttl;

    public SimpleCache(TimeSpan ttl) => _ttl = ttl;

    public bool TryGet(string key, out T? value)
    {
        if (_map.TryGetValue(key, out var e) && e.Exp > DateTime.UtcNow)
        {
            value = e.Value;
            return true;
        }
        value = default;
        return false;
    }

    public void Set(string key, T value)
        => _map[key] = (value, DateTime.UtcNow + _ttl);
}
```

(Chưa thread-safe / chưa size limit — bài tập mở rộng.)

### Trung cấp — pool có Reset

```csharp
sealed class BufferPool
{
    private readonly Stack<byte[]> _stack = new();
    private readonly int _size;

    public BufferPool(int size, int prewarm = 0)
    {
        _size = size;
        for (int i = 0; i < prewarm; i++) _stack.Push(new byte[size]);
    }

    public byte[] Rent() => _stack.Count > 0 ? _stack.Pop() : new byte[_size];

    public void Return(byte[] buffer)
    {
        Array.Clear(buffer); // nếu cần bảo mật / tránh data leak
        if (_stack.Count < 64) _stack.Push(buffer);
    }
}
```

### Nâng cao — cache + factory

```csharp
async Task<T> GetOrCreateAsync<T>(string key, Func<Task<T>> factory)
{
    if (_cache.TryGetValue(key, out T? hit) && hit is not null)
        return hit;
    // TODO: lock theo key để tránh stampede
    T value = await factory();
    _cache.Set(key, value);
    return value;
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Bug “data cũ” | Không invalidate | Event hết hạn / version key |
| Pool trả object bẩn | Quên Reset | `IResettable` / clear bắt buộc |
| Double Return | Bug ownership | Debug bit / throw nếu detect |
| Cache không giới hạn | OOM | SizeLimit + eviction |
| Pool trong multi-thread không sync | Race | ConcurrentBag / channel |

## 8. Gỡ lỗi

1. Log hit ratio: `hits / (hits+misses)`.
2. Pool: đếm `rented - returned` theo thời gian — lệch = leak.
3. Unit test: Get → mutate → Return → Get lại phải “sạch”.
4. Load test sau khi bật cache: đúng kỳ vọng miss lúc đầu, ổn định sau.

## 9. Best practices

- Cache **kết quả**, không cache object mutable đang share lung tung nếu không bất biến.
- Pool chỉ khi allocation đã đo là vấn đề — hoặc pattern Unity bắt buộc.
- Document ownership: ai Return?
- Prefer `ArrayPool` / thư viện mature hơn tự viết khi production.
- Metrics: hit rate, pool depth, alloc/s trước-sau.

## 10. Bài tập

**Bài 1** — Thêm `SizeLimit` + eviction FIFO đơn giản cho `SimpleCache`.

**Bài 2** — Pool cho class `StringBuilder`: Get, clear khi Return.

**Bài 3** — Dùng `IMemoryCache` cache kết quả “tính Fibonacci chậm” (cố ý) theo n.

**Bài 4** — Viết test: Return thiếu → sau N lần Rent, số instance sống tăng (giả lập leak detector).

## 11. Gợi ý

- Bài 1: `Queue<string>` thứ tự key + Dictionary.
- Bài 2: `sb.Clear()` trước khi push stack.
- Bài 3: `GetOrCreate`.
- Bài 4: static counter trong class pooled.

## 12. Đáp án

**Bài 2**:

```csharp
sealed class SbPool
{
    private readonly Stack<StringBuilder> _stack = new();

    public StringBuilder Get() => _stack.Count > 0 ? _stack.Pop() : new StringBuilder();

    public void Return(StringBuilder sb)
    {
        sb.Clear();
        if (_stack.Count < 32) _stack.Push(sb);
    }
}
```

**Bài 3**:

```csharp
return cache.GetOrCreate(n, e =>
{
    e.AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10);
    return SlowFib(n);
})!;
```

**Bài 1** — Khi Set vượt limit, dequeue key cũ và `Remove`.

**Bài 4** — `Interlocked` tăng lúc tạo mới, giảm không bao giờ nếu không Return — assert trong test dài.

## 13. Đáp án thay thế

`Microsoft.Extensions.ObjectPool` + `DefaultObjectPoolProvider`. Redis cho distributed cache (L20 config). Hybrid: L1 memory + L2 Redis.

## 14. Thử thách

Trong project production L19–20: chọn 1 endpoint/command đắt → thêm cache có TTL + metric hit rate; benchmark trước/sau.

## 15. Ứng dụng thực tế

- CDN / HTTP cache headers
- ORM second-level cache (cẩn thận)
- DB connection pooling (driver đã làm)
- Game: pool đạn, cache static data ScriptableObject

## 16. Liên hệ Unity

- `ObjectPool<T>` (UnityEngine.Pool)
- Không pool mọi thứ — UI một lần tạo có thể không cần
- Addressables + pool instance
- Cache `WaitForSeconds` — anti-pattern cũ; dùng cache reference component

## 17. Kiểm tra kiến thức

1. Cache giải quyết gì?  
   **Đáp án:** Tránh lặp lại công việc/IO đắt bằng lưu kết quả.

2. Rủi ro chính của cache?  
   **Đáp án:** Dữ liệu stale và phình bộ nhớ.

3. Pool giải quyết gì?  
   **Đáp án:** Giảm allocation / chi phí tạo-hủy object.

4. Vì sao cần Reset khi Return?  
   **Đáp án:** Tránh lần Get sau nhận state bẩn → bug khó tái hiện.

5. Khi nào không nên pool?  
   **Đáp án:** Khi chưa đo thấy alloc là vấn đề, hoặc object ít tạo / ownership phức tạp hơn lợi ích.
