# Chương 3 — DI & Dependency Inversion (tầm ứng dụng)

## 1. Mục tiêu học

- Áp DIP ở quy mô solution: ports, composition root, lifetimes
- Dùng Microsoft.Extensions.DependencyInjection (hoặc container tương đương)
- Tránh Service Locator trong domain
- Unity: so sánh với VContainer/Zenject (khái niệm)

## 2. Điều kiện tiên quyết

- L16 DIP
- Clean Architecture (chương 2)

## 3. Khái niệm

Ở tầm app, **mọi concrete infra** được tạo ở composition root và inject vào use case. Container đăng ký `interface → implementation` + lifetime (Transient/Scoped/Singleton).

## 4. Mô hình tư duy

```text
Program.cs (Composition Root)
  services.AddSingleton<IClock, SystemClock>();
  services.AddScoped<IQuestRepository, SqliteQuestRepository>();
  services.AddTransient<CompleteQuest>();
→ Resolve root adapter (Controller/CLI) → đồ thị phụ thuộc
```

## 5. Cú pháp

```csharp
var services = new ServiceCollection();
services.AddSingleton<IQuestRepository, InMemoryQuestRepository>();
services.AddTransient<CompleteQuest>();
var sp = services.BuildServiceProvider();
var useCase = sp.GetRequiredService<CompleteQuest>();
```

## 6. Ví dụ

### Cơ bản

Wire tay không container — vẫn DI nếu ctor inject.

### Trung cấp — MS.DI

Như mục 5; CLI:

```csharp
var app = sp.GetRequiredService<QuestCli>();
app.Run(args);
```

### Nâng cao / Unity

```csharp
// VContainer pseudo
builder.Register<InventoryService>(Lifetime.Singleton).As<IInventoryService>();
builder.RegisterComponentInHierarchy<LootPickup>();
```

Lifetime: Singleton cho service không state scene-risky; cẩn thận MonoBehaviour lifetime.

## 7. Lỗi thường gặp

| Hiện tượng | Cách xử lý |
|------------|------------|
| Captive dependency | Scoped trong Singleton giữ sai |
| `BuildServiceProvider` nhiều lần | Một root |
| Resolve trong Domain | Cấm — chỉ adapter/root |
| Circular DI | Tách use case / event |

## 8. Gỡ lỗi

Exception “Unable to resolve” — thiếu đăng ký. ValidateScopes khi dev. Log đồ thị khi startup (debug).

## 9. Best practices

- Registration theo module/feature extension methods.  
- Domain không reference container package.  
- Prefer ctor inject.

## 10. Bài tập

**Bài 1** — Đăng ký 2 repo + 1 use case MS.DI.  
**Bài 2** — Cố ý captive dependency; giải thích.  
**Bài 3** — Chuyển `Xxx.Instance` sang DI.  
**Bài 4** — So Transient/Scoped/Singleton cho `DbContext` vs `IClock`.

## 11. Gợi ý

`IClock` → Singleton; `DbContext` → Scoped (web request).

## 12. Đáp án

```csharp
services.AddSingleton<IClock, SystemClock>();
services.AddScoped<IUserRepository, EfUserRepository>();
services.AddTransient<RegisterUser>();
```

## 13. Đáp án thay thế

Pure DI (manual factories) — không package; tốt khi đồ thị nhỏ.

## 14. Thử thách

Extension `AddInventoryModule(IServiceCollection)` đăng ký hết feature.

## 15. Ứng dụng thực tế

- ASP.NET Core hosting  
- Worker services  
- Tests: thay registration fake

## 16. Liên hệ Unity

- VContainer/Zenject/strangeIoC  
- Entry point installer scene  
- Không ServiceLocator trong combat loop nóng nếu tránh được

## 17. Kiểm tra kiến thức

1. Composition root? **Nơi wire concrete.**  
2. Domain được reference DI container? **Không nên.**  
3. Captive dependency? **Lifetime dài giữ phụ thuộc ngắn hơn sai cách.**  
4. DIP vs DI? **Nguyên tắc vs kỹ thuật.**  
5. Vì sao extension `AddFeature`? **Modular registration.**
