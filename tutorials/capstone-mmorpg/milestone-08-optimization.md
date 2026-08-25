# Milestone 08 — Optimization

## Requirements

- **Profile** build Development Player (hoặc Editor có ghi chú hạn chế) trên scene combat đầy đủ
- Liệt kê top GC Alloc / CPU markers
- **Pooling audit**: mọi Instantiate/Destroy vòng combat phải có lý do hoặc chuyển pool
- **GC tuning**: giảm string/LINQ/Find trong hot path; Incremental GC ghi chú
- **Benchmark** (BDN cho pure C# services **hoặc** Unity Performance Testing) ≥ 1 case: inventory add, hoặc damage apply, hoặc save export
- Báo cáo `OPTIMIZATION_REPORT.md`: trước/sau số liệu + quyết định không tối ưu chỗ nào (và vì sao)

**Điều kiện:** M01–M07 chơi được vertical slice.

---

## Architecture

```text
Optimization loop:

  Capture Profiler (baseline)
       │
       ▼
  Classify: CPU bound / GC / IO / UI rebuild
       │
       ▼
  Change (pool / cache / algorithm / dirty UI)
       │
       ▼
  Capture again (compare)
       │
       ▼
  Document + regression checklist

Audit sheet:
  System        | Alloc/frame | Action
  Combat bullets| ...         | pooled
  Quest UI      | ...         | event dirty
  Save debounce | ...         | OK
```

```text
Tools:
  Unity Profiler + Memory Profiler
  optional: BenchmarkDotNet on DLL pure logic
```

---

## Tasks

1. Viết checklist “Play 60s combat + quest + bag open”.  
2. Baseline capture — lưu ảnh/table.  
3. Pool audit toàn solution (grep Instantiate/Destroy).  
4. Sửa tối thiểu 3 hot issues đo được.  
5. Benchmark 1 pure method.  
6. `OPTIMIZATION_REPORT.md`.  
7. Cập nhật ARCHITECTURE mục Performance budget (ví dụ: gameplay < X KB/frame).

---

## Expected result

- Báo cáo có số liệu thật (không chỉ “cảm thấy mượt”).  
- Ít nhất 3 thay đổi có before/after.  
- Nơi **không** sửa được giải thích (đọcability / premature).  
- Vertical slice vẫn chơi đúng sau tối ưu.

---

## Exercises

**E1** — Overlap NonAlloc cho AoE nếu còn alloc.  
**E2** — UI Text chỉ khi score/quest dirty.  
**E3** — Prewarm pools lúc loading screen.  
**E4** — So Incremental GC on/off ghi chú cảm nhận (không kết luận tuyệt đối).

---

## Hints

- Đo Player build khi có thể.  
- Deep Profile đắt — dùng ngắn.  
- Grep `new ` trong Update paths.  
- Đừng tối ưu mock HTTP latency như CPU game — tách IO.

---

## Solution outline

Ví dụ mục tiêu sửa:

1. `Debug.Log` trong hit → strip / conditional.  
2. Damage number `Instantiate` → pool.  
3. Quest UI rebuild mỗi kill → cập nhật 1 row.  
4. `Camera.main` cache.  
5. Inventory `ToList` LINQ → list đệm.

Báo cáo mẫu:

```markdown
## Baseline
- GC Alloc combat idle: ~2.5KB/frame
- Fire spam: spikes 50KB

## Changes
1. Bullet trail reset pooled — spikes → 5KB
2. ...

## Budget
- Target mobile: < 1KB/frame steady combat
```

Benchmark:

```csharp
[MemoryDiagnoser]
public class InventoryBench
{
    [Benchmark] public void AddStack() => _inv.TryAdd(id, 1);
}
```

---

## Code review checklist

- [ ] OPTIMIZATION_REPORT.md tồn tại và có số  
- [ ] ≥ 3 thay đổi đo được  
- [ ] Pool audit không còn Destroy đạn/VFX mỗi hit  
- [ ] Không regress chức năng M03–M07  
- [ ] Budget ghi trong ARCHITECTURE  
- [ ] Benchmark hoặc Performance test checked-in  
- [ ] Không “tối ưu” phá SOLID vô cớ  
- [ ] Ghi môi trường đo (Editor/Player, máy, Unity version)  

---

## Kết thúc Capstone

Khi checklist này đạt → bạn đã hoàn thành lộ trình **From Zero → Unity MMORPG Architecture** ở mức học tập nghiêm túc. Hãy giữ repo làm portfolio: README chơi được, ARCHITECTURE rõ, báo cáo tối ưu có số.
