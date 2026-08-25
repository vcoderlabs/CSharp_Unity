# C# MASTER LEARNING ROADMAP
### Từ Zero đến Unity/Game Development — Lộ trình 6-12 tháng

---

## Tổng quan Learning Track

```text
Level 0  → Programming Fundamentals        [~15h]
Level 1  → C# Fundamentals                 [~40h]
Level 2  → Object-Oriented Programming     [~35h]
Level 3  → Value Type vs Reference Type    [~15h]
Level 4  → Collections & Data Structures   [~25h]
Level 5  → Generics                        [~15h]
Level 6  → Exception & Error Handling      [~12h]
Level 7  → Delegates, Lambda & Events      [~20h]
Level 8  → LINQ                            [~25h]
Level 9  → Advanced C#                     [~30h]
Level 10 → Memory Management               [~20h]
Level 11 → Async / Await / Multithreading  [~30h]
Level 12 → File / IO / Serialization       [~15h]
Level 13 → Networking                      [~15h]
Level 14 → Testing                         [~15h]
Level 15 → Clean Code                      [~10h]
Level 16 → SOLID                           [~20h]
Level 17 → Design Patterns                 [~30h]
Level 18 → Architecture                    [~20h]
Level 19 → Performance                     [~15h]
Level 20 → Professional C#                 [~10h]
Level 21 → C# for Unity                    [~40h]
                                           ─────────
                            TOTAL         ~412 giờ
```

**Dependency chain (không thể bỏ qua thứ tự):**
`L0 → L1 → L2 → L3 → L4 → L5` là xương sống bắt buộc tuần tự. Từ L6 trở đi có thể học song song một phần (xem mục "Song song" bên dưới).

---

## LEVEL 0 — Programming Fundamentals *(~15h)*

**Mục tiêu:** Có tư duy lập trình cơ bản trước khi chạm vào C#.

**Chapters:**
1. Máy tính hoạt động thế nào (CPU, RAM, storage)
2. Program execution — source code → compiler → machine code/IL
3. Variables & Data — biểu diễn dữ liệu trong bộ nhớ
4. Expressions & Statements
5. Control Flow — if/else, loops
6. Functions — trừu tượng hóa hành vi
7. Input/Output cơ bản
8. Debugging fundamentals — đọc lỗi, đặt breakpoint
9. Algorithms cơ bản — tìm kiếm, sắp xếp đơn giản
10. Basic Data Structures (khái niệm, chưa code) — array, list, map
11. Time complexity cơ bản (Big O trực giác)

**Lessons:** ~11 | **Exercises:** ~35 (chủ yếu pseudo-code/tư duy) | **Project:** *Flowchart-to-Pseudocode Solver* (giải 5 bài toán logic bằng pseudocode, không cần chạy code thật)

**Bỏ qua nếu:** đã từng lập trình ngôn ngữ khác (Java, Python, C++...).

---

## LEVEL 1 — C# Fundamentals *(~40h)*

**Chapters:**
1. C# & .NET Ecosystem — SDK, CLR, Runtime, Compiler, `dotnet` CLI
2. Project & Solution structure, `.csproj`, namespace, `Main`
3. Variables, Constants, Type Inference (`var`)
4. Primitive Types — numeric types, `bool`, `char`, `string`, `enum`
5. Operators & Expressions
6. Control Flow — `if/else`, `switch`, loops (`for`, `while`, `do-while`, `foreach`), `break`/`continue`
7. Methods — parameters, return, overloading, scope
8. Nullable Value Types (`int?`)
9. String Handling & `StringBuilder`
10. Built-in utility types — `DateTime`, `TimeSpan`, `Math`, `Random`, `Console`

**Lessons:** ~28 | **Exercises:** ~9/lesson ≈ 250 | **Project (Level 1):** Console Calculator (hỗ trợ biểu thức nhiều toán tử, lịch sử, xử lý input sai)

---

## LEVEL 2 — Object-Oriented Programming *(~35h)*

**Chapters:**
1. Class, Object, Instance, Field, Property, Method
2. Constructor / Destructor (Finalizer)
3. Encapsulation & Access Modifiers
4. `static`, `readonly`, `const`
5. Inheritance — `base`, `this`
6. Polymorphism — `virtual`, `override`, `abstract`, `sealed`
7. Interface — thiết kế contract
8. Class vs Interface vs Abstract Class — khi nào dùng gì, khi nào inheritance là lựa chọn tệ (composition over inheritance)

**Lessons:** ~22 | **Exercises:** ~200 | **Project (Level 2):** Banking System (Account, SavingsAccount, CheckingAccount, TransactionHistory, interest calculation)

---

## LEVEL 3 — Value Type vs Reference Type *(~15h)*

**Chapters:**
1. Stack vs Heap (ASCII memory diagrams)
2. `struct` vs `class`
3. Boxing/Unboxing
4. Copying semantics — value vs reference semantics
5. Parameter passing — `ref`, `out`, `in`

**Lessons:** ~10 | **Exercises:** ~90 | **Project (Level 3):** Inventory System (dùng `struct` cho `ItemSlot` nhẹ, `class` cho `Inventory`, minh họa side-effect khi truyền tham chiếu sai cách)

---

## LEVEL 4 — Collections & Data Structures *(~25h)*

**Chapters:**
1. `Array`, `List<T>`
2. `Dictionary<TKey,TValue>`, `HashSet<T>`
3. `Queue<T>`, `Stack<T>`, `LinkedList<T>`
4. `SortedDictionary`, `SortedSet`
5. Interfaces: `IEnumerable`, `ICollection`, `IList`, `IReadOnlyCollection`
6. Collection expressions (C# 12)
7. So sánh performance/memory (Big O table cho từng collection)

**Lessons:** ~14 | **Exercises:** ~120 | **Project (Level 4):** Data Management System (import CSV → chọn cấu trúc dữ liệu phù hợp cho tra cứu/sắp xếp nhanh)

---

## LEVEL 5 — Generics *(~15h)*

**Chapters:**
1. Generic class/method/interface
2. Generic constraints (`where`, `struct`, `class`, `new()`, interface constraint)
3. Covariance/Contravariance (`in`/`out`)

**Lessons:** ~8 | **Exercises:** ~70 | **Project (Level 5):** Generic Collection Library (tự viết `GenericStack<T>`, `GenericRepository<T>`)

> ⚠️ **Không nên học lướt L0-L5** — đây là nền tảng bắt buộc, sai sót ở đây sẽ gây khó khăn nghiêm trọng ở Unity (đặc biệt struct vs class ảnh hưởng trực tiếp performance trong Unity).

---

## LEVEL 6 — Exception & Error Handling *(~12h)* 🔀 *học song song được với L7-L8*

**Chapters:** try/catch/finally, custom exceptions, exception hierarchy, exception filters, khi nào nên/không nên throw exception, logging chiến lược

**Lessons:** ~7 | **Exercises:** ~55 | **Project:** Logging System (custom exception hierarchy + file logger)

---

## LEVEL 7 — Delegates, Lambda & Events *(~20h)*

**Chapters:** delegate, `Action`/`Func`/`Predicate`, lambda, anonymous method, closure, event & event handler, multicast delegate

**Lessons:** ~10 | **Exercises:** ~85 | **Project:** Event-driven Notification System

> 🎯 Nền tảng trực tiếp cho UnityEvents/C# events trong Unity — không được học lướt.

---

## LEVEL 8 — LINQ *(~25h)*

**Chapters:**
1. Core operators: `Where`, `Select`, `SelectMany`, `OrderBy`/`ThenBy`, `GroupBy`, `Join`/`GroupJoin`, `Aggregate`
2. Element operators: `Any`, `All`, `Contains`, `First(OrDefault)`, `Single(OrDefault)`, `Last`, `Count`, `Sum`, `Average`, `Min`, `Max`
3. `Distinct`, `Skip`, `Take`, `Chunk`
4. Materialization: `ToList`, `ToArray`, `ToDictionary`
5. Deferred vs Immediate execution, `IEnumerable` vs `IQueryable`, multiple enumeration pitfalls

**Lessons:** ~12 | **Exercises:** ~100 | **Project:** Data Analytics Console Application

---

## LEVEL 9 — Advanced C# *(~30h)*

**Chapters:** Pattern matching & switch expressions, `record`/`record struct`, tuples & deconstruction, nullable reference types, attributes & reflection, `dynamic`, expression trees, extension methods, iterators/`yield`, index/range, local functions, anonymous types

**Lessons:** ~16 | **Exercises:** ~130 | **Project:** Reflection-based Serializer

---

## LEVEL 10 — Memory Management *(~20h)*

**Chapters:** CLR memory model, GC & generations (Gen0/1/2, LOH), object lifetime, `IDisposable`, `using`/`using declaration`, Dispose pattern, memory leaks (đặc biệt event leaks), object pooling → **liên hệ trực tiếp Unity GC spikes**

**Lessons:** ~10 | **Exercises:** ~75 | **Project:** Object Pooling System

> ⚠️ **Cực kỳ quan trọng cho Unity MMORPG** — GC spike là nguyên nhân hàng đầu gây giật lag trong game. Không học lướt.

---

## LEVEL 11 — Async / Await / Multithreading *(~30h)*

**Chapters:** Thread vs Task, `async`/`await` cơ chế thật sự (state machine), `Task<T>`, `ValueTask`, `CancellationToken`, `WhenAll`/`WhenAny`, `Parallel`, `lock`/`Monitor`/`Mutex`/`Semaphore`, concurrent collections, race condition, deadlock, synchronization context

**Lessons:** ~15 | **Exercises:** ~110 | **Project:** Async Download Manager

---

## LEVEL 12 — File / IO / Serialization *(~15h)* 🔀

**Chapters:** File/Directory, Stream/FileStream/MemoryStream, StreamReader/Writer, `System.Text.Json`, serialization/deserialization

**Lessons:** ~8 | **Exercises:** ~60 | **Project:** File-based Database

---

## LEVEL 13 — Networking *(~15h)* 🔀

**Chapters:** HTTP/HTTPS, `HttpClient`, REST API, WebSocket, TCP/UDP concepts — liên hệ game backend/multiplayer

**Lessons:** ~8 | **Exercises:** ~55 | **Project:** REST API Client

---

## LEVEL 14 — Testing *(~15h)* 🔀

**Chapters:** Unit vs Integration testing, Arrange-Act-Assert, mocking, Dependency Injection cơ bản, xUnit/NUnit

**Lessons:** ~8 | **Exercises:** ~55 | **Project:** Tested Application (viết test cho project L1-L8)

---

## LEVEL 15 — Clean Code *(~10h)*

**Chapters:** Naming, function/class design, code smell, refactoring, DRY/KISS/YAGNI, coupling & cohesion

**Lessons:** ~7 | **Exercises:** ~40

---

## LEVEL 16 — SOLID *(~20h)*

**Chapters:** SRP, OCP, LSP, ISP, DIP — mỗi nguyên tắc: bad code → problem → refactor → good code → Unity example

**Lessons:** ~5 (1/nguyên tắc, rất sâu) | **Exercises:** ~45

---

## LEVEL 17 — Design Patterns *(~30h)*

**Chapters:**
- Creational: Singleton, Factory, Abstract Factory, Builder, Prototype
- Structural: Adapter, Decorator, Facade, Proxy, Composite
- Behavioral: Observer, Strategy, Command, State, Template Method, Mediator, Chain of Responsibility
- Khi nào KHÔNG nên dùng pattern (over-engineering)

**Lessons:** ~17 | **Exercises:** ~150

---

## LEVEL 15-18 Project chung: Clean Architecture Application

---

## LEVEL 18 — Architecture *(~20h)*

**Chapters:** Layered architecture, Clean Architecture, DI & Dependency Inversion, Repository/Service pattern, DTO vs Entity, modular architecture

**Lessons:** ~8 | **Exercises:** ~50

---

## LEVEL 19 — Performance *(~15h)*

**Chapters:** Allocation cost, GC pressure, boxing, LINQ overhead, string allocation, algorithm complexity thực chiến, profiling & benchmarking (BenchmarkDotNet), caching, pooling

**Lessons:** ~8 | **Exercises:** ~45

---

## LEVEL 19-20 Project chung: Production-grade C# application

## LEVEL 20 — Professional C# *(~10h)* 🔀

**Chapters:** Git/GitHub, NuGet, project structure, coding standards, logging, config/environment, CI/CD basics, code review, debugging production issues

**Lessons:** ~8 | **Exercises:** ~30

---

## LEVEL 21 — C# for Unity *(~40h)*

**Chapters:**
1. Unity C# architecture, MonoBehaviour, script lifecycle (Awake/Start/Update/FixedUpdate/LateUpdate/OnDestroy)
2. GameObject/Component model, ScriptableObject
3. Serialization & Inspector
4. Coroutines vs async/await trong Unity
5. UnityEvents vs C# events/delegates
6. Garbage Collection trong Unity — cách tránh GC spike
7. Object Pooling áp dụng thực tế trong Unity
8. Addressables concepts
9. Architecture cho Unity project lớn (MVC/MVP/ECS overview), dependency management (VContainer/Zenject)

**Lessons:** ~18 | **Exercises:** ~140 | **Project:** Unity Architecture Project (mini game với pooling + event system + DI)

---

## CAPSTONE PROJECT — Unity MMORPG / Game Architecture

```text
Milestone 01 → Core Architecture       (DI container, service layer, game loop structure)
Milestone 02 → Entity System           (ECS-style hoặc component-based entity, generics, interfaces)
Milestone 03 → Inventory               (generic collections, SOLID, events)
Milestone 04 → Quest System            (state pattern, observer, serialization)
Milestone 05 → Combat System           (strategy pattern, command pattern, object pooling cho VFX/projectiles)
Milestone 06 → Networking              (REST/WebSocket client, async/await, serialization JSON)
Milestone 07 → Persistence             (save/load, file IO, serialization, versioning)
Milestone 08 → Optimization            (profiling, GC tuning, pooling audit, benchmarking)
```

Mỗi milestone có: Requirements → Architecture diagram → Tasks → Expected result → Exercises → Hints → Complete solution → Code review checklist.

---

## 3 LEARNING TRACKS

| | **Fast Track** | **Normal Track** | **Deep Track** |
|---|---|---|---|
| Thời lượng | 3 tháng | 6 tháng | 9–12 tháng |
| Giờ/ngày | 3–4h | 1.5–2h | 1–1.5h |
| Ngày/tuần | 6 | 5 | 5 |
| Lesson/day | 2–3 | 1 | 0.5–1 |
| Exercise/day | 8–10 | 4–5 | 3–4 |
| Review | Cuối mỗi Level | Cuối mỗi 2 Levels + cuối mỗi Level | Sau mỗi Chapter + Level |
| Phù hợp | Đã có nền tảng lập trình khác | Người mới, muốn chắc nền tảng | Người mới hoàn toàn, muốn hiểu sâu triệt để |
| Bỏ Level nào | L0 (nếu có nền), L13/L20 lướt nhanh | Không bỏ, có thể lướt L0/L13 | Không bỏ gì, đào sâu thêm L10/L11/L17 |

**Spaced repetition:** review sau 1 lesson (quick recall 5 câu) → sau 3 lessons (coding exercise tổng hợp) → sau 1 chapter (debugging exercise) → sau 1 level (refactoring exercise + interview questions).

---

## TỔNG KẾT

| Hạng mục | Số lượng |
|---|---|
| Tổng số Levels | 22 (Level 0–21) |
| Tổng số Chapters | ~150 |
| Ước lượng Lessons | ~230 |
| Ước lượng Exercises | ~1,850 (9/lesson trung bình) |
| Số Projects (theo Level) | 20 |
| Capstone Project | 1 (Unity MMORPG Architecture, 8 milestones) |

**1. Bắt buộc trước Unity:** L1–L11 (Fundamentals → OOP → Value/Reference Type → Collections → Generics → Delegates/Events → Memory Management → Async). Đây là xương sống không thể thiếu vì Unity dùng trực tiếp các khái niệm này (struct vs class ảnh hưởng performance, delegates = UnityEvents, GC = spike lag, async = loading).

**2. Có thể học song song:** L6 (Exception), L12 (File/IO), L13 (Networking), L14 (Testing), L20 (Professional C#) — không phụ thuộc chặt vào nhau, có thể xen kẽ.

**3. Cần học sâu cho Unity MMORPG:** L10 (Memory/GC — quan trọng nhất cho performance), L11 (Async/Multithreading — networking realtime), L13 (Networking), L17 (Design Patterns — Object Pool, Observer, State, Command đều dùng liên tục trong game), L18 (Architecture — quản lý codebase lớn nhiều hệ thống).

**4. Có thể học lướt:** L0 (nếu đã biết lập trình), L13-Networking chi tiết giao thức thấp (TCP/UDP raw), L14-Testing (framework cụ thể, chỉ cần hiểu khái niệm), L20 (CI/CD chi tiết — có thể học khi cần).

**5. Tuyệt đối không nên học lướt:** L2 (OOP), L3 (Value/Reference Type — sai ở đây gây bug khó debug nhất), L4 (Collections — performance-critical), L7 (Delegates/Events — nền Unity events), L10 (Memory/GC — sống còn cho game performance), L16 (SOLID) & L17 (Design Patterns — quyết định codebase MMORPG có scale được hay không).

---

## Trạng thái tiến độ

- [x] Level 0 — Programming Fundamentals → `tutorials/level-00-programming-fundamentals/`
- [x] Level 1 — C# Fundamentals → `tutorials/level-01-csharp-fundamentals/`
- [x] Level 2 — Object-Oriented Programming → `tutorials/level-02-oop/`
- [x] Level 3 — Value Type vs Reference Type → `tutorials/level-03-value-vs-reference/`
- [x] Level 4 — Collections & Data Structures → `tutorials/level-04-collections/`
- [x] Level 5 — Generics → `tutorials/level-05-generics/`
- [x] Level 6 — Exception & Error Handling → `tutorials/level-06-exceptions/`
- [x] Level 7 — Delegates, Lambda & Events → `tutorials/level-07-delegates-events/`
- [x] Level 8 — LINQ → `tutorials/level-08-linq/`
- [x] Level 9 — Advanced C# → `tutorials/level-09-advanced-csharp/`
- [x] Level 10 — Memory Management → `tutorials/level-10-memory/`
- [ ] Level 11 — Async / Await / Multithreading → `tutorials/level-11-async/` (khung README)
- [ ] Level 12 — File / IO / Serialization → `tutorials/level-12-file-io/` (khung README)
- [ ] Level 13 — Networking → `tutorials/level-13-networking/` (khung README)
- [ ] Level 14 — Testing → `tutorials/level-14-testing/` (khung README)
- [ ] Level 15 — Clean Code → `tutorials/level-15-clean-code/` (khung README)
- [ ] Level 16 — SOLID → `tutorials/level-16-solid/` (khung README)
- [ ] Level 17 — Design Patterns → `tutorials/level-17-design-patterns/` (khung README)
- [ ] Level 18 — Architecture → `tutorials/level-18-architecture/` (khung README)
- [ ] Level 19 — Performance → `tutorials/level-19-performance/` (khung README)
- [ ] Level 20 — Professional C# → `tutorials/level-20-professional/` (khung README)
- [ ] Level 21 — C# for Unity → `tutorials/level-21-unity/` (khung README)
- [ ] Capstone Project — Unity MMORPG / Game Architecture → `tutorials/capstone-mmorpg/` (khung README)

**Bước tiếp theo:** Đã có bài L0–10. Tiếp theo batch L11–15, rồi L16–21 + Capstone. Mỗi 5 level cập nhật tiến độ và commit.
