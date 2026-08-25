# Level 19 — Performance (~15 giờ)

Level này dạy bạn **đo trước khi tối ưu**: allocation, GC pressure, boxing, LINQ/string overhead, độ phức tạp thuật toán, BenchmarkDotNet, caching và pooling — áp dụng cho app production và chuẩn bị cho Unity.

**Điều kiện:** Đã hoàn thành [Level 10 — Memory](../level-10-memory/) (GC, pooling) và nên có [Level 8 — LINQ](../level-08-linq/). Nên đã biết async cơ bản (L11).

**Song song / tiếp theo:** [Level 20 — Professional C#](../level-20-professional/) — project chung **Production-grade app**. Sau đó [Level 21 — Unity](../level-21-unity/).

---

## Mục tiêu cấp độ

Sau Level 19 bạn sẽ:

- Nhận diện nguồn allocation và GC pressure trong hot path
- Tránh boxing / LINQ / string allocation không cần thiết khi performance quan trọng
- Ước lượng Big-O và chọn cấu trúc dữ liệu phù hợp
- Đo bằng profiler và **BenchmarkDotNet** (không đoán)
- Áp dụng caching và object pooling có kiểm soát
- Đóng góp phần tối ưu vào project chung L19–20

---

## Cảnh báo xuyên suốt Level 19

> **Không tối ưu sớm.** Đo → tìm hot path → sửa → đo lại.  
> Micro-optimization làm code khó đọc mà không có số liệu = lãng phí.  
> Trong Unity, GC spike = frame hitch — nhưng nguyên tắc “đo trước” vẫn giữ nguyên.

---

## 5 chương + project chung L19–20

| # | File | Nội dung | ~Giờ |
|---|------|----------|------|
| 1 | [01-allocation-gc-pressure.md](./01-allocation-gc-pressure.md) | Chi phí allocation, GC pressure | 3 |
| 2 | [02-boxing-linq-string.md](./02-boxing-linq-string.md) | Boxing, LINQ, string allocation | 3 |
| 3 | [03-algorithm-complexity.md](./03-algorithm-complexity.md) | Complexity thực chiến | 2–3 |
| 4 | [04-profiling-benchmarkdotnet.md](./04-profiling-benchmarkdotnet.md) | Profiling & BenchmarkDotNet | 3 |
| 5 | [05-caching-pooling.md](./05-caching-pooling.md) | Caching & pooling | 2–3 |
| — | [project-production-app.md](../level-20-professional/project-production-app.md) | App production (chung L19–20) | chia sẻ với L20 |

**Tổng ước lượng Level 19 (lý thuyết + bài tập): ~15 giờ**  
Phần project production làm tiếp / song song ở Level 20.

---

## Cách học đề xuất

1. Chương 1–2: viết micro-demo “trước/sau”, quan sát Gen0 bằng `GC.CollectionCount`.
2. Chương 3: vẽ Big-O lên bài toán thật (inventory lookup, pathfind giả lập).
3. Chương 4: cài BenchmarkDotNet, so sánh 2–3 phiên bản cùng API.
4. Chương 5: cache có TTL/size limit; pool có `Get`/`Return` rõ ràng.
5. Project L19–20: chọn 1 hot path trong app, benchmark, tối ưu có PR/ghi chú số liệu.

---

## Checklist hoàn thành Level 19

- [ ] Giải thích được allocation vs GC pressure
- [ ] Chỉ ra 3 nguồn boxing / LINQ / string thường gặp
- [ ] Chọn đúng cấu trúc dữ liệu theo complexity
- [ ] Chạy được ít nhất 1 benchmark BenchmarkDotNet
- [ ] Implement cache hoặc pool có giới hạn tài nguyên
- [ ] Trả lời đúng ≥ 4/5 câu **Kiểm tra kiến thức** mỗi chương
- [ ] Đóng góp đo/tối ưu vào [project production](../level-20-professional/project-production-app.md)

Khi xong → tiếp tục **Level 20** (nếu chưa) hoặc **Level 21**.
