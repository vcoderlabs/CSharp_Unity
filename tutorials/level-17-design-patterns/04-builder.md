# Chương 4 — Builder

## 1. Mục tiêu học

- Xây object phức tạp **từng bước**, fluent API
- Tách construction khỏi representation
- So sánh với telescoping ctor / object initializer
- Unity: build loadout, dialog, procedural room

## 2. Điều kiện tiên quyết

- Class design, immutability cơ bản
- Optional: record C#

## 3. Khái niệm

**Builder** giúp tạo object nhiều phần tùy chọn mà không cần ctor 10 tham số. Director (optional) định nghĩa thứ tự build chuẩn.

## 4. Mô hình tư duy

```text
Builder.WithX().WithY().Build() → Product bất biến
Invalid combination → validate trong Build()
```

## 5. Cú pháp

```csharp
public sealed class WeaponBuilder
{
    private string _name = "Blade";
    private int _dmg = 1;
    private bool _fire;

    public WeaponBuilder Named(string n) { _name = n; return this; }
    public WeaponBuilder Damage(int d) { _dmg = d; return this; }
    public WeaponBuilder Fire() { _fire = true; return this; }
    public Weapon Build() => new(_name, _dmg, _fire);
}
```

## 6. Ví dụ

### Cơ bản

```csharp
var w = new WeaponBuilder().Named("Inferno").Damage(12).Fire().Build();
```

### Trung cấp — Director

```csharp
public static class StarterLoadout
{
    public static Weapon Create() =>
        new WeaponBuilder().Named("Stick").Damage(2).Build();
}
```

### Nâng cao

Immutable product + builder validate (`dmg > 0`). Unity: builder tạo `GameObject` hierarchy (cẩn thận GC).

## 7. Lỗi thường gặp

| Hiện tượng | Cách xử lý |
|------------|------------|
| Builder mutable reuse sai | New builder mỗi lần / Clear |
| Build() partial state | Validate + default rõ |
| Fluent quá dài | Chia bước / Director |

## 8. Gỡ lỗi

Assert sau Build: invariant product. Log chuỗi gọi fluent nếu thiếu field.

## 9. Best practices

- Product prefer immutable.  
- `Build` một lần; đừng sửa product qua builder sau đó.

## 10. Bài tập

**Bài 1** — `QueryBuilder` Select/Where/Limit.  
**Bài 2** — `CharacterBuilder` stats + validate tổng điểm.  
**Bài 3** — So sánh với `record` + `with`.  
**Bài 4** — Builder tạo email MIME multipart (stub).

## 11. Gợi ý

Validate trong `Build()`, không ở từng setter nếu rule liên field.

## 12. Đáp án

```csharp
public sealed class CharacterBuilder
{
    private int _str, _agi, _int;
    public CharacterBuilder Str(int v) { _str = v; return this; }
    public CharacterBuilder Agi(int v) { _agi = v; return this; }
    public CharacterBuilder Int(int v) { _int = v; return this; }
    public Character Build()
    {
        if (_str + _agi + _int != 30) throw new InvalidOperationException("points");
        return new Character(_str, _agi, _int);
    }
}
```

## 13. Đáp án thay thế

Telescoping optional args C# 12 / object initializer nếu object đơn giản — không cần Builder.

## 14. Thử thách

Quest definition builder: objectives + rewards + level gate.

## 15. Ứng dụng thực tế

- `StringBuilder` (biến thể)
- Test data builders
- URL / SQL builders

## 16. Liên hệ Unity

- Procedural dungeon step builder  
- UI screen builder (cẩn thận đừng over-abstract)

## 17. Kiểm tra kiến thức

1. Builder giải quyết gì? **Ctor phức tạp / nhiều optional.**  
2. `Build` nên làm gì thêm? **Validate.**  
3. Director là gì? **Quy trình build chuẩn tái sử dụng.**  
4. Khác Factory? **Factory chọn loại; Builder lắp từng phần.**  
5. Khi bỏ Builder? **Object ít field, không rule phức tạp.**
