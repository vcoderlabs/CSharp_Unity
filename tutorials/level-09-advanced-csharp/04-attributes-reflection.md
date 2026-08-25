# Chương 4 — Attributes & reflection

## 1. Mục tiêu học

- Khai báo và gắn **attribute** (`[Obsolete]`, custom attribute)
- Dùng **reflection** đọc type/property/method lúc runtime
- Đọc custom attribute trên class/property
- Hiểu chi phí reflection và khi nào nên tránh

## 2. Điều kiện tiên quyết

- Level 2: class, property, method
- Level 5: generics cơ bản
- Level 9 chương 3: nullability khi đọc `GetValue` trả `object?`

## 3. Khái niệm

**Attribute** = metadata gắn vào code (declarative). Compiler/runtime/tool đọc để đổi hành vi (serialize, validate, obsolete…).

**Reflection** (`System.Reflection`) = API soi và thao tác metadata/type lúc chạy:

- `typeof(T)`, `obj.GetType()`
- `GetProperties`, `GetMethods`, `GetCustomAttributes`
- `PropertyInfo.GetValue` / `SetValue`
- `Activator.CreateInstance`

| Thành phần | Ví dụ |
|------------|--------|
| Attribute có sẵn | `[Obsolete]`, `[Serializable]`, `[JsonPropertyName]` |
| Custom attribute | `[Author("Ada")]` |
| Reflection đọc | `prop.GetCustomAttribute<AuthorAttribute>()` |

## 4. Mô hình tư duy

```text
Compile-time:  [MinLength(3)] trên property  → metadata trong assembly
Runtime:       Reflection đọc MinLength → validate / serialize

Serializer đơn giản:
  foreach PropertyInfo p in type.GetProperties()
      if không bỏ qua → lấy tên + GetValue → ghi text/json
```

## 5. Cú pháp

```csharp
[AttributeUsage(AttributeTargets.Property | AttributeTargets.Class)]
public sealed class NameAttribute : Attribute
{
    public string Value { get; }
    public NameAttribute(string value) => Value = value;
}

[Name("hero")]
public class Player
{
    [Name("nick")]
    public string Name { get; set; } = "";
}

Type t = typeof(Player);
var props = t.GetProperties();
object? instance = Activator.CreateInstance(t);
object? val = props[0].GetValue(instance);
```

## 6. Ví dụ

### Cơ bản — Obsolete + đọc attribute

```csharp
[Obsolete("Dùng NewPing")]
static void OldPing() { }

static void NewPing() { }

var attr = typeof(Program)
    .GetMethod(nameof(OldPing))!
    .GetCustomAttribute<ObsoleteAttribute>();
Console.WriteLine(attr?.Message);
```

### Trung cấp — custom attribute trên property

```csharp
[AttributeUsage(AttributeTargets.Property)]
public sealed class IgnoreAttribute : Attribute { }

public class UserDto
{
    public string Name { get; set; } = "";
    [Ignore]
    public string PasswordHash { get; set; } = "";
}

static void PrintPublic(object obj)
{
    foreach (var p in obj.GetType().GetProperties())
    {
        if (p.GetCustomAttribute<IgnoreAttribute>() is not null)
            continue;
        Console.WriteLine($"{p.Name}={p.GetValue(obj)}");
    }
}
```

### Nâng cao — tạo instance + set property theo tên

```csharp
static object? CreateAndSet(Type type, string propName, object? value)
{
    var obj = Activator.CreateInstance(type)
        ?? throw new InvalidOperationException("no ctor");
    var prop = type.GetProperty(propName)
        ?? throw new ArgumentException(propName);
    if (!prop.CanWrite)
        throw new InvalidOperationException("readonly");
    prop.SetValue(obj, value);
    return obj;
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| `GetProperty` null | Sai tên / non-public | BindingFlags, đúng tên |
| `SetValue` fail | Kiểu value sai / no setter | Convert / kiểm `CanWrite` |
| Attribute không thấy | Sai `AttributeUsage` / Inherited | Targets đúng; `inherit: true` |
| Chậm | Reflection trong hot loop | Cache `PropertyInfo`, tránh mỗi frame |
| `CreateInstance` fail | Không có ctor không tham số | Factory / ctor info |

## 8. Gỡ lỗi

1. In `type.FullName` và danh sách `GetProperties().Select(p => p.Name)`.
2. Exception `TargetException` / `ArgumentException` từ SetValue — kiểm kiểu.
3. Đặt breakpoint sau `GetCustomAttributes`.
4. Unit test attribute mapping với 1 class mẫu.

## 9. Best practices

- Cache `PropertyInfo[]` theo `Type` trong dictionary tĩnh.
- Prefer source generators / compile-time cho serializer production (System.Text.Json source gen).
- Attribute mỏng: data chỉ cần cho tool; logic nặng không nhét trong attribute.
- `AttributeUsage` siết target — tránh gắn lung tung.
- Không dùng reflection thay polymorphism khi biết kiểu lúc biên dịch.

## 10. Bài tập

**Bài 1** — Tạo `[Team("Core")]` gắn lên class; đọc và in.

**Bài 2** — Attribute `[Column("user_name")]` trên property; in map property → column.

**Bài 3** — Method in mọi property public writable của object.

**Bài 4** — Bỏ qua property có `[Ignore]`.

## 11. Gợi ý

- Kế thừa `Attribute`; sealed class thường dùng.
- `GetCustomAttribute<T>()`.
- Bài 3: `CanRead`/`CanWrite`.

## 12. Đáp án

```csharp
[AttributeUsage(AttributeTargets.Class)]
sealed class TeamAttribute : Attribute
{
    public string Name { get; }
    public TeamAttribute(string name) => Name = name;
}

[AttributeUsage(AttributeTargets.Property)]
sealed class ColumnAttribute : Attribute
{
    public string Name { get; }
    public ColumnAttribute(string name) => Name = name;
}

[AttributeUsage(AttributeTargets.Property)]
sealed class IgnoreAttribute : Attribute { }

[Team("Core")]
class User
{
    [Column("user_name")]
    public string Name { get; set; } = "";
    [Ignore]
    public string Secret { get; set; } = "";
}

static void Demo()
{
    var team = typeof(User).GetCustomAttribute<TeamAttribute>();
    Console.WriteLine(team?.Name);

    foreach (var p in typeof(User).GetProperties())
    {
        if (p.GetCustomAttribute<IgnoreAttribute>() is not null) continue;
        var col = p.GetCustomAttribute<ColumnAttribute>()?.Name ?? p.Name;
        Console.WriteLine($"{p.Name} -> {col}");
    }
}
```

## 13. Đáp án thay thế

Dùng `GetCustomAttributes(typeof(ColumnAttribute), false).FirstOrDefault()` kiểu non-generic (API cũ). Metadata bằng dictionary thủ công thay attribute nếu team không thích annotation.

## 14. Thử thách

Viết `ValidateRequired(object obj)`: property có `[Required]` mà `GetValue` null/`string.IsNullOrWhiteSpace` → thu thập lỗi vào `List<string>`.

## 15. Ứng dụng thực tế

- JSON/XML serializers
- ORMs mapping column
- DI container đăng ký
- Test frameworks, analyzers (Roslyn cũng đọc attribute)

## 16. Liên hệ Unity

- `[SerializeField]`, `[Header]`, `[Range]` — attribute editor
- Custom `PropertyAttribute` + drawer
- Reflection tìm mọi `MonoBehaviour` implements interface (editor tool)
- Runtime hot path: tránh GetProperties mỗi `Update`

## 17. Kiểm tra kiến thức

1. Attribute dùng để làm gì?  
   **Đáp án:** Gắn metadata khai báo cho code để tool/runtime đọc.

2. `typeof(Player)` khác `player.GetType()`?  
   **Đáp án:** `typeof` biết lúc biên dịch; `GetType()` theo instance runtime (có thể kiểu con).

3. Cách đọc attribute generic?  
   **Đáp án:** `GetCustomAttribute<MyAttribute>()`.

4. Vì sao cache `PropertyInfo`?  
   **Đáp án:** Reflection đắt; tránh lặp lookup.

5. `Activator.CreateInstance` cần gì thường gặp?  
   **Đáp án:** Constructor public không tham số (hoặc truyền args phù hợp).
