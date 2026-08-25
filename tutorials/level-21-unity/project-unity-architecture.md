# Project — Unity Architecture Mini Game

Mini game tổng hợp Level 21: **object pooling + event system + DI**, MonoBehaviour mỏng.

**Thời lượng:** ~8–10 giờ.

---

## 1. Mục tiêu học

- Thiết kế architecture trước khi spam script
- Bắn đạn có pool; hit dùng event/channel
- DI wire `AttackService`, `ScoreService`, pool
- Đo GC: có pool vs không (ghi chú ngắn)

## 2. Điều kiện tiên quyết

- Xong chương 1–9 L21 (đặc biệt 5, 7, 9)
- Unity project riêng `L21_MiniArch`

## 3. Khái niệm sản phẩm

**Tên gợi ý:** *Arena Slice*

- Player WASD + chuột bắn  
- Enemy đơn giản đi tới player  
- Đụng đạn → enemy “chết” (pool hoặc deactivate) + cộng điểm  
- UI score  
- Không cần animation đẹp  

## 4. Mô hình tư duy / Architecture

```text
GameLifetimeScope
├── ScoreService : IScoreService
├── BulletPool (MB hoặc service)
├── EnemySpawner
├── GameRunner (EntryPoint)
└── VoidEventChannel / IntEventChannel (SO)  — EnemyKilled

PlayerInput (MB) → AttackService.Fire()
Bullet (MB) OnTrigger → Raise EnemyKilled / Return pool
HudView (MB) ← subscribe ScoreService / channel
```

```text
[Scene]
  LifetimeScope
  Player
  Spawner
  PoolRoot
  Canvas/HUD
  EventChannels (refs)
```

## 5. Cú pháp — skeleton AttackService

```csharp
public sealed class AttackService
{
    readonly BulletPool _pool;
    public AttackService(BulletPool pool) => _pool = pool;

    public void Fire(Vector3 origin, Vector3 direction)
    {
        var b = _pool.Get();
        b.Launch(origin, direction.normalized);
    }
}
```

## 6. Ví dụ phạm vi MVP

### Phải có

1. DI container chạy  
2. Pool đạn Get/Release  
3. Ít nhất 1 C# event hoặc SO channel khi kill  
4. Score tăng + UI  
5. Enemy spawn định kỳ  
6. Ghi `PERF_NOTES.md`: GC snapshot trước/sau pool  

### Không bắt buộc

Addressables remote, multiplayer, inventory đầy đủ, ECS DOTS.

## 7. Lỗi thường gặp

| Hiện tượng | Cách xử lý |
|------------|------------|
| Circular inject | Tách interface; tránh MB inject lẫn nhau vòng |
| Đạn không return | Timer + OnHit đều Release một lần (flag) |
| Score UI không cập nhật | Subscribe OnEnable |
| Scope không cover HUD | Hierarchy dưới LifetimeScope |

## 8. Gỡ lỗi

1. Validate DI.  
2. collectionCheck pool.  
3. Profiler GC Alloc khi spam fire.  
4. Log Raise channel.

## 9. Best practices

- Prefab Enemy/Bullet sạch.  
- Magic numbers → SO `ArenaConfig`.  
- Input tách service.  
- README chơi được: phím + scene name.

## 10. Bài tập (deliverables)

**Bài 1** — Scene + DI + player di chuyển.  
**Bài 2** — Pool bắn.  
**Bài 3** — Enemy + hit + score event.  
**Bài 4** — PERF_NOTES với 2 ảnh Profiler (hoặc mô tả số liệu).

## 11. Gợi ý

- Enemy đơn giản: `Transform.position = MoveTowards`.  
- Hit: `OnTriggerEnter` layer matrix.  
- ScoreService thuần C# `event Action<int>`.

## 12. Đáp án — outline hoàn chỉnh

1. `ArenaConfig` SO: moveSpeed, fireRate, enemyHp  
2. `IScoreService` / `ScoreService`  
3. `BulletPool` + `Bullet.Launch` + auto return sau 2s  
4. `AttackService` + `PlayerInput` cooldown  
5. `Enemy` HP; 0 → raise killed, deactivate, spawner reuse hoặc Destroy tạm  
6. `HudView` bind score  
7. `GameLifetimeScope` đăng ký tất cả  
8. Optional: `EnemyKilledChannel` SO  

## 13. Đáp án thay thế

Zenject thay VContainer — cùng cấu trúc. Không DI package: composition root MB thủ công `new` services (vẫn tách class).

## 14. Thử thách

- Enemy cũng pool  
- Addressables load VFX hit  
- Pause với `CancellationToken`  

## 15. Ứng dụng thực tế

Đây là “hạt giống” combat Capstone MMORPG — giữ folder structure khi nâng cấp.

## 16. Liên hệ lộ trình

L19 đo GC · L20 README/git · L7 events · L10 pool · L18 layers — tất cả hội tụ.

## 17. Kiểm tra kiến thức / DoD

1. MB player có < ~80 dòng logic nghiệp vụ?  
2. Đạn luôn Return về pool?  
3. Kill cập nhật UI qua event/service (không Find HUD mỗi lần)?  
4. Có PERF_NOTES?  
5. Người khác mở scene Play được theo README?

**Done = 5× Có.** Sang [Capstone](../capstone-mmorpg/).
