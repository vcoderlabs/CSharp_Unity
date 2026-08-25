# Project Level 8 — Data Analytics Console Application

## 1. Mục tiêu học

- Áp dụng Where/Select/GroupBy/OrderBy/Aggregate/Join trong một app thật
- Dùng element operators cho KPI (Count, Sum, Average, Min, Max)
- Paging báo cáo bằng Skip/Take; batch bằng Chunk (tuỳ chọn)
- Materialize đúng lúc; tránh multiple enumeration khi in nhiều báo cáo

## 2. Điều kiện tiên quyết

- Hoàn thành 5 chương Level 8
- .NET 8 console app
- Biết đọc/ghi file text đơn giản (`File.ReadAllLines`) — hoặc hard-code dataset

## 3. Khái niệm / Yêu cầu sản phẩm

Xây **Data Analytics Console** phân tích doanh số bán hàng in-memory (hoặc CSV).

### Model tối thiểu

```csharp
record Sale(
    DateOnly Date,
    string Region,
    string Product,
    string Category,
    int Quantity,
    decimal UnitPrice);
```

`LineTotal => Quantity * UnitPrice` (property hoặc Select).

### Chức năng bắt buộc

| # | Báo cáo | Gợi ý LINQ |
|---|---------|------------|
| 1 | Tổng doanh thu | `Sum` |
| 2 | Doanh thu theo Region | `GroupBy` + `Sum` |
| 3 | Top N sản phẩm theo doanh thu | `GroupBy` Product → OrderByDescending → Take |
| 4 | Lọc theo khoảng ngày + category | `Where` |
| 5 | Join với bảng `ProductInfo` (Category chính thức / Unit) | `Join` hoặc `GroupJoin` |
| 6 | Paging danh sách giao dịch | `OrderBy` + `Skip` + `Take` |
| 7 | Menu console chọn báo cáo | vòng lặp `switch` |

### Dataset

Ít nhất **50–100** dòng sale (hard-code generator hoặc CSV `sales.csv`).

## 4. Mô hình tư duy

```text
Load/Generate sales → List<Sale> (materialize nguồn 1 lần)
                   ↓
            queries trên list (an toàn enumerate nhiều báo cáo)
                   ↓
     Report methods trả IReadOnlyList / in trực tiếp
                   ↓
     Không giữ IEnumerable gắn I/O nếu đọc file mỗi lần duyệt
```

## 5. Cú pháp / Skeleton

```bash
dotnet new console -n DataAnalytics -f net8.0
cd DataAnalytics
```

Cấu trúc đề xuất:

```text
DataAnalytics/
  Program.cs
  Models/Sale.cs
  Models/ProductInfo.cs
  Data/SaleRepository.cs
  Reports/SalesReports.cs
  sales.csv          (tuỳ chọn)
```

## 6. Ví dụ hướng dẫn

### Cơ bản — model + generator

```csharp
record Sale(DateOnly Date, string Region, string Product, string Category, int Quantity, decimal UnitPrice)
{
    public decimal LineTotal => Quantity * UnitPrice;
}

static List<Sale> Generate(int count, Random rng)
{
    string[] regions = ["North", "South", "East", "West"];
    string[] products = ["Sword", "Shield", "Potion", "Bow", "Armor"];
    string[] cats = ["Weapon", "Weapon", "Consumable", "Weapon", "Armor"];

    var list = new List<Sale>(count);
    var start = new DateOnly(2026, 1, 1);
    for (int i = 0; i < count; i++)
    {
        int p = rng.Next(products.Length);
        list.Add(new Sale(
            start.AddDays(rng.Next(0, 90)),
            regions[rng.Next(regions.Length)],
            products[p],
            cats[p],
            rng.Next(1, 6),
            rng.Next(10, 200)));
    }
    return list;
}
```

### Trung cấp — báo cáo

```csharp
static class SalesReports
{
    public static decimal TotalRevenue(IEnumerable<Sale> sales) =>
        sales.Sum(s => s.LineTotal);

    public static IReadOnlyList<(string Region, decimal Revenue)> RevenueByRegion(IEnumerable<Sale> sales) =>
        sales.GroupBy(s => s.Region)
             .Select(g => (g.Key, g.Sum(x => x.LineTotal)))
             .OrderByDescending(x => x.Item2)
             .ToList();

    public static IReadOnlyList<(string Product, decimal Revenue)> TopProducts(IEnumerable<Sale> sales, int n) =>
        sales.GroupBy(s => s.Product)
             .Select(g => (g.Key, g.Sum(x => x.LineTotal)))
             .OrderByDescending(x => x.Item2)
             .Take(n)
             .ToList();

    public static IReadOnlyList<Sale> Page(IEnumerable<Sale> sales, int pageIndex, int pageSize) =>
        sales.OrderBy(s => s.Date).ThenBy(s => s.Product)
             .Skip(pageIndex * pageSize)
             .Take(pageSize)
             .ToList();
}
```

### Nâng cao — Join + Aggregate + menu

```csharp
record ProductInfo(string Product, string OfficialCategory, string Unit);

static IEnumerable<(Sale Sale, ProductInfo Info)> WithInfo(
    IEnumerable<Sale> sales, IEnumerable<ProductInfo> infos) =>
    sales.Join(infos,
        s => s.Product,
        i => i.Product,
        (s, i) => (s, i));

// Ví dụ Aggregate: chuỗi mô tả top region
static string SummarizeTopRegion(IEnumerable<Sale> sales) =>
    sales.GroupBy(s => s.Region)
         .Select(g => new { Region = g.Key, Total = g.Sum(x => x.LineTotal) })
         .OrderByDescending(x => x.Total)
         .Take(3)
         .Aggregate("Top regions:", (acc, x) => acc + $"\n- {x.Region}: {x.Total:C}");
```

Menu tối giản trong `Program.cs`: load data **một lần** → `switch` gọi report.

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Mỗi menu chọn đều chậm dần | Đọc file / generate lại trong query deferred | Materialize `List<Sale>` lúc start |
| Top N sai | Quên GroupBy trước Take | Group theo Product rồi mới sort |
| Paging trùng/thiếu | Không OrderBy ổn định | `OrderBy Date ThenBy Product` |
| Join mất dòng | Tên Product lệch | Chuẩn hóa string / Distinct product list |
| Format tiền | Culture | `ToString("C")` hoặc `N2` + đơn vị |

## 8. Gỡ lỗi

1. In `sales.Count` sau load.
2. Với một report: `ToList` trung gian + `Console.WriteLine` Count từng bước.
3. So khớp Join: `sales.Select(s => s.Product).Distinct()` vs keys `ProductInfo`.
4. Đảm bảo không gọi `Generate()` bên trong property get của query.

## 9. Best practices

- Tách **data** / **reports** / **UI menu**.
- Report methods nhận `IEnumerable<Sale>` nhưng app truyền `IReadOnlyList<Sale>` đã load.
- Trả `IReadOnlyList` từ report (ToList cuối) — dễ test.
- Dùng `record` cho model bất biến.
- CSV: parse lỗi → skip dòng + log (đừng crash cả app).

## 10. Bài tập (deliverable)

**Bài 1** — Tạo solution console net8.0, generate ≥ 80 sales.

**Bài 2** — Implement 4 báo cáo: Total, ByRegion, TopProducts(5), Page.

**Bài 3** — Thêm `ProductInfo` + Join; in Category chính thức khác Category trên sale (nếu cố tình lệch).

**Bài 4** — Menu số; thêm báo cáo Average order value theo Region (`Average`).

## 11. Gợi ý

- Giữ `var sales = Generate(100, new Random(42));` — seed cố định để debug.
- Top products: copy mẫu mục 6.
- Join: 5 `ProductInfo` khớp tên `products[]`.
- Average: `GroupBy` Region → `Average(s => s.LineTotal)`.

## 12. Đáp án

Skeleton `Program.cs`:

```csharp
var sales = Generate(100, new Random(42));
var infos = new List<ProductInfo>
{
    new("Sword", "Weapon", "pcs"),
    new("Shield", "Weapon", "pcs"),
    new("Potion", "Consumable", "bottle"),
    new("Bow", "Weapon", "pcs"),
    new("Armor", "Armor", "set"),
};

while (true)
{
    Console.WriteLine("""
        1) Tổng doanh thu
        2) Theo region
        3) Top 5 sản phẩm
        4) Paging
        0) Thoát
        """);
    switch (Console.ReadLine())
    {
        case "1":
            Console.WriteLine(SalesReports.TotalRevenue(sales));
            break;
        case "2":
            foreach (var r in SalesReports.RevenueByRegion(sales))
                Console.WriteLine($"{r.Region}: {r.Revenue:N0}");
            break;
        case "3":
            foreach (var r in SalesReports.TopProducts(sales, 5))
                Console.WriteLine($"{r.Product}: {r.Revenue:N0}");
            break;
        case "4":
            foreach (var s in SalesReports.Page(sales, 0, 10))
                Console.WriteLine($"{s.Date} {s.Product} {s.LineTotal}");
            break;
        case "0":
            return;
    }
}
```

(Kèm các method ở mục 6.)

## 13. Đáp án thay thế

- Đọc CSV thay generator.
- Query syntax cho RevenueByRegion.
- `Chunk(20)` để in báo cáo theo “trang in ấn”.
- Thêm `Any`/`All` validation dataset trước khi report.

## 14. Thử thách

1. Export báo cáo Top Products ra `top.csv`.
2. Thêm filter tương tác: nhập Region + from/to date rồi mới aggregate.
3. Viết unit test (xUnit) cho `TotalRevenue` với dataset cố định.
4. Cấm multiple enumeration: helper chỉ cho phép duyệt nguồn một lần trong test thử thách chương 5.

## 15. Ứng dụng thực tế

- Dashboard nội bộ nhỏ không cần BI tool
- Tiền xử lý trước khi đưa vào chart library
- Prototype logic trước khi viết SQL/EF tương đương
- Game telemetry offline: phân tích session log

## 16. Liên hệ Unity

- Phân tích log playtest (damage, drop rate) bằng cùng LINQ
- Editor window / tool menu thay console menu
- Tránh chạy analytics nặng runtime — editor hoặc build tool
- ScriptableObject bảng ProductInfo ≈ dimension table

## 17. Kiểm tra kiến thức

1. Vì sao load `List<Sale>` một lần rồi mới report?  
   **Đáp án:** Tránh generate/đọc file lặp; an toàn khi chạy nhiều báo cáo (không multiple I/O).

2. Top N sản phẩm cần GroupBy trước vì sao?  
   **Đáp án:** Cộng doanh thu theo sản phẩm trước khi xếp hạng và Take.

3. Paging giao dịch nên OrderBy thế nào?  
   **Đáp án:** Khóa ổn định (Date, Product/Id) trước Skip/Take.

4. Join với ProductInfo để làm gì?  
   **Đáp án:** Ghép dữ liệu chiều (dimension) — category/unit chuẩn.

5. `Sum(s => s.LineTotal)` thuộc nhóm toán tử nào về execution?  
   **Đáp án:** Immediate — duyệt và trả một giá trị ngay.
