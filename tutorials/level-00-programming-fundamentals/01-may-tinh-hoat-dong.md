# Chương 01 — Máy tính hoạt động thế nào

## 1. Mục tiêu học

- Giải thích được vai trò của **CPU**, **RAM** và **storage** bằng ngôn ngữ đời thường.
- Phân biệt được dữ liệu **tạm thời** (trong RAM) và **lâu dài** (trên ổ cứng).
- Hiểu vì sao chương trình “chậm” hoặc “hết bộ nhớ”.
- Liên hệ được các khái niệm này với việc viết chương trình sau này.

## 2. Điều kiện tiên quyết

- Không cần biết lập trình.
- Biết dùng máy tính ở mức cơ bản (mở file, lưu file).

## 3. Khái niệm

Hãy tưởng tượng máy tính như một **nhà bếp**:

| Thành phần | Ví dụ đời thường | Vai trò |
|------------|------------------|---------|
| **CPU** | Đầu bếp | Thực hiện mọi phép tính / lệnh |
| **RAM** | Bàn làm việc | Chỗ để nguyên liệu đang dùng — nhanh nhưng mất khi tắt bếp |
| **Storage** (SSD/HDD) | Tủ lạnh / kho | Lưu lâu dài — chậm hơn bàn làm việc nhưng bền |

### CPU (Central Processing Unit)

CPU là “bộ não” thực thi lệnh từng bước: cộng, so sánh, nhảy đến lệnh khác…  
CPU **không nhớ lâu** — nó lấy dữ liệu từ RAM, xử lý, rồi ghi kết quả trở lại RAM (hoặc storage).

### RAM (Random Access Memory)

RAM là bộ nhớ **làm việc tạm thời**. Khi bạn mở ứng dụng, dữ liệu cần thiết được nạp vào RAM để CPU làm việc nhanh.  
**Tắt máy / tắt chương trình → dữ liệu trong RAM thường mất.**

### Storage (ổ cứng / SSD)

Storage lưu trữ **lâu dài**: hệ điều hành, game, tài liệu, file save game…  
CPU không làm việc trực tiếp trên storage từng byte một cách nhanh như RAM — dữ liệu thường được **đọc vào RAM** rồi mới xử lý.

### Tóm tắt một câu

> Chương trình nằm trên **storage**; khi chạy được nạp vào **RAM**; **CPU** đọc lệnh từ RAM và thực thi.

## 4. Mô hình tư duy

```text
┌─────────────────────────────────────────────────┐
│                    STORAGE                       │
│   (ổ cứng / SSD: file chương trình, save game)  │
└──────────────────────┬──────────────────────────┘
                       │ nạp khi chạy
                       ▼
┌─────────────────────────────────────────────────┐
│                      RAM                         │
│   (dữ liệu đang dùng, biến, code đang chạy)     │
└──────────────────────┬──────────────────────────┘
                       │ đọc / ghi liên tục
                       ▼
┌─────────────────────────────────────────────────┐
│                      CPU                         │
│   (tính toán, so sánh, quyết định bước tiếp)    │
└─────────────────────────────────────────────────┘
```

Luồng đơn giản khi mở game:

```text
1. Click icon game          → hệ điều hành tìm file trên STORAGE
2. Nạp phần cần thiết       → đưa vào RAM
3. CPU bắt đầu vòng lặp game → Update từng frame từ dữ liệu trong RAM
4. Bạn Save game            → ghi từ RAM xuống STORAGE
5. Bạn thoát game           → giải phóng RAM (storage vẫn còn save)
```

## 5. Cú pháp / Pseudocode

Ở chương này chưa có cú pháp ngôn ngữ. Dùng **mô tả bước** (pseudocode đời thường):

```text
KHI người_dùng_mở_ứng_dụng:
    Đọc file từ STORAGE vào RAM
    LẶP:
        CPU lấy lệnh tiếp theo từ RAM
        CPU thực thi lệnh
        NẾU cần lưu kết quả lâu dài:
            Ghi từ RAM xuống STORAGE
    KHI người_dùng_thoát:
        Giải phóng vùng RAM của ứng dụng
```

## 6. Ví dụ

### Cơ bản

Bạn mở Word, gõ một đoạn văn, **chưa bấm Save**, rồi mất điện.  
→ Nội dung vừa gõ chủ yếu nằm trong **RAM** → thường **mất**.  
Nếu đã Save → đã ghi xuống **storage** → còn lại được.

### Trung cấp

Máy có 8 GB RAM, bạn mở Chrome 30 tab + Unity Editor + Spotify.  
→ RAM đầy → hệ điều hành phải **đẩy bớt dữ liệu** xuống storage (swap) → máy chậm rõ rệt.  
Đây không phải “CPU yếu” thuần túy — thường là **thiếu RAM / quá nhiều tiến trình**.

### Nâng cao

Game load map lớn: thanh loading dài vì đang **đọc từ storage → RAM**.  
Sau khi vào map, di chuyển mượt hơn vì dữ liệu đang nằm sẵn trong RAM để CPU xử lý từng frame.

## 7. Lỗi thường gặp

1. Nghĩ “máy chậm = CPU kém” — đôi khi là thiếu RAM hoặc ổ cứng chậm.
2. Nghĩ RAM và ổ cứng là một — **không**: một cái tạm thời & nhanh, một cái lâu dài & chậm hơn.
3. Nghĩ tắt màn hình = tắt máy = mất hết trên ổ cứng — **không**: storage vẫn giữ file.
4. Nghĩ “xóa thùng rác là xóa RAM” — thùng rác liên quan **storage**.

## 8. Gỡ lỗi / Kiểm tra hiểu biết

Tình huống: Bạn chỉnh sửa file, Ctrl+S thành công, rồi rút USB nguồn (máy tắt). Mở lại máy — file còn không?

**Cách nghĩ:** Ctrl+S thường ghi xuống **storage**. Tắt máy làm mất RAM, nhưng file đã lưu trên storage vẫn còn (trừ khi lưu lỗi / ổ hỏng).

## 9. Best practices

- Coi **Save** là “đưa từ bàn làm việc vào tủ” — làm thường xuyên với dữ liệu quan trọng.
- Khi máy chậm: kiểm tra **RAM đang dùng bao nhiêu**, không chỉ nhìn CPU.
- Hiểu giới hạn: dữ liệu lớn → thời gian load dài (storage → RAM).
- Khi học lập trình: nhớ biến đang sống trong **RAM** lúc chương trình chạy.

## 10. Bài tập

**Bài 1.** Giải thích bằng 3–5 câu: vì sao mở nhiều tab Chrome làm máy chậm.

**Bài 2.** Phân loại hành động sau thuộc chủ yếu CPU / RAM / Storage:  
(a) Nhân hai số  
(b) Lưu ảnh vào thư mục Pictures  
(c) Giữ trạng thái game đang chơi (chưa save)  
(d) Cài đặt Unity Hub

**Bài 3.** Vẽ (ASCII hoặc giấy) sơ đồ: “Mở game → chơi → Save → thoát”.

## 11. Gợi ý

- Bài 1: nghĩ đến RAM đầy và hệ điều hành phải quản lý nhiều tiến trình.
- Bài 2: CPU = tính toán; RAM = đang giữ tạm; Storage = ghi/đọc lâu dài.
- Bài 3: mũi tên nạp từ storage → RAM → CPU; Save đi ngược về storage.

## 12. Đáp án + Giải thích đáp án

**Bài 1.** Mỗi tab giữ dữ liệu trang web trong RAM. Nhiều tab → RAM căng thẳng → hệ điều hành swap / giảm hiệu năng → cảm giác máy chậm. CPU cũng bận hơn nhưng gốc thường là **áp lực bộ nhớ**.

**Bài 2.**  
(a) CPU  
(b) Storage  
(c) RAM  
(d) Storage (cài đặt ghi file lâu dài; khi chạy mới dùng RAM/CPU)

**Bài 3 (mẫu):**

```text
STORAGE --nạp--> RAM <--> CPU
                 │
              Save │
                 ▼
              STORAGE
Thoát → giải phóng RAM; save vẫn trên STORAGE
```

## 13. Đáp án thay thế

Bài 2(c) có thể nói “RAM + một phần cache storage” nếu game streaming asset — với Level 0, trả lời **RAM** là đủ.

## 14. Thử thách

Giải thích vì sao SSD làm máy “mượt hơn” khi mở app so với HDD cổ, dù CPU và RAM giống nhau. (Gợi ý: tốc độ nạp storage → RAM.)

## 15. Ứng dụng thực tế

Mọi phần mềm — từ máy tính tiền đến app ngân hàng — đều dựa trên mô hình này. Hiểu CPU/RAM/storage giúp bạn đoán được bottleneck khi hệ thống chậm.

## 16. Liên hệ Unity

Trong Unity, scene/asset nằm trên **ổ đĩa**; khi Play được nạp vào **RAM**; mỗi frame **CPU** (và GPU) xử lý Update. Loading screen dài thường là đang đọc asset từ storage. Hiểu điều này giúp bạn không bất ngờ khi build game nặng bị giật lúc load.

## 17. Kiểm tra kiến thức

1. Dữ liệu nào mất khi tắt máy đột ngột nếu chưa Save?  
2. Thành phần nào thực thi phép cộng `2 + 3`?  
3. File `.exe` hoặc build game nằm ở đâu khi bạn chưa mở?  
4. Vì sao Save game quan trọng?  
5. RAM đầy thường gây hiện tượng gì?

### Đáp án kiểm tra kiến thức

1. Dữ liệu đang ở RAM (chưa ghi storage).  
2. CPU.  
3. Storage.  
4. Đưa tiến trình từ RAM xuống storage để giữ lâu dài.  
5. Máy chậm / swap / có thể bị đóng app.
