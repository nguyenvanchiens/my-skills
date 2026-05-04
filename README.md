# my-skills

Bộ sưu tập skills cá nhân cho Claude Code và các AI coding harness khác. Đóng gói để dùng nhanh ở mọi project chỉ bằng một lệnh `npx skills add`.

## Skills

| Skill | Mô tả | Nguồn gốc | License |
|---|---|---|---|
| [`karpathy-guidelines`](skills/karpathy-guidelines/) | 4 nguyên tắc giảm lỗi LLM khi code (Think Before Coding, Simplicity First, Surgical Changes, Goal-Driven Execution). | [forrestchang/andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills) | MIT |
| [`impeccable`](skills/impeccable/) | Hệ thống thiết kế frontend chuyên sâu (typography, color, spatial, motion, interaction, responsive, UX writing). Bao gồm 23 command như `/craft`, `/polish`, `/critique`, `/audit`, `/animate`. | [pbakaus/impeccable](https://github.com/pbakaus/impeccable) | Apache-2.0 |
| [`commit`](skills/commit/) | Tạo commit theo Conventional Commits với Jira ID đứng đầu dòng subject (`/commit WRA-9`). Tự phân tích diff, chọn `type`/`scope`, soạn message tiếng Việt. | Nội bộ | MIT |
| [`review-branch`](skills/review-branch/) | Review toàn bộ thay đổi của branch hiện tại so với `main` (committed + uncommitted) qua 3 agent song song: reuse, quality, efficiency — rồi tự fix issue. | Nội bộ | MIT |

## Cài đặt

Yêu cầu: Node.js (để dùng `npx`).

### Cài tất cả skills cho project hiện tại

```bash
npx skills add nguyenvanchiens/my-skills --all -a claude-code --copy
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
    └── review-branch/
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
