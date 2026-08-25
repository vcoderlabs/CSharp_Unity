# Chương 1 — C# và hệ sinh thái .NET

## 1. Mục tiêu học

- Phân biệt C#, .NET SDK, Runtime (CLR), Compiler
- Hiểu vai trò của `dotnet` CLI trong quy trình lập trình hàng ngày
- Cài SDK, tạo project console và chạy chương trình đầu tiên

## 2. Điều kiện tiên quyết

- Đã biết khái niệm chương trình, biến, luồng điều khiển (Level 0 hoặc kinh nghiệm ngôn ngữ khác)
- Có quyền cài phần mềm trên máy; biết mở terminal (PowerShell / Terminal / bash)

## 3. Khái niệm

**C#** là ngôn ngữ lập trình do Microsoft thiết kế: cú pháp gần C/Java, hỗ trợ OOP mạnh, dùng rộng rãi cho backend, desktop, mobile và **Unity**.

**.NET** là nền tảng chạy mã C# (và F#, VB). Bạn viết C# → compiler dịch thành **IL (Intermediate Language)** → **CLR (Common Language Runtime)** thực thi IL trên máy.

| Thành phần | Vai trò ngắn gọn |
|------------|------------------|
| **SDK** | Bộ công cụ phát triển: compiler, thư viện, `dotnet` CLI |
| **Runtime** | Môi trường chạy chương trình đã biên dịch |
| **CLR** | “máy ảo” của .NET: quản lý bộ nhớ (GC), nạp assembly, JIT |
| **Compiler (`csc` / Roslyn)** | Biến `.cs` → IL trong file `.dll` / `.exe` |
| **`dotnet` CLI** | Lệnh dòng lệnh: tạo project, build, run, test, publish |

**SDK vs Runtime:** SDK dùng để *viết* phần mềm; Runtime chỉ cần để *chạy*. Trên máy học, cài **SDK** là đủ (SDK đã kèm runtime).

## 4. Mô hình tư duy

Hình dung pipeline:

```text
Bạn gõ C# (.cs)
        │
        ▼
   Compiler (Roslyn)
        │
        ▼
   IL trong assembly (.dll)
        │
        ▼
   CLR + JIT → mã máy CPU
        │
        ▼
   Chương trình chạy
```

`dotnet run` = “biên dịch nếu cần + chạy ngay”. Bạn ít khi gọi compiler thủ công.

## 5. Cú pháp (C# thật)

Chương trình console tối giản (.NET 6+ top-level statements — file `Program.cs` có thể không cần `Main` tường minh):

```csharp
Console.WriteLine("Xin chào C# và .NET!");
```

Hoặc kiểu cổ điển (vẫn hợp lệ):

```csharp
namespace HelloApp;

class Program
{
    static void Main(string[] args)
    {
        Console.WriteLine("Xin chào C# và .NET!");
    }
}
```

## 6. Ví dụ

### Cơ bản

In thông tin phiên bản runtime đang chạy:

```csharp
Console.WriteLine($"Runtime: {Environment.Version}");
Console.WriteLine($"OS: {Environment.OSVersion}");
Console.WriteLine("Hello, .NET!");
```

### Trung cấp

Đọc đối số dòng lệnh (`args`):

```csharp
// Chạy: dotnet run -- Alice
if (args.Length == 0)
{
    Console.WriteLine("Cách dùng: dotnet run -- <tên>");
    return;
}

Console.WriteLine($"Xin chào, {args[0]}!");
```

### Nâng cao

In các biến môi trường liên quan .NET (hữu ích khi debug máy khác nhau):

```csharp
Console.WriteLine($"DOTNET_ROOT: {Environment.GetEnvironmentVariable("DOTNET_ROOT") ?? "(không set)"}");
Console.WriteLine($"FrameworkDescription: {System.Runtime.InteropServices.RuntimeInformation.FrameworkDescription}");
Console.WriteLine($"ProcessArchitecture: {System.Runtime.InteropServices.RuntimeInformation.ProcessArchitecture}");
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân thường gặp | Cách xử lý |
|------------|------------------------|------------|
| `dotnet: command not found` | SDK chưa cài / PATH sai | Cài lại SDK, mở lại terminal |
| Build fail sau khi sửa code | Lỗi cú pháp C# | Đọc dòng lỗi trong terminal |
| Chạy được nhưng “không đổi” | Đang chạy binary cũ / sai thư mục | `cd` đúng project, `dotnet run` lại |
| Nhầm .NET Framework cũ | Tài liệu Windows cũ (4.x) | Học **.NET (Core) 8+**, không phải Framework 4.8 |

## 8. Gỡ lỗi

1. Chạy `dotnet --info` — xem SDK/runtime nào đang được dùng.
2. Đọc thông báo lỗi từ trên xuống: file + số dòng thường đúng vị trí sai.
3. Thử `dotnet build` tách khỏi `run` để chỉ xem lỗi biên dịch.
4. Nếu nghi ngờ cache: `dotnet clean` rồi `dotnet build`.

## 9. Best practices

- Cài **một SDK LTS** (.NET 8) trước; nâng cấp có chủ đích.
- Luôn làm việc trong thư mục có file `.csproj`.
- Dùng `dotnet new` thay vì copy project tay khi mới học.
- Ghi chú lệnh bạn hay dùng vào một file cheat-sheet cá nhân.

## 10. Bài tập

**Bài 1 — Kiểm tra môi trường**  
*Input:* không.  
*Output:* in ra phiên bản `dotnet --version` (làm trên terminal) và trong chương trình in `Environment.Version`.

**Bài 2 — Hello có tên**  
*Input:* người dùng nhập một dòng tên từ bàn phím (`Console.ReadLine`).  
*Output:* `Xin chào, <tên>!`

**Bài 3 — Đối số CLI**  
*Input:* `dotnet run -- 3 5` (hai số nguyên).  
*Output:* tổng của hai số; nếu thiếu đối số thì in hướng dẫn.

**Bài 4 — Thông tin máy**  
*Input:* không.  
*Output:* in OS, số lượng processor (`Environment.ProcessorCount`), thư mục hiện tại (`Environment.CurrentDirectory`).

**Bài 5 — Script lệnh**  
*Input:* không (làm trên shell).  
*Output:* tạo project tên `EnvProbe`, chạy được, commit danh sách lệnh bạn đã gõ vào comment đầu `Program.cs`.

## 11. Gợi ý

- Bài 2: `var name = Console.ReadLine();` — nhớ kiểm tra `null`.
- Bài 3: `int.Parse(args[0])` chỉ dùng khi chắc input đúng; sau sẽ học `TryParse`.
- Phân biệt lệnh **shell** (`dotnet --version`) và **code C#** (`Environment.Version`).

## 12. Đáp án

**Bài 1** — In phiên bản runtime trong C#:

```csharp
Console.WriteLine($"CLR/Runtime version: {Environment.Version}");
```

**Bài 2** — Đọc tên và chào:

```csharp
Console.Write("Nhập tên: ");
string? name = Console.ReadLine();
if (string.IsNullOrWhiteSpace(name))
{
    Console.WriteLine("Bạn chưa nhập tên.");
}
else
{
    Console.WriteLine($"Xin chào, {name.Trim()}!");
}
```

**Bài 3** — Cộng hai đối số:

```csharp
if (args.Length < 2)
{
    Console.WriteLine("Cách dùng: dotnet run -- <a> <b>");
    return;
}

int a = int.Parse(args[0]);
int b = int.Parse(args[1]);
Console.WriteLine($"Tổng = {a + b}");
```

**Bài 4** — Thông tin máy:

```csharp
Console.WriteLine($"OS: {Environment.OSVersion}");
Console.WriteLine($"CPU count: {Environment.ProcessorCount}");
Console.WriteLine($"CWD: {Environment.CurrentDirectory}");
```

**Bài 5** — Các lệnh shell mẫu (ghi chú):

```csharp
// mkdir EnvProbe && cd EnvProbe
// dotnet new console -n EnvProbe
// cd EnvProbe
// dotnet run
Console.WriteLine("EnvProbe OK");
```

## 13. Đáp án thay thế

Bài 3 dùng `int.TryParse` an toàn hơn (sẽ gặp lại ở chương nullable / xử lý input):

```csharp
if (args.Length < 2
    || !int.TryParse(args[0], out int a)
    || !int.TryParse(args[1], out int b))
{
    Console.WriteLine("Cách dùng: dotnet run -- <số> <số>");
    return;
}

Console.WriteLine($"Tổng = {a + b}");
```

## 14. Thử thách

Viết chương trình in bảng so sánh ngắn: *SDK / Runtime / CLR / Compiler / CLI* — mỗi dòng một vai trò (tiếng Việt). Chạy bằng `dotnet run`.

## 15. Ứng dụng thực tế

Mọi backend ASP.NET, tool CLI nội bộ, bot Discord/Telegram bằng C#, và pipeline CI (`dotnet test` / `dotnet publish`) đều dựa trên SDK + CLI bạn vừa học.

## 16. Liên hệ Unity

Unity dùng C# nhưng **không** dùng `dotnet new console` làm workflow chính — bạn viết script gắn GameObject. Tuy vậy:

- Cú pháp C# giống hệt.
- Hiểu CLR/GC giúp giải thích “spike” FPS khi GC chạy (học sâu ở Level 10).
- Một số package / editor tool hiện đại vẫn dùng `dotnet` bên ngoài Unity.

Học console trước giúp bạn tập trung vào ngôn ngữ, không bị lệch sang Scene/Inspector quá sớm.

## 17. Kiểm tra kiến thức

1. **SDK** khác **Runtime** chỗ nào?  
   **Đáp án:** SDK để phát triển (compiler + CLI + thư viện); Runtime để chạy chương trình.

2. **CLR** làm gì?  
   **Đáp án:** Thực thi IL, quản lý bộ nhớ (GC), cung cấp dịch vụ runtime chung.

3. Lệnh nào tạo project console?  
   **Đáp án:** `dotnet new console`

4. `dotnet run` làm gì?  
   **Đáp án:** Biên dịch (nếu cần) và chạy project hiện tại.

5. C# được biên dịch ra gì trước khi JIT?  
   **Đáp án:** IL (Intermediate Language) trong assembly.
