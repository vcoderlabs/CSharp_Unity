# Level 9 — Advanced C# (~30 giờ)

Level này dạy các tính năng C# hiện đại giúp code **ngắn hơn, an toàn hơn, linh hoạt hơn**: pattern matching, record/tuple, nullable reference types, attributes & reflection, dynamic & expression trees, extension methods, iterators, index/range, local functions, anonymous types.

**Điều kiện:** Hoàn thành [Level 2 — OOP](../level-02-oop/), [Level 5 — Generics](../level-05-generics/), và nên xong [Level 8 — LINQ](../level-08-linq/) (nhiều ví dụ dùng `IEnumerable` / projection).

**Tiếp theo:** [Level 10 — Memory](../level-10-memory/) (hoặc song song theo plan).

---

## Mục tiêu cấp độ

Sau Level 9 bạn sẽ:

- Viết `switch` expression / pattern matching thành thạo
- Dùng `record` / `record struct`, tuple và deconstruction
- Bật và xử lý **nullable reference types** đúng cách
- Đọc custom attribute bằng reflection; hiểu giới hạn performance
- Phân biệt `dynamic` và expression trees
- Viết extension methods, iterators (`yield`), index/range, local functions
- Xây **Reflection-based Serializer** tối giản

---

## Cảnh báo xuyên suốt Level 9

> **Reflection và `dynamic` mạnh nhưng mất an toàn biên dịch và thường chậm hơn.**  
> Prefer pattern matching, generics, source generator / compile-time khi có thể. Dùng reflection có chủ đích (plugin, serializer, tooling).

---

## 8 chương + 1 project

| # | File | Nội dung | ~Giờ |
|---|------|----------|------|
| 1 | [01-pattern-matching.md](./01-pattern-matching.md) | Pattern matching & switch expressions | 3–4 |
| 2 | [02-records-tuples.md](./02-records-tuples.md) | record / record struct, tuples & deconstruction | 3–4 |
| 3 | [03-nullable-reference.md](./03-nullable-reference.md) | Nullable reference types | 2–3 |
| 4 | [04-attributes-reflection.md](./04-attributes-reflection.md) | Attributes & reflection | 3–4 |
| 5 | [05-dynamic-expression-trees.md](./05-dynamic-expression-trees.md) | dynamic, expression trees | 3–4 |
| 6 | [06-extension-methods.md](./06-extension-methods.md) | Extension methods | 2 |
| 7 | [07-iterators-yield.md](./07-iterators-yield.md) | Iterators / yield | 2–3 |
| 8 | [08-index-range-local-anon.md](./08-index-range-local-anon.md) | Index/range, local functions, anonymous types | 2–3 |
| — | [project-reflection-serializer.md](./project-reflection-serializer.md) | Reflection-based Serializer | 5–6 |

**Tổng ước lượng: ~30 giờ**

---

## Cách học đề xuất

1. Chương 1–2: viết lại if/else cũ bằng pattern + record — cảm nhận độ gọn.
2. Chương 3: bật NRT, sửa hết warning trên một file nhỏ.
3. Chương 4–5 + project: reflection serializer — đọc attribute, lấy property.
4. Chương 6–8: tiện ích hàng ngày; tự viết lại vài helper LINQ bằng `yield`.

---

## Checklist hoàn thành Level 9

- [ ] Switch expression + property/list pattern cơ bản
- [ ] Record với `with`; tuple deconstruct
- [ ] Giải thích được `string` vs `string?` khi NRT bật
- [ ] Đọc custom attribute bằng reflection
- [ ] Biết khi nào *không* dùng `dynamic`
- [ ] Viết extension method và iterator `yield return`
- [ ] Dùng `^` / `..` và local function
- [ ] Hoàn thành Reflection-based Serializer
- [ ] Trả lời đúng ≥ 4/5 câu **Kiểm tra kiến thức** mỗi chương

Khi xong checklist → chuyển sang **Level 10** (hoặc level kế tiếp theo plan).
