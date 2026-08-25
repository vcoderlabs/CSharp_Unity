# Chương 15 — Template Method

## 1. Mục tiêu học

- Định nghĩa **khung thuật toán** ở base; subclass override bước
- Phân biệt với Strategy (composition vs inheritance)
- Dùng cho match flow, import pipeline, turn sequence

## 2. Điều kiện tiên quyết

- Abstract class, `virtual`/`abstract`
- LSP — override không phá khung

## 3. Khái niệm

**Template Method:** method cha gọi các bước; một số bước abstract/hook. Subclass điền chi tiết, **không đổi thứ tự tổng thể**.

## 4. Mô hình tư duy

```text
PlayMatch()
  1. Load()
  2. Spawn()      ← abstract
  3. Loop()
  4. Settle()     ← abstract
  5. Cleanup()    ← hook optional
```

## 5. Cú pháp

```csharp
public abstract class MatchTemplate
{
    public void Run()
    {
        Load();
        SpawnPlayers();
        PlayLoop();
        Settle();
        Cleanup();
    }

    protected virtual void Load() { /* default */ }
    protected abstract void SpawnPlayers();
    protected abstract void PlayLoop();
    protected abstract void Settle();
    protected virtual void Cleanup() { }
}
```

## 6. Ví dụ

### Cơ bản

`DeathmatchMatch` / `CapturePointMatch` override spawn & settle.

### Trung cấp — data import

```csharp
public abstract class ImportTemplate
{
    public void Import(Stream s)
    {
        var raw = Read(s);
        var model = Parse(raw);
        Validate(model);
        Save(model);
    }
    protected abstract string Read(Stream s);
    protected abstract Model Parse(string raw);
    protected virtual void Validate(Model m) { }
    protected abstract void Save(Model m);
}
```

### Nâng cao / Unity

`LevelFlow`: LoadScene → InjectDeps → Spawn → StartSystems — mode khác nhau override spawn.

## 7. Lỗi thường gặp

| Hiện tượng | Cách xử lý |
|------------|------------|
| Subclass gọi sai thứ tự | Chỉ public `Run`; bước protected |
| Quá nhiều abstract | Hook default rỗng |
| Inheritance sâu | Prefer Strategy/pipeline |

## 8. Gỡ lỗi

Log mỗi bước template. Test subclass chỉ cần verify bước riêng + thứ tự qua spy.

## 9. Best practices

- Final template method (`public` không virtual).  
- Ít bước; rõ hợp đồng.  
- Nếu đổi thuật toán mạnh → Strategy.

## 10. Bài tập

**Bài 1** — `Beverage`: boil → brew → pour → addCondiments.  
**Bài 2** — CSV vs JSON import.  
**Bài 3** — Hook `afterPour`.  
**Bài 4** — So Strategy.

## 11. Gợi ý

Tea/Coffee khác `brew` và `addCondiments`.

## 12. Đáp án

```csharp
public abstract class Beverage
{
    public void Prepare() { Boil(); Brew(); Pour(); AddCondiments(); }
    void Boil() => Console.WriteLine("boil");
    void Pour() => Console.WriteLine("pour");
    protected abstract void Brew();
    protected abstract void AddCondiments();
}
```

## 13. Đáp án thay thế

Pipeline list `IStep` — Template Method dạng composition (dễ OCP hơn).

## 14. Thử thách

Turn-based: StartTurn → Draw → Main → End — PvE vs PvP override Draw.

## 15. Ứng dụng thực tế

- Frameworks (ASP.NET filter pipeline gần ý)  
- Document generation

## 16. Liên hệ Unity

- Loading flow modes  
- Editor importers  
- Đừng lạm dụng abstract MonoBehaviour sâu

## 17. Kiểm tra kiến thức

1. Template Method cố định gì? **Khung/thứ tự thuật toán.**  
2. Subclass làm gì? **Điền bước.**  
3. Khác Strategy? **Inheritance khung vs inject thuật toán.**  
4. Hook? **Bước optional có default.**  
5. Template method nên virtual? **Thường không — khóa quy trình.**
