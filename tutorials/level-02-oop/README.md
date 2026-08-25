# Level 2 — Object-Oriented Programming (~35 giờ)

Lập trình hướng đối tượng với C#: class/object, encapsulation, inheritance, polymorphism, interface — và biết **khi nào dùng inheritance vs composition**.

**Điều kiện:** Đã hoàn thành [Level 1 — C# Fundamentals](../level-01-csharp-fundamentals/).

**Tiếp theo:** [Level 3 — Value vs Reference](../level-03-value-vs-reference/)

---

## Mục tiêu cấp độ

Sau Level 2 bạn sẽ:

- Thiết kế class với field, property, method, constructor
- Áp dụng encapsulation và access modifiers đúng mục đích
- Phân biệt `static`, `readonly`, `const`
- Dùng inheritance (`base`/`this`) có chủ đích — không lạm dụng
- Viết polymorphism với `virtual` / `override` / `abstract` / `sealed`
- Thiết kế interface như hợp đồng (contract)
- Chọn giữa class, abstract class, interface; ưu tiên **composition over inheritance**
- Hoàn thành project **Banking System**

---

## 8 chương + Project

| # | File | Nội dung | ~Giờ |
|---|------|----------|------|
| 1 | [01-class-object.md](./01-class-object.md) | Class, Object, Instance, Field, Property, Method | 3–4 |
| 2 | [02-constructor-destructor.md](./02-constructor-destructor.md) | Constructor / Finalizer | 3 |
| 3 | [03-encapsulation.md](./03-encapsulation.md) | Encapsulation & Access Modifiers | 3–4 |
| 4 | [04-static-readonly-const.md](./04-static-readonly-const.md) | `static`, `readonly`, `const` | 3 |
| 5 | [05-inheritance.md](./05-inheritance.md) | Inheritance — `base`, `this` | 4 |
| 6 | [06-polymorphism.md](./06-polymorphism.md) | `virtual`, `override`, `abstract`, `sealed` | 4–5 |
| 7 | [07-interface.md](./07-interface.md) | Interface — thiết kế contract | 3–4 |
| 8 | [08-class-vs-interface-vs-abstract.md](./08-class-vs-interface-vs-abstract.md) | Class vs Interface vs Abstract; composition | 4 |

**Project cuối cấp:** [project-banking-system.md](./project-banking-system.md) (~6–8 giờ)

---

## Cách học đề xuất

1. Học tuần tự chương 1 → 8; mỗi chương làm **Bài tập** trước khi xem đáp án.
2. Mỗi ngày ~2–3 giờ: đọc khái niệm + ví dụ + 2–3 bài tập.
3. Sau chương 8, dành 1–2 ngày cho **Banking System**.
4. Ghi chú chỗ bạn muốn “kế thừa mọi thứ” — trong Unity thường nên **composition** (gắn component) thay vì cây inheritance sâu.

---

## Nguyên tắc vàng Level 2

```text
"is-a" thật sự  → inheritance có thể hợp lý
"has-a" / hành vi ghép  → composition (ưu tiên)
"có thể làm X"  → interface
"cùng khung, khác chi tiết" → abstract class hoặc template method
```

**Cảnh báo Unity:** kế thừa sâu từ `MonoBehaviour` hoặc “mọi enemy đều là subclass của một God-class” dễ vỡ khi thêm feature. Prefer gắn nhiều component nhỏ.

---

## Checklist hoàn thành Level 2

- [ ] Viết được class có field, property, method, constructor
- [ ] Hiểu encapsulation và chọn đúng access modifier
- [ ] Phân biệt `static` / `readonly` / `const`
- [ ] Dùng inheritance có chủ đích; giải thích được `base` và `this`
- [ ] Viết `virtual`/`override`/`abstract`/`sealed` đúng ngữ cảnh
- [ ] Thiết kế interface và implement nhiều contract
- [ ] Biết khi nào *không* nên inheritance; áp dụng composition
- [ ] Hoàn thành Banking System (Account hierarchy + TransactionHistory + interest)
- [ ] Điểm kiểm tra kiến thức mỗi chương ≥ 4/5 đúng

Khi xong checklist → chuyển sang **Level 3 — Value Type vs Reference Type**.
