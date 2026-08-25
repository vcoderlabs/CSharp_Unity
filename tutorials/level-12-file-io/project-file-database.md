# Project Level 12 — File-based Database

## 1. Mục tiêu học

- Thiết kế lưu trữ đơn giản trên file (JSON array hoặc JSON Lines)
- CRUD: Create / Read / Update / Delete
- Atomic write (ghi temp + replace) tránh file hỏng khi crash
- Kết hợp Stream + System.Text.Json + async

## 2. Điều kiện tiên quyết

- Hoàn thành 4 chương Level 12
- Level 5: generic repository mindset
- `dotnet new console -n FileDatabase -f net8.0`

## 3. Khái niệm / Yêu cầu sản phẩm

Xây **FileDb** lưu entity:

```csharp
public sealed class Product
{
    public Guid Id { get; set; }
    public string Name { get; set; } = "";
    public decimal Price { get; set; }
    public DateTimeOffset UpdatedAt { get; set; }
}
```

API tối thiểu:

| Method | Mô tả |
|--------|-------|
| `Task InsertAsync(Product p)` | Thêm; Id trùng → lỗi |
| `Task<Product?> GetAsync(Guid id)` | Tìm theo Id |
| `Task<IReadOnlyList<Product>> GetAllAsync()` | Tất cả |
| `Task<bool> UpdateAsync(Product p)` | Cập nhật |
| `Task<bool> DeleteAsync(Guid id)` | Xóa |
| `Task SaveAsync()` | Flush nếu có cache |

**Yêu cầu kỹ thuật:**

1. File `data/products.json` (array) **hoặc** `products.jsonl` (một JSON/dòng).
2. Ghi an toàn: viết `products.json.tmp` → `File.Replace` / Move overwrite.
3. `JsonSerializerOptions` camelCase, indented (nếu array).
4. Console menu demo CRUD.

## 4. Mô hình tư duy

```text
Load:  file → List<Product> (memory)
Mutate: list in memory
Save:  serialize → .tmp → replace gốc

JSONL: append-friendly; update/delete = rewrite file
Array JSON: đơn giản cho dataset nhỏ (< vài nghìn)
```

Khuyến nghị bài lab: **JSON array** + rewrite atomic.

## 5. Cú pháp / Skeleton

```bash
dotnet new console -n FileDatabase -f net8.0
```

```csharp
sealed class FileProductStore
{
    private readonly string _path;
    private readonly JsonSerializerOptions _json;
    private List<Product> _items = new();

    public FileProductStore(string path, JsonSerializerOptions json)
    {
        _path = path;
        _json = json;
    }

    public async Task LoadAsync(CancellationToken ct = default) { /* ... */ }
    public async Task SaveAsync(CancellationToken ct = default) { /* ... */ }
    // Insert/Get/Update/Delete...
}
```

## 6. Ví dụ hướng dẫn

Load / Save atomic:

```csharp
public async Task LoadAsync(CancellationToken ct = default)
{
    Directory.CreateDirectory(Path.GetDirectoryName(_path)!);
    if (!File.Exists(_path))
    {
        _items = new();
        return;
    }
    await using var fs = File.OpenRead(_path);
    _items = await JsonSerializer.DeserializeAsync<List<Product>>(fs, _json, ct) ?? new();
}

public async Task SaveAsync(CancellationToken ct = default)
{
    string? dir = Path.GetDirectoryName(_path);
    if (!string.IsNullOrEmpty(dir)) Directory.CreateDirectory(dir);
    string tmp = _path + ".tmp";
    await using (var fs = File.Create(tmp))
        await JsonSerializer.SerializeAsync(fs, _items, _json, ct);
    File.Move(tmp, _path, overwrite: true);
}

public Task InsertAsync(Product p, CancellationToken ct = default)
{
    if (_items.Any(x => x.Id == p.Id)) throw new InvalidOperationException("duplicate");
    p.UpdatedAt = DateTimeOffset.UtcNow;
    _items.Add(p);
    return SaveAsync(ct);
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| File JSON hỏng sau crash | Ghi đè trực tiếp | Ghi `.tmp` rồi replace |
| Mất dữ liệu | Quên Save | Save sau mỗi mutate hoặc batch rõ |
| Race 2 process | Hai app một file | FileStream lock / nêu giới hạn “single process” |
| Decimal JSON | Culture | STJ xử lý số OK; đừng qua string culture-sensitive |

## 8. Gỡ lỗi

1. Mở `products.json` xem indented.
2. Nếu deserialize null → file rỗng/`null` — xử lý `?? new()`.
3. Giữ backup `.bak` trước Save khi đang dev.

## 9. Best practices

- Document: **không** phải DB đa user — lab learning.
- Id: `Guid.NewGuid()` khi Insert nếu chưa có.
- Tách DTO file vs domain nếu sau này đổi storage.
- Có thể thêm `SemaphoreSlim(1,1)` nếu async API gọi song song trong một process.

## 10. Bài tập (milestone)

1. **M1** — Model + serialize mẫu ra file.
2. **M2** — `LoadAsync` / `SaveAsync` atomic.
3. **M3** — Insert / Get / GetAll.
4. **M4** — Update / Delete.
5. **M5** — Menu console; xử lý duplicate Id.

## 11. Gợi ý

- M1: một `Product` ghi file.
- M2: copy pattern mục 6.
- M3–4: thao tác `List` rồi `SaveAsync`.
- M5: `while(true)` switch input.

## 12. Đáp án

Insert/Update/Delete cốt lõi:

```csharp
public async Task<Product?> GetAsync(Guid id)
    => _items.FirstOrDefault(x => x.Id == id);

public async Task<bool> UpdateAsync(Product p, CancellationToken ct = default)
{
    int i = _items.FindIndex(x => x.Id == p.Id);
    if (i < 0) return false;
    p.UpdatedAt = DateTimeOffset.UtcNow;
    _items[i] = p;
    await SaveAsync(ct);
    return true;
}

public async Task<bool> DeleteAsync(Guid id, CancellationToken ct = default)
{
    int removed = _items.RemoveAll(x => x.Id == id);
    if (removed == 0) return false;
    await SaveAsync(ct);
    return true;
}
```

Menu tối giản:

```csharp
var store = new FileProductStore(Path.Combine("data", "products.json"), options);
await store.LoadAsync();
// switch: 1 add, 2 list, 3 update price, 4 delete, 0 exit
```

## 13. Đáp án thay thế

**JSON Lines:** mỗi dòng một product; `Insert` append; `Update`/`Delete` đọc tất cả dòng rewrite. Tốt hơn khi append-heavy.

## 14. Thử thách

- Index phụ: `Dictionary<Guid, int>` sau Load để Get O(1).
- Export CSV.
- Soft delete: flag `IsDeleted` thay vì xóa cứng.

## 15. Ứng dụng thực tế

- Prototype trước khi có SQLite/EF
- Local settings store
- Tool admin offline

## 16. Liên hệ Unity

- Save game inventory tương tự `products.json` trong `persistentDataPath`.
- Atomic replace tránh save hỏng khi kill app giữa chừng.
- Mobile: async save khi pause (`OnApplicationPause`).

## 17. Kiểm tra kiến thức

1. Vì sao ghi `.tmp` rồi Move?  
   **Đáp án:** Giảm nguy cơ file gốc nửa chừng khi crash.

2. File DB lab này an toàn đa process?  
   **Đáp án:** Không đảm bảo — cần lock/DB thật.

3. CRUD gồm những gì?  
   **Đáp án:** Create, Read, Update, Delete.

4. Khi nào chọn JSONL?  
   **Đáp án:** Append nhiều, dataset lớn hơn một chút, update ít.

5. `Guid` làm Id có lợi gì?  
   **Đáp án:** Unique không phụ thuộc auto-increment file.
