# Project Level 1 — Console Calculator

Máy tính console hỗ trợ **biểu thức nhiều toán tử**, **lịch sử phép tính**, và **xử lý input sai** — tổng hợp kiến thức Chương 1–10.

**Thời lượng đề xuất:** 6–8 giờ  
**Target:** .NET 8+ console app  
**Độ khó:** Trung cấp (cuối Level 1)

---

## 1. Mục tiêu học

- Ghép biến, kiểu, toán tử, điều khiển luồng, method, string, nullable, utility thành một app hoàn chỉnh
- Thiết kế vòng lặp menu + phân tách method rõ trách nhiệm
- Parse biểu thức đơn giản và báo lỗi thân thiện (không crash)

## 2. Điều kiện tiên quyết

- Đã học xong (hoặc đọc kèm) các chương Level 1
- Thành thạo `dotnet new console`, method, `switch`, vòng lặp, `string` / `StringBuilder`, `TryParse`

## 3. Khái niệm / Đặc tả sản phẩm

### Chức năng bắt buộc (MVP)

1. **Tính biểu thức nhiều toán tử** với `+ - * /` và số thực, ví dụ:
   - `3 + 4 * 2` → `11` (nhân/chia trước cộng/trừ)
   - `(1 + 2) * 3` → `9` (có ngoặc — *nâng cao MVP+*)
2. **Lịch sử:** lưu các biểu thức đã tính thành công (ít nhất 20 mục gần nhất).
3. **Xử lý input sai:** không ném exception ra user; in thông báo rõ (chia 0, ký tự lạ, ngoặc lệch…).
4. **Menu:**
   - `calc` / nhập biểu thức trực tiếp
   - `history` — xem lịch sử
   - `clear` — xóa lịch sử
   - `help` — hướng dẫn
   - `exit` / `quit` — thoát

### Chức năng khuyến khích

- Hỗ trợ `%` (modulo) và unary minus (`-3 + 5`)
- Lệnh `last` — in kết quả gần nhất
- Ghi lịch sử kèm `DateTime.Now`
- Màu Console: lỗi đỏ, thành công xanh

### Không bắt buộc (để Level sau)

- Biến nhớ `ans`, hàm `sin`/`cos`, đồ thị — vượt Level 1

## 4. Mô hình tư duy

```text
┌─────────────────────────────────────────┐
│                 Menu loop               │
│  đọc lệnh → dispatch method             │
└───────────────┬─────────────────────────┘
                │ biểu thức
                ▼
        Tokenizer (chuỗi → tokens)
                │
                ▼
        Parser / Evaluator
        (ưu tiên * / ; rồi + -)
                │
                ▼
        double? result + thông báo lỗi
                │
                ▼
        History store (List hoặc mảng)
```

Hai hướng triển khai phổ biến ở Level 1:

| Hướng | Ý tưởng | Độ khó |
|-------|---------|--------|
| **A — Hai toán hạng lặp** | Tách số và toán tử thành hàng đợi; evaluate theo precedence | Trung bình |
| **B — Shunting-yard rút gọn** | Đổi infix → postfix rồi tính stack | Khó hơn, sạch hơn |

Bài project chấp nhận **hướng A hoặc B** miễn đúng kết quả và không crash.

## 5. Cú pháp / Skeleton gợi ý

Tạo project:

```bash
dotnet new console -n ConsoleCalculator
cd ConsoleCalculator
dotnet run
```

Khung `Program.cs` (top-level + local functions hoặc tách class):

```csharp
using System.Globalization;

var history = new List<string>();
double? last = null;

Console.WriteLine("Console Calculator — gõ help để xem lệnh");

while (true)
{
    Console.Write("> ");
    string? line = Console.ReadLine();
    if (line is null) break;
    line = line.Trim();
    if (line.Length == 0) continue;

    string cmd = line.ToLowerInvariant();
    if (cmd is "exit" or "quit") break;
    if (cmd == "help") { PrintHelp(); continue; }
    if (cmd == "history") { PrintHistory(history); continue; }
    if (cmd == "clear") { history.Clear(); Console.WriteLine("Đã xóa lịch sử."); continue; }
    if (cmd == "last")
    {
        Console.WriteLine(last is null ? "Chưa có kết quả." : last.Value.ToString(CultureInfo.InvariantCulture));
        continue;
    }

    // Coi toàn bộ dòng là biểu thức
    if (TryEvaluate(line, out double value, out string? error))
    {
        last = value;
        string entry = $"{line} = {value.ToString(CultureInfo.InvariantCulture)}";
        history.Add($"{DateTime.Now:HH:mm:ss}  {entry}");
        if (history.Count > 20) history.RemoveAt(0);
        WriteSuccess(entry);
    }
    else
    {
        WriteError(error ?? "Lỗi không xác định");
    }
}
```

Phần `TryEvaluate` / tokenizer bạn tự hoàn thiện theo các bước dưới.

## 6. Ví dụ hành vi mong đợi

### Cơ bản

```text
> 1 + 2
1 + 2 = 3
> 10 / 4
10 / 4 = 2.5
```

### Trung cấp

```text
> 3 + 4 * 2
3 + 4 * 2 = 11
> 8 / 0
Lỗi: chia cho 0
> abc + 1
Lỗi: token không hợp lệ
> history
12:01:03  1 + 2 = 3
12:01:10  3 + 4 * 2 = 11
```

### Nâng cao (có ngoặc)

```text
> (1 + 2) * 3
(1 + 2) * 3 = 9
> 2 * (3 + (4 - 1))
2 * (3 + (4 - 1)) = 12
```

## 7. Lỗi thường gặp (khi làm project)

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| `3+4` không parse | Tokenizer chỉ split theo space | Cho phép số dính toán tử hoặc chèn space khi tokenize |
| `4*2+3` = 20 sai | Không tôn trọng precedence | Tính `*`/`/` trước; hoặc dùng postfix |
| Crash `FormatException` | `Parse` thay vì `TryParse` | Mọi chỗ dùng `Try*` |
| Lịch sử phình vô hạn | Không giới hạn | Giữ tối đa 20–50 mục |
| Culture `3,14` vs `3.14` | Local parse | Dùng `CultureInfo.InvariantCulture` |

## 8. Gỡ lỗi

1. In tokens sau bước tokenize: `[3] [+] [4] [*] [2]`.
2. Viết bảng test cố định (mục 10) — chạy tay từng case.
3. Tách `TryEvaluate` test độc lập khỏi menu.
4. Khi nghi chia 0: breakpoint tại chỗ `/`.

## 9. Best practices

- `TryEvaluate(...)` trả `bool` + `out` result/error — API rõ.
- Không để logic parse nằm dài trong `while` menu.
- Số luôn format `InvariantCulture` khi in lịch sử (tái lập dễ).
- Message lỗi tiếng Việt ngắn, actionable.

## 10. Bài tập / Hạng mục bàn giao

Hoàn thành các hạng mục sau (đánh số để tự checklist):

**Hạng mục 1 — Bootstrap**  
*Input:* lệnh shell.  
*Output:* project `ConsoleCalculator` chạy được menu `help` / `exit`.

**Hạng mục 2 — Hai toán hạng**  
*Input:* `a + b` (có space).  
*Output:* kết quả đúng cho `+ - * /`; chia 0 báo lỗi.

**Hạng mục 3 — Nhiều toán tử + precedence**  
*Input:* `3 + 4 * 2`.  
*Output:* `11` (không phải `14`).

**Hạng mục 4 — Lịch sử**  
*Input:* vài biểu thức thành công rồi `history` / `clear`.  
*Output:* danh sách đúng; sau `clear` trống.

**Hạng mục 5 — Robust input**  
*Input:* chuỗi rác, thiếu toán hạng, ngoặc (nếu hỗ trợ).  
*Output:* không crash; có message lỗi.

## 11. Gợi ý triển khai từng bước (starter)

### Bước 1 — Menu xương

Chỉ `help`, `exit`, echo lại dòng người dùng.

### Bước 2 — Evaluate hai toán hạng

Thuật toán:

1. `Split` theo space → phải đúng 3 phần: `số`, `op`, `số`.
2. `double.TryParse(..., InvariantCulture, ...)`.
3. `switch (op)`.

### Bước 3 — Tokenize linh hoạt hơn

Duyệt từng `char`:

- Nếu digit hoặc `.` → gom thành số
- Nếu `+ - * / ( )` → token toán tử
- Bỏ qua whitespace
- Ký tự khác → lỗi

Gợi ý xử lý unary minus: nếu `-` đứng đầu hoặc sau `(` / toán tử khác → gắn vào số âm.

### Bước 4 — Precedence không ngoặc

Cách đơn giản (hướng A):

1. Quét trái→phải các phép `*` và `/` trước, thay 3 token `a op b` bằng 1 token kết quả.
2. Sau đó quét `+` và `-` tương tự.

Hoặc chuyển infix → postfix (Shunting-yard) rồi evaluate bằng `Stack<double>`.

### Bước 5 — History + màu

`List<string> history`; `Console.ForegroundColor` khi in.

### Bước 6 — (Tuỳ chọn) Ngoặc

Shunting-yard xử lý ngoặc tự nhiên hơn hướng A thủ công.

## 12. Đáp án — Outline giải pháp hoàn chỉnh

Dưới đây là **khung đáp án tham khảo** (đủ chạy cho biểu thức không ngoặc, có precedence). Bạn có thể mở rộng ngoặc sau.

Giải thích: tokenizer tách số và toán tử; evaluator xử lý `*` `/` trước rồi `+` `-`; menu giữ lịch sử.

```csharp
using System.Globalization;
using System.Text;

static class Calculator
{
    public static bool TryEvaluate(string expression, out double result, out string? error)
    {
        result = 0;
        error = null;

        if (!TryTokenize(expression, out var tokens, out error))
            return false;

        if (tokens.Count == 0)
        {
            error = "Biểu thức trống.";
            return false;
        }

        // Phase 1: * /
        if (!Reduce(tokens, highPrecedence: true, out error))
            return false;

        // Phase 2: + -
        if (!Reduce(tokens, highPrecedence: false, out error))
            return false;

        if (tokens.Count != 1 || !double.TryParse(tokens[0], NumberStyles.Float, CultureInfo.InvariantCulture, out result))
        {
            error = "Biểu thức không hợp lệ.";
            return false;
        }
        return true;
    }

    static bool Reduce(List<string> tokens, bool highPrecedence, out string? error)
    {
        error = null;
        int i = 1;
        while (i < tokens.Count - 1)
        {
            string op = tokens[i];
            bool isHigh = op is "*" or "/";
            bool isLow = op is "+" or "-";
            bool match = highPrecedence ? isHigh : isLow;
            if (!match)
            {
                i += 2;
                continue;
            }

            if (!double.TryParse(tokens[i - 1], NumberStyles.Float, CultureInfo.InvariantCulture, out double a)
                || !double.TryParse(tokens[i + 1], NumberStyles.Float, CultureInfo.InvariantCulture, out double b))
            {
                error = "Số không hợp lệ.";
                return false;
            }

            double value;
            switch (op)
            {
                case "+": value = a + b; break;
                case "-": value = a - b; break;
                case "*": value = a * b; break;
                case "/":
                    if (b == 0) { error = "Chia cho 0."; return false; }
                    value = a / b;
                    break;
                default:
                    error = $"Toán tử không hỗ trợ: {op}";
                    return false;
            }

            tokens[i - 1] = value.ToString(CultureInfo.InvariantCulture);
            tokens.RemoveAt(i);     // op
            tokens.RemoveAt(i);     // b cũ đã dịch trái
            // không tăng i: tiếp tục tại vị trí mới
        }
        return true;
    }

    static bool TryTokenize(string input, out List<string> tokens, out string? error)
    {
        tokens = new List<string>();
        error = null;
        int i = 0;
        while (i < input.Length)
        {
            char c = input[i];
            if (char.IsWhiteSpace(c)) { i++; continue; }

            if (char.IsDigit(c) || c == '.')
            {
                var sb = new StringBuilder();
                while (i < input.Length && (char.IsDigit(input[i]) || input[i] == '.'))
                {
                    sb.Append(input[i]);
                    i++;
                }
                tokens.Add(sb.ToString());
                continue;
            }

            // unary minus
            if (c == '-' && (tokens.Count == 0 || tokens[^1] is "+" or "-" or "*" or "/"))
            {
                var sb = new StringBuilder("-");
                i++;
                if (i >= input.Length || !(char.IsDigit(input[i]) || input[i] == '.'))
                {
                    error = "Dấu trừ đơn không hợp lệ.";
                    return false;
                }
                while (i < input.Length && (char.IsDigit(input[i]) || input[i] == '.'))
                {
                    sb.Append(input[i]);
                    i++;
                }
                tokens.Add(sb.ToString());
                continue;
            }

            if (c is '+' or '-' or '*' or '/')
            {
                tokens.Add(c.ToString());
                i++;
                continue;
            }

            error = $"Ký tự không hợp lệ: '{c}'";
            return false;
        }
        return true;
    }
}
```

Giải thích: điểm vào gọi `Calculator.TryEvaluate`, lưu lịch sử khi thành công — ghép với skeleton mục 5.

```csharp
// Trong menu loop khi không phải lệnh đặc biệt:
if (Calculator.TryEvaluate(line, out double value, out string? err))
{
    // cập nhật last + history như skeleton
}
else
{
    Console.ForegroundColor = ConsoleColor.Red;
    Console.WriteLine(err);
    Console.ResetColor();
}
```

Helpers in màu:

```csharp
static void WriteError(string msg)
{
    Console.ForegroundColor = ConsoleColor.Red;
    Console.WriteLine(msg);
    Console.ResetColor();
}

static void WriteSuccess(string msg)
{
    Console.ForegroundColor = ConsoleColor.Green;
    Console.WriteLine(msg);
    Console.ResetColor();
}

static void PrintHelp()
{
    Console.WriteLine("""
        Lệnh:
          <biểu thức>   ví dụ: 3 + 4 * 2
          history       xem lịch sử
          clear         xóa lịch sử
          last          kết quả gần nhất
          help          trợ giúp
          exit|quit     thoát
        Toán tử: + - * / ; nhân chia trước cộng trừ.
        """);
}

static void PrintHistory(List<string> history)
{
    if (history.Count == 0) { Console.WriteLine("(trống)"); return; }
    foreach (var h in history) Console.WriteLine(h);
}
```

## 13. Đáp án thay thế

**Shunting-yard rút gọn** (ý tưởng):

1. Output queue + operator stack.
2. Số → output; toán tử pop khi precedence ≥; cuối pop hết.
3. Evaluate postfix bằng stack số.

Phù hợp khi thêm `( )` và nhiều mức precedence.

**Thư viện:** không dùng parser NuGet — mục tiêu là luyện tay Level 1.

## 14. Thử thách

1. Thêm ngoặc `(` `)`.
2. Thêm toán tử `^` (lũy thừa, ưu tiên cao hơn `* /`, kết hợp phải→trái).
3. Lệnh `save path` / `load path` ghi lịch sử ra file text (xem trước Level 12 — IO).
4. Hỗ trợ dùng lại kết quả: gõ `ans + 2`.

## 15. Ứng dụng thực tế

Prototype công cụ nội bộ, REPL nhỏ, bước đệm trước khi làm interpreter/DSL. Kỹ năng tokenize + evaluate tái sử dụng khi đọc công thức cấu hình game / spreadsheet đơn giản.

## 16. Liên hệ Unity

Trong game ít làm calculator CLI, nhưng **cùng tư duy**:

- Parse lệnh chat `/give item 3`
- Công thức damage có ngoặc và hệ số
- Debug console trong game (menu lệnh)

Tách `TryEvaluate` thành module thuần C# có thể mang sang Unity (không phụ thuộc `MonoBehaviour`).

## 17. Kiểm tra kiến thức

1. Vì sao `3 + 4 * 2` phải ra `11`?  
   **Đáp án:** Quy tắc ưu tiên toán tử: nhân trước cộng.

2. `TryEvaluate` nên trả lỗi thế nào thay vì throw?  
   **Đáp án:** `bool` + `out`/message (hoặc Result type) để UI/menu chủ động hiển thị.

3. Lịch sử nên giới hạn để làm gì?  
   **Đáp án:** Tránh tốn bộ nhớ / spam màn hình khi session dài.

4. Vì sao dùng `InvariantCulture` khi parse số?  
   **Đáp án:** Thống nhất dấu `.` thập phân, tránh lệch theo locale máy.

5. Tokenize khác Split theo space chỗ nào?  
   **Đáp án:** Tokenize theo ký tự nhận `3+4` không space; Split cứng nhắc hơn.

---

## Checklist nộp bài (tự đánh giá)

- [ ] `dotnet run` vào menu, `help` / `exit` ổn
- [ ] Tính đúng `+ - * /` nhiều toán tử có precedence
- [ ] Chia 0 và input rác không crash
- [ ] `history` / `clear` / (khuyến khích) `last`
- [ ] Code tách method/class, đọc được trong < 5 phút / file chính
- [ ] Ít nhất 8 case test thủ công ghi trong comment đầu file

Khi hoàn thành project → bạn sẵn sàng chuyển **Level 2 — Object-Oriented Programming**.
