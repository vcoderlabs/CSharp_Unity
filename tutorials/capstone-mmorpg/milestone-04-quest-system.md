# Milestone 04 — Quest System

## Requirements

- Quest definition data-driven (SO hoặc JSON)
- **State pattern** (hoặc state machine rõ): Locked / Available / Active / Completed / TurnedIn (tùy thiết kế, tối thiểu Active/Completed)
- **Observer**: UI và hệ thống khác lắng nghe tiến độ
- Điều kiện hoàn thành ví dụ: Kill N enemy type **hoặc** Collect N item
- NPC/quest giver tối thiểu (interact)
- Serialize được **progress runtime** (chuẩn bị M07; tạm JSON memory OK)

**Không yêu cầu:** quest graph hàng trăm node, dialogue tree AAA.

---

## Architecture

```text
QuestDefinition (SO)
  Id, Title, Steps[]
  Step: KillGoal { EnemyId, Count } | CollectGoal { ItemId, Count }

QuestState enum: Available → Active → Completed

IQuestService
  Start(questId)
  NotifyKill(enemyId)
  NotifyItem(itemId, qty)
  GetProgress(questId)

raises QuestUpdated / QuestCompleted

QuestLogPresenter → QuestLogView
QuestGiverMB → Start / TurnIn

InventoryService ──event──► QuestService (collect goals)
Death/Kill bus ──event──► QuestService (kill goals)
```

```text
State transitions:
  Available --Start--> Active --goals met--> Completed --TurnIn--> TurnedIn
```

---

## Tasks

1. Model definition + progress (`currentCount` per step).  
2. `QuestService` inject event buses.  
3. Một quest kill + một quest collect.  
4. UI log hiển thị tiến độ.  
5. Quest giver interact.  
6. Test: giả lập 3 kill → completed.  
7. Cập nhật ARCHITECTURE (State + Observer).

---

## Expected result

- Nhận quest → giết đủ → UI hiện Completed.  
- Collect quest lắng nghe inventory.  
- Không hardcode tiến độ trong `Update` scan toàn scene.  
- Progress object serialize được (POC `JsonUtility` / STJ).

---

## Exercises

**E1** — Quest có 2 bước tuần tự.  
**E2** — Reward: gọi `IInventory.TryAdd` khi TurnIn.  
**E3** — Không Start lại quest TurnedIn.  
**E4** — Localization key thay title cứng (optional).

---

## Hints

- Goal interface `IQuestGoal.OnKill` / `OnItem` trả bool changed.  
- State pattern: class `ActiveQuestState` vs enum + switch — chọn một, nhất quán.  
- Tránh UI poll mỗi frame — event-driven.  
- Id enemy khớp M02 Team/definition.

---

## Solution outline

```csharp
public interface IQuestGoal
{
    bool IsMet { get; }
    void OnEnemyKilled(string enemyId);
    void OnItemGained(ItemId id, int qty);
    string DebugProgress { get; }
}

public sealed class KillGoal : IQuestGoal { /* count++ khi match */ }

public sealed class QuestService : IQuestService
{
    // dictionary progress
    // subscribe killChannel & inventory.Changed / ItemAdded
}
```

TurnIn: check Completed → grant reward → state TurnedIn → save dirty flag.

---

## Code review checklist

- [ ] Có state chuyển rõ, không boolean `isDone` mơ hồ nhiều cờ  
- [ ] Observer unsubscribe đúng lifecycle  
- [ ] Definition tách khỏi Progress  
- [ ] Test tiến độ không cần vào Play nếu logic thuần  
- [ ] Quest id ổn định  
- [ ] ARCHITECTURE nêu State + Observer  
- [ ] Không Complete ngay khi Start vì bug count  
- [ ] Reward không duplicate khi spam TurnIn  
