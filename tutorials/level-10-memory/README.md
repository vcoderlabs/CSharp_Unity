# Level 10 — Memory Management (~20 giờ)

Level này dạy bạn **cách CLR quản lý bộ nhớ**, **Garbage Collector (GC)**, vòng đời object, **`IDisposable` / Dispose pattern**, **memory leak** (đặc biệt event leak), và **object pooling** — kỹ năng sống còn khi làm game Unity.

**Điều kiện:** Đã hoàn thành [Level 3 — Value vs Reference](../level-03-value-vs-reference/) (stack/heap, boxing) và nên biết [Level 7 — Delegates/Events](../level-07-delegates-events/) trước chương event leak. .NET 8+.

**Tiếp theo:** [Level 11 — Async](../level-11-async/).

---

## Cảnh báo xuyên suốt Level 10

> **Cực kỳ quan trọng cho Unity MMORPG.**  
> **GC spike** (GC dừng thế giới một lúc để thu gom) là nguyên nhân hàng đầu gây **giật khung hình / lag**. Mỗi lần bạn `new` trong `Update()`, nối chuỗi, boxing, hay leak event → heap phình → GC chạy → player cảm thấy “giật”.  
> **Không học lướt.** Mỗi chương đều liên hệ lại Unity GC pressure.

```text
Frame N:  Update() new DamageEvent() × 200 enemy
Frame N+1: Gen0 đầy → GC pause 5–20ms → frame drop
           ▲
           └── Đây chính là “GC spike”
```

---

## Mục tiêu cấp độ

Sau Level 10 bạn sẽ:

- Vẽ được mô hình bộ nhớ CLR (stack, heap managed, LOH)
- Giải thích Gen0 / Gen1 / Gen2 và khi nào LOH gây đau
- Biết object sống đến khi nào; finalizer khác Dispose thế nào
- Dùng `IDisposable`, `using` / `using` declaration đúng lúc
- Implement Dispose pattern an toàn
- Phát hiện & sửa memory leak (đặc biệt event subscription)
- Xây **Object Pool** để giảm allocation → giảm GC spike trong Unity

---

## 7 chương + 1 project

| # | File | Nội dung | ~Giờ |
|---|------|----------|------|
| 1 | [01-clr-memory-model.md](./01-clr-memory-model.md) | CLR memory model, stack/heap managed | 2–3 |
| 2 | [02-gc-generations.md](./02-gc-generations.md) | GC Gen0/1/2, LOH, modes | 3–4 |
| 3 | [03-object-lifetime.md](./03-object-lifetime.md) | Vòng đời object, roots, finalizer | 2 |
| 4 | [04-idisposable-using.md](./04-idisposable-using.md) | `IDisposable`, `using` / using declaration | 2–3 |
| 5 | [05-dispose-pattern.md](./05-dispose-pattern.md) | Dispose pattern đầy đủ | 2 |
| 6 | [06-memory-leaks-events.md](./06-memory-leaks-events.md) | Memory leaks, event leaks | 2–3 |
| 7 | [07-object-pooling.md](./07-object-pooling.md) | Object pooling ↔ Unity GC spikes | 2–3 |
| — | [project-object-pooling.md](./project-object-pooling.md) | **Object Pooling System** | 4–5 |

**Tổng ước lượng: ~20 giờ**

---

## Cách học đề xuất

1. Chương 1–2: vẽ ASCII heap/GC trên giấy trước khi code.
2. Chương 3–5: mở Visual Studio / Rider Memory Profiler hoặc `dotnet-counters` xem allocation (console cũng được).
3. Chương 6: cố tình leak event rồi sửa — cảm nhận “object không chết”.
4. Chương 7 + Project: pool bullet/damage text — đo số lần `new` trước/sau.
5. Luôn hỏi: *“Code này chạy trong Unity `Update` 60 lần/giây thì sao?”*

---

## Checklist hoàn thành Level 10

- [ ] Giải thích được stack vs managed heap vs LOH
- [ ] Biết Gen0/1/2 và vì sao object “sống lâu” đắt hơn
- [ ] Dùng `using` / Dispose đúng với resource unmanaged / stream
- [ ] Viết được Dispose pattern chuẩn
- [ ] Unsubscribe event / dùng weak pattern khi cần
- [ ] Implement object pool generic + demo giảm allocation
- [ ] Trả lời đúng ≥ 4/5 câu **Kiểm tra kiến thức** mỗi chương
- [ ] Nói được bằng lời: *GC spike trong Unity đến từ đâu và pool giúp gì*

Khi xong checklist → chuyển sang **Level 11 — Async**.
