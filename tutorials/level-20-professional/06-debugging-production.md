# Chương 6 — Debugging production

## 1. Mục tiêu học

- Tiếp cận lỗi **không reproduce** trên máy local
- Dùng log, metric, health check, correlation id
- Phân biệt symptom vs root cause
- Biết giới hạn: khi cần dump / feature flag / rollback

## 2. Điều kiện tiên quyết

- Logging + config (chương 3–4)
- Git bisect khái niệm (chương 1)
- Async/exception cơ bản

## 3. Khái niệm

Production khác dev:

- Dữ liệu thật, traffic thật, timing thật
- Không gắn debugger tùy tiện
- Quan sát qua **telemetry**: logs, metrics, traces

| Tín hiệu | Câu hỏi |
|----------|---------|
| Logs | Chuyện gì xảy ra với request X? |
| Metrics | Error rate / latency có tăng không? |
| Traces | Request đi qua service nào chậm? |
| Health | Process còn sống / dependency chết? |

**Rollback** thường nhanh hơn “fix nóng” khi incident lớn.

## 4. Mô hình tư duy

```text
Alert: p99 latency ↑ / 5xx ↑
  → Dashboard: bắt đầu lúc nào? deploy nào?
  → Logs: correlation id mẫu lỗi
  → Giả thuyết → xác minh bằng dữ liệu
  → Mitigate (rollback/flag) → fix gốc → postmortem ngắn
```

## 5. Cú pháp

Correlation:

```csharp
app.Use(async (ctx, next) =>
{
    string id = ctx.Request.Headers["X-Request-Id"].FirstOrDefault()
                ?? Guid.NewGuid().ToString("N");
    ctx.Response.Headers["X-Request-Id"] = id;
    using (ctx.RequestServices.GetRequiredService<ILoggerFactory>()
               .CreateLogger("Request")
               .BeginScope(new Dictionary<string, object> { ["RequestId"] = id }))
    {
        await next();
    }
});
```

Health:

```csharp
builder.Services.AddHealthChecks()
    .AddCheck("self", () => HealthCheckResult.Healthy());
app.MapHealthChecks("/health");
```

## 6. Ví dụ

### Cơ bản — đọc log theo id

User báo lỗi 14:03 → tìm `RequestId` trong response header (nếu có) hoặc khoảng thời gian + UserId.

### Trung cấp — giả thuyết

```text
Sympton: timeout checkout
Giả thuyết A: DB chậm — xem metric DB
Giả thuyết B: deadlock — xem thread / lock log
Giả thuyết C: downstream HTTP — xem trace span
```

### Nâng cao — safe hotfix

Feature flag tắt đường mới; hoặc revert PR; hotfix branch + CI + deploy có audit.

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| “Không tái hiện” | Data/race/env | Thêm log có điều kiện; capture payload an toàn |
| Log thiếu context | Không có id | Correlation bắt buộc |
| Debug prod bằng SSH sửa file | Không audit | Immutable deploy |
| Sửa triệu chứng | Restart hết | Tìm root; restart chỉ mitigate |

## 8. Gỡ lỗi

1. Timeline: deploy, config change, traffic spike.  
2. Một request lỗi end-to-end.  
3. So sánh request thành công vs thất bại.  
4. `git bisect` trên staging với script tái hiện.  
5. Memory dump chỉ khi cần và có quy trình (PII!).

## 9. Best practices

- Fail fast + message lỗi có mã (`ORD-429`) user gửi support.
- Alert actionable (không tỉnh dậy vì CPU 1 phút).
- Runbook: “khi alert X thì làm Y”.
- Postmortem không đổ lỗi cá nhân — cải tiến hệ thống.
- Không log body nhạy cảm; redact.

## 10. Bài tập

**Bài 1** — Thêm RequestId middleware + log Information mỗi request path/status.

**Bài 2** — Map `/health` và gọi bằng curl.

**Bài 3** — Viết runbook 1 trang: “API 5xx > 5% trong 5 phút”.

**Bài 4** — Giả sử bug chỉ khi `items.Count == 0`: viết test + log Warning trước khi fix.

## 11. Gợi ý

- Bài 3: gồm detect → mitigate → escalate → fix.
- Bài 4: TDD production bug.

## 12. Đáp án

**Bài 3 — Runbook mẫu:**

1. Xác nhận metric trên dashboard  
2. Kiểm tra deploy gần nhất → rollback nếu trùng thời điểm  
3. Lấy 3 RequestId lỗi từ log  
4. Phân loại exception  
5. Thông báo status  
6. Fix + test + deploy  
7. Ghi postmortem ngắn  

**Bài 1–2** — Theo cú pháp.

## 13. Đáp án thay thế

APM: Application Insights, Datadog, Grafana Tempo. Exception tracking: Sentry.

## 14. Thử thách

Mô phỏng incident: deploy cố ý chậm endpoint; dùng log+metric để “phát hiện” và rollback.

## 15. Ứng dụng thực tế

- On-call rotation
- SLO/SLI/SLA
- Chaos engineering mức nhẹ

## 16. Liên hệ Unity

- Crashlytics / Cloud Diagnostics
- Player.log trên máy user
- Không “debug production” client bằng cheats lộ
- Server-authoritative: log phía server tin cậy hơn client

## 17. Kiểm tra kiến thức

1. Vì sao khó debug prod bằng Visual Studio attach?  
   **Đáp án:** Scale, quyền, PII, downtime; thường dùng telemetry.

2. Correlation id giúp gì?  
   **Đáp án:** Ghép mọi log của một request.

3. Mitigate khác fix?  
   **Đáp án:** Mitigate giảm thiệt hại ngay (rollback/flag); fix sửa gốc.

4. Health check dùng để làm gì?  
   **Đáp án:** Orchestrator biết instance có nhận traffic không.

5. Postmortem nên tập trung gì?  
   **Đáp án:** Timeline, nguyên nhân, action items — không đổ lỗi cá nhân.
