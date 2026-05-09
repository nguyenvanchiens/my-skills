# my-skills

Bộ sưu tập skills cá nhân cho Claude Code và các AI coding harness khác. Đóng gói để dùng nhanh ở mọi project chỉ bằng một lệnh `npx skills add`.

## Skills

| Skill | Mô tả | Nguồn gốc | License |
|---|---|---|---|
| [`karpathy-guidelines`](skills/karpathy-guidelines/) | 4 nguyên tắc giảm lỗi LLM khi code (Think Before Coding, Simplicity First, Surgical Changes, Goal-Driven Execution). | [forrestchang/andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills) | MIT |
| [`impeccable`](skills/impeccable/) | Hệ thống thiết kế frontend chuyên sâu (typography, color, spatial, motion, interaction, responsive, UX writing). Bao gồm 23 command như `/craft`, `/polish`, `/critique`, `/audit`, `/animate`. | [pbakaus/impeccable](https://github.com/pbakaus/impeccable) | Apache-2.0 |
| [`commit`](skills/commit/) | Tạo commit theo Conventional Commits, Jira ID ở cuối subject trong ngoặc đơn `(WRA-9)` (`/commit WRA-9`). Tự phân tích diff, chọn `type`/`scope`, có `--quick` mode, partial-staging guard, đọc `.commit-scopes` allowlist. **Standalone** — nếu đã cài `gitlab-flow` thì không cần cài thêm (gitlab-flow đã kế thừa toàn bộ spec này qua trigger `commit and push`). | Nội bộ | MIT |
| [`review-branch`](skills/review-branch/) | Review toàn bộ thay đổi của branch hiện tại so với `main` (committed + uncommitted) qua 3 agent song song: reuse, quality, efficiency — rồi tự fix issue. | Nội bộ | MIT |
| [`gitlab-flow`](skills/gitlab-flow/) | Quy trình end-to-end Jira → branch → commit → MR → review → fix → merge dùng `glab`. Chuẩn hoá branch naming, commit format và safety rules cho team GitLab. | Nội bộ | MIT |
| [`tailwind-v4-shadcn`](skills/tailwind-v4-shadcn/) | Setup Tailwind CSS v4 + shadcn/ui + **Vite** + React. Pattern `@theme inline`, CSS variable architecture, dark mode với `ThemeProvider`, gotchas khi migrate từ v3. | [secondsky/claude-skills](https://github.com/secondsky/claude-skills) | MIT |
| [`shadcnblocks-ui`](skills/shadcnblocks-ui/) | Kiến thức về 1,338 premium blocks + 1,189 free components từ ShadcnBlocks. Tự chọn block phù hợp khi user yêu cầu landing/dashboard/auth/ecommerce/navbar/footer. | [masonjames/Shadcnblocks-Skill](https://github.com/masonjames/Shadcnblocks-Skill) | MIT |
| [`aceternity-ui`](skills/aceternity-ui/) | 100+ animated React components (Aceternity UI) cho Tailwind. Hero parallax, 3D effects, motion-based interactions. | [secondsky/claude-skills](https://github.com/secondsky/claude-skills) | MIT |
| [`react-best-practices`](skills/react-best-practices/) | 57 quy tắc tối ưu performance React/Next.js từ Vercel Engineering, chia 8 nhóm theo impact để hướng dẫn refactor và sinh code. | [secondsky/claude-skills](https://github.com/secondsky/claude-skills) (Vercel) | MIT |
| [`react-composition-patterns`](skills/react-composition-patterns/) | Compound components, render props, context provider — pattern composition để component scale, tránh boolean prop proliferation. | [secondsky/claude-skills](https://github.com/secondsky/claude-skills) (Vercel) | MIT |
| [`react-hook-form-zod`](skills/react-hook-form-zod/) | Form type-safe với React Hook Form + Zod (`zodResolver`). Field arrays, multi-step forms, nested field, validation error pattern. | [secondsky/claude-skills](https://github.com/secondsky/claude-skills) | MIT |
| [`theme-factory`](skills/theme-factory/) | 10 theme preset (color palette + font pairing) cho slides, docs, HTML landing page. Có thể sinh theme mới on-the-fly. | [anthropics/skills](https://github.com/anthropics/skills) | Source-available |
| [`web-artifacts-builder`](skills/web-artifacts-builder/) | Bộ scripts build HTML artifact đa component (React + Tailwind + shadcn/ui) cho claude.ai — state management, routing, multi-component. | [anthropics/skills](https://github.com/anthropics/skills) | Source-available |
| [`vitest-testing`](skills/vitest-testing/) | Unit + integration test với Vitest (Vite-powered HMR, native ESM, mocking). Hợp project Vite. | [secondsky/claude-skills](https://github.com/secondsky/claude-skills) | MIT |

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

#### Workflow / code quality — tách cài từng skill

Các skill này dùng tùy task cụ thể, thường chỉ cần chọn 1-2 cái phù hợp:

```bash
npx skills add nguyenvanchiens/my-skills -s commit             -y -a claude-code --copy
npx skills add nguyenvanchiens/my-skills -s review-branch      -y -a claude-code --copy
npx skills add nguyenvanchiens/my-skills -s gitlab-flow        -y -a claude-code --copy
npx skills add nguyenvanchiens/my-skills -s karpathy-guidelines -y -a claude-code --copy
```

> **Lưu ý**: nếu cài `gitlab-flow` thì không cần cài `commit` + `review-branch` riêng (đã tích hợp qua trigger `commit and push` và `review the whole branch`).

Thêm `-g` vào cuối lệnh nếu muốn cài global (dùng cho mọi project).

#### Bundle UI/UX + React/Vite stack — 1 lệnh cài 10 skills

Bundle gồm: design system (`impeccable`), component library (`shadcnblocks-ui`, `aceternity-ui`), Tailwind v4 + shadcn setup (`tailwind-v4-shadcn`), React patterns (`react-best-practices`, `react-composition-patterns`), forms (`react-hook-form-zod`), theme (`theme-factory`), artifact builder (`web-artifacts-builder`), test (`vitest-testing`).

Cài cho project hiện tại:
```bash
npx skills add nguyenvanchiens/my-skills -s impeccable -s tailwind-v4-shadcn -s shadcnblocks-ui -s aceternity-ui -s react-best-practices -s react-composition-patterns -s react-hook-form-zod -s theme-factory -s web-artifacts-builder -s vitest-testing -y -a claude-code --copy
```

Cài global (dùng cho mọi project):
```bash
npx skills add nguyenvanchiens/my-skills -s impeccable -s tailwind-v4-shadcn -s shadcnblocks-ui -s aceternity-ui -s react-best-practices -s react-composition-patterns -s react-hook-form-zod -s theme-factory -s web-artifacts-builder -s vitest-testing -y -g -a claude-code --copy
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

> **Note**: hướng dẫn dùng chi tiết bên dưới chỉ tập trung cho `gitlab-flow` (skill chính của bộ này). Các skill còn lại (`karpathy-guidelines`, `impeccable`, `commit`, `review-branch`, và các skill nhóm React/Vite stack: `tailwind-v4-shadcn`, `shadcnblocks-ui`, `aceternity-ui`, `react-best-practices`, `react-composition-patterns`, `react-hook-form-zod`, `theme-factory`, `web-artifacts-builder`, `vitest-testing`) là standalone — cài rồi đọc `SKILL.md` của từng skill để biết cách dùng. Riêng `commit` và `review-branch` đã được tích hợp vào `gitlab-flow` qua các trigger `commit and push` và `review the whole branch` — nếu đã dùng `gitlab-flow` thì không cần cài lại.

## Sử dụng `gitlab-flow`

Skill này không phải `/slash command` mà kích hoạt bằng **trigger phrase tiếng Anh** trong prompt thường. Claude tự match phrase và chạy procedure tương ứng.

### Yêu cầu trước khi dùng

- `git` (luôn có)
- [`glab`](https://gitlab.com/gitlab-org/cli) — GitLab CLI. Trên Windows: `winget install GLab.GLab`
- Đăng nhập 1 lần: `glab auth login --hostname <gitlab-host>` (token scope `api` + `write_repository`)

### Bảng trigger

| Prompt | Hành động |
|---|---|
| `create branch from task <TASK-ID>` | Bóc tách task title → đề xuất 1-2 branch ngắn (2-4 từ key, ≤50 chars), **DỪNG hỏi user pick**, pull `main`, rồi mới `git checkout -b feature/<TASK-ID>-<short-desc>` |
| `rename branch <new-name>` | Đổi tên branch hiện tại đồng bộ local + remote. Detect upstream → nếu chưa push: rename thuần. Đã push: rename local + push tên mới + hỏi xóa branch cũ trên remote. Tránh tình trạng local≠remote name làm hỏng push/MR sau đó |
| (paste mô tả task Jira) | Đọc scope, sinh code theo convention project |
| `review the last change` | Chạy `git diff`, list issues `#1`, `#2`... |
| `review the whole branch` | Review cumulative branch vs `main` qua 3 agent song song (Reuse / Quality / Efficiency), tự fix issues. **Macro review** trước khi commit cuối / mở MR. |
| `commit and push` (kèm `--quick` nếu cần) | Self-contained — kế thừa toàn bộ spec của `/commit`: probe repo, partial-staging guard, atomic check, `.commit-scopes` allowlist, 11 types, footer (`Closes`/`Refs`...), Quick mode, WIP/Spike, revert format. TASK-ID tự lấy từ tên nhánh. Commit local xong **HỎI user** có push không (không tự push). Detect upstream tracking — nếu local branch khác upstream (rename scenario) → STOP, hướng user qua `rename branch`. Không cần cài skill `commit` riêng. |
| `create a merge request` | `glab mr create` với title/description chuẩn |
| `review the MR !<N>` | Lấy `glab mr diff <N>` + comment đã có. **MR chưa có comment** → review mới, list issues + verdict. **MR đã có comment** → review tiếp nối: đối chiếu issue cũ (`✓ Resolved` / `❌ Still open` / `⚠️ Partially`) + chỉ review commit mới push thêm |
| `post review result to the MR` | `glab mr note` đăng comment Markdown |
| `fix all issues` / `fix issue #<N>` | Fix các issue → tóm tắt + đề xuất commit message `fix(<scope>): address review issues #N (<TASK-ID>)` → **HỎI user xác nhận** trước khi commit/push (không tự động) |
| `merge the request` | Check approve + CI pass → `glab mr merge --squash --remove-source-branch` |

### Flow điển hình end-to-end

```
1. create branch from task WRA-40 giới hạn domain account
2. (paste mô tả task)              → Claude code
3. review the last change          → fix nếu cần (lặp 2↔3 nhiều lần)
4. commit and push                 (lặp 2-4 cho từng đoạn)
   ...
5. review the whole branch         → macro review + auto-fix, trước MR
6. commit and push                 → commit fix nếu /review-branch sửa gì
7. create a merge request

   --- chuyển sang vai Reviewer ---

8.  review the MR !21              → Claude in review ra terminal (chưa lên GitLab)
9.  (đọc, chỉnh nếu cần)
10. post review result to the MR   → mới đẩy comment lên GitLab

    --- quay lại vai Developer ---

11. fix all issues                 → fix xong, đợi user xác nhận
12. (xác nhận) commit and push
13. merge the request
```

> **Lưu ý**: bước 8 và 10 là **2 prompt riêng**, không tự động nối. Mục đích để reviewer xem trước nội dung review, có thể yêu cầu Claude bổ sung/sửa, mới quyết định post lên MR.

### Convention

- **Branch**: **default `feature/<TASK-ID>-<short-desc>`** cho mọi loại thay đổi (kể cả bug fix). User override bằng cách tự gõ `bugfix/...` hoặc `hotfix/...` (Mode A — skill respect nguyên si). Desc 2-4 từ key, kebab-case, không dấu, **tổng ≤50 chars**. Drop **type filler** (`Cai-tien`, `Improve`, `Fix`, `Sua`, `Tao`, `Add`, `Create`, `Them`, `Bo-sung`) nhưng **KEEP direction marker** (`Allow`, `Validate`, `Block`, `Duplicate`, `Stale`, `Missing`) **và context marker** (`Show`/`Display`, `Filter`/`Sort`, `Sync`/`Migrate`). Vd `feature/SMT-460-Allow-qty-0-checkin-checkout` (43), bug fix: `feature/HNCW-311-Duplicate-survey-log` (37)
- **Commit**: `<type>(<scope>): <subject> (<TASK-ID>)` (vd `feat(auth): restrict login to allowed domains (WRA-40)`)
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
- **Anthropic** ([anthropics/skills](https://github.com/anthropics/skills)) — `theme-factory`, `web-artifacts-builder`.
- **Mason James** ([@masonjames](https://github.com/masonjames)) — `shadcnblocks-ui` (Shadcn UI & ShadcnBlocks integration).
- **Secondsky / Claude Skills Maintainers** ([secondsky/claude-skills](https://github.com/secondsky/claude-skills)) — `tailwind-v4-shadcn`, `aceternity-ui`, `react-hook-form-zod`, `vitest-testing`.
- **Vercel Engineering** (qua secondsky) — `react-best-practices`, `react-composition-patterns`.

## License

Code và cấu hình của repo này: MIT (xem [LICENSE](LICENSE)).
Mỗi skill trong `skills/<name>/` giữ license riêng của tác giả gốc.
