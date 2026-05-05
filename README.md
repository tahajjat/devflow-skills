# devflow-skills

> A unified library of AI prompt skills covering the full software development lifecycle —
> language-agnostic, framework-flexible, and built for use with **Claude Code** and **Claude.ai**.

Works with any stack: **Java . Spring Boot · Python . Django · ASP.NET Core . Laravel · PHP · Node.js · Rails · FastAPI** —
and any database: **Oracle . MySQL · PostgreSQL · MSSQL · SQLite · MongoDB**.

---

## Install

Add the marketplace source:

```bash
claude plugin marketplace add tahajjat/devflow-skills
```

Install the plugin:

```bash
claude plugin install devflow-skills@tahajjat
```

---

## The pipeline

Skills chain together — each skill's output becomes the next skill's input.
The pipeline is **stack-independent**: the same phases apply whether you are
building in Java, Python, PHP, C#, or any other language.

```
┌─────────────────────────────────────────────────────────────────────────┐
│  PLAN                                                                   │
│                                                                         │
│  Phase 1            Phase 2            Phase 3          Phase 4        │
│  requirement-   →   plan-          →   plan-        →   plan-          │
│  analysis           requirements       architecture      database       │
│  (you)              (you + Claude)     (you + Claude)    (Claude)       │
│  BRD-*.md           URS-*.md           ARCH-*.md         ERD-*.md       │
│                     FRS-*.md                                            │
└─────────────────────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────────────────────┐
│  BUILD                                                                  │
│                                                                         │
│  Phase 5            Phase 6            Phase 7                         │
│  tdd-spec       →   code-          →   code-review                     │
│  (Claude)           generate           (you + Claude)                  │
│  TEST-*.md          src/               REVIEW-*.md                     │
└─────────────────────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────────────────────┐
│  SHIP                                                                   │
│                                                                         │
│  Phase 8            Phase 9                                            │
│  performance-   →   commit-                                            │
│  audit              changelog                                           │
│  (Claude)           (support)                                           │
│  AUDIT-*.md         git commit                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### Entry points by scenario

| Scenario | Start at | Skip |
|----------|----------|------|
| New project from scratch | Phase 1 | — |
| New feature on existing system | Phase 2 | Phase 1 |
| Performance issue in production | Phase 8 | Phases 1–7 |
| Code review before PR | Phase 7 | Phases 1–6 |
| Bug fix (root cause first) | Phase 1 → Phase 6 | Phases 3–5 |
| Database design only | Phase 4 | Phases 1–3, 5–9 |

---

## Skills

### Core pipeline skills
> Stack-agnostic — work with any language, framework, or database.

| Skill | Phase | Status | Description |
|-------|-------|--------|-------------|
| [requirement-analysis](./skills/requirement-analysis/) | 1 · Plan | Planned | Converts a raw project brief into a structured BRD via guided interview |
| [plan-requirements](./skills/plan-requirements/) | 2 · Plan | Planned | BRD → URS + FRS with FR/NFR IDs and Given/When/Then acceptance criteria |
| [plan-architecture](./skills/plan-architecture/) | 3 · Plan | Planned | System architecture doc: component diagram, tech stack decisions, API boundaries |
| [plan-database](./skills/plan-database/) | 4 · Plan | Planned | ERD, migration plan, index strategy, seeding spec from FRS |
| [tdd-spec](./skills/tdd-spec/) | 5 · Build | Planned | Test specs written before implementation: Arrange/Act/Assert, edge cases, mocks |
| [code-generate](./skills/code-generate/) | 6 · Build | Planned | Production-ready scaffolding from task list and architecture doc |
| [code-review](./skills/code-review/) | 7 · Build | Planned | Structured diff review: correctness, security, performance, style with severity rubric |
| [performance-audit](./skills/performance-audit/) | 8 · Ship | Planned | Universal performance audit template — extend for any stack |
| [commit-changelog](./skills/commit-changelog/) | 9 · Ship | Planned | Conventional commit messages + CHANGELOG.md from task ID and diff |

### Stack-specific performance audit skills
> Specialised variants of the performance audit with stack-aware code examples,
> database-specific tuning blocks, and framework-specific observability tools.

| Skill | Status | Stack |
|-------|--------|-------|
| [performance-audit-laravel-mysql](./skills/performance-audit-laravel-mysql/) |  **v1.0.0** | Laravel (10–13) · PHP 8.x · MySQL 8.0 |
| [performance-audit-laravel-postgresql](./skills/performance-audit-laravel-postgresql/) |  Planned | Laravel (10–13) · PHP 8.x · PostgreSQL 15/16 |
| [performance-audit-asp-core-mssql](./skills/performance-audit-asp-core-mssql/) |  Planned | ASP.NET Core 8 · C# · MSSQL 2019/2022 |
| [performance-audit-spring-boot](./skills/performance-audit-spring-boot/) |  Planned | Spring Boot 3 · Java 21 · MySQL / PostgreSQL |

> More variants planned: Django + PostgreSQL, Node.js + MongoDB, Rails + PostgreSQL, FastAPI + PostgreSQL.
> Open a [new skill request](https://github.com/tahajjat/devflow-skills/issues/new?template=new_skill.md) to vote for the next one.

---

## How to use

### Option A — Claude Code custom command (recommended)

```bash
# Copy a skill into your project's Claude commands folder
mkdir -p .claude/commands

cp path/to/devflow-skills/skills/performance-audit-laravel-mysql/SKILL.md \
   .claude/commands/performance-audit.md
```

Invoke inside Claude Code:

```
/performance-audit
```

### Option B — Claude Code inline

```bash
claude "Run a performance audit on this project." \
  --context devflow-skills/skills/performance-audit-laravel-mysql/SKILL.md
```

### Option C — Claude.ai (paste)

1. Open `skills/<skill-name>/SKILL.md`
2. Copy the full file contents
3. Replace any `{PARAMETER}` placeholders with your actual values
4. Paste into [claude.ai](https://claude.ai) as your prompt

---

## Skill parameters

Skills use `{PARAMETER}` placeholders. Replace these before running.
Each skill's `SKILL.md` lists all parameters and their defaults in a table
at the top of the file.

Example — `performance-audit-laravel-mysql`:

| Parameter | Default | Options |
|-----------|---------|---------|
| `{LARAVEL_VERSION}` | 12 | 10, 11, 12, 13 |
| `{SERVER_RAM}` | 16 | any integer (GB) |

If you do not provide a value, the skill uses the default and states the
assumption at the top of every output document.

---

## Skill format

Every skill is a self-contained folder:

```
skill-name/
├── SKILL.md       ← main prompt (YAML frontmatter + instructions)
└── examples/      ← sample outputs for reference
```

`SKILL.md` YAML frontmatter:

```yaml
---
name: skill-name           # kebab-case — must match folder name
version: 1.0.0             # semver — bump on every change
phase: plan|build|ship     # pipeline phase
description: >             # what it does + when to trigger it
  ...
inputs:                    # parameters the user must provide
  - Parameter (type — default)
outputs:                   # files or documents produced
  - output-pattern.md
stack: [laravel, mysql]    # technology tags (use "agnostic" if stack-independent)
---
```

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Kebab-case identifier — must match the folder name |
| `version` | Yes | Semver — bump on every meaningful change |
| `phase` | Yes | `plan` / `build` / `ship` |
| `description` | Yes | Used by Claude to decide when to trigger the skill |
| `inputs` | Yes | Parameters the user must supply before running |
| `outputs` | Yes | Files or documents the skill produces |
| `stack` | No | Technology tags — use `agnostic` for stack-independent skills |

---

## Repository structure

```
devflow-skills/
├── README.md
├── CONTRIBUTING.md
├── LICENSE
│
├── .claude-plugin/
│   ├── plugin.json                                ← plugin metadata
│   └── marketplace.json                           ← marketplace manifest
│
├── skills/
│   │
│   │   ── Core pipeline skills (stack-agnostic) ──
│   ├── requirement-analysis/                      ←  planned
│   ├── plan-requirements/                         ←  planned
│   ├── plan-architecture/                         ←  planned
│   ├── plan-database/                             ←  planned
│   ├── tdd-spec/                                  ←  planned
│   ├── plan-tasks/                                ←  planned
│   ├── code-generate/                             ←  planned
│   ├── code-review/                               ←  planned
│   ├── performance-audit/                         ←  planned
│   ├── commit-changelog/                          ←  planned
│   │
│   │   ── Stack-specific performance audits ──
│   ├── performance-audit-laravel-mysql/           ←  v1.0.0
│   │   ├── SKILL.md
│   │   └── examples/
│   ├── performance-audit-laravel-postgresql/      ←  planned
│   ├── performance-audit-asp-core-mssql/          ←  planned
│   └── performance-audit-spring-boot/             ←  planned
│
└── .github/
    └── ISSUE_TEMPLATE/
        ├── bug_report.md
        └── new_skill.md
```

---

## Roadmap

**Phase 0 — Foundation (current)**
- [x] `performance-audit-laravel-mysql` v1.0.0
- [ ] `performance-audit-laravel-postgresql` v1.0.0
- [ ] `performance-audit-asp-core-mssql` v1.0.0
- [ ] `performance-audit-spring-boot` v1.0.0

**Phase 1 — Core pipeline**
- [ ] `requirement-analysis` v1.0.0
- [ ] `plan-requirements` v1.0.0
- [ ] `plan-architecture` v1.0.0
- [ ] `plan-database` v1.0.0
- [ ] `tdd-spec` v1.0.0
- [ ] `code-generate` v1.0.0
- [ ] `code-review` v1.0.0
- [ ] `commit-changelog` v1.0.0

**Phase 2 — Full pipeline release**
- [ ] All 9 core skills complete and tested → **v1.0.0**
- [ ] Full pipeline end-to-end example in `examples/`
- [ ] Additional performance audit variants (Django, Rails, Node.js)

---

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md) for how to add or improve a skill.
Use [GitHub Issues](https://github.com/tahajjat/devflow-skills/issues) to
report bugs or request new skills or stack variants.

---

## License

MIT — see [LICENSE](./LICENSE)

Built by [Tahajjat](https://github.com/tahajjat)
