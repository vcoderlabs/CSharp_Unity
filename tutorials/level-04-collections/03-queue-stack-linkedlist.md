# Chương 3 — Queue, Stack và LinkedList

## 1. Mục tiêu học

- Phân biệt FIFO (`Queue<T>`) và LIFO (`Stack<T>`)
- Dùng `Enqueue`/`Dequeue`, `Push`/`Pop` đúng ngữ cảnh
- Hiểu `LinkedList<T>` và khi nào nó hữu ích hơn List
- Tránh nhầm Peek với thao tác lấy-ra

## 2. Điều kiện tiên quyết

- Chương 1–2: List, Dictionary
- Tư duy thuật toán cơ bản (Level 0 Big O trực giác)

## 3. Khái niệm

### Queue\<T\> — hàng đợi (FIFO)

- Vào trước → ra trước
- `Enqueue` thêm cuối; `Dequeue` lấy đầu; `Peek` xem đầu không xóa

### Stack\<T\> — ngăn xếp (LIFO)

- Vào sau → ra trước
- `Push` thêm đỉnh; `Pop` lấy đỉnh; `Peek` xem đỉnh

### LinkedList\<T\>

- Danh sách liên kết đôi: mỗi node có Previous/Next
- Thêm/xóa **ở đầu/cuối hoặc tại node đã biết**: O(1)
- Truy cập theo index: O(n) (không có indexer nhanh như List)
- Ít dùng hơn List/Dictionary trong app thường ngày — hữu ích khi cần chèn/xóa giữa nhiều lần và đã giữ reference tới node

| Cấu trúc | Thứ tự | Thêm/xóa đặc trưng |
|----------|--------|---------------------|
| Queue | FIFO | Đuôi vào, đầu ra |
| Stack | LIFO | Đỉnh vào/ra |
| LinkedList | Tùy vị trí node | O(1) nếu có node |
| List | Index | Cuối nhanh; giữa chậm |

## 4. Mô hình tư duy

```text
Queue (FIFO) — xử lý request:
Enqueue A, B, C     Dequeue → A rồi B rồi C
front → [A][B][C] ← back

Stack (LIFO) — Undo:
Push A, B, C        Pop → C rồi B rồi A
        [C] ← top
        [B]
        [A]

LinkedList:
null ← [A] ⇄ [B] ⇄ [C] → null
Thêm trước B: O(1) nếu đã có node B
```

## 5. Cú pháp

```csharp
var q = new Queue<string>();
q.Enqueue("job1");
q.Enqueue("job2");
string next = q.Dequeue();   // job1
string look = q.Peek();      // job2 (còn trong queue)

var st = new Stack<int>();
st.Push(1);
st.Push(2);
int top = st.Pop();          // 2

var ll = new LinkedList<string>();
ll.AddLast("A");
ll.AddLast("C");
var nodeB = ll.AddAfter(ll.First!, "B"); // A-B-C
ll.Remove(nodeB);
```

Ưu tiên `TryDequeue` / `TryPop` / `TryPeek` (.NET) để tránh exception khi rỗng.

## 6. Ví dụ

### Cơ bản

Mô phỏng máy in (queue):

```csharp
var printQueue = new Queue<string>();
printQueue.Enqueue("docA.pdf");
printQueue.Enqueue("docB.pdf");

while (printQueue.TryDequeue(out string? doc))
    Console.WriteLine($"Printing {doc}");
```

### Trung cấp

Kiểm tra ngoặc đúng bằng Stack:

```csharp
static bool IsBalanced(string s)
{
    var stack = new Stack<char>();
    foreach (char c in s)
    {
        if (c is '(' or '[' or '{')
            stack.Push(c);
        else if (c is ')' or ']' or '}')
        {
            if (!stack.TryPop(out char open)) return false;
            if (open == '(' && c != ')') return false;
            if (open == '[' && c != ']') return false;
            if (open == '{' && c != '}') return false;
        }
    }
    return stack.Count == 0;
}
```

### Nâng cao

LRU đơn giản bằng LinkedList + Dictionary (ý tưởng cache):

```csharp
// Ý tưởng: Dictionary<key, LinkedListNode<(key,value)>>
// + LinkedList để duy trì thứ tự dùng gần nhất
// Get: move node về First; Put: AddFirst, nếu quá capacity RemoveLast
```

(Implement đầy đủ để làm thử thách / project nhỏ.)

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| `InvalidOperationException` | Pop/Dequeue khi rỗng | `TryPop` / `TryDequeue` hoặc kiểm `Count` |
| Nhầm Peek xóa phần tử | Peek chỉ xem | Dùng Pop/Dequeue khi cần lấy ra |
| Dùng List như Queue (`RemoveAt(0)`) | O(n) mỗi lần | Dùng `Queue<T>` |
| LinkedList indexer `ll[i]` | Không có | Duyệt hoặc giữ node reference |

## 8. Gỡ lỗi

1. In `Count` trước Pop/Dequeue.
2. Vẽ trên giấy thứ tự Push/Enqueue khi thuật toán sai.
3. Với ngoặc: in stack sau mỗi ký tự.
4. LinkedList: kiểm `First`/`Last` null khi list rỗng.

## 9. Best practices

- Hàng đợi công việc / BFS → `Queue`
- Undo/Redo, DFS, parse → `Stack`
- Đừng dùng `List.RemoveAt(0)` để mô phỏng queue
- Prefer `Try*` API trên collection rỗng có thể xảy ra
- LinkedList: chỉ chọn khi profile cho thấy chèn giữa trên List là bottleneck **và** bạn giữ được node

## 10. Bài tập

**Bài 1 — Đảo chuỗi bằng Stack**  
*Input:* chuỗi.  
*Output:* chuỗi đảo (dùng Stack, không dùng `Reverse` có sẵn trên LINQ).

**Bài 2 — Hot potato**  
*Input:* danh sách tên + số `k`.  
*Output:* người còn lại sau khi mỗi vòng loại người thứ k (Queue Josephus đơn giản).

**Bài 3 — Next Greater Element**  
*Input:* `int[]`.  
*Output:* với mỗi phần tử, phần tử lớn hơn đầu tiên bên phải; không có thì -1. (Stack monotonic.)

**Bài 4 — LinkedList insert sorted**  
*Input:* các số lần lượt.  
*Output:* `LinkedList<int>` luôn tăng dần sau mỗi lần chèn.

## 11. Gợi ý

- Bài 1: Push từng char, rồi Pop nối vào `StringBuilder`.
- Bài 2: Enqueue tất cả; mỗi bước Dequeue rồi Enqueue lại (k-1) lần, Dequeue bỏ người thứ k.
- Bài 3: Duyệt từ phải hoặc trái với stack chỉ số.
- Bài 4: Tìm node đầu tiên `> value`, `AddBefore`; nếu không có thì `AddLast`.

## 12. Đáp án

**Bài 1** — Stack chứa char, Pop ra theo LIFO:

```csharp
static string ReverseWithStack(string s)
{
    var stack = new Stack<char>();
    foreach (char c in s) stack.Push(c);
    var sb = new System.Text.StringBuilder(s.Length);
    while (stack.TryPop(out char c)) sb.Append(c);
    return sb.ToString();
}
```

**Bài 2** — Josephus với Queue:

```csharp
static string HotPotato(IEnumerable<string> names, int k)
{
    var q = new Queue<string>(names);
    while (q.Count > 1)
    {
        for (int i = 0; i < k - 1; i++)
            q.Enqueue(q.Dequeue());
        q.Dequeue(); // loại
    }
    return q.Dequeue();
}
```

**Bài 3** — Monotonic stack (duyệt phải → trái):

```csharp
static int[] NextGreater(int[] nums)
{
    var result = new int[nums.Length];
    Array.Fill(result, -1);
    var stack = new Stack<int>(); // indices, values tăng dần từ đỉnh...
    for (int i = nums.Length - 1; i >= 0; i--)
    {
        while (stack.Count > 0 && nums[stack.Peek()] <= nums[i])
            stack.Pop();
        if (stack.Count > 0)
            result[i] = nums[stack.Peek()];
        stack.Push(i);
    }
    return result;
}
```

**Bài 4** — Chèn giữ thứ tự tăng:

```csharp
static void InsertSorted(LinkedList<int> list, int value)
{
    for (var n = list.First; n != null; n = n.Next)
    {
        if (n.Value >= value)
        {
            list.AddBefore(n, value);
            return;
        }
    }
    list.AddLast(value);
}
```

## 13. Đáp án thay thế

Bài 1 không Stack — hai con trỏ trên `char[]` (nhanh hơn, nhưng lệch mục tiêu chương). Bài 2 có thể dùng `List` + index modulo; Queue rõ ràng hơn về FIFO.

## 14. Thử thách

Implement `MiniLruCache<TKey, TValue>` với capacity cố định: `Get`/`Put` amortized O(1) dùng `Dictionary` + `LinkedList`.

## 15. Ứng dụng thực tế

- Message queue in-process, job worker
- Browser back stack / editor Undo
- BFS pathfinding (Queue), DFS (Stack)
- Undo history trong tool nội bộ

## 16. Liên hệ Unity

- `Queue` cho damage numbers / toast UI lần lượt
- `Stack` state machine UI (mở panel → Push; Back → Pop)
- Pathfinding grid: BFS với `Queue<Vector2Int>`
- Object pool đơn giản: `Queue<GameObject>` lấy/trả instance

## 17. Kiểm tra kiến thức

1. FIFO là cấu trúc nào?  
   **Đáp án:** Queue.

2. Pop trên Stack rỗng (không Try)?  
   **Đáp án:** `InvalidOperationException`.

3. Peek khác Pop chỗ nào?  
   **Đáp án:** Peek xem không xóa; Pop lấy và xóa.

4. LinkedList truy cập index i?  
   **Đáp án:** O(n) — phải duyệt từ đầu/cuối.

5. Vì sao không dùng `List.RemoveAt(0)` làm queue?  
   **Đáp án:** Mỗi lần O(n) do dịch phần tử.
