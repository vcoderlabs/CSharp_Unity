# Chương 4 — Config & environment

## 1. Mục tiêu học

- Tách **cấu hình** khỏi code
- Dùng `appsettings.json` + biến môi trường + User Secrets
- Phân biệt Development / Staging / Production
- Không commit secret

## 2. Điều kiện tiên quyết

- ASP.NET Core hoặc Generic Host (console cũng dùng được)
- Chương 2–3 Level 20

## 3. Khái niệm

**Configuration providers** xếp lớp: file → env vars → secrets → dòng lệnh. Cái sau ghi đè cái trước (theo thứ tự đăng ký).

| Nguồn | Dùng khi |
|-------|----------|
| `appsettings.json` | Giá trị không bí mật |
| `appsettings.Development.json` | Local override |
| Environment variables | Prod / container |
| User Secrets | Secret lúc dev |
| Azure Key Vault / etc. | Secret prod |

**Options pattern:** bind section → class typed `IOptions<T>`.

## 4. Mô hình tư duy

```text
Code chỉ biết: IOptions<SmtpOptions>
                 ▲
    appsettings + ENV + secrets
                 ▲
    Dev máy bạn ≠ Prod cluster
```

`ASPNETCORE_ENVIRONMENT` / `DOTNET_ENVIRONMENT` = tên môi trường.

## 5. Cú pháp

```json
// appsettings.json
{
  "Caching": {
    "TtlSeconds": 60
  },
  "ConnectionStrings": {
    "Main": ""
  }
}
```

```csharp
public sealed class CachingOptions
{
    public const string SectionName = "Caching";
    public int TtlSeconds { get; set; } = 60;
}

builder.Services.Configure<CachingOptions>(
    builder.Configuration.GetSection(CachingOptions.SectionName));

// Inject
public sealed class CacheService(IOptions<CachingOptions> options)
{
    private readonly TimeSpan _ttl = TimeSpan.FromSeconds(options.Value.TtlSeconds);
}
```

User Secrets:

```bash
dotnet user-secrets init --project src/ProdApp.Api
dotnet user-secrets set "ConnectionStrings:Main" "Server=...;Pwd=..."
```

Env var (hierarchical): `Caching__TtlSeconds=120`

## 6. Ví dụ

### Cơ bản — đọc chuỗi

```csharp
string? cs = builder.Configuration.GetConnectionString("Main");
```

### Trung cấp — validate Options

```csharp
builder.Services.AddOptions<CachingOptions>()
    .BindConfiguration(CachingOptions.SectionName)
    .Validate(o => o.TtlSeconds > 0, "Ttl must be positive")
    .ValidateOnStart();
```

### Nâng cao — không nhánh môi trường trong business

```csharp
// Tránh: if (env.IsProduction()) { specialLogic(); }
// Thích: config khác nhau, cùng code path
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Prod dùng connection local | Sai env / thiếu env var | Kiểm tra ENVIRONMENT |
| Secret trên GitHub | Commit appsettings có pwd | Secrets + rotate |
| Bind null | Sai tên section | Khớp key / case |
| `__` vs `:` | Env hierarchy | Dùng `__` trên env |

## 8. Gỡ lỗi

1. In (tạm, cẩn thận) `configuration.AsEnumerable()` lúc startup — không log secret.
2. `dotnet user-secrets list`
3. Docker: kiểm tra `-e` / compose `environment`.
4. Options ValidateOnStart fail → app không lên — đúng ý.

## 9. Best practices

- Default an toàn trong code; prod bắt buộc cấu hình rõ.
- Connection string chỉ từ env/secret store.
- Feature flags qua config nếu cần (không hardcode forever).
- Document mọi key trong README (không giá trị secret).
- `appsettings.Production.json` thường tối giản — giá trị thật từ platform.

## 10. Bài tập

**Bài 1** — Tạo `CachingOptions`, bind, inject vào service.

**Bài 2** — Đặt TTL khác nhau bằng env var, chạy và xác nhận.

**Bài 3** — Chuyển một mật khẩu giả từ json sang user-secrets.

**Bài 4** — Viết bảng “key config → ý nghĩa → ví dụ non-secret”.

## 11. Gợi ý

- Bài 2: `Caching__TtlSeconds=5` trước `dotnet run`.
- Bài 3: xóa pwd khỏi json đã track bằng git carefully.

## 12. Đáp án

**Bài 1** — Như mục Cú pháp.

**Bài 2** — Đọc `options.Value.TtlSeconds` in ra Information log lúc start.

**Bài 3** — `dotnet user-secrets set "ConnectionStrings:Main" "..."`; json để trống hoặc placeholder.

## 13. Đáp án thay thế

`IOptionsSnapshot` / `IOptionsMonitor` khi cần reload. Consul/etcd cho dynamic config.

## 14. Thử thách

Thêm validation: nếu Production mà connection string trống → fail fast với message rõ.

## 15. Ứng dụng thực tế

- 12-factor app
- Kubernetes ConfigMap + Secret
- Slot settings Azure App Service

## 16. Liên hệ Unity

- `ScriptableObject` config gameplay
- Build-time scripting define symbols
- Remote config services
- Không hardcode API key trong client (luôn rủi ro — proxy server)

## 17. Kiểm tra kiến thức

1. Options pattern là gì?  
   **Đáp án:** Bind config section vào class typed, inject qua DI.

2. Env var ghi đè json được không?  
   **Đáp án:** Có, nếu provider đăng ký sau/ưu tiên đúng.

3. User Secrets dùng ở đâu?  
   **Đáp án:** Máy dev — không thay secret production.

4. `Caching__TtlSeconds` map thế nào?  
   **Đáp án:** Section Caching, key TtlSeconds.

5. Vì sao không if (IsProduction) trong domain?  
   **Đáp án:** Khó test; đẩy khác biệt vào config/behavior inject.
