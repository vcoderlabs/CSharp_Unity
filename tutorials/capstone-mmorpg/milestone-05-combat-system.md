# Milestone 05 — Combat System

## Requirements

- Combat dùng **Strategy** cho cách tính damage / skill behavior
- **Command** cho hành động (fire projectile, melee swing) có thể enqueue
- **Object pooling** cho đạn và VFX hit (bắt buộc đo được)
- Tích hợp Entity Health (M02) + kill bus cho Quest (M04)
- Player tấn công + ≥1 enemy AI đơn giản (chase/attack)
- Không allocate bất thường khi spam attack (mục tiêu: ổn định GC Alloc)

**Không yêu cầu:** combo animation AAA, rollback netcode.

---

## Architecture

```text
IAttackStrategy
  MeleeStrategy / RangedStrategy
  Compute(AttackContext) → DamageInfo

ICombatCommand
  Execute()
  FireProjectileCommand / MeleeSwingCommand

CombatService
  Enqueue(ICombatCommand) hoặc Execute ngay
  dùng IAttackStrategy theo weapon

BulletPool / VfxPool
Bullet.OnHit → HealthSystem.ApplyDamage → Release pool → spawn Vfx pooled

EnemyBrain (MB mỏng / system)
  target player → Move → AttackCommand theo cooldown
```

```text
Input ──► Command ──► Strategy (damage) ──► HealthSystem
                         │
                         └── Pool projectiles/VFX
```

---

## Tasks

1. `DamageInfo` (amount, source, crit flag đơn giản).  
2. `IAttackStrategy` + 2 strategy.  
3. Command fire + melee.  
4. Pool đạn + VFX; prewarm.  
5. Enemy AI tối thiểu.  
6. Wire kill event → QuestService.  
7. Profiler capture 10s combat — lưu vào `PERF_M05.md`.

---

## Expected result

- Đánh nhau được; enemy chết; quest kill tiến triển.  
- Đạn/VFX không `Destroy` vĩnh viễn mỗi hit (Return pool).  
- Đổi strategy (melee/ranged) không sửa nửa codebase (OCP).  
- Có số liệu Profiler.

---

## Exercises

**E1** — Crit 10% trong strategy (IRng từ M01).  
**E2** — Command queue 1 frame delay (telegraph).  
**E3** — AoE command (overlap NonAlloc).  
**E4** — Damage popup text pooled.

---

## Hints

- Reset Rigidbody/trail khi Get từ pool.  
- Layer matrix đạn vs enemy.  
- Strategy inject vào weapon definition SO.  
- Tránh LINQ trong hit callback.

---

## Solution outline

```csharp
public interface IAttackStrategy
{
    DamageInfo Build(in AttackContext ctx);
}

public sealed class RangedStrategy : IAttackStrategy
{
    public DamageInfo Build(in AttackContext ctx)
        => new(ctx.BaseDamage, ctx.Source, false);
}

public sealed class FireProjectileCommand : ICombatCommand
{
    public void Execute()
    {
        var b = _pool.Get();
        b.Launch(_origin, _dir, _damage, _source);
    }
}
```

Hit:

```text
HealthSystem.ApplyDamage(target, dmg)
VfxPool.PlayAt(pos)
Bullet.Return()
if dead → EntityDied bus
```

---

## Code review checklist

- [ ] Strategy/Command thực sự tách — không chỉ tên class cho vui  
- [ ] Pool collectionCheck bật lúc dev  
- [ ] Không double-Return đạn  
- [ ] Kill bus nối quest  
- [ ] PERF_M05.md có trước/sau hoặc có/không pool  
- [ ] NonAlloc physics nếu AoE  
- [ ] Enemy không `Find` player mỗi frame (cache/inject)  
- [ ] ARCHITECTURE combat section  
