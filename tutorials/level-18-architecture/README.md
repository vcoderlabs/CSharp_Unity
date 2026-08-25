# Level 18 — Architecture (~20 giờ)

Level này dạy **ranh giới hệ thống**: Layered, Clean Architecture, DI & Dependency Inversion ở tầm app, Repository/Service, DTO vs Entity, Modular architecture. Pattern (L17) và SOLID (L16) nằm *bên trong* các layer — không thay thế sơ đồ tổng thể.

**Điều kiện:** [Level 16 — SOLID](../level-16-solid/) + [Level 17 — Design Patterns](../level-17-design-patterns/) (đặc biệt Facade, Adapter, DIP). Nên có [Level 14 — Testing](../level-14-testing/) và [Level 15 — Clean Code](../level-15-clean-code/).

**Tiếp theo:** [Level 19 — Performance](../level-19-performance/) hoặc [Level 21 — Unity](../level-21-unity/) tùy track. Capstone MMORPG dựa nặng L17–18.

**Project chung L15–18:** [project-clean-architecture-app.md](./project-clean-architecture-app.md) — outline + task cụ thể; hoàn thiện kiến trúc ở level này.

---

## Mục tiêu cấp độ

Sau Level 18 bạn sẽ:

- Vẽ và implement layered / clean architecture cho app console (.NET)
- Đặt dependency hướng vào Domain; infra ở ngoài (DIP tầm hệ thống)
- Tách Repository / Application Service / DTO rõ ràng
- Chia module theo feature; tránh “big ball of mud”
- Hoàn thành **Clean Architecture Application** dùng chung từ L15

---

## Cảnh báo xuyên suốt Level 18

> **Architecture phục vụ thay đổi và test — không phải sơ đồ đẹp trên slide.**  
> App nhỏ có thể 2 layer. MMORPG / backend lớn cần ranh giới rõ. Đừng copy Clean Architecture 8 project khi CRUD 3 màn hình (xem L17 chương 18).

---

## 6 chương + 1 project

| # | File | Nội dung | ~Giờ |
|---|------|----------|------|
| 1 | [01-layered.md](./01-layered.md) | Layered architecture | 2–3 |
| 2 | [02-clean-architecture.md](./02-clean-architecture.md) | Clean Architecture | 3–4 |
| 3 | [03-di-dependency-inversion.md](./03-di-dependency-inversion.md) | DI & DIP tầm app | 2–3 |
| 4 | [04-repository-service.md](./04-repository-service.md) | Repository / Service | 2–3 |
| 5 | [05-dto-vs-entity.md](./05-dto-vs-entity.md) | DTO vs Entity | 2–3 |
| 6 | [06-modular-architecture.md](./06-modular-architecture.md) | Modular architecture | 2–3 |
| — | [project-clean-architecture-app.md](./project-clean-architecture-app.md) | Project chung L15–18 | 5–7 |

**Tổng ước lượng: ~20 giờ**

---

## Cách học đề xuất

1. Vẽ layer trên giấy trước khi tạo project.
2. Domain **không** reference EF/Unity/HTTP — enforce bằng project reference.
3. Làm project chung song song từ chương 2–3.
4. Cuối level: một integration test xuyên layer + unit test domain thuần.

---

## Checklist hoàn thành Level 18

- [ ] Giải thích khác Layered vs Clean (hướng phụ thuộc)
- [ ] Composition root wire DI đúng chỗ
- [ ] Repository + Application Service tách I/O khỏi use case
- [ ] DTO không lộ entity navigation/infra
- [ ] ≥ 2 module feature với API rõ
- [ ] Hoàn thành task bắt buộc trong project chung
- [ ] ≥ 4/5 câu kiểm tra mỗi chương

Khi xong checklist → Level 19 hoặc đi sâu Unity / Capstone tùy lộ trình.
