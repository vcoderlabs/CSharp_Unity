# Chương 08 — Debug cơ bản

## 1. Mục tiêu học

- Phân loại lỗi: **cú pháp / biên dịch**, **runtime**, **logic**.
- Biết quy trình gỡ lỗi: tái hiện → giả thuyết → kiểm chứng → sửa → kiểm tra lại.
- Hiểu ý tưởng **breakpoint**, **in dấu vết** (print debugging), đọc thông báo lỗi.
- Rèn thói quen kiểm tra giả định bằng dữ liệu nhỏ.

## 2. Điều kiện tiên quyết

- Chương 02 (biên dịch vs chạy) và 05–07 (logic có thể sai).
- Đã từng viết pseudocode có if/loop/hàm.

## 3. Khái niệm

### Ba họ lỗi

| Loại | Khi nào lộ | Ví dụ |
|------|------------|-------|
| Cú pháp / biên dịch | Trước khi chạy | Thiếu cấu trúc, sai chính tả từ khóa |
| Runtime | Đang chạy | Chia 0, chỉ số mảng ngoài phạm vi |
| Logic | Chạy xong nhưng sai ý | Công thức sai, điều kiện ngược |

### Debug là gì?

Debug = **săn bug** một cách có hệ thống, không phải sửa lung tung.

Đời thường: đèn nhà không sáng → kiểm tra bóng → cầu dao → dây… từng giả thuyết.

### Print debugging

Chèn lệnh in để xem giá trị biến tại các điểm:

```text
HIỂN_THỊ "DEBUG mau=", mau, "dmg=", dmg
```

### Breakpoint (ý tưởng)

Bạn đánh dấu một dòng: “khi chạy đến đây thì **tạm dừng**”, rồi quan sát biến, rồi bước từng dòng (step over / step into). IDE hiện đại hỗ trợ việc này — bạn sẽ dùng nhiều ở Level 1+.

### Đọc lỗi

Thông báo lỗi thường có: **loại lỗi**, **mô tả**, **vị trí (file/dòng)**. Đọc từ trên xuống; sửa nguyên nhân gốc trước.

## 4. Mô hình tư duy

```text
1. TÁI HIỆN     → làm lại đúng bước để lỗi xuất hiện
2. THU HẸP      → lỗi ở đoạn nào? (chia đôi / in dấu vết)
3. GIẢ THUYẾT   → “mình nghĩ biến X sai vì…”
4. KIỂM CHỨNG   → in X / breakpoint / test nhỏ
5. SỬA          → một thay đổi có chủ đích
6. XÁC NHẬN     → chạy lại case lỗi + vài case liên quan
```

```text
     [Mong đợi] ----so sánh---- [Thực tế]
                      │
                      ▼
              lệch ở bước nào?
```

## 5. Cú pháp / Pseudocode

Kỹ thuật chèn dấu vết:

```text
FUNCTION tinh_tong(danh_sach):
    SET tong = 0
    FOR MỖI x TRONG danh_sach:
        HIỂN_THỊ "DEBUG x=", x, "tong_truoc=", tong
        SET tong = tong + x
    HIỂN_THỊ "DEBUG tong_cuoi=", tong
    RETURN tong
```

Checklist đọc lỗi:

```text
1. Copy nguyên thông báo
2. Tìm dòng số / tên hàm
3. Hiểu “cái gì failed”, không chỉ “failed”
4. Đừng sửa 10 chỗ cùng lúc
```

## 6. Ví dụ

### Cơ bản

```text
SET a = 10
SET b = 0
HIỂN_THỊ a / b    // runtime: chia 0
```

Sửa: kiểm tra `b != 0` trước khi chia.

### Trung cấp — bug logic

```text
// Muốn: cộng điểm nếu trả lời đúng
NẾU cau_tra_loi != dap_an:     // điều kiện NGƯỢC
    SET diem = diem + 1
```

Hiện tượng: trả lời sai lại được điểm.  
Debug: in `cau_tra_loi`, `dap_an`, điều kiện boolean.

### Nâng cao — thu hẹp vòng lặp

Tổng 1..5 ra 10 thay vì 15:

```text
SET tong = 0
FOR i TỪ 1 ĐẾN 5:
    SET tong = tong + i
    HIỂN_THỊ i, tong
// Quan sát: nếu i chạy 1..4 → off-by-one ở biên vòng lặp
```

## 7. Lỗi thường gặp khi debug

1. Sửa nhiều chỗ cùng lúc → không biết gì đã “fix”.
2. Không tái hiện ổn định → đoán mò.
3. Xóa hết print/debug lung tung trước khi hiểu root cause.
4. Tin rằng “biên dịch sạch = không bug”.

## 8. Gỡ lỗi / Kiểm tra hiểu biết

Đoạn sau in toàn số âm trong khi dữ liệu toàn dương — bạn làm gì trước?

```text
FOR MỖI x TRONG ds:
    NẾU x < 0:
        HIỂN_THỊ x
```

**Trả lời:** Có thể điều kiện đúng nghĩa “chỉ in âm” — nếu yêu cầu là in dương thì điều kiện ngược. In thử từng `x` trước khi if để xác nhận dữ liệu vào.

## 9. Best practices

- Một bug một giả thuyết.
- Dùng **test case nhỏ** (2–3 phần tử) trước dữ liệu lớn.
- Giữ ghi chú: “đã thử gì, kết quả gì”.
- Sau khi sửa: chạy lại case lỗi + case biên (0, rỗng, max).

## 10. Bài tập

**Bài 1.** Phân loại lỗi: (a) thiếu `RETURN` làm hàm luôn rỗng ở ngôn ngữ yêu cầu return (b) công thức diện tích nhầm `dai + rong` (c) truy cập phần tử thứ 100 của mảng dài 3.

**Bài 2.** Đoạn while sau lỗi gì? Viết cách sửa và cách bạn phát hiện bằng print.

```text
SET i = 10
WHILE i > 0:
    HIỂN_THỊ i
    SET i = i + 1
```

**Bài 3.** Mô tả 4 bước debug cho bug: “hàm xếp loại luôn trả Trượt dù điểm 10”.

## 11. Gợi ý

- Bài 2: điều kiện và cập nhật đi ngược chiều mong muốn.
- Bài 3: in tham số đầu vào và nhánh if được chọn.

## 12. Đáp án + Giải thích đáp án

**Bài 1.**  
(a) Logic / thiết kế hàm (có thể kèm cảnh báo biên dịch tùy ngôn ngữ) — thiên **logic** hoặc lỗi dùng hàm. Với Level 0: coi là **logic / sử dụng hàm sai**.  
(b) **Logic**.  
(c) **Runtime**.

**Bài 2.** `i` tăng mãi trong khi điều kiện `i > 0` luôn đúng → vòng vô hạn. Sửa: `SET i = i - 1`. Phát hiện: in `i` thấy 10, 11, 12…

**Bài 3 (mẫu):**  
1) Gọi hàm với `diem = 10`, tái hiện.  
2) In `diem` ngay đầu hàm.  
3) In kết quả từng điều kiện `diem >= 5`.  
4) Phát hiện so sánh sai (ví dụ `diem > 10` hoặc nhầm biến) → sửa → gọi lại với 10, 5, 4.

## 13. Đáp án thay thế

Bài 1(a) có thể xếp “biên dịch” nếu ngôn ngữ bắt buộc return và compiler bắt được — miễn bạn giải thích nhất quán.

## 14. Thử thách

Tự viết một đoạn pseudocode **cố ý sai**, đưa bạn học cùng (hoặc tự sau 10 phút), rồi debug theo quy trình 6 bước. Ghi nhật ký debug ngắn.

## 15. Ứng dụng thực tế

Kỹ sư hỗ trợ production: đọc log, tái hiện ticket, bisect thay đổi — cùng một tư duy debug. Càng hệ thống, càng ít “sửa cầu may”.

## 16. Liên hệ Unity

Console của Unity đầy exception (`NullReferenceException`, …). Biết đọc stack trace + dùng breakpoint trong IDE gắn Unity là kỹ năng sống còn. Print (`Debug.Log`) là bạn thân khi prototype — nhưng nhớ gỡ log thừa trước build.

## 17. Kiểm tra kiến thức

1. Lỗi logic khác runtime thế nào?  
2. Breakpoint dùng để làm gì?  
3. Vì sao cần tái hiện lỗi?  
4. Print debugging là gì?  
5. Nên sửa bao nhiêu thứ cùng lúc khi debug?

### Đáp án kiểm tra kiến thức

1. Runtime làm chương trình dừng/ném lỗi; logic vẫn chạy nhưng sai kết quả.  
2. Tạm dừng tại dòng để quan sát trạng thái.  
3. Để kiểm chứng giả thuyết và xác nhận đã sửa xong.  
4. In giá trị trung gian để lần theo luồng.  
5. Lý tưởng: một thay đổi có chủ đích mỗi lần.
