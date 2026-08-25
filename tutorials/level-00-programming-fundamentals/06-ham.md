# Chương 06 — Hàm

## 1. Mục tiêu học

- Hiểu **hàm** là khối lệnh có tên, nhận đầu vào (tham số), có thể trả kết quả.
- Biết lý do tách hàm: tái sử dụng, dễ đọc, dễ kiểm tra.
- Viết được hàm pseudocode với tham số và giá trị trả về.
- Phân biệt biến cục bộ (trong hàm) và ý tưởng phạm vi (scope).

## 2. Điều kiện tiên quyết

- Chương 03–05 (biến, biểu thức, if/loop).
- Biết viết một đoạn logic liền mạch.

## 3. Khái niệm

### Hàm là gì?

Đời thường: công thức nấu phở là một “hàm” — **đầu vào** (nguyên liệu), **các bước**, **đầu ra** (tô phở).  
Bạn không viết lại toàn bộ công thức mỗi lần; chỉ **gọi** “nấu phở”.

Trong lập trình:

```text
FUNCTION cong(a, b):
    RETURN a + b

SET tong = cong(2, 3)   // tong = 5
```

### Thành phần

| Thành phần | Ý nghĩa |
|------------|---------|
| Tên hàm | Hành động: `tinh_sat_thuong`, `in_menu` |
| Tham số | Đầu vào |
| Thân hàm | Các bước |
| Giá trị trả về | Kết quả `RETURN` (có thể không trả — chỉ làm việc) |

### Tại sao cần hàm?

1. **Không lặp code** (DRY).  
2. **Đặt tên ý nghĩa** — đọc như chuyện.  
3. **Sửa một chỗ** — mọi nơi gọi được hưởng.  
4. **Kiểm tra từng phần** nhỏ dễ hơn cả chương trình.

### Scope (phạm vi) — trực giác

Biến tạo trong hàm thường **chỉ sống trong hàm**. Bên ngoài không thấy trực tiếp.

## 4. Mô hình tư duy

```text
Người gọi                         Hàm
─────────                         ───
SET kq = tinh_BMI(70, 1.75) ──► nhận can=70, cao=1.75
                                 tính bmi
                          ◄── RETURN bmi
// kq nhận giá trị trả về


Stack gọi đơn giản:

main
 └─► tinh_BMI
      └─► (tính toán, return)
 main tiếp tục
```

## 5. Cú pháp / Pseudocode

```text
FUNCTION ten_ham(tham_so1, tham_so2):
    // biến cục bộ
    SET tam = ...
    RETURN kết_quả

// Hàm không trả về (thủ tục)
FUNCTION in_loi(msg):
    HIỂN_THỊ "Lỗi:", msg
    // không RETURN giá trị

// Gọi
SET x = ten_ham(1, 2)
in_loi("Thiếu máu")
```

## 6. Ví dụ

### Cơ bản

```text
FUNCTION chao(ten):
    HIỂN_THỊ "Xin chào,", ten

chao("An")
chao("Bình")
```

### Trung cấp

```text
FUNCTION max2(a, b):
    NẾU a >= b:
        RETURN a
    NGƯỢC_LẠI:
        RETURN b

SET m = max2(10, 7)   // 10
```

### Nâng cao

Tách logic combat:

```text
FUNCTION tinh_sat_thuong(cong, giap_doi_phuong):
    SET raw = cong * (1 - giap_doi_phuong)
    NẾU raw < 0:
        RETURN 0
    RETURN raw

FUNCTION apply_damage(mau, sat_thuong):
    SET mau_moi = mau - sat_thuong
    NẾU mau_moi < 0:
        RETURN 0
    RETURN mau_moi

SET mau = 100
SET dmg = tinh_sat_thuong(40, 0.25)   // 30
SET mau = apply_damage(mau, dmg)      // 70
```

## 7. Lỗi thường gặp

1. Quên `RETURN` khi hàm lẽ ra phải trả kết quả.
2. Nhầm thứ tự tham số khi gọi: `tinh_BMI(1.75, 70)` sai ý.
3. Hàm làm quá nhiều việc (nấu phở + rửa bát + tính tiền) — khó bảo trì.
4. Dùng biến “bên ngoài” lung tung thay vì truyền tham số — khó theo dõi.

## 8. Gỡ lỗi / Kiểm tra hiểu biết

```text
FUNCTION tang(x):
    SET x = x + 1
    RETURN x

SET a = 5
SET b = tang(a)
// a=?  b=?
```

Với mô hình truyền **giá trị số**: `a` vẫn 5, `b` = 6. Hàm làm việc trên bản sao tham số (ý tưởng Level 0; chi tiết ref sau này).

## 9. Best practices

- Tên hàm = động từ / cụm hành động rõ.
- Một hàm một nhiệm vụ chính.
- Tham số đầu vào rõ; trả về một kết quả rõ.
- Hàm ngắn đủ để đọc trong một màn hình tư duy.

## 10. Bài tập

**Bài 1.** Viết hàm `binh_phuong(n)` trả về `n * n`. Gọi với 4 và 9.

**Bài 2.** Viết hàm `la_chan(n)` trả về true nếu `n % 2 == 0`.

**Bài 3.** Viết hàm `xep_loai(diem)` trả về `"Dat"` nếu `diem >= 5`, ngược lại `"Truot"`.

**Bài 4.** Viết hàm `tong_tu_1_den(n)` dùng vòng lặp, trả về tổng 1..n.

## 11. Gợi ý

- Bài 2: `RETURN (n % 2 == 0)`.
- Bài 4: giống bài tổng ở Chương 05 nhưng bọc trong hàm.

## 12. Đáp án + Giải thích đáp án

**Bài 1.**

```text
FUNCTION binh_phuong(n):
    RETURN n * n

HIỂN_THỊ binh_phuong(4)   // 16
HIỂN_THỊ binh_phuong(9)   // 81
```

**Bài 2.**

```text
FUNCTION la_chan(n):
    RETURN (n % 2 == 0)
```

**Bài 3.**

```text
FUNCTION xep_loai(diem):
    NẾU diem >= 5:
        RETURN "Dat"
    NGƯỢC_LẠI:
        RETURN "Truot"
```

**Bài 4.**

```text
FUNCTION tong_tu_1_den(n):
    SET tong = 0
    FOR i TỪ 1 ĐẾN n:
        SET tong = tong + i
    RETURN tong
```

## 13. Đáp án thay thế

Bài 4 với công thức `n * (n + 1) / 2` cũng đúng (thuật toán khác, cùng kết quả).

## 14. Thử thách

Tách “máy bán hàng đơn giản” thành ít nhất 3 hàm: `doc_lua_chon`, `tinh_tien(gia, so_luong)`, `in_hoa_don(...)`. Viết pseudocode gọi chúng từ `main`.

## 15. Ứng dụng thực tế

API, thư viện, service ngân hàng… đều là tập hàm/method được gọi lại. Kỹ năng tách hàm là nền của mọi codebase lớn.

## 16. Liên hệ Unity

Trong Unity, mọi thứ bạn viết trong script (`TakeDamage`, `SpawnEnemy`, `UpdateUI`) đều là hàm/method. GameObject gọi method của nhau. Học tách hàm sớm giúp script không thành một khối `Update` khổng lồ khó debug.

## 17. Kiểm tra kiến thức

1. Tham số dùng để làm gì?  
2. `RETURN` nghĩa là gì?  
3. Vì sao nên tách hàm?  
4. Biến cục bộ trong hàm thường thấy được từ bên ngoài không?  
5. Hàm không `RETURN` vẫn hữu ích khi nào?

### Đáp án kiểm tra kiến thức

1. Truyền dữ liệu đầu vào cho hàm.  
2. Trả kết quả về nơi gọi và kết thúc hàm.  
3. Tái sử dụng, rõ nghĩa, dễ sửa/kiểm tra.  
4. Không (ở mức tư duy cơ bản).  
5. Khi chỉ cần thực hiện hành động (in, lưu file, đổi trạng thái).
