# Chương 7 — Object Pooling trong Unity

## 1. Mục tiêu học

- Implement pool với `UnityEngine.Pool.ObjectPool<T>`
- Vòng đời Get/Release + reset state
- Pool đạn / VFX cơ bản
- Đo trước-sau trên Profiler

## 2. Điều kiện tiên quyết

- Chương 6 GC
- Level 10 pooling khái niệm
- Prefab Instantiate

## 3. Khái niệm

**Object pool:** giữ instance đã tạo, tái sử dụng thay vì `Instantiate`/`Destroy` liên tục (đắt + GC).

```text
Get → enable + Reset
dùng xong → Release → disable + về kho
```

Warmup: prewarm N instance lúc load level.

## 4. Mô hình tư duy

```text
BulletPool
  collection check (debug)
  create: Instantiate(prefab)
  get: SetActive true
  release: SetActive false
  destroy: Destroy(go) khi clear pool
```

## 5. Cú pháp

```csharp
using UnityEngine.Pool;

public class BulletPool : MonoBehaviour
{
    [SerializeField] Bullet bulletPrefab;
    ObjectPool<Bullet> _pool;

    void Awake()
    {
        _pool = new ObjectPool<Bullet>(
            createFunc: () => Instantiate(bulletPrefab),
            actionOnGet: b => b.gameObject.SetActive(true),
            actionOnRelease: b => b.gameObject.SetActive(false),
            actionOnDestroy: b => Destroy(b.gameObject),
            collectionCheck: true,
            defaultCapacity: 32,
            maxSize: 128);
    }

    public Bullet Get() => _pool.Get();
    public void Release(Bullet b) => _pool.Release(b);
}
```

```csharp
public class Bullet : MonoBehaviour
{
    BulletPool _pool;
    public void Init(BulletPool pool) => _pool = pool;
    public void Return() => _pool.Release(this);
}
```

## 6. Ví dụ

### Cơ bản — spawn theo space

`Get`, set position/velocity, timer gọi `Return`.

### Trung cấp — particle pool

Release khi `OnParticleSystemStopped`.

### Nâng cao — nhiều prefab

`Dictionary<id, ObjectPool<>>` hoặc pool riêng từng loại.

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Object bẩn | Quên reset velocity/trail | Reset OnGet |
| Double release | Bug | collectionCheck |
| Pool không warmup | Spike lúc đầu | prewarm |
| Parent hierarchy loạn | Không set parent | Pool root transform |

## 8. Gỡ lỗi

1. Đếm active children dưới pool root.  
2. collectionCheck = true lúc dev.  
3. Profiler Instantiate count ↓.  
4. Log khi createFunc gọi — quá nhiều = max thiếu / leak không return.

## 9. Best practices

- Reset đầy đủ: Rigidbody velocity, trail, collider enable.  
- Giới hạn maxSize.  
- Không Destroy object thuộc pool ở chỗ khác.  
- API rõ `Return to pool` ownership.  
- Đo trên device thấp.

## 10. Bài tập

**Bài 1** — Pool đạn bắn 10 viên/giây trong 30s; so GC với không pool.  
**Bài 2** — Prewarm 50.  
**Bài 3** — Reset trail renderer khi Get.  
**Bài 4** — Pool VFX hit.

## 11. Gợi ý

- Bài 1: hai scene/mode.  
- Bài 3: `trail.Clear()`.

## 12. Đáp án

Dùng `ObjectPool` như cú pháp; Bullet `fixedTime` coroutine/awaitable rồi `Return()`.

## 13. Đáp án thay thế

Tự Stack\<T\> pool. Addressables.InstantiateAsync + pool (phức tạp hơn — ch.8).

## 14. Thử thách

Pool generic `ComponentPool<T>` dùng chung nhiều prefab type.

## 15. Ứng dụng thực tế

Đạn, hit spark, damage text, enemy nhỏ wave — xương sống combat MMORPG.

## 16. Liên hệ C# thuần

Giống `ObjectPool<T>` MS.Extensions — Unity gắn GameObject active flag.

## 17. Kiểm tra kiến thức

1. Pool giảm gì?  
   **Đáp án:** Chi phí tạo/hủy + GC từ Destroy liên tục.

2. actionOnGet thường làm gì?  
   **Đáp án:** Enable và reset state.

3. collectionCheck để làm gì?  
   **Đáp án:** Phát hiện double release / lỗi ownership lúc dev.

4. Prewarm là gì?  
   **Đáp án:** Tạo sẵn N object trước khi gameplay nóng.

5. Quên Return dẫn tới?  
   **Đáp án:** Pool tạo mãi / cạn logic; leak instance active.
