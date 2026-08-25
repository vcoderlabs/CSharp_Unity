# Level 3 — Value Type vs Reference Type (~15 giờ)

Level này giải thích **cách dữ liệu sống trong bộ nhớ** và **khi nào bạn đang sửa bản gốc hay chỉ sửa bản sao**. Hiểu sai ở đây = bug “không đổi gì” trong Unity (ví dụ `transform.position.x = ...`), side-effect khó đoán, và GC spike.

**Điều kiện:** Đã hoàn thành [Level 1](../level-01-csharp-fundamentals/) và [Level 2 — OOP](../level-02-oop/) (class, object, property, method).

**Tiếp theo:** [Level 4 — Collections](../level-04-collections/)

---

## Mục tiêu cấp độ

Sau Level 3 bạn sẽ:

- Phân biệt **Stack** và **Heap**, biết value type / reference type nằm đâu
- Chọn đúng **`struct` vs `class`** theo kích thước, tuổi thọ, và semantics
- Hiểu **boxing/unboxing** và khi nào nó tốn bộ nhớ / CPU
- Dự đoán đúng kết quả khi **gán / truyền / copy** (value vs reference semantics)
- Dùng **`ref` / `out` / `in`** có chủ đích
- Hoàn thành project **Inventory System** (`struct ItemSlot` + `class Inventory`)

---

## Cảnh báo xuyên suốt Level 3

> **Mutating a struct copy không đổi bản gốc.**  
> Trong Unity: `transform.position.x = 5;` **không** di chuyển object — bạn sửa `Vector3` tạm rồi vứt đi. Phải: `var p = transform.position; p.x = 5; transform.position = p;`  
> Mỗi chương sẽ nhắc lại — đây là bug kinh điển nhất khi nhầm value/reference.

---

## 5 chương + 1 project

| # | File | Nội dung | ~Giờ |
|---|------|----------|------|
| 1 | [01-stack-vs-heap.md](./01-stack-vs-heap.md) | Stack vs Heap, ASCII memory | 2–3 |
| 2 | [02-struct-vs-class.md](./02-struct-vs-class.md) | `struct` vs `class`, khi nào dùng gì | 2–3 |
| 3 | [03-boxing-unboxing.md](./03-boxing-unboxing.md) | Boxing / Unboxing, chi phí ẩn | 2 |
| 4 | [04-copying-semantics.md](./04-copying-semantics.md) | Copy value vs copy reference | 2–3 |
| 5 | [05-ref-out-in.md](./05-ref-out-in.md) | Truyền tham số: `ref`, `out`, `in` | 2 |
| — | [project-inventory-system.md](./project-inventory-system.md) | Inventory System + side-effect demos | 3–4 |

**Tổng ước lượng: ~15 giờ**

---

## Cách học đề xuất

1. Học tuần tự chương 1 → 5; **vẽ lại** sơ đồ Stack/Heap bằng tay trước khi làm bài tập.
2. Mỗi chương: làm **Bài tập** trước Gợi ý / Đáp án.
3. Cố tình tái hiện bug Unity `position.x = ...` trong console (dùng struct giả) để “cảm nhận” lỗi.
4. Dành 1–2 buổi cho **Inventory System** — phần demo side-effect là quan trọng nhất.

---

## Checklist hoàn thành Level 3

- [ ] Giải thích được Stack vs Heap và vẽ sơ đồ gán `int` / `class`
- [ ] Biết khi nào chọn `struct`, khi nào chọn `class`
- [ ] Nhận diện boxing trong code và giải thích chi phí
- [ ] Dự đoán đúng kết quả sau gán `struct` vs gán `class`
- [ ] Dùng được `ref` / `out` / `in` và biết khi nào **không** cần
- [ ] Hoàn thành Inventory System + demo side-effect khi truyền reference sai
- [ ] Trả lời đúng ≥ 4/5 câu **Kiểm tra kiến thức** mỗi chương
- [ ] Nhớ luật vàng: **sửa bản sao struct ≠ sửa bản gốc** (Unity `Transform.position`)

Khi xong checklist → chuyển sang **Level 4 — Collections**.
