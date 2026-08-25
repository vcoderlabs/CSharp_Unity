# Chương 5 — Prototype (+ Object Pool reference)

## 1. Mục tiêu học

- Clone object tốn kém tạo lại từ đầu
- Shallow vs deep copy trong C#
- Liên hệ **Object Pool**: tái sử dụng instance thay vì clone/new liên tục
- Unity: prefab = prototype; pool cho đạn/VFX

## 2. Điều kiện tiên quyết

- Reference vs value (L3)
- `ICloneable` (hiểu hạn chế)
- Factory cơ bản

## 3. Khái niệm

**Prototype:** tạo object mới bằng **sao chép** prototype đã cấu hình.  
**Object Pool:** giữ sẵn instance, `Get`/`Release` — giảm alloc/GC (quan trọng Unity). Pool thường *đi cùng* Factory, không phải Prototype thuần, nhưng cùng mục tiêu “tránh tạo đắt”.

## 4. Mô hình tư duy

```text
Prototype registry: "orc_archer" → template
Spawn = template.Clone() rồi tweak

Pool: Get() → reset state → dùng → Release()
```

## 5. Cú pháp

```csharp
public interface IProto<T>
{
    T Clone();
}

public sealed class EnemyProto : IProto<EnemyProto>
{
    public string Id { get; init; } = "";
    public int Hp { get; set; }
    public EnemyProto Clone() => new() { Id = Id, Hp = Hp };
}
```

## 6. Ví dụ

### Cơ bản — Prototype

```csharp
var proto = new EnemyProto { Id = "orc", Hp = 50 };
var a = proto.Clone(); a.Hp = 40;
var b = proto.Clone();
```

### Trung cấp — Registry

```csharp
public sealed class ProtoCatalog
{
    private readonly Dictionary<string, EnemyProto> _map = new();
    public void Register(string id, EnemyProto p) => _map[id] = p;
    public EnemyProto Spawn(string id) => _map[id].Clone();
}
```

### Nâng cao — Pool (Unity-critical)

```csharp
public sealed class Pool<T> where T : class, new()
{
    private readonly Stack<T> _free = new();
    public T Get() => _free.Count > 0 ? _free.Pop() : new T();
    public void Release(T item) => _free.Push(item);
}

// Unity: ObjectPool<T> (UnityEngine.Pool) + prefab Instantiate/Disable
```

## 7. Lỗi thường gặp

| Hiện tượng | Cách xử lý |
|------------|------------|
| Shallow copy chia sẻ List | Deep copy / clone collection |
| Pool trả object bẩn | Reset bắt buộc khi Get/Release |
| `ICloneable` không generic | Tự interface `Clone()` typed |

## 8. Gỡ lỗi

So sánh reference sau clone — phải khác instance. Pool: log Count free; leak nếu quên Release.

## 9. Best practices

- Document shallow/deep.  
- Pool: clear event subscription khi release.  
- Prefab Unity = prototype có sẵn.

## 10. Bài tập

**Bài 1** — Clone `Spell` có `List<Effect>` deep.  
**Bài 2** — Catalog 2 prototype spawn.  
**Bài 3** — Pool `Bullet` reset `Position`.  
**Bài 4** — Giải thích vì sao Instantiate mỗi frame xấu (GC).

## 11. Gợi ý

Deep: `Effects = Effects.Select(e => e.Clone()).ToList()`.

## 12. Đáp án

```csharp
public sealed class Bullet
{
    public Vector3 Pos;
    public void Reset() => Pos = default;
}

public sealed class BulletPool
{
    private readonly Stack<Bullet> _free = new();
    public Bullet Get()
    {
        var b = _free.Count > 0 ? _free.Pop() : new Bullet();
        b.Reset();
        return b;
    }
    public void Release(Bullet b) { b.Reset(); _free.Push(b); }
}
```

## 13. Đáp án thay thế

`record` với `with` cho copy bất biến — không thay pool runtime nóng.

## 14. Thử thách

Unity-style: pool GameObject setActive true/false; đo alloc Profiler (L19).

## 15. Ứng dụng thực tế

- Document template clone  
- Game entity archetypes

## 16. Liên hệ Unity

- Prefab = Prototype  
- **Object Pool** cho projectile/VFX/combat (Capstone Milestone 05)  
- Addressables + pool nâng cao

## 17. Kiểm tra kiến thức

1. Prototype tạo object bằng? **Clone template.**  
2. Shallow copy rủi ro? **Chia sẻ reference lồng nhau.**  
3. Pool khác Prototype? **Tái sử dụng instance vs sao chép cấu hình.**  
4. Vì sao pool quan trọng Unity? **Giảm GC spike.**  
5. Quên Release gây? **Leak / alloc tăng / pool cạn.**
