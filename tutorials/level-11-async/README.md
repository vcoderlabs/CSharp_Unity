# Level 11 — Async / Await / Multithreading (~30 giờ)

Level này dạy bạn viết code **không chặn thread** với `async`/`await`, điều phối nhiều `Task`, hủy bằng `CancellationToken`, song song với `Parallel`, và **đồng bộ hóa** an toàn (`lock`, concurrent collections) để tránh race/deadlock.

**Điều kiện:** Đã hoàn thành [Level 10 — Memory](../level-10-memory/) (hiểu heap/stack giúp lý giải `ValueTask` và continuation). Nên vững Level 6 (exceptions) và Level 7 (delegates).

**Tiếp theo:** [Level 12 — File / IO / Serialization](../level-12-file-io/) (IO bất đồng bộ + JSON).

---

## Mục tiêu cấp độ

Sau Level 11 bạn sẽ:

- Phân biệt **Thread** vs **Task** và khi nào dùng cái nào
- Giải thích `async`/`await` qua **state machine** (không chỉ “magic keyword”)
- Dùng `Task` / `Task<T>` / `ValueTask`, `CancellationToken`, `WhenAll` / `WhenAny`, `Parallel`
- Đồng bộ bằng `lock` / `Monitor` / `Mutex` / `Semaphore(Slim)`
- Nhận diện race condition, deadlock, và **SynchronizationContext** (đặc biệt Unity)
- Xây **Async Download Manager** (console .NET 8+)

---

## Cảnh báo xuyên suốt Level 11

> **`async void` hầu như luôn sai** (trừ event handler UI).  
> Prefer `async Task` / `async Task<T>`. Đừng `.Result` / `.Wait()` trên UI/main thread — dễ **deadlock**. Trong Unity: đừng `await` rồi cập nhật scene từ thread pool mà không marshal về main thread.

---

## 7 chương + 1 project

| # | File | Nội dung | ~Giờ |
|---|------|----------|------|
| 1 | [01-thread-vs-task.md](./01-thread-vs-task.md) | Thread vs Task, ThreadPool | 3–4 |
| 2 | [02-async-await.md](./02-async-await.md) | `async`/`await`, state machine | 4–5 |
| 3 | [03-task-valuetask.md](./03-task-valuetask.md) | `Task<T>`, `ValueTask` | 3–4 |
| 4 | [04-cancellation.md](./04-cancellation.md) | `CancellationToken` | 2–3 |
| 5 | [05-whenall-whenany-parallel.md](./05-whenall-whenany-parallel.md) | `WhenAll`/`WhenAny`, `Parallel` | 3–4 |
| 6 | [06-synchronization.md](./06-synchronization.md) | `lock`, Monitor, Mutex, Semaphore | 3–4 |
| 7 | [07-concurrent-race-deadlock.md](./07-concurrent-race-deadlock.md) | Concurrent collections, race, deadlock, sync context | 3–4 |
| — | [project-async-download-manager.md](./project-async-download-manager.md) | Async Download Manager | 5–6 |

**Tổng ước lượng: ~30 giờ**

---

## Cách học đề xuất

1. Chương 1: chạy `Thread.Sleep` vs `await Task.Delay` — cảm nhận sự khác biệt.
2. Chương 2: đặt breakpoint trong method `async` — quan sát continuation sau `await`.
3. Chương 3–4: luôn truyền `CancellationToken` xuống IO giả lập.
4. Chương 5: so sánh tuần tự vs `WhenAll` bằng đồng hồ.
5. Chương 6–7: cố tình tạo race rồi sửa bằng `lock` / `ConcurrentDictionary`.
6. Project: download song song + progress + cancel bằng Ctrl+C.

---

## Checklist hoàn thành Level 11

- [ ] Giải thích được Thread vs Task bằng ví dụ
- [ ] Viết được `async Task` / `async Task<T>` đúng idiom
- [ ] Biết khi nào cân nhắc `ValueTask`
- [ ] Hủy được thao tác bằng `CancellationToken`
- [ ] Dùng `WhenAll` / `WhenAny` và `Parallel` đúng ngữ cảnh
- [ ] Dùng `lock` / `SemaphoreSlim` an toàn
- [ ] Nhận diện race/deadlock và SyncContext (Unity)
- [ ] Hoàn thành Async Download Manager
- [ ] Trả lời đúng ≥ 4/5 câu **Kiểm tra kiến thức** mỗi chương

Khi xong checklist → chuyển sang **Level 12**.
