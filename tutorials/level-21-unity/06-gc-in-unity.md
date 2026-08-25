# Chương 6 — Garbage Collection trong Unity

## 1. Mục tiêu học

- Hiểu GC spike gây **frame hitch**
- Đọc cột **GC Alloc** trên Profiler
- Nhận diện allocation Unity thường gặp
- Áp dụng chiến lược zero-alloc trên hot path

## 2. Điều kiện tiên quyết

- Level 10 + Level 19 allocation
- Profiler Unity cơ bản

## 3. Khái niệm

Unity dùng Boehm / Incremental GC (tùy version/setting) trên managed heap. Khi GC chạy nặng → frame time tăng đột biến.

**Incremental GC** trải pause — vẫn cần giảm alloc.

Nguồn alloc điển hình trong game loop:

- `GetComponent` / `Find` / LINQ  
- string / `$` / `ToString`  
- boxing  
- `new` class mỗi frame  
- `GetComponents` mảng mới  
- foreach trên `IEnumerable` tùy loại  
- closure lambda trong Update  

## 4. Mô hình tư duy

```text
Profiler: CPU Usage → Timeline → GC.Alloc spikes
Đứng frame nghi ngờ → xem script nào alloc
Sửa → Play lại đo
Mục tiêu mobile: gần 0 B/frame ở gameplay ổn định
```

## 5. Cú pháp

```csharp
// Bad
void Update()
{
    Debug.Log($"HP {_hp}"); // alloc string gần như mỗi frame nếu luôn gọi
    var enemies = FindObjectsOfType<Enemy>(); // rất đắt + alloc
}

// Better
void Update()
{
    // chỉ log khi dirty / editor
    _cachedMotor.Tick(Time.deltaTime);
}
```

Player Settings → GC Incremental: bật khi phù hợp; vẫn giảm alloc.

## 6. Ví dụ

### Cơ bản — đo

Profiler module **GC Alloc**, deep profile tạm thời (chậm).

### Trung cấp — cache

```csharp
Colliders[] _results = new Colliders[16];
int count = Physics.OverlapSphereNonAlloc(pos, r, _results);
```

### Nâng cao — không alloc UI rebuild

Rebuild text chỉ khi giá trị đổi; dùng `StringBuilder` / ZString / UI Toolkit đúng cách.

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Chỉ đo Editor | Khác Player | Development Player build |
| Deep Profile luôn bật | Làm chậm sai lệch | Bật khi cần |
| Tối ưu chỗ không alloc | Đo sai | Sort theo GC.Alloc |
| Empty event invoke? | Có thể OK | Đo thực tế |

## 8. Gỡ lỗi

1. Frame Debugger ≠ GC — dùng Profiler.  
2. Show Related Objects.  
3. Memory Profiler package cho leak lâu dài (native + managed).  
4. So sánh 2 build có/không thay đổi.

## 9. Best practices

- Hot path: không string, không LINQ, không Find.  
- NonAlloc physics APIs.  
- Pool thay Instantiate/Destroy spam.  
- Struct + job khi dùng DOTS (nâng cao).  
- Budget: ghi rõ “gameplay 0–2KB/frame max” team.

## 10. Bài tập

**Bài 1** — Script cố ý alloc trong Update; chụp Profiler.  
**Bài 2** — Sửa về 0 alloc; chụp lại.  
**Bài 3** — Thay `OverlapSphere` bằng NonAlloc.  
**Bài 4** — Liệt kê 10 anti-pattern alloc Unity.

## 11. Gợi ý

- Bài 1: `new byte[100]` hoặc string.  
- Bài 4: gồm GetComponents mỗi frame.

## 12. Đáp án

**Bài 4 mẫu:** FindObjectsOfType mỗi frame; LINQ; string concat; boxing enum Dictionary cũ; Instantiate đạn; foreach Regex; params object log; box trong UnityEvent đôi khi; `Camera.main` mỗi frame (lookup); tạo material mới mỗi lần (leak + alloc).

## 13. Đáp án thay thế

Burst + Jobs giảm managed alloc mạnh (đường học riêng).

## 14. Thử thách

Đạt 0 B/frame trong scene combat giả lập 100 viên đạn (dùng pool chương 7).

## 15. Ứng dụng thực tế

Mobile MMORPG: GC spike = 1 sao store. Console cũng nhạy cảm frame time.

## 16. Liên hệ C# thuần

Cùng CLR ý tưởng Gen; Unity thêm ràng buộc 16.6ms/frame (60FPS).

## 17. Kiểm tra kiến thức

1. GC spike là gì?  
   **Đáp án:** Pause/thu gom làm frame dài bất thường.

2. Công cụ xem alloc/frame?  
   **Đáp án:** Unity Profiler — GC Alloc.

3. `Camera.main` mỗi Update sao xấu?  
   **Đáp án:** Tìm theo tag mỗi lần — đắt; cache.

4. Incremental GC thay thế việc giảm alloc?  
   **Đáp án:** Không — chỉ làm mượt hơn; vẫn cần giảm.

5. NonAlloc API giúp gì?  
   **Đáp án:** Ghi vào buffer có sẵn — tránh mảng mới mỗi lần gọi.
