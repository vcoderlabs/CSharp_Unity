# Capstone — Unity MMORPG / Game Architecture

Tổng hợp toàn lộ trình C# → Unity thành **kiến trúc game kiểu MMORPG** (slice có thể chơi được, không cần MMO thật hàng nghìn user). Tập trung **architecture đúng**, hệ thống tách module, và kỷ luật production.

**Điều kiện:** Hoàn thành [Level 21](../level-21-unity/) (và nền L1–L20 khuyến nghị). Project L21 mini game là điểm khởi đầu tốt.

**Ước lượng:** Tùy track (Fast / Normal / Deep) — thường **vài tuần đến vài tháng** part-time. Không tính vào ~412h lý thuyết level; đây là tích hợp.

---

## Mục tiêu capstone

Bạn sẽ xây (ít nhất ở mức vững):

- DI + service layer + game loop rõ
- Entity/component-oriented gameplay objects
- Inventory, Quest, Combat có pattern
- Networking client (REST và/hoặc WebSocket giả lập)
- Save/Load có versioning
- Pass tối ưu: profile, pool audit, GC

---

## Cảnh báo

> Đây **không** phải ship MMORPG thương mại.  
> Phạm vi là **architecture vertical slice**: đăng nhập giả → thế giới nhỏ → đánh quái → nhận quest → lưu progress → nói chuyện với “server” mock.  
> Ưu tiên biên giới module sạch hơn skin 3D đẹp.

---

## 8 milestones

| # | File | Trọng tâm |
|---|------|-----------|
| 01 | [milestone-01-core-architecture.md](./milestone-01-core-architecture.md) | DI, service layer, game loop |
| 02 | [milestone-02-entity-system.md](./milestone-02-entity-system.md) | Entity / components / generics |
| 03 | [milestone-03-inventory.md](./milestone-03-inventory.md) | Inventory, SOLID, events |
| 04 | [milestone-04-quest-system.md](./milestone-04-quest-system.md) | State + Observer + serialize định nghĩa |
| 05 | [milestone-05-combat-system.md](./milestone-05-combat-system.md) | Strategy, Command, pooling VFX/đạn |
| 06 | [milestone-06-networking.md](./milestone-06-networking.md) | REST/WebSocket client, async, JSON |
| 07 | [milestone-07-persistence.md](./milestone-07-persistence.md) | Save/load, versioning |
| 08 | [milestone-08-optimization.md](./milestone-08-optimization.md) | Profiling, GC, pool audit, benchmark |

Mỗi milestone gồm: **Requirements → Architecture → Tasks → Expected result → Exercises → Hints → Solution outline → Code review checklist**.

---

## Cách làm đề xuất

1. Một Unity project `CapstoneMmoArch` (hoặc nâng từ L21).  
2. Git từ ngày 1; PR theo milestone.  
3. Mỗi milestone: đọc Requirements → vẽ Architecture vào README milestone → code → checklist review tự làm.  
4. Mock server có thể là ASP.NET Minimal API local hoặc file JSON giả — không bắt buộc infra lớn.  
5. Milestone 08 chỉ làm khi 01–07 chơi được end-to-end.

---

## Definition of Done (toàn capstone)

- [ ] Vào play mode: di chuyển, nhặt item, làm 1 quest, đánh 1 enemy, thấy damage/VFX pooled
- [ ] “Login” qua HTTP mock + nhận profile JSON
- [ ] Save/Load lại inventory + quest state sau khi tắt Play
- [ ] Tài liệu `ARCHITECTURE.md` + sơ đồ ASCII cập nhật
- [ ] Milestone 08 có số liệu Profiler trước/sau
- [ ] Code review checklist mỗi milestone tự tick ≥ 80%

**Chúc bạn hoàn thành lộ trình From Zero → Unity.**
