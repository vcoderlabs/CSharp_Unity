# Project chung L15–18 — Clean Architecture Application

Project **dùng chung Level 15 → 18**: một codebase tiến hóa từ clean code → SOLID → patterns có chọn lọc → architecture rõ. Không tạo bốn repo khác nhau.

**Ước lượng riêng phần hoàn thiện kiến trúc (L18): ~5–7 giờ** (đã có nền từ L15–17).

---

## 1. Mục tiêu học

- Xây monolith modular theo Clean Architecture bằng C# (.NET console hoặc Minimal API)
- Áp dụng checklist L15–18 trên **cùng** domain
- Có test domain/use case không phụ thuộc I/O thật
- Chuẩn bị tư duy trước Capstone Unity MMORPG (cùng kiểu tách Core)

## 2. Điều kiện tiên quyết

- Level 15: naming, refactor, YAGNI
- Level 16: SOLID trên service/domain
- Level 17: Observer/State/Command *khi cần*; đọc “when not to use”
- Level 18 chương 1–6 (hoặc song song từ chương 2)

## 3. Khái niệm — chọn domain

Chọn **một** (không làm cả hai lần đầu):

| Domain gợi ý | Aggregates / features |
|--------------|------------------------|
| **A. Quest & Inventory** (gần MMORPG) | Player, Quest, Inventory, Item |
| **B. Order & Pricing** | Customer, Order, Product, Discount rules |

README bên dưới giả định **Domain A** — đổi tên tương đương nếu chọn B.

## 4. Mô hình tư duy — cấu trúc solution

```text
CleanApp.sln
├── src/
│   ├── CleanApp.Domain/           # Entities, value objects, domain events (optional)
│   ├── CleanApp.Application/      # Use cases, ports (interfaces), DTOs/contracts
│   ├── CleanApp.Infrastructure/   # File/SQLite repos, clock, email stub
│   └── CleanApp.Hosts.Cli/        # Composition root + CLI adapter
└── tests/
    ├── CleanApp.Domain.Tests/
    └── CleanApp.Application.Tests/
```

**Dependency Rule:** Domain ← Application ← Hosts; Infrastructure → Application/Domain (implement ports). Hosts reference tất cả để wire.

## 5. Cú pháp — skeleton tối thiểu

```csharp
// Domain
public readonly record struct PlayerId(Guid Value);
public sealed class Quest
{
    public QuestId Id { get; }
    public string Title { get; }
    public QuestStatus Status { get; private set; }
    public void Start() { /* NotStarted → Active */ }
    public void Complete() { /* Active → Completed */ }
}

// Application port
public interface IQuestRepository
{
    Task<Quest?> GetAsync(QuestId id, CancellationToken ct = default);
    Task SaveAsync(Quest quest, CancellationToken ct = default);
}
```

## 6. Ví dụ / phạm vi chức năng bắt buộc

### MVP (bắt buộc)

1. Tạo player + inventory trống  
2. Thêm item vào inventory  
3. Tạo quest yêu cầu item  
4. Start quest → Complete khi đủ item (consume item)  
5. Persist JSON hoặc SQLite  
6. CLI: lệnh `inventory`, `quest list`, `quest complete <id>`

### Mở rộng (khuyến nghị L17)

- `QuestStatus` = **State** pattern hoặc enum + transition rõ  
- `QuestCompleted` event → **Observer** (log + achievement stub)  
- `CompleteQuest` như **Command** object (optional)  
- **Không** bắt buộc Abstract Factory

## 7. Lỗi thường gặp

| Hiện tượng | Cách xử lý |
|------------|------------|
| Domain reference Newtonsoft/EF | Chuyển ra Infrastructure |
| Use case 200 dòng | Tách domain methods |
| CLI chứa rule | Chỉ parse args + gọi use case |
| Over-pattern | Xóa theo L17 ch.18 |

## 8. Gỡ lỗi

1. Test đỏ domain trước khi wire file.  
2. Breakpoint composition root registrations.  
3. Xóa DB/file save để tái hiện bug persistence.  
4. Architecture: cố `dotnet add` Domain → Infra — phải là việc *không* làm.

## 9. Best practices

- Một use case / class (hoặc folder rõ).  
- DTO chỉ ở biên CLI/Application public.  
- Fake repos trong unit test; một integration test Infra.  
- Commit nhỏ theo level (L15 rename → L16 split → L18 projects).

## 10. Bài tập — task theo level (cụ thể)

### Phase L15 — Clean Code

- [ ] Đặt tên entity/use case rõ tiếng Anh thống nhất (hoặc Việt có chủ đích — chọn một)  
- [ ] Không God `GameManager`  
- [ ] Hàm dài > 30 dòng: tách  
- [ ] README domain 20 dòng: glossary Player/Quest/Item  

### Phase L16 — SOLID

- [ ] SRP: tách `CompleteQuest` khỏi CLI  
- [ ] OCP: thêm quest reward type mới (XP **hoặc** Item) bằng class/handler mới  
- [ ] ISP: `IQuestRead` / `IQuestWrite` nếu client khác nhu cầu  
- [ ] DIP: không `new FileQuestRepository` trong use case  

### Phase L17 — Patterns (có chọn lọc)

- [ ] Observer: log khi quest completed  
- [ ] State: cấm complete khi NotStarted  
- [ ] (Optional) Command cho CLI actions  
- [ ] Viết mục “Patterns đã từ chối” trong README (≥ 2)

### Phase L18 — Architecture (bắt buộc hoàn thiện)

- [ ] Tách ≥ 4 project như mục 4  
- [ ] Composition root đăng ký DI  
- [ ] `InMemory*` + `File*` (hoặc Sqlite*) repository  
- [ ] DTO cho CLI output — không in entity dump  
- [ ] Module biên: folder/project `Inventory` vs `Quest` + contracts  
- [ ] Test: ≥ 5 unit domain/app + ≥ 1 persistence integration  

## 11. Gợi ý

- Bắt đầu InMemory để xong use case; file sau.  
- `PlayerId`/`QuestId` strong type tránh `Guid` nhầm.  
- Complete flow: load quest + inventory → domain checks → save cả hai (UoW giả: tuần tự + handle fail).  

## 12. Đáp án — outline use case Complete

```csharp
public sealed class CompleteQuestHandler
{
    private readonly IQuestRepository _quests;
    private readonly IInventoryRepository _inventory;
    private readonly IEnumerable<IQuestCompletedHandler> _onCompleted;

    public async Task HandleAsync(PlayerId player, QuestId questId, CancellationToken ct)
    {
        var quest = await _quests.GetAsync(questId, ct)
            ?? throw new InvalidOperationException("quest missing");
        var inv = await _inventory.GetAsync(player, ct)
            ?? throw new InvalidOperationException("inventory missing");

        // domain: quest.Requirements met? inv.Consume(...); quest.Complete();
        await _inventory.SaveAsync(inv, ct);
        await _quests.SaveAsync(quest, ct);

        foreach (var h in _onCompleted)
            await h.HandleAsync(quest, ct);
    }
}
```

(Chi tiết `Consume`/`Complete` để bạn tự viết — đó là phần chấm điểm.)

## 13. Đáp án thay thế

- Host Minimal API thay CLI — cùng Application.  
- Một project + folders + arch tests — chấp nhận nếu reference rule được enforce.  
- Domain B (Order): `SubmitOrder` + `IPricing` + `IOrderRepository` map 1:1 task trên.

## 14. Thử thách

- Thêm `AchievementModule` chỉ listen event — không reference Quest infra.  
- Version save file v1 → v2 migration mỏng.  
- Benchmark add 10k items (chuẩn bị L19) — chỉ đo, chưa tối ưu sớm.

## 15. Ứng dụng thực tế

- Đây là bản thu nhỏ của backend feature team + shared kernel.  
- Portfolio: link solution + test xanh + ADR ngắn.  
- Cùng tư duy chuyển sang Unity asmdef Core.

## 16. Liên hệ Unity / Capstone

| Project này | Capstone MMORPG |
|-------------|-----------------|
| Domain + Application | Core gameplay assemblies |
| CLI Host | MonoBehaviour adapters / UI |
| File repo | Save system / server API later |
| Quest State + Observer | Milestone 04 Quest |
| Inventory API module | Milestone 03 Inventory |
| DI composition root | Milestone 01 DI container |

**Không** nhét UnityEngine vào Domain của project này — giữ thuần để học architecture.

## 17. Kiểm tra kiến thức

1. Vì sao project chung L15–18 thay vì 4 app?  
   **Đáp án:** Cùng domain tiến hóa kỹ năng; tránh viết lại từ đầu mỗi level.

2. Dependency Rule cấm gì?  
   **Đáp án:** Layer trong phụ thuộc chi tiết ngoài (Domain → Infra).

3. Use case CompleteQuest nên phụ thuộc gì?  
   **Đáp án:** Ports (repo/API), không file path cụ thể.

4. Pattern nào *bắt buộc* trong project?  
   **Đáp án:** Không bắt buộc trừ State/Observer tối thiểu ở phase L17; tránh over-pattern.

5. Dấu hiệu hoàn thành L18?  
   **Đáp án:** Multi-project đúng chiều phụ thuộc, DI root, DTO biên, tests xanh, module Quest/Inventory tách contracts.

---

## Checklist nộp bài

- [ ] Solution build được (`dotnet build`)  
- [ ] `dotnet test` xanh  
- [ ] README glossary + “patterns từ chối”  
- [ ] CLI demo được flow MVP  
- [ ] Không còn God class / Singleton bừa bãi trong Application/Domain  

Xong project → bạn sẵn sàng **Level 19 Performance** hoặc **Level 21 / Capstone** với nền tảng kiến trúc vững.
