# Chương 14 — State

## 1. Mục tiêu học

- Behavior phụ thuộc **trạng thái nội tại**; chuyển state có kiểm soát
- Thay `enum` + `switch` khổng lồ
- Unity: character anim/AI, cửa, quest step

## 2. Điều kiện tiên quyết

- Strategy (tương tự cấu trúc, khác intent)
- OOP polymorphism

## 3. Khái niệm

**State:** mỗi trạng thái là class/`IState` với `Enter`/`Exit`/`Tick`/`Handle(input)`. Context ủy thác cho state hiện tại; state có thể request chuyển.

## 4. Mô hình tư duy

```text
Character
  state: Idle → [Run] → Moving → [Jump] → Airborne → [Land] → Idle
Mỗi state chỉ biết transition hợp lệ
```

## 5. Cú pháp

```csharp
public interface IPlayerState
{
    void Enter(Player ctx);
    void Exit(Player ctx);
    void Tick(Player ctx, float dt);
    void Handle(Player ctx, Input x);
}
```

## 6. Ví dụ

### Cơ bản — cửa

```csharp
public interface IDoorState
{
    void Open(Door d);
    void Close(Door d);
}

public sealed class ClosedState : IDoorState
{
    public void Open(Door d) => d.SetState(new OpenState());
    public void Close(Door d) { /* already */ }
}
```

### Trung cấp — player

```csharp
public sealed class Player
{
    private IPlayerState _state = new IdleState();
    public void SetState(IPlayerState s) { _state.Exit(this); _state = s; _state.Enter(this); }
    public void Tick(float dt) => _state.Tick(this, dt);
}
```

### Nâng cao / Unity MMORPG

Quest: `NotStarted` → `Active` → `TurnIn` → `Completed` — illegal transition throw/log.  
Animator Unity ≈ state machine — logic gameplay nên mirror bằng State C# test được.  
AI: Idle/Chase/Attack/Flee (kết hợp Strategy cho đường đi).

## 7. Lỗi thường gặp

| Hiện tượng | Cách xử lý |
|------------|------------|
| State biết quá nhiều Player fields | Truyền API hẹp / interface ctx |
| Quên Exit unsubscribe | Enter/Exit đối xứng |
| God enum switch song song | Một nguồn sự thật — class state |

## 8. Gỡ lỗi

Log transition `A → B`. Assert illegal transition trong test. Visualize graph.

## 9. Best practices

- Explicit transition table hoặc method trên state.  
- Enter/Exit cho side-effect (anim, collider).  
- Data-driven state cho designer (ScriptableObject) khi số state lớn.

## 10. Bài tập

**Bài 1** — TCP-like Closed/Listen/Open stub.  
**Bài 2** — Enemy Idle/Chase/Attack.  
**Bài 3** — Cấm Attack←Idle trực tiếp.  
**Bài 4** — So Strategy vs State bằng 5 câu.

## 11. Gợi ý

Bài 3: `Idle.Handle(Attack)` ignore hoặc chuyển Chase trước.

## 12. Đáp án

```csharp
public sealed class IdleState : IEnemyState
{
    public void Handle(Enemy e, Stimulus s)
    {
        if (s == Stimulus.SeePlayer) e.SetState(new ChaseState());
    }
}
```

## 13. Đáp án thay thế

Enum + transition dictionary — State “nhẹ” khi ít hành vi per state.

## 14. Thử thách

Skill cast: Windup → Active → Recovery; interrupt bởi stun.

## 15. Ứng dụng thực tế

- Workflow documents  
- Game/UI wizards  
- Connection lifecycles

## 16. Liên hệ Unity

- **Critical** AI, anim sync, quest  
- Animator Controller vs code state — đừng duplicate mâu thuẫn  
- Capstone Milestone 04 Quest System

## 17. Kiểm tra kiến thức

1. State thay đổi gì? **Behavior theo trạng thái.**  
2. Enter/Exit để? **Setup/teardown khi chuyển.**  
3. Khác Strategy? **Transition + lifecycle; Strategy swap thuật toán.**  
4. Switch enum xấu khi? **Phình + illegal transition khó kiểm soát.**  
5. Quest dùng State vì? **Bước hợp lệ có thứ tự.**
