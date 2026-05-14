# my-skills

Bộ sưu tập skills cá nhân cho Claude Code và các AI coding harness khác. Đóng gói để dùng nhanh ở mọi project chỉ bằng một lệnh `npx skills add`.

## Skills

| Skill | Mô tả | Nguồn gốc | License |
|---|---|---|---|
| [`karpathy-guidelines`](skills/karpathy-guidelines/) | 4 nguyên tắc giảm lỗi LLM khi code (Think Before Coding, Simplicity First, Surgical Changes, Goal-Driven Execution). | [forrestchang/andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills) | MIT |
| [`impeccable`](skills/impeccable/) | Hệ thống thiết kế frontend chuyên sâu (typography, color, spatial, motion, interaction, responsive, UX writing). Bao gồm 23 command như `/craft`, `/polish`, `/critique`, `/audit`, `/animate`. | [pbakaus/impeccable](https://github.com/pbakaus/impeccable) | Apache-2.0 |
| [`gitlab-sync`](skills/gitlab-sync/) | Resolve conflict khi sync `main → builds/dev/<app>` để deploy QA. Hỗ trợ monorepo multi-app (`portal-web-admin`, `gift-api`, ...). Dùng nhánh trung gian `sync/*` để giữ code chảy 1 chiều — không bao giờ leak `builds/*` ngược về `main`. Pair với `gitlab-flow` (đã tách sang repo riêng [`my-skills-gitlab-flow`](https://github.com/nguyenvanchiens/my-skills-gitlab-flow)). | Nội bộ | MIT |
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
npx skills add nguyenvanchiens/my-skills -s gitlab-sync        -y -a claude-code --copy
npx skills add nguyenvanchiens/my-skills -s karpathy-guidelines -y -a claude-code --copy
```

> **Lưu ý**: Các skill GitLab workflow (`gitlab-flow`, `commit`, `review-branch`) đã được tách sang repo riêng [`nguyenvanchiens/my-skills-gitlab-flow`](https://github.com/nguyenvanchiens/my-skills-gitlab-flow). Cài qua:
> ```bash
> npx skills add nguyenvanchiens/my-skills-gitlab-flow -s gitlab-flow -y -a claude-code --copy
> ```
> `gitlab-sync` (vẫn ở repo này) là **add-on** cho `gitlab-flow` — chỉ cần cài thêm nếu team dùng nhánh `builds/dev/<app>` để trigger deploy QA và cần resolve conflict `main → builds/dev`.

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

> **Note**: hướng dẫn dùng chi tiết bên dưới tập trung cho `gitlab-sync`. Các skill còn lại (`karpathy-guidelines`, `impeccable`, các skill nhóm React/Vite stack: `tailwind-v4-shadcn`, `shadcnblocks-ui`, `aceternity-ui`, `react-best-practices`, `react-composition-patterns`, `react-hook-form-zod`, `theme-factory`, `web-artifacts-builder`, `vitest-testing`, và các skill nhóm .NET stack: `aspnet-core`, `web-api`, `minimal-apis`, `entity-framework-core`, `optimizing-ef-core-queries`, `modern-csharp`, `dotnet`, `xunit`) là standalone — cài rồi đọc `SKILL.md` của từng skill để biết cách dùng. Skills `gitlab-flow`, `commit`, `review-branch` đã được tách sang repo riêng [`my-skills-gitlab-flow`](https://github.com/nguyenvanchiens/my-skills-gitlab-flow) — xem README repo đó để biết cách dùng.

## Sử dụng `gitlab-sync` (deploy QA cho monorepo multi-app)

Skill pair với `gitlab-flow` (xem [repo riêng](https://github.com/nguyenvanchiens/my-skills-gitlab-flow)). Sau khi feature merged main qua `gitlab-flow`, Maintainer dùng `gitlab-sync` để đưa code từ `main` lên các nhánh `builds/dev/<app>` trigger deploy QA — đặc biệt khi 2 nhánh bị conflict.

### Khi nào cần `gitlab-sync`

- Team dùng convention `main → builds/dev/<app>` để trigger CI/CD deploy QA
- Project là **monorepo multi-app** (có nhiều nhánh build dạng `builds/dev/portal-web-admin`, `builds/dev/gift-api`, `builds/dev/portal-api`...)
- `main → builds/dev/<app>` thỉnh thoảng bị conflict, cần resolve mà không leak code `builds/*` ngược về `main`

Nếu team chỉ có 1 build branch hoặc không dùng convention này → không cần `gitlab-sync`.

### Sơ đồ flow (4 bước)

```
main ──────●─────────────●  (giữ nguyên, không đụng vào)
            \
             ↓ (1) tạo sync branch từ main
             ●─────────●  sync/main-to-dev-<app>
                  ↑    ↑
                  │   (3) resolve conflict (giữ phía main) + commit
                  │
                  (2) merge builds/dev/<app> vào sync
                  │
builds/dev/<app> ─●─┘─────●  ← (4) tạo MR sync/* → builds/dev/<app>
```

**Nguyên tắc**: code chảy 1 chiều `main → builds/dev/<app>`. KHÔNG bao giờ PR ngược `builds/* → main`.

### Bảng trigger

| Prompt | Hành động |
|---|---|
| **list build branches** | List tất cả `builds/dev/<app>` có trong repo, để user pick app cần sync |
| **sync main to dev-&lt;app&gt;** | Sync `main → builds/dev/<app>`. Tạo nhánh `sync/main-to-dev-<app>`, merge `builds/dev/<app>` vào, resolve conflict, push, tạo MR. **HỎI user xác nhận** trước khi push |
| **sync main to dev-all** | Sync nhiều app cùng lúc. Loop tuần tự, mỗi app 1 MR riêng, dừng giữa từng app để user confirm |
| **kiểm tra build hygiene** / **audit all dev builds** | Phát hiện vi phạm rule "không commit thẳng `builds/*`". List commit lạ + đề xuất cleanup (cherry-pick về main hoặc reset build) |

### Naming convention

- **Sync branch**: `sync/main-to-dev-<app>` (ephemeral, xoá ngay sau khi MR merged)
- **Commit message**: `chore(sync): resolve conflict main → builds/dev/<app>`
- **MR target**: luôn là `builds/dev/<app>` — KHÔNG bao giờ là `main`

### Out of scope

Skill chỉ tập trung `main → builds/dev/<app>` vì các flow khác (`release → builds/prod`, cut release, cherry-pick hotfix) trong thực tế gần như luôn fast-forward, Maintainer làm tay được. Nếu sau này phát sinh nhu cầu sẽ mở rộng.

### Safety rules

- KHÔNG tạo MR `builds/* → main` dưới bất kỳ hình thức nào
- KHÔNG force push vào `main`/`builds/*` (kể cả `--force-with-lease`) trừ khi user/Maintainer chủ động ra lệnh
- KHÔNG dùng `git checkout --ours <file>` cho code logic mà không đọc qua diff
- Hỏi user trước khi resolve nếu không chắc bên nào đúng (đặc biệt với `.env`, config, route)

Xem chi tiết đầy đủ ở [`skills/gitlab-sync/SKILL.md`](skills/gitlab-sync/SKILL.md).

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

**Phân nhóm skills** (xem table ở trên cho mô tả):

| Nhóm | Skills |
|---|---|
| Workflow & quality | `gitlab-sync`, `karpathy-guidelines` (xem thêm `gitlab-flow` + `commit` + `review-branch` ở repo [`my-skills-gitlab-flow`](https://github.com/nguyenvanchiens/my-skills-gitlab-flow)) |
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
- **Nội bộ** — bổ sung cho stack ASP.NET Core production-grade: `aspnet-mvc`, `blazor`, `aspnet-logging`, `aspnet-caching`, `aspnet-health-checks`, `background-jobs`, `aspnet-auth-advanced`, `dotnet-testing-patterns`. Workflow GitLab: `gitlab-sync` (cặp `gitlab-flow`/`commit`/`review-branch` đã tách sang repo [`my-skills-gitlab-flow`](https://github.com/nguyenvanchiens/my-skills-gitlab-flow)).

## License

Code và cấu hình của repo này: MIT (xem [LICENSE](LICENSE)).
Mỗi skill trong `skills/<name>/` giữ license riêng của tác giả gốc.
