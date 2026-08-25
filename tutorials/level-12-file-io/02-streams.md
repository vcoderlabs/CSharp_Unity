# Chương 2 — Stream, FileStream, MemoryStream

## 1. Mục tiêu học

- Hiểu trừu tượng `Stream`: Read / Write / Seek
- Dùng `FileStream` đọc/ghi nhị phân với buffer
- Dùng `MemoryStream` như buffer trong RAM
- `await using` và copy stream (`CopyToAsync`)

## 2. Điều kiện tiên quyết

- Chương 1: Path/File
- Level 11: async + cancel (CopyToAsync)

## 3. Khái niệm

**Stream** = chuỗi byte tuần tự. Không phải lúc nào cũng hỗ trợ Seek (ví dụ network stream).

| Loại | Đặc điểm |
|------|----------|
| `FileStream` | File trên đĩa; sync/async |
| `MemoryStream` | Byte trong RAM; Seek tốt |
| `BufferedStream` | Bọc stream khác thêm buffer |
| `NetworkStream` | TCP (Level 13) |

`CanRead` / `CanWrite` / `CanSeek` cho biết khả năng.

## 4. Mô hình tư duy

```text
Nguồn ──Stream.Read(buffer)──► chương trình ──Write──► đích

Copy: source.CopyToAsync(destination)

File lớn: đọc từng chunk 8K–80K, không load hết
```

## 5. Cú pháp

Mở FileStream async và copy:

```csharp
await using var src = new FileStream(path, FileMode.Open, FileAccess.Read, FileShare.Read, 4096, FileOptions.Asynchronous);
await using var dst = new FileStream(dest, FileMode.Create, FileAccess.Write, FileShare.None, 4096, FileOptions.Asynchronous);
await src.CopyToAsync(dst);
```

MemoryStream từ bytes:

```csharp
byte[] data = "hello"u8.ToArray();
await using var ms = new MemoryStream(data);
byte[] buffer = new byte[2];
int read = await ms.ReadAsync(buffer);
```

## 6. Ví dụ

### Cơ bản

Ghi vài byte vào file:

```csharp
await using var fs = new FileStream("a.bin", FileMode.Create, FileAccess.Write);
byte[] bytes = { 1, 2, 3, 4 };
await fs.WriteAsync(bytes);
```

### Trung cấp

Đọc bằng buffer thủ công:

```csharp
static async Task HexDumpAsync(string path, CancellationToken ct = default)
{
    await using var fs = new FileStream(path, FileMode.Open, FileAccess.Read, FileShare.Read, 4096, true);
    byte[] buffer = new byte[16];
    int n;
    while ((n = await fs.ReadAsync(buffer.AsMemory(0, buffer.Length), ct)) > 0)
    {
        Console.WriteLine(Convert.ToHexString(buffer.AsSpan(0, n)));
    }
}
```

### Nâng cao

MemoryStream làm trung gian serialize:

```csharp
await using var ms = new MemoryStream();
await using (var writer = new StreamWriter(ms, leaveOpen: true))
{
    await writer.WriteAsync("payload");
    await writer.FlushAsync();
}
ms.Position = 0;
using var reader = new StreamReader(ms);
Console.WriteLine(await reader.ReadToEndAsync());
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| File bị khóa | Quên dispose | `await using` |
| `ObjectDisposedException` | Dùng sau using | Scope rõ |
| Read trả 0 | EOF | Thoát vòng |
| Seek trên network | CanSeek=false | Không Rely Position |
| Sync Read trên UI | Block | ReadAsync |

## 8. Gỡ lỗi

1. Kiểm `Length` / `Position` trên FileStream.
2. Log số byte `ReadAsync` mỗi vòng.
3. Process Explorer / lsof nếu file locked.

## 9. Best practices

- Buffer 4K–80K tùy workload.
- `FileOptions.Asynchronous` / constructor `useAsync: true`.
- `leaveOpen: true` khi bọc Stream mà vẫn cần dùng outer.
- Không giữ FileStream lâu hơn cần — lock file.
- Prefer `CopyToAsync` thay vòng tự viết trừ khi cần progress từng chunk.

## 10. Bài tập

**Bài 1** — Copy file bằng `CopyToAsync`.

**Bài 2** — Copy thủ công buffer 81920, đếm tổng byte.

**Bài 3** — Ghi 1MB vào MemoryStream, `ToArray`, kiểm Length.

**Bài 4** — So sánh thời gian `File.ReadAllBytes` vs stream copy file ~10MB (tự tạo).

## 11. Gợi ý

- Bài 1: mục 5.
- Bài 2: while ReadAsync + WriteAsync + `total += n`.
- Bài 3: `ms.SetLength` hoặc Write vòng.
- Bài 4: `Stopwatch` + file temp.

## 12. Đáp án

**Bài 1** — CopyToAsync:

```csharp
await using var src = File.OpenRead("in.bin");
await using var dst = File.Create("out.bin");
await src.CopyToAsync(dst);
```

**Bài 2** — Buffer thủ công:

```csharp
static async Task<long> CopyCountAsync(string from, string to)
{
    await using var src = File.OpenRead(from);
    await using var dst = File.Create(to);
    byte[] buffer = new byte[81920];
    long total = 0;
    int n;
    while ((n = await src.ReadAsync(buffer)) > 0)
    {
        await dst.WriteAsync(buffer.AsMemory(0, n));
        total += n;
    }
    return total;
}
```

**Bài 3** — 1MB MemoryStream:

```csharp
await using var ms = new MemoryStream();
byte[] chunk = new byte[1024];
for (int i = 0; i < 1024; i++) await ms.WriteAsync(chunk);
Console.WriteLine(ms.Length); // 1048576
```

**Bài 4** — Tự đo bằng Stopwatch (viết 10MB rồi so 2 cách đọc).

## 13. Đáp án thay thế

`File.Open` với `FileStreamOptions` (.NET 6+) cấu hình `Options = FileOptions.Asynchronous`, `BufferSize`, `Access`, `Mode`.

## 14. Thử thách

Copy có progress: mỗi 1MB in phần trăm (cần biết `src.Length`).

## 15. Ứng dụng thực tế

- Upload/download pipeline
- Hash file (đọc stream vào incremental hash)
- Zip / crypto stream chaining (`CryptoStream`, `GZipStream`)

## 16. Liên hệ Unity

- Load texture/raw từ persistent path bằng FileStream trên worker — tạo Texture trên main.
- Resources/`StreamingAssets` trên Android có thể không phải file thuần — dùng `UnityWebRequest` / API Unity.
- Tránh đọc sync file lớn trong `Update`.

## 17. Kiểm tra kiến thức

1. Stream là gì?  
   **Đáp án:** Trừu tượng đọc/ghi chuỗi byte.

2. MemoryStream khác FileStream?  
   **Đáp án:** Dữ liệu nằm RAM vs đĩa.

3. `ReadAsync` trả 0 nghĩa là gì?  
   **Đáp án:** Hết dữ liệu (EOF).

4. Vì sao `await using` quan trọng?  
   **Đáp án:** Giải phóng handle file kịp thời.

5. Mọi Stream đều Seek được?  
   **Đáp án:** Không — kiểm `CanSeek`.
