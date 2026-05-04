# Contributing to devflow-skills

devflow-skills is a **language-agnostic** skill library. Contributions for any
programming language, framework, or database are welcome — Java, PHP, Python,
C#, JavaScript, Ruby, Go, Rust, or anything else.

---

## Types of contributions

| Type | What it means |
|------|---------------|
| **New core skill** | A new pipeline phase skill (stack-agnostic) |
| **New stack variant** | A stack-specific version of an existing skill (e.g. `performance-audit-django-postgresql`) |
| **Improvement** | Fix, extend, or sharpen an existing skill |
| **Example output** | Add a sample output to an existing skill's `examples/` folder |

---

## Adding a new skill

```bash
# 1. Create the skill folder
mkdir -p skills/<skill-name>/examples

# 2. Create SKILL.md with valid YAML frontmatter
touch skills/<skill-name>/SKILL.md

# 3. Add the entry to skills.json (plugins array)

# 4. Add a row to the skills table in README.md

# 5. Add a checkbox to the Roadmap section in README.md

# 6. Commit and open a PR
```

**PR title:** `feat: add <skill-name> skill`

---

## Adding a new stack variant

Stack variants follow the naming pattern:

```
performance-audit-<framework>-<database>
```

Examples:
- `performance-audit-django-postgresql`
- `performance-audit-rails-postgresql`
- `performance-audit-nodejs-mongodb`
- `performance-audit-fastapi-postgresql`

Steps are the same as adding a new skill. Also add the variant to the
"Stack-specific" table in `README.md`.

---

## Improving an existing skill

1. Edit `skills/<skill-name>/SKILL.md`
2. Bump `version` in frontmatter:
   - Patch (`1.0.x`) — bug fix or wording improvement
   - Minor (`1.x.0`) — new feature, new section, or extended coverage
3. Match the version in `skills.json`
4. Update or add an example in `examples/` if output format changed

**PR title:** `fix: improve <skill-name>` or `feat: extend <skill-name>`

---

## Skill quality checklist

Verify every item before opening a PR:

- [ ] YAML frontmatter is valid — all required fields present
- [ ] `name` matches the folder name exactly (kebab-case)
- [ ] All `{PARAMETER}` placeholders are listed under `inputs`
- [ ] At least one example output exists in `examples/`
- [ ] All code examples are runnable — zero `// your logic here` comments
- [ ] `version` in `SKILL.md` frontmatter and `skills.json` match
- [ ] `description` is specific enough to reliably trigger the skill
- [ ] Skill has been tested at least once end-to-end with Claude
- [ ] Stack-specific skills use only syntax for their declared stack
  (no MySQL syntax in a PostgreSQL skill, no PHP in a Python skill)

---

## SKILL.md frontmatter reference

```yaml
---
name: skill-name           # kebab-case — must match folder name exactly
version: 1.0.0             # semver
phase: plan|build|ship     # pipeline phase this skill belongs to
description: >             # trigger conditions + what it produces
  Describe what this skill does and when Claude should use it.
  Be specific — this is what Claude reads to decide whether to trigger.
inputs:
  - "Parameter name (type — default value)"
outputs:
  - "output-filename-pattern.md"
stack:                     # technology tags — lowercase
  - laravel                # use "agnostic" for stack-independent skills
  - mysql
  - php
---
```

---

## skills.json registration

Add an entry to the `plugins` array in `skills.json`:

```json
{
  "name": "your-skill-name",
  "description": "One sentence matching the SKILL.md description.",
  "version": "1.0.0",
  "author": {
    "name": "Your Name or GitHub username"
  },
  "source": "./skills/your-skill-name",
  "category": "plan | build | performance | review | ship | agnostic"
}
```

---

## Naming conventions

| Thing | Convention | Example |
|-------|-----------|---------|
| Skill folder | kebab-case | `performance-audit-laravel-mysql` |
| Stack-specific variant | `<skill>-<framework>-<database>` | `performance-audit-spring-boot-postgresql` |
| Output files | `SCREAMING-KEBAB-<date>.md` | `AUDIT-2026-05-05.md` |
| PR titles | `feat:` / `fix:` / `docs:` + skill name | `feat: add tdd-spec skill` |
| Git tags | semver prefixed with `v` | `v1.0.0`, `v0.2.0` |

---

## Reporting issues

Use GitHub Issues with the appropriate template:

- **Bug report** — skill produces incorrect, broken, or incomplete output
- **New skill request** — propose a skill or stack variant not yet in the library

---

## Code of conduct

Be specific and constructive. Reference the quality checklist when reviewing PRs.
Suggest the exact change needed — vague feedback is not actionable.
