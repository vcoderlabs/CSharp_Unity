# Chương 02 — Chương trình chạy như thế nào

## 1. Mục tiêu học

- Mô tả được chu trình: **mã nguồn → biên dịch → mã máy / IL → chạy**.
- Phân biệt **source code**, **compiler**, **machine code**, **IL** (ý tưởng).
- Hiểu vì sao có lỗi “biên dịch” khác lỗi “khi chạy”.
- Biết vai trò của runtime (.NET / máy ảo) ở mức khái niệm.

## 2. Điều kiện tiên quyết

- Đã đọc Chương 01 (CPU, RAM, storage).
- Biết rằng chương trình là dãy lệnh máy tính thực thi.

## 3. Khái niệm

### Source code (mã nguồn)

Là văn bản **con người đọc được**, ví dụ:

```text
NẾU máu <= 0 THÌ in "Game Over"
```

Bạn viết source bằng editor (VS Code, Visual Studio…). Source được lưu trên **storage** dạng file `.cs`, `.py`, …

### Compiler (trình biên dịch)

Compiler là chương trình **dịch** source sang dạng máy hiểu được tốt hơn.  
Giống biên dịch viên: tiếng Việt kỹ thuật → ngôn ngữ mà “nhà máy” hiểu.

### Machine code (mã máy)

Là chuỗi lệnh rất thấp mà **CPU** thực thi trực tiếp (0 và 1 / instruction cụ thể). Khác nhau giữa loại CPU (x86, ARM…).

### IL (Intermediate Language) — ý tưởng với C# / .NET

Với C#, compiler thường dịch source → **IL** (ngôn ngữ trung gian), rồi runtime (.NET CLR) dịch/JIT tiếp thành mã máy khi chạy.  
Bạn chưa cần thuộc chi tiết — chỉ cần nhớ: **có bước trung gian**, không phải lúc nào cũng “một phát ra mã máy”.

### Interpreter vs Compiler (trực giác)

| Cách | Ý tưởng | Ví dụ gần |
|------|---------|-----------|
| Biên dịch trước | Dịch xong rồi chạy | C#, C++ (mô hình biên dịch) |
| Thông dịch | Đọc và chạy từng phần | Nhiều script / một số ngôn ngữ động |

Thực tế hiện đại thường **lai** (có IL + JIT). Ở Level 0 chỉ cần hình dung: **phải có bước biến source thành thứ máy chạy được**.

## 4. Mô hình tư duy

```text
  Bạn viết                 Compiler                 Runtime / CPU
┌──────────┐            ┌──────────┐            ┌────────────────┐
│ Source   │  ───────►  │  Dịch    │  ───────►  │ Machine code   │
│ (chữ)    │            │ (+ kiểm  │            │ hoặc IL → JIT  │
│          │            │  tra lỗi)│            │ rồi chạy       │
└──────────┘            └──────────┘            └────────────────┘
     ▲                        │
     │                        │ nếu sai cú pháp
     │                        ▼
     │                  Báo lỗi biên dịch
     │                  (chưa chạy được)
```

Hai “cổng lỗi” quan trọng:

```text
1) Lỗi biên dịch  → sai cú pháp / sai kiểu rõ ràng → KHÔNG ra chương trình chạy
2) Lỗi runtime    → biên dịch OK nhưng khi chạy mới vỡ (chia 0, null, sai logic…)
```

## 5. Cú pháp / Pseudocode

Mô tả quy trình bằng pseudocode:

```text
FUNCTION chạy_chương_trình(file_nguồn):
    kết_quả_dịch = COMPILER.dịch(file_nguồn)
    NẾU kết_quả_dịch.có_lỗi:
        HIỂN_THỊ lỗi_biên_dịch
        DỪNG
    chương_trình = kết_quả_dịch.output   // mã máy hoặc IL/assembly
    RUNTIME.nạp(chương_trình) vào RAM
    RUNTIME.bắt_đầu_thực_thi()            // CPU chạy lệnh
```

## 6. Ví dụ

### Cơ bản

Bạn viết nhầm:

```text
NẾU x > 10
    in "lớn"
// thiếu từ khóa hoặc cấu trúc đóng — tùy ngôn ngữ
```

Compiler báo lỗi → **chưa có chương trình để chạy**. Đây là lỗi biên dịch.

### Trung cấp

Code biên dịch thành công, nhưng:

```text
a = 10
b = 0
in a / b
```

Khi chạy mới lỗi chia cho 0 → **lỗi runtime**.

### Nâng cao (C#/.NET — chỉ trực giác)

```text
file .cs  →  (Roslyn/compiler)  →  IL trong file assembly (.dll/.exe)
                                    ↓
                              CLR + JIT
                                    ↓
                              mã máy CPU thực thi
```

Unity cũng dựa trên hệ sinh thái này (script C# được biên dịch rồi chạy trong player). Chi tiết để Level 1 và Level 21.

## 7. Lỗi thường gặp

1. Nghĩ “máy hiểu tiếng Việt/Anh trong source” — máy hiểu **mã đã dịch**.
2. Nhầm lỗi chính tả source với lỗi logic: sửa hết đỏ trong IDE chưa chắc chương trình đúng.
3. Nghĩ mọi ngôn ngữ đều biên dịch giống nhau — mô hình khác nhau, ý tưởng “dịch trước khi/ khi chạy” là đủ ở Level 0.
4. Sợ IL/JIT — tạm thời chỉ cần biết **có tầng trung gian**.

## 8. Gỡ lỗi / Kiểm tra hiểu biết

Tình huống: Bạn đổi một dòng code, bấm Run, chương trình vẫn hành xử như cũ.

**Cách nghĩ:** Có thể bạn chưa lưu file, hoặc đang chạy bản build cũ, hoặc IDE chưa biên dịch lại. Chu trình đúng: **lưu source → biên dịch lại → chạy bản mới**.

## 9. Best practices

- Mỗi lần sửa: **Save → Build/Run**.
- Đọc thông báo lỗi biên dịch từ **dòng đầu tiên** — thường chỉ đúng chỗ gốc.
- Phân loại nhanh: đây là lỗi **trước khi chạy** hay **khi đang chạy**?
- Giữ source rõ ràng; compiler không “đoán ý” nếu bạn viết mơ hồ.

## 10. Bài tập

**Bài 1.** Sắp xếp đúng thứ tự: Machine code / IL thực thi → Viết source → Compiler dịch → Nạp vào RAM & chạy.

**Bài 2.** Phân loại: (a) thiếu dấu ngoặc → ? (b) chia cho 0 → ? (c) công thức tính sai nhưng chạy ra số → ?

**Bài 3.** Viết 5–7 dòng pseudocode mô tả “từ lúc bạn gõ code đến lúc thấy kết quả trên màn hình”.

## 11. Gợi ý

- Bài 1: bắt đầu từ con người viết, kết thúc khi CPU chạy.
- Bài 2: biên dịch / runtime / lỗi logic (logic sai vẫn có thể “chạy được”).
- Bài 3: đừng quên bước biên dịch và nạp RAM.

## 12. Đáp án + Giải thích đáp án

**Bài 1.** Viết source → Compiler dịch → Nạp vào RAM & chạy → Machine code/IL được thực thi (thứ tự ý tưởng: viết → dịch → chạy).

Thứ tự chuẩn:

1. Viết source  
2. Compiler dịch  
3. Nạp vào RAM & chạy  
4. CPU thực thi machine code (sau JIT nếu có IL)

**Bài 2.**  
(a) Lỗi biên dịch  
(b) Lỗi runtime  
(c) Lỗi logic (bug) — chương trình chạy nhưng sai ý

**Bài 3 (mẫu):**

```text
1. Gõ source vào editor, lưu file
2. Gọi compiler dịch file
3. Nếu lỗi cú pháp → sửa, quay lại 2
4. Tạo ra file chương trình (assembly/IL/binary)
5. Hệ điều hành/runtime nạp vào RAM
6. CPU thực thi, in kết quả ra màn hình
```

## 13. Đáp án thay thế

Bài 1 có thể gộp “Nạp RAM & thực thi machine code” thành một bước “Chạy” — miễn là thứ tự **viết → dịch → chạy** đúng.

## 14. Thử thách

Giải thích vì sao cùng một file source C# có thể chạy trên Windows và Mac (ý tưởng: IL + runtime phù hợp từng nền tảng), trong khi mã máy thuần một kiến trúc CPU thì khó chạy trực tiếp sang máy khác.

## 15. Ứng dụng thực tế

CI/CD, đóng gói app, store game… đều xoay quanh: **build (biên dịch) → artifact → chạy trên máy người dùng**. Hiểu pipeline này giúp bạn không bối rối khi “trên máy tôi chạy được, máy bạn không” (thiếu runtime, build khác, …).

## 16. Liên hệ Unity

Script C# trong Unity được biên dịch trước khi Play/Build. Lỗi đỏ trong Console lúc biên dịch script khác với lỗi NullReference khi đang Play. Phân biệt sớm giúp bạn biết đang sửa **cú pháp** hay **logic runtime** — kỹ năng dùng hàng ngày khi làm game.

## 17. Kiểm tra kiến thức

1. Con người chủ yếu viết gì: source hay machine code?  
2. Compiler làm gì?  
3. Lỗi thiếu `;` (ở ngôn ngữ cần nó) thường là lỗi gì?  
4. IL là gì ở mức ý tưởng?  
5. Vì sao lỗi logic nguy hiểm?

### Đáp án kiểm tra kiến thức

1. Source code.  
2. Dịch source sang dạng máy/runtime dùng được; thường kèm kiểm tra cú pháp.  
3. Lỗi biên dịch.  
4. Ngôn ngữ trung gian giữa source và mã máy.  
5. Vì chương trình vẫn chạy nhưng cho kết quả sai — khó phát hiện hơn lỗi biên dịch.
