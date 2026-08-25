# Level 5 — Generics (~15 giờ)

Level này dạy bạn viết code **tái sử dụng an toàn về kiểu** với generic class/method/interface, **constraints**, và **covariance/contravariance** (`out`/`in`).

**Điều kiện:** Đã hoàn thành [Level 4 — Collections](../level-04-collections/) (bạn đã *dùng* `List<T>`, `Dictionary<TKey,TValue>` — giờ học *viết* kiểu đó).

**Tiếp theo:** [Level 6 — Exceptions](../level-06-exceptions/) (có thể song song một phần với L7–L8 theo plan).

---

## Mục tiêu cấp độ

Sau Level 5 bạn sẽ:

- Viết generic class, method, interface
- Dùng `where` constraints: `struct`, `class`, `new()`, interface/base class
- Hiểu `out` (covariance) và `in` (contravariance) trên generic interface/delegate
- Tự xây **GenericStack\<T\>** và **GenericRepository\<T\>**

---

## Cảnh báo xuyên suốt Level 5

> **Generics ≠ `object` + cast.**  
> `List<object>` mất an toàn kiểu và dễ boxing. `List<T>` giữ thông tin kiểu lúc biên dịch — ít bug, thường nhanh hơn với value type.

---

## 3 chương + 1 project

| # | File | Nội dung | ~Giờ |
|---|------|----------|------|
| 1 | [01-generic-basics.md](./01-generic-basics.md) | Generic class / method / interface | 4–5 |
| 2 | [02-generic-constraints.md](./02-generic-constraints.md) | `where`, `struct`, `class`, `new()`, interface | 3–4 |
| 3 | [03-covariance-contravariance.md](./03-covariance-contravariance.md) | `in` / `out` | 2–3 |
| — | [project-generic-library.md](./project-generic-library.md) | `GenericStack<T>`, `GenericRepository<T>` | 4–5 |

**Tổng ước lượng: ~15 giờ**

---

## Cách học đề xuất

1. Chương 1: tự viết lại `Box<T>`, `Swap<T>`, `IRepository<T>` tối giản trước khi xem đáp án.
2. Chương 2: mỗi constraint một ví dụ “không constraint thì lỗi gì”.
3. Chương 3: vẽ mũi tên “Animal ← Dog” và chỗ gán `IEnumerable` / `Action`.
4. Project: viết test console cho stack + repository trước khi thêm tính năng phụ.

---

## Checklist hoàn thành Level 5

- [ ] Viết được generic class và method độc lập
- [ ] Khai báo generic interface và implement
- [ ] Dùng đúng các `where` phổ biến
- [ ] Giải thích được `out` vs `in` bằng ví dụ
- [ ] Hoàn thành GenericStack + GenericRepository
- [ ] Trả lời đúng ≥ 4/5 câu **Kiểm tra kiến thức** mỗi chương

Khi xong checklist → chuyển sang **Level 6** (hoặc song song Delegates/LINQ theo plan).
