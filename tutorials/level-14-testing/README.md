# Level 14 — Testing (~15 giờ)

Level này dạy bạn **kiểm chứng hành vi code bằng test tự động**: phân biệt unit / integration, viết test theo **Arrange–Act–Assert**, dùng **mock**, áp dụng **Dependency Injection** để test được, và làm quen **xUnit / NUnit**.

**Điều kiện:** Đã hoàn thành Level 1–8 (C# cơ bản → OOP → Collections → Generics → Exceptions → Delegates → LINQ). Biết tạo class, interface, và project .NET console.

**Tiếp theo:** [Level 15 — Clean Code](../level-15-clean-code/) (code dễ đọc + dễ test đi đôi với nhau).

---

## Mục tiêu cấp độ

Sau Level 14 bạn sẽ:

- Phân biệt unit test và integration test, biết khi nào dùng cái nào
- Viết test rõ ràng theo AAA (Arrange–Act–Assert)
- Mock dependency bằng interface + thư viện (Moq / NSubstitute) hoặc fake thủ công
- Thiết kế code **injectable** (constructor injection) để unit test không cần DB/HTTP thật
- Tạo project test với **xUnit** (và so sánh nhanh với NUnit)
- Hoàn thành **Tested Application**: calculator/service + bộ test cho khái niệm L1–L8

---

## Cảnh báo xuyên suốt Level 14

> **Test không phải “chạy thử một lần trong Main”.**  
> Test tự động chạy lặp lại, fail rõ ràng khi regression, và ép bạn thiết kế API rõ ràng. Code khó test thường là code quá gắn cứng dependency (new SqlConnection bên trong method).

---

## 5 chương + 1 project

| # | File | Nội dung | ~Giờ |
|---|------|----------|------|
| 1 | [01-unit-vs-integration.md](./01-unit-vs-integration.md) | Unit vs Integration, pyramid | 2–3 |
| 2 | [02-arrange-act-assert.md](./02-arrange-act-assert.md) | AAA, assert, naming test | 2–3 |
| 3 | [03-mocking.md](./03-mocking.md) | Fake, stub, mock | 2–3 |
| 4 | [04-dependency-injection-basics.md](./04-dependency-injection-basics.md) | Constructor injection, testability | 2–3 |
| 5 | [05-xunit-nunit.md](./05-xunit-nunit.md) | xUnit / NUnit, Fact/Theory | 2 |
| — | [project-tested-application.md](./project-tested-application.md) | Calculator + service + tests L1–L8 | 4–5 |

**Tổng ước lượng: ~15 giờ**

---

## Cách học đề xuất

1. Chương 1: tự viết 1 unit test và 1 “integration-style” test (đọc file) để cảm nhận khác biệt.
2. Chương 2: mọi test đều tách rõ 3 khối Arrange / Act / Assert — không gộp.
3. Chương 3: trước khi dùng Moq, viết **fake class** thủ công một lần.
4. Chương 4: refactor `new EmailSender()` trong class thành inject `IEmailSender`.
5. Chương 5: tạo solution 2 project (`App` + `App.Tests`) với xUnit.
6. Project: viết calculator/service rồi phủ test theo checklist L1–L8.

---

## Checklist hoàn thành Level 14

- [ ] Giải thích được unit vs integration bằng ví dụ của mình
- [ ] Viết test AAA có tên mô tả hành vi (Given/When/Then hoặc Method_Scenario_Expected)
- [ ] Tạo fake/mock cho interface và assert interaction khi cần
- [ ] Refactor một class sang constructor injection để test được
- [ ] Chạy được `dotnet test` với xUnit (hoặc NUnit)
- [ ] Hoàn thành project Tested Application
- [ ] Trả lời đúng ≥ 4/5 câu **Kiểm tra kiến thức** mỗi chương

Khi xong checklist → chuyển sang **Level 15 — Clean Code**.
