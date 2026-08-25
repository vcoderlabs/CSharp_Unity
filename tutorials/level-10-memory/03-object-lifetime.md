# Chương 3 — Object Lifetime

## 1. Mục tiêu học

- Biết object **sống** khi nào và **chết** (eligible) khi nào
- Hiểu **GC roots**: static, stack local, CPU register, handles…
- Phân biệt **unreachable**, **finalizer**, **Dispose**
- Tránh giữ reference “vô tình” khiến object sống mãi (nền cho chương leak)

## 2. Điều kiện tiên quyết

- Chương 1–2: heap, generations
- Level 2: constructor; destructor/`~` (nếu đã học) — sẽ làm rõ hạn chế

## 3. Khái niệm

### Vòng đời reference type (managed)

```text
1. new T()          → cấp phát + ctor chạy
2. Có ≥ 1 root      → object sống (reachable)
3. Mất hết root     → unreachable = garbage (chưa chắc đã free ngay)
4. GC đánh dấu      → reclaim memory (compact tùy vùng)
5. (Hiếm) finalizer → hàng đợi finalizer, chậm hơn 1 nhịp GC
```

### GC roots (gốc tham chiếu)

Object còn sống nếu có **đường dẫn** từ root:

| Root típ | Ví dụ |
|----------|--------|
| Biến local / tham số đang sống | `Player p` trong method chưa return |
| Static field | `GameManager.Instance`, `static List<>` |
| CPU register | JIT đang giữ reference tạm |
| GCHandle / interop | Pin object cho native |
| Thread stack frames | Mọi frame active |

### Finalizer (`~T`) — không phải Dispose

- Finalizer chạy **không đoán trước**, trên thread riêng
- Object có finalizer phải sống thêm → **đắt**, dễ Gen2
- **Không** dùng finalizer để thay `Dispose` cho logic game thông thường
- .NET hiện đại: ưu tiên `CriticalFinalizerObject` / `SafeHandle` cho unmanaged — bạn hầu như dùng `IDisposable`

### Resurrection (hiếm)

Finalizer có thể gán `static Holder = this` → object “sống lại”. Tránh tuyệt đối trong app thường.

### Unity note sớm

`Destroy(gameObject)` hủy **native** side; C# wrapper có thể vẫn reachable nếu bạn còn field trỏ tới → “zombie” reference. Set `null` / bỏ khỏi list.

## 4. Mô hình tư duy

```text
ROOTS
  static Game.Players ──┐
  local p in Update ────┼──► Player #1   (sống)
                        └──► Player #2   (sống)

  (không ai trỏ) ··········► Enemy #9   (garbage — chờ GC)

Timeline:
  ctor ──► [reachable] ──► last ref gone ──► [unreachable] ──► GC reclaim
                              ▲
                              └── event handler / list / static còn giữ = LEAK (chương 6)
```

## 5. Cú pháp

```csharp
class ResourceProbe
{
    public string Name { get; }
    public ResourceProbe(string name) => Name = name;

    // Tránh dùng trừ khi học — minh họa finalizer
    ~ResourceProbe()
    {
        // Không an toàn: đụng managed object khác có thể đã chết
        // Chỉ log diagnostic trong demo
    }
}

// Ép finalizer chạy (học)
GC.Collect();
GC.WaitForPendingFinalizers();
GC.Collect();

// WeakReference: quan sát không giữ sống
var strong = new Player();
var weak = new WeakReference(strong);
strong = null;
GC.Collect();
Console.WriteLine(weak.IsAlive); // có thể false
```

## 6. Ví dụ

### Cơ bản — Hết scope = mất root local

```csharp
static void ScopeDemo()
{
    var p = new Player { Name = "Temp" };
    Console.WriteLine(p.Name);
} // p hết scope → nếu không static giữ, object eligible
```

### Trung cấp — Static giữ sống

```csharp
static class Registry
{
    public static List<Player> All { get; } = new();
}

static void LeakShape()
{
    var p = new Player { Name = "Pinned" };
    Registry.All.Add(p);
} // local chết nhưng Registry.All vẫn trỏ → p SỐNG
```

### Nâng cao — WeakReference theo dõi lifetime

```csharp
static void WeakDemo()
{
    WeakReference? wr = null;
    void Create()
    {
        var obj = new byte[1024];
        wr = new WeakReference(obj);
    }
    Create();
    GC.Collect();
    GC.WaitForPendingFinalizers();
    Console.WriteLine($"Alive={wr!.IsAlive}");
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Object “không bao giờ chết” | Static / event / collection còn ref | Bỏ ref, unsubscribe |
| Tin vào `~Class()` để đóng file | Finalizer muộn + không chắc | `IDisposable` + `using` |
| `Destroy` rồi vẫn dùng C# ref | Wrapper còn đó | Null-check; clear collections |
| Gọi `GC.Collect` để “destroy ngay” | Không phải API Destroy | Thiết kế lifetime rõ |
| Finalizer đụng Unity API | Sai thread / object chết | Không bao giờ |

## 8. Gỡ lỗi

1. Memory Profiler: object còn sống → xem **shortest path to root**.
2. Đặt `WeakReference` tạm quanh nghi vấn.
3. Log `GetGeneration` + `CollectionCount` sau scene unload.
4. Unity: sau `OnDestroy`, breakpoint xem ai còn giữ `this`.

## 9. Best practices

- Ownership rõ: ai **create** thì ai **release** (Dispose / trả pool / Destroy).
- Tránh finalizer cho gameplay objects.
- Scene unload: clear static caches có chủ đích.
- Dùng `WeakReference` / weak events chỉ khi design cần — không phải default.
- Object pool chủ động quản lý lifetime thay vì phó thác GC (chương 7).

## 10. Bài tập

**Bài 1** — Liệt kê ít nhất 4 loại GC root bằng ví dụ code.

**Bài 2** — Viết demo: object bị giữ bởi `static List`; xóa khỏi list rồi `GC.Collect`; chứng minh bằng `WeakReference`.

**Bài 3** — Giải thích khác nhau: unreachable vs collected.

**Bài 4** — Vì sao finalizer làm object “đắt” hơn object thường?

## 11. Gợi ý

- Bài 1: static, local, field của object sống, GCHandle…
- Bài 2: `Add` → weak còn Alive sau Collect; `Remove` + null strong → có thể !Alive.
- Bài 3: unreachable = logic garbage; collected = bộ nhớ đã reclaim.
- Bài 4: phải xếp hàng finalizer; thêm pass GC.

## 12. Đáp án

**Bài 1** — Ví dụ: (1) `static Player Boss`, (2) local `var e = new Enemy()`, (3) `player.Weapon` field, (4) pinned `GCHandle.Alloc(obj, Pinned)`.

**Bài 2**:

```csharp
static readonly List<object> Hold = new();

var o = new object();
var wr = new WeakReference(o);
Hold.Add(o);
o = null;
GC.Collect();
Console.WriteLine(wr.IsAlive); // True — list còn giữ

Hold.Clear();
GC.Collect();
Console.WriteLine(wr.IsAlive); // thường False
```

**Bài 3** — Unreachable: không còn path từ root — *sẽ* bị thu. Collected: GC đã chạy và tái sử dụng bộ nhớ đó.

**Bài 4** — Object có finalizer không giải phóng ngay trong collect đầu; đưa vào finalizer queue → sống lâu hơn, thường tốn Gen2 hơn, latency kém.

## 13. Đáp án thay thế

Dùng `ConditionalWeakTable` cho metadata không kéo dài lifetime chủ (advanced). Unity: theo dõi bằng Memory Profiler capture hai frame.

## 14. Thử thách

Thiết kế enum `LifetimeState { Alive, Pooled, Destroyed }` cho entity; viết rule chuyển trạng thái và chỗ set reference = null. Không cần Unity — console mô phỏng.

## 15. Ứng dụng thực tế

- Cache với eviction = kiểm soát lifetime
- DI container: singleton vs scoped vs transient
- File/DB: deterministic Dispose > finalizer
- Game: pool + explicit despawn > chờ GC

## 16. Liên hệ Unity

- `OnEnable` / `OnDisable`: subscribe/unsubscribe — gắn lifetime component.
- `OnDestroy`: trả về pool hoặc clear event.
- Đừng dựa finalizer cho `MonoBehaviour`.
- Reference còn trong `List<Enemy>` sau khi Destroy → null/`MissingReferenceException` / leak tùy case.
- Hiểu lifetime = bước đệm bắt buộc trước event leak & pooling.

## 17. Kiểm tra kiến thức

1. Object managed được free ngay khi hết `}` scope?  
   **Đáp án:** Không — chỉ mất root local; GC thu sau.

2. Static field ảnh hưởng lifetime thế nào?  
   **Đáp án:** Là root — giữ object sống suốt AppDomain/process (hoặc đến khi gán null).

3. Finalizer thay `Dispose` được không?  
   **Đáp án:** Không nên — không deterministic, đắt, dễ sai.

4. `WeakReference` giữ object sống không?  
   **Đáp án:** Không (không tính là strong root).

5. Vì sao biết “path to root” quan trọng khi debug leak?  
   **Đáp án:** Cho biết *ai* đang giữ object — chỗ cần cắt reference.
