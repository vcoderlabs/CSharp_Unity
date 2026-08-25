# Chương 4 — System.Text.Json

## 1. Mục tiêu học

- Serialize object → JSON và ngược lại với `JsonSerializer`
- Cấu hình `JsonSerializerOptions` (naming, write indented, null)
- Làm việc với `JsonDocument` / `JsonNode` khi không có model cố định
- Serialize async ra Stream

## 2. Điều kiện tiên quyết

- Level 2: class/property
- Chương 2–3: Stream, text file
- .NET 8+ (`System.Text.Json` built-in)

## 3. Khái niệm

**Serialization:** object → dạng lưu/truyền (JSON).  
**Deserialization:** JSON → object.

```csharp
string json = JsonSerializer.Serialize(player);
Player? p = JsonSerializer.Deserialize<Player>(json);
```

| API | Khi nào |
|-----|---------|
| `JsonSerializer` | Có model C# |
| `JsonDocument` | Đọc DOM nhanh, dispose |
| `JsonNode` | JSON mutable linh hoạt |
| Source generator | AOT / perf (nâng cao) |

Nhớ: tên property mặc định **case-sensitive** match; thường bật `PropertyNameCaseInsensitive` và `JsonNamingPolicy.CamelCase`.

## 4. Mô hình tư duy

```text
C# record/class  ←options→  JSON text/utf8 bytes/stream
Attribute [JsonPropertyName] ghi đè tên
```

Không tin JSON từ mạng: validate sau deserialize; giới hạn depth/size khi parse input lạ.

## 5. Cú pháp

Options chuẩn và async stream:

```csharp
var options = new JsonSerializerOptions
{
    PropertyNamingPolicy = JsonNamingPolicy.CamelCase,
    PropertyNameCaseInsensitive = true,
    WriteIndented = true,
    DefaultIgnoreCondition = JsonIgnoreCondition.WhenWritingNull,
};

var player = new Player(1, "Ada", 10);
string json = JsonSerializer.Serialize(player, options);
Player? back = JsonSerializer.Deserialize<Player>(json, options);

await using var fs = File.Create("player.json");
await JsonSerializer.SerializeAsync(fs, player, options);
```

## 6. Ví dụ

### Cơ bản

Record và serialize:

```csharp
public record Player(int Id, string Name, int Level);

var json = JsonSerializer.Serialize(new Player(1, "Ada", 5));
Console.WriteLine(json);
```

### Trung cấp

Đọc file JSON:

```csharp
await using var fs = File.OpenRead("player.json");
var p = await JsonSerializer.DeserializeAsync<Player>(fs, options);
Console.WriteLine(p?.Name);
```

### Nâng cao

JsonNode chỉnh sửa:

```csharp
var node = JsonNode.Parse("""{"id":1,"name":"Ada"}""")!;
node["level"] = 99;
Console.WriteLine(node.ToJsonString(options));
```

Hoặc `JsonDocument`:

```csharp
using var doc = JsonDocument.Parse(json);
int id = doc.RootElement.GetProperty("id").GetInt32();
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Property null sau deserialize | Sai tên / case | CamelCase + case insensitive |
| `JsonException` | JSON hỏng | Validate; try/catch |
| Không serialize field | Chỉ property public | Dùng property hoặc `[JsonInclude]` |
| Cycle object | Tham chiếu vòng | `ReferenceHandler` hoặc DTO phẳng |
| Số long vs double | JSON number | Kiểu C# khớp |

## 8. Gỡ lỗi

1. In JSON `WriteIndented = true`.
2. So sánh tên property với key JSON.
3. `JsonException.Path` chỉ vị trí lỗi.

## 9. Best practices

- DTO riêng cho IO — không expose entity phức tạp vòng.
- Options **tái sử dụng** instance tĩnh (thread-safe sau init).
- Prefer `SerializeAsync` với Stream cho file lớn.
- Version field trong JSON khi schema tiến hóa.
- Production AOT: JSON source generator.

## 10. Bài tập

**Bài 1** — Class `Item { string Name; int Qty }` serialize indented.

**Bài 2** — Deserialize chuỗi JSON vào `Item`.

**Bài 3** — Lưu `List<Item>` ra `items.json`, đọc lại.

**Bài 4** — Dùng `[JsonPropertyName("n")]` map property `Name`.

## 11. Gợi ý

- Bài 1–3: `JsonSerializer` + File stream.
- Bài 4: attribute trên property.

## 12. Đáp án

**Bài 1–2** — Item:

```csharp
public class Item
{
    public string Name { get; set; } = "";
    public int Qty { get; set; }
}

var options = new JsonSerializerOptions { WriteIndented = true };
string json = JsonSerializer.Serialize(new Item { Name = "Potion", Qty = 3 }, options);
Item? item = JsonSerializer.Deserialize<Item>(json, options);
```

**Bài 3** — List file:

```csharp
var list = new List<Item> { new() { Name = "A", Qty = 1 }, new() { Name = "B", Qty = 2 } };
await using (var fs = File.Create("items.json"))
    await JsonSerializer.SerializeAsync(fs, list, options);

await using var read = File.OpenRead("items.json");
var back = await JsonSerializer.DeserializeAsync<List<Item>>(read, options);
```

**Bài 4** — Đổi tên JSON:

```csharp
public class Item
{
    [JsonPropertyName("n")]
    public string Name { get; set; } = "";
    public int Qty { get; set; }
}
```

## 13. Đáp án thay thế

Newtonsoft.Json (`JsonConvert`) nếu codebase cũ — API khác một chút; tutorial này ưu tiên STJ.

## 14. Thử thách

Viết polymorphic JSON: `Animal` / `Dog` / `Cat` với `JsonDerivedType` (.NET 7+).

## 15. Ứng dụng thực tế

- REST API body
- Appsettings / save game
- Message queue payload

## 16. Liên hệ Unity

- Unity thường dùng **Newtonsoft** package hoặc `JsonUtility` (hạn chế: không Dictionary tốt, field serialize).
- `JsonUtility` ≠ System.Text.Json — biết khác biệt khi port code.
- Save `persistentDataPath + "/save.json"` bằng STJ trong non-IL2CPP hạn chế; IL2CPP/AOT cần chú ý reflection — source gen / DTOs đơn giản.

## 17. Kiểm tra kiến thức

1. Namespace JSON mặc định .NET hiện đại?  
   **Đáp án:** `System.Text.Json`.

2. `PropertyNamingPolicy = CamelCase` làm gì?  
   **Đáp án:** Xuất `PlayerName` → `playerName`.

3. Vì sao tái sử dụng `JsonSerializerOptions`?  
   **Đáp án:** Tránh tạo lại tốn kém; options thread-safe sau cấu hình.

4. `JsonDocument` cần gì sau khi dùng?  
   **Đáp án:** `Dispose` / `using` — trả pooled memory.

5. Serialize ra file lớn nên dùng API nào?  
   **Đáp án:** `SerializeAsync` vào Stream.
