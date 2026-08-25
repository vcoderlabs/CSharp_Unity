# Level 21 — C# for Unity (~40 giờ)

Level này đưa C# bạn đã học vào **Unity**: lifecycle `MonoBehaviour`, GameObject/Component, ScriptableObject, serialization, coroutine vs async, events, GC/pooling, Addressables, và kiến trúc (MVC/MVP/ECS + DI).

**Điều kiện:** Nền C# vững (khuyến nghị L1–L11, đặc biệt L3, L7, L10, L11). Nên hoàn thành [L19 Performance](../level-19-performance/) tư duy đo. Unity Editor **2022 LTS** hoặc **6** khuyến nghị.

**Tiếp theo:** [Capstone MMORPG](../capstone-mmorpg/).

---

## Mục tiêu cấp độ

Sau Level 21 bạn sẽ:

- Đặt đúng logic vào Awake/Start/Update/FixedUpdate/LateUpdate/OnDestroy
- Thiết kế data với Component + ScriptableObject
- Hiểu serialization Inspector và những bẫy thường gặp
- Chọn coroutine vs async đúng ngữ cảnh
- So UnityEvent vs C# event; tránh leak
- Giảm GC spike; implement object pool Unity
- Load nội dung với Addressables (khái niệm + workflow)
- Phác kiến trúc MVC/MVP/ECS và dùng VContainer hoặc Zenject cơ bản
- Hoàn thành mini game: pooling + events + DI

---

## Cảnh báo xuyên suốt Level 21

> **Unity chạy game loop.** Mọi thứ trong `Update` phải rẻ.  
> C# “đúng” ngoài Unity chưa đủ — allocation, reference Inspector, và lifetime scene quyết định bug.  
> Đừng học Unity bằng cách chỉ kéo thả mà không hiểu script lifecycle.

---

## 9 chương + 1 project

| # | File | Nội dung | ~Giờ |
|---|------|----------|------|
| 1 | [01-monobehaviour-lifecycle.md](./01-monobehaviour-lifecycle.md) | Lifecycle MonoBehaviour | 3–4 |
| 2 | [02-gameobject-component-scriptableobject.md](./02-gameobject-component-scriptableobject.md) | GO / Component / SO | 4 |
| 3 | [03-serialization-inspector.md](./03-serialization-inspector.md) | Serialization & Inspector | 3–4 |
| 4 | [04-coroutines-vs-async.md](./04-coroutines-vs-async.md) | Coroutines vs async | 4 |
| 5 | [05-unityevents-vs-csharp-events.md](./05-unityevents-vs-csharp-events.md) | UnityEvent vs C# event | 3–4 |
| 6 | [06-gc-in-unity.md](./06-gc-in-unity.md) | GC trong Unity | 4 |
| 7 | [07-object-pooling-unity.md](./07-object-pooling-unity.md) | Object pooling | 4 |
| 8 | [08-addressables.md](./08-addressables.md) | Addressables | 4 |
| 9 | [09-architecture-di.md](./09-architecture-di.md) | MVC/MVP/ECS + VContainer/Zenject | 5 |
| — | [project-unity-architecture.md](./project-unity-architecture.md) | Mini game architecture | 8–10 |

**Tổng ước lượng: ~40 giờ**

---

## Cách học đề xuất

1. Mỗi chương: scene trống + 1–2 script thực hành — đừng chỉ đọc.  
2. Bật Profiler sớm (chương 6–7).  
3. Project cuối: thiết kế trước (sơ đồ), rồi code thin MonoBehaviour.  
4. Capstone sẽ phóng to cùng pattern — làm project L21 nghiêm túc.

---

## Checklist hoàn thành Level 21

- [ ] Giải thích đúng thứ tự Awake/OnEnable/Start/Update…
- [ ] Tách data ScriptableObject khỏi runtime state
- [ ] Serialize field đúng; tránh bẫy null reference Inspector
- [ ] Viết được coroutine và biết hạn chế vs async
- [ ] Unsubscribe event / không leak
- [ ] Profiler thấy giảm GC sau pool
- [ ] Load 1 asset Addressables (async)
- [ ] DI đăng ký ≥ 2 service; MonoBehaviour mỏng
- [ ] Project mini game Done
- [ ] ≥ 4/5 kiểm tra kiến thức mỗi chương

Khi xong → **Capstone MMORPG**.
