# Level 15 — Clean Code (~10 giờ)

Level này dạy bạn viết C# **dễ đọc, dễ sửa, dễ test**: đặt tên, thiết kế hàm/class, nhận diện **code smell**, **refactor** an toàn, áp dụng **DRY / KISS / YAGNI**, và cân bằng **coupling / cohesion**.

**Điều kiện:** Đã hoàn thành Level 1–8 (nên có [Level 14 — Testing](../level-14-testing/) để refactor có “lưới an toàn”).

**Tiếp theo:** Level 16+ (architecture). Project lớn **Clean Architecture Application** là **project chung Level 15–18** — ở Level 15 chỉ cần nắm clean code + phác outline; implement sâu ở các level sau.

---

## Mục tiêu cấp độ

Sau Level 15 bạn sẽ:

- Đặt tên biến/hàm/class nói lên ý định
- Giữ hàm ngắn, class có một lý do để đổi (SRP thực dụng)
- Nhận diện smell phổ biến và hướng refactor
- Refactor từng bước nhỏ (ideal: có test)
- Áp dụng DRY/KISS/YAGNI **đúng mức** — không over-engineer
- Giải thích coupling vs cohesion bằng ví dụ C#

---

## Cảnh báo xuyên suốt Level 15

> **Clean Code ≠ viết càng nhiều pattern càng tốt.**  
> Code sạch phục vụ người đọc (kể cả bạn sau 3 tháng). YAGNI: đừng xây framework khi chỉ cần một hàm. KISS: giải pháp đơn giản thắng giải pháp thông minh khó hiểu.

---

## 6 chương + project chung (outline)

| # | File | Nội dung | ~Giờ |
|---|------|----------|------|
| 1 | [01-naming.md](./01-naming.md) | Đặt tên | 1.5–2 |
| 2 | [02-function-class-design.md](./02-function-class-design.md) | Hàm & class | 1.5–2 |
| 3 | [03-code-smells.md](./03-code-smells.md) | Code smells | 1.5–2 |
| 4 | [04-refactoring.md](./04-refactoring.md) | Refactoring | 1.5–2 |
| 5 | [05-dry-kiss-yagni.md](./05-dry-kiss-yagni.md) | DRY / KISS / YAGNI | 1–1.5 |
| 6 | [06-coupling-cohesion.md](./06-coupling-cohesion.md) | Coupling & cohesion | 1–1.5 |

**Project chung L15–18 — Clean Architecture Application** (outline nhẹ ở cuối README).  
**Tổng ước lượng Level 15 bài học: ~10 giờ** (chưa kể implement full architecture ở L16–18).

---

## Project chung L15–18 (outline)

Tên: **Clean Architecture Application** (ví dụ: quản lý quest/inventory đơn giản hoặc order/pricing — bạn chọn một domain).

Các vòng sau (chi tiết ở level tương ứng):

| Level | Tập trung |
|-------|-----------|
| **15** | Đặt tên, tách hàm/class, bỏ smell; domain model đọc được |
| **16** | [SOLID](../level-16-solid/) trên cùng codebase |
| **17** | [Design Patterns](../level-17-design-patterns/) có chọn lọc |
| **18** | [Architecture](../level-18-architecture/) + [project chung](../level-18-architecture/project-clean-architecture-app.md) |

Ở **Level 15**, chỉ cần:

1. Tạo solution trống hoặc tái sử dụng idea từ TestedApp (L14).
2. Viết 1–2 use case với naming + hàm sạch.
3. Ghi `docs/clean-architecture-outline.md` ½–1 trang: Entity / Use Case / Interface / Infrastructure sẽ tách thế nào (chưa bắt buộc implement hết).

---

## Cách học đề xuất

1. Mỗi chương: lấy **code xấu** → viết lại **code sạch** (before/after).
2. Có test (L14) trước khi refactor chương 4.
3. Chương 5: cố tình **không** DRY hai chỗ gần giống nếu abstraction đắt hơn.
4. Chương 6: vẽ mũi tên phụ thuộc giữa class.
5. Outline project chung — đừng implement cả Clean Architecture trong một cuối tuần.

---

## Checklist hoàn thành Level 15

- [ ] Đổi tên được đoạn code “bí ẩn” thành tên có ý định
- [ ] Tách được hàm dài / class làm nhiều việc
- [ ] Liệt kê ≥ 5 smell và cách xử lý
- [ ] Refactor một module nhỏ với các bước an toàn
- [ ] Giải thích DRY vs “trùng ngẫu nhiên”; KISS; YAGNI
- [ ] Phân biệt high cohesion / low coupling bằng ví dụ
- [ ] Có outline Clean Architecture Application (L15–18)
- [ ] ≥ 4/5 câu **Kiểm tra kiến thức** mỗi chương

Khi xong → sẵn sàng Level 16 (và tiếp tục project chung).
