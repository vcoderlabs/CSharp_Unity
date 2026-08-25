# Chương 8 — Addressables

## 1. Mục tiêu học

- Hiểu vấn đề **Resources/** và build size
- Khái niệm Addressables: address, group, catalog, handle
- Load/unload async an toàn
- Biết remote content (ý tưởng CDN)

## 2. Điều kiện tiên quyết

- Async chương 4
- Prefab / SO chương 2

## 3. Khái niệm

**Addressables:** hệ thống quản lý asset bằng địa chỉ logic, hỗ trợ đóng gói, dependency, remote download, memory release có kiểm soát.

| Khái niệm | Ý nghĩa |
|-----------|---------|
| Address | Chuỗi/key để load |
| Group | Bundling & profile build |
| Catalog | Mục lục map address → location |
| Handle | Tham chiếu load — cần Release |
| Labels | Load theo nhóm label |

So với `Resources.Load`: Addressables scale tốt hơn cho game lớn / patch nội dung.

## 4. Mô hình tư duy

```text
Designer đánh dấu asset Addressable + address "Hero/Warrior"
Runtime: Addressables.LoadAssetAsync<GameObject>("Hero/Warrior")
       → await / callback
       → Instantiate
       → xong: Addressables.Release(handle) / ReleaseInstance
```

## 5. Cú pháp

Cài package **Addressables** từ Package Manager. Window → Asset Management → Addressables.

```csharp
using UnityEngine.AddressableAssets;
using UnityEngine.ResourceManagement.AsyncOperations;

async Awaitable<GameObject> SpawnAsync(string key, Vector3 pos)
{
    var handle = Addressables.LoadAssetAsync<GameObject>(key);
    await handle.Task;
    if (handle.Status != AsyncOperationStatus.Succeeded)
        throw new System.Exception($"Load fail {key}");

    GameObject instance = Instantiate(handle.Result, pos, Quaternion.identity);
    // Giữ handle để Release sau; hoặc dùng InstantiateAsync
    return instance;
}
```

`Addressables.InstantiateAsync(key)` tiện hơn cho GO — nhớ `ReleaseInstance`.

## 6. Ví dụ

### Cơ bản — load Sprite UI

Load icon theo id item → gán Image → Release khi đóng panel.

### Trung cấp — label

```csharp
Addressables.LoadAssetsAsync<GameObject>("Enemies", null);
```

### Nâng cao — remote

Profile: Local vs Remote URL CDN. Catalog update khi patch. (Cấu hình Editor — thực hành theo docs Unity version bạn dùng.)

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Memory tăng mãi | Không Release | Pair load/release |
| Key not found | Sai address / chưa build | Check Groups + Play Mode Script |
| Load chậm lần đầu | Download / IO | Loading screen + preload |
| Dependency thiếu | Group sai | Analyze window |

## 8. Gỡ lỗi

1. Addressables Event Viewer.  
2. Play Mode Script: Use Asset Database vs Simulate Groups.  
3. Log handle.Status + OperationException.  
4. Build Player Content trước khi test bundle mode.

## 9. Best practices

- Đặt convention address (`ui/icons/sword`).  
- Preload critical; lazy phần còn lại.  
- Release khi đổi scene / đóng UI.  
- Không trộn Resources + Addressables loạn xạ.  
- Đo memory trước/sau load dump.

## 10. Bài tập

**Bài 1** — Đánh dấu 1 prefab Addressable, load + instantiate bằng key.  
**Bài 2** — Release đúng lúc destroy.  
**Bài 3** — Load SO stats bằng Addressables.  
**Bài 4** — Loading UI: spinner đến khi handle xong.

## 11. Gợi ý

- Bài 1: Groups window → tick Addressable.  
- Bài 2: `ReleaseInstance` nếu InstantiateAsync.

## 12. Đáp án

```csharp
var handle = Addressables.InstantiateAsync("Enemies/Goblin", pos, rot);
await handle.Task;
GameObject go = handle.Result;
// later:
Addressables.ReleaseInstance(go);
```

## 13. Đáp án thay thế

AssetBundle thủ công (cũ hơn). Content Delivery riêng studio.

## 14. Thử thách

Catalog remote giả lập: đổi profile Remote, host thư mục build bằng http-server local.

## 15. Ứng dụng thực tế

MMORPG: skin, map chunk, event seasonal download — không nhét hết APK.

## 16. Liên hệ C# thuần

Giống plugin loader + package manifest. Handle ≈ `IDisposable` resource lifetime.

## 17. Kiểm tra kiến thức

1. Addressables giải quyết vấn đề gì so với Resources?  
   **Đáp án:** Quản lý bundle/remote/dependency/memory có kiểm soát hơn khi scale.

2. Vì sao phải Release?  
   **Đáp án:** Giảm refcount để unload asset khỏi memory.

3. Address là gì?  
   **Đáp án:** Key logic để tìm asset lúc runtime.

4. InstantiateAsync khác LoadAssetAsync?  
   **Đáp án:** InstantiateAsync tạo instance GO và gắn lifetime Addressables helper.

5. Catalog dùng để?  
   **Đáp án:** Ánh xạ address → vị trí file/bundle (local/remote).
