# my-skills

Bộ sưu tập skills cá nhân cho Claude Code và các AI coding harness khác. Đóng gói để dùng nhanh ở mọi project chỉ bằng một lệnh `npx skills add`.

## Skills

| Skill | Mô tả | Nguồn gốc | License |
|---|---|---|---|
| [`karpathy-guidelines`](skills/karpathy-guidelines/) | 4 nguyên tắc giảm lỗi LLM khi code (Think Before Coding, Simplicity First, Surgical Changes, Goal-Driven Execution). | [forrestchang/andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills) | MIT |
| [`impeccable`](skills/impeccable/) | Hệ thống thiết kế frontend chuyên sâu (typography, color, spatial, motion, interaction, responsive, UX writing). Bao gồm 23 command như `/craft`, `/polish`, `/critique`, `/audit`, `/animate`. | [pbakaus/impeccable](https://github.com/pbakaus/impeccable) | Apache-2.0 |
| [`commit`](skills/commit/) | Tạo commit theo Conventional Commits với Jira ID đứng đầu dòng subject (`/commit WRA-9`). Tự phân tích diff, chọn `type`/`scope`, soạn message tiếng Việt. | Nội bộ | MIT |
| [`review-branch`](skills/review-branch/) | Review toàn bộ thay đổi của branch hiện tại so với `main` (committed + uncommitted) qua 3 agent song song: reuse, quality, efficiency — rồi tự fix issue. | Nội bộ | MIT |
| [`gitlab-flow`](skills/gitlab-flow/) | Quy trình end-to-end Jira → branch → commit → MR → review → fix → merge dùng `glab`. Chuẩn hoá branch naming, commit format và safety rules cho team GitLab. | Nội bộ | MIT |

## Cài đặt

Yêu cầu: Node.js (để dùng `npx`).

### Cài tất cả skills cho project hiện tại

```bash
npx skills add nguyenvanchiens/my-skills --all -a claude-code --copy
```

### Cài riêng cho Claude
```bash
npx skills add nguyenvanchiens/my-skills -s "*" -y -a claude-code --copy
```

### Cài global (dùng cho mọi project)

```bash
npx skills add nguyenvanchiens/my-skills --all -g -a claude-code --copy
```
### Cài chọn lọc

Liệt kê skills có trong repo:
```bash
npx skills add nguyenvanchiens/my-skills -l
```

Cài skill cụ thể (thêm `-g` nếu muốn cài global):
```bash
npx skills add nguyenvanchiens/my-skills -s karpathy-guidelines -y -a claude-code --copy
npx skills add nguyenvanchiens/my-skills -s impeccable        -y -a claude-code --copy
npx skills add nguyenvanchiens/my-skills -s commit            -y -a claude-code --copy
npx skills add nguyenvanchiens/my-skills -s review-branch     -y -a claude-code --copy
npx skills add nguyenvanchiens/my-skills -s gitlab-flow       -y -a claude-code --copy
```

Cài nhiều skill cùng lúc (lặp cờ `-s`):
```bash
npx skills add nguyenvanchiens/my-skills -s commit -s review-branch -y -a claude-code --copy
```

### Cập nhật skills lên bản mới

```bash
npx skills update         # cập nhật tất cả
npx skills update -g      # chỉ global
```

### Gỡ cài đặt

```bash
npx skills remove
```

## Hỗ trợ harness khác

Cờ `-a` chấp nhận: `claude-code`, `cursor`, `gemini-cli`, `codex`, `opencode`, `windsurf`, `copilot`, ... (xem `npx skills --help` cho danh sách đầy đủ).

```bash
# Cursor
npx skills add nguyenvanchiens/my-skills --all -a cursor --copy

# Tất cả harness phát hiện được
npx skills add nguyenvanchiens/my-skills --all -a "*" --copy
```

## Sử dụng `gitlab-flow`

Skill này không phải `/slash command` mà kích hoạt bằng **trigger phrase tiếng Anh** trong prompt thường. Claude tự match phrase và chạy procedure tương ứng.

### Yêu cầu trước khi dùng

- `git` (luôn có)
- [`glab`](https://gitlab.com/gitlab-org/cli) — GitLab CLI. Trên Windows: `winget install GLab.GLab`
- Đăng nhập 1 lần: `glab auth login --hostname <gitlab-host>` (token scope `api` + `write_repository`)

### Bảng trigger

| Prompt | Hành động |
|---|---|
| `create branch from task <TASK-ID>` | Pull `main`, tạo nhánh `feature/<TASK-ID>-<desc>` theo convention |
| (paste mô tả task Jira) | Đọc scope, sinh code theo convention project |
| `review the last change` | Chạy `git diff`, list issues `#1`, `#2`... |
| `commit and push` | Commit `<type>(<scope>): <subject> [<TASK-ID>]` rồi push |
| `create a merge request` | `glab mr create` với title/description chuẩn |
| `review the MR !<N>` | `glab mr diff <N>`, list issues, verdict APPROVE/REQUEST_CHANGES |
| `post review result to the MR` | `glab mr note` đăng comment Markdown |
| `fix all issues` / `fix issue #<N>` | Fix → commit `fix(<scope>): address review issues #N [<TASK-ID>]` → push |
| `merge the request` | Check approve + CI pass → `glab mr merge --squash --remove-source-branch` |

### Flow điển hình end-to-end

```
1. create branch from task WRA-40 giới hạn domain account
2. (paste mô tả task)              → Claude code
3. review the last change          → fix nếu cần
4. commit and push
5. create a merge request

   --- chuyển sang vai Reviewer ---

6. review the MR !21               → Claude in review ra terminal (chưa lên GitLab)
7. (đọc, chỉnh nếu cần)
8. post review result to the MR    → mới đẩy comment lên GitLab

   --- quay lại vai Developer ---

9. fix all issues                  → tự commit + push
10. merge the request
```

> **Lưu ý**: bước 6 và 8 là **2 prompt riêng**, không tự động nối. Mục đích để reviewer xem trước nội dung review, có thể yêu cầu Claude bổ sung/sửa, mới quyết định post lên MR.

### Convention

- **Branch**: `feature/<TASK-ID>-<desc>` | `bugfix/<TASK-ID>-<desc>` | `hotfix/<TASK-ID>-<desc>`
- **Commit**: `<type>(<scope>): <subject> [<TASK-ID>]` (vd `feat(auth): restrict login to allowed domains [WRA-40]`)
- **Target branch**: mặc định `main` — nếu repo dùng `master`, thêm dòng vào `CLAUDE.md` của project: `Default branch: master (not main)`

### Safety rules

- KHÔNG force push vào nhánh đã có MR mở
- KHÔNG merge thẳng vào `main` từ local — luôn qua MR
- KHÔNG bypass hooks (`--no-verify`) trừ khi user yêu cầu rõ
- KHÔNG commit secrets (`.env`, key, token, password)

Xem chi tiết đầy đủ ở [`skills/gitlab-flow/SKILL.md`](skills/gitlab-flow/SKILL.md).

## Cấu trúc repo

```
my-skills/
├── README.md
├── LICENSE
└── skills/
    ├── karpathy-guidelines/
    │   ├── SKILL.md
    │   └── LICENSE
    ├── impeccable/
    │   ├── SKILL.md
    │   ├── LICENSE
    │   ├── agents/
    │   ├── reference/
    │   └── scripts/
    ├── commit/
    │   └── SKILL.md
    ├── review-branch/
    │   └── SKILL.md
    └── gitlab-flow/
        └── SKILL.md
```

Khi thêm skill mới: tạo thư mục `skills/<skill-name>/` chứa `SKILL.md` (có frontmatter `name`, `description`). CLI `npx skills` sẽ tự nhận diện.

## Ghi nhận

Repo này tổng hợp lại các skill open-source xuất sắc từ cộng đồng. Toàn bộ tác quyền và LICENSE gốc được giữ nguyên trong từng thư mục skill. Cảm ơn các tác giả gốc:

- **Andrej Karpathy** — quan sát ban đầu về lỗi LLM khi code.
- **Forrest Chang** ([@forrestchang](https://github.com/forrestchang)) — đóng gói Karpathy Guidelines thành skill.
- **Paul Bakaus** ([@pbakaus](https://github.com/pbakaus)) — Impeccable design system.

## License

Code và cấu hình của repo này: MIT (xem [LICENSE](LICENSE)).
Mỗi skill trong `skills/<name>/` giữ license riêng của tác giả gốc.
