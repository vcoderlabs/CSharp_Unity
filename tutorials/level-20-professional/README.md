# Level 20 — Professional C# (~10 giờ)

Level này dạy thói quen làm việc **như developer chuyên nghiệp**: Git/GitHub, NuGet, cấu trúc solution, coding standards, logging, config theo môi trường, CI/CD, code review, và debug production.

**Điều kiện:** Biết C# đến mức viết app nhỏ (khuyến nghị xong L1–L14 cơ bản). Có thể học **song song** nhiều level khác.

**Liên kết:** Project chung với [Level 19 — Performance](../level-19-performance/) — [Production-grade application](./project-production-app.md).

**Tiếp theo:** [Level 21 — C# for Unity](../level-21-unity/).

---

## Mục tiêu cấp độ

Sau Level 20 bạn sẽ:

- Dùng Git cơ bản thành thạo và mở PR trên GitHub
- Tổ chức solution + NuGet packages sạch
- Áp dụng convention + logging có cấu trúc
- Tách config Dev/Staging/Prod
- Viết workflow CI tối thiểu + review checklist
- Tiếp cận debug khi chỉ có log/metric production
- Hoàn thành (hoặc đóng góp lớn) app production L19–20

---

## Cảnh báo xuyên suốt Level 20

> **Professional ≠ thêm tool cho vui.** Mỗi tool phải phục vụ: tái lập môi trường, giảm rủi ro merge, quan sát hệ thống khi lỗi.  
> Secret không bao giờ commit. Config ≠ code cứng.

---

## 6 chương + 1 project

| # | File | Nội dung | ~Giờ |
|---|------|----------|------|
| 1 | [01-git-github.md](./01-git-github.md) | Git / GitHub | 1.5 |
| 2 | [02-nuget-project-structure.md](./02-nuget-project-structure.md) | NuGet & cấu trúc project | 1.5 |
| 3 | [03-coding-standards-logging.md](./03-coding-standards-logging.md) | Standards & logging | 1.5 |
| 4 | [04-config-environment.md](./04-config-environment.md) | Config / environment | 1.5 |
| 5 | [05-cicd-code-review.md](./05-cicd-code-review.md) | CI/CD & code review | 1.5 |
| 6 | [06-debugging-production.md](./06-debugging-production.md) | Debug production | 1.5 |
| — | [project-production-app.md](./project-production-app.md) | App production (chung L19–20) | ~4–8 (chia sẻ) |

**Tổng ước lượng lý thuyết L20: ~10 giờ** (+ project tùy độ sâu).

---

## Cách học đề xuất

1. Tạo repo Git thật cho project production ngay chương 1.
2. Mỗi chương áp dụng một thay đổi nhỏ lên cùng solution.
3. Bật CI sớm (dù chỉ `dotnet build` + `test`).
4. Phần performance (benchmark/cache) làm theo L19 trên cùng app.

---

## Checklist hoàn thành Level 20

- [ ] Repo có lịch sử commit sạch, README, `.gitignore`
- [ ] Solution nhiều project (src / tests) + NuGet hợp lý
- [ ] EditorConfig / analyzer + logging có level
- [ ] `appsettings` + env vars, không commit secret
- [ ] PR template + CI build/test xanh
- [ ] Biết quy trình đọc log khi “prod lỗi”
- [ ] [Project production](./project-production-app.md) đạt MVP checklist
- [ ] ≥ 4/5 câu kiểm tra mỗi chương

Khi xong → **Level 21** hoặc Capstone nếu đã xong Unity.
