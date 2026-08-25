# C# — Từ Zero đến Unity / Game Development

Lộ trình học C# bằng **tiếng Việt**, hướng tới Unity và game architecture (~6–12 tháng, ~412 giờ).

## Nội dung repo

| File / Thư mục | Mô tả |
|----------------|--------|
| [`plan.md`](./plan.md) | Roadmap đầy đủ: 22 level (0–21) + Capstone MMORPG |
| [`tutorials/`](./tutorials/) | Bài giảng + bài tập theo từng level |

## Tiến độ tutorials

| Level | Chủ đề | Trạng thái |
|-------|--------|------------|
| 0–5 | Fundamentals → OOP → Value/Ref → Collections → Generics | ✅ Có bài đầy đủ |
| 6–10 | Exception → Delegates → LINQ → Advanced → Memory | ✅ Có bài đầy đủ |
| 11–21 | Async → … → Unity | 📝 Khung README |
| Capstone | Unity MMORPG Architecture (8 milestones) | 📝 Khung |

**Chuỗi bắt buộc:** Level 0 → 1 → 2 → 3 → 4 → 5. Không nhảy cóc.

Bỏ Level 0 nếu đã biết lập trình (Java, Python, C++, JavaScript…) — bắt đầu từ Level 1.

## Cách bắt đầu

1. Đọc [`plan.md`](./plan.md) để nắm toàn bộ lộ trình.
2. Vào [`tutorials/README.md`](./tutorials/README.md) chọn level.
3. Mỗi chương: đọc khái niệm → làm bài tập **trước** khi xem đáp án → kiểm tra kiến thức.
4. Hoàn thành project cuối mỗi level trước khi sang level tiếp theo.

## Cấu trúc mỗi bài học

Mỗi chương trong `tutorials/` theo cùng một khung: mục tiêu, điều kiện tiên quyết, khái niệm, mô hình tư duy, cú pháp, ví dụ (cơ bản → nâng cao), lỗi thường gặp, gỡ lỗi, best practices, bài tập, gợi ý, đáp án, thử thách, liên hệ Unity, kiểm tra kiến thức.

## Yêu cầu môi trường (từ Level 1)

- [.NET SDK](https://dotnet.microsoft.com/download) 8 trở lên
- Editor: Cursor / VS Code / Visual Studio / Rider

```bash
dotnet --version
dotnet new console -n HelloApp
cd HelloApp && dotnet run
```

## Track học gợi ý

| Track | Thời lượng | Phù hợp |
|-------|------------|---------|
| Fast | ~3 tháng | Đã có nền lập trình khác |
| Normal | ~6 tháng | Người mới, muốn chắc nền |
| Deep | 9–12 tháng | Người mới hoàn toàn, muốn hiểu sâu |

Chi tiết giờ/ngày và review nằm trong [`plan.md`](./plan.md).
