# Chương 1 — File, Directory, Path

## 1. Mục tiêu học

- Ghép và chuẩn hóa đường dẫn bằng `Path`
- Tạo / xóa / di chuyển file và thư mục
- Liệt kê file an toàn (`EnumerateFiles`)
- Phân biệt absolute vs relative path

## 2. Điều kiện tiên quyết

- Level 1: string, exception cơ bản
- Level 6: try/catch với IOException
- .NET 8+

## 3. Khái niệm

| API | Việc làm |
|-----|----------|
| `Path` | Combine, GetExtension, GetFileName, GetTempPath… |
| `File` | Exists, ReadAllText, Copy, Delete ( tiện static) |
| `Directory` | CreateDirectory, Exists, Delete, Enumerate* |
| `FileInfo` / `DirectoryInfo` | Object-oriented, metadata (Length, LastWriteTime) |

`File.ReadAllText` tiện nhưng tải **cả file vào RAM**. File lớn → Stream (chương 2).

## 4. Mô hình tư duy

```text
Path.Combine(root, "data", "a.json")  →  đường dẫn đúng OS
Directory.CreateDirectory(dir)        →  tạo cả chuỗi thư mục cha
EnumerateFiles                        →  lazy, tốt hơn GetFiles khi nhiều
```

Luôn xác định **working directory** (`Environment.CurrentDirectory`) khi dùng relative path.

## 5. Cú pháp

Các thao tác Path / File / Directory thường dùng:

```csharp
string dir = Path.Combine(Path.GetTempPath(), "csharp-l12");
Directory.CreateDirectory(dir);

string file = Path.Combine(dir, "note.txt");
await File.WriteAllTextAsync(file, "hello");
bool exists = File.Exists(file);
string text = await File.ReadAllTextAsync(file);

foreach (string f in Directory.EnumerateFiles(dir, "*.txt"))
    Console.WriteLine(f);

File.Delete(file);
Directory.Delete(dir, recursive: true);
```

## 6. Ví dụ

### Cơ bản

In thông tin path:

```csharp
string p = Path.Combine("data", "player.save");
Console.WriteLine(Path.GetFullPath(p));
Console.WriteLine(Path.GetExtension(p)); // .save
Console.WriteLine(Path.GetFileNameWithoutExtension(p)); // player
```

### Trung cấp

Sao lưu file nếu tồn tại:

```csharp
static void BackupIfExists(string path)
{
    if (!File.Exists(path)) return;
    string bak = path + ".bak";
    File.Copy(path, bak, overwrite: true);
}
```

### Nâng cao

Duyệt đệ quy bỏ thư mục bị cấm:

```csharp
static IEnumerable<string> SafeEnumerate(string root, string pattern)
{
    var pending = new Stack<string>();
    pending.Push(root);
    while (pending.Count > 0)
    {
        string dir = pending.Pop();
        IEnumerable<string> files = Enumerable.Empty<string>();
        try { files = Directory.EnumerateFiles(dir, pattern); }
        catch (UnauthorizedAccessException) { continue; }
        foreach (var f in files) yield return f;
        try
        {
            foreach (var sub in Directory.EnumerateDirectories(dir))
                pending.Push(sub);
        }
        catch (UnauthorizedAccessException) { }
    }
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| File không thấy | Relative path sai cwd | `GetFullPath` / đường dẫn tuyệt đối |
| `DirectoryNotFoundException` | Thiếu thư mục cha | `CreateDirectory` trước khi ghi |
| Xóa không được | File đang mở / quyền | Dispose stream; kiểm tra lock |
| Path inject `..` | User input | Chuẩn hóa + đảm bảo nằm trong root cho phép |

## 8. Gỡ lỗi

1. In `Path.GetFullPath` và `Directory.GetCurrentDirectory()`.
2. Kiểm tra tồn tại từng segment.
3. Catch `IOException` / `UnauthorizedAccessException` riêng.

## 9. Best practices

- `Path.Combine` / `Path.Join` — không hardcode separator.
- Validate user path nằm trong sandbox directory.
- Prefer `Enumerate*` hơn `Get*` khi tập lớn.
- Dùng `async` variants khi trên request thread/server.
- Temp: `Path.GetTempFileName` / subfolder riêng cho bài lab.

## 10. Bài tập

**Bài 1** — Tạo thư mục `./workspace/l12` và file `readme.md` với nội dung tùy ý.

**Bài 2** — Đếm số file `*.md` trong một thư mục (không đệ quy).

**Bài 3** — Đổi tên file bằng `File.Move`.

**Bài 4** — Hàm `EnsureChildPath(root, relative)` ném nếu path thoát khỏi root (`..`).

## 11. Gợi ý

- Bài 1: `CreateDirectory` + `WriteAllTextAsync`.
- Bài 2: `EnumerateFiles(dir, "*.md").Count()`.
- Bài 3: `File.Move(src, dest)`.
- Bài 4: `GetFullPath` cả root và combined; `StartsWith` root.

## 12. Đáp án

**Bài 1** — Tạo workspace:

```csharp
string dir = Path.Combine("workspace", "l12");
Directory.CreateDirectory(dir);
await File.WriteAllTextAsync(Path.Combine(dir, "readme.md"), "# lab\n");
```

**Bài 2** — Đếm md:

```csharp
int count = Directory.EnumerateFiles(dir, "*.md").Count();
```

**Bài 3** — Move:

```csharp
File.Move(Path.Combine(dir, "readme.md"), Path.Combine(dir, "README.md"), overwrite: true);
```

**Bài 4** — Sandbox path:

```csharp
static string EnsureChildPath(string root, string relative)
{
    string fullRoot = Path.GetFullPath(root).TrimEnd(Path.DirectorySeparatorChar) + Path.DirectorySeparatorChar;
    string full = Path.GetFullPath(Path.Combine(root, relative));
    if (!full.StartsWith(fullRoot, StringComparison.OrdinalIgnoreCase) &&
        !string.Equals(full.TrimEnd(Path.DirectorySeparatorChar), fullRoot.TrimEnd(Path.DirectorySeparatorChar), StringComparison.OrdinalIgnoreCase))
        throw new InvalidOperationException("Path escapes root");
    return full;
}
```

## 13. Đáp án thay thế

Bài 4 trên .NET mới có thể dùng `Path.GetRelativePath` và từ chối nếu bắt đầu bằng `..`.

## 14. Thử thách

Viết mini `du`: tính tổng kích thước file đệ quy bằng `FileInfo.Length` — bỏ qua lỗi quyền.

## 15. Ứng dụng thực tế

- Lưu config/save game
- Log rotation (đổi tên file theo ngày)
- Tool migrate tài nguyên

## 16. Liên hệ Unity

- `Application.persistentDataPath` / `streamingAssetsPath` — đừng giả định cwd Editor.
- Mobile: quyền và path khác PC — luôn qua Unity API path.
- Addressables vs file thô: file thô tốt cho save JSON nhỏ.

## 17. Kiểm tra kiến thức

1. Vì sao tránh `dir + "\\" + file`?  
   **Đáp án:** Sai trên Unix; `Path.Combine` xử lý separator.

2. `EnumerateFiles` khác `GetFiles`?  
   **Đáp án:** Enumerate lazy; Get trả mảng đầy đủ ngay.

3. Relative path dựa vào đâu?  
   **Đáp án:** Current working directory (có thể khác chỗ bạn nghĩ).

4. Tạo file trong subdir chưa tồn tại cần gì?  
   **Đáp án:** `Directory.CreateDirectory` trước.

5. User truyền `../../etc/passwd` nguy hiểm thế nào?  
   **Đáp án:** Path traversal — cần sandbox `EnsureChildPath`.
