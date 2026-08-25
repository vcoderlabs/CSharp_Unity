# Chương 04 — Biểu thức và câu lệnh

## 1. Mục tiêu học

- Phân biệt **biểu thức** (có giá trị) và **câu lệnh** (hành động).
- Viết được biểu thức số học và so sánh cơ bản.
- Hiểu thứ tự ưu tiên toán tử ở mức trực giác.
- Kết hợp biểu thức vào câu lệnh gán và điều kiện.

## 2. Điều kiện tiên quyết

- Chương 03 (biến và dữ liệu).
- Biết các phép `+ - * /` và so sánh `> < >= <=`.

## 3. Khái niệm

### Biểu thức (Expression)

Biểu thức là mảnh code **tính ra một giá trị**.

Ví dụ:

```text
3 + 4              → giá trị 7
mau > 0            → true hoặc false
"Xin" + " chào"    → "Xin chào" (ý tưởng nối chuỗi)
```

Biểu thức **không tự thay đổi thế giới** — nó chỉ ra giá trị (trừ khi nằm trong câu lệnh có tác dụng phụ).

### Câu lệnh (Statement)

Câu lệnh là **hành động hoàn chỉnh** mà chương trình thực hiện:

```text
SET x = 3 + 4          // câu lệnh gán (bên phải là biểu thức)
HIỂN_THỊ x             // câu lệnh xuất
NẾU x > 5 THÌ ...      // câu lệnh điều kiện
```

### So sánh nhanh

| | Biểu thức | Câu lệnh |
|---|-----------|----------|
| Có giá trị? | Có | Thường là hành động (một số ngôn ngữ có biểu thức-câu lệnh) |
| Ví dụ | `a + 1`, `a > b` | `SET a = a + 1`, `IN a` |
| Đời thường | “3 + 4 bằng mấy?” | “Hãy ghi 7 vào sổ” |

### Toán tử thường gặp

- Số học: `+`, `-`, `*`, `/`, `%` (chia lấy dư)
- So sánh: `==`, `!=`, `>`, `<`, `>=`, `<=`
- Logic: `VÀ` (AND), `HOẶC` (OR), `KHÔNG` (NOT)

### Ưu tiên (trực giác)

Giống toán học: nhân chia trước cộng trừ. Dùng ngoặc `()` khi nghi ngờ.

```text
2 + 3 * 4     → 14   (không phải 20)
(2 + 3) * 4   → 20
```

## 4. Mô hình tư duy

```text
Câu lệnh gán:

   SET  đích  =  biểu_thức
         │           │
         │           └─► tính ra GIÁ TRỊ trước
         └─► rồi GHI giá trị đó vào biến đích

Ví dụ: SET mau = mau - sat_thuong
              │
              ├─ đọc mau, đọc sat_thuong
              ├─ tính hiệu
              └─ ghi vào mau
```

Cây biểu thức đơn giản:

```text
        (-)
       /   \
     mau   sat_thuong
```

## 5. Cú pháp / Pseudocode

```text
// Biểu thức số
SET tong = a + b
SET trung_binh = (a + b + c) / 3
SET so_du = n % 2          // 0 nếu chẵn

// Biểu thức so sánh / logic
SET du_dieu_kien = (tuoi >= 18) VÀ (co_ve == true)

// Câu lệnh
HIỂN_THỊ tong
SET x = x + 1
```

## 6. Ví dụ

### Cơ bản

```text
SET a = 10
SET b = 3
SET tong = a + b        // 13
SET tich = a * b        // 30
SET du = a % b          // 1
```

### Trung cấp

Tính sát thương sau giảm:

```text
SET sat_thuong_goc = 50
SET giap = 0.2          // giảm 20%
SET sat_thuong_that = sat_thuong_goc * (1 - giap)   // 40
SET mau = 100
SET mau = mau - sat_thuong_that
```

### Nâng cao

Biểu thức điều kiện phức:

```text
SET co_the_mo_cua = (co_chia_khoa == true) HOẶC (la_admin == true)
SET hop_le = (diem >= 0) VÀ (diem <= 100)

NẾU KHÔNG hop_le:
    HIỂN_THỊ "Điểm không hợp lệ"
```

## 7. Lỗi thường gặp

1. Quên ngoặc: `a + b / 2` khác `(a + b) / 2`.
2. Viết điều kiện đời thường mơ hồ: “nếu máu thấp và không có bình máu hoặc đang boss” — cần ngoặc logic rõ.
3. Chia số nguyên (sau này trong C#) có thể mất phần thập phân — tạm nhớ phép chia có thể cần kiểu số thực.
4. Nhầm `%` với phần trăm: trong lập trình `%` thường là **modulo**, không phải “phần trăm”.

## 8. Gỡ lỗi / Kiểm tra hiểu biết

```text
SET x = 5
SET y = 2
SET z = x + y * 3
```

`z` = ?  
→ `y * 3 = 6`, rồi `5 + 6 = 11`.

Nếu muốn `(x + y) * 3` phải viết ngoặc.

## 9. Best practices

- Biểu thức dài → tách biến trung gian cho dễ đọc.
- Luôn dùng ngoặc khi kết hợp AND/OR.
- Một câu lệnh một ý chính.
- Đặt tên biến trung gian: `sat_thuong_that` thay vì nhồi mọi thứ vào một dòng.

## 10. Bài tập

**Bài 1.** Cho `a = 8`, `b = 3`. Tính (bằng tay / pseudocode) `a + b * 2`, `(a + b) * 2`, `a % b`.

**Bài 2.** Viết biểu thức boolean: người chơi được vào khu vực nếu `cap >= 10` và `co_nhiem_vu == true`.

**Bài 3.** Viết các câu lệnh tính BMI giả lập: `bmi = can_nang / (chieu_cao * chieu_cao)` với số mẫu tự chọn, rồi in kết quả.

## 11. Gợi ý

- Bài 1: nhớ nhân trước cộng; `%` là dư.
- Bài 2: dùng `VÀ`.
- Bài 3: gán biến trước, rồi một biểu thức gán `bmi`.

## 12. Đáp án + Giải thích đáp án

**Bài 1.**  
`a + b * 2` = 8 + 6 = **14**  
`(a + b) * 2` = 11 * 2 = **22**  
`a % b` = 8 % 3 = **2**

**Bài 2.**

```text
SET duoc_vao = (cap >= 10) VÀ (co_nhiem_vu == true)
```

**Bài 3 (mẫu):**

```text
SET can_nang = 70
SET chieu_cao = 1.75
SET bmi = can_nang / (chieu_cao * chieu_cao)
HIỂN_THỊ bmi
```

## 13. Đáp án thay thế

Bài 2 có thể viết `cap >= 10 VÀ co_nhiem_vu` nếu quy ước biến boolean đứng một mình nghĩa là `== true`.

## 14. Thử thách

Viết biểu thức (và vài câu lệnh) tính tổng tiền sau giảm giá: giá gốc, phần trăm giảm, thuế VAT sau giảm. Tách ít nhất 2 biến trung gian.

## 15. Ứng dụng thực tế

Công thức lương, phí ship, điểm tín dụng, rule khuyến mãi… đều là biểu thức + câu lệnh gán/kiểm tra.

## 16. Liên hệ Unity

Trong game, sát thương, tốc độ, cooldown… là biểu thức tính mỗi frame hoặc mỗi sự kiện. Viết biểu thức rõ giúp tránh bug “nhân trước cộng” làm mất cân bằng combat. Sau này trong C# Unity bạn sẽ viết đúng những công thức này trong method.

## 17. Kiểm tra kiến thức

1. `3 * 4` là biểu thức hay câu lệnh?  
2. `SET x = 3 * 4` là gì?  
3. `10 % 4` bằng bao nhiêu?  
4. Vì sao cần ngoặc trong logic AND/OR?  
5. Biểu thức so sánh trả về kiểu gì (ý tưởng)?

### Đáp án kiểm tra kiến thức

1. Biểu thức.  
2. Câu lệnh gán chứa biểu thức.  
3. 2.  
4. Để đúng thứ tự điều kiện, tránh nhập nhằng.  
5. Đúng/sai (boolean).
