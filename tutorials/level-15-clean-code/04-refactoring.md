# Chương 4 — Refactoring

## 1. Mục tiêu học

- Hiểu refactor = **đổi cấu trúc, giữ hành vi**
- Áp dụng vài kỹ thuật: Rename, Extract Method/Class, Inline, Move, Replace Conditional
- Refactor **từng bước nhỏ** với test (ideal)
- Tránh “rewrite lớn” giả danh refactor

## 2. Điều kiện tiên quyết

- Chương 1–3
- Level 14 khuyến nghị mạnh: có unit test trước khi đụng hot code

## 3. Khái niệm

**Refactoring** (Martin Fowler): cải thiện thiết kế nội bộ mà không đổi hành vi quan sát được từ bên ngoài.

| Là refactor | Không phải refactor |
|-------------|---------------------|
| Đổi tên, tách hàm | Thêm feature mới |
| Chuyển method sang class phù hợp | Fix bug (đôi khi đi kèm, nhưng khác mục tiêu) |
| Thay `switch` bằng strategy **cùng output** | Đổi rule nghiệp vụ |

### Quy trình an toàn

```text
1. Có test (hoặc characterization test)
2. Một bước refactor nhỏ
3. Chạy test
4. Commit / checkpoint
5. Lặp
```

## 4. Mô hình tư duy

```text
Hành vi bên ngoài ════════════════════╗
                                      ║ giữ nguyên
Cấu trúc bên trong ─── refactor ───► ║
```

Nếu vừa refactor vừa thêm feature → tách commit hoặc tách PR tư duy: trước sạch, sau feature.

## 5. Cú pháp

Các thao tác IDE thường dùng:

- **Rename** (F2)
- **Extract Method**
- **Extract Interface**
- **Move Type to File**
- **Inline Method** (khi tách thừa)

Ví dụ Extract Method thủ công:

```csharp
// Trước
public void SavePlayer(Player p)
{
    string json = JsonSerializer.Serialize(p);
    File.WriteAllText(Path.Combine(_dir, p.Id + ".json"), json);
}

// Sau
public void SavePlayer(Player player)
{
    string json = Serialize(player);
    WritePlayerFile(player.Id, json);
}

private static string Serialize(Player player)
    => JsonSerializer.Serialize(player);

private void WritePlayerFile(string playerId, string json)
    => File.WriteAllText(Path.Combine(_dir, playerId + ".json"), json);
```

## 6. Ví dụ

### Cơ bản — Rename + Extract

**Trước:**

```csharp
int C(List<int> a)
{
    int r = 0;
    foreach (var x in a) if (x > 0) r += x;
    return r;
}
```

**Sau:**

```csharp
int SumPositiveNumbers(IEnumerable<int> numbers)
    => numbers.Where(n => n > 0).Sum();
```

### Trung cấp — Replace Magic Number / Conditional

**Trước:**

```csharp
if (user.Level >= 10 && user.Level < 20) multiplier = 1.2;
else if (user.Level >= 20) multiplier = 1.5;
```

**Sau:**

```csharp
decimal multiplier = LoyaltyMultiplier.ForLevel(user.Level);

public static class LoyaltyMultiplier
{
    public static decimal ForLevel(int level) => level switch
    {
        >= 20 => 1.5m,
        >= 10 => 1.2m,
        _ => 1.0m
    };
}
```

### Nâng cao — Extract Class từ God Class (từng phần)

1. Xác định nhóm method liên quan inventory trong `GameManager`
2. Tạo `InventoryService`, move method + fields cần thiết
3. `GameManager` giữ field `InventoryService` và ủy quyền
4. Test xanh sau mỗi move

Đừng chuyển 15 trách nhiệm trong một PR khổng lồ.

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Refactor + feature lẫn lộn | Vội | Tách bước |
| Không test, đổi lớn | Tự tin quá | Characterization tests trước |
| Extract quá vụn | “Clean” cực đoan | Inline lại nếu rối hơn |
| Đổi hành vi “tiện thể” | Scope creep | Ghi bug/feature riêng |
| Rename không hết | Find-replace tay | IDE Rename / analyzer |

## 8. Gỡ lỗi

1. Test đỏ sau Extract → so sánh diff; thường cắt nhầm biến local.
2. Characterization: ghi snapshot output hiện tại thành assert trước khi đụng.
3. Bisect: nếu nhiều bước không commit, khó biết bước nào gãy — commit nhỏ.
4. `git diff` review chính mình: có dòng đổi rule số học không?

## 9. Best practices

- Red–Green–Refactor khi TDD; hoặc Refactor với test xanh sẵn.
- Một ý refactor mỗi commit khi học.
- Prefer IDE automated refactor.
- Giữ public API ổn định; đổi private tự do hơn.
- Document quyết định lớn trong PR description ngắn.

## 10. Bài tập

**Bài 1** — Characterization: hàm `Format(int n)` legacy lạ. Viết 5 Theory assert theo output hiện tại, rồi mới rename/extract.

**Bài 2** — Extract Method từ vòng lặp vừa filter vừa map vừa print.

**Bài 3** — Inline một method 1 dòng đặt tên xấu hơn body.

**Bài 4** — Liệt kê 5 bước refactor God `ShopManager` (chỉ outline, chưa code hết).

## 11. Gợi ý

- Bài 1: chạy thủ công vài input, khóa assert.
- Bài 2: `FilterActive`, `MapToDto`, `WriteLog`.
- Bài 3: nếu `GetX() => _x` và chỉ dùng 1 nơi — có thể inline.
- Bài 4: tách Pricing → Inventory → UI presenter → Persistence.

## 12. Đáp án

**Bài 4 (outline mẫu):**

1. Thêm test cho `Buy`, `Refund`, `ListStock`
2. Extract `PricingCalculator` (pure)
3. Extract `StockLedger`
4. `ShopManager` chỉ orchestration
5. Đổi tên public API cho rõ; chạy full test

**Bài 2 (ý):**

```csharp
var active = FilterActive(items);
var lines = MapToReportLines(active);
WriteReport(lines);
```

## 13. Đáp án thay thế

Dùng Resharper/Rider refactor tools; hoặc `dotnet format` chỉ cho style — style ≠ cấu trúc nhưng đi kèm.

## 14. Thử thách

Refactor module L14 TestedApp (Pricing hoặc Inventory) cho sạch hơn **mà không đổi test** (test vẫn xanh không sửa assert — trừ khi đổi tên symbol).

## 15. Ứng dụng thực tế

- Chuẩn bị thêm feature: refactor nhẹ đường đi trước
- Giảm thời gian review bằng PR refactor riêng
- Strangler: thay dần module legacy

## 16. Liên hệ Unity

- Refactor `Update` phình → extract methods trước khi tách component
- Đổi tên serialized field cẩn thận (`FormerlySerializedAs`)
- Play Mode test / Edit Mode test làm lưới an toàn

## 17. Kiểm tra kiến thức

1. Refactor khác feature thế nào?  
   **Đáp án:** Giữ hành vi; chỉ cải cấu trúc.

2. Vì sao cần bước nhỏ + test?  
   **Đáp án:** Dễ tìm regression; rollback rẻ.

3. Characterization test là gì?  
   **Đáp án:** Khóa hành vi hiện tại bằng assert trước khi đụng legacy.

4. Extract Method giải smell nào?  
   **Đáp án:** Long Method / trùng đoạn / nhiều mức abstraction.

5. Có nên vừa đổi rule giá vừa extract class không?  
   **Đáp án:** Nên tách — khó review và khó biết nguyên nhân fail.
