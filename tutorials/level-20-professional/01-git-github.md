# Chương 1 — Git & GitHub

## 1. Mục tiêu học

- Thành thạo vòng đời: clone / branch / commit / push / pull request
- Viết commit message rõ “vì sao”
- Dùng `.gitignore` đúng cho .NET
- Hiểu nhánh `main` bảo vệ và review qua PR

## 2. Điều kiện tiên quyết

- Cài Git, tài khoản GitHub
- Biết dòng lệnh cơ bản
- Không cần biết hết Git nâng cao (rebase interactive…)

## 3. Khái niệm

**Git** = quản lý phiên bản phân tán trên máy bạn.  
**GitHub** = hosting remote + PR + Issues + Actions.

| Lệnh | Việc |
|------|------|
| `git status` | Xem thay đổi |
| `git add` | Stage |
| `git commit` | Lưu snapshot local |
| `git push` | Đẩy lên remote |
| `git pull` | Lấy + hợp nhất |
| `git checkout -b` / `git switch -c` | Nhánh mới |

**Pull Request (PR):** đề xuất merge nhánh → `main`, kèm mô tả + review.

## 4. Mô hình tư duy

```text
main ──────────────────────────● (protected)
  \                           /
   ●─●─● feature/login-api ─── PR review ─── merge
```

Mỗi commit = một ý thay đổi có thể giải thích.

## 5. Cú pháp

```bash
git init
git remote add origin git@github.com:USER/REPO.git

git switch -c feature/add-logging
git add .
git commit -m "$(cat <<'EOF'
Add structured logging for order pipeline.

EOF
)"
git push -u origin HEAD

# GitHub CLI
gh pr create --title "Add logging" --body "## Summary\n- ..."
```

`.gitignore` tối thiểu .NET:

```gitignore
bin/
obj/
.vs/
*.user
appsettings.*.local.json
.env
```

## 6. Ví dụ

### Cơ bản

```bash
git status
git diff
git log --oneline -5
```

### Trung cấp — sửa commit chưa push (chỉ khi bạn chắc)

```bash
git add file.cs
git commit --amend --no-edit   # cẩn thận: chỉ amend commit của bạn, chưa push
```

(Trong khóa này ưu tiên commit mới hơn amend phức tạp.)

### Nâng cao — conflict

```bash
git pull origin main
# sửa file conflict
git add .
git commit -m "Merge main into feature/x"
```

## 7. Lỗi thường gặp

| Hiện tượng | Nguyên nhân | Cách xử lý |
|------------|-------------|------------|
| Commit nhầm secret | Thiếu gitignore | Rotate key; dùng BFG/filter nếu đã push |
| Commit quá lớn | Gom nhiều ý | Tách commit / PR nhỏ |
| Force push `main` | Nguy hiểm | Tránh; bảo vệ branch |
| “Detached HEAD” | Checkout commit cũ | `git switch -c` nhánh mới |

## 8. Gỡ lỗi

1. `git status` luôn là bước đầu.
2. `git reflog` tìm commit “mất”.
3. Conflict: tìm marker `<<<<<<<`.
4. PR checks đỏ: đọc log CI trước khi hỏi người khác.

## 9. Best practices

- Nhánh ngắn đời, PR nhỏ.
- Message: imperative, nói lý do (“Fix null when cart empty”).
- Không commit `bin/obj`.
- `main` luôn build được.
- Review chính mình trên GitHub diff trước khi request review.

## 10. Bài tập

**Bài 1** — Tạo repo, 3 commit có ý nghĩa trên README.

**Bài 2** — Tạo nhánh, sửa file, mở PR (gh hoặc UI).

**Bài 3** — Thêm `.gitignore` .NET chuẩn, xác nhận `bin` không track.

**Bài 4** — Viết PR description theo mẫu Summary + Test plan.

## 11. Gợi ý

- Bài 1: init → add README → commit → đổi README → commit…
- Bài 2: `gh auth login` nếu cần.
- Bài 4: checklist test thủ công.

## 12. Đáp án

Không có một đoạn code duy nhất — checklist:

1. `git log` có ≥ 3 commit rõ ràng  
2. PR link mở được  
3. `git check-ignore -v bin/` có rule  
4. Body PR có Summary + Test plan  

## 13. Đáp án thay thế

GitLab / Azure DevOps cùng mô hình MR/PR. Conventional Commits (`feat:`, `fix:`) nếu team dùng.

## 14. Thử thách

Bật branch protection: require 1 review + status check `build` trước khi merge.

## 15. Ứng dụng thực tế

- Mọi team phần mềm
- Open source contribution
- Audit lịch sử khi incident

## 16. Liên hệ Unity

- `.gitignore` Unity: `Library/`, `Temp/`, `Logs/`, `Obj/`
- Không commit Library
- LFS cho asset lớn nếu cần
- Cùng PR discipline cho script C#

## 17. Kiểm tra kiến thức

1. `commit` khác `push` thế nào?  
   **Đáp án:** Commit lưu local; push gửi lên remote.

2. PR để làm gì?  
   **Đáp án:** Đề xuất merge có review và CI.

3. Vì sao ignore `bin/`?  
   **Đáp án:** Artifact build tái tạo được; làm bẩn repo.

4. Nhánh feature nên sống lâu không?  
   **Đáp án:** Không — càng lâu càng khó merge.

5. Secret bị commit phải làm gì trước?  
   **Đáp án:** Thu hồi/rotate secret; coi như đã lộ.
