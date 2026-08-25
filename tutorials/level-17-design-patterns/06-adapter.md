# Chương 6 — Adapter

## 1. Mục tiêu học

- Bọc API lệch contract thành interface client cần
- Phân biệt class adapter vs object adapter
- Dùng ở biên hệ thống (third-party, legacy, Unity API)

## 2. Điều kiện tiên quyết

- Interface, composition
- DIP (L16)

## 3. Khái niệm

**Adapter** chuyển đổi interface của class có sẵn sang interface khác mà client expect — giống ổ cắm chuyển.

Không thêm business mới; chỉ **dịch**.

## 4. Mô hình tư duy

```text
Client → ITarget ← Adapter → Adaptee (API lạ)
```

## 5. Cú pháp

```csharp
public interface ILogger { void Info(string m); }

public sealed class NLogAdapter : ILogger
{
    private readonly NLog.Logger _inner;
    public NLogAdapter(NLog.Logger inner) => _inner = inner;
    public void Info(string m) => _inner.Info(m);
}
```

## 6. Ví dụ

### Cơ bản

```csharp
public sealed class LegacyXmlService { public string FetchXml() => "<hp>10</hp>"; }

public interface IHealthSource { int GetHp(); }

public sealed class XmlHealthAdapter : IHealthSource
{
    private readonly LegacyXmlService _legacy;
    public XmlHealthAdapter(LegacyXmlService legacy) => _legacy = legacy;
    public int GetHp() => /* parse xml */ 10;
}
```

### Trung cấp

Adapter đồng thời map error code legacy → exception domain.

### Nâng cao / Unity

```csharp
public interface IInputAxis { float Horizontal { get; } }

public sealed class UnityInputAdapter : IInputAxis
{
    public float Horizontal => Input.GetAxisRaw("Horizontal");
}
// Sau này đổi Input System package — chỉ sửa adapter
```

## 7. Lỗi thường gặp

| Hiện tượng | Cách xử lý |
|------------|------------|
| Adapter chứa rule nghiệp vụ | Đẩy vào use case |
| Adapter leak type adaptee | Không expose adaptee ra ngoài |
| “Adapter” chỉ rename method | Vẫn OK nếu đúng mục tiêu biên |

## 8. Gỡ lỗi

Test client với fake `ITarget`; integration test adapter với adaptee thật/stub.

## 9. Best practices

- Adapter mỏng.  
- Đặt ở Infrastructure layer (L18).  
- Một adaptee — một hoặc vài adapter rõ ràng.

## 10. Bài tập

**Bài 1** — Adapt `DateTime.Now` thành `IClock`.  
**Bài 2** — Adapt `HttpClient` JSON thành `IWeatherClient`.  
**Bài 3** — Adapt Unity `PlayerPrefs` thành `IKeyValueStore`.  
**Bài 4** — Khác Facade? (chương 8)

## 11. Gợi ý

`IClock` giúp test freeze time.

## 12. Đáp án

```csharp
public interface IClock { DateTime UtcNow { get; } }
public sealed class SystemClock : IClock
{
    public DateTime UtcNow => DateTime.UtcNow;
}
```

## 13. Đáp án thay thế

Source generator / AutoMapper cho DTO map — khác mục tiêu nhưng cùng “dịch hình dạng”.

## 14. Thử thách

Adapt SDK thanh toán third-party về `IPaymentGateway` project chung.

## 15. Ứng dụng thực tế

- Anti-Corruption Layer (DDD)
- Legacy COM/XML bridges

## 16. Liên hệ Unity

- New Input System vs legacy Input  
- Platform IAP SDKs  
- File save WebGL vs standalone

## 17. Kiểm tra kiến thức

1. Adapter làm gì? **Đổi interface cho khớp client.**  
2. Object adapter dùng? **Composition bọc adaptee.**  
3. Khác Decorator? **Decorator thêm hành vi cùng interface; Adapter đổi interface.**  
4. Nên chứa business? **Không.**  
5. Liên hệ DIP? **High-level phụ thuộc ITarget; adapter là detail.**
