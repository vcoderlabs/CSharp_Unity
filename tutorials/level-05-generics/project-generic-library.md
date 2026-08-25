# Project Level 5 — Generic Collection Library

## 1. Mục tiêu học

- Tự implement `GenericStack<T>` (không chỉ wrap `Stack<T>` một dòng)
- Xây `GenericRepository<T>` với constraint hợp lý
- Viết interface tách biệt, demo type safety
- (Bonus) Áp dụng `out`/`in` trên read/write interfaces

## 2. Điều kiện tiên quyết

- Hoàn thành 3 chương Level 5
- Level 4: List, Dictionary
- .NET 8 console app

## 3. Khái niệm / Yêu cầu sản phẩm

### Phần A — GenericStack\<T\>

API tối thiểu:

| Member | Mô tả |
|--------|--------|
| `void Push(T item)` | Đẩy lên đỉnh |
| `T Pop()` | Lấy và xóa đỉnh; rỗng → exception |
| `bool TryPop(out T item)` | An toàn khi rỗng |
| `bool TryPeek(out T item)` | Xem đỉnh |
| `int Count { get; }` | Số phần tử |
| `void Clear()` | Xóa hết |

Yêu cầu triển khai: dùng **mảng nội bộ** + `count` (học resize giống List), **hoặc** `List<T>` nếu bạn ghi rõ trade-off trong README. Khuyến nghị: mảng + `EnsureCapacity` để ôn tư duy Level 4.

### Phần B — GenericRepository\<T\>

```csharp
interface IEntity<TKey>
{
    TKey Id { get; }
}

interface IRepository<TEntity, TKey>
    where TEntity : IEntity<TKey>
    where TKey : notnull
{
    void Add(TEntity entity);
    bool Remove(TKey id);
    bool TryGet(TKey id, out TEntity? entity);
    IReadOnlyCollection<TEntity> GetAll();
}
```

Implement `DictionaryRepository<TEntity, TKey>` bằng `Dictionary<TKey, TEntity>`.

### Phần C — Demo

Console demo:

1. Stack `int` và stack `string`
2. Repository `Player` với `Id: int`
3. Cố tình `Add` trùng Id → hành vi rõ ràng (throw hoặc overwrite — chọn một và document)

## 4. Mô hình tư duy

```text
GenericStack<T>
  _items: T[]
  _count: int
  Push → nếu đầy, newCapacity = max(4, _count*2), Array.Copy
  Pop  → lấy [_count-1], _items[--_count] = default! (tránh giữ reference)

GenericRepository
  Dictionary<TKey, TEntity>
  Add: ContainsKey? → policy
  GetAll: trả Values snapshot hoặc IReadOnlyCollection view
```

## 5. Cú pháp / Skeleton

```bash
dotnet new console -n GenericLibrary -f net8.0
cd GenericLibrary
```

Có thể tách file: `GenericStack.cs`, `IRepository.cs`, `DictionaryRepository.cs`, `Program.cs`.

## 6. Ví dụ hướng dẫn

### Cơ bản — Push/Pop với mảng

```csharp
public class GenericStack<T>
{
    private T[] _items = new T[4];
    private int _count;

    public int Count => _count;

    public void Push(T item)
    {
        if (_count == _items.Length)
            Grow();
        _items[_count++] = item;
    }

    private void Grow()
    {
        int newCap = _items.Length * 2;
        var next = new T[newCap];
        Array.Copy(_items, next, _count);
        _items = next;
    }
}
```

### Trung cấp — TryPop + Clear references

```csharp
public bool TryPop(out T item)
{
    if (_count == 0)
    {
        item = default!;
        return false;
    }
    item = _items[--_count];
    _items[_count] = default!; // giúp GC nếu T là reference type
    return true;
}
```

### Nâng cao — Repository

```csharp
public sealed class DictionaryRepository<TEntity, TKey> : IRepository<TEntity, TKey>
    where TEntity : IEntity<TKey>
    where TKey : notnull
{
    private readonly Dictionary<TKey, TEntity> _map = new();

    public void Add(TEntity entity)
    {
        if (!_map.TryAdd(entity.Id, entity))
            throw new InvalidOperationException($"Duplicate id: {entity.Id}");
    }

    public bool Remove(TKey id) => _map.Remove(id);

    public bool TryGet(TKey id, out TEntity? entity) => _map.TryGetValue(id, out entity);

    public IReadOnlyCollection<TEntity> GetAll() => _map.Values;
}
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Pop trả dữ liệu “ma” | Không giảm count / không clear slot | `--_count` + `default` |
| Grow quên copy | Chỉ `new T[n]` | `Array.Copy` |
| Repository Add im lặng ghi đè | Dùng indexer | `TryAdd` nếu muốn fail |
| `GetAll` bị sửa từ ngoài | Trả mutable nội bộ | `ToArray()` / copy nếu cần đóng băng |
| Constraint thiếu | `TEntity` không có Id | Implement `IEntity<TKey>` |

## 8. Gỡ lỗi

1. Unit test tay: Push 1..5, Pop kỳ vọng 5..1.
2. Grow: Push đến vượt capacity ban đầu — xem không mất phần tử.
3. Repository: Add hai lần cùng Id — xác nhận exception.
4. Sau Pop stack object, đảm bảo không giữ reference (tuỳ chọn: WeakReference thử nghiệm).

## 9. Best practices

- Không kế thừa `Stack<T>` chỉ để đổi tên — mục tiêu là **hiểu implement**.
- XML doc ngắn cho public API.
- `Try*` cho thao tác có thể fail vì rỗng.
- Repository: quyết định rõ duplicate policy.
- Bonus: `IReadOnlyRepository<out TEntity, TKey>` nếu muốn ôn variance (TKey thường invariant/`notnull`).

## 10. Bài tập (milestone)

**M1** — `GenericStack<T>` Push/Pop/Count với mảng.  
**M2** — TryPop/TryPeek/Clear.  
**M3** — `IEntity` + `DictionaryRepository`.  
**M4** — Demo Program với Player.  
**M5** — (Bonus) `IReadStack`/`IWriteStack` hoặc read-only repository + test gán covariance.  
**M6** — (Bonus) So sánh Stopwatch Push×1e6 của bạn vs `Stack<T>` BCL.

## 11. Gợi ý

- Capacity khởi đầu 4 hoặc 16.
- `Pop` ném `InvalidOperationException` giống BCL.
- Player: `record Player(int Id, string Name) : IEntity<int> { int IEntity<int>.Id => Id; }` hoặc property Id public khớp interface.
- Demo in `GetAll()` sau Add/Remove.

## 12. Đáp án

Giải thích: Stack dùng mảng động; Repository dùng Dictionary để O(1) theo Id — áp dụng Level 4 + 5.

```csharp
public class GenericStack<T>
{
    private T[] _items = new T[4];
    private int _count;
    public int Count => _count;

    public void Push(T item)
    {
        if (_count == _items.Length)
        {
            var next = new T[_items.Length * 2];
            Array.Copy(_items, next, _count);
            _items = next;
        }
        _items[_count++] = item;
    }

    public T Pop()
    {
        if (!TryPop(out T item))
            throw new InvalidOperationException("Stack empty");
        return item;
    }

    public bool TryPop(out T item)
    {
        if (_count == 0)
        {
            item = default!;
            return false;
        }
        item = _items[--_count];
        _items[_count] = default!;
        return true;
    }

    public bool TryPeek(out T item)
    {
        if (_count == 0)
        {
            item = default!;
            return false;
        }
        item = _items[_count - 1];
        return true;
    }

    public void Clear()
    {
        for (int i = 0; i < _count; i++)
            _items[i] = default!;
        _count = 0;
    }
}

public interface IEntity<TKey>
{
    TKey Id { get; }
}

public interface IRepository<TEntity, TKey>
    where TEntity : IEntity<TKey>
    where TKey : notnull
{
    void Add(TEntity entity);
    bool Remove(TKey id);
    bool TryGet(TKey id, out TEntity? entity);
    IReadOnlyCollection<TEntity> GetAll();
}

public sealed class DictionaryRepository<TEntity, TKey> : IRepository<TEntity, TKey>
    where TEntity : IEntity<TKey>
    where TKey : notnull
{
    private readonly Dictionary<TKey, TEntity> _map = new();

    public void Add(TEntity entity)
    {
        if (!_map.TryAdd(entity.Id, entity))
            throw new InvalidOperationException($"Duplicate id: {entity.Id}");
    }

    public bool Remove(TKey id) => _map.Remove(id);

    public bool TryGet(TKey id, out TEntity? entity)
        => _map.TryGetValue(id, out entity);

    public IReadOnlyCollection<TEntity> GetAll() => _map.Values;
}

// Demo entity
public sealed record Player(int Id, string Name) : IEntity<int>;
```

`Program.cs` mẫu: tạo stack, push/pop; tạo repo, add players, TryGet, Remove.

## 13. Đáp án thay thế

- Stack dựa `List<T>`: ngắn hơn, ít học resize.
- Repository overwrite: `_map[entity.Id] = entity` — Upsert semantics.
- Thêm `IReadOnlyRepository<out TEntity>` chỉ có TryGet/GetAll — ôn chương 3.

## 14. Thử thách

1. `GenericQueue<T>` ring buffer (mảng + head/tail).  
2. `GenericRepository` hỗ trợ `Find(Func<TEntity, bool>)` trả `List<TEntity>`.  
3. Viết file `TESTS.md` mô tả 10 case thủ công bạn đã chạy.

## 15. Ứng dụng thực tế

- Nền tảng hiểu BCL `Stack<T>` / repository packages
- Framework nội bộ: generic service `IRepository<T>`
- Chuẩn bị Level 14 Testing: dễ viết unit test cho generic library

## 16. Liên hệ Unity

- `GenericStack` cho undo editor tool / spell cast stack
- `DictionaryRepository<Enemy, int>` cho entity id trên client
- Pool: `GenericStack<T>` hoặc `Queue<T>` trả object về pool
- Constraint `where T : Component` khi store Unity objects

## 17. Kiểm tra kiến thức

1. Vì sao Clear/Pop nên gán `default` vào slot mảng?  
   **Đáp án:** Tránh giữ reference chết → giúp GC thu hồi sớm hơn với reference type.

2. Repository dùng Dictionary thay List vì sao?  
   **Đáp án:** Tra cứu/xóa theo Id O(1) average.

3. `where TKey : notnull` giúp gì?  
   **Đáp án:** Key Dictionary không null; rõ ràng hơn với nullable reference types.

4. `TryPop` khác `Pop`?  
   **Đáp án:** TryPop trả false khi rỗng; Pop ném exception.

5. Mục tiêu project không phải wrap BCL một dòng — vì sao?  
   **Đáp án:** Để hiểu cấu trúc dữ liệu + generics khi tự thiết kế API.
