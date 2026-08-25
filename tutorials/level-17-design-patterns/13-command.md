# Chương 13 — Command

## 1. Mục tiêu học

- Đóng gói request thành object: `Execute` / `Undo`
- Hàng đợi, macro, replay input
- Unity: skill queue, input buffer, editor undo tâm lý

## 2. Điều kiện tiên quyết

- Interface, queue (L4)
- Observer hữu ích cho “sau khi execute”

## 3. Khái niệm

**Command:** tách *người gọi* khỏi *người làm*. Object lệnh mang đủ thông tin thực thi (và hoàn tác).

## 4. Mô hình tư duy

```text
Input → ICommand → Receiver (Player, Inventory)
Invoker.Execute(cmd) / Undo stack
```

## 5. Cú pháp

```csharp
public interface ICommand
{
    void Execute();
    void Undo();
}
```

## 6. Ví dụ

### Cơ bản

```csharp
public sealed class MoveCommand : ICommand
{
    private readonly Player _p;
    private readonly Vector3 _delta;
    public MoveCommand(Player p, Vector3 d) { _p = p; _delta = d; }
    public void Execute() => _p.Position += _delta;
    public void Undo() => _p.Position -= _delta;
}
```

### Trung cấp — Invoker + history

```csharp
public sealed class CommandHistory
{
    private readonly Stack<ICommand> _undo = new();
    public void Run(ICommand c) { c.Execute(); _undo.Push(c); }
    public void Undo() { if (_undo.Count > 0) _undo.Pop().Undo(); }
}
```

### Nâng cao / Unity MMORPG

```csharp
public interface IGameCommand
{
    void Execute(GameContext ctx);
}

// Network: serialize command lên server (authoritative)
// Skill: enqueue CastFireballCommand — animation lock xử lý ở invoker
```

MacroCommand: list lệnh Execute tuần tự; Undo ngược.

## 7. Lỗi thường gặp

| Hiện tượng | Cách xử lý |
|------------|------------|
| Undo không đủ state | Lưu snapshot / reverse ops đủ |
| Command giữ MonoBehaviour chết | Capture id/data không giữ destroyed ref |
| Queue không giới hạn | Cap buffer input |

## 8. Gỡ lỗi

Log command name khi execute/undo. Test round-trip Execute→Undo khôi phục state.

## 9. Best practices

- Command nhỏ, dữ liệu rõ.  
- Receiver chứa domain; command mỏng.  
- Server game: validate command, đừng tin client.

## 10. Bài tập

**Bài 1** — `Deposit`/`Withdraw` command có Undo.  
**Bài 2** — Macro bật 3 đèn.  
**Bài 3** — Input map phím → command factory.  
**Bài 4** — So sánh vs event (Observer).

## 11. Gợi ý

Event thông báo đã xảy ra; Command là ý định *sắp/đang* thực thi có thể xếp hàng/undo.

## 12. Đáp án

```csharp
public sealed class DepositCommand : ICommand
{
    private readonly Account _acc;
    private readonly decimal _amount;
    public DepositCommand(Account acc, decimal amount) { _acc = acc; _amount = amount; }
    public void Execute() => _acc.Deposit(_amount);
    public void Undo() => _acc.Withdraw(_amount);
}
```

## 13. Đáp án thay thế

`Action` queue không Undo — Command “tối giản”. CQRS command objects ở backend.

## 14. Thử thách

Skill system: queue command, cancel nếu stun (state).

## 15. Ứng dụng thực tế

- GUI buttons  
- Job queues  
- Transaction scripts

## 16. Liên hệ Unity

- **Critical:** input, combat skills, replay/demo  
- `UnityEvent` ≠ full Command nhưng gần “gọi hành động”  
- Capstone Milestone 05 combat

## 17. Kiểm tra kiến thức

1. Command đóng gói gì? **Request/hành động thành object.**  
2. Invoker làm gì? **Gọi Execute, giữ history.**  
3. Undo cần gì? **Đủ thông tin đảo ngược.**  
4. Macro? **Nhiều command như một.**  
5. Mạng game? **Serialize command, server execute.**
