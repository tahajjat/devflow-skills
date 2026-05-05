---
name: performance-audit-laravel-mysql
version: 1.0.0
description: >
  Audits a Laravel + MySQL application for performance issues and generates
  two structured markdown reports: an issues report (ISSUE-XXX schema, three
  categories, severity rubric) and a resolution report with before/after
  runnable code, verification steps, and a production readiness appendix.
  Use when: slow response times, high MySQL CPU, N+1 query complaints, memory
  spikes, missing indexes, cache misuse, queue bottlenecks, pre-production
  readiness review, or any request containing "performance audit", "optimize
  Laravel", "slow queries", "N+1", "database tuning", "Eloquent performance",
  or "production readiness" in a Laravel or PHP context.
---

# Laravel + MySQL Performance Audit

## Overview

Performs a root-cause-first performance audit of any Laravel application backed
by MySQL 8.0. Produces two output files under `.devflow/performance/`  covering issue discovery, structured
resolutions with runnable before/after code, and a production readiness appendix
sized to the target server.

Works with Laravel 10, 11, 12, and 13 (current active releases as of 2026).
Default target is Laravel 12. Use Laravel 13 for new projects.

---

## When to use

**Use this skill when:**
- Response times are degrading or inconsistent under load
- MySQL CPU is high despite reasonable traffic
- Laravel Telescope or Debugbar shows > 30 queries per request
- N+1 query warnings appear in logs or profiling tools
- Memory usage spikes during batch jobs or list endpoints
- Pre-production audit is needed before a release
- A developer asks to "optimize" or "audit" a Laravel project

**Do not use when:**
- The project uses PostgreSQL, MSSQL, or SQLite — use the matching variant skill
- The issue is infrastructure-only (server sizing, load balancing) with no code component
- The project is not Laravel / PHP

---

## Parameters — confirm before starting

| Parameter | Default | Options |
|-----------|---------|---------|
| `{LARAVEL_VERSION}` | 12 | 10, 11, 12, 13 |
| `{SERVER_RAM}` | 16 | any integer in GB |

If not provided by the user, use defaults and state the assumption in a callout
at the top of each output document:

```
> **Audit assumptions:** Laravel {LARAVEL_VERSION} · MySQL 8.0 · {SERVER_RAM} GB RAM
> Update these values if your environment differs.
```

---

## Core process

Follow these steps in order. Do not skip or reorder.

### Step 1 — Establish context

Before writing any output, confirm or infer:
- Laravel version in use
- Server RAM available for MySQL tuning
- Any specific symptoms or modules the user wants prioritised
- Whether this is a new audit or a follow-up to a previous one

If the user provided symptoms (e.g. "the orders page is slow"), treat those as
priority anchors — the highest-severity issues should relate to them directly.

### Step 2 — Prepare output folder and gitignore

Execute silently before generating any report. Do not ask for confirmation.
Do not mention this step in the output summary.

**2a — Create the output folder:**
```bash
mkdir -p .devflow/performance
```

**2b — Update .gitignore:**

Check if `.gitignore` exists at the project root:
- If it exists: check whether `.devflow/` is already present
  - If YES — do nothing, continue
  - If NO — append this block at the end of the file:
    ```
    # devflow-skills generated output — never commit
    .devflow/
    ```
- If it does not exist: create `.gitignore` with this content:
  ```
  # devflow-skills generated output — never commit
  .devflow/
  ```

Execute both sub-steps silently and proceed to Step 3.

### Step 3 — Generate Task A: Issues Report

**Output file:** `.devflow/performance/project_issues_<YYYY-MM-DD>.md`
Use today's date. If unavailable, use the literal placeholder and add a note.

**Document header must include:**
- Title: "Laravel Performance Audit — Issues Report"
- Date, Laravel version, PHP version, MySQL version
- Audit scope summary (1–2 sentences)

**Organise all issues into exactly three sections:**
1. Database Layer — queries, indexes, schema decisions
2. Application Layer — Eloquent misuse, memory, caching, queue design
3. Infrastructure Layer — MySQL config, PHP-FPM, connection pooling

**Coverage targets:**

| Category | Min | Max |
|----------|-----|-----|
| Database Layer | 5 | 7 |
| Application Layer | 6 | 8 |
| Infrastructure Layer | 4 | 6 |

Prioritise depth over count. Five well-documented issues beat eight shallow ones.

**Use this exact schema for every issue:**

```
#### [ISSUE-XXX] <Short descriptive title>

**Category:** Database | Application | Infrastructure
**Severity:** Critical | High | Medium | Low
**Impact:** CPU | Memory | Response Time | Throughput

**Description:**
2–3 sentences. Name the specific code pattern or config value causing the
problem. State what architectural decision led to it.

**Manifestation example:**
```php
// Minimal runnable code snippet showing the exact problematic pattern
```

**Observable symptoms:**
- What appears in Telescope, Debugbar, or the MySQL slow query log
- Include specific thresholds where possible (e.g. "100+ queries per request")
```

**Severity rubric — apply consistently:**

| Severity | Definition |
|----------|------------|
| Critical | Causes production outages, data loss, or cascading failures under normal load |
| High | Degrades response times > 2s or causes > 30% CPU spike under moderate load |
| Medium | Causes measurable degradation; unlikely to cause outages alone |
| Low | Minor inefficiency; acceptable in isolation but compounds at scale |

**Quality gate — verify all before Step 4:**
- [ ] All ISSUE-XXX IDs are sequential with no gaps
- [ ] Each category meets its minimum count
- [ ] At least one Critical or High issue in every category
- [ ] Every issue has a concrete code or config example — no vague descriptions

### Step 4 — Generate Task B: Resolution Report

**Output file:** `.devflow/performance/project_issues_resolve_<YYYY-MM-DD>.md`
Use the same date as Task A.

Every ISSUE-XXX from Task A must have exactly one resolution entry. No gaps.
Generate Task B immediately after Task A — do not wait for a follow-up prompt.

**Use this exact schema for every resolution:**

```
#### [ISSUE-XXX] Resolution — <Same title as in Task A>

> See [ISSUE-XXX] for full description and symptoms.

**Root cause:**
One sentence naming the architectural or design decision that caused this.

**Fix — step by step:**
1. ...
2. ...

**Before:**
```php
// Exact problematic code from the issues report
```

**After:**
```php
// Corrected, runnable code — valid for {LARAVEL_VERSION} / PHP 8.x / MySQL 8.0
// Zero placeholder comments
```

**Verification:**
How to confirm the fix worked — be specific:
- Query fix: EXPLAIN output change (e.g. type changes from ALL to ref)
- N+1 fix: Telescope query count drops from N+1 to 2 for this endpoint
- Memory fix: memory_get_peak_usage() drops below target MB
- Config fix: SHOW VARIABLES LIKE 'innodb_buffer_pool_size' returns new value

**Prevention:**
One architectural rule or code-review checklist item that prevents recurrence.
If this fix conflicts with another resolution, document the trade-off explicitly
here rather than omitting either approach.
```

### Step 5 — Append Task C: Production Readiness Appendix

Append to the end of the Resolution Report file under:
`## Appendix: Production Readiness`
Separated from resolutions by `---`.

**C1 — MySQL 8.0 tuning block**
Produce an annotated `my.cnf` `[mysqld]` block sized for `{SERVER_RAM}` GB RAM.
Every setting must have an inline `# comment` explaining what it does and why
the value was chosen. Workload assumed: mixed OLTP, ~65% reads / ~35% writes,
200–500 peak concurrent connections. Never use MySQL install defaults — every
value must be calculated from `{SERVER_RAM}`.

**C2 — Laravel performance configuration checklist**
Cover each item with a concrete command or config value, not a description:
- `php artisan config:cache` and `route:cache` — when to run, when not to
- OPcache `php.ini` recommended values
- Queue worker count formula, `--timeout`, `--tries`, `--max-jobs`
- `chunk()` vs `lazy()` vs `cursor()` — one-line decision rule for each
- Read/write splitting via `config/database.php` sticky connections

**C3 — Observability stack**
For each tool: what to instrument AND the numeric threshold that triggers an alert:
- Laravel Telescope
- Laravel Debugbar
- MySQL slow query log (`long_query_time` value + file path)
- New Relic or Datadog (pick one, be specific about metric names)

**C4 — Pre-deployment checklist**
Exactly 10 binary (yes/no) items — no more, no fewer. Each must be verifiable
with a single command or observable output. Obvious items must still be listed
explicitly — they are the ones most often skipped under deadline pressure.

### Step 6 — Final self-check

Before delivering output, verify every item:
- [ ] `.devflow/performance/` folder exists in the project root
- [ ] `.devflow/` is present in `.gitignore`
- [ ] Task A and Task B cover identical ISSUE-XXX sets
- [ ] No "After" code block contains placeholder comments of any kind
- [ ] All DB config values in Task C are calculated for `{SERVER_RAM}` — not MySQL defaults
- [ ] Task C contains no issue resolutions — forward-looking guidance only
- [ ] Both filenames use today's actual date in YYYY-MM-DD format
- [ ] All code is valid for `{LARAVEL_VERSION}` / PHP 8.x / MySQL 8.0
- [ ] C4 checklist has exactly 10 items

---

## Examples

### Correct issue entry (shape reference)

#### [ISSUE-001] Unindexed foreign key on high-traffic join column

**Category:** Database
**Severity:** High
**Impact:** Response Time, CPU

**Description:**
The `orders` table declares a `user_id` foreign key with no index. Every
`$user->orders()->get()` call performs a full table scan against orders.
At 500k rows this consistently exceeds 2s response time on the user dashboard.

**Manifestation example:**
```php
// Migration — foreign key declared but index never added
Schema::create('orders', function (Blueprint $table) {
    $table->id();
    $table->unsignedBigInteger('user_id'); // missing ->index()
    $table->timestamps();
});
```

**Observable symptoms:**
- EXPLAIN shows `type: ALL` on the orders table for any user-scoped query
- Telescope: user dashboard endpoint consistently > 800ms under moderate load

---

### Correct resolution entry (shape reference)

#### [ISSUE-001] Resolution — Unindexed foreign key on high-traffic join column

> See [ISSUE-001] for full description and symptoms.

**Root cause:**
The migration was written when the orders table was small and the full-scan
cost was invisible; no index policy was enforced at review time.

**Fix — step by step:**
1. Generate a migration: `php artisan make:migration add_index_user_id_to_orders`
2. Add `$table->index('user_id')` in the `up()` method
3. Add `$table->dropIndex(['user_id'])` in the `down()` method
4. Run on staging, verify with EXPLAIN, then deploy to production

**Before:**
```php
$table->unsignedBigInteger('user_id');
```

**After:**
```php
$table->unsignedBigInteger('user_id')->index();
```

**Verification:**
Run `EXPLAIN SELECT * FROM orders WHERE user_id = 1` before and after.
Before: `type: ALL`, `rows: ~500000`. After: `type: ref`, `rows: ~12`.
Telescope response time for the user dashboard drops below 200ms.

**Prevention:**
Code review rule — every foreign key column in a migration must have
`->index()` or `->foreign()`. Reviewer rejects the migration without it.

---

## Common rationalizations

| Rationalization | Reality |
|----------------|---------|
| "I'll use generic code examples since I don't have the actual codebase" | Every example must show a specific, plausible real-world pattern — not `// example code here`. Generic examples produce generic fixes that developers cannot act on. |
| "I'll use the latest Laravel version for all examples" | Match all examples strictly to `{LARAVEL_VERSION}`. If the user targets Laravel 11, do not use Laravel 12/13 APIs or syntax that did not exist in 11. Version-specific accuracy is non-negotiable. |
| "I'll skip Infrastructure issues since those aren't code" | Config issues like an undersized `innodb_buffer_pool_size` cause more production outages than any N+1 query. Infrastructure is mandatory and must meet its minimum count. |
| "I'll combine the two output files to save space" | Task A and Task B must be separate files. The issues report is used standalone during triage without the resolution context. |
| "The resolution can just say 'fix the query'" | Every resolution requires a runnable Before/After code block, a specific verification method, and a prevention rule. Vague advice fails the quality gate. |
| "I'll skip the Task A quality gate and go straight to Task B" | Skipping the gate produces sequential ID gaps, under-populated categories, or missing severity coverage — all of which make the report unusable in triage. |
| "Task A is done — I'll stop here and let the user ask for Task B" | Both files are produced in a single skill run. Generate Task B immediately after Task A without waiting for a follow-up prompt. |
| "The server RAM wasn't specified so I'll use generic my.cnf values" | Default to `{SERVER_RAM}` = 16 GB if not provided. MySQL install defaults (e.g. `innodb_buffer_pool_size = 128M`) signal that the config was never tuned. Always calculate values from the assumed RAM. |
| "The C4 checklist has 8 items — the other 2 are obvious" | The checklist must have exactly 10 items. Obvious items must be listed explicitly — they are the ones most often skipped under deadline pressure before deployment. |
| "The user can add .gitignore themselves" | The skill must handle .gitignore automatically on every run. Audit reports contain environment-specific findings that must never reach a shared repository. |

---

## Red flags

- Output files written to project root instead of `.devflow/performance/`
- `.devflow/` missing from `.gitignore` after the skill runs
- Issue descriptions reference vague categories ("bad query") without naming the specific code pattern
- "After" code blocks contain `// your logic here`, `// implement this`, or `// add logic here`
- Laravel version in code examples does not match `{LARAVEL_VERSION}` (e.g. PHP 7.x syntax in a Laravel 12 audit)
- Task B has fewer resolution entries than Task A has issues
- MySQL config values in Task C are install defaults (e.g. `innodb_buffer_pool_size = 128M`)
- Task C appendix contains issue resolutions instead of forward-looking guidance only
- C4 pre-deployment checklist has fewer or more than 10 items
- Issue IDs are not sequential or contain a gap (e.g. ISSUE-003 followed by ISSUE-005)

---

## Verification

After completing the full process, confirm every item before delivering output:

- [ ] `.devflow/performance/` folder exists at the project root
- [ ] `.devflow/` is present in `.gitignore` — verified by reading the file
- [ ] `.devflow/performance/project_issues_<date>.md` exists with today's date in the filename
- [ ] `.devflow/performance/project_issues_resolve_<date>.md` exists with the same date
- [ ] Issue count per category: Database 5–7, Application 6–8, Infrastructure 4–6
- [ ] Every ISSUE-XXX in Task A has a matching resolution entry in Task B
- [ ] No sequential gaps in ISSUE-XXX IDs
- [ ] At least one Critical or High severity issue in each of the three categories
- [ ] Every "After" code block is syntactically valid for `{LARAVEL_VERSION}` / PHP 8.x / MySQL 8.0
- [ ] No "After" block contains placeholder comments of any kind
- [ ] Task C appendix contains all four sub-sections: C1, C2, C3, C4
- [ ] `my.cnf` values in C1 are calculated for `{SERVER_RAM}` GB — not MySQL install defaults
- [ ] C4 pre-deployment checklist has exactly 10 binary items
