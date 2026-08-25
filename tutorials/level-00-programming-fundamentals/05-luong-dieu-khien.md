# Chương 05 — Luồng điều khiển

## 1. Mục tiêu học

- Hiểu chương trình không chỉ chạy thẳng từ trên xuống: có **rẽ nhánh** và **lặp**.
- Viết được `NẾU / NGƯỢC_LẠI` (if/else) bằng pseudocode.
- Viết được vòng lặp `WHILE` và `FOR` (ý tưởng đếm).
- Biết khi nào dùng nhánh, khi nào dùng lặp; tránh vòng lặp vô hạn ở mức tư duy.

## 2. Điều kiện tiên quyết

- Chương 03–04 (biến, biểu thức boolean).
- Hiểu biểu thức đúng/sai.

## 3. Khái niệm

### Luồng điều khiển là gì?

Là cách chương trình quyết định **làm bước nào tiếp theo**: đi thẳng, rẽ trái/phải, hoặc lặp lại.

Đời thường:

- **If/else:** Nếu trời mưa thì mang ô, không thì không mang.
- **Loop:** Trong khi còn bát bẩn thì rửa tiếp. Hoặc: lặp 10 lần chống đẩy.

### Rẽ nhánh (if / else)

```text
NẾU điều_kiện:
    làm A
NGƯỢC_LẠI:
    làm B
```

Có thể có nhiều nhánh: `NẾU / NGƯỢC_LẠI_NẾU / NGƯỢC_LẠI`.

### Vòng lặp while

Lặp **khi điều kiện còn đúng**:

```text
WHILE còn_mau > 0:
    chơi_tiếp()
```

Cần có thứ gì đó trong vòng làm điều kiện **có ngày thành sai**, nếu không sẽ lặp mãi.

### Vòng lặp for (đếm)

Lặp một số lần biết trước:

```text
FOR i TỪ 1 ĐẾN 5:
    HIỂN_THỊ i
```

### break / continue (ý tưởng)

- `BREAK`: thoát khỏi vòng ngay.
- `CONTINUE`: bỏ phần còn lại của vòng hiện tại, sang lần lặp tiếp.

## 4. Mô hình tư duy

```text
If/else:

          [ điều kiện? ]
           /         \
        true         false
         |             |
        làm A         làm B
         \             /
          \           /
           [ tiếp tục ]


While:

     ┌──► [ điều kiện? ] --false--> thoát
     │           |
     │         true
     │           |
     │      [ thân vòng ]
     │           |
     └───────────┘
```

## 5. Cú pháp / Pseudocode

```text
// If / else if / else
NẾU diem >= 90:
    loai = "A"
NGƯỢC_LẠI_NẾU diem >= 70:
    loai = "B"
NGƯỢC_LẠI:
    loai = "C"

// While
SET n = 3
WHILE n > 0:
    HIỂN_THỊ n
    SET n = n - 1

// For
FOR i TỪ 0 ĐẾN 4:          // 5 lần: 0,1,2,3,4
    HIỂN_THỊ "Lần", i

// Break
WHILE true:
    SET lenh = ĐỌC_INPUT()
    NẾU lenh == "quit":
        BREAK
    XỬ_LÝ(lenh)
```

## 6. Ví dụ

### Cơ bản

```text
SET mau = 0
NẾU mau <= 0:
    HIỂN_THỊ "Game Over"
NGƯỢC_LẠI:
    HIỂN_THỊ "Tiếp tục chơi"
```

### Trung cấp

Đếm số chẵn từ 1 đến 10:

```text
SET dem = 0
FOR i TỪ 1 ĐẾN 10:
    NẾU i % 2 == 0:
        SET dem = dem + 1
HIỂN_THỊ dem    // 5
```

### Nâng cao

Menu giả lập:

```text
SET dang_chay = true
WHILE dang_chay:
    HIỂN_THỊ "1. Chơi  2. Thoát"
    SET chon = ĐỌC_INPUT()
    NẾU chon == "1":
        HIỂN_THỊ "Bắt đầu..."
    NGƯỢC_LẠI_NẾU chon == "2":
        SET dang_chay = false
    NGƯỢC_LẠI:
        HIỂN_THỊ "Lựa chọn sai"
HIỂN_THỊ "Tạm biệt"
```

## 7. Lỗi thường gặp

1. Vòng `WHILE` quên cập nhật biến điều kiện → **lặp vô hạn**.
2. Nhầm `NẾU` lồng nhau: else “dính” nhầm nhánh.
3. Dùng nhiều if riêng thay vì else-if khi các nhánh loại trừ nhau → có thể chạy nhầm nhiều nhánh.
4. Off-by-one: `TỪ 1 ĐẾN 10` có 10 lần; `TỪ 0 ĐẾN 9` cũng 10 lần — đếm kỹ.

## 8. Gỡ lỗi / Kiểm tra hiểu biết

```text
SET i = 0
WHILE i < 3:
    HIỂN_THỊ i
    // quên i = i + 1
```

Chuyện gì xảy ra? → In `0` mãi (vòng vô hạn). Sửa: thêm `SET i = i + 1` trong vòng.

## 9. Best practices

- Điều kiện vòng lặp phải **tiến triển** hướng tới kết thúc.
- Ưu tiên `else-if` khi chỉ một nhánh được chọn.
- Vòng biết trước số lần → `FOR`; chờ điều kiện → `WHILE`.
- Tránh lồng quá sâu: tách hàm (Chương 06) khi logic phức tạp.

## 10. Bài tập

**Bài 1.** Viết if/else: nếu `tuoi >= 18` in “Đủ tuổi”, ngược lại “Chưa đủ tuổi”.

**Bài 2.** Dùng while: bắt đầu `x = 1`, nhân đôi `x` cho đến khi `x >= 100`, in các giá trị.

**Bài 3.** Dùng for: tính tổng các số từ 1 đến 20.

**Bài 4.** Viết logic: đoán số — lặp đến khi đoán đúng `bi_mat = 7` (đọc input mỗi vòng).

## 11. Gợi ý

- Bài 2: điều kiện `WHILE x < 100`, trong vòng in rồi `x = x * 2`.
- Bài 3: `SET tong = 0`, cộng dồn trong for.
- Bài 4: while đoán sai thì đọc tiếp.

## 12. Đáp án + Giải thích đáp án

**Bài 1.**

```text
NẾU tuoi >= 18:
    HIỂN_THỊ "Đủ tuổi"
NGƯỢC_LẠI:
    HIỂN_THỊ "Chưa đủ tuổi"
```

**Bài 2.**

```text
SET x = 1
WHILE x < 100:
    HIỂN_THỊ x
    SET x = x * 2
// in: 1, 2, 4, 8, 16, 32, 64 (sau đó x=128 dừng trước khi in)
```

**Bài 3.**

```text
SET tong = 0
FOR i TỪ 1 ĐẾN 20:
    SET tong = tong + i
HIỂN_THỊ tong    // 210
```

**Bài 4.**

```text
SET bi_mat = 7
SET doan = ĐỌC_INPUT()
WHILE doan != bi_mat:
    HIỂN_THỊ "Sai, đoán lại"
    SET doan = ĐỌC_INPUT()
HIỂN_THỊ "Đúng!"
```

## 13. Đáp án thay thế

Bài 2 có thể `WHILE true` + `BREAK` khi `x >= 100`.  
Bài 4 có thể dùng `WHILE true` và break khi đúng.

## 14. Thử thách

Viết pseudocode FizzBuzz 1..30: chia hết 15 in “FizzBuzz”, hết 3 in “Fizz”, hết 5 in “Buzz”, còn lại in số. Chú ý thứ tự if.

## 15. Ứng dụng thực tế

ATM (menu lặp), đăng nhập (lặp đến khi đúng hoặc hết lần), chấm điểm (nhánh xếp loại), xử lý đơn hàng theo trạng thái — đều là control flow.

## 16. Liên hệ Unity

Gameplay đầy rẫy nhánh và vòng: nếu hết máu thì Die; while còn enemy trong wave thì spawn; mỗi frame `Update` là vòng lặp game do engine chạy. Tư duy if/loop vững giúp bạn viết AI đơn giản, UI menu, và điều kiện thắng/thua rõ ràng.

## 17. Kiểm tra kiến thức

1. If/else dùng khi nào?  
2. While khác for chủ yếu ở điểm nào?  
3. Vòng vô hạn thường do đâu?  
4. `BREAK` làm gì?  
5. `1 ĐẾN 5` lặp mấy lần?

### Đáp án kiểm tra kiến thức

1. Khi cần chọn hành động theo điều kiện.  
2. For thường biết số lần; while theo điều kiện còn đúng.  
3. Điều kiện không bao giờ thành sai / quên cập nhật biến.  
4. Thoát khỏi vòng lặp hiện tại.  
5. 5 lần.
