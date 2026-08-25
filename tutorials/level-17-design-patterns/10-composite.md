# Chương 10 — Composite

## 1. Mục tiêu học

- Cây object đồng nhất: leaf và composite cùng interface
- Thao tác đệ quy (UI tree, buff tree, org chart)
- Tránh xử lý leaf/composite bằng `is` khắp nơi

## 2. Điều kiện tiên quyết

- Cây / đệ quy cơ bản
- Collection (L4)

## 3. Khái niệm

**Composite** cho phép client xử lý **phần tử đơn** và **nhóm phần tử** như nhau qua `IComponent`.

## 4. Mô hình tư duy

```text
INode
├─ Leaf (File)
└─ Composite (Folder)
     ├─ Leaf
     └─ Composite ...
```

## 5. Cú pháp

```csharp
public interface IFileSystemEntry
{
    string Name { get; }
    long Size();
}

public sealed class FileLeaf : IFileSystemEntry
{
    public string Name { get; }
    private readonly long _size;
    public FileLeaf(string name, long size) { Name = name; _size = size; }
    public long Size() => _size;
}

public sealed class Folder : IFileSystemEntry
{
    public string Name { get; }
    private readonly List<IFileSystemEntry> _children = new();
    public Folder(string name) => Name = name;
    public void Add(IFileSystemEntry e) => _children.Add(e);
    public long Size() => _children.Sum(c => c.Size());
}
```

## 6. Ví dụ

### Cơ bản

Tính tổng size folder như trên.

### Trung cấp — UI

```csharp
public interface IUiNode
{
    void Draw(int indent);
}

public sealed class Panel : IUiNode
{
    private readonly List<IUiNode> _kids = new();
    public void Add(IUiNode n) => _kids.Add(n);
    public void Draw(int indent)
    {
        Console.WriteLine($"{new string(' ', indent)}Panel");
        foreach (var k in _kids) k.Draw(indent + 2);
    }
}
```

### Nâng cao / Unity

Skill/composite buff: `CompositeEffect` apply tất cả children. Transform hierarchy Unity *tương tự tinh thần* composite scene graph.

## 7. Lỗi thường gặp

| Hiện tượng | Cách xử lý |
|------------|------------|
| `Add` trên Leaf throw | ISP: tách `IComposite` nếu cần |
| Cycle trong cây | Guard parent/id visited |
| Quá sâu đệ quy | Iterative stack / limit depth |

## 8. Gỡ lỗi

Unit test size cây nhỏ. Detect cycle bằng HashSet khi Add.

## 9. Best practices

- Client chỉ thấy `IComponent`.  
- Document leaf có `Add` hay không (tinh khiết vs tiện dụng).  
- Immutable composite khi share cây.

## 10. Bài tập

**Bài 1** — Menu/MenuItem composite print.  
**Bài 2** — Damage composite = tổng children.  
**Bài 3** — Cấm cycle khi Add.  
**Bài 4** — So sánh với Decorator.

## 11. Gợi ý

Cycle: không Add tổ tiên của chính mình.

## 12. Đáp án

```csharp
public long TotalDamage(IDamageNode node) => node.Evaluate();
// Composite.Evaluate = sum children; Leaf = fixed value
```

## 13. Đáp án thay thế

Flatten tree thành list khi chỉ cần iterate một lần — đơn giản hơn nếu không cần cấu trúc.

## 14. Thử thách

Quest objective composite: AllOf / AnyOf children complete.

## 15. Ứng dụng thực tế

- DOM / UI trees  
- Organization permissions rollup

## 16. Liên hệ Unity

- Hierarchy GameObject  
- Nested inventory containers  
- Behavior tree (họ hàng ý tưởng)

## 17. Kiểm tra kiến thức

1. Composite thống nhất gì? **Leaf và group cùng interface.**  
2. Lợi ích client? **Không cần phân nhánh loại.**  
3. Rủi ro? **Cycle, đệ quy sâu, API Add trên leaf.**  
4. Khác Decorator? **Cây nhiều con vs bọc 1 component.**  
5. `Size()` folder? **Tổng đệ quy children.**
