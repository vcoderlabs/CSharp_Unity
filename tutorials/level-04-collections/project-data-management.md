# Project Level 4 — Data Management System

## 1. Mục tiêu học

- Import dữ liệu CSV đơn giản vào bộ nhớ
- **Chọn đúng cấu trúc** cho từng nhu cầu: duyệt, tra cứu Id, unique, hàng đợi xử lý
- Xây mini hệ thống quản lý dữ liệu console hoàn chỉnh
- Áp dụng Big O đã học vào quyết định thiết kế

## 2. Điều kiện tiên quyết

- Hoàn thành chương 1–7 Level 4
- Biết đọc/ghi file text cơ bản (`File.ReadAllLines`) — đủ cho project này
- .NET 8+ console app

## 3. Khái niệm / Yêu cầu sản phẩm

Bạn xây **Data Management System** cho dữ liệu “sản phẩm” (có thể đổi thành player/item):

| Cột CSV | Kiểu | Ý nghĩa |
|---------|------|---------|
| Id | int | Khóa chính |
| Name | string | Tên |
| Category | string | Nhóm |
| Price | decimal | Giá |
| Stock | int | Tồn kho |

**Chức năng bắt buộc:**

1. Import từ file `.csv`
2. Tra cứu theo **Id** nhanh
3. Liệt kê theo **Category**
4. Tìm theo tên (contains, không phân biệt hoa thường)
5. Báo cáo: số SKU, tổng giá trị tồn (`Price * Stock`)
6. Export lại CSV (sau khi sửa stock)
7. (Tùy chọn) Hàng đợi “đơn hàng chờ xử lý” bằng `Queue`

**Ràng buộc thiết kế (quan trọng hơn UI):**

- Phải giải thích trong README ngắn của project: cấu trúc nào dùng cho việc gì và Big O kỳ vọng.

## 4. Mô hình tư duy

```text
CSV file
   │ ReadAllLines + Parse
   ▼
List<Product>           ← giữ thứ tự import, duyệt báo cáo O(n)
   │
   ├── Dictionary<int, Product>           ← GetById O(1)
   ├── Dictionary<string, List<Product>>  ← theo Category O(1) + list
   └── HashSet<string> names?             ← (tuỳ) unique name check

Sửa Stock qua Id → cập nhật object trong Dictionary
(List cùng reference → đồng bộ)
```

## 5. Cú pháp / Skeleton

Tạo project:

```bash
dotnet new console -n DataManagement -f net8.0
cd DataManagement
```

Mẫu entity:

```csharp
public sealed class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public string Category { get; set; } = "";
    public decimal Price { get; set; }
    public int Stock { get; set; }
}
```

CSV mẫu `products.csv`:

```text
Id,Name,Category,Price,Stock
1,Health Potion,Consumable,10.5,100
2,Iron Sword,Weapon,120,15
3,Mana Potion,Consumable,12,80
4,Wooden Shield,Armor,45,20
```

## 6. Ví dụ hướng dẫn từng phần

### Cơ bản — Parse một dòng

```csharp
static Product ParseLine(string line)
{
    // Đơn giản: không hỗ trợ dấu phẩy trong tên
    var p = line.Split(',');
    return new Product
    {
        Id = int.Parse(p[0]),
        Name = p[1],
        Category = p[2],
        Price = decimal.Parse(p[3]),
        Stock = int.Parse(p[4])
    };
}
```

### Trung cấp — Build indexes

```csharp
sealed class ProductStore
{
    private readonly List<Product> _all = new();
    private readonly Dictionary<int, Product> _byId = new();
    private readonly Dictionary<string, List<Product>> _byCategory =
        new(StringComparer.OrdinalIgnoreCase);

    public void Add(Product p)
    {
        _all.Add(p);
        _byId[p.Id] = p;
        if (!_byCategory.TryGetValue(p.Category, out var list))
        {
            list = new List<Product>();
            _byCategory[p.Category] = list;
        }
        list.Add(p);
    }

    public bool TryGet(int id, out Product? p) => _byId.TryGetValue(id, out p);

    public IReadOnlyList<Product> ByCategory(string category)
        => _byCategory.TryGetValue(category, out var list) ? list : Array.Empty<Product>();
}
```

### Nâng cao — Menu + Queue đơn hàng

```csharp
var pendingOrders = new Queue<int>(); // product Ids cần restock
// Enqueue khi Stock == 0; worker Dequeue để in phiếu nhập kho
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Sai parse decimal | Culture (dấu `,` vs `.`) | `CultureInfo.InvariantCulture` |
| Header CSV bị coi là data | Không skip dòng 1 | Bỏ qua dòng đầu |
| Id trùng | Import hai lần | `Add` kiểm tra `ContainsKey` |
| Sửa Product trong Dict nhưng Category list cũ | Đổi Category sau import | Rebuild index hoặc cấm đổi Category |
| File không tìm thấy | Sai đường dẫn | Dùng path tương đối từ thư mục chạy |

## 8. Gỡ lỗi

1. In số dòng đọc được vs số Product parse thành công.
2. Breakpoint `ParseLine` với dòng lỗi.
3. Sau import: `byId.Count` phải bằng `_all.Count` nếu Id unique.
4. Test tay: GetById, ByCategory, Search name với dataset 4 dòng trước khi scale.

## 9. Best practices

- Tách `CsvLoader`, `ProductStore`, `ConsoleUI` — dễ test.
- Trả về `IReadOnlyList` từ store.
- Không linear search theo Id nếu đã có Dictionary.
- Ghi chú Big O trong comment ngắn tại từng API public.
- CSV production: dùng thư viện CSV khi có quote/escape — ở đây cố ý đơn giản để tập trung collection.

## 10. Bài tập (milestone)

**M1** — Tạo project + class `Product` + file CSV mẫu.  
**M2** — Import + in toàn bộ danh sách.  
**M3** — Dictionary theo Id + lệnh `get <id>`.  
**M4** — Index Category + lệnh `cat <name>`.  
**M5** — Search tên + report tổng giá trị tồn.  
**M6** — Update stock + export CSV.  
**M7** — (Bonus) Queue restock + SortedDictionary thống kê theo Category name.

## 11. Gợi ý

- Menu `while(true)` đọc `Console.ReadLine`, `Split` lệnh.
- Search: duyệt `_all` hoặc duy trì list — n vừa phải O(n) OK; nếu cực lớn mới cần full-text.
- Export: `File.WriteAllLines` với header + từng product.
- `decimal.Parse(s, CultureInfo.InvariantCulture)`.

## 12. Đáp án (khung hoàn chỉnh tối thiểu)

Giải thích: Store giữ List cho duyệt; Dictionary Id cho O(1); Dictionary Category → List để lọc nhóm nhanh.

```csharp
using System.Globalization;
using System.Text;

var store = new ProductStore();
foreach (var p in CsvLoader.Load("products.csv"))
    store.Add(p);

Console.WriteLine($"Loaded {store.Count} products.");
// ... menu gọi store.TryGet / ByCategory / Search / Report / Save

static class CsvLoader
{
    public static IEnumerable<Product> Load(string path)
    {
        var lines = File.ReadAllLines(path);
        for (int i = 1; i < lines.Length; i++)
        {
            if (string.IsNullOrWhiteSpace(lines[i])) continue;
            var c = lines[i].Split(',');
            yield return new Product
            {
                Id = int.Parse(c[0]),
                Name = c[1],
                Category = c[2],
                Price = decimal.Parse(c[3], CultureInfo.InvariantCulture),
                Stock = int.Parse(c[4])
            };
        }
    }

    public static void Save(string path, IEnumerable<Product> products)
    {
        var sb = new StringBuilder();
        sb.AppendLine("Id,Name,Category,Price,Stock");
        foreach (var p in products)
            sb.AppendLine($"{p.Id},{p.Name},{p.Category},{p.Price.ToString(CultureInfo.InvariantCulture)},{p.Stock}");
        File.WriteAllText(path, sb.ToString());
    }
}

sealed class ProductStore
{
    private readonly List<Product> _all = new();
    private readonly Dictionary<int, Product> _byId = new();
    private readonly Dictionary<string, List<Product>> _byCategory =
        new(StringComparer.OrdinalIgnoreCase);

    public int Count => _all.Count;
    public IReadOnlyList<Product> All => _all;

    public void Add(Product p)
    {
        if (_byId.ContainsKey(p.Id))
            throw new InvalidOperationException($"Duplicate Id {p.Id}");
        _all.Add(p);
        _byId[p.Id] = p;
        if (!_byCategory.TryGetValue(p.Category, out var list))
            _byCategory[p.Category] = list = new List<Product>();
        list.Add(p);
    }

    public bool TryGet(int id, out Product? p) => _byId.TryGetValue(id, out p);

    public IReadOnlyList<Product> ByCategory(string category)
        => _byCategory.TryGetValue(category, out var list) ? list : Array.Empty<Product>();

    public List<Product> SearchName(string keyword)
    {
        var result = new List<Product>();
        foreach (var p in _all)
            if (p.Name.Contains(keyword, StringComparison.OrdinalIgnoreCase))
                result.Add(p);
        return result;
    }

    public decimal TotalInventoryValue()
    {
        decimal sum = 0;
        foreach (var p in _all) sum += p.Price * p.Stock;
        return sum;
    }
}

public sealed class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public string Category { get; set; } = "";
    public decimal Price { get; set; }
    public int Stock { get; set; }
}
```

Tự bổ sung vòng menu lệnh (`list`, `get`, `cat`, `search`, `report`, `setstock`, `save`, `quit`).

## 13. Đáp án thay thế

- Chỉ dùng `List` + LINQ (Level 8): ngắn nhưng tra cứu Id O(n) — **không đạt** tinh thần Level 4 nếu n lớn.
- Dùng `SortedDictionary<int, Product>` nếu cần duyệt Id tăng dần; chậm hơn Dictionary một chút.
- SQLite nếu vượt quá in-memory — ngoài phạm vi level.

## 14. Thử thách

1. Sinh CSV 100_000 dòng; so sánh thời gian `GetById` List linear vs Dictionary.  
2. Thêm `HashSet<string>` đảm bảo Name unique khi import.  
3. Restock worker: `Queue<(int id, int qty)>` xử lý batch.

## 15. Ứng dụng thực tế

- Admin tool nội bộ, import catalog
- Prototype trước khi gắn database thật
- Game design: import item table từ CSV/Spreadsheet

## 16. Liên hệ Unity

- Import item balance từ CSV vào `Dictionary<int, ItemDef>` lúc load game
- Addressables/catalog: tương tự index theo key
- Tránh parse CSV trong `Update`; chỉ load ở boot / editor tool
- ScriptableObject là “store” phía designer; runtime vẫn có thể build Dictionary từ SO list

## 17. Kiểm tra kiến thức

1. Vì sao cần Dictionary theo Id thay vì chỉ List?  
   **Đáp án:** Tra cứu O(1) thay vì O(n).

2. List và Dictionary cùng giữ reference Product — sửa Stock ở đâu?  
   **Đáp án:** Sửa một object; cả hai “nhìn thấy” vì cùng reference.

3. Index Category nên kiểu gì?  
   **Đáp án:** Thường `Dictionary<string, List<Product>>`.

4. Search tên substring trên n vừa phải chấp nhận Big O nào?  
   **Đáp án:** O(n) duyệt toàn bộ (hoặc O(n×m) với độ dài tên).

5. Mục tiêu chính của project này là gì?  
   **Đáp án:** Chọn đúng data structure theo thao tác, không chỉ “parse CSV xong”.
