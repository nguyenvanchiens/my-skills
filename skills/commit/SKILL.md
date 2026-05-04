---
name: commit
description: Commit staged + working-tree changes following Conventional Commits, with the Jira ID as the first token on the subject line. Takes the Jira ID as an argument, e.g. `/commit WRA-9`.
---

# Commit theo Conventional Commits + Jira ID

Lấy Jira ID từ tham số của skill (`args`). Ví dụ: `/commit WRA-9` → `args = "WRA-9"`. Nếu `args` trống, dừng lại và hỏi người dùng Jira ID trước khi làm tiếp.

## Quy trình

1. **Kiểm tra trạng thái repo** — chạy song song (một message, nhiều tool call):
   - `git status` (KHÔNG dùng cờ `-uall`)
   - `git diff HEAD` (bao gồm cả phần đã staged và phần chưa staged — mọi thứ sẽ được commit)
   - `git log -n 5 --oneline` để nắm convention hiện tại của repo

2. **Nếu không có thay đổi nào (không có file untracked và không có modification), KHÔNG tạo commit rỗng.** Báo cho người dùng và dừng.

3. **Phân tích thay đổi** và soạn commit message:
   - Xác định `type` từ bảng bên dưới dựa trên bản chất thay đổi thực tế.
   - Chọn `scope` ngắn gọn (thường là tên module / thư mục chính bị ảnh hưởng, ví dụ `auth`, `admin`, `reports`). Có thể bỏ `scope` nếu đổi trải rộng nhiều module.
   - **Jira ID BẮT BUỘC nằm ở vị trí đầu tiên của dòng summary**, trước cả `<type>`. Format: `<JIRA-ID> <type>(<scope>): <subject>`. Không lặp lại Jira ID ở footer.
   - `subject` viết ngắn, imperative (dạng mệnh lệnh), **KHÔNG** chấm cuối, ưu tiên tiếng Việt nếu các commit trước trong repo đang dùng tiếng Việt (kiểm tra `git log`).
   - `body` (tùy chọn) giải thích *why* hơn là *what*; tránh mô tả chi tiết diff.
   - Nếu là breaking change, thêm `!` sau `type(scope)` (vd `feat(api)!:`) và thêm footer `BREAKING CHANGE: <mô tả>`.

4. **Không commit các file có khả năng chứa secrets** (`.env`, `credentials.json`, key files, v.v.). Nếu người dùng chủ động yêu cầu commit các file này, cảnh báo trước.

5. **Stage đúng file cần thiết** — liệt kê từng file bằng tên thay vì `git add -A` / `git add .` để tránh lôi nhầm file nhạy cảm hay binary lớn.

6. **Tạo commit** bằng HEREDOC để giữ format. `<scope>` là tùy chọn — nếu thay đổi trải rộng nhiều module thì bỏ luôn phần `(<scope>)`:

```bash
# Dạng có scope
git commit -m "$(cat <<'EOF'
<JIRA-ID> <type>(<scope>): <subject>

<body tùy chọn>
EOF
)"

# Dạng không scope
git commit -m "$(cat <<'EOF'
<JIRA-ID> <type>: <subject>

<body tùy chọn>
EOF
)"
```

KHÔNG tự động chèn dòng `Co-Authored-By:` vào commit — repo này không dùng convention đó.

7. **Sau khi commit**, chạy `git status` để xác nhận commit đã tạo thành công.

8. **Nếu pre-commit hook fail**, KHÔNG dùng `--amend` (commit trước đó đã fail nên chưa tồn tại). Sửa lỗi, stage lại, tạo commit mới.

9. **KHÔNG push** trừ khi người dùng yêu cầu rõ ràng.

## Các `type` được phép

| Type | Ý nghĩa | Version bump |
|------|---------|--------------|
| `feat` | Tính năng mới | MINOR (1.X.0) |
| `fix` | Sửa bug | PATCH (1.0.X) |
| `perf` | Cải thiện hiệu năng | PATCH |
| `refactor` | Refactor không đổi behavior | — |
| `docs` | Tài liệu | — |
| `test` | Thêm/sửa test | — |
| `chore` | Maintenance, build config | — |
| `ci` | CI/CD config | — |

> **Breaking change** là một *modifier*, không phải type riêng. Bất kỳ type nào có hậu tố `!` (vd `feat(api)!:`) hoặc có footer `BREAKING CHANGE: <mô tả>` đều kích hoạt MAJOR bump (X.0.0).

## Ví dụ tham chiếu

```
WRA-201 feat(auth): thêm JWT refresh token rotation

Implement sliding expiration cho refresh token, revoke
token cũ khi phát hiện reuse.
```

```
WRA-334 fix(billing): tính sai VAT cho đơn hàng có discount
```

```
WRA-412 refactor(order): tách OrderService thành các handler nhỏ

Không đổi behavior, chuẩn bị cho việc thêm payment provider.
```

```
WRA-450 feat(api)!: đổi response format endpoint /users

BREAKING CHANGE: field `user_id` đổi thành `id`. Clients
phải cập nhật trước khi deploy.
```

## Lưu ý quan trọng

- Nhiều thay đổi thuộc các `type` khác nhau → chia thành nhiều commit khi hợp lý, mỗi commit một `type`. Nếu người dùng muốn một commit duy nhất, chọn `type` phản ánh thay đổi chủ đạo.
- Trước khi commit, hỏi lại người dùng nếu có nghi ngờ về việc gộp/tách commit.
- Tuân thủ Git Safety Protocol của Claude Code (không `--no-verify`, không force push, không sửa git config).
