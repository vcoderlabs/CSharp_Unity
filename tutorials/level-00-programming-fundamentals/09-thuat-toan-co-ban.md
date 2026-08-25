# Chương 09 — Thuật toán cơ bản

## 1. Mục tiêu học

- Hiểu **thuật toán** là chuỗi bước hữu hạn, rõ ràng để giải một bài toán.
- Viết được tìm kiếm tuần tự (linear search) bằng pseudocode.
- Hiểu ý tưởng tìm kiếm nhị phân (binary search) khi dữ liệu đã sắp xếp.
- Viết được sắp xếp đơn giản: interchange / selection sort hoặc bubble sort ý tưởng.

## 2. Điều kiện tiên quyết

- Chương 05–06 (vòng lặp, hàm).
- Biết khái niệm danh sách các phần tử (sẽ sâu hơn ở Chương 10).

## 3. Khái niệm

### Thuật toán là gì?

Công thức / quy trình: đầu vào → các bước → đầu ra.  
Đời thường: công thức nấu ăn, chỉ đường từ A → B.

Tiêu chí tốt ở mức nhập môn:

1. **Đúng**  
2. **Rõ ràng** (không mơ hồ)  
3. **Kết thúc** được  
4. (Sau này) đủ hiệu quả

### Tìm kiếm tuần tự (Linear Search)

Duyệt từ đầu đến cuối, so từng phần tử với mục tiêu.

- Ưu: đơn giản, không cần sắp xếp.  
- Nhược: chậm khi danh sách rất dài (ý tưởng — Chương 11).

### Tìm kiếm nhị phân (Binary Search) — trực giác

Khi danh sách **đã sắp xếp**, mỗi lần so sánh với phần tử giữa, bỏ đi một nửa.

Đời thường: đoán số 1..100, mỗi lần được biết “lớn hơn/nhỏ hơn”.

### Sắp xếp đơn giản

**Selection sort (ý tưởng):** mỗi vòng chọn phần tử nhỏ nhất còn lại, đưa về đúng vị trí.

**Bubble sort (ý tưởng):** cặp bên cạnh sai thứ tự thì đổi chỗ; lặp đến khi không còn đổi.

## 4. Mô hình tư duy

```text
Linear search:

[ 7 | 2 | 9 | 4 | 5 ]  tìm 9
  ^
  so sánh 7 ≠ 9
      ^
      2 ≠ 9
          ^
          9 = 9 → tìm thấy tại vị trí 2


Binary search (đã sort tăng dần):

[ 1 | 3 | 4 | 7 | 9 | 12 | 15 ] tìm 12
              ^ mid=7  12>7 → bỏ nửa trái
                      [ 9 | 12 | 15 ]
                            ^ 12=12 → thấy
```

## 5. Cú pháp / Pseudocode

```text
FUNCTION tim_tuan_tu(ds, muc_tieu):
    FOR i TỪ 0 ĐẾN length(ds) - 1:
        NẾU ds[i] == muc_tieu:
            RETURN i
    RETURN -1    // không thấy

FUNCTION tim_nhi_phan(ds_da_sort, muc_tieu):
    SET trai = 0
    SET phai = length(ds_da_sort) - 1
    WHILE trai <= phai:
        SET mid = (trai + phai) / 2   // chia lấy phần nguyên
        NẾU ds_da_sort[mid] == muc_tieu:
            RETURN mid
        NGƯỢC_LẠI_NẾU ds_da_sort[mid] < muc_tieu:
            SET trai = mid + 1
        NGƯỢC_LẠI:
            SET phai = mid - 1
    RETURN -1

FUNCTION sap_xep_chon(ds):
    FOR i TỪ 0 ĐẾN length(ds) - 2:
        SET vi_tri_min = i
        FOR j TỪ i + 1 ĐẾN length(ds) - 1:
            NẾU ds[j] < ds[vi_tri_min]:
                SET vi_tri_min = j
        ĐỔI_CHỖ ds[i] VÀ ds[vi_tri_min]
    RETURN ds
```

## 6. Ví dụ

### Cơ bản

Tìm `4` trong `[2, 4, 6]`:

```text
i=0: 2 ≠ 4
i=1: 4 = 4 → trả về 1
```

### Trung cấp

Binary search tìm `7` trong `[1,3,5,7,9]`:

```text
trai=0, phai=4, mid=2 → 5 < 7 → trai=3
trai=3, phai=4, mid=3 → 7 = 7 → trả về 3
```

### Nâng cao

Selection sort `[5, 2, 4, 1]`:

```text
i=0: min là 1 ở cuối → [1, 2, 4, 5] sau các vòng...
(chi tiết từng vòng: chọn min trong phần chưa sắp, đổi về i)
```

Vòng minh họa:

```text
[5,2,4,1] → chọn 1 đổi với 5 → [1,2,4,5]
[1|2,4,5] → phần còn lại đã gần đúng; tiếp tục tinh chỉnh
```

## 7. Lỗi thường gặp

1. Binary search trên mảng **chưa sort** → kết quả sai.
2. Off-by-one ở biên `trai`/`phai`/`mid`.
3. Quên trường hợp không tìm thấy (`RETURN -1`).
4. Đổi chỗ thiếu biến tạm → mất dữ liệu.

## 8. Gỡ lỗi / Kiểm tra hiểu biết

Binary search không bao giờ thấy phần tử có thật. Kiểm tra gì?  
→ Mảng đã sort đúng chiều chưa? Điều kiện `trai <= phai`? Cập nhật `trai/phai` có bỏ sót mid không?

## 9. Best practices

- Chứng minh đúng với 2–3 case: có phần tử, không có, 1 phần tử, rỗng.
- Đặt tên hàm rõ: `tim_tuan_tu`, không phải `xu_ly`.
- Chọn thuật toán đủ đơn giản trước khi tối ưu.
- Ghi chú giả định: “ds đã sắp xếp tăng dần”.

## 10. Bài tập

**Bài 1.** Viết linear search tìm `"kiếm"` trong danh sách tên vật phẩm; trả về chỉ số hoặc -1.

**Bài 2.** Cho mảng đã sort `[2,4,6,8,10]`. Mô tả từng bước binary search tìm `8`.

**Bài 3.** Chạy tay (trace) một vòng bubble: duyệt cặp cạnh nhau trên `[3,1,2]` một lượt từ trái sang phải — kết quả sau 1 lượt?

**Bài 4.** Viết hàm đếm số lần xuất hiện của `x` trong danh sách (thuật toán đếm tuần tự).

## 11. Gợi ý

- Bài 2: mid lần 1 là phần tử giữa; so sánh rồi hẹp nửa.
- Bài 3: đổi nếu trái > phải.
- Bài 4: biến `dem`, tăng khi bằng `x`.

## 12. Đáp án + Giải thích đáp án

**Bài 1.**

```text
FUNCTION tim_vat_pham(ds, ten):
    FOR i TỪ 0 ĐẾN length(ds) - 1:
        NẾU ds[i] == ten:
            RETURN i
    RETURN -1
```

**Bài 2.**  
`trai=0,phai=4,mid=2` → `6 < 8` → `trai=3`  
`mid=3` → `8 == 8` → chỉ số **3**.

**Bài 3.**  
`[3,1,2]`: so 3>1 đổi → `[1,3,2]`; so 3>2 đổi → `[1,2,3]`. Sau 1 lượt: `[1,2,3]`.

**Bài 4.**

```text
FUNCTION dem_x(ds, x):
    SET dem = 0
    FOR MỖI phan_tu TRONG ds:
        NẾU phan_tu == x:
            SET dem = dem + 1
    RETURN dem
```

## 13. Đáp án thay thế

Bài 3 nếu chỉ đổi một lần rồi dừng khác quy ước — đáp án chuẩn “một lượt bubble full pass” là `[1,2,3]`.

## 14. Thử thách

So sánh số lần so sánh tối đa: linear vs binary trên danh sách 1000 phần tử đã sort (ước lượng trực giác, không cần công thức chặt). Viết nhận xét 3–5 câu.

## 15. Ứng dụng thực tế

Tìm kiếm đơn hàng, sắp xếp bảng điểm, gợi ý từ điển… Thuật toán đúng quyết định app có dùng được với dữ liệu thật hay không.

## 16. Liên hệ Unity

Tìm enemy gần nhất, sắp xếp điểm high-score, pathfinding đơn giản… đều là thuật toán. Chọn linear search trên 5 enemy thì ổn; trên 10.000 entity mỗi frame thì có thể làm game giật — liên hệ Chương 11 và tối ưu Unity sau này.

## 17. Kiểm tra kiến thức

1. Thuật toán cần kết thúc được — đúng/sai?  
2. Binary search cần điều kiện gì?  
3. Linear search có cần sort không?  
4. Selection sort làm gì mỗi vòng ngoài?  
5. Trả về `-1` thường nghĩa gì trong tìm kiếm?

### Đáp án kiểm tra kiến thức

1. Đúng.  
2. Dữ liệu đã sắp xếp (theo đúng chiều so sánh).  
3. Không.  
4. Chọn phần tử nhỏ nhất (hoặc lớn nhất) trong vùng chưa sắp và đưa về vị trí đúng.  
5. Không tìm thấy.
