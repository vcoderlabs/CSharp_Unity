# Milestone 07 — Persistence

## Requirements

- **Save / Load** local: inventory + quest progress + profile cache tối thiểu
- Dùng **File I/O** (`Application.persistentDataPath`)
- Serialization JSON rõ ràng (DTO save riêng — không dump MB)
- **Versioning**: field `Version` / `SchemaVersion`; có đường migrate v1→v2 giả định
- Autosave khi quan trọng (quest complete / inventory change debounced) + nút Save thủ công
- Load khi boot (sau login hoặc offline mode)

**Không yêu cầu:** cloud save mã hóa enterprise, anti-cheat.

---

## Architecture

```text
ISaveService
  Task SaveAsync(CancellationToken)
  Task<bool> LoadAsync(CancellationToken)

GameSaveDto
  int SchemaVersion
  string PlayerId
  List<SlotDto> Inventory
  List<QuestProgressDto> Quests
  long SavedAtUnix

SaveService
  ├── IInventoryExporter
  ├── IQuestExporter
  └── IFileGateway (File.WriteAllTextAsync)

Migrators:
  ISaveMigrator { int From, To; GameSaveDto Migrate(GameSaveDto) }
  Pipeline: while version < current → apply

Boot:
  Session → LoadAsync → hydrate services → Playing
```

```text
persistentDataPath/saves/slot1.json
```

---

## Tasks

1. Định nghĩa `GameSaveDto` + schema version = 1.  
2. Export/import inventory & quests.  
3. `SaveService` ghi/đọc async.  
4. Debounce autosave 1–2s sau event Changed.  
5. Giả lập v2: thêm field `Settings.MasterVolume`; migrator set default.  
6. Test: save → đổi memory → load lại khôi phục.  
7. Document đường dẫn file trên Windows/macOS trong README.

---

## Expected result

- Thoát Play (Editor) vẫn mất — **đúng**; nhưng Stop rồi Play lại **có** load file nếu bạn không xóa save (Editor domain reload — kiểm tra `Enter Play Mode Options`; hoặc Save trong Play và đọc file bằng tay).  
- Khuyến nghị: nút Save + Load trong UI để verify rõ.  
- File JSON đọc được, có `SchemaVersion`.  
- Migrator chạy khi mở file v1 trên code v2.

---

## Exercises

**E1** — 3 save slots.  
**E2** — Checksum/hash đơn giản phát hiện file truncat.  
**E3** — Backup `slot1.bak` trước khi ghi.  
**E4** — Không save token dạng plain nếu có (redact).

---

## Hints

- Editor: persistentDataPath vẫn dùng được.  
- Debounce: `float _timer` trong tick hoặc `CancellationTokenSource` delay.  
- STJ: `[JsonPropertyName]` ổn định.  
- Hydrate: clear inventory rồi Add từ DTO — tránh duplicate.

---

## Solution outline

```csharp
public sealed class GameSaveDto
{
    public int SchemaVersion { get; set; } = 1;
    public string PlayerId { get; set; }
    public List<SlotDto> Inventory { get; set; } = new();
    public List<QuestProgressDto> Quests { get; set; } = new();
}

public sealed class SaveService : ISaveService
{
    public async Task SaveAsync(CancellationToken ct)
    {
        var dto = _exporter.Export();
        var json = JsonSerializer.Serialize(dto, _opts);
        var path = Path.Combine(Application.persistentDataPath, "saves/slot1.json");
        Directory.CreateDirectory(Path.GetDirectoryName(path)!);
        await File.WriteAllTextAsync(path, json, ct);
    }
}
```

Migrator v1→v2: `dto.SchemaVersion = 2; dto.Settings ??= new() { MasterVolume = 1f };`

---

## Code review checklist

- [ ] DTO tách — không serialize UnityEngine.Object  
- [ ] SchemaVersion bắt buộc  
- [ ] Có migrator hoặc policy “xóa save cũ” được document  
- [ ] Autosave không ghi mỗi frame  
- [ ] Lỗi I/O surface (log + UI)  
- [ ] Token/secret không nằm save  
- [ ] Test hoặc checklist thủ công Save/Load  
- [ ] ARCHITECTURE persistence  
