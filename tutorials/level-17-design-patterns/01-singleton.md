# Chương 1 — Singleton

## 1. Mục tiêu học

- Hiểu Singleton: **một instance** toàn app + điểm truy cập toàn cục
- Biết thread-safe lazy init trong C#
- Nhận diện lạm dụng (God Singleton, ẩn phụ thuộc)
- Unity: `DontDestroyOnLoad` vs DI — khi nào chấp nhận Singleton

## 2. Điều kiện tiên quyết

- OOP, `static`, thread cơ bản (L11 hữu ích)
- Level 16 DIP — Singleton thường căng với DIP

## 3. Khái niệm

**Singleton** đảm bảo một class có đúng một instance và cung cấp global access.

Dùng hợp lý: cấu hình đọc-once, device service duy nhất.  
Nguy hiểm: trở thành Service Locator / God object, khó test, ẩn dependency.

## 4. Mô hình tư duy

```text
Ai cũng gọi GameManager.Instance  →  coupling ẩn
Prefer: inject IAudioService vào chỗ cần
Singleton chỉ khi “đúng một” là invariant nghiệp vụ/kỹ thuật
```

## 5. Cú pháp

```csharp
public sealed class AppConfig
{
    private static readonly Lazy<AppConfig> _lazy = new(() => new AppConfig());
    public static AppConfig Instance => _lazy.Value;
    private AppConfig() { /* load */ }
    public string Env { get; private set; } = "dev";
}
```

## 6. Ví dụ

### Cơ bản

```csharp
public sealed class Logger
{
    public static Logger Instance { get; } = new();
    private Logger() { }
    public void Info(string m) => Console.WriteLine(m);
}
```

### Trung cấp (Lazy thread-safe)

Dùng `Lazy<T>` như mục 5 — tránh double-check lock tự viết sai.

### Nâng cao / Unity

```csharp
public class AudioHub : MonoBehaviour
{
    public static AudioHub? Instance { get; private set; }
    void Awake()
    {
        if (Instance != null && Instance != this) { Destroy(gameObject); return; }
        Instance = this;
        DontDestroyOnLoad(gameObject);
    }
}
```

Vẫn nên expose `IAudioPlayer` và inject nơi test được.

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Hai instance Unity | Awake race / scene reload | Guard + DDOL cẩn thận |
| Khó unit test | Phụ thuộc static | Interface + reset/hook test |
| God Singleton | Nhồi mọi API | Tách service (SRP) |

## 8. Gỡ lỗi

1. Log `GetHashCode()` instance ở nhiều chỗ — khác nhau → không còn singleton.
2. Test parallel: race trên init tự viết.
3. Tìm `.Instance` trong domain logic → candidate DIP.

## 9. Best practices

- `sealed` + private ctor.
- Prefer DI scope “single instance” hơn static Singleton khi có container.
- Không dùng Singleton chỉ vì “tiện gọi”.

## 10. Bài tập

**Bài 1** — Viết `Lazy` Singleton `IdGenerator` tăng long.  
**Bài 2** — Refactor code gọi `Db.Instance` thành ctor inject `IDb`.  
**Bài 3** — Liệt kê 3 lý do Singleton xấu cho `CombatCalculator`.  
**Bài 4** — Unity: so sánh static Instance vs ScriptableObject config.

## 11. Gợi ý

- Bài 2: composition root tạo một `Db` rồi inject.  
- Bài 3: test, parallel combat, ẩn phụ thuộc.

## 12. Đáp án

```csharp
public sealed class IdGenerator
{
    private static readonly Lazy<IdGenerator> Lazy = new(() => new IdGenerator());
    public static IdGenerator Instance => Lazy.Value;
    private long _n;
    private IdGenerator() { }
    public long Next() => Interlocked.Increment(ref _n);
}
```

## 13. Đáp án thay thế

`static class` thuần utility không state — không cần Singleton. Hoặc Ambient Context có kiểm soát (hiếm khi cần).

## 14. Thử thách

Cấm `.Instance` trong use case project chung — chỉ composition root được biết concrete lifetime.

## 15. Ứng dụng thực tế

- Log framework hubs (vẫn tranh luận)
- Hardware handle duy nhất
- Tránh trong domain thuần

## 16. Liên hệ Unity

- Audio / Input / SceneLoader thường bị Singleton hóa
- Capstone: ưu tiên DI; Singleton tối thiểu cho bootstrap

## 17. Kiểm tra kiến thức

1. Singleton đảm bảo gì? **Một instance + global access.**  
2. Quan hệ với DIP? **Thường xung đột nếu dùng static khắp nơi.**  
3. `Lazy<T>` giúp gì? **Init thread-safe lười.**  
4. God Singleton xấu vì? **Nhiều trách nhiệm + coupling.**  
5. Thay thế tốt hơn? **DI lifetime singleton/scoped.**
