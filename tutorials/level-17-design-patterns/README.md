# Level 17 — Design Patterns (~30 giờ)

Level **then chốt** cho Unity MMORPG: học pattern **Creational / Structural / Behavioral**, biết **khi nào dùng** và đặc biệt **khi nào không** (over-engineering). Pattern là *ngôn ngữ chung* — không phải mục tiêu tự thân.

**Điều kiện:** [Level 16 — SOLID](../level-16-solid/). Nên vững OOP (L2), delegates/events (L7), generics (L5).

**Tiếp theo:** [Level 18 — Architecture](../level-18-architecture/) (ranh giới hệ thống; pattern nằm *trong* layer).

**Project chung:** [Clean Architecture Application](../level-18-architecture/project-clean-architecture-app.md) (L15–18). Dùng pattern có chọn lọc — xem chương 18.

---

## Mục tiêu cấp độ

Sau Level 17 bạn sẽ:

- Nhận diện và implement ~17 pattern phổ biến bằng C#
- Map sang Unity: **Observer**, **State**, **Command**, tham chiếu **Object Pool** (liên hệ Prototype/Factory)
- Tránh God Singleton, pattern nesting vô nghĩa
- Giải thích trade-off từng pattern trong checklist cuối chương

---

## Cảnh báo xuyên suốt Level 17

> **Pattern là công cụ giải quyết lực cản thiết kế — không phải huy hiệu.**  
> Nếu bài toán chưa có biến thể / chưa đau → đừng “Factory Abstract Builder Decorator” xếp tầng. Đọc [18-when-not-to-use.md](./18-when-not-to-use.md) sớm, không để cuối mới đọc.

---

## Nhóm chương

### Creational (~6–7h)

| # | File | Pattern |
|---|------|---------|
| 1 | [01-singleton.md](./01-singleton.md) | Singleton |
| 2 | [02-factory.md](./02-factory.md) | Factory Method / Simple Factory |
| 3 | [03-abstract-factory.md](./03-abstract-factory.md) | Abstract Factory |
| 4 | [04-builder.md](./04-builder.md) | Builder |
| 5 | [05-prototype.md](./05-prototype.md) | Prototype (+ gợi ý Pool) |

### Structural (~6–7h)

| # | File | Pattern |
|---|------|---------|
| 6 | [06-adapter.md](./06-adapter.md) | Adapter |
| 7 | [07-decorator.md](./07-decorator.md) | Decorator |
| 8 | [08-facade.md](./08-facade.md) | Facade |
| 9 | [09-proxy.md](./09-proxy.md) | Proxy |
| 10 | [10-composite.md](./10-composite.md) | Composite |

### Behavioral (~12–14h) — trọng tâm game

| # | File | Pattern | Unity |
|---|------|---------|-------|
| 11 | [11-observer.md](./11-observer.md) | Observer | Events / UI / quest |
| 12 | [12-strategy.md](./12-strategy.md) | Strategy | Combat AI / damage |
| 13 | [13-command.md](./13-command.md) | Command | Input, undo, skills |
| 14 | [14-state.md](./14-state.md) | State | Anim, AI, quest step |
| 15 | [15-template-method.md](./15-template-method.md) | Template Method | Match flow |
| 16 | [16-mediator.md](./16-mediator.md) | Mediator | UI hub / systems |
| 17 | [17-chain-of-responsibility.md](./17-chain-of-responsibility.md) | Chain of Responsibility | Damage pipeline |

### Kỷ luật thiết kế (~2h)

| # | File | Nội dung |
|---|------|----------|
| 18 | [18-when-not-to-use.md](./18-when-not-to-use.md) | Over-engineering, YAGNI vs pattern |

**Tổng ước lượng: ~30 giờ**

---

## Cách học đề xuất

1. Creational: code console trước — đừng nhảy Unity Singleton ngay.
2. Behavioral (11–14): làm mini demo combat/quest — đây là xương Capstone.
3. Mỗi pattern: 1 ví dụ “đúng chỗ” + 1 ví dụ “lạm dụng”.
4. Chương 18: review lại code L16–17, **xóa** pattern thừa.

---

## Checklist hoàn thành Level 17

- [ ] Implement được ≥ 1 pattern mỗi nhóm Creational / Structural / Behavioral
- [ ] Observer + State + Command có demo (console hoặc Unity)
- [ ] Giải thích Object Pool liên hệ Prototype/Factory
- [ ] Chỉ ra ≥ 3 chỗ **không** nên dùng pattern trong bài tập chương 18
- [ ] Áp 2–3 pattern có chủ đích vào project chung L15–18
- [ ] ≥ 4/5 câu kiểm tra mỗi chương đã học

Khi xong → **Level 18 — Architecture**.
