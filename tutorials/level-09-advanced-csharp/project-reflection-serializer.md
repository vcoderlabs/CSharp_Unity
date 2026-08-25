# Project Level 9 — Reflection-based Serializer

## 1. Mục tiêu học

- Áp dụng **attributes + reflection** để serialize/deserialize object đơn giản
- Hỗ trợ bỏ qua property (`[Ignore]`), đổi tên (`[Name]`), giá trị bắt buộc (`[Required]`)
- Materialize kết quả rõ ràng; xử lý null với NRT
- Hiểu giới hạn serializer reflection (không thay System.Text.Json production)

## 2. Điều kiện tiên quyết

- Hoàn thành các chương Level 9 (đặc biệt 3–4; record/pattern hữu ích)
- .NET 8 console app
- Quen `PropertyInfo`, `Activator`, `string.Split`

## 3. Khái niệm / Yêu cầu sản phẩm

Xây thư viện tối giản **MiniSerializer** định dạng text dòng:

```text
Type=Namespace.Player
Name=Ada
Level=10
Hp=100
```

hoặc dạng gọn key=value (bạn chọn **một** format và document).

### Attributes bắt buộc

```csharp
[AttributeUsage(AttributeTargets.Property)]
public sealed class IgnoreAttribute : Attribute { }

[AttributeUsage(AttributeTargets.Property)]
public sealed class NameAttribute : Attribute
{
    public string Name { get; }
    public NameAttribute(string name) => Name = name;
}

[AttributeUsage(AttributeTargets.Property)]
public sealed class RequiredAttribute : Attribute { }
```

### API tối thiểu

```csharp
public static class MiniSerializer
{
    public static string Serialize<T>(T obj);
    public static T Deserialize<T>(string payload) where T : new();
}
```

### Quy tắc

| Quy tắc | Chi tiết |
|---------|----------|
| Property | Public get; serialize nếu có get; deserialize nếu có set |
| Ignore | Bỏ qua hoàn toàn |
| Name | Tên key trên wire; mặc định = tên property |
| Required | Deserialize thiếu key / null → exception rõ |
| Kiểu hỗ trợ phase 1 | `string`, `int`, `long`, `bool`, `double`, `decimal`, `DateOnly` (tuỳ chọn) |
| Null | `string?` ghi `Name=` (rỗng) hoặc bỏ key — chọn 1 policy |

### Demo types

```csharp
public class Player
{
    [Name("nick")]
    public string Name { get; set; } = "";
    public int Level { get; set; }
    [Ignore]
    public string Password { get; set; } = "";
    [Required]
    public int Hp { get; set; }
}
```

Round-trip: `Deserialize<Player>(Serialize(player))` giữ Name/Level/Hp; Password không lên wire.

## 4. Mô hình tư duy

```text
Serialize:
  type.GetProperties()
    → filter Ignore, CanRead
    → key = NameAttr ?? prop.Name
    → value = prop.GetValue → Convert.ToString / format

Deserialize:
  parse payload → Dictionary<string,string>
  Activator.CreateInstance<T>()
  foreach prop writable:
    key = …
    if missing && Required → throw
    Convert từ string → SetValue
```

## 5. Cú pháp / Skeleton

```bash
dotnet new console -n MiniSerializerApp -f net8.0
cd MiniSerializerApp
```

```text
MiniSerializerApp/
  Program.cs
  Serialization/
    IgnoreAttribute.cs
    NameAttribute.cs
    RequiredAttribute.cs
    MiniSerializer.cs
  Models/Player.cs
```

Bật NRT: `<Nullable>enable</Nullable>`.

## 6. Ví dụ hướng dẫn

### Cơ bản — Serialize

```csharp
public static string Serialize<T>(T obj)
{
    ArgumentNullException.ThrowIfNull(obj);
    var sb = new StringBuilder();
    var type = typeof(T);
    sb.Append("Type=").Append(type.FullName).AppendLine();

    foreach (var prop in type.GetProperties())
    {
        if (!prop.CanRead) continue;
        if (prop.GetCustomAttribute<IgnoreAttribute>() is not null) continue;

        string key = prop.GetCustomAttribute<NameAttribute>()?.Name ?? prop.Name;
        object? value = prop.GetValue(obj);
        sb.Append(key).Append('=').Append(Format(value)).AppendLine();
    }
    return sb.ToString();
}

static string Format(object? value) => value switch
{
    null => "",
    bool b => b ? "true" : "false",
    IFormattable f => f.ToString(null, CultureInfo.InvariantCulture) ?? "",
    _ => value.ToString() ?? ""
};
```

### Trung cấp — Deserialize

```csharp
public static T Deserialize<T>(string payload) where T : new()
{
    var map = Parse(payload);
    var obj = new T();
    var type = typeof(T);

    foreach (var prop in type.GetProperties())
    {
        if (!prop.CanWrite) continue;
        if (prop.GetCustomAttribute<IgnoreAttribute>() is not null) continue;

        string key = prop.GetCustomAttribute<NameAttribute>()?.Name ?? prop.Name;
        bool required = prop.GetCustomAttribute<RequiredAttribute>() is not null;

        if (!map.TryGetValue(key, out var raw))
        {
            if (required)
                throw new InvalidOperationException($"Missing required '{key}'");
            continue;
        }

        object? converted = ConvertTo(prop.PropertyType, raw);
        prop.SetValue(obj, converted);
    }
    return obj;
}
```

### Nâng cao — Parse + Convert + cache

```csharp
static Dictionary<string, string> Parse(string payload)
{
    var dict = new Dictionary<string, string>(StringComparer.Ordinal);
    foreach (var line in payload.Split('\n', StringSplitOptions.RemoveEmptyEntries | StringSplitOptions.TrimEntries))
    {
        if (line.StartsWith("Type=", StringComparison.Ordinal)) continue;
        int i = line.IndexOf('=');
        if (i <= 0) continue;
        dict[line[..i]] = line[(i + 1)..];
    }
    return dict;
}

static object? ConvertTo(Type type, string raw)
{
    if (type == typeof(string)) return raw;
    if (type == typeof(int)) return int.Parse(raw, CultureInfo.InvariantCulture);
    if (type == typeof(bool)) return bool.Parse(raw);
    // … long, double, decimal
    throw new NotSupportedException(type.FullName);
}

// Bonus: cache PropertyInfo[] theo Type trong ConcurrentDictionary
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Password vẫn serialize | Quên `[Ignore]` / CanRead vẫn true | Check attribute |
| `nick` không vào Name | Quên đọc `NameAttribute` | key = attr ?? prop.Name |
| SetValue type mismatch | Parse sai | ConvertTo đúng kiểu |
| `new()` constraint | record positional only init | Dùng class có set; hoặc ctor + reflection nâng cao |
| Culture `1,5` vs `1.5` | Current culture | `InvariantCulture` |

## 8. Gỡ lỗi

1. In payload sau Serialize trước khi Deserialize.
2. Dump keys trong dictionary parse.
3. Log mỗi `SetValue(prop.Name, value)`.
4. Unit test round-trip một `Player` cố định.

## 9. Best practices

- Invariant culture cho số.
- Exception message có tên key/property.
- Cache metadata reflection.
- Không hỗ trợ graph phức tạp (nested object) ở MVP — ghi rõ trong README.
- So sánh nhanh với `JsonSerializer` để thấy production path.

## 10. Bài tập (deliverable)

**Bài 1** — Tạo attributes + `Player` demo.

**Bài 2** — Implement `Serialize` / `Deserialize` round-trip.

**Bài 3** — Thiếu `Hp` khi Required → throw.

**Bài 4** — Console menu: nhập Name/Level/Hp → serialize in ra → deserialize lại in object.

## 11. Gợi ý

- Bắt đầu chỉ `string` + `int`.
- Parse từng dòng `key=value`.
- `ArgumentNullException.ThrowIfNull` ở đầu API.
- Test Ignore trước Required.

## 12. Đáp án

`Program.cs` demo:

```csharp
var p = new Player { Name = "Ada", Level = 10, Hp = 100, Password = "secret" };
string text = MiniSerializer.Serialize(p);
Console.WriteLine(text);

var p2 = MiniSerializer.Deserialize<Player>(text);
Console.WriteLine($"{p2.Name} L{p2.Level} HP{p2.Hp} pwd='{p2.Password}'");
// pwd rỗng/default — không có trên wire
```

(Phần thân `MiniSerializer` theo mục 6.)

## 13. Đáp án thay thế

- Format JSON tối giản tự viết (vẫn reflection) thay key=value.
- Dùng `record class` với `{ get; set; }` thay class.
- Nested object phase 2: serialize property phức tạp bằng đệ quy + prefix `Address.City=`.

## 14. Thử thách

1. Cache `PropertyInfo` + attribute map theo `Type`.
2. Hỗ trợ `enum` và `DateTime` UTC ISO.
3. `Serialize` collection `List<T>` (nhiều block Type=).
4. So benchmark với `System.Text.Json` (Stopwatch) — viết nhận xét trong comment.

## 15. Ứng dụng thực tế

- Hiểu nội bộ serializer hoạt động thế nào
- Tooling editor / save game format đơn giản
- Config file nhỏ nội bộ
- Nền tảng học source generator (bước sau thay reflection)

## 16. Liên hệ Unity

- Save/Load bản prototype trước khi dùng JsonUtility/Newtonsoft
- Custom attribute giống `[SerializeField]` metadata
- Editor window export ScriptableObject ra text
- Runtime: cache reflection; tránh serialize mỗi frame

## 17. Kiểm tra kiến thức

1. `[Ignore]` giúp gì trong serializer?  
   **Đáp án:** Loại property khỏi wire (ví dụ secret/password).

2. Vì sao dùng `NameAttribute`?  
   **Đáp án:** Đổi tên key trên payload tách khỏi tên property C#.

3. `Required` kiểm tra lúc nào?  
   **Đáp án:** Deserialize khi thiếu key/giá trị bắt buộc.

4. Vì sao `InvariantCulture`?  
   **Đáp án:** Tránh lệch dấu thập phân / format theo máy người dùng.

5. Reflection serializer production — hạn chế chính?  
   **Đáp án:** Chậm hơn, dễ lỗi runtime, khó AOT; production thường dùng source-gen / STJ.
