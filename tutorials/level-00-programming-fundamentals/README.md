# Level 0 — Programming Fundamentals

> **Thời lượng ước tính:** ~15 giờ  
> **Đối tượng:** Người mới hoàn toàn, chưa từng lập trình  
> **Ngôn ngữ:** Tiếng Việt (khái niệm + pseudocode — chưa cần C# chạy được)

---

## Mục tiêu Level này

Sau khi hoàn thành Level 0, bạn sẽ:

1. Hiểu máy tính lưu trữ và thực thi chương trình như thế nào (CPU, RAM, storage).
2. Biết chu trình **source → compiler → machine code / IL**.
3. Tư duy được bằng **biến, biểu thức, câu lệnh, luồng điều khiển, hàm**.
4. Đọc/ghi dữ liệu cơ bản (Input/Output) và biết cách **gỡ lỗi tư duy**.
5. Nắm thuật toán đơn giản (tìm kiếm, sắp xếp) và cấu trúc dữ liệu cơ bản (mảng, danh sách, map).
6. Có **trực giác Big O** — ước lượng “chậm/nhanh” của giải pháp.
7. Giải được bài toán logic bằng **flowchart + pseudocode**.

Level này **không yêu cầu viết C# chạy được**. Mục tiêu là xây **tư duy lập trình** trước khi vào Level 1.

---

## Danh sách 11 chương

| # | File | Chủ đề | Thời lượng gợi ý |
|---|------|--------|------------------|
| 01 | [01-may-tinh-hoat-dong.md](./01-may-tinh-hoat-dong.md) | Máy tính hoạt động thế nào (CPU, RAM, storage) | ~1h |
| 02 | [02-chuong-trinh-chay-nhu-the-nao.md](./02-chuong-trinh-chay-nhu-the-nao.md) | Chương trình chạy như thế nào | ~1h |
| 03 | [03-bien-va-du-lieu.md](./03-bien-va-du-lieu.md) | Biến và dữ liệu trong bộ nhớ | ~1.5h |
| 04 | [04-bieu-thuc-va-cau-lenh.md](./04-bieu-thuc-va-cau-lenh.md) | Biểu thức và câu lệnh | ~1h |
| 05 | [05-luong-dieu-khien.md](./05-luong-dieu-khien.md) | Luồng điều khiển (if/else, vòng lặp) | ~1.5h |
| 06 | [06-ham.md](./06-ham.md) | Hàm — trừu tượng hóa hành vi | ~1.5h |
| 07 | [07-input-output.md](./07-input-output.md) | Input / Output cơ bản | ~1h |
| 08 | [08-debug-co-ban.md](./08-debug-co-ban.md) | Debugging fundamentals | ~1h |
| 09 | [09-thuat-toan-co-ban.md](./09-thuat-toan-co-ban.md) | Thuật toán cơ bản (tìm kiếm, sắp xếp) | ~1.5h |
| 10 | [10-cau-truc-du-lieu-co-ban.md](./10-cau-truc-du-lieu-co-ban.md) | Cấu trúc dữ liệu cơ bản | ~1.5h |
| 11 | [11-do-phuc-tap-thoi-gian.md](./11-do-phuc-tap-thoi-gian.md) | Độ phức tạp thời gian (Big O trực giác) | ~1.5h |

**Project cuối Level:** [project-flowchart-pseudocode.md](./project-flowchart-pseudocode.md) — *Flowchart-to-Pseudocode Solver* (~2h)

**Tổng bài tập tư duy:** ~35 bài (trải đều các chương + project).

---

## Project: Flowchart-to-Pseudocode Solver

Bạn sẽ giải **5 bài toán logic** bằng cách:

1. Vẽ/mô tả **flowchart** (sơ đồ khối).
2. Viết **pseudocode** (mã giả) tương ứng.
3. Kiểm tra với vài bộ dữ liệu đầu vào.

Không cần biên dịch hay chạy C#. Tập trung vào **đúng logic**.

---

## Hướng dẫn học

### Cách học hiệu quả

1. **Đọc tuần tự** từ chương 01 → 11. Đừng nhảy cóc (mỗi chương dựa trên chương trước).
2. **Làm bài tập trước khi xem đáp án.** Gợi ý (mục 11) chỉ mở khi bí.
3. **Viết tay / gõ pseudocode** — đừng chỉ đọc. Tư duy lập trình hình thành khi bạn tự viết bước.
4. **Kiểm tra kiến thức** (5 câu cuối mỗi chương) — làm ngay sau khi đọc xong.
5. **Hoàn thành project** trước khi sang Level 1.

### Nhịp học gợi ý

| Track | Cách chia |
|-------|-----------|
| **Deep (người mới hoàn toàn)** | 1 chương / ngày (~1–1.5h) + project cuối tuần |
| **Normal** | 2 chương / ngày, hoàn Level trong ~1 tuần |
| **Fast** | Bỏ qua Level 0 nếu đã biết lập trình ngôn ngữ khác |

### Dấu hiệu “đã sẵn sàng Level 1”

- [ ] Giải thích được CPU / RAM / storage bằng ngôn ngữ đời thường.
- [ ] Viết được if/else và vòng lặp bằng pseudocode.
- [ ] Tách bài toán thành các **hàm** nhỏ.
- [ ] Tự tìm lỗi logic trong một đoạn pseudocode sai.
- [ ] Biết khi nào dùng mảng vs danh sách vs map (khái niệm).
- [ ] Ước lượng được “cách A chậm hơn cách B” ở mức trực giác Big O.
- [ ] Hoàn thành project 5 bài toán flowchart → pseudocode.

---

## Liên hệ lộ trình tổng

```text
Level 0 (bạn đang ở đây)  →  Level 1 C# Fundamentals  →  ...  →  Level 21 Unity
```

Level 0 là nền **tư duy**. Level 1 sẽ gắn tư duy đó vào cú pháp C# thật.

**Bỏ qua Level 0 nếu:** bạn đã từng lập trình thành thạo Java, Python, C++, JavaScript… và đã quen biến, vòng lặp, hàm, cấu trúc dữ liệu cơ bản.

---

## Cấu trúc mỗi chương

Mỗi file chương có **17 mục cố định**:

1. Mục tiêu học  
2. Điều kiện tiên quyết  
3. Khái niệm  
4. Mô hình tư duy  
5. Cú pháp / Pseudocode  
6. Ví dụ (Cơ bản / Trung cấp / Nâng cao)  
7. Lỗi thường gặp  
8. Gỡ lỗi / Kiểm tra hiểu biết  
9. Best practices  
10. Bài tập  
11. Gợi ý  
12. Đáp án + Giải thích  
13. Đáp án thay thế  
14. Thử thách  
15. Ứng dụng thực tế  
16. Liên hệ Unity  
17. Kiểm tra kiến thức  

---

Chúc bạn học vững nền tảng. Bắt đầu tại: **[Chương 01 — Máy tính hoạt động thế nào](./01-may-tinh-hoat-dong.md)**.
