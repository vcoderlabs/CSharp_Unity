# Chương 17 — Chain of Responsibility

## 1. Mục tiêu học

- Chuỗi handler: mỗi handler xử lý hoặc chuyển tiếp
- Pipeline damage/armor/buff, middleware, validation
- Tránh if-else dài cứng thứ tự trong một class

## 2. Điều kiện tiên quyết

- Linked list / list handlers
- OCP — thêm handler mới
- Decorator khác: chuỗi *cùng request* có thể dừng sớm

## 3. Khái niệm

**Chain of Responsibility:** gửi request dọc theo chuỗi handlers. Handler quyết định xử lý, chuyển tiếp, hoặc vừa xử lý vừa chuyển (pipeline).

## 4. Mô hình tư duy

```text
DamageRequest → ArmorHandler → BuffHandler → CritHandler → Apply
Mỗi mắt xích đọc/chỉnh request rồi Next()
```

## 5. Cú pháp

```csharp
public abstract class DamageHandler
{
    private DamageHandler? _next;
    public DamageHandler SetNext(DamageHandler next) { _next = next; return next; }
    public void Handle(DamageContext ctx)
    {
        Process(ctx);
        _next?.Handle(ctx);
    }
    protected abstract void Process(DamageContext ctx);
}
```

## 6. Ví dụ

### Cơ bản — support tickets

`BillingHandler` → `TechHandler` → `GeneralHandler` — ai `CanHandle` thì xử lý và dừng.

### Trung cấp — damage pipeline

```csharp
public sealed class ArmorHandler : DamageHandler
{
    protected override void Process(DamageContext ctx)
        => ctx.Amount = Math.Max(0, ctx.Amount - ctx.TargetArmor);
}
```

### Nâng cao / Unity

Input: UI consume click → không tới world.  
Network message middleware: auth → rate limit → handler.  
ASP.NET middleware = Chain.

## 7. Lỗi thường gặp

| Hiện tượng | Cách xử lý |
|------------|------------|
| Quên gọi next | Base template gọi next |
| Thứ tự sai | Cấu hình list explicit |
| Handler God | Một concern / handler |

## 8. Gỡ lỗi

Log tên handler + amount sau mỗi bước. Test từng handler + integration chuỗi.

## 9. Best practices

- Prefer `IEnumerable<IHandler>` inject hơn linked list thủ công khi dùng DI.  
- Document short-circuit.  
- Immutable request + cumulative result object.

## 10. Bài tập

**Bài 1** — Logger chain Debug→Info→Error filter level.  
**Bài 2** — Validation chain email/password.  
**Bài 3** — Damage: flat armor → % resist → clamp 0.  
**Bài 4** — Khác Decorator.

## 11. Gợi ý

Decorator thường bọc cùng interface bổ sung hành vi; Chain nhấn *chuyển tiếp / dừng* theo trách nhiệm.

## 12. Đáp án

```csharp
public interface IValidator
{
    void Validate(UserSignup u, IValidator? next);
}

public sealed class EmailValidator : IValidator
{
    public void Validate(UserSignup u, IValidator? next)
    {
        if (string.IsNullOrEmpty(u.Email) || !u.Email.Contains('@'))
            throw new ValidationException("email");
        next?.Validate(u, null); // or pass remaining chain
    }
}
```

(Hoặc list `foreach` validators — vẫn là Chain tinh thần.)

## 13. Đáp án thay thế

`foreach (var h in handlers) h.Process(ctx);` — Chain phẳng, dễ test.

## 14. Thử thách

Combat Capstone: pipeline damage có invuln handler short-circuit.

## 15. Ứng dụng thực tế

- HTTP middleware  
- Exception filters  
- Approval workflows

## 16. Liên hệ Unity

- Damage/heal pipelines MMORPG  
- UI raycast consume  
- Cheat command filters

## 17. Kiểm tra kiến thức

1. Chain cho phép? **Nhiều handler lần lượt xử lý/chuyển.**  
2. Short-circuit? **Dừng không gọi next.**  
3. Liên OCP? **Thêm handler không sửa handler cũ.**  
4. Khác Decorator? **Chuỗi trách nhiệm / dừng vs bọc thêm hành vi.**  
5. DI thường wire thế nào? **List ordered handlers.**
