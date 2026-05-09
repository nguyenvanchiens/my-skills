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
| [`aspnet-core`](skills/aspnet-core/) | Build/debug/modernize ASP.NET Core: hosting, middleware, security, configuration, logging, deployment patterns trên .NET hiện hành. | [managedcode/dotnet-skills](https://github.com/managedcode/dotnet-skills) | MIT |
| [`aspnet-mvc`](skills/aspnet-mvc/) | ASP.NET Core MVC server-rendered: Razor Views, View Models, ModelState validation, Tag Helpers, View Components, Areas, PRG pattern. | Nội bộ | MIT |
| [`blazor`](skills/blazor/) | Blazor .NET 8+ unified: 5 render modes (Static SSR, Stream, InteractiveServer, InteractiveWebAssembly, InteractiveAuto), components, lifecycle, EditForm, JS interop, state management. | Nội bộ | MIT |
| [`web-api`](skills/web-api/) | Controller-based ASP.NET Core API: controller convention, model binding nâng cao, validation, OData, JsonPatch. | [managedcode/dotnet-skills](https://github.com/managedcode/dotnet-skills) | MIT |
| [`minimal-apis`](skills/minimal-apis/) | Thiết kế Minimal API trên ASP.NET Core 6+: handler-first endpoint, route group, filter, composition nhẹ. | [managedcode/dotnet-skills](https://github.com/managedcode/dotnet-skills) | MIT |
| [`entity-framework-core`](skills/entity-framework-core/) | EF Core 7+/8/9: modeling, migration, query translation, performance, DbContext lifetime. | [managedcode/dotnet-skills](https://github.com/managedcode/dotnet-skills) | MIT |
| [`optimizing-ef-core-queries`](skills/optimizing-ef-core-queries/) | Tối ưu query EF Core: fix N+1, chọn tracking mode, compiled queries, performance trap thường gặp. | [managedcode/dotnet-skills](https://github.com/managedcode/dotnet-skills) (mirror official MS) | MIT |
| [`modern-csharp`](skills/modern-csharp/) | C# hiện đại theo `LangVersion` của repo (đặc biệt C# 13/14): record, pattern matching, async, nullable refs. | [managedcode/dotnet-skills](https://github.com/managedcode/dotnet-skills) | MIT |
| [`dotnet`](skills/dotnet/) | Router skill cho .NET tổng quát — phân loại app model + cross-cutting concern, rồi chuyển tới skill .NET cụ thể. | [managedcode/dotnet-skills](https://github.com/managedcode/dotnet-skills) | MIT |
| [`xunit`](skills/xunit/) | Viết/chạy/sửa test xUnit (`[Fact]`, `[Theory]`, `xunit.v3`). Đúng CLI, package, runner trên VSTest hoặc Microsoft.Testing.Platform. | [managedcode/dotnet-skills](https://github.com/managedcode/dotnet-skills) | MIT |
| [`dotnet-testing-patterns`](skills/dotnet-testing-patterns/) | Test design: AAA, Moq/NSubstitute, FluentAssertions, `WebApplicationFactory`, Testcontainers, Bogus, `TimeProvider`/`FakeTimeProvider`, Builder pattern. Bổ sung cho `xunit` (focus runner). | Nội bộ | MIT |
| [`aspnet-logging`](skills/aspnet-logging/) | Structured logging với `ILogger<T>` + Serilog (sinks, enrichment, correlation ID), request logging middleware, sensitive data redaction, OpenTelemetry. | Nội bộ | MIT |
| [`aspnet-caching`](skills/aspnet-caching/) | `IMemoryCache` / `IDistributedCache` (Redis) / `HybridCache` (.NET 9), Output Caching, cache-aside, stampede prevention, TTL/invalidation strategies. | Nội bộ | MIT |
| [`aspnet-health-checks`](skills/aspnet-health-checks/) | Health probes cho K8s/Docker: tách `/healthz/live` vs `/healthz/ready`, AddHealthChecks() + DB/Redis/external probes, custom checks, UI dashboard. | Nội bộ | MIT |
| [`background-jobs`](skills/background-jobs/) | `BackgroundService`/`Channel<T>` (built-in), Hangfire (persistent + dashboard + retry), Quartz.NET (cron), pattern fire-and-forget/scheduled/recurring/continuation. | Nội bộ | MIT |
| [`aspnet-auth-advanced`](skills/aspnet-auth-advanced/) | JWT issue + refresh rotation, OAuth2/OIDC (Google/Microsoft/AzureAD), ASP.NET Identity scaffolding, multi-scheme (JWT+Cookie), policy/claim/role-based authorization, anti-forgery cho SPA. | Nội bộ | MIT |

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

#### Bundle Backend ASP.NET Core — Full Stack 16 skills (★ Recommended)

Bundle đầy đủ cho stack ASP.NET Core production-grade: framework + UI (MVC/Blazor) + API + ORM + observability (logging/caching/health checks) + background jobs + auth nâng cao + testing.

> Cài 1 lệnh là xong. Skill chỉ activate khi trigger pattern khớp → có nhiều skill không làm Claude chậm. Dung lượng ~3MB cho cả 16 SKILL.md. **Default cho hầu hết project ASP.NET Core mới.**

Cài cho project hiện tại:
```bash
npx skills add nguyenvanchiens/my-skills -s aspnet-core -s aspnet-mvc -s blazor -s web-api -s minimal-apis -s entity-framework-core -s optimizing-ef-core-queries -s modern-csharp -s dotnet -s aspnet-logging -s aspnet-caching -s aspnet-health-checks -s background-jobs -s aspnet-auth-advanced -s xunit -s dotnet-testing-patterns -y -a claude-code --copy
```

Cài global (recommended nếu bạn chuyên ASP.NET Core — mọi project mới tự có sẵn):
```bash
npx skills add nguyenvanchiens/my-skills -s aspnet-core -s aspnet-mvc -s blazor -s web-api -s minimal-apis -s entity-framework-core -s optimizing-ef-core-queries -s modern-csharp -s dotnet -s aspnet-logging -s aspnet-caching -s aspnet-health-checks -s background-jobs -s aspnet-auth-advanced -s xunit -s dotnet-testing-patterns -y -g -a claude-code --copy
```

<details>
<summary><b>Advanced — Bundle Core (8) hoặc Production (8) riêng</b></summary>

Chỉ dùng nếu cần lựa chọn theo từng bước hoặc muốn cài subset. Nếu không chắc → dùng Full Stack ở trên.

##### Bundle Core 8 — đủ cho dev cơ bản, POC, learn

Gồm: `aspnet-core`, `aspnet-mvc`, `blazor`, `web-api`, `minimal-apis`, `entity-framework-core`, `modern-csharp`, `dotnet`.

```bash
npx skills add nguyenvanchiens/my-skills -s aspnet-core -s aspnet-mvc -s blazor -s web-api -s minimal-apis -s entity-framework-core -s modern-csharp -s dotnet -y -a claude-code --copy
```

Global: thêm `-g` trước `-a claude-code`.

##### Bundle Production 8 — bổ sung cho app production-grade

> ⚠️ Bundle này KHÔNG tự đứng được — phải đã có Core cài trước. Skill production reference Core (vd `optimizing-ef-core-queries` reference `entity-framework-core`).

Gồm: `optimizing-ef-core-queries`, `aspnet-logging`, `aspnet-caching`, `aspnet-health-checks`, `background-jobs`, `aspnet-auth-advanced`, `xunit`, `dotnet-testing-patterns`.

```bash
npx skills add nguyenvanchiens/my-skills -s optimizing-ef-core-queries -s aspnet-logging -s aspnet-caching -s aspnet-health-checks -s background-jobs -s aspnet-auth-advanced -s xunit -s dotnet-testing-patterns -y -a claude-code --copy
```

Global: thêm `-g` trước `-a claude-code`.

</details>

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

> **Note**: hướng dẫn dùng chi tiết bên dưới chỉ tập trung cho `gitlab-flow` (skill chính của bộ này). Các skill còn lại (`karpathy-guidelines`, `impeccable`, `commit`, `review-branch`, các skill nhóm React/Vite stack: `tailwind-v4-shadcn`, `shadcnblocks-ui`, `aceternity-ui`, `react-best-practices`, `react-composition-patterns`, `react-hook-form-zod`, `theme-factory`, `web-artifacts-builder`, `vitest-testing`, và các skill nhóm .NET stack: `aspnet-core`, `web-api`, `minimal-apis`, `entity-framework-core`, `optimizing-ef-core-queries`, `modern-csharp`, `dotnet`, `xunit`) là standalone — cài rồi đọc `SKILL.md` của từng skill để biết cách dùng. Riêng `commit` và `review-branch` đã được tích hợp vào `gitlab-flow` qua các trigger `commit and push` và `review the whole branch` — nếu đã dùng `gitlab-flow` thì không cần cài lại.

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
    ├── <skill-name>/
    │   ├── SKILL.md          # bắt buộc — frontmatter `name` + `description`
    │   ├── LICENSE           # tùy chọn — giữ license gốc nếu fork
    │   ├── manifest.json     # tùy chọn — metadata bổ sung
    │   └── references/       # tùy chọn — markdown sâu cho deep-dive
    │       └── *.md
    └── ...
```

**Phân nhóm skills** (~22 skills, xem table ở trên cho mô tả):

| Nhóm | Skills |
|---|---|
| Workflow & quality | `gitlab-flow`, `commit`, `review-branch`, `karpathy-guidelines` |
| Frontend / Design | `impeccable`, `tailwind-v4-shadcn`, `shadcnblocks-ui`, `aceternity-ui`, `theme-factory`, `web-artifacts-builder` |
| React patterns | `react-best-practices`, `react-composition-patterns`, `react-hook-form-zod` |
| Frontend testing | `vitest-testing` |
| .NET — Core | `dotnet`, `aspnet-core`, `aspnet-mvc`, `blazor`, `web-api`, `minimal-apis`, `entity-framework-core`, `modern-csharp` |
| .NET — Production | `optimizing-ef-core-queries`, `aspnet-logging`, `aspnet-caching`, `aspnet-health-checks`, `background-jobs`, `aspnet-auth-advanced` |
| .NET — Testing | `xunit`, `dotnet-testing-patterns` |

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
- **Managed Code** ([managedcode/dotnet-skills](https://github.com/managedcode/dotnet-skills)) — nhóm .NET base (`aspnet-core`, `web-api`, `minimal-apis`, `entity-framework-core`, `optimizing-ef-core-queries`, `modern-csharp`, `dotnet`, `xunit`). Repo này embed sẵn các skill chính thức từ [dotnet/skills](https://github.com/dotnet/skills) (Microsoft) trong các thư mục `Official-DotNet-*`.
- **Nội bộ** — bổ sung cho stack ASP.NET Core production-grade: `aspnet-mvc`, `blazor`, `aspnet-logging`, `aspnet-caching`, `aspnet-health-checks`, `background-jobs`, `aspnet-auth-advanced`, `dotnet-testing-patterns`.

## License

Code và cấu hình của repo này: MIT (xem [LICENSE](LICENSE)).
Mỗi skill trong `skills/<name>/` giữ license riêng của tác giả gốc.
