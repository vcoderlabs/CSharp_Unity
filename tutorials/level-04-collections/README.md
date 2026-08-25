# Level 4 — Collections & Data Structures (~25 giờ)

Level này dạy bạn **chọn và dùng đúng cấu trúc dữ liệu** trong .NET: mảng, list, dictionary, set, queue/stack, sorted collections, interface collection, cú pháp C# 12, và so sánh Big O.

**Điều kiện:** Đã hoàn thành [Level 1](../level-01-csharp-fundamentals/), [Level 2 — OOP](../level-02-oop/), [Level 3 — Value vs Reference](../level-03-value-vs-reference/).

**Tiếp theo:** [Level 5 — Generics](../level-05-generics/)

---

## Mục tiêu cấp độ

Sau Level 4 bạn sẽ:

- Thành thạo `Array`, `List<T>`, `Dictionary<TKey,TValue>`, `HashSet<T>`
- Dùng `Queue<T>`, `Stack<T>`, `LinkedList<T>` đúng ngữ cảnh
- Biết khi nào cần `SortedDictionary` / `SortedSet`
- Đọc và dùng interface: `IEnumerable`, `ICollection`, `IList`, `IReadOnlyCollection`
- Viết collection expressions (C# 12)
- So sánh Big O / bộ nhớ để chọn collection phù hợp
- Hoàn thành project **Data Management System** (import CSV → chọn cấu trúc dữ liệu)

---

## Cảnh báo xuyên suốt Level 4

> **Collection sai = chậm + bug khó thấy.**  
> Tra cứu theo ID bằng `List` → O(n); bằng `Dictionary` → O(1) trung bình.  
> Trong Unity: `List`/`Dictionary` tạo/hủy mỗi frame → GC spike. Học chọn đúng cấu trúc **và** tái sử dụng buffer.

---

## 7 chương + 1 project

| # | File | Nội dung | ~Giờ |
|---|------|----------|------|
| 1 | [01-array-list.md](./01-array-list.md) | `Array`, `List<T>` | 3–4 |
| 2 | [02-dictionary-hashset.md](./02-dictionary-hashset.md) | `Dictionary`, `HashSet` | 3–4 |
| 3 | [03-queue-stack-linkedlist.md](./03-queue-stack-linkedlist.md) | `Queue`, `Stack`, `LinkedList` | 3 |
| 4 | [04-sorted-collections.md](./04-sorted-collections.md) | `SortedDictionary`, `SortedSet` | 2–3 |
| 5 | [05-collection-interfaces.md](./05-collection-interfaces.md) | `IEnumerable`, `ICollection`, `IList`, `IReadOnlyCollection` | 3 |
| 6 | [06-collection-expressions.md](./06-collection-expressions.md) | Collection expressions (C# 12) | 2 |
| 7 | [07-performance-big-o.md](./07-performance-big-o.md) | So sánh Big O / memory | 2–3 |
| — | [project-data-management.md](./project-data-management.md) | CSV → chọn data structure | 4–5 |

**Tổng ước lượng: ~25 giờ**

---

## Cách học đề xuất

1. Học tuần tự chương 1 → 7; mỗi chương làm **Bài tập** trước Gợi ý / Đáp án.
2. Với mỗi collection, tự trả lời: *thêm / tìm / xóa / duyệt* mất bao nhiêu (Big O).
3. Chương 7: in bảng so sánh ra giấy — đây là “cheat sheet” dùng cả đời.
4. Dành 1–2 ngày cho **Data Management System**.

---

## Checklist hoàn thành Level 4

- [ ] Thành thạo `Array` vs `List<T>` (khi nào cố định kích thước)
- [ ] Dùng `Dictionary` / `HashSet` cho tra cứu O(1)
- [ ] Biết FIFO (`Queue`) vs LIFO (`Stack`) vs `LinkedList`
- [ ] Phân biệt sorted vs unsorted collections
- [ ] Viết method nhận `IEnumerable<T>` / `IReadOnlyList<T>` thay vì concrete type khi phù hợp
- [ ] Dùng được collection expressions `[...]`
- [ ] Nhớ bảng Big O các thao tác phổ biến
- [ ] Hoàn thành Data Management System
- [ ] Trả lời đúng ≥ 4/5 câu **Kiểm tra kiến thức** mỗi chương

Khi xong checklist → chuyển sang **Level 5 — Generics**.
