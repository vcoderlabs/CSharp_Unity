# Milestone 01 — Core Architecture

## Requirements

- Chọn và cài **một** DI container (VContainer hoặc Zenject)
- Có **composition root** (LifetimeScope / Installer) rõ ràng
- Tách **service layer** thuần C# khỏi MonoBehaviour mỏng
- Có **game loop / tick** tập trung (forward từ một `GameRunner` hoặc tương đương)
- Logging tối thiểu (Unity logger wrapper có level)
- Scene bootstrap: DontDestroyOnLoad cho services sống qua scene (nếu multi-scene) **hoặc** single scene rõ ràng trong README
- Repo Git + README chạy được

**Không yêu cầu:** combat, inventory đầy đủ, network.

---

## Architecture

```text
                    ┌─────────────────────┐
                    │  Bootstrap Scene     │
                    │  GameLifetimeScope     │
                    └──────────┬──────────┘
                               │ Register
         ┌─────────────────────┼─────────────────────┐
         ▼                     ▼                     ▼
  ITimeService           IGameStateMachine      ILogService
  (delta wrappers)       (Boot/Playing/...)     (wrapper)
         │                     │
         └──────────┬──────────┘
                    ▼
              GameRunner (EntryPoint / ITickable)
                    │ Tick(dt)
         ┌──────────┼──────────┐
         ▼          ▼          ▼
   Future systems (M02+) — chưa cần implement hết

MonoBehaviours:
  PlayerView / CameraFollow — chỉ input & presentation, gọi service
```

```text
Folders:
  Assets/_Project/
    Architecture/
    Gameplay/
    UI/
    Shared/
```

---

## Tasks

1. Tạo project / nâng cấp từ L21; tạo cấu trúc folder.  
2. Cài DI; `GameLifetimeScope` đăng ký `ILogService`, `ITimeService`, `GameRunner`.  
3. `GameRunner` implement tick: log mỗi 5s “heartbeat” (tắt được bằng flag).  
4. `PlayerMotor` MB mỏng: đọc input → gọi `IMovementService` hoặc trực tiếp move (tạm).  
5. Viết `ARCHITECTURE.md` mô tả sơ đồ trên.  
6. Commit: `chore: milestone 01 core architecture`.

---

## Expected result

- Enter Play: không lỗi DI; player di chuyển được.  
- Console có heartbeat có kiểm soát.  
- Người khác đọc ARCHITECTURE.md hiểu composition root.  
- Không còn singleton `GameManager.Instance` bắt buộc (nếu còn, phải ghi chú technical debt và kế hoạch bỏ).

---

## Exercises

**E1** — Thêm `IRng` inject (seed cố định) dùng cho lần roll thử.  
**E2** — State machine 2 trạng thái `Boot` → `Playing` (enum + service).  
**E3** — Tắt heartbeat bằng Options/ScriptableObject config.  
**E4** — Viết 1 unit test Editor cho class thuần (assembly definition test).

---

## Hints

- VContainer: `RegisterEntryPoint<GameRunner>()` + `ITickable`.  
- Tránh `FindObjectOfType` trong service.  
- MB lấy dependency bằng `[Inject]` method/field theo docs package — hoặc serialize chỉ view refs.  
- Single scene OK cho capstone giai 1.

---

## Solution outline

1. `ILogService` / `UnityLogService` map Debug.Log* theo level.  
2. `GameRunner : ITickable` giữ `_accum` cho heartbeat.  
3. `MovementService.Move(TransformProxy hoặc IMover, input)`.  
4. `PlayerInputMB` → service.  
5. LifetimeScope parent ở scene Bootstrap.  
6. asmdef `_Project.Gameplay` tách nếu muốn test.

Pseudo:

```csharp
public class GameRunner : ITickable
{
    readonly ILogService _log;
    readonly HeartbeatOptions _opts;
    float _t;
    public void Tick()
    {
        if (!_opts.Enabled) return;
        _t += Time.deltaTime;
        if (_t >= _opts.Interval) { _t = 0; _log.Info("heartbeat"); }
    }
}
```

---

## Code review checklist

- [ ] Một composition root rõ, không đăng ký lung tung trong MB ngẫu nhiên  
- [ ] Không secret/API key hardcode  
- [ ] Service không reference `UnityEngine.UI` trừ khi là Presenter/View layer  
- [ ] README: version Unity, package DI, scene mở nào  
- [ ] Không spam log mỗi frame  
- [ ] Folder structure nhất quán  
- [ ] Git history sạch (không `Library/`)  
- [ ] Technical debt (nếu có) ghi trong ARCHITECTURE.md  
