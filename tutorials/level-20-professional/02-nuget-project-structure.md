# Chương 2 — NuGet & cấu trúc project

## 1. Mục tiêu học

- Hiểu **NuGet** package là dependency có phiên bản
- Tổ chức **solution** nhiều project (`src`, `tests`)
- Phân biệt ProjectReference vs PackageReference
- Tránh dependency hell cơ bản (version conflict)

## 2. Điều kiện tiên quyết

- `dotnet` CLI
- Level 20 chương 1 (repo sẵn)

## 3. Khái niệm

**NuGet:** kho thư viện (.nupkg). File `.csproj` khai báo:

```xml
<PackageReference Include="Serilog" Version="4.0.0" />
```

**ProjectReference:** tham chiếu project trong solution (code của bạn).

Cấu trúc gợi ý:

```text
/src
  App.Domain/
  App.Application/
  App.Infrastructure/
  App.Api/          (hoặc App.Console)
/tests
  App.UnitTests/
App.sln
Directory.Build.props   (tuỳ chọn)
```

Chưa bắt buộc Clean Architecture đầy đủ — đủ tách **domain logic** khỏi **host** và **tests**.

## 4. Mô hình tư duy

```text
App.Api ──► App.Application ──► App.Domain
   │                │
   └──── App.Infrastructure (implements interfaces)

tests tham chiếu Application/Domain — không cần UI
```

NuGet = “thư viện ngoài”; ProjectReference = “module trong repo”.

## 5. Cú pháp

```bash
dotnet new sln -n ProdApp
dotnet new classlib -n ProdApp.Domain -o src/ProdApp.Domain
dotnet new classlib -n ProdApp.Application -o src/ProdApp.Application
dotnet new webapi -n ProdApp.Api -o src/ProdApp.Api
dotnet new xunit -n ProdApp.UnitTests -o tests/ProdApp.UnitTests

dotnet sln add src/**/*.csproj tests/**/*.csproj
dotnet add src/ProdApp.Application reference src/ProdApp.Domain
dotnet add src/ProdApp.Api package Serilog.AspNetCore
dotnet restore
dotnet build
```

Central package management (tuỳ chọn): `Directory.Packages.props`.

## 6. Ví dụ

### Cơ bản — thêm package

```bash
dotnet add package Newtonsoft.Json
# hoặc System.Text.Json đã in-box — cân nhắc không thêm thừa
```

### Trung cấp — Directory.Build.props

```xml
<Project>
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <TreatWarningsAsErrors>true</TreatWarningsAsErrors>
  </PropertyGroup>
</Project>
```

### Nâng cao — tránh vòng tham chiếu

Api → Application → Domain. Infrastructure → Application/Domain.  
Domain **không** reference Infrastructure.

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| NU1107 conflict | Hai version package | Align version / binding |
| Circular reference | Project A↔B | Đảo dependency / interface ở Domain |
| Commit file `packages/` cũ | Sai toolchain | Dùng PackageReference hiện đại |
| Package thừa | Copy-paste | Định kỳ `dotnet list package` |

## 8. Gỡ lỗi

1. `dotnet list package --outdated`
2. `dotnet nuget locals all --list`
3. Xóa `bin/obj` rồi `restore` khi restore “ma”.
4. Đọc lỗi MSB4057 / project not found — path sai trong sln.

## 9. Best practices

- Pin version có chủ đích; nâng có changelog.
- Ít package hơn: mỗi dependency là bề mặt tấn công + maintenance.
- Một solution cho app; thư viện dùng chung tách repo khi thật sự share.
- README nói rõ `dotnet run --project ...`
- Private feed (Azure Artifacts / GitHub Packages) khi package nội bộ.

## 10. Bài tập

**Bài 1** — Tạo solution 3 project: Domain, Application, Console host.

**Bài 2** — Thêm xUnit project, reference Application, viết 1 test xanh.

**Bài 3** — Thêm `Microsoft.Extensions.Logging.Abstractions` vào Application (chỉ abstractions).

**Bài 4** — Vẽ sơ đồ mũi tên reference hiện tại của solution bạn.

## 11. Gợi ý

- Bài 1: `dotnet new` như mục Cú pháp.
- Bài 3: không kéo Serilog vào Domain.
- Bài 4: ASCII trong README.

## 12. Đáp án

Cấu trúc tối thiểu chấp nhận được:

```text
src/ProdApp.Domain        (entities, interfaces)
src/ProdApp.Application   (use cases)
src/ProdApp.Console       (DI compose, run)
tests/ProdApp.UnitTests
```

Test ví dụ:

```csharp
[Fact]
public void Add_Works() => Assert.Equal(4, 2 + 2);
```

## 13. Đáp án thay thế

Single project cho prototype cực nhỏ — nâng cấp khi có test/DI thật. Modulith folders thay nhiều csproj nếu team nhỏ.

## 14. Thử thách

Bật Central Package Management và chuyển toàn bộ Version sang `Directory.Packages.props`.

## 15. Ứng dụng thực tế

- Microservice: nhiều sln/repo
- Monorepo: nhiều service chung props
- Plugin: load package động (hiếm, phức tạp)

## 16. Liên hệ Unity

- Unity Package Manager ≠ NuGet (có bridge hạn chế)
- asmdef = “project boundary” trong Unity
- Tránh circular asmdef giống circular csproj
- Shared library: DLL / UPM git URL

## 17. Kiểm tra kiến thức

1. PackageReference khác ProjectReference?  
   **Đáp án:** Package từ NuGet; Project là module trong solution.

2. Domain nên reference UI không?  
   **Đáp án:** Không — đảo ngược dependency.

3. `dotnet restore` làm gì?  
   **Đáp án:** Tải/khôi phục dependencies.

4. Vì sao dùng Logging.Abstractions ở Application?  
   **Đáp án:** Không gắn chặt Serilog; Infrastructure/host cấu hình provider.

5. Circular reference xử lý sao?  
   **Đáp án:** Đưa interface vào layer trong; implement ra ngoài.
