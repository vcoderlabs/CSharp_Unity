# Project Level 0 — Flowchart-to-Pseudocode Solver

> **Thời lượng:** ~2 giờ  
> **Yêu cầu:** Không cần biên dịch C#. Nộp flowchart (ASCII hoặc giấy chụp) + pseudocode + vài test case tay.  
> **Mục tiêu:** Ghép mọi kỹ năng Level 0: biến, if/loop, hàm, I/O, thuật toán tư duy, kiểm thử.

---

## 1. Mục tiêu học

- Chuyển bài toán đời thường → **flowchart** → **pseudocode**.
- Tách bài thành hàm nhỏ khi logic dài.
- Tự kiểm tra bằng bảng input/output.
- Trình bày lời giải rõ ràng, người khác đọc được.

## 2. Điều kiện tiên quyết

- Hoàn thành (hoặc đọc xong) các chương 01–11.
- Biết if, while/for, hàm, list/map ở mức khái niệm.

## 3. Khái niệm — Quy trình làm project

Với **mỗi** bài toán, làm đủ 5 bước:

```text
1. Đọc kỹ đề — xác định INPUT / OUTPUT / ràng buộc
2. Vẽ FLOWCHART (sơ đồ khối)
3. Viết PSEUDOCODE
4. TRACE bằng tay ≥ 3 bộ dữ liệu (gồm case biên)
5. (Tuỳ chọn) Tối ưu tên biến / tách hàm / ghi chú Big O trực giác
```

### Ký hiệu flowchart gợi ý

```text
    ┌─────────┐
    │  Bắt đầu │
    └────┬────┘
         ▼
    ┌─────────┐
    │  Nhập   │     hình chữ nhật: xử lý / gán
    └────┬────┘
         ▼
    ◆ điều kiện ◆   hình thoi: quyết định
     /         \
   Có          Không
```

## 4. Mô hình tư duy

```text
Bài toán thực tế
      │
      ▼
  INPUT / OUTPUT rõ
      │
      ▼
  Flowchart (nhìn được nhánh & vòng)
      │
      ▼
  Pseudocode (viết được từng bước)
      │
      ▼
  Test tay → sửa logic
```

## 5. Cú pháp / Pseudocode

Dùng thống nhất:

```text
FUNCTION ten(...):
    ...
    RETURN ...

NẾU ... / NGƯỢC_LẠI
WHILE ... / FOR ...
HIỂN_THỊ / ĐỌC_INPUT / PARSE_INT
```

## 6. Đề bài — 5 bài toán logic

### Bài A — Máy bán nước (Cơ bản)

**Mô tả:** Máy có 3 loại: A=10000, B=12000, C=15000. Người dùng nhập mã (A/B/C) và số tiền đưa.  
**Output:** Nếu đủ tiền → in “Đã lấy nước X, thối lại Y”. Nếu thiếu → in “Thiếu Z”. Nếu mã sai → in “Mã không hợp lệ”.

**Gợi ý flowchart:** nhập → kiểm tra mã → so sánh tiền → nhánh thối/thiếu.

### Bài B — Đoán số có giới hạn (Trung cấp)

**Mô tả:** Máy chọn `bi_mat` trong 1..20 (coi như đã biết sẵn trong test, ví dụ 13). Người chơi đoán tối đa 5 lần. Mỗi lần: báo “lớn hơn” / “nhỏ hơn” / “đúng”. Hết 5 lần mà sai → “Thua, đáp án là …”.

**Yêu cầu thêm:** Viết ít nhất 1 hàm phụ (ví dụ `danh_gia_doan(doan, bi_mat)`).

### Bài C — Điểm danh lớp (Trung cấp)

**Mô tả:** Cho danh sách tên học sinh (list). Nhập lần lượt tên có mặt cho đến khi nhập `"xong"`.  
**Output:**  
- Số người có mặt  
- Số người vắng  
- Danh sách vắng  

**Gợi ý:** Dùng list / ý tưởng “đánh dấu có mặt”. Có thể dùng map tên→boolean.

### Bài D — Sắp xếp bảng xếp hạng (Nâng cao hơn)

**Mô tả:** Cho list điểm số (số nguyên). Hãy **sắp xếp giảm dần** bằng selection sort hoặc bubble (tự chọn), rồi in Top 3 (nếu ít hơn 3 phần tử thì in hết).

**Yêu cầu:** Flowchart thể hiện vòng lặp sắp xếp; pseudocode có hàm `sap_xep_giam(ds)` và `in_top3(ds)`.

### Bài E — Rương đồ thông minh (Nâng cao)

**Mô tả:** Túi đồ là map `ten_vat → so_luong`. Hỗ trợ các lệnh (đọc chuỗi, lặp đến `quit`):

| Lệnh | Ý nghĩa |
|------|---------|
| `add <ten> <n>` | thêm n vào vật (n > 0) |
| `use <ten> <n>` | bớt n; nếu không đủ → báo lỗi, không đổi |
| `count <ten>` | in số lượng (0 nếu chưa có) |
| `list` | in mọi vật có số lượng > 0 |
| `quit` | thoát |

**Yêu cầu:** Tách hàm xử lý từng lệnh; validate `n`.

## 7. Lỗi thường gặp trong project

1. Flowchart thiếu nhánh “mã sai” / “hết lượt”.  
2. Pseudocode nhảy cóc — không khớp flowchart.  
3. Không test case biên (tiền đúng bằng giá, đoán đúng lần cuối, list rỗng…).  
4. Off-by-one khi Top 3 / số lần đoán.  
5. `use` làm số lượng âm vì quên kiểm tra.

## 8. Gỡ lỗi / Kiểm tra hiểu biết

Với mỗi bài, lập bảng:

| # | Input | Output mong đợi | Output khi trace | Khớp? |
|---|-------|-----------------|------------------|-------|
| 1 | … | … | … | |

Nếu không khớp: đánh dấu bước flowchart đầu tiên lệch.

## 9. Best practices

- Đặt tên biến tiếng Việt không dấu hoặc English rõ (`tien_thoi`, `so_lan_doan`).  
- Một khối flowchart ≈ một đoạn pseudocode ngắn.  
- Case biên bắt buộc: rỗng, đúng biên, không hợp lệ.  
- Sau khi xong 5 bài: viết ½ trang tự rút ra “mình hay sai ở đâu”.

## 10. Bài tập (checklist nộp)

Đánh số bắt buộc:

1. **Bài A** — flowchart + pseudocode + ≥3 test.  
2. **Bài B** — flowchart + pseudocode (có hàm) + ≥3 test.  
3. **Bài C** — flowchart + pseudocode + ≥3 test.  
4. **Bài D** — flowchart + pseudocode + trace 1 mảng mẫu đến Top 3.  
5. **Bài E** — flowchart tổng + pseudocode + kịch bản 5–7 lệnh mẫu.

*(Đây là 5 bài project; cộng với ~30 bài trong các chương ≈ 35 bài Level 0.)*

## 11. Gợi ý (spoiler nhẹ)

- **A:** `NẾU ma == "A" ... NGƯỢC_LẠI_NẾU ... NGƯỢC_LẠI mã sai`; tính `thoi = tien - gia`.  
- **B:** `WHILE so_lan < 5 VÀ chua_dung`; tăng `so_lan` mỗi lần đoán sai/đúng đều tính lần.  
- **C:** Duyệt danh sách lớp; với mỗi tên, kiểm tra có trong tập “có mặt” không.  
- **D:** Đổi điều kiện sort thành chọn **max** thay vì min nếu làm selection giảm dần.  
- **E:** Parse lệnh bằng tách từ (ý tưởng `parts = TACH(chuoi)`); `parts[0]` là verb.

## 12. Đáp án mẫu + Giải thích

> Đáp án dưới đây là **một** cách đúng. Bạn có thể khác miễn logic đủ case.

### Đáp án Bài A (rút gọn)

```text
FUNCTION gia_cua(ma):
    NẾU ma == "A": RETURN 10000
    NẾU ma == "B": RETURN 12000
    NẾU ma == "C": RETURN 15000
    RETURN -1

HIỂN_THỊ "Nhập mã:"
SET ma = ĐỌC_INPUT()
SET gia = gia_cua(ma)
NẾU gia < 0:
    HIỂN_THỊ "Mã không hợp lệ"
NGƯỢC_LẠI:
    HIỂN_THỊ "Nhập tiền:"
    SET tien = PARSE_INT(ĐỌC_INPUT())
    NẾU tien >= gia:
        HIỂN_THỊ "Đã lấy nước", ma, ", thối lại", tien - gia
    NGƯỢC_LẠI:
        HIỂN_THỊ "Thiếu", gia - tien
```

**Giải thích:** Tách giá theo mã giúp tránh if lồng; kiểm tra mã trước khi hỏi tiền (cũng có thể hỏi tiền trước — chấp nhận được nếu nhất quán).

**Test mẫu:**  
`(A, 10000)` → thối 0; `(B, 5000)` → thiếu 7000; `(Z, 99999)` → mã không hợp lệ.

### Đáp án Bài B (rút gọn)

```text
FUNCTION danh_gia(doan, bi_mat):
    NẾU doan == bi_mat: RETURN "dung"
    NẾU doan < bi_mat: RETURN "nho_hon"
    RETURN "lon_hon"

SET bi_mat = 13
SET so_lan = 0
SET thang = false
WHILE so_lan < 5 VÀ thang == false:
    SET doan = PARSE_INT(ĐỌC_INPUT())
    SET so_lan = so_lan + 1
    SET kq = danh_gia(doan, bi_mat)
    NẾU kq == "dung":
        SET thang = true
        HIỂN_THỊ "Đúng!"
    NGƯỢC_LẠI_NẾU kq == "nho_hon":
        HIỂN_THỊ "Số bí mật LỚN HƠN"
    NGƯỢC_LẠI:
        HIỂN_THỊ "Số bí mật NHỎ HƠN"
NẾU thang == false:
    HIỂN_THỊ "Thua, đáp án là", bi_mat
```

**Giải thích:** Đếm lần đoán trong vòng; điều kiện dừng kép tránh đoán vượt 5 lần.

### Đáp án Bài C (rút gọn)

```text
SET lop = ["An", "Bình", "Chi", "Dũng"]
SET co_mat = {}    // map tên → true
SET ten = ĐỌC_INPUT()
WHILE ten != "xong":
    SET co_mat[ten] = true
    SET ten = ĐỌC_INPUT()

SET so_mat = 0
SET vang = []
FOR MỖI hs TRONG lop:
    NẾU CÓ_KHÓA(co_mat, hs):
        SET so_mat = so_mat + 1
    NGƯỢC_LẠI:
        THÊM_CUỐI(vang, hs)

HIỂN_THỊ "Có mặt:", so_mat
HIỂN_THỊ "Vắng:", length(lop) - so_mat
HIỂN_THỊ "DS vắng:", vang
```

**Giải thích:** Map cho phép đánh dấu nhanh; duyệt `lop` gốc để biết ai thiếu.

**Test:** Có mặt An, Chi, xong → vắng Bình, Dũng; so_mat=2.

### Đáp án Bài D (selection giảm dần)

```text
FUNCTION sap_xep_giam(ds):
    FOR i TỪ 0 ĐẾN length(ds) - 2:
        SET i_max = i
        FOR j TỪ i + 1 ĐẾN length(ds) - 1:
            NẾU ds[j] > ds[i_max]:
                SET i_max = j
        ĐỔI_CHỖ ds[i], ds[i_max]
    RETURN ds

FUNCTION in_top3(ds):
    SET gioi_han = MIN(3, length(ds))
    FOR i TỪ 0 ĐẾN gioi_han - 1:
        HIỂN_THỊ i + 1, ")", ds[i]

SET diem = [8, 10, 7, 9, 10]
SET diem = sap_xep_giam(diem)   // [10,10,9,8,7]
in_top3(diem)
```

**Giải thích:** Mỗi vòng chọn phần tử lớn nhất vùng chưa xếp đưa về đầu — ra thứ tự giảm dần.

### Đáp án Bài E (khung)

```text
SET tui = {}

FUNCTION add(ten, n):
    NẾU n <= 0: HIỂN_THỊ "n không hợp lệ"; RETURN
    NẾU KHÔNG CÓ_KHÓA(tui, ten): SET tui[ten] = 0
    SET tui[ten] = tui[ten] + n

FUNCTION use(ten, n):
    NẾU n <= 0: HIỂN_THỊ "n không hợp lệ"; RETURN
    SET co = 0
    NẾU CÓ_KHÓA(tui, ten): SET co = tui[ten]
    NẾU co < n:
        HIỂN_THỊ "Không đủ"
    NGƯỢC_LẠI:
        SET tui[ten] = co - n

FUNCTION count(ten):
    NẾU CÓ_KHÓA(tui, ten): HIỂN_THỊ tui[ten]
    NGƯỢC_LẠI: HIỂN_THỊ 0

FUNCTION list_all():
    FOR MỖI (ten, sl) TRONG tui:
        NẾU sl > 0: HIỂN_THỊ ten, sl

// vòng lệnh chính
SET line = ĐỌC_INPUT()
WHILE line != "quit":
    SET p = TACH(line)    // ["add","sword","2"]
    NẾU p[0] == "add": add(p[1], PARSE_INT(p[2]))
    NGƯỢC_LẠI_NẾU p[0] == "use": use(p[1], PARSE_INT(p[2]))
    NGƯỢC_LẠI_NẾU p[0] == "count": count(p[1])
    NGƯỢC_LẠI_NẾU p[0] == "list": list_all()
    NGƯỢC_LẠI: HIỂN_THỊ "Lệnh lạ"
    SET line = ĐỌC_INPUT()
```

**Kịch bản mẫu:**

```text
add potion 2
add sword 1
use potion 1
count potion      → 1
list              → potion 1; sword 1
use potion 5      → Không đủ
quit
```

## 13. Đáp án thay thế

- **A:** Dùng map `gia["A"]=10000` thay cho hàm `gia_cua`.  
- **C:** Dùng list `co_mat` + linear search thay map.  
- **D:** Bubble sort đổi điều kiện `>` để đưa số lớn về đầu.  
- **E:** Dùng `switch` ý tưởng theo verb; hoặc bắt lỗi thiếu tham số chi tiết hơn.

## 14. Thử thách tổng hợp

Gộp Bài E + ý Big O: giả sử `list` được gọi mỗi frame trong game với 10.000 loại vật — bạn nhận xét gì? Đề xuất cấu trúc / chiến lược in UI (chỉ in khi túi đổi, phân trang…).

## 15. Ứng dụng thực tế

Flowchart → pseudocode chính là bước phân tích trước khi code thật trong team: giảm viết sai, dễ review, dễ chia việc (“bạn làm hàm validate, tôi làm vòng lệnh”).

## 16. Liên hệ Unity

Thiết kế hệ thống inventory, đoán mini-game, bảng rank, vendor/shop trong Unity đều bắt đầu bằng logic như 5 bài trên. Làm rõ flowchart trước khi tạo script MonoBehaviour giúp tránh `Update()` spaghetti và bug “use item làm âm số lượng”.

## 17. Kiểm tra kiến thức (cuối project)

1. Thứ tự khuyến nghị: code C# trước hay flowchart trước (ở Level 0)?  
2. Case biên là gì? Cho 1 ví dụ từ Bài A.  
3. Vì sao Bài B cần giới hạn số lần?  
4. Map giúp gì ở Bài C/E?  
5. Dấu hiệu bạn đã sẵn sàng Level 1?

### Đáp án kiểm tra kiến thức

1. Flowchart / pseudocode trước — Level 0 chưa yêu cầu C# chạy.  
2. Ví dụ: tiền đưa **đúng bằng** giá → thối 0.  
3. Để trò chơi kết thúc được; tránh vòng vô hạn / trải nghiệm không công bằng.  
4. Tra cứu theo tên/khóa nhanh, rõ nghĩa hơn tìm trong list dài.  
5. Tự viết được if/loop/hàm/pseudocode cho bài mới và hoàn thành 5 bài project.

---

## Tiêu chí hoàn thành Level 0

- [ ] Đủ 5 bài A–E (flowchart + pseudocode + test)  
- [ ] Tự giải thích được CPU/RAM/storage, biên dịch, biến, control flow, hàm, I/O, debug, search/sort, list/map, Big O trực giác  
- [ ] Sẵn sàng sang **Level 1 — C# Fundamentals**

Quay lại: [README Level 0](./README.md)
