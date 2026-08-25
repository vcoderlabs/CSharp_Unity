# Chương 4 — Profiling & BenchmarkDotNet

## 1. Mục tiêu học

- Phân biệt **profiling** (tìm hot spot hệ thống) và **benchmarking** (so micro API)
- Dùng **BenchmarkDotNet** với `MemoryDiagnoser`
- Tránh sai số: warm-up, release, anti-optimization
- Đọc kết quả Mean, Error, Allocated

## 2. Điều kiện tiên quyết

- .NET SDK, biết tạo class library / console
- Level 19 chương 1–3: biết mình muốn đo gì

## 3. Khái niệm

| Công cụ | Mục đích |
|---------|----------|
| Profiler (VS, dotnet-trace, PerfView) | “App chậm chỗ nào?” CPU, alloc, wait |
| BenchmarkDotNet | “Hàm A vs B nhanh/alloc thế nào?” lặp nhiều lần, thống kê |
| Load test | Throughput hệ thống dưới concurrent users |

**Release** (`-c Release`): debug bỏ tối ưu → số liệu sai.

BenchmarkDotNet chạy nhiều iteration, warm-up, có thể ngăn JIT/GC làm lệch (cấu hình).

## 4. Mô hình tư duy

```text
1. User/scenario chậm
2. Profile → 80% thời gian ở Method X
3. Viết benchmark X_v1 vs X_v2
4. Đổi code nếu số liệu thắng + không phá correctness
5. Profile lại end-to-end
```

## 5. Cú pháp

```bash
dotnet new console -n PerfLab
cd PerfLab
dotnet add package BenchmarkDotNet
```

```csharp
using BenchmarkDotNet.Attributes;
using BenchmarkDotNet.Running;

[MemoryDiagnoser]
public class SumBench
{
    private int[] _data = default!;

    [GlobalSetup]
    public void Setup() => _data = Enumerable.Range(0, 10_000).ToArray();

    [Benchmark(Baseline = true)]
    public int ForLoop()
    {
        int s = 0;
        for (int i = 0; i < _data.Length; i++) s += _data[i];
        return s;
    }

    [Benchmark]
    public int LinqSum() => _data.Sum();
}

public static class Program
{
    public static void Main(string[] args)
        => BenchmarkRunner.Run<SumBench>();
}
```

Chạy: `dotnet run -c Release`

## 6. Ví dụ

### Cơ bản — một class benchmark

Như trên: Baseline + MemoryDiagnoser.

### Trung cấp — Params

```csharp
[Params(100, 10_000, 1_000_000)]
public int N;

[GlobalSetup]
public void Setup() => _data = new int[N];
```

### Nâng cao — đọc bảng

```text
| Method  | Mean     | Error    | Allocated |
| ForLoop | 2.1 us   | 0.01 us  | -         |
| LinqSum | 15 us    | 0.2 us   | 32 B      |
```

- **Mean**: trung bình thời gian
- **Allocated**: heap alloc / operation (ước lượng)
- So sánh Ratio vs Baseline

Tránh micro-bench quá nhỏ đến mức không meaningful (đo nhiễu).

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Debug build | Không tối ưu | `-c Release` |
| Dead code eliminated | JIT bỏ tính toán không dùng | Trả về / `Consumer.Consume` |
| Đo cả I/O | Network/disk nhiễu | Tách pure CPU bench |
| Một lần Stopwatch | Không ổn định | BenchmarkDotNet |
| So máy khác nhau | Nhiệt, load | Ghi chú môi trường |

## 8. Gỡ lỗi

1. Kết quả phân tán (Error lớn): tăng iteration, đóng app nền.
2. Alloc bất ngờ: xem có boxing, display class, params không?
3. Profile sampling vs instrumentation: sampling nhẹ hơn; instrumentation chi tiết hơn.
4. `dotnet-trace collect --process-id ...` rồi mở trong PerfView / Speedscope.

## 9. Best practices

- Benchmark **cùng kích thước dữ liệu production**.
- Tên method rõ: `Parse_Span` vs `Parse_Split`.
- Commit số liệu tóm tắt trong PR performance.
- Không tối ưu 0.1% nếu làm API khó dùng — trừ khi hot path đã chứng minh.
- Tách correctness test (xUnit) khỏi benchmark project.

## 10. Bài tập

**Bài 1** — Tạo project BenchmarkDotNet, so `string.StartsWith` vs `span.StartsWith` trên chuỗi dài.

**Bài 2** — Benchmark `List.Add` không capacity vs có capacity (N=100k).

**Bài 3** — Thêm `[MemoryDiagnoser]`, giải thích cột Allocated của LINQ `Where.ToList` vs loop.

**Bài 4** — Dùng Stopwatch sai cách một lần vs BDN — viết đoạn văn ngắn khác biệt.

## 11. Gợi ý

- Bài 1: `ReadOnlySpan<char>` từ string.
- Bài 2: `[GlobalSetup]` tạo N.
- Bài 3: LINQ thường > 0 B.
- Bài 4: cold start, GC, thiếu warm-up.

## 12. Đáp án

**Bài 2** (ý):

```csharp
[Benchmark]
public List<int> NoCap()
{
    var list = new List<int>();
    for (int i = 0; i < N; i++) list.Add(i);
    return list;
}

[Benchmark]
public List<int> WithCap()
{
    var list = new List<int>(N);
    for (int i = 0; i < N; i++) list.Add(i);
    return list;
}
```

Expect: WithCap nhanh hơn / ít alloc hơn (ít grow).

**Bài 4** — Stopwatch một lần dễ bị nhiễu; BDN lặp, thống kê, warm-up, cấu hình chuẩn.

**Bài 1/3** — Học viên tự chạy và dán bảng vào ghi chú.

## 13. Đáp án thay thế

Dùng `BenchmarkDotNet.Diagnostics.Windows` (ETW) trên Windows. Hoặc DisassemblyDiagnoser xem codegen.

## 14. Thử thách

Benchmark JSON: `System.Text.Json` serialize class vs `record struct` source-gen — báo cáo Mean + Allocated.

## 15. Ứng dụng thực tế

- SLA API: p99 latency — profile + load test
- Game server tick budget (ví dụ 50ms) — tìm system vượt budget
- Regression: CI chạy subset benchmark (cẩn thận flaky)

## 16. Liên hệ Unity

- Unity Profiler ≠ BDN nhưng cùng tư duy “đo”
- Profile Editor vs Player khác nhau — luôn đo build gần shipping
- Frame Timing, GC.Alloc module
- Không tin FPS trong Editor là đủ

## 17. Kiểm tra kiến thức

1. Profiler dùng khi nào?  
   **Đáp án:** Tìm chỗ chậm/alloc trong hệ thống thật.

2. BenchmarkDotNet dùng khi nào?  
   **Đáp án:** So sánh có kiểm soát các cách triển khai nhỏ.

3. Vì sao cần Release?  
   **Đáp án:** Debug tắt nhiều tối ưu → số liệu không đại diện.

4. `MemoryDiagnoser` cho biết gì?  
   **Đáp án:** Ước lượng heap allocation mỗi lần chạy benchmark.

5. Vì sao trả về kết quả từ method benchmark?  
   **Đáp án:** Tránh JIT loại bỏ tính toán như dead code.
