# Level 1 — C# Fundamentals (~40 giờ)

Nền tảng C# thực hành: viết và chạy chương trình console trên .NET, nắm kiểu dữ liệu, điều khiển luồng, method, string và các utility cơ bản.

**Điều kiện:** Đã hoàn thành [Level 0](../level-00-programming-fundamentals/) *hoặc* đã lập trình ngôn ngữ khác (Java, Python, C++, JavaScript…).

**Tiếp theo:** [Level 2 — OOP](../level-02-oop/)

---

## Mục tiêu cấp độ

Sau Level 1 bạn sẽ:

- Cài .NET SDK, tạo và chạy project console bằng `dotnet` CLI
- Hiểu cấu trúc `.csproj`, namespace, điểm vào `Main`
- Dùng biến, hằng, `var`, kiểu nguyên thủy, `enum`, nullable
- Viết biểu thức, `if`/`switch`, vòng lặp, method (overload, scope)
- Xử lý chuỗi, `StringBuilder`, `DateTime` / `Math` / `Random` / `Console`
- Hoàn thành project **Console Calculator**

---

## 10 chương

| # | File | Nội dung | ~Giờ |
|---|------|----------|------|
| 1 | [01-csharp-va-dotnet.md](./01-csharp-va-dotnet.md) | SDK, CLR, Runtime, Compiler, `dotnet` CLI | 3–4 |
| 2 | [02-project-solution.md](./02-project-solution.md) | `.csproj`, namespace, `Main` | 3 |
| 3 | [03-bien-hang-var.md](./03-bien-hang-var.md) | Biến, hằng, `var` | 3 |
| 4 | [04-kieu-du-lieu-nguyen-thuy.md](./04-kieu-du-lieu-nguyen-thuy.md) | Số, `bool`, `char`, `string`, `enum` | 4 |
| 5 | [05-toan-tu-bieu-thuc.md](./05-toan-tu-bieu-thuc.md) | Toán tử & biểu thức | 3–4 |
| 6 | [06-luong-dieu-khien.md](./06-luong-dieu-khien.md) | `if`, `switch`, vòng lặp, `break`/`continue` | 5 |
| 7 | [07-methods.md](./07-methods.md) | Tham số, return, overloading, scope | 4–5 |
| 8 | [08-nullable.md](./08-nullable.md) | `int?` và nullable value types | 2–3 |
| 9 | [09-string-stringbuilder.md](./09-string-stringbuilder.md) | String & `StringBuilder` | 3–4 |
| 10 | [10-utility-types.md](./10-utility-types.md) | `DateTime`, `TimeSpan`, `Math`, `Random`, `Console` | 3–4 |

**Project cuối cấp:** [project-console-calculator.md](./project-console-calculator.md) (~6–8 giờ)

---

## Cài .NET SDK

1. Tải SDK tại [https://dotnet.microsoft.com/download](https://dotnet.microsoft.com/download) (khuyến nghị **.NET 8** trở lên).
2. Cài đặt theo hệ điều hành (Windows / macOS / Linux).
3. Mở terminal và kiểm tra:

```bash
dotnet --version
```

Nếu hiện số phiên bản (ví dụ `8.0.x` hoặc `9.0.x`) là đã sẵn sàng.

### Tạo và chạy project console lần đầu

```bash
# Tạo thư mục làm việc (tùy bạn)
mkdir HelloCSharp && cd HelloCSharp

# Tạo project console
dotnet new console -n HelloApp
cd HelloApp

# Chạy
dotnet run
```

Lệnh `dotnet new console` tạo file `Program.cs` và `HelloApp.csproj`. `dotnet run` biên dịch rồi chạy chương trình.

Sửa `Program.cs`, lưu file, rồi `dotnet run` lại để thấy thay đổi.

---

## Cách học đề xuất

1. Học tuần tự chương 1 → 10; mỗi chương làm **Bài tập** trước khi xem đáp án.
2. Mỗi ngày ~2–3 giờ: đọc khái niệm + ví dụ + 2–3 bài tập.
3. Sau chương 10, dành 1–2 ngày cho **Console Calculator**.
4. Ghi chú lỗi bạn gặp — chúng thường lặp lại ở Unity sau này.

---

## Checklist hoàn thành Level 1

- [ ] Cài SDK và chạy được `dotnet new console` / `dotnet run`
- [ ] Hiểu `.csproj`, namespace, `Main`
- [ ] Thành thạo biến, kiểu, toán tử, điều khiển luồng
- [ ] Viết method có tham số / return / overload
- [ ] Dùng `int?`, string, `StringBuilder`, utility types
- [ ] Hoàn thành Console Calculator (biểu thức nhiều toán tử + lịch sử + xử lý input sai)
- [ ] Điểm kiểm tra kiến thức mỗi chương ≥ 4/5 đúng

Khi xong checklist → chuyển sang **Level 2 — OOP**.
