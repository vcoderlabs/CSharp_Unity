# Level 16 — SOLID (~20 giờ)

Level này đào sâu **năm nguyên tắc SOLID** — mỗi nguyên tắc theo chuỗi: **bad code → problem → refactor → good code → Unity example**. Mục tiêu không phải thuộc lòng tên, mà **nhận ra khi code đang vi phạm** và biết cách tách/ghép lại an toàn.

**Điều kiện:** Đã hoàn thành [Level 15 — Clean Code](../level-15-clean-code/) (naming, smell, DRY/KISS/YAGNI, coupling/cohesion). Nên có nền OOP (L2) + interface/generics (L2, L5).

**Tiếp theo:** [Level 17 — Design Patterns](../level-17-design-patterns/) (áp dụng pattern *sau khi* SOLID đã định hướng thiết kế).

**Project chung:** [Clean Architecture Application](../level-18-architecture/project-clean-architecture-app.md) — **dùng chung Level 15–18**. Ở L16 bạn áp SOLID lên domain/service của project đó.

---

## Mục tiêu cấp độ

Sau Level 16 bạn sẽ:

- Giải thích và áp dụng **SRP, OCP, LSP, ISP, DIP** bằng ví dụ C#
- Nhận diện vi phạm SOLID trong code “God class”, `switch` kiểu, inheritance sai, fat interface, `new` cứng
- Refactor từng bước nhỏ (ideal: có test từ L14)
- Liên hệ từng nguyên tắc với Unity (MonoBehaviour, ScriptableObject, DI container)

---

## Cảnh báo xuyên suốt Level 16

> **SOLID là la bàn, không phải checklist bắt buộc mọi file.**  
> Áp dụng quá sớm → over-engineering. Vi phạm cố ý tạm thời → nợ kỹ thuật. Cân bằng với YAGNI (L15): tách khi có **lý do đổi** thật sự, không tách vì “trông giống sách”.

---

## 5 chương (1 nguyên tắc / chương)

| # | File | Nguyên tắc | ~Giờ |
|---|------|------------|------|
| 1 | [01-srp.md](./01-srp.md) | Single Responsibility | 3–4 |
| 2 | [02-ocp.md](./02-ocp.md) | Open/Closed | 3–4 |
| 3 | [03-lsp.md](./03-lsp.md) | Liskov Substitution | 3–4 |
| 4 | [04-isp.md](./04-isp.md) | Interface Segregation | 3–4 |
| 5 | [05-dip.md](./05-dip.md) | Dependency Inversion | 3–4 |

**Project chung L15–18:** chi tiết task ở [Level 18 — project](../level-18-architecture/project-clean-architecture-app.md).  
**Tổng ước lượng Level 16: ~20 giờ** (gồm bài tập + áp vào project chung).

---

## Cách học đề xuất

1. Mỗi chương: đọc **bad → problem** trước, **tự refactor** rồi mới xem good code.
2. Viết 1–2 test cho hành vi cũ trước khi tách class (nếu đã học L14).
3. Với Unity: đừng copy nguyên mẫu web vào `MonoBehaviour` — tách **logic thuần C#** khỏi `Update`/`SerializeField`.
4. Cuối level: trên project chung, liệt kê 5 chỗ vi phạm SOLID và sửa ít nhất 3.

---

## Liên hệ Capstone / Unity MMORPG

SOLID là nền để inventory, quest, combat **scale** mà không biến thành một `GameManager` 3000 dòng. Level 17 thêm pattern; Level 18 thêm ranh giới layer.

---

## Checklist hoàn thành Level 16

- [ ] SRP: tách được God class thành ≥ 2 trách nhiệm rõ
- [ ] OCP: thêm behavior mới bằng class mới, không sửa `switch` khổng lồ
- [ ] LSP: chỉ ra 1 inheritance sai và sửa bằng composition/interface
- [ ] ISP: tách fat interface thành interface nhỏ theo client
- [ ] DIP: phụ thuộc abstraction; inject qua ctor (console hoặc Unity DI)
- [ ] Áp ≥ 3 nguyên tắc vào project chung L15–18
- [ ] Trả lời đúng ≥ 4/5 câu **Kiểm tra kiến thức** mỗi chương

Khi xong checklist → chuyển sang **Level 17 — Design Patterns**.
