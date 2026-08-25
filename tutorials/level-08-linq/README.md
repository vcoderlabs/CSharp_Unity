# Level 8 — LINQ (~25 giờ)

Level này dạy bạn **truy vấn và biến đổi dữ liệu** bằng LINQ: toán tử lõi, toán tử phần tử/tập hợp, phân trang, materialization, và **deferred vs immediate execution**.

**Điều kiện:** Đã hoàn thành [Level 4 — Collections](../level-04-collections/) và [Level 7 — Delegates & Events](../level-07-delegates-events/) (cần quen `Func`/`Predicate` và `IEnumerable<T>`). [Level 5 — Generics](../level-05-generics/) giúp đọc chữ ký generic của LINQ dễ hơn.

**Tiếp theo:** [Level 9 — Advanced C#](../level-09-advanced-csharp/).

---

## Mục tiêu cấp độ

Sau Level 8 bạn sẽ:

- Viết pipeline LINQ: lọc (`Where`), chiếu (`Select`/`SelectMany`), sắp xếp, nhóm, join, aggregate
- Dùng đúng element operators: `Any`/`All`/`First`/`Single`/`Count`/`Sum`/`Average`/`Min`/`Max`
- Phân trang với `Skip`/`Take`/`Chunk`, loại trùng với `Distinct`
- Materialize kết quả (`ToList`/`ToArray`/`ToDictionary`) đúng lúc
- Giải thích deferred execution, phân biệt `IEnumerable` vs `IQueryable`, tránh multiple enumeration
- Xây **Data Analytics Console Application**

---

## Cảnh báo xuyên suốt Level 8

> **LINQ không “chạy” khi bạn viết chuỗi toán tử — phần lớn chỉ mô tả truy vấn.**  
> Kết quả thật sự được tính khi **enumerate** (foreach, `ToList`, `Count`, `First`, …). Enumerate nhiều lần = chạy lại nhiều lần — dễ chậm và kết quả lệch nếu nguồn đổi.

---

## 5 chương + 1 project

| # | File | Nội dung | ~Giờ |
|---|------|----------|------|
| 1 | [01-core-operators.md](./01-core-operators.md) | Where, Select, SelectMany, OrderBy/ThenBy, GroupBy, Join/GroupJoin, Aggregate | 5–6 |
| 2 | [02-element-operators.md](./02-element-operators.md) | Any, All, Contains, First/Single/Last, Count, Sum, Average, Min, Max | 3–4 |
| 3 | [03-set-paging.md](./03-set-paging.md) | Distinct, Skip, Take, Chunk | 2–3 |
| 4 | [04-materialization.md](./04-materialization.md) | ToList, ToArray, ToDictionary | 2–3 |
| 5 | [05-deferred-execution.md](./05-deferred-execution.md) | Deferred vs Immediate, IEnumerable vs IQueryable, multiple enumeration | 3–4 |
| — | [project-data-analytics.md](./project-data-analytics.md) | Data Analytics Console Application | 5–6 |

**Tổng ước lượng: ~25 giờ**

---

## Cách học đề xuất

1. Chương 1: viết cùng một truy vấn bằng **query syntax** và **method syntax** — hiểu chúng tương đương.
2. Chương 2: mỗi lần dùng `First` vs `FirstOrDefault` vs `Single` — ghi rõ khi nào throw.
3. Chương 3–4: làm paging + materialize; đo (Console.WriteLine) xem code chạy mấy lần.
4. Chương 5: cố tình enumerate 2 lần một query có side-effect — thấy “bẫy”.
5. Project: CSV/list in-memory → filter → group → report; không cần database.

---

## Checklist hoàn thành Level 8

- [ ] Viết pipeline Where + Select + OrderBy thành thạo
- [ ] Dùng GroupBy / Join đúng ngữ cảnh
- [ ] Chọn đúng First / Single / OrDefault
- [ ] Paging với Skip/Take; Chunk khi cần batch
- [ ] Biết khi nào gọi ToList / ToDictionary
- [ ] Giải thích deferred execution và tránh multiple enumeration
- [ ] Hoàn thành Data Analytics Console App
- [ ] Trả lời đúng ≥ 4/5 câu **Kiểm tra kiến thức** mỗi chương

Khi xong checklist → chuyển sang **Level 9 — Advanced C#**.
