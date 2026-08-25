# Chương 3 — StreamReader và StreamWriter

## 1. Mục tiêu học

- Đọc/ghi **text** qua `StreamReader` / `StreamWriter`
- Chọn encoding (UTF-8)
- Đọc từng dòng hiệu quả (`ReadLineAsync`, `ReadToEndAsync`)
- Kết hợp với `FileStream` và `leaveOpen`

## 2. Điều kiện tiên quyết

- Chương 2: Stream
- Level 1: string

## 3. Khái niệm

Stream làm việc với **byte**. Text cần encoding:

```text
string  ←encoding→  bytes  ←→ Stream
```

`StreamWriter` / `StreamReader` bọc Stream (hoặc path tiện lợi).

| Method | Dùng khi |
|--------|----------|
| `ReadLineAsync` | Log, CSV đơn giản, từng dòng |
| `ReadToEndAsync` | File text nhỏ/trung bình |
| `WriteLineAsync` | Ghi dòng có newline |

`File.ReadAllLinesAsync` tiện — bên dưới tương tự reader.

## 4. Mô hình tư duy

```text
FileStream (bytes)
   └─ StreamWriter (chars → bytes UTF-8)
   └─ StreamReader (bytes → chars)
```

Luôn `FlushAsync` hoặc dispose writer trước khi đọc lại cùng file.

## 5. Cú pháp

Đọc/ghi text UTF-8:

```csharp
await using var writer = new StreamWriter("log.txt", append: true, Encoding.UTF8);
await writer.WriteLineAsync($"{DateTime.UtcNow:o} hello");

await using var reader = new StreamReader("log.txt", Encoding.UTF8);
string? line;
while ((line = await reader.ReadLineAsync()) is not null)
    Console.WriteLine(line);
```

## 6. Ví dụ

### Cơ bản

Ghi và đọc hết:

```csharp
await File.WriteAllTextAsync("a.txt", "xin chào", Encoding.UTF8);
string s = await File.ReadAllTextAsync("a.txt", Encoding.UTF8);
```

### Trung cấp

Đếm dòng không load hết mảng:

```csharp
static async Task<int> CountLinesAsync(string path)
{
    int count = 0;
    await using var fs = new FileStream(path, FileMode.Open, FileAccess.Read, FileShare.Read, 4096, true);
    using var reader = new StreamReader(fs, Encoding.UTF8);
    while (await reader.ReadLineAsync() is not null) count++;
    return count;
}
```

### Nâng cao

Writer trên MemoryStream + reader:

```csharp
await using var ms = new MemoryStream();
await using (var sw = new StreamWriter(ms, Encoding.UTF8, leaveOpen: true))
{
    await sw.WriteLineAsync("one");
    await sw.WriteLineAsync("two");
}
ms.Position = 0;
using var sr = new StreamReader(ms, Encoding.UTF8);
Console.WriteLine(await sr.ReadToEndAsync());
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Tiếng Việt lỗi font | Sai encoding / thiếu UTF-8 | Chỉ định `Encoding.UTF8` |
| Đọc rỗng sau ghi | Chưa flush / Position | Dispose writer; Seek 0 |
| BOM bất ngờ | UTF8 có BOM | `new UTF8Encoding(false)` |
| File lock | Reader giữ lâu | using ngắn |

## 8. Gỡ lỗi

1. Mở file bằng editor hex — xem BOM `EF BB BF`.
2. In `reader.CurrentEncoding`.
3. Kiểm newline `\r\n` vs `\n` khi parse.

## 9. Best practices

- UTF-8 không BOM cho hầu hết file data/modern.
- Đọc dòng lớn: cân nhắc giới hạn độ dài chống OOM.
- `await using` writer; `FlushAsync` nếu share stream.
- CSV phức tạp: đừng parse tay — dùng thư viện (CsvHelper) khi production.

## 10. Bài tập

**Bài 1** — Ghi 10 dòng số bằng StreamWriter.

**Bài 2** — Đọc lại, cộng tổng số.

**Bài 3** — Append một dòng timestamp mỗi lần chạy chương trình.

**Bài 4** — Chuyển file sang UTF-8 (đọc encoding cũ giả định Latin1, ghi UTF-8).

## 11. Gợi ý

- Bài 1–2: WriteLine vòng for; ReadLine + int.Parse.
- Bài 3: `append: true`.
- Bài 4: `Encoding.GetEncoding(28591)` hoặc `ISO-8859-1` → Write UTF8.

## 12. Đáp án

**Bài 1–2** — Ghi và cộng:

```csharp
await using (var sw = new StreamWriter("nums.txt", false, Encoding.UTF8))
{
    for (int i = 1; i <= 10; i++) await sw.WriteLineAsync(i.ToString());
}

int sum = 0;
using (var sr = new StreamReader("nums.txt", Encoding.UTF8))
{
    string? line;
    while ((line = await sr.ReadLineAsync()) is not null)
        sum += int.Parse(line);
}
Console.WriteLine(sum); // 55
```

**Bài 3** — Append log:

```csharp
await using var sw = new StreamWriter("run.log", append: true, Encoding.UTF8);
await sw.WriteLineAsync(DateTime.UtcNow.ToString("o"));
```

**Bài 4** — Đổi encoding:

```csharp
string content = await File.ReadAllTextAsync("old.txt", Encoding.Latin1);
await File.WriteAllTextAsync("new.txt", content, new UTF8Encoding(encoderShouldEmitUTF8Identifier: false));
```

## 13. Đáp án thay thế

`File.AppendAllLinesAsync` / `ReadLinesAsync` (.NET) cho API ngắn hơn.

## 14. Thử thách

Viết tail -f đơn giản: mỗi 500ms đọc dòng mới từ Position đã lưu (cần FileStream Seek).

## 15. Ứng dụng thực tế

- Log file text
- Export báo cáo CSV đơn giản
- Đọc config `.env` / `.ini` dòng

## 16. Liên hệ Unity

- Save text vào `persistentDataPath`.
- Editor tools: generate `.cs` / `.asset` text — cẩn thận encoding.
- Tránh `StreamReader` sync trên main với file lớn.

## 17. Kiểm tra kiến thức

1. Vì sao cần StreamReader thay vì chỉ FileStream?  
   **Đáp án:** Chuyển bytes ↔ text theo encoding.

2. Encoding nên dùng mặc định cho data hiện đại?  
   **Đáp án:** UTF-8.

3. `append: true` làm gì?  
   **Đáp án:** Ghi tiếp cuối file, không xóa nội dung cũ.

4. Đọc từng dòng giúp gì với file lớn?  
   **Đáp án:** Không cần giữ toàn bộ nội dung trong một string lớn.

5. `leaveOpen: true` khi nào?  
   **Đáp án:** Khi dispose writer/reader nhưng vẫn cần dùng Stream bên dưới.
