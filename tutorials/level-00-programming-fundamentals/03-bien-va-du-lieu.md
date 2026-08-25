# Chương 03 — Biến và dữ liệu

## 1. Mục tiêu học

- Hiểu **biến** là “hộp gắn nhãn” chứa giá trị trong RAM khi chương trình chạy.
- Phân biệt **tên biến**, **kiểu dữ liệu** (ý tưởng), **giá trị**.
- Biết gán giá trị, đọc giá trị, cập nhật giá trị.
- Hình dung dữ liệu số, chữ, đúng/sai được biểu diễn khái niệm trong bộ nhớ.

## 2. Điều kiện tiên quyết

- Chương 01–02 (RAM, chương trình chạy).
- Biết cộng trừ cơ bản.

## 3. Khái niệm

### Biến là gì?

Biến = **tên** bạn đặt cho một ô nhớ để lưu thông tin có thể thay đổi.

Đời thường: hộp trên bàn ghi nhãn `so_tien`, bên trong để tờ 100.000đ.  
Đổi tiền trong hộp = **gán giá trị mới**; nhãn hộp vẫn là `so_tien`.

### Ba thành phần

| Thành phần | Ý nghĩa | Ví dụ |
|------------|---------|-------|
| Tên | Cách gọi ô nhớ | `mau`, `ten_nguoi_choi` |
| Giá trị | Nội dung hiện tại | `100`, `"An"` |
| Kiểu (ý tưởng) | Loại dữ liệu hộp chứa | số nguyên, chữ, đúng/sai |

### Các kiểu đời thường (chưa cần cú pháp C#)

- **Số nguyên:** 0, 1, 42, -3 (máu, điểm, số lượng item)
- **Số thực:** 3.14, 0.5 (tỉ lệ, thời gian lẻ)
- **Chuỗi ký tự:** `"Hello"`, `"Sword"`
- **Logic (boolean):** đúng / sai (`true` / `false`) — cửa mở?, còn sống?

### Gán và đọc

```text
mau = 100        // ghi 100 vào hộp tên mau
in mau           // đọc giá trị trong hộp → in 100
mau = mau - 10   // đọc 100, trừ 10, ghi 90 lại vào mau
```

## 4. Mô hình tư duy

```text
        RAM (đơn giản hóa)
┌──────────────────────────────┐
│  tên: mau      giá trị: 90   │
│  tên: ten      giá trị: "An" │
│  tên: song     giá trị: true │
└──────────────────────────────┘
         ▲
         │ chương trình dùng tên để đọc/ghi
         │
    CPU thực thi: mau = mau - 10
```

Luồng gán:

```text
1. Tính vế phải (biểu thức)
2. Ghi kết quả vào ô nhớ mang tên biến ở vế trái
```

## 5. Cú pháp / Pseudocode

```text
// Khai báo / tạo biến và gán
SET mau = 100
SET ten = "An"
SET dang_song = true

// Đọc
HIỂN_THỊ mau

// Cập nhật
SET mau = mau - 25

// Đặt tên tốt
SET so_luong_vat_pham = 3     // rõ nghĩa
SET x = 3                     // khó hiểu nếu không có ngữ cảnh
```

Quy ước đặt tên (ý tưởng, sau này dùng ở C#):

- Rõ nghĩa: `playerHealth` hoặc `mau_nguoi_choi`
- Tránh tên một chữ trừ khi vòng lặp ngắn (`i`, `j`)

## 6. Ví dụ

### Cơ bản

```text
SET diem = 0
SET diem = diem + 10
SET diem = diem + 5
// diem đang là 15
```

### Trung cấp

```text
SET mau_toi_da = 100
SET mau_hien_tai = mau_toi_da
SET sat_thuong = 30
SET mau_hien_tai = mau_hien_tai - sat_thuong
// mau_hien_tai = 70
SET het_mau = (mau_hien_tai <= 0)   // false
```

### Nâng cao

Theo dõi nhiều trạng thái:

```text
SET vang = 50
SET gia_kiem = 40
SET du_tien = (vang >= gia_kiem)

NẾU du_tien:
    SET vang = vang - gia_kiem
    SET co_kiem = true
NGƯỢC_LẠI:
    SET co_kiem = false
    HIỂN_THỊ "Không đủ vàng"
```

## 7. Lỗi thường gặp

1. Dùng biến **trước khi gán** giá trị ban đầu → giá trị rác / lỗi.
2. Nhầm **so sánh** và **gán** (đời sau: `=` vs `==`).
3. Đặt tên mơ hồ: `a1`, `temp2`, `data`.
4. Nghĩ đổi biến `ban_sao = goc` luôn luôn tách độc lập — với kiểu tham chiếu sau này sẽ khác (Level 3). Ở Level 0: với số đơn giản thì sao chép giá trị.

## 8. Gỡ lỗi / Kiểm tra hiểu biết

```text
SET x = 5
SET y = x
SET x = 9
// y bằng bao nhiêu?
```

**Trả lời với số:** `y` vẫn là `5`, vì đã sao chép giá trị tại thời điểm gán. Đổi `x` sau đó không tự đổi `y`.

## 9. Best practices

- Khởi tạo biến trước khi dùng.
- Tên biến = tiếng nói được ý nghĩa trong bài toán.
- Một biến một trách nhiệm: đừng tái sử dụng `temp` cho năm việc khác nhau nếu gây rối.
- Phân biệt rõ hằng số ý niệm (`mau_toi_da`) và giá trị thay đổi (`mau_hien_tai`).

## 10. Bài tập

**Bài 1.** Viết pseudocode: bắt đầu `mau = 80`, bị đánh 3 lần mỗi lần mất 15 máu. In máu còn lại.

**Bài 2.** Có `gia = 120`, `tien = 100`. Tạo biến boolean `mua_duoc` thể hiện đủ tiền hay không.

**Bài 3.** Mô tả bằng lời: chuyện gì xảy ra trong RAM khi chạy `SET diem = diem + 1`.

## 11. Gợi ý

- Bài 1: trừ ba lần hoặc dùng vòng sau; ở đây trừ tuần tự cũng được.
- Bài 2: so sánh `tien >= gia`.
- Bài 3: đọc → cộng → ghi lại.

## 12. Đáp án + Giải thích đáp án

**Bài 1.**

```text
SET mau = 80
SET mau = mau - 15
SET mau = mau - 15
SET mau = mau - 15
HIỂN_THỊ mau    // 35
```

**Bài 2.**

```text
SET gia = 120
SET tien = 100
SET mua_duoc = (tien >= gia)   // false
```

**Bài 3.** CPU đọc giá trị hiện tại của `diem` từ RAM, cộng 1, ghi kết quả mới vào cùng ô nhớ tên `diem`.

## 13. Đáp án thay thế

Bài 1 có thể dùng vòng lặp `LẶP 3 LẦN: mau = mau - 15` (xem Chương 05).

## 14. Thử thách

Thiết kế biến cần thiết cho “nhân vật RPG đơn giản”: máu, mana, cấp, tên, còn sống. Gán giá trị mẫu và mô tả khi nào mỗi biến đổi.

## 15. Ứng dụng thực tế

Form đăng ký, giỏ hàng, điểm tín dụng… đều là tập biến (họ tên, số lượng, số dư) được đọc/ghi theo nghiệp vụ.

## 16. Liên hệ Unity

Trong Unity, máu player, tốc độ chạy, số đạn… thường là biến (field/property) trên script. Mỗi frame có thể đọc/ghi chúng. Hiểu “biến = trạng thái trong RAM” giúp bạn thiết kế trạng thái nhân vật trước khi học cú pháp C#.

## 17. Kiểm tra kiến thức

1. Biến sống chủ yếu ở đâu khi chương trình chạy?  
2. `ten = "An"` — đâu là tên, đâu là giá trị?  
3. Boolean dùng để làm gì?  
4. Vì sao nên đặt tên rõ?  
5. `a = b` với số nguyên thường nghĩa là gì?

### Đáp án kiểm tra kiến thức

1. RAM.  
2. Tên: `ten`; giá trị: `"An"`.  
3. Lưu đúng/sai — phục vụ quyết định.  
4. Để người đọc (và bạn sau này) hiểu ý nghĩa dữ liệu.  
5. Sao chép giá trị từ `b` sang `a`.
