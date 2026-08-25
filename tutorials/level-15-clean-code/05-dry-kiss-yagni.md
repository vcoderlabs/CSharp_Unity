# Chương 5 — DRY · KISS · YAGNI

## 1. Mục tiêu học

- Hiểu **DRY**, **KISS**, **YAGNI** và khi nào **không** áp dụng máy móc
- Phân biệt trùng lặp *ngẫu nhiên* vs trùng *cùng kiến thức*
- Tránh over-engineering “cho tương lai tưởng tượng”
- Cân bằng ba nguyên tắc khi chúng xung đột

## 2. Điều kiện tiên quyết

- Chương 1–4
- Đã từng copy-paste và từng viết abstraction “thông minh” quá sớm

## 3. Khái niệm

| Nguyên tắc | Nghĩa ngắn | Câu hỏi tự hỏi |
|------------|------------|----------------|
| **DRY** — Don't Repeat Yourself | Mỗi *mảnh kiến thức* có một đại diện duy nhất | Hai chỗ này sẽ **luôn** đổi cùng nhau? |
| **KISS** — Keep It Simple, Stupid | Ưu tiên giải pháp đơn giản đủ dùng | Có cách nào ít moving parts hơn không? |
| **YAGNI** — You Aren't Gonna Need It | Đừng xây thứ chưa có nhu cầu thật | Feature/abstraction này có request/test hiện tại không? |

### DRY sai cách

Hai đoạn giống nhau vì **tình cờ** (export CSV và render HTML đều có `foreach`) — gộp sớm tạo coupling giả.

### DRY đúng cách

Công thức thuế xuất hiện ở checkout, invoice, report — **một** `TaxCalculator`.

## 4. Mô hình tư duy

```text
Thấy copy-paste
    │
    ├─ Cùng rule nghiệp vụ? → DRY (extract)
    └─ Chỉ giống cú pháp? → có thể để riêng (Rule of Three)

Muốn thêm “framework cấu hình đa tenant”
    │
    └─ Chưa có tenant 2? → YAGNI; giữ hard-code rõ ràng

Giữa Strategy + Factory + DI container cho 1 if
    │
    └─ KISS: dùng if/switch đến khi đau thật
```

**Rule of Three (thực dụng):** lần trùng thứ 3 cùng ý nghĩa → mới extract mạnh.

## 5. Cú pháp

**DRY — trước/sau:**

```csharp
// Trước — kiến thức "VIP = 10%" lặp
total1 = price * 0.9m;
total2 = price * 0.9m;

// Sau
const decimal VipFactor = 0.9m;
decimal ApplyVip(decimal price) => price * VipFactor;
```

**KISS — trước/sau:**

```csharp
// Trước — overkill
IOrderProcessor processor = AbstractOrderProcessorFactory
    .Create(OrderProcessorType.Standard)
    .WithPipeline(new ValidationStep(), new PersistStep());

// Sau
public void Place(Order order)
{
    Validate(order);
    _repo.Save(order);
}
```

**YAGNI — trước/sau:**

```csharp
// Trước — plugin engine chưa ai dùng
public interface IPricingPlugin { ... }
public class PluginLoader { /* reflection */ }

// Sau — đủ hôm nay
public decimal ApplyDiscount(decimal price) => price * 0.95m;
```

## 6. Ví dụ

### Cơ bản — DRY hằng số / hàm

Magic number `86400` rải rác → `SecondsPerDay` hoặc `TimeSpan.FromDays(1)`.

### Trung cấp — Xung đột DRY vs YAGNI

Bạn đoán sau này có 5 loại discount → viết `IDiscountStrategy` + DI + config JSON.  
Hiện chỉ có 1 loại → **YAGNI + KISS**: một hàm/`PercentDiscount`. Khi loại 2–3 xuất hiện và khác nhau thật → mới strategy.

### Nâng cao — Trùng lặp có chủ đích (two-phase)

Layer API DTO và Domain Entity *cố ý* tách — map hơi trùng field nhưng **kiến thức khác** (contract ngoài vs model trong). Ép một type dùng chung có thể vi phạm boundary (sẽ gặp lại ở Clean Architecture L16–18).

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| “Utils” chứa 40 hàm không liên quan | DRY quá đà | Tách theo chủ đề |
| Abstraction 5 lớp cho 1 if | Sợ không DRY/KISS | Xóa lớp thừa |
| Config cho tương lai | YAGNI bị quên | Ship đơn giản; mở rộng khi có case |
| Copy logic thuế vì “không kịp” | Nợ | Ticket trả nợ + test |
| Micro-optimize đọc kém | Không KISS | Ưu tiên rõ ràng trước |

## 8. Gỡ lỗi

1. Đổi rule một chỗ quên chỗ kia → thiếu DRY đúng nghĩa.
2. Không ai hiểu abstract factory → vi phạm KISS; đơn giản hóa.
3. Feature flag chết / code path không bao giờ bật → YAGNI nợ; xóa hoặc hoàn thiện có chủ đích.
4. Hỏi pair: “Giải pháp này giải quyết đau *hiện tại* nào?”

## 9. Best practices

- DRY cho **kiến thức**, không cho **ký tự giống nhau**.
- KISS: code đọc được thắng code “thông minh”.
- YAGNI: xây khi có bằng chứng (ticket, khách hàng, test thứ hai).
- Khi xung đột: ưu tiên **đúng hành vi + dễ đọc**; rồi mới nén duplication.
- Refactor về phía đơn giản cũng là thắng lợi.

## 10. Bài tập

**Bài 1** — Hai hàm `PrintInvoice` và `PrintReceipt` giống 80% nhưng khác header. Có DRY ngay không? Giải thích.

**Bài 2** — Viết lại đơn giản:

```csharp
IComparer<Player> cmp = new PlayerComparerFactory()
    .Create(SortMode.ByScoreThenName);
list.Sort(cmp);
```

thành KISS tương đương (giả sử chỉ sort theo score).

**Bài 3** — Liệt kê 3 thứ bạn từng viết theo YAGNI lẽ ra không cần.

**Bài 4** — Công thức XP `level * 100 + 50` ở 3 file. DRY đúng cách.

## 11. Gợi ý

- Bài 1: có thể extract phần *body chung* nếu cùng kiến thức; giữ header riêng. Đừng gộp thành một hàm với 6 cờ bool ngay.
- Bài 2: `list.Sort((a, b) => b.Score.CompareTo(a.Score));`
- Bài 4: `XpTable.RequiredFor(level)` một chỗ.

## 12. Đáp án

**Bài 1:** Không máy móc gộp hết. Nếu phần tính dòng tiền giống nhau → extract `BuildLineItems`. Header/footer khác → để riêng. Tránh `Print(doc, bool isInvoice, bool showTax, ...)`.

**Bài 2:**

```csharp
list.Sort((a, b) => b.Score.CompareTo(a.Score));
```

**Bài 4:**

```csharp
public static class XpRequirements
{
    public static int ForLevel(int level) => level * 100 + 50;
}
```

## 13. Đáp án thay thế

Bài 2 giữ `IComparer` nếu cần reuse nhiều nơi — vẫn KISS hơn factory phức tạp. Bài 1 dùng template method khi thật sự có khung cố định.

## 14. Thử thách

Tìm một abstraction trong project học của bạn và **xóa** nếu chỉ có 1 implementation — đo xem code có dễ đọc hơn không (YAGNI retrospective).

## 15. Ứng dụng thực tế

- Startup: YAGNI + KISS để ship
- Domain tài chính: DRY rule tiền tệ/thuế rất quan trọng
- Platform team: cẩn thận DRY xuyên service tạo coupling

## 16. Liên hệ Unity

- Đừng viết “ultimate GameFramework” trước game đầu
- ScriptableObject config: đủ field đang dùng (YAGNI)
- Trùng `SerializeField` setup: extract prefab/variant thay vì inheritance sâu ngay

## 17. Kiểm tra kiến thức

1. DRY nhắm vào gì?  
   **Đáp án:** Không lặp *kiến thức* / quy tắc — không phải mọi đoạn giống chữ.

2. Rule of Three gợi ý gì?  
   **Đáp án:** Đợi trùng ý nghĩa lần thứ ba rồi extract mạnh.

3. YAGNI khuyên gì?  
   **Đáp án:** Đừng xây thứ chưa cần.

4. KISS xung đột với “architecture đẹp” thì ưu tiên?  
   **Đáp án:** Đơn giản đủ đúng nhu cầu hiện tại; kiến trúc phức tạp khi có lý do.

5. Hai DTO giống field có bắt buộc một class không?  
   **Đáp án:** Không — có thể cố ý tách theo boundary.
