# Level 6 — Exceptions & Error Handling (~12 giờ)

Level này dạy bạn **xử lý lỗi đúng cách** trong C#: `try`/`catch`/`finally`, custom exception, hierarchy, exception filter (`when`), biết **khi nào nên throw**, và thiết kế **logging** để debug production.

**Điều kiện:** Đã hoàn thành [Level 5 — Generics](../level-05-generics/) (và các level trước). Biết class/inheritance giúp hiểu exception hierarchy nhanh hơn.

**Tiếp theo:** [Level 7 — Delegates, Lambda & Events](../level-07-delegates-events/) (nền tảng UnityEvents — rất quan trọng cho Unity).

---

## Mục tiêu cấp độ

Sau Level 6 bạn sẽ:

- Dùng `try` / `catch` / `finally` / `using` đúng ngữ cảnh
- Tạo custom exception và tổ chức hierarchy rõ ràng
- Dùng exception filter (`catch when`) để lọc theo điều kiện
- Phân biệt lỗi có thể phục hồi vs lỗi lập trình; biết khi nào *không* throw
- Thiết kế chiến lược log (level, context, không log secret)
- Xây **Logging System** với hierarchy exception + file logger

---

## Cảnh báo xuyên suốt Level 6

> **Exception ≠ luồng điều khiển bình thường.**  
> Đừng dùng `throw`/`catch` thay `if` cho trường hợp thường gặp (vd: tìm key trong dictionary — dùng `TryGetValue`). Exception dành cho tình huống **bất thường** mà caller không dễ kiểm tra trước.

---

## 6 chương + 1 project

| # | File | Nội dung | ~Giờ |
|---|------|----------|------|
| 1 | [01-try-catch-finally.md](./01-try-catch-finally.md) | `try` / `catch` / `finally`, `using` | 2 |
| 2 | [02-custom-exceptions.md](./02-custom-exceptions.md) | Tạo exception riêng | 1.5–2 |
| 3 | [03-exception-hierarchy.md](./03-exception-hierarchy.md) | Cây kế thừa, bắt đúng tầng | 1.5–2 |
| 4 | [04-exception-filters.md](./04-exception-filters.md) | `catch when (...)` | 1–1.5 |
| 5 | [05-when-to-throw.md](./05-when-to-throw.md) | Khi nào nên / không nên throw | 1.5–2 |
| 6 | [06-logging-strategy.md](./06-logging-strategy.md) | Level log, context, best practices | 1.5–2 |
| — | [project-logging-system.md](./project-logging-system.md) | Custom hierarchy + file logger | 3–4 |

**Tổng ước lượng: ~12 giờ**

---

## Cách học đề xuất

1. Chương 1: cố tình gây lỗi file/parse, quan sát stack trace trước khi bắt.
2. Chương 2–3: vẽ cây exception domain (App → Domain → cụ thể) rồi mới code.
3. Chương 4: so sánh `if` trong catch vs `when` — hiểu stack unwind.
4. Chương 5: viết checklist “có nên throw?” cho 5 tình huống thực tế.
5. Chương 6 + Project: log có level + message + exception; không ghi mật khẩu.

---

## Checklist hoàn thành Level 6

- [ ] Viết được `try`/`catch`/`finally` và giải thích thứ tự chạy
- [ ] Tạo custom exception với message + inner exception
- [ ] Bắt exception theo hierarchy (cụ thể trước, tổng quát sau)
- [ ] Dùng được `catch when`
- [ ] Giải thích được khi nào dùng `Try*` / Result thay vì throw
- [ ] Log được Error/Warning/Info với context hữu ích
- [ ] Hoàn thành project Logging System
- [ ] Trả lời đúng ≥ 4/5 câu **Kiểm tra kiến thức** mỗi chương

Khi xong checklist → chuyển sang **Level 7** (Delegates & Events — critical cho Unity).
