# Level 7 — Delegates, Lambda & Events (~20 giờ)

Level này là **nền tảng callback và phản ứng sự kiện** trong C# — và là cầu nối trực tiếp tới **UnityEvents / C# events** trong Unity. Học kỹ; không lướt.

**Điều kiện:** Đã hoàn thành [Level 5 — Generics](../level-05-generics/). Nên hoàn thành hoặc song song [Level 6 — Exceptions](../level-06-exceptions/).

**Tiếp theo:** [Level 8 — LINQ](../level-08-linq/) (dựa mạnh vào `Func` / lambda).

---

## Mục tiêu cấp độ

Sau Level 7 bạn sẽ:

- Khai báo và dùng **delegate** như kiểu “con trỏ hàm có an toàn kiểu”
- Thành thạo `Action`, `Func`, `Predicate`
- Viết **lambda** và anonymous method; hiểu **closure** (capture biến)
- Thiết kế **event** / event handler đúng encapsulation (`event` vs public delegate field)
- Hiểu **multicast**: `+=` / `-=`, thứ tự gọi, exception trong chuỗi
- Xây **Event-driven Notification System**
- Nhìn ra mối liên hệ **C# event ↔ UnityEvent** trong Inspector

---

## Cảnh báo xuyên suốt Level 7

> **`event` ≠ chỉ là multicast delegate công khai.**  
> Từ khóa `event` giới hạn bên ngoài chỉ `+=`/`-=`, không cho gán `=` xóa hết subscriber hay gọi trực tiếp từ ngoài. Trong Unity, `UnityEvent` là lớp serializable riêng — tư duy subscribe/publish giống nhau, API khác.

> **Closure capture:** lambda bắt biến vòng `for` cần hiểu rõ (bản sao / scope) — bug kinh điển.

---

## 6 chương + 1 project

| # | File | Nội dung | ~Giờ |
|---|------|----------|------|
| 1 | [01-delegates.md](./01-delegates.md) | Delegate là gì, khai báo, Invoke | 3 |
| 2 | [02-action-func-predicate.md](./02-action-func-predicate.md) | `Action` / `Func` / `Predicate` | 3 |
| 3 | [03-lambda-anonymous.md](./03-lambda-anonymous.md) | Lambda & anonymous method | 3 |
| 4 | [04-closure.md](./04-closure.md) | Closure, capture, pitfall vòng lặp | 2–3 |
| 5 | [05-events.md](./05-events.md) | `event`, handler, pattern chuẩn | 4 |
| 6 | [06-multicast-delegates.md](./06-multicast-delegates.md) | `+=`/`-=`, GetInvocationList | 2–3 |
| — | [project-notification-system.md](./project-notification-system.md) | Event-driven Notification System | 4–5 |

**Tổng ước lượng: ~20 giờ**

---

## Cách học đề xuất

1. Chương 1: viết callback không lambda — method group trước.
2. Chương 2: thay mọi delegate tự viết bằng Action/Func nếu khớp chữ ký.
3. Chương 3–4: làm bài closure trong `for` / `foreach` đến khi giải thích được.
4. Chương 5: so sánh field `Action` public vs `event Action` — thử gán `=` từ ngoài.
5. Chương 6 + Project: publisher/subscriber; nhớ `-=` tránh leak (rất quan trọng Unity).
6. Mỗi chương đọc mục **Liên hệ Unity** — đặc biệt UnityEvent.

---

## Checklist hoàn thành Level 7

- [ ] Khai báo delegate và `Invoke` / gọi như hàm
- [ ] Chọn đúng Action vs Func vs Predicate
- [ ] Viết lambda một dòng và khối `{ }`
- [ ] Giải thích được closure + fix bug biến vòng lặp
- [ ] Dùng `event` với `EventHandler` / `EventHandler<T>`
- [ ] Subscribe/unsubscribe multicast an toàn
- [ ] Hoàn thành Notification System
- [ ] Giải thích được điểm giống/khác UnityEvent
- [ ] Trả lời đúng ≥ 4/5 câu **Kiểm tra kiến thức** mỗi chương

Khi xong checklist → chuyển sang **Level 8 (LINQ)** hoặc áp dụng ngay events vào prototype Unity.
