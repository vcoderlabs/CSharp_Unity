# Chương 18 — Khi nào KHÔNG nên dùng pattern

## 1. Mục tiêu học

- Nhận diện **over-engineering** và “Pattern Pride”
- Áp dụng YAGNI/KISS (L15) lên quyết định pattern
- Review code L16–17: xóa abstraction thừa
- Chuẩn bị tư duy sạch trước Level 18 Architecture

## 2. Điều kiện tiên quyết

- Đã học đại diện Creational/Structural/Behavioral
- Level 15 DRY/KISS/YAGNI
- Level 16 SOLID (đặc biệt OCP vs YAGNI)

## 3. Khái niệm

Pattern sinh ra để giải **lực cản** (variability, decoupling, lifecycle). Nếu lực cản **chưa tồn tại**, pattern chỉ thêm indirection, file, và cognitive load.

| Dấu hiệu lạm dụng | Thực tế thường đủ |
|-------------------|-------------------|
| Abstract Factory cho 1 product | `new` / Simple Factory |
| 5 Strategy class 1 dòng | `switch` nhỏ hoặc `Func` |
| Mediator + EventBus + Facade chồng | Một use case method |
| Singleton mọi service | DI bình thường |
| Decorator 8 lớp | 2 hàm rõ tên |

## 4. Mô hình tư duy

```text
1. Có đau thật? (thay đổi lặp, test khó, coupling)
2. Giải pháp đơn giản nhất đủ chưa?
3. Pattern có tên khớp lực cản?
4. Chi phí files/indirection có đáng?
5. Team đọc được không?
```

## 5. Cú pháp

Không có cú pháp — có **checklist từ chối**:

```csharp
// Đủ tốt:
int Damage(int atk, int def) => Math.Max(0, atk - def);

// Chỉ nâng Strategy khi có ≥2–3 công thức + đổi runtime / test riêng
```

## 6. Ví dụ

### Cơ bản — “Factory của Factory”

```csharp
// Bad: IAbstractFactoryProviderFactory — không ai nhờ bạn
// Good: một EnemyFactory hoặc thậm chí prefab reference
```

### Trung cấp — premature OCP

Viết plugin architecture ngày 1 cho tool nội bộ 2 màn hình → chậm ship. Viết thẳng; extract khi biến thể thứ 2 xuất hiện (**Rule of Three** mềm).

### Nâng cao / Unity

Đừng port cả Java enterprise stack vào một casual scene.  
MMORPG **cần** Observer/State/Command/Pool — nhưng inventory CRUD đơn giản không cần Abstract Factory + CQRS + Event Sourcing ngày đầu.

## 7. Lỗi thường gặp

| Hiện tượng | Cách xử lý |
|------------|------------|
| “Clean” = nhiều interface | Interface tại boundary thật |
| Copy pattern từ blog | Khớp problem statement trước |
| Refactor vàng mãi | Timebox; đo giá trị |
| Sợ `new` | `new` concrete trong composition root OK |

## 8. Gỡ lỗi

1. Đếm tầng gọi để làm một việc — >3–4 không domain → nghi.  
2. Hỏi “xóa pattern này mất gì?” — nếu “không mất” → xóa.  
3. Onboarding: member mới giải thích được trong 2 phút?

## 9. Best practices

- Prefer **design smell fix** hơn gắn nhãn pattern.  
- Document *vì sao* dùng pattern trong PR/README module.  
- Architecture (L18) chọn ranh giới trước; pattern là chi tiết bên trong.  
- Capstone: chỉ pattern trong milestone yêu cầu + chỗ đau thật.

## 10. Bài tập

**Bài 1** — Review 1 file tự viết: liệt kê pattern; đánh dấu Keep/Remove.  
**Bài 2** — Viết lại Abstract Factory 1-product thành Simple Factory/`new`.  
**Bài 3** — Tìm God Singleton trong project mẫu — đề xuất DI.  
**Bài 4** — Viết ADR ngắn (5–7 dòng): “Dùng State cho Quest vì…”.

## 11. Gợi ý

ADR: Context → Decision → Consequences. Pattern name chỉ là Decision.

## 12. Đáp án

Ví dụ ADR:

```text
Context: Quest có bước và transition bất hợp lệ gây bug.
Decision: State pattern (NotStarted/Active/Completed).
Consequences: thêm vài class; illegal transition rõ; UI subscribe event Completed.
```

Remove ví dụ: `IDamageStrategy` chỉ một class `Default` suốt 6 tháng → inline method.

## 13. Đáp án thay thế

Giữ abstraction nhưng flatten: một file partial, ít type — vẫn đơn giản hơn tree pattern.

## 14. Thử thách

Trên project chung L15–18: xóa ≥1 abstraction thừa; thêm ≥1 pattern ở chỗ đo được đau (test khó / switch phình).

## 15. Ứng dụng thực tế

- Code review culture  
- “Boring technology” — chọn đơn giản có chủ đích  
- Chi phí bảo trì > chi phí viết lần đầu

## 16. Liên hệ Unity

- Prefab + SerializeField thường thắng hierarchy pattern sớm  
- Pool/Observer/State/Command: **nên** có khi scale combat/UI  
- ECS/Patterns mạng: học khi profiling/product đòi hỏi (L19+)

## 17. Kiểm tra kiến thức

1. Pattern khi nào dùng? **Khi có lực cản thiết kế rõ.**  
2. Over-engineering là gì? **Abstraction vượt nhu cầu hiện tại.**  
3. Rule of Three gợi ý? **Đợi lặp lại trước khi trừu tượng hóa mạnh.**  
4. `new` trong composition root có xấu không? **Thường chấp nhận được.**  
5. Quan hệ YAGNI? **Đừng xây điểm mở rộng “phòng khi” vô căn cứ.**
