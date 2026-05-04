---
name: gitlab-flow
description: Standard end-to-end workflow for shipping a feature/bugfix from a Jira task to a merged GitLab MR. Use when the user references a Jira task ID (WRA-XX, etc.), asks to "start a task", "create branch from task", "create a merge request", "review the MR !N", "post review result to the MR", "fix all issues", or "merge the request". Covers branch naming, commit format, MR creation, review, fix loop, and merge.
---

# GitLab Flow (Jira → Code → MR → Merge)

Quy trình chuẩn cho một feature/bugfix mới. Có 2 vai trò: **Developer** (người làm task) và **Reviewer** (người review MR). Skill này hướng dẫn Claude thực hiện đúng từng bước theo prompt mà user gọi.

## Conventions

### Branch naming
- Feature: `feature/<TASK-ID>-<short-description-kebab>`
- Bugfix: `bugfix/<TASK-ID>-<short-description-kebab>`
- Hotfix: `hotfix/<TASK-ID>-<short-description-kebab>`
- Mô tả ngắn gọn, không dấu, dùng `-` ngăn cách. Ví dụ: `feature/WRA-40-Gioi-han-domain-account`

### Commit message
- Format: `<type>(<scope>): <subject> [<TASK-ID>]`
- type: `feat | fix | refactor | chore | docs | test | style`
- Ví dụ: `feat(auth): restrict login to allowed domains [WRA-40]`
- Body (tuỳ chọn): giải thích **why**, không lặp lại what
- Trailer: `Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>`

### Target branch
- MR luôn merge vào `main` trừ khi user chỉ định khác

## Triggers & Procedures

### "create branch <name>" hoặc "create branch from task <TASK-ID>"
1. Nếu user chỉ đưa TASK-ID, hỏi mô tả ngắn để gắn vào tên nhánh
2. Đảm bảo working tree sạch (`git status`); nếu có thay đổi chưa commit, hỏi user trước khi tiếp tục
3. Checkout `main`, pull về bản mới nhất
4. Tạo nhánh mới theo convention ở trên: `git checkout -b feature/<TASK-ID>-<desc>`
5. Báo lại tên nhánh đã tạo

### Sinh code từ mô tả task
- Khi user paste mô tả task Jira làm prompt, đọc kỹ và xác nhận lại scope trước khi code nếu có chỗ mơ hồ
- Code theo convention của project (tham khảo CLAUDE.md nếu có, hoặc đọc file gần khu vực sửa để bắt chước style)
- Không thêm tính năng/refactor ngoài scope task
- Sau khi xong, tóm tắt ngắn các file đã thay đổi

### "review the last change"
1. Chạy `git diff` (hoặc `git diff HEAD` nếu đã staged) để xem thay đổi gần nhất
2. Review theo các tiêu chí:
   - Logic đúng với mô tả task không
   - Có edge case nào chưa cover không
   - Có vi phạm convention/coding standard không
   - Có code thừa, dead code, hoặc abstraction không cần thiết
   - Có lỗ hổng bảo mật (input validation, auth bypass, injection) không
   - Có ảnh hưởng performance đáng kể không
3. Báo cáo dưới dạng danh sách có đánh số: `#1`, `#2`, ... để user dễ tham chiếu khi fix

### "Commit and push"
1. `git status` để xem files thay đổi
2. `git diff` xem nội dung
3. Soạn commit message theo convention (lấy TASK-ID từ tên nhánh hiện tại)
4. KHÔNG dùng `git add -A` mà liệt kê các file cụ thể
5. KHÔNG commit file nhạy cảm: `.env`, `credentials.*`, `*.key`, `*.pem`, file binary lớn
6. Tạo commit với HEREDOC để giữ định dạng message
7. `git push -u origin <branch>` (lần đầu) hoặc `git push` (lần sau)
8. Báo lại commit hash + URL push thành công

### "create a merge request" / "create an MR"
1. Đảm bảo đã push lên remote
2. Dùng `glab mr create`:
   ```bash
   glab mr create \
     --target-branch main \
     --title "<TASK-ID>: <subject>" \
     --description "<body>" \
     --remove-source-branch
   ```
3. Title MR = subject của commit gần nhất (hoặc tóm tắt nếu nhiều commit)
4. Description MR cần có:
   - **## Summary**: 1-3 bullet point về thay đổi
   - **## Test plan**: checklist test
   - **## Related**: link Jira task `[<TASK-ID>](<jira-url>)` nếu biết URL
5. Trả về URL của MR và số `!N`

### "review the MR !<N>" (vai trò Reviewer)
1. Yêu cầu `glab` CLI đã cài: kiểm tra `glab --version`
2. Lấy thông tin MR: `glab mr view <N> --json`
3. Lấy diff: `glab mr diff <N>`
4. Review theo cùng tiêu chí ở mục "review the last change"
5. Liệt kê issues theo dạng `#1`, `#2`, ...; mỗi issue cần có:
   - File + dòng
   - Vấn đề
   - Đề xuất fix
6. Đánh giá tổng thể: APPROVE / REQUEST_CHANGES / COMMENT

### "post review result to the MR"
1. Format kết quả review thành Markdown:
   ```markdown
   ## Review !<N>

   **Verdict:** REQUEST_CHANGES | APPROVE

   ### Issues
   - #1 `path/to/file.js:42` — <vấn đề>. Đề xuất: <fix>
   - #2 ...
   ```
2. Đăng comment: `glab mr note <N> --message "<markdown>"`
3. Nếu APPROVE: `glab mr approve <N>`

### "fix all issues" / "fix issue #<N>" / "fix issues #1, #2"
1. Đọc lại các issue đã raise (từ comment trên MR hoặc từ output review trước đó)
2. Nếu user chỉ định số issue → chỉ fix các issue đó
3. Nếu "fix all" → fix tất cả
4. Sau mỗi fix, verify ngắn (chạy test/build nếu có)
5. Khi hoàn tất, tự động chạy luôn flow "Commit and push" với commit message dạng:
   `fix(<scope>): address review issues #1,#2 [<TASK-ID>]`
6. Báo lại hash commit và xác nhận đã push

### "merge the request"
1. Kiểm tra MR đã có:
   - At least 1 approve
   - CI pipeline pass: `glab mr view <N>` (hoặc `glab ci status`)
   - Không có conflict
2. Nếu thiếu điều kiện, BÁO CHO USER và hỏi có override không (KHÔNG tự ý merge)
3. Merge: `glab mr merge <N> --remove-source-branch --squash`
4. Checkout về `main`, pull về bản mới nhất
5. Báo merge thành công + commit hash trên main

## Safety rules

- **KHÔNG force push** vào nhánh đã có MR mở (sẽ làm mất review history). Nếu phải sửa lịch sử, hỏi user trước
- **KHÔNG merge thẳng vào main** từ local — luôn qua MR
- **KHÔNG xoá nhánh** khác ngoài branch của MR vừa merge
- **KHÔNG bypass hooks** (`--no-verify`) trừ khi user yêu cầu rõ
- **KHÔNG commit secrets**: `.env`, key, token, password
- Nếu pre-commit hook fail: fix nguyên nhân và tạo commit MỚI, KHÔNG dùng `--amend`
- Khi `git status` cho thấy file lạ/branch lạ không quen thuộc, KHÔNG xoá — hỏi user xem có phải work-in-progress không

## Tools required

- `git` (luôn có)
- `glab` (GitLab CLI) — cần cho mục review/post comment/merge MR. Nếu chưa cài, hướng dẫn user: https://gitlab.com/gitlab-org/cli
