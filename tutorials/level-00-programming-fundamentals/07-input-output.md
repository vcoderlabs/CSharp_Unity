# Chương 07 — Input / Output cơ bản

## 1. Mục tiêu học

- Hiểu **Input** (đầu vào) và **Output** (đầu ra) là cầu nối chương trình ↔ thế giới ngoài.
- Phân biệt các nguồn I/O phổ biến: bàn phím, màn hình, file (ý tưởng).
- Viết pseudocode đọc dữ liệu, kiểm tra hợp lệ, rồi xử lý và in kết quả.
- Nhận ra lỗi thường gặp khi đọc số / chuỗi.

## 2. Điều kiện tiên quyết

- Chương 03–06 (biến, điều kiện, hàm).
- Biết if để kiểm tra dữ liệu.

## 3. Khái niệm

### Input

Dữ liệu **đi vào** chương trình: phím bấm, chuột, file cấu hình, tín hiệu mạng, cảm biến…

Đời thường: bạn hỏi quán “muốn món gì?” → câu trả lời là input.

### Output

Dữ liệu **đi ra**: chữ trên console, hình trên màn hình, âm thanh, file log, gói tin gửi đi…

Đời thường: quán đưa tô phở + hóa đơn.

### Console I/O (mức Level 0)

```text
HIỂN_THỊ "Nhập tuổi:"     // output nhắc người dùng
SET tuoi = ĐỌC_INPUT()    // input — thường là chuỗi ký tự
```

Lưu ý quan trọng: dữ liệu đọc từ bàn phím **thường là chữ**, dù bạn gõ `18`. Cần bước **chuyển thành số** nếu muốn tính toán (ý tưởng `PARSE_INT`).

### File I/O (ý tưởng)

```text
SET noi_dung = ĐỌC_FILE("save.txt")
GHI_FILE("save.txt", noi_dung_moi)
```

Liên quan Chương 01: đọc/ghi **storage**.

## 4. Mô hình tư duy

```text
  Người dùng / File / Mạng
           │
           ▼
     ┌──────────┐
     │  INPUT   │  → biến trong RAM
     └──────────┘
           │
           ▼
     ┌──────────┐
     │ XỬ LÝ    │  (if, loop, hàm…)
     └──────────┘
           │
           ▼
     ┌──────────┐
     │  OUTPUT  │  → màn hình / file / mạng
     └──────────┘
```

Luồng chương trình tương tác điển hình:

```text
1. In hướng dẫn (output)
2. Đọc dữ liệu (input)
3. Kiểm tra hợp lệ
4. Xử lý
5. In kết quả (output)
6. (lặp lại nếu cần)
```

## 5. Cú pháp / Pseudocode

```text
FUNCTION doc_so_nguyen(thong_bao):
    HIỂN_THỊ thong_bao
    SET chuoi = ĐỌC_INPUT()
    SET so = PARSE_INT(chuoi)    // ý tưởng chuyển "12" → 12
    RETURN so

SET ten = ĐỌC_INPUT()
HIỂN_THỊ "Xin chào", ten

// Kiểm tra hợp lệ
SET n = doc_so_nguyen("Nhập n > 0:")
WHILE n <= 0:
    HIỂN_THỊ "Không hợp lệ"
    SET n = doc_so_nguyen("Nhập lại:")
```

## 6. Ví dụ

### Cơ bản

```text
HIỂN_THỊ "Nhập tên:"
SET ten = ĐỌC_INPUT()
HIỂN_THỊ "Chào", ten
```

### Trung cấp

Máy tính cộng hai số:

```text
SET a = PARSE_INT(ĐỌC_INPUT())
SET b = PARSE_INT(ĐỌC_INPUT())
HIỂN_THỊ "Tổng =", a + b
```

### Nâng cao

Đăng nhập giả lập tối đa 3 lần:

```text
SET MAT_KHAU = "secret"
SET so_lan = 0
SET ok = false

WHILE so_lan < 3 VÀ ok == false:
    HIỂN_THỊ "Nhập mật khẩu:"
    SET thu = ĐỌC_INPUT()
    NẾU thu == MAT_KHAU:
        SET ok = true
    NGƯỢC_LẠI:
        SET so_lan = so_lan + 1
        HIỂN_THỊ "Sai,", 3 - so_lan, "lần còn lại"

NẾU ok:
    HIỂN_THỊ "Đăng nhập thành công"
NGƯỢC_LẠI:
    HIỂN_THỊ "Bị khóa"
```

## 7. Lỗi thường gặp

1. Cộng chuỗi `"1" + "2"` ra `"12"` thay vì số `3` — quên chuyển kiểu.
2. Không kiểm tra input rỗng / không phải số → lỗi runtime.
3. In thiếu hướng dẫn — người dùng không biết phải nhập gì.
4. Lưu file quên đường dẫn / ghi đè mất dữ liệu (ý tưởng cẩn thận).

## 8. Gỡ lỗi / Kiểm tra hiểu biết

Người dùng gõ `ba` khi bạn mong số nguyên.  
**Cách nghĩ:** `PARSE_INT` thất bại → cần báo lỗi và hỏi lại, đừng giả sử luôn thành công.

## 9. Best practices

- Luôn **prompt rõ** trước khi đọc.
- **Validate** input (phạm vi, định dạng).
- Tách hàm `doc_so_hop_le()` để tái sử dụng.
- Phân biệt thông báo cho người dùng và log kỹ thuật (sau này).

## 10. Bài tập

**Bài 1.** Pseudocode: hỏi tên và năm sinh, in tuổi ước lượng `2026 - nam_sinh`.

**Bài 2.** Đọc một số `n`, in `"Chẵn"` hoặc `"Lẻ"`. Nếu không parse được — mô tả hướng xử lý.

**Bài 3.** Đọc số nguyên dương bằng vòng lặp đến khi hợp lệ, rồi in giai bậc từ 1 đến số đó.

## 11. Gợi ý

- Bài 1: nhớ PARSE_INT cho năm.
- Bài 3: while số <= 0 thì nhập lại; sau đó for in.

## 12. Đáp án + Giải thích đáp án

**Bài 1.**

```text
HIỂN_THỊ "Tên:"
SET ten = ĐỌC_INPUT()
HIỂN_THỊ "Năm sinh:"
SET nam = PARSE_INT(ĐỌC_INPUT())
SET tuoi = 2026 - nam
HIỂN_THỊ ten, "khoảng", tuoi, "tuổi"
```

**Bài 2.**

```text
SET chuoi = ĐỌC_INPUT()
NẾU KHÔNG PHẢI_SỐ(chuoi):
    HIỂN_THỊ "Vui lòng nhập số"
NGƯỢC_LẠI:
    SET n = PARSE_INT(chuoi)
    NẾU n % 2 == 0:
        HIỂN_THỊ "Chẵn"
    NGƯỢC_LẠI:
        HIỂN_THỊ "Lẻ"
```

**Bài 3.**

```text
SET n = PARSE_INT(ĐỌC_INPUT())
WHILE n <= 0:
    HIỂN_THỊ "Nhập lại số > 0"
    SET n = PARSE_INT(ĐỌC_INPUT())
FOR i TỪ 1 ĐẾN n:
    HIỂN_THỊ i
```

## 13. Đáp án thay thế

Bài 3 có thể gói việc đọc số dương vào hàm `doc_so_duong()`.

## 14. Thử thách

Thiết kế “máy tính 4 phép” trên console: đọc `a`, phép (`+ - * /`), `b`; xử lý chia 0; cho phép tính tiếp đến khi nhập `q`.

## 15. Ứng dụng thực tế

Mọi app đều là I/O: form web, API request/response, cảm biến IoT, log server. Chương trình “không I/O” gần như vô dụng với người dùng.

## 16. Liên hệ Unity

Trong Unity, input là bàn phím/chuột/gamepad/`Input System`; output là hình ảnh, UI Text, âm thanh, haptic. Logic gameplay nằm giữa: đọc input → đổi biến → cập nhật hiển thị. Nắm I/O giúp bạn không nhầm “UI hiển thị” với “dữ liệu thật trong biến”.

## 17. Kiểm tra kiến thức

1. Input và output khác nhau thế nào?  
2. Dữ liệu từ bàn phím thường ở dạng gì trước khi parse?  
3. Vì sao cần validate?  
4. Ghi file liên quan thành phần máy nào nhất?  
5. Prompt nghĩa là gì?

### Đáp án kiểm tra kiến thức

1. Input vào chương trình; output ra ngoài.  
2. Chuỗi ký tự.  
3. Tránh dữ liệu sai làm chương trình lỗi hoặc sai logic.  
4. Storage.  
5. Thông báo hướng dẫn trước khi nhận input.
