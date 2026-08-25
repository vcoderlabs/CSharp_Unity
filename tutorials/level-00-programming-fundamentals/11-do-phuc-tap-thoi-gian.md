# Chương 11 — Độ phức tạp thời gian (Big O trực giác)

## 1. Mục tiêu học

- Hiểu Big O là cách nói **độ lớn thời gian/bộ nhớ tăng thế nào khi dữ liệu lớn dần**, không phải đo mili-giây chính xác.
- Nhận diện trực giác: O(1), O(n), O(n²), O(log n).
- So sánh linear search vs binary search bằng ngôn ngữ Big O.
- Biết “đủ tốt” quan trọng hơn “tối ưu sớm” với dữ liệu nhỏ.

## 2. Điều kiện tiên quyết

- Chương 09–10 (thuật toán & cấu trúc dữ liệu).
- Biết vòng lặp lồng nhau là gì.

## 3. Khái niệm

### Bài toán đo hiệu năng tư duy

Thuật toán A xử lý 100 phần tử mất “một ít thời gian”.  
Khi 1.000.000 phần tử — A có còn dùng được không?

Big O giúp trả lời: khi **n** (kích thước dữ liệu) tăng, số bước tăng **tuyến tính**, **bình phương**, hay gần như không đổi?

### Các bậc thường gặp (trực giác)

| Ký hiệu | Ý nghĩa đời thường | Ví dụ |
|---------|-------------------|--------|
| **O(1)** | Làm vài bước, không phụ thuộc n | Lấy `a[0]`, cộng hai số |
| **O(log n)** | Mỗi bước bỏ đi một nửa | Binary search |
| **O(n)** | Chạm mỗi phần tử một lần | Linear search, duyệt list |
| **O(n²)** | Với mỗi phần tử, lại duyệt gần cả list | Vòng lồng đơn giản, bubble/selection sort |

### Không phải phép đo đồng hồ

Big O **bỏ hằng số** và chi tiết máy: nói về **xu hướng**.  
O(2n) và O(n) cùng họ tuyến tính trong cách nói nhập môn.

### Log n là gì (đời thường)?

Số lần chia đôi n đến khi còn ~1.  
n = 1000 → khoảng 10 lần chia đôi. Vì thế binary search “cay” hơn linear rất nhiều khi n lớn.

## 4. Mô hình tư duy

```text
n tăng gấp đôi:

O(1)     ■ ■                 (gần như không đổi)
O(log n) ■ ■■                (tăng rất chậm)
O(n)     ■■■■ → ■■■■■■■■     (gấp đôi)
O(n²)    ■■■■ → ■■■■■■■■...  (gấp ~4 lần)


Vòng đơn:
FOR i TỪ 1 ĐẾN n:        → O(n)
    làm_việc_O(1)

Vòng lồng:
FOR i TỪ 1 ĐẾN n:        → O(n²)
    FOR j TỪ 1 ĐẾN n:
        làm_việc_O(1)
```

## 5. Cú pháp / Pseudocode

Gán nhãn độ phức tạp khi đọc code:

```text
FUNCTION tim_tuan_tu(ds, x):          // O(n)
    FOR MỖI p TRONG ds:               // ~n lần
        NẾU p == x: RETURN true
    RETURN false

FUNCTION lay_dau(ds):                 // O(1) giả sử truy cập đầu trực tiếp
    RETURN ds[0]

FUNCTION co_cap_tong_bang_s(ds, s):   // O(n²) cách ngây thơ
    FOR i TỪ 0 ĐẾN n - 1:
        FOR j TỪ i + 1 ĐẾN n - 1:
            NẾU ds[i] + ds[j] == s:
                RETURN true
    RETURN false
```

## 6. Ví dụ

### Cơ bản

In mọi phần tử list → **O(n)**.

### Trung cấp

```text
FOR i TỪ 1 ĐẾN n:
    HIỂN_THỊ i
FOR j TỪ 1 ĐẾN n:
    HIỂN_THỊ j
```

Hai vòng **nối tiếp** → O(n) + O(n) = **O(n)** (vẫn tuyến tính).

### Nâng cao

Selection sort: vòng ngoài ~n, vòng trong ~n → **O(n²)**.  
Binary search: **O(log n)** nhưng cần dữ liệu đã sort (sort có thể tốn O(n log n) hoặc O(n²) tùy thuật toán — tạm nhớ: “có giá phải trả để được tìm nhanh”).

## 7. Lỗi thường gặp

1. Nghĩ O(n²) luôn “không dùng được” — với n = 50 vẫn ổn.
2. Tối ưu vi mô khi n nhỏ thay vì viết code đúng và rõ.
3. Đếm nhầm vòng nối tiếp thành nhân (cộng vs nhân độ phức tạp).
4. Quên chi phí ẩn: đọc file, network thường đắt hơn vòng lặp nhỏ trong RAM.

## 8. Gỡ lỗi / Kiểm tra hiểu biết

Game bị giật khi có 5000 enemy, mỗi frame mỗi enemy duyệt lại toàn bộ 5000 để tìm mục tiêu.  
→ ~5000² phép/frame → tư duy O(n²) theo số enemy. Hướng xử lý sau này: giảm tần suất, spatial partition, cấu trúc tìm gần hơn…

## 9. Best practices

- Đoán Big O bằng **đếm vòng lặp theo n**.
- Ưu tiên thuật toán đúng + đủ nhanh với quy mô thật.
- Khi n có thể lớn (online game, MMORPG), nghĩ Big O sớm hơn.
- Ghi chú giả định: “n = số enemy trong scene”.

## 10. Bài tập

**Bài 1.** Gán Big O: (a) lấy phần tử giữa mảng bằng chỉ số (b) linear search (c) hai vòng for lồng nhau 1..n.

**Bài 2.** n = 10 vs n = 1000: giải thích vì sao O(n²) đau hơn rõ ở n = 1000.

**Bài 3.** Code sau phức tạp bao nhiêu theo n = length(ds)?

```text
SET dem = 0
FOR MỖI x TRONG ds:
    FOR MỖI y TRONG ds:
        NẾU x < y: SET dem = dem + 1
```

**Bài 4.** Vì sao binary search O(log n) vẫn có thể chậm hơn linear O(n) khi n = 5? (trực giác hằng số / đơn giản triển khai)

## 11. Gợi ý

- Bài 1: O(1), O(n), O(n²).
- Bài 3: vòng lồng trên cùng ds → n * n.
- Bài 4: overhead, hằng số, dữ liệu nhỏ.

## 12. Đáp án + Giải thích đáp án

**Bài 1.** (a) O(1) (b) O(n) (c) O(n²)

**Bài 2.** n² với 10 ≈ 100 bước; với 1000 ≈ 1.000.000 bước — tăng gấp 100 khi n tăng 100, rất khác cảm giác “chậm dần”.

**Bài 3.** **O(n²)** vì mỗi phần tử so với mọi phần tử.

**Bài 4.** Với n nhỏ, chi phí tổ chức binary (tính mid, điều kiện) + code phức tạp có thể không thắng được vài phép so sánh tuyến tính; Big O nói về **xu hướng khi n lớn**.

## 13. Đáp án thay thế

Bài 3 có thể nói “khoảng n²/2 so sánh” nhưng vẫn thuộc lớp O(n²).

## 14. Thử thách

Ước lượng: thuật toán O(n²) với n = 10.000, giả sử 1 triệu phép/giây — khoảng bao lâu? (Ước: 10.000² = 100 triệu phép → ~100 giây.) Viết lại nếu có thuật toán O(n) cho cùng nhu cầu — ý tưởng thôi.

## 15. Ứng dụng thực tế

Tìm kiếm web, đề xuất video, xử lý log hàng triệu dòng — đều chọn thuật toán/cấu trúc theo độ phức tạp. Phỏng vấn lập trình cũng hỏi Big O thường xuyên.

## 16. Liên hệ Unity

`Update()` chạy mỗi frame. Đặt O(n²) nặng trong Update với n lớn → FPS rơi. Object pooling, tránh allocate, giảm tìm kiếm tuyến tự trên tập lớn… đều bắt đầu từ trực giác “mỗi frame được phép tốn bao nhiêu bước”. Level 10–19 sẽ đào sâu hơn; Level 0 chỉ cần **cảm** được độ lớn.

## 17. Kiểm tra kiến thức

1. Big O đo chính xác mili-giây — đúng/sai?  
2. Duyệt list một lần thường là?  
3. Vòng lồng hai tầng thường là?  
4. Binary search thuộc bậc nào?  
5. n nhỏ thì có luôn cần thuật toán tối ưu nhất không?

### Đáp án kiểm tra kiến thức

1. Sai — là ước lượng xu hướng tăng theo n.  
2. O(n).  
3. O(n²).  
4. O(log n).  
5. Không — ưu tiên đúng và rõ; tối ưu khi đo được nhu cầu.
