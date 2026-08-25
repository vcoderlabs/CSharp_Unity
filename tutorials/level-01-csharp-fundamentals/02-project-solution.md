# Chương 2 — Project, Solution, `.csproj`, namespace và `Main`

## 1. Mục tiêu học

- Hiểu project vs solution trong hệ sinh thái .NET
- Đọc file `.csproj` cơ bản và vai trò của nó
- Giải thích `namespace`, điểm vào chương trình (`Main` / top-level statements)
- Tạo, build, chạy project một cách chủ động

## 2. Điều kiện tiên quyết

- Đã cài .NET SDK và chạy được `dotnet --version`
- Đã hoàn thành Chương 1 (biết `dotnet new console`, `dotnet run`)

## 3. Khái niệm

**Project** là một đơn vị biên dịch: thường một thư mục có file `*.csproj` + mã nguồn `.cs` → ra một assembly (`.dll`).

**Solution** (`.sln`) là “thùng chứa” nhiều project liên quan (app + thư viện + test). Khi mới học, một project là đủ; solution hữu ích khi app lớn hơn.

**`.csproj`** là file XML (SDK-style) mô tả: target framework (`net8.0`), loại output (`Exe` / `Library`), package NuGet, v.v. MSBuild đọc file này khi `dotnet build`.

**Namespace** nhóm kiểu (`class`, `enum`…) để tránh trùng tên và tổ chức code theo miền (`MyApp.Math`, `MyApp.UI`).

**Điểm vào (`Main`)** là nơi CLR bắt đầu chạy ứng dụng console. Từ .NET 6, template dùng **top-level statements**: compiler tự sinh `Main` ẩn.

## 4. Mô hình tư duy

```text
Solution (tùy chọn)
 └── Project A (App.csproj)  → App.dll  (Exe)
 └── Project B (Lib.csproj)  → Lib.dll  (Library)

Trong một project console:
  Program.cs  ──►  điểm vào (Main / top-level)
  Other.cs    ──►  class phụ trợ
```

Namespace giống “đường dẫn logic”; thư mục vật lý *nên* khớp namespace nhưng không bắt buộc 100%.

## 5. Cú pháp (C# thật)

`.csproj` tối giản:

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>
</Project>
```

Namespace + class + `Main`:

```csharp
namespace DemoApp;

class Program
{
    static void Main(string[] args)
    {
        Console.WriteLine("Entry: Main");
    }
}
```

File-scoped namespace (C# 10+): `namespace DemoApp;` áp dụng cho cả file, giảm indent.

## 6. Ví dụ

### Cơ bản

Tạo project và xem cấu trúc:

```bash
dotnet new console -n DemoApp
cd DemoApp
ls   # Program.cs, DemoApp.csproj
dotnet build
dotnet run
```

### Trung cấp

Thêm class trong namespace riêng — giải thích: tách logic chào hỏi khỏi điểm vào.

```csharp
// Greeter.cs
namespace DemoApp.Services;

public class Greeter
{
    public string Hello(string name) => $"Xin chào, {name}!";
}
```

```csharp
// Program.cs
using DemoApp.Services;

var greeter = new Greeter();
Console.WriteLine(greeter.Hello("Lan"));
```

### Nâng cao

Tạo solution gồm app + class library:

```bash
dotnet new sln -n DemoSolution
dotnet new console -n DemoApp
dotnet new classlib -n DemoLib
dotnet sln add DemoApp/DemoApp.csproj
dotnet sln add DemoLib/DemoLib.csproj
dotnet add DemoApp/DemoApp.csproj reference DemoLib/DemoLib.csproj
```

Trong `DemoLib` thêm class, trong `DemoApp` gọi class đó rồi `dotnet run --project DemoApp`.

## 7. Lỗi thường gặp

| Lỗi | Nguyên nhân | Cách xử lý |
|-----|-------------|------------|
| `The type or namespace name ... could not be found` | Thiếu `using` / sai namespace / chưa reference project | Thêm `using`, sửa namespace, `dotnet add reference` |
| Nhiều điểm vào | Hai `Main` hoặc top-level + `Main` cùng project | Chỉ giữ một entry point |
| Build nhầm project | Đang đứng sai thư mục / chưa chỉ `--project` | `cd` đúng hoặc `dotnet run --project đường/dẫn.csproj` |
| Sửa `.csproj` sai XML | Tag không đóng | Validate XML, đọc lỗi MSBuild |

## 8. Gỡ lỗi

1. `dotnet build -v n` (normal verbosity) để xem project nào đang build.
2. Mở `.csproj` kiểm tra `TargetFramework` và `OutputType`.
3. Nếu “không thấy class ở project khác”: kiểm tra `ProjectReference` trong `.csproj`.
4. Trong IDE: nhìn Solution Explorer / file tree khớp với namespace bạn khai báo.

## 9. Best practices

- Một class public / một file (thói quen tốt khi mới học).
- Namespace theo: `TênCôngTy.TênApp.LớpChứcNăng`.
- Bật `Nullable` và `ImplicitUsings` như template mặc định.
- Đặt tên project/solution rõ ràng, tránh `ConsoleApp1` lâu dài.

## 10. Bài tập

**Bài 1 — Đọc `.csproj`**  
*Input:* mở file `.csproj` project của bạn.  
*Output:* viết comment đầu `Program.cs` ghi `TargetFramework` và `OutputType` bạn thấy.

**Bài 2 — Namespace thủ công**  
*Input:* không.  
*Output:* tạo class `Calculator` trong namespace `Level01.Tools` với method `Add(int,int)`; từ `Program` gọi và in `2+3=5`.

**Bài 3 — Hai file**  
*Input:* người dùng nhập họ và tên (hai dòng).  
*Output:* class `NameFormatter` trả về `"Họ, Tên"`; `Program` in kết quả.

**Bài 4 — Solution nhỏ**  
*Input:* lệnh CLI.  
*Output:* solution có `App` (console) + `Shared` (classlib); `App` in chuỗi từ class trong `Shared`.

**Bài 5 — Entry point cổ điển**  
*Input:* không.  
*Output:* viết lại `Program.cs` dùng `static void Main` tường minh (không top-level), vẫn chạy được.

## 11. Gợi ý

- Bài 2: nhớ `using Level01.Tools;` hoặc dùng tên đầy đủ `Level01.Tools.Calculator`.
- Bài 4: `dotnet add App/App.csproj reference Shared/Shared.csproj`.
- Top-level và `Main` tường minh **không** trộn trong cùng một project.

## 12. Đáp án

**Bài 1** — Ví dụ ghi chú:

```csharp
// TargetFramework: net8.0
// OutputType: Exe
Console.WriteLine("OK");
```

**Bài 2** — Class và lời gọi:

```csharp
// Calculator.cs
namespace Level01.Tools;

public class Calculator
{
    public int Add(int a, int b) => a + b;
}
```

```csharp
// Program.cs
using Level01.Tools;

var calc = new Calculator();
Console.WriteLine($"2+3={calc.Add(2, 3)}");
```

**Bài 3** — Format họ tên:

```csharp
namespace Level01.Tools;

public class NameFormatter
{
    public string Format(string familyName, string givenName)
        => $"{familyName}, {givenName}";
}
```

```csharp
using Level01.Tools;

Console.Write("Họ: ");
string? family = Console.ReadLine();
Console.Write("Tên: ");
string? given = Console.ReadLine();

var formatter = new NameFormatter();
Console.WriteLine(formatter.Format(family ?? "", given ?? ""));
```

**Bài 4** — Outline lệnh (làm trên shell):

```bash
dotnet new sln -n Mini
dotnet new console -n App
dotnet new classlib -n Shared
dotnet sln add App App.csproj Shared/Shared.csproj
dotnet add App/App.csproj reference Shared/Shared.csproj
```

**Bài 5** — `Main` tường minh:

```csharp
namespace DemoApp;

class Program
{
    static void Main(string[] args)
    {
        Console.WriteLine($"Args count: {args.Length}");
    }
}
```

## 13. Đáp án thay thế

Dùng **global using** (file `GlobalUsings.cs`) thay vì `using` lặp lại:

```csharp
global using Level01.Tools;
```

Khi đó `Program.cs` gọi `Calculator` trực tiếp mà không cần `using` cục bộ.

## 14. Thử thách

Tạo solution 3 project: `Cli` (Exe), `Core` (Library chứa logic), `Core.Tests` (dùng `dotnet new xunit`). Viết 1 unit test cho `Add` và chạy `dotnet test`.

## 15. Ứng dụng thực tế

Solution nhiều project là chuẩn industry: tách `Domain`, `Infrastructure`, `WebApi`, `Tests` — build một lần, tái sử dụng library, CI chạy test riêng.

## 16. Liên hệ Unity

Unity không dùng `.sln` theo cách bạn tạo bằng `dotnet new sln` hàng ngày (Editor tự sinh solution). Nhưng:

- Mỗi script C# vẫn thuộc namespace (khuyến nghị đặt namespace theo folder).
- Assembly Definition (`.asmdef`) trong Unity gần ý tưởng “nhiều project / assembly”.
- Hiểu entry point giúp bạn không tìm `Main` trong game — vòng đời Unity là `Awake`/`Start`/`Update`.

## 17. Kiểm tra kiến thức

1. File nào mô tả target framework của project?  
   **Đáp án:** `*.csproj`

2. Solution dùng để làm gì?  
   **Đáp án:** Gom nhiều project liên quan để quản lý/build chung.

3. `OutputType` = `Exe` nghĩa là gì?  
   **Đáp án:** Xuất chương trình chạy được (executable / app console).

4. Top-level statements thay thế gì?  
   **Đáp án:** Thay cho việc viết tường minh `static void Main` trong nhiều trường hợp.

5. Vì sao cần `ProjectReference`?  
   **Đáp án:** Để project A dùng được kiểu/public API từ project B khi build.
