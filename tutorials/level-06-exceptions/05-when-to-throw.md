# Chương 5 — Khi nào nên / không nên throw

## 1. Mục tiêu học

- Phân biệt **lỗi lập trình**, **lỗi vận hành**, **kết quả nghiệp vụ thất bại**
- Chọn giữa: throw exception, `Try*` pattern, `Result`/`bool` return
- Biết quy tắc “fail fast” vs “soft fail”
- Áp dụng checklist trước khi `throw` trong API công khai

## 2. Điều kiện tiên quyết

- Chương 1–4
- Level 4: `TryGetValue`, `TryParse` đã gặp
- Level 5: có thể dùng `Result<T>` generic (ôn lại)

## 3. Khái niệm

| Tình huống | Hướng xử lý điển hình |
|------------|------------------------|
| Bug: null không được phép, index sai do logic | Fix code; có thể `ArgumentNullException` để fail fast |
| Input user sai (parse form) | Validate / `TryParse` — **không** dùng exception làm validation chính |
| Không tìm thấy entity (thường gặp) | `null`, `bool TryGet`, `Option`/`Result` |
| Vi phạm bất biến nghiêm trọng (corrupt state) | Throw |
| I/O / mạng thất bại | Throw hoặc Result tùy layer; luôn log |
| Điều khiển luồng (loop break) | **Không** dùng exception |

**Chi phí:** exception chậm hơn nhánh `if` khi dùng đường “nóng” và xảy ra thường xuyên. Đừng thiết kế API ném hàng nghìn lần/giây trong game loop.

### Try pattern

```csharp
bool TryGetPlayer(string id, out Player? player);
int.TryParse(s, out int value);
Dictionary.TryGetValue(key, out var v);
```

Caller quyết định khi thất bại mà không cần try/catch.

## 4. Mô hình tư duy

```text
Câu hỏi trước khi throw:
1. Caller có thể kiểm tra trước một cách rẻ và rõ không?
2. Thất bại này có phải “bất thường” với hợp đồng API không?
3. Có ai cần catch riêng / có stack trace có ích không?
4. Có xảy ra trên hot path không?

Có / bất thường / cần stack → throw
Thường gặp / hot path / caller tự xử lý → Try* hoặc Result
```

## 5. Cú pháp

```csharp
// Fail fast tham số
public void SetHp(int hp)
{
    ArgumentOutOfRangeException.ThrowIfNegative(hp); // .NET 8
    Hp = hp;
}

// Try pattern
public bool TryFind(string id, out Item? item)
{
    return _map.TryGetValue(id, out item);
}

// Result nhẹ
public Result<Order> PlaceOrder(...)
{
    if (!stock.Has(sku))
        return Result<Order>.Fail("Hết hàng");
    return Result<Order>.Ok(order);
}

// Throw khi bất biến vỡ
public void Complete()
{
    if (Status != Status.Pending)
        throw new InvalidOperationException("Chỉ complete đơn Pending.");
}
```

## 6. Ví dụ

### Cơ bản

```csharp
// Không nên
try { int x = int.Parse(userInput); }
catch { ShowError(); }

// Nên
if (!int.TryParse(userInput, out int x))
{
    ShowError();
    return;
}
```

### Trung cấp

```csharp
public Player GetRequired(string id)
{
    if (!_players.TryGetValue(id, out var p))
        throw new PlayerNotFoundException(id); // API “phải tồn tại”
    return p;
}

public bool TryGet(string id, out Player? p) => _players.TryGetValue(id, out p);
```

Hai API: một nghiêm ngặt, một mềm — caller chọn.

### Nâng cao

Layer biên:

```csharp
// Domain: Result cho nghiệp vụ thường
Result<Payment> Pay(...);

// Infrastructure: I/O bất thường → exception
// Application service: map Result → HTTP 400; map exception infra → 503
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Dùng exception làm `if` | Thói quen Java checked / chưa biết Try | Rewrite Try/Result |
| Nuốt mọi exception trả null | Che lỗi hệ thống | Phân loại; log; chỉ soft-fail lỗi nghiệp vụ |
| Throw trong property getter thường | Side-effect khó đoán | Validate ở setter/method |
| Message mơ hồ `"Error"` | Khó hỗ trợ | Message + property ngữ cảnh |
| Catch rồi empty return | Mất tín hiệu lỗi | Ít nhất log |

## 8. Gỡ lỗi

1. Nếu profiler thấy exception hotspot → đổi Try pattern.
2. Review API: public method có document “throws” không?
3. Đếm: thất bại này % request? > vài % → cân nhắc không throw.

## 9. Best practices

- Public API: nhất quán (hoặc throw, hoặc Try — đừng lẫn lộn vô tổ chức).
- `.NET`: `ArgumentNullException.ThrowIfNull`, `ThrowIfNegative`, …
- Game/Unity hot path: flags, Result, object pool — tránh throw.
- Không dùng exception cho control flow (break nested loops, v.v.).
- Khi throw: message actionable cho developer/ops.

## 10. Bài tập

**Bài 1** — Viết lại đoạn `int.Parse` + catch thành `TryParse`.

**Bài 2** — Class `Vault` có `Get(string key)` throw nếu thiếu, và `TryGet` trả bool.

**Bài 3** — Checklist 5 tình huống (bạn tự đặt) — mỗi cái ghi Throw / Try / Result và lý do 1 câu.

**Bài 4** — Method `Divide` hai phiên bản: `int DivideOrThrow` và `bool TryDivide(..., out int)`.

## 11. Gợi ý

- Bài 2: nội bộ `Dictionary`; Get gọi TryGet rồi throw custom.
- Bài 4: `TryDivide` trả false khi `b == 0`.

## 12. Đáp án

**Bài 1** — Validate bằng TryParse:

```csharp
if (!int.TryParse(userInput, out int x))
{
    Console.WriteLine("Số không hợp lệ");
    return;
}
Console.WriteLine(x);
```

**Bài 2** — Get nghiêm + TryGet mềm:

```csharp
class Vault
{
    private readonly Dictionary<string, string> _data = new();

    public void Set(string key, string value) => _data[key] = value;

    public bool TryGet(string key, out string? value) =>
        _data.TryGetValue(key, out value);

    public string Get(string key)
    {
        if (!TryGet(key, out var value))
            throw new KeyNotFoundException($"Thiếu key '{key}'");
        return value!;
    }
}
```

**Bài 3** — Ví dụ checklist:

```text
1. User nhập tuổi sai format → TryParse (thường gặp)
2. Save file disk full → Throw + log (vận hành bất thường)
3. Tìm item trong túi không có → TryGet / null (thường trong gameplay)
4. hp setter nhận âm do bug AI → Throw ArgumentOutOfRange (fail fast)
5. Không đủ vàng mua item → Result.Fail hoặc bool (nghiệp vụ thường)
```

**Bài 4** — Hai phiên bản Divide:

```csharp
static int DivideOrThrow(int a, int b)
{
    if (b == 0)
        throw new DivideByZeroException();
    return a / b;
}

static bool TryDivide(int a, int b, out int result)
{
    if (b == 0)
    {
        result = 0;
        return false;
    }
    result = a / b;
    return true;
}
```

## 13. Đáp án thay thế

`Result<T>` thay TryDivide cho thông điệp lỗi phong phú hơn `bool`. `Get` có thể trả `T?` thay throw tùy style codebase (nullable reference).

## 14. Thử thách

Thiết kế API inventory: liệt kê 6 thao tác (Add, Remove, Get, Equip, …) và chọn Throw vs Try cho từng cái; viết XML doc `/// <exception>` cho method throw.

## 15. Ứng dụng thực tế

- REST: validation errors → 400 (không phải unhandled 500)
- Library: `Try*` cho parsing; throw cho misuse API
- Batch: skip record lỗi nghiệp vụ; abort khi infra chết
- Game design: “miss attack” không phải exception

## 16. Liên hệ Unity

- `GetComponent` trả null — kiểm null, không expect exception
- `Instantiate` fail hiếm — có thể log error
- **Tuyệt đối tránh** throw trong `Update`/`FixedUpdate` mỗi frame
- Addressables: handle failure callback thay spam exception
- Editor tools: throw OK (fail fast khi import sai)

## 17. Kiểm tra kiến thức

1. User gõ sai số — nên Parse+catch hay TryParse?  
   **Đáp án:** TryParse (hoặc validate), không dùng exception làm validation chính.

2. Try pattern trả gì khi thất bại?  
   **Đáp án:** Thường `false` + `out` default; không throw.

3. Vì sao exception trên hot path xấu?  
   **Đáp án:** Chi phí cao, khó đoán control flow, spam log.

4. `ArgumentNullException` phù hợp khi nào?  
   **Đáp án:** Caller truyền null vi phạm hợp đồng — fail fast.

5. “Miss skill” trong game nên throw không?  
   **Đáp án:** Không — đó là kết quả nghiệp vụ bình thường.
