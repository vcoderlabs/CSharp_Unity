# Chương 5 — CI/CD & code review

## 1. Mục tiêu học

- Viết **CI** tối thiểu: restore → build → test
- Hiểu CD khái niệm (deploy sau khi xanh)
- Review PR hiệu quả: đúng/sai/rủi ro
- Dùng checklist thay vì “LGTM” mù

## 2. Điều kiện tiên quyết

- Repo GitHub (chương 1)
- Solution build + có test (chương 2)

## 3. Khái niệm

**CI (Continuous Integration):** mỗi push/PR chạy tự động kiểm tra.  
**CD (Continuous Delivery/Deployment):** tự đóng gói / triển khai khi đạt điều kiện.

**Code review:** người khác (hoặc bạn sau 1 đêm) đọc diff trước khi vào `main`.

Mục tiêu review: correctness, readability, security, test, performance rủi ro — không nit style nếu máy đã format.

## 4. Mô hình tư duy

```text
Push nhánh → GitHub Actions
              ├─ build
              ├─ test
              └─ (optional) format/analyze
PR review humans ──► merge main ──► deploy staging/prod
```

## 5. Cú pháp

`.github/workflows/ci.yml`:

```yaml
name: ci
on:
  push:
    branches: [main]
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '8.0.x'
      - run: dotnet restore
      - run: dotnet build --no-restore -c Release
      - run: dotnet test --no-build -c Release --verbosity normal
```

## 6. Ví dụ

### Cơ bản — CI xanh/đỏ

Mở PR cố ý phá test → CI đỏ → sửa → xanh.

### Trung cấp — PR template

`.github/PULL_REQUEST_TEMPLATE.md`:

```markdown
## Summary
-

## Test plan
- [ ] Unit tests
- [ ] Manual: ...
```

### Nâng cao — review comment tốt

```text
Thay vì: "Không thích tên này"
Nên: "Đổi Place() → PlaceOrderAsync() vì còn gọi HTTP; dễ nhầm sync."
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| CI khác máy local | SDK version / OS | Pin SDK trong workflow |
| Test flaky | Race / thời gian | Sửa test; đừng rerun mãi |
| Review chỉ style | Thiếu tool format | `dotnet format` trong CI |
| Merge khi đỏ | Tắt protection | Bật required checks |

## 8. Gỡ lỗi

1. Đọc log step thất bại — thường dòng cuối.
2. Reproduce: `dotnet test -c Release` local giống CI.
3. Cache NuGet Actions nếu restore chậm (tối ưu sau).
4. Permission `GITHUB_TOKEN` khi cần push tag — cẩn thận.

## 9. Best practices

- CI < 10–15 phút cho repo học / app nhỏ.
- Required check trên `main`.
- Review nhỏ hơn 400 dòng dễ hơn.
- CD: bắt đầu deploy thủ công từ artifact; tự động hóa sau.
- Không commit credentials vào workflow; dùng GitHub Secrets.

## 10. Bài tập

**Bài 1** — Thêm `ci.yml`, push, xem Actions chạy.

**Bài 2** — Thêm PR template.

**Bài 3** — Tự review 1 PR cũ theo checklist cuối bài (tự viết 5 mục).

**Bài 4** — Cố ý fail test trên nhánh, xác nhận CI chặn merge (nếu bật protection).

## 11. Gợi ý

- Bài 1: YAML indent spaces.
- Bài 4: Settings → Branches → rule.

## 12. Đáp án

**Bài 1** — File như mục Cú pháp; badge xanh trên README optional.

Checklist review mẫu:

1. Behavior đúng requirement?  
2. Null/edge cases?  
3. Secret/config?  
4. Test kèm theo?  
5. Alloc/hot path nếu liên quan L19?

## 13. Đáp án thay thế

Azure Pipelines, GitLab CI, TeamCity. CD: Docker build → registry → K8s.

## 14. Thử thách

Thêm step `dotnet format --verify-no-changes` và sửa code đến khi xanh.

## 15. Ứng dụng thực tế

- Team ≥ 2 người bắt buộc
- Compliance audit (ai merge gì)
- Giảm “works on my machine”

## 16. Liên hệ Unity

- CI Unity cần license/agent nặng hơn — Cloud Build / GitHub runners đặc biệt
- Vẫn CI cho asmdef pure C# libraries
- Review script giống C# thường + kiểm tra serialize field break

## 17. Kiểm tra kiến thức

1. CI mang lại gì?  
   **Đáp án:** Phát hiện build/test hỏng sớm trên mọi PR.

2. CD khác CI?  
   **Đáp án:** CD liên quan đóng gói/triển khai liên tục sau khi tích hợp.

3. Review nên tập trung gì trước?  
   **Đáp án:** Đúng sai hành vi, rủi ro, test — không nit style đã có tool.

4. Flaky test nguy hiểm thế nào?  
   **Đáp án:** Làm mất niềm tin CI; dễ bỏ qua lỗi thật.

5. Secret trong workflow đặt đâu?  
   **Đáp án:** GitHub Secrets / OIDC — không hardcode.
