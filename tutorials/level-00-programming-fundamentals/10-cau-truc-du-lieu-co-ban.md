# Chương 10 — Cấu trúc dữ liệu cơ bản

## 1. Mục tiêu học

- Hiểu cấu trúc dữ liệu là **cách tổ chức thông tin** để thao tác (thêm, lấy, tìm) thuận tiện.
- Nắm khái niệm **mảng (array)**, **danh sách (list)**, **map/dictionary**.
- Biết chọn cấu trúc phù hợp theo nhu cầu (truy cập theo chỉ số vs theo khóa).
- Chưa cần code C# — tập trung tư duy “dữ liệu nằm thế nào”.

## 2. Điều kiện tiên quyết

- Chương 03 (biến), 09 (duyệt tìm kiếm trên dãy phần tử).
- Biết vòng lặp for-each ý tưởng.

## 3. Khái niệm

### Vì sao cần cấu trúc dữ liệu?

Một biến `mau` đủ cho một số. Nhưng 100 enemy thì sao?  
→ Cần “hộp nhiều ngăn” có tổ chức.

### Array (mảng)

Dãy phần tử **cùng ý tưởng kiểu**, mỗi phần tử có **chỉ số** (thường từ 0).

```text
diem:  [10 | 8 | 9 | 7]
chỉ số:  0   1   2   3
```

- Truy cập `diem[2]` rất trực tiếp.  
- Kích thước thường **cố định** khi tạo (ý tưởng cổ điển).

### List (danh sách động)

Giống mảng nhưng **thêm/bớt** phần tử linh hoạt hơn (ý tưởng).

```text
inventory = ["kiếm", "khiên"]
THÊM "thuốc" → ["kiếm", "khiên", "thuốc"]
```

### Map / Dictionary (ánh xạ khóa → giá trị)

Tra cứu theo **khóa** thay vì chỉ số số nguyên.

```text
vang_cua["An"] = 100
vang_cua["Bình"] = 50
// hỏi vàng của An → 100
```

Đời thường: danh bạ điện thoại (tên → số), từ điển (từ → nghĩa).

### So sánh nhanh

| Nhu cầu | Nên nghĩ tới |
|---------|----------------|
| Dãy có thứ tự, truy cập vị trí thứ i | Array / List |
| Thêm bớt thường xuyên | List |
| Tra cứu theo tên/id | Map |
| Chỉ cần tập không trùng (ý tưởng) | Set (học sau) |

## 4. Mô hình tư duy

```text
ARRAY / LIST (theo chỉ số)
index:  0      1      2
      ┌──────┬──────┬──────┐
      │ "A"  │ "B"  │ "C"  │
      └──────┴──────┴──────┘


MAP (theo khóa)
      ┌─────────┬────────┐
      │  khóa   │ giá trị│
      ├─────────┼────────┤
      │ "An"    │  100   │
      │ "Bình"  │   50   │
      └─────────┴────────┘
```

Chọn cấu trúc:

```text
Câu hỏi: "Tôi tìm dữ liệu bằng gì?"
  - bằng vị trí / thứ tự duyệt → list/array
  - bằng mã / tên / id        → map
```

## 5. Cú pháp / Pseudocode

```text
// Array / List
SET ds = [3, 1, 4]
SET x = ds[0]              // 3
SET ds[1] = 99             // [3, 99, 4]
THÊM_CUỐI(ds, 7)           // list: [3, 99, 4, 7]
XÓA_TẠI(ds, 1)

FOR i TỪ 0 ĐẾN length(ds) - 1:
    HIỂN_THỊ ds[i]

FOR MỖI phan_tu TRONG ds:
    HIỂN_THỊ phan_tu

// Map
SET tuoi = {}
SET tuoi["An"] = 20
SET tuoi["Bình"] = 21
HIỂN_THỊ tuoi["An"]
NẾU CÓ_KHÓA(tuoi, "Cuong"):
    ...
NGƯỢC_LẠI:
    HIỂN_THỊ "Không có"
```

## 6. Ví dụ

### Cơ bản

```text
SET item = ["Sword", "Shield", "Potion"]
HIỂN_THỊ item[0]    // Sword
```

### Trung cấp

Đếm số vật phẩm:

```text
FUNCTION dem(ds):
    SET c = 0
    FOR MỖI _ TRONG ds:
        SET c = c + 1
    RETURN c
```

(Thực tế có `length` — bài tập tư duy duyệt.)

### Nâng cao

Bảng điểm bằng map + duyệt:

```text
SET diem = {"An": 8, "Bình": 9, "Chi": 7}
SET tong = 0
SET so_hs = 0
FOR MỖI (ten, d) TRONG diem:
    SET tong = tong + d
    SET so_hs = so_hs + 1
SET tb = tong / so_hs
```

## 7. Lỗi thường gặp

1. Chỉ số ngoài phạm vi: mảng dài 3 thì chỉ số hợp lệ 0..2.
2. Nhầm chỉ số bắt đầu từ 1 (nhiều ngôn ngữ từ **0**).
3. Dùng list khi cần tra cứu theo id liên tục → chậm / code rối; nên map.
4. Giả sử thứ tự duyệt map luôn cố định (tùy ngôn ngữ — đừng phụ thuộc nếu chưa chắc).

## 8. Gỡ lỗi / Kiểm tra hiểu biết

```text
SET a = [10, 20, 30]
HIỂN_THỊ a[3]
```

→ Lỗi runtime (ngoài biên). Phần tử cuối là `a[2]`.

## 9. Best practices

- Chọn cấu trúc theo **câu hỏi tra cứu** chính.
- Đặt tên: `enemies`, `scoreByPlayerId` — rõ nội dung.
- Kiểm tra rỗng trước khi lấy phần tử đầu/cuối.
- Đừng tối ưu sớm: list 20 phần tử thì linear search vẫn ổn.

## 10. Bài tập

**Bài 1.** Mô tả cấu trúc lưu 5 điểm số bài kiểm tra và cách tính trung bình (pseudocode).

**Bài 2.** Thiết kế map cho “mã vật phẩm → số lượng trong túi”. Thêm 1 kiếm, thêm 1 kiếm nữa, lấy số lượng kiếm.

**Bài 3.** Khi nào dùng list thay vì map? Cho 1 ví dụ game.

**Bài 4.** Tìm lỗi: duyệt `FOR i TỪ 0 ĐẾN length(ds)` rồi dùng `ds[i]` — vấn đề gì?

## 11. Gợi ý

- Bài 2: khóa `"sword"`, giá trị số; nếu đã có thì cộng dồn.
- Bài 4: biên trên — thường phải đến `length - 1`.

## 12. Đáp án + Giải thích đáp án

**Bài 1.**

```text
SET diem = [7, 8, 9, 6, 10]
SET tong = 0
FOR MỖI d TRONG diem:
    SET tong = tong + d
SET tb = tong / length(diem)
```

**Bài 2.**

```text
SET tui = {}
SET tui["sword"] = 0
SET tui["sword"] = tui["sword"] + 1
SET tui["sword"] = tui["sword"] + 1
HIỂN_THỊ tui["sword"]    // 2
```

**Bài 3.** Khi cần **danh sách có thứ tự** hoặc duyệt tuần tự mọi phần tử mà không cần khóa: ví dụ danh sách enemy đang trong phòng, hàng đợi skill theo thứ tự cast.

**Bài 4.** `i` chạy tới `length` → truy cập `ds[length]` ngoài biên (chỉ số max là `length - 1`). Sửa: `ĐẾN length(ds) - 1`.

## 13. Đáp án thay thế

Bài 2 có thể dùng list các cặp `(id, so_luong)` — được nhưng tra cứu kém tiện hơn map.

## 14. Thử thách

Thiết kế dữ liệu cho inventory RPG: vừa cần thứ tự hiển thị UI (ô 1, ô 2…), vừa cần hỏi nhanh “có bao nhiêu potion?”. Bạn kết hợp cấu trúc nào? Phác thảo.

## 15. Ứng dụng thực tế

Giỏ hàng, bảng điểm, cache session userId→profile, cấu hình feature flags… đều là array/list/map trong thực tế.

## 16. Liên hệ Unity

`List<GameObject>`, mảng waypoint, `Dictionary` id→data là xương sống nhiều hệ thống Unity (inventory, pooling lookup, UI bind). Chọn sai cấu trúc lúc prototype có thể buộc refactor lớn khi game phình ra.

## 17. Kiểm tra kiến thức

1. Chỉ số mảng thường bắt đầu từ?  
2. Map khác list ở điểm cốt lõi nào?  
3. Thêm phần tử linh hoạt: nghĩ tới cấu trúc nào?  
4. `ds[length]` thường có vấn đề gì?  
5. Danh bạ tên→SĐT giống cấu trúc nào?

### Đáp án kiểm tra kiến thức

1. 0 (trong hầu hết ngôn ngữ phổ biến, kể cả C#).  
2. Map truy cập bằng khóa; list/array bằng chỉ số/thứ tự.  
3. List (danh sách động).  
4. Ngoài biên.  
5. Map / dictionary.
