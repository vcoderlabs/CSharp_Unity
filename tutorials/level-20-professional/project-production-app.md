# Project chung L19–20 — Production-grade C# application

Project này **gom kỹ năng Level 19 (Performance) và Level 20 (Professional)** thành một ứng dụng nhỏ nhưng “đủ nghề”: Git, solution sạch, config, logging, CI, đo và tối ưu một hot path.

**Thời lượng gợi ý:** 4–8 giờ (tùy độ sâu). Có thể bắt đầu từ giữa L20 và quay lại phần perf khi xong L19.

---

## 1. Mục tiêu học

- Xây app console **hoặc** Minimal API có: use-case rõ, test, log, config
- Áp dụng GitHub + CI
- Chọn **một hot path**, benchmark, tối ưu có số liệu
- Viết README người khác chạy được trong 5 phút

## 2. Điều kiện tiên quyết

- Level 19 chương 1–5 (ít nhất đọc + làm bài chính)
- Level 20 chương 1–6
- .NET 8 SDK, GitHub account

## 3. Khái niệm sản phẩm (chọn 1)

| Đề xuất | Mô tả | Hot path gợi ý |
|---------|--------|----------------|
| **A. Order mini-API** | Tạo đơn, lấy đơn theo id, list theo user | Serialize / lookup dictionary / validation |
| **B. Inventory CLI** | Import CSV items, query by id/tag | Parse CSV, index build |
| **C. Leaderboard service** | Ingest score events, top-N | Sort vs heap / cache top-N |

Chọn **một** — đừng làm cả ba.

## 4. Mô hình tư duy / kiến trúc

```text
ProdApp.sln
├── src/ProdApp.Domain          # entities, interfaces
├── src/ProdApp.Application     # commands/queries
├── src/ProdApp.Infrastructure  # in-memory repo, file, cache
├── src/ProdApp.Host            # API hoặc Console + DI
└── tests/ProdApp.UnitTests
     + benchmarks/ProdApp.Benchmarks   (tùy chọn project riêng)
```

```text
Request/Command
    → Application service (log + validate)
        → IRepository / ICache
            → Infrastructure
```

## 5. Cú pháp / skeleton Host

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.Configure<CachingOptions>(
    builder.Configuration.GetSection(CachingOptions.SectionName));
builder.Services.AddSingleton<IOrderRepository, InMemoryOrderRepository>();
builder.Services.AddSingleton<OrderService>();
// logging đã có sẵn trong WebApplication

var app = builder.Build();
app.MapHealthChecks("/health");
app.MapPost("/orders", async (CreateOrderRequest req, OrderService svc) =>
    Results.Ok(await svc.CreateAsync(req)));
app.Run();
```

(Console host: `Host.CreateApplicationBuilder` + vòng lệnh.)

## 6. Ví dụ phạm vi MVP

### Phải có

1. Domain entity + interface repository  
2. ≥ 5 unit tests xanh  
3. `appsettings` + ít nhất 1 Options class  
4. Structured logging khi tạo/lấy resource  
5. GitHub Actions CI build+test  
6. README: chạy thế nào, cấu hình gì  
7. Một báo cáo ngắn `docs/PERF.md`: trước/sau Mean + Allocated  

### Không bắt buộc

- Database thật, auth OAuth, Docker, frontend

## 7. Lỗi thường gặp

| Hiện tượng | Cách xử lý |
|------------|------------|
| Scope phình | Cắt feature; MVP trước |
| Benchmark Debug | `-c Release` |
| Secret trong repo | User secrets / env |
| Test phụ thuộc tường hệ thống tệ | In-memory + time abstraction |

## 8. Gỡ lỗi

1. `dotnet test` đỏ → sửa trước perf.  
2. CI đỏ local xanh → pin SDK.  
3. Perf không cải thiện → sai hot path; profile lại.  
4. Cache “sai dữ liệu” → TTL/invalidation trong PERF note.

## 9. Best practices

- PR nhỏ: “feat: create order”, “perf: pool buffer parse”, “ci: add workflow”.  
- Mọi tối ưu gắn issue/đo.  
- Public API ổn định trước khi micro-optimize.  
- `TreatWarningsAsErrors` nếu chịu được.

## 10. Bài tập (deliverables)

**Bài 1** — Skeleton solution + 1 use-case end-to-end.  
**Bài 2** — CI xanh trên GitHub.  
**Bài 3** — Benchmark 2 phiên bản hot path (ví dụ parse/lookup).  
**Bài 4** — Áp dụng cache **hoặc** pool; ghi hit rate / alloc trong `PERF.md`.

## 11. Gợi ý

- Inventory CSV: so `Split` vs Span parser (L19 ch.2 thử thách).  
- Leaderboard: full sort mỗi lần vs cập nhật `PriorityQueue` / cached top-N.  
- Order API: `Dictionary<Guid, Order>` vs linear list.

## 12. Đáp án — outline hoàn chỉnh (Order API)

1. `Order` record: Id, UserId, Items, CreatedAt  
2. `IOrderRepository`: Add, Get, ListByUser  
3. `OrderService`: validate items không rỗng; log Information  
4. `InMemoryOrderRepository` + optional `MemoryCache` theo order id  
5. Tests: create ok; create empty fail; get missing  
6. Benchmark: `ListByUser` trên List.scan vs index `Dictionary<UserId, List<OrderId>>`  
7. CI yaml như L20 ch.5  
8. `PERF.md` dán bảng BDN  

## 13. Đáp án thay thế

CLI Inventory: không cần Web SDK — dễ hơn nếu chưa học ASP.NET. Vẫn đủ professional skills.

## 14. Thử thách

- OpenAPI + một integration test `WebApplicationFactory`  
- Metric đếm request + cache hit (log định kỳ)  
- Deploy container local bằng Dockerfile đơn giản  

## 15. Ứng dụng thực tế

Đây là “mẫu” portfolio: recruiter/clone chạy được, thấy CI, thấy bạn biết đo perf.

## 16. Liên hệ Unity

Sang L21/Capstone: cùng kỷ luật (tầng service, log có mức, pool có đo). Host khác (Player) nhưng tư duy production giữ nguyên.

## 17. Kiểm tra kiến thức / Definition of Done

1. Người lạ clone → `dotnet test` xanh trong README?  
2. Có số liệu perf trước/sau?  
3. Không có secret trên git?  
4. Hot path đã chứng minh bằng đo, không đoán?  
5. PR/CI story rõ?

**Definition of Done project:** cả 5 câu trả lời **Có**.
