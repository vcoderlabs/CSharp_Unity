# Chương 6 — Multicast Delegates

## 1. Mục tiêu học

- Hiểu delegate có thể trỏ **nhiều** method (`+=` kết hợp)
- Thứ tự gọi: thường theo thứ tự đăng ký
- `-=` gỡ đúng handler; `GetInvocationList`
- Hành vi khi một handler **throw** giữa chuỗi
- Liên hệ UnityEvent nhiều listener

## 2. Điều kiện tiên quyết

- Chương 1, 5: delegate, event, += / -=
- Level 6: exception cơ bản (khi một listener lỗi)

## 3. Khái niệm

Delegate trong .NET là **multicast**: invocation list có 0..n entry.

```csharp
Action a = () => Console.WriteLine("A");
a += () => Console.WriteLine("B");
a(); // A rồi B
a -= /* handler B */; // cần cùng reference
```

Với `event`, bên ngoài chỉ thao tác list qua +=/-=; publisher `Invoke` cả list.

Nếu một handler ném exception: các handler **sau** có thể không chạy (trừ khi bạn tự `GetInvocationList` và try/catch từng cái).

## 4. Mô hình tư duy

```text
Invocation list: [h1] → [h2] → [h3]
Invoke:
  h1() 
  h2()  ← nếu throw ở đây, h3 có thể bị bỏ qua
  h3()

Gỡ: -= phải cùng instance delegate/method group đã +=
Lambda mới mỗi lần = không gỡ được bằng lambda “giống chữ”
```

## 5. Cú pháp

```csharp
Action pipeline = null!;
pipeline += Step1;
pipeline += Step2;
pipeline();

foreach (Delegate d in pipeline.GetInvocationList())
{
    try
    {
        ((Action)d)();
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Listener lỗi: {ex.Message}");
    }
}
```

`Delegate.Combine` / `Remove` là API tường minh đằng sau +=/-=.

## 6. Ví dụ

### Cơ bản

```csharp
void Log() => Console.WriteLine("log");
void Audit() => Console.WriteLine("audit");

Action onSave = Log;
onSave += Audit;
onSave(); // log \n audit
```

### Trung cấp

```csharp
static void SafeRaise(Action? handlers)
{
    if (handlers is null) return;
    foreach (Action h in handlers.GetInvocationList())
    {
        try { h(); }
        catch (Exception ex) { Console.WriteLine(ex.Message); }
    }
}
```

### Nâng cao

Thứ tự quan trọng (validation trước persistence):

```csharp
event Action? Saving;
void Save()
{
    Saving?.Invoke(); // listeners: validate, then write — đăng ký đúng thứ tự
}
```

Hoặc tách hai event: `Validating` / `Saving`.

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| `-=` không ăn | Lambda khác instance | Lưu handler vào field |
| Một listener làm im các listener sau | Exception giữa Invoke | SafeRaise từng cái |
| Kỳ vọng return value multicast | Chỉ nhận return của **last** handler | Đừng dựa return; dùng event void |
| `+=` trùng vô ý | Subscribe 2 lần | Guard hoặc unsubscribe trước |

## 8. Gỡ lỗi

1. In `GetInvocationList().Length` và `Method.Name`.  
2. Breakpoint từng listener.  
3. Reproduce: listener đầu throw — xem listener sau có chạy không.

## 9. Best practices

- Event/Action void cho notification — tránh multicast Func dựa return.
- Publisher quan trọng: cân nhắc invoke an toàn từng handler.
- Document thứ tự có guarantee hay không.
- Unity: nhiều `AddListener` = multicast tương đương; một lỗi trong gọi native có thể log và tiếp/không tùy phiên bản — vẫn nên listener tự try/catch nếu critical.
- Idempotent subscribe khi có thể.

## 10. Bài tập

**Bài 1** — Ba Action in `"1"`,`"2"`,`"3"`; combine và gọi.

**Bài 2** — Gỡ listener `"2"`; gọi lại chỉ còn 1 và 3.

**Bài 3** — Listener giữa throw; quan sát (viết comment) listener sau.

**Bài 4** — Viết `SafeInvoke(Action?)` dùng GetInvocationList + try/catch.

## 11. Gợi ý

- Bài 1–2: method có tên `Print1`, `Print2`, … để -= được.
- Bài 3: `Invoke` thẳng vs SafeInvoke.

## 12. Đáp án

**Bài 1** — Combine ba listener:

```csharp
static void Print1() => Console.WriteLine("1");
static void Print2() => Console.WriteLine("2");
static void Print3() => Console.WriteLine("3");

Action a = Print1;
a += Print2;
a += Print3;
a(); // 1 2 3
```

**Bài 2** — Gỡ Print2:

```csharp
a -= Print2;
a(); // 1 3
```

**Bài 3** — Exception cắt chuỗi:

```csharp
static void Boom() => throw new InvalidOperationException("boom");

Action b = Print1;
b += Boom;
b += Print3;
try { b(); }
catch { Console.WriteLine("caught at caller"); }
// Print3 thường KHÔNG chạy khi Invoke gộp; comment quan sát của bạn
```

**Bài 4** — SafeInvoke:

```csharp
static void SafeInvoke(Action? handlers)
{
    if (handlers is null) return;
    foreach (var d in handlers.GetInvocationList())
    {
        try
        {
            ((Action)d)();
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Handler failed: {ex.Message}");
        }
    }
}

SafeInvoke(b); // 1, log boom, vẫn chạy 3
```

## 13. Đáp án thay thế

Dùng `event` thay `Action` field — cùng multicast semantics khi raise bên trong. `Immutable` list listener tự quản lý thay multicast built-in.

## 14. Thử thách

Đo thời gian: 1000 listener rỗng — Invoke một lần vs vòng foreach GetInvocationList. Ghi nhận (không cần micro-optimize sớm).

## 15. Ứng dụng thực tế

- Pipeline notify nhiều module (analytics + UI + audio)
- Plugin nhiều extension cùng hook
- Validation chain (cẩn thận exception)
- Logging + metrics song song trên một event

## 16. Liên hệ Unity

- `UnityEvent` giữ list persistent (Inspector) + runtime listeners
- `RemoveAllListeners` xóa runtime — cẩn thận với persistent
- Một listener NRE có thể làm các listener sau không chạy tùy cách invoke — listener nên defensive (`if (!this) return` pattern Unity)
- Cùng quy tắc: giữ reference để RemoveListener

## 17. Kiểm tra kiến thức

1. `+=` trên delegate làm gì?  
   **Đáp án:** Thêm method vào invocation list (multicast).

2. Return value của multicast Func lấy từ đâu?  
   **Đáp án:** Thường chỉ từ handler **cuối** — tránh phụ thuộc.

3. Một handler throw khi `Invoke` gộp?  
   **Đáp án:** Các handler sau có thể không được gọi.

4. Làm sao gọi tiếp dù có lỗi?  
   **Đáp án:** Duyệt `GetInvocationList` và try/catch từng cái.

5. Vì sao lambda inline khó `-=`?  
   **Đáp án:** Mỗi lambda là instance khác — không khớp entry đã thêm.
