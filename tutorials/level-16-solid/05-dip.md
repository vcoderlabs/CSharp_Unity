# Chương 5 — DIP (Dependency Inversion Principle)

## 1. Mục tiêu học

- Hiểu DIP: **module cấp cao không phụ thuộc module cấp thấp** — cả hai phụ thuộc abstraction
- Phân biệt DIP vs DI (Dependency Injection) vs IoC container
- Refactor `new SqlRepo()` cứng trong service
- Áp dụng Unity (VContainer/Zenject/manual inject) và project chung L15–18

## 2. Điều kiện tiên quyết

- Level 16 đầy đủ chương 1–4 (SRP/OCP/LSP/ISP)
- Interface + ctor injection
- Level 14 testing (mock abstraction) là lợi thế

## 3. Khái niệm

**DIP (Dependency Inversion Principle):**

1. High-level policy **không** phụ thuộc low-level detail.
2. Cả hai phụ thuộc **abstraction** (interface/abstract).
3. Abstraction **không** phụ thuộc detail; detail phụ thuộc abstraction.

**DI (Dependency Injection)** là *kỹ thuật* truyền phụ thuộc từ ngoài (ctor/property/method).  
**IoC container** tự resolve đồ thị phụ thuộc. DIP là *nguyên tắc*; DI/container là *cách làm*.

| Anti-pattern | Sửa |
|--------------|-----|
| `new FilePlayerRepository()` trong use case | Inject `IPlayerRepository` |
| Singleton service lấy `Database.Instance` | Inject abstraction |
| Unity `FindObjectOfType` khắp nơi | Composition root + inject |

## 4. Mô hình tư duy

```text
[UseCase / Domain Policy]  ──depends on──►  IRepository (abstraction)
                                                    ▲
                                                    │ implements
                                          [SqlRepository] (detail)

Composition Root (Program / Installer):
  bind IRepository → SqlRepository
  create UseCase(repo)
```

## 5. Cú pháp

```csharp
public interface IQuestRepository
{
    Quest? Get(Guid id);
    void Save(Quest quest);
}

public sealed class CompleteQuestUseCase
{
    private readonly IQuestRepository _repo;
    public CompleteQuestUseCase(IQuestRepository repo) => _repo = repo;

    public void Execute(Guid questId)
    {
        var q = _repo.Get(questId) ?? throw new InvalidOperationException("missing");
        q.Complete();
        _repo.Save(q);
    }
}

// Composition root
var repo = new JsonQuestRepository("quests.json");
var useCase = new CompleteQuestUseCase(repo);
```

## 6. Ví dụ

### Bad code

```csharp
public sealed class LoginService
{
    public bool Login(string user, string pass)
    {
        var db = new SqlConnection("Server=..."); // detail trong high-level
        // query users...
        File.AppendAllText("audit.log", $"{user} login");
        return true;
    }
}
```

### Problem

- Đổi SQL → MySQL / JSON phải sửa `LoginService`
- Không unit test được بدون DB + file
- Vi phạm SRP + DIP cùng lúc

### Refactor

1. `IUserStore`, `IAuditLogger`.
2. `LoginService` chỉ orchestration.
3. `Program` / installer wire implementation.

### Good code

```csharp
public interface IUserStore
{
    bool Validate(string user, string pass);
}

public interface IAuditLogger
{
    void LoginAttempt(string user, bool ok);
}

public sealed class LoginService
{
    private readonly IUserStore _users;
    private readonly IAuditLogger _audit;

    public LoginService(IUserStore users, IAuditLogger audit)
    {
        _users = users;
        _audit = audit;
    }

    public bool Login(string user, string pass)
    {
        bool ok = _users.Validate(user, pass);
        _audit.LoginAttempt(user, ok);
        return ok;
    }
}

public sealed class SqlUserStore : IUserStore
{
    public bool Validate(string user, string pass) { /* ADO/EF */ return true; }
}
```

### Unity example

```csharp
public interface IInventoryService
{
    bool TryAdd(ItemId id, int qty);
}

public class InventoryService : IInventoryService
{
    public bool TryAdd(ItemId id, int qty) { /* ... */ return true; }
}

public class LootPickup : MonoBehaviour
{
    private IInventoryService _inventory;

    // Manual inject từ bootstrap hoặc [Inject] nếu dùng Zenject/VContainer
    public void Construct(IInventoryService inventory) => _inventory = inventory;

    public void OnPickup(ItemId id) => _inventory.TryAdd(id, 1);
}

public class GameInstaller : MonoBehaviour
{
    [SerializeField] private LootPickup _loot;

    void Awake()
    {
        IInventoryService inv = new InventoryService();
        _loot.Construct(inv);
    }
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Interface trùng 1:1 class vô nghĩa | “DIP theater” | Abstraction theo *boundary* thật |
| Service Locator ẩn phụ thuộc | `Global.Get<T>()` | Ctor injection rõ |
| Circular dependency | Hai service new lẫn nhau | Tách use case / event / mediator |
| Unity inject null | Quên composition root | Awake/bootstrap wire trước use |

## 8. Gỡ lỗi

1. Tìm `new` concrete infra trong domain/use case → DIP candidate.
2. Unit test: replace `IUserStore` bằng fake in-memory — nếu không được, abstraction chưa đúng chỗ.
3. NullReference trên service field Unity → chưa `Construct`/inject.

## 9. Best practices

- Composition root duy nhất (Program, `GameInstaller`) — không `new` rải rác.
- Phụ thuộc abstraction ổn định; detail ở outer layer (L18 Clean Architecture).
- Prefer ctor injection; tránh property inject trừ UI framework bắt buộc.
- Container hữu ích khi đồ thị lớn — không bắt buộc cho app nhỏ.

## 10. Bài tập

**Bài 1** — Refactor service đang `new HttpClient` + parse JSON thành `IWeatherClient`.

**Bài 2** — Viết fake in-memory cho `IQuestRepository` + test `CompleteQuestUseCase`.

**Bài 3** — Vẽ đồ thị phụ thuộc 4 class project chung; chỉ ra composition root.

**Bài 4** — So sánh DIP vs Service Locator: nêu 2 nhược điểm locator.

## 11. Gợi ý

- Bài 1: interface trả DTO; HttpClient nằm trong adapter.
- Bài 2: `Dictionary<Guid, Quest>`.
- Bài 4: phụ thuộc ẩn, khó thấy từ signature, khó test.

## 12. Đáp án

```csharp
public interface IWeatherClient
{
    Task<WeatherInfo> GetAsync(string city);
}

public sealed class HttpWeatherClient : IWeatherClient
{
    private readonly HttpClient _http;
    public HttpWeatherClient(HttpClient http) => _http = http;
    public async Task<WeatherInfo> GetAsync(string city) { /* GET + JSON */ return new(); }
}

public sealed class OutfitSuggestionService
{
    private readonly IWeatherClient _weather;
    public OutfitSuggestionService(IWeatherClient weather) => _weather = weather;
}
```

Fake:

```csharp
public sealed class InMemoryQuestRepository : IQuestRepository
{
    private readonly Dictionary<Guid, Quest> _data = new();
    public Quest? Get(Guid id) => _data.TryGetValue(id, out var q) ? q : null;
    public void Save(Quest quest) => _data[quest.Id] = quest;
}
```

## 13. Đáp án thay thế

Factory method inject `Func<IRepo>` khi lifetime phức tạp. Hoặc partial compose thủ công không container.

## 14. Thử thách

Trên project chung L15–18: invert mọi I/O (file/db) ra interface; domain/use case không `using` namespace infra.

## 15. Ứng dụng thực tế

- ASP.NET Core DI built-in
- Hexagonal / Clean Architecture ports & adapters
- Mobile/game: swap mock backend khi offline

## 16. Liên hệ Unity

- VContainer / Zenject / strangeIoC = IoC cho scene
- Tránh singleton `GameServices.Instance` phình (anti-DIP)
- Capstone: service layer inject vào systems; xem L17 Facade/Mediator khi giao tiếp chéo

## 17. Kiểm tra kiến thức

1. DIP đảo chiều phụ thuộc thế nào?  
   **Đáp án:** High-level và low-level cùng phụ thuộc abstraction.

2. DI có bằng DIP không?  
   **Đáp án:** Không — DI là kỹ thuật hỗ trợ thực thi DIP.

3. Composition root là gì?  
   **Đáp án:** Nơi duy nhất wire concrete vào abstraction.

4. Vì sao `new SqlRepo()` trong use case xấu?  
   **Đáp án:** High-level phụ thuộc detail — khó đổi/test.

5. Service Locator khác ctor inject ở điểm nào?  
   **Đáp án:** Locator ẩn phụ thuộc; ctor làm phụ thuộc tường minh.
