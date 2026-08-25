# Level 12 — File / IO / Serialization (~15 giờ)

Level này dạy bạn làm việc với **file hệ thống**, **Stream**, đọc/ghi text qua `StreamReader`/`StreamWriter`, và serialize JSON bằng **System.Text.Json** (.NET 8+).

**Điều kiện:** Đã hoàn thành [Level 11 — Async](../level-11-async/) (ưu tiên API `*Async` + `CancellationToken`). Level 4–5 giúp với collections/generics khi làm database file.

**Tiếp theo:** [Level 13 — Networking](../level-13-networking/) (HTTP + JSON body).

---

## Mục tiêu cấp độ

Sau Level 12 bạn sẽ:

- Dùng `File`, `Directory`, `Path` an toàn (path combine, tồn tại, enumerate)
- Hiểu mô hình **Stream**: `FileStream`, `MemoryStream`
- Đọc/ghi text hiệu quả với `StreamReader` / `StreamWriter`
- Serialize/deserialize JSON bằng `System.Text.Json`
- Xây **File-based Database** nhỏ (CRUD trên JSON/lines)

---

## Cảnh báo xuyên suốt Level 12

> **Đừng** nối path bằng `+ "\\"`. Dùng `Path.Combine` / `Path.Join`.  
> **Đừng** đọc file lớn bằng `ReadAllBytes` nếu có thể stream.  
> Prefer `async` IO trong app server; dispose bằng `await using`. JSON: tránh `JavaScriptSerializer` / Newtonsoft trừ khi project bắt buộc — mặc định học **System.Text.Json**.

---

## 4 chương + 1 project

| # | File | Nội dung | ~Giờ |
|---|------|----------|------|
| 1 | [01-file-directory.md](./01-file-directory.md) | File, Directory, Path | 2–3 |
| 2 | [02-streams.md](./02-streams.md) | Stream, FileStream, MemoryStream | 3–4 |
| 3 | [03-readers-writers.md](./03-readers-writers.md) | StreamReader / StreamWriter | 2–3 |
| 4 | [04-json-serialization.md](./04-json-serialization.md) | System.Text.Json | 3–4 |
| — | [project-file-database.md](./project-file-database.md) | File-based Database | 4–5 |

**Tổng ước lượng: ~15 giờ**

---

## Cách học đề xuất

1. Chương 1: thao tác thư mục temp riêng — đừng xóa nhầm home.
2. Chương 2: copy file bằng buffer 8KB — hiểu stream.
3. Chương 3: đọc log từng dòng; ghi UTF-8.
4. Chương 4: model C# ↔ JSON, options naming.
5. Project: CRUD entity lưu JSON lines hoặc một file JSON array.

---

## Checklist hoàn thành Level 12

- [ ] Dùng đúng `Path.Combine`, kiểm tra tồn tại, enumerate file
- [ ] Copy/đọc nhị phân bằng Stream + buffer
- [ ] Đọc/ghi text với Reader/Writer (async)
- [ ] Serialize/deserialize JSON có options rõ ràng
- [ ] Hoàn thành File-based Database
- [ ] Trả lời đúng ≥ 4/5 câu **Kiểm tra kiến thức** mỗi chương

Khi xong checklist → chuyển sang **Level 13**.
