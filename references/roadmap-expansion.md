# Roadmap expansion mode

Turn any product plan or roadmap into per-release, agentic-worker-ready implementation
plans. Each release gets its own Markdown file. Every task inside a release is a
self-contained unit of work: files to create, checkbox steps with code, a smoke
check, and a git commit. The whole file drives a `superpowers:subagent-driven-development`
or `superpowers:executing-plans` worker end-to-end.

---

## When to use this skill

- User hands you a plan doc (roadmap, spec, industrial) and says "break into tasks" /
  "task-by-task" / "writing-plans format" / "break it down into tasks" / "step-by-step plan".
- User wants to expand a single release (e.g. "expand R2 now").
- User wants to re-expand an existing plan file with corrections.

**Do not** use for: general code generation, answering questions about the plan,
or when the user only wants a summary / outline.

---

## Workflow

### Step 1 — Understand the plan

Read the input document carefully. Identify:

1. **Releases** — what are R1, R1.0, R1.1, R2, R3 (or whatever the phases are)?
2. **Tech stack** — extract exact versions; if missing, ask the user or note
   "TBD" in the header of each file.
3. **Architecture** — services, packages, infra; this goes into each file's header.
4. **TDD rule** — if the plan mandates test-first, every task must include a
   failing-test step *before* the implementation step.
5. **Language** — determine one language for the whole output (EN or RU or other).
   If unclear, ask. Once decided, every heading, comment, commit message, and prose
   must be in that language. Code identifiers stay in English always.

If critical info is missing (tech stack, architecture, language), ask before writing.
Do not invent versions or constraints.

### Step 2 — Plan the files

Announce to the user which files you will produce, e.g.:

```
Will produce:
  YYYY-MM-DD-R1_0-paid-alpha.md
  YYYY-MM-DD-R1_1-buybot-alerts.md
  YYYY-MM-DD-R2-launch-suite.md
  YYYY-MM-DD-R3-mm-workstation.md
```

Use today's date as the file prefix. Use kebab-case for the slug.

Ask: "Expand all at once, or release-by-release?"
- **All at once** → produce every file, then present them all.
- **Release-by-release** → produce one file, confirm, then the next.

### Step 3 — Write each release file

Follow the **File Template** section below exactly. Do not skip sections.

### Step 4 — Present

Use `present_files` to hand all produced files to the user.

---

## File Template

Each release file must contain every section below, in this order.

### 1. Header block

```markdown
# {Release Name} Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use `superpowers:subagent-driven-development`
> (recommended) or `superpowers:executing-plans` to implement this plan task-by-task.
> Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** {One sentence — what working software ships at the end of this release.}

**Architecture:** {2-4 sentences describing services, packages, data flow, constraints.
What is new vs inherited. No key custody. All signing via TON Connect if applicable.}

**Tech Stack:** {Comma-separated list of packages with pinned versions.}

**Prerequisites:** {Prior releases or external accounts needed. "None" if R1.0.}
```

### 2. File Structure

Show a tree of *new or changed* files only (for R1.0+ show inherited context briefly):

```markdown
## File Structure

\`\`\`
apps/
  api/src/...
packages/
  shared/...
\`\`\`

One file = one responsibility. Cross-app code lives in `packages/`. App-specific code in `apps/<name>/src/`.
```

### 3. Tasks

Number tasks sequentially. Each task = one git commit.

```markdown
## Task N: {Short descriptive title}

**Files:**
- Create: `path/to/file.ts`
- Modify: `path/to/other.ts`

- [ ] **Step 1:** {What to do — use imperative verb}

\`\`\`lang
{exact code or commands}
\`\`\`

- [ ] **Step 2:** ...

Expected: {one-line check — what green looks like, e.g. "PASS", "server starts", "no TS errors"}

- [ ] **Step N:** Commit

\`\`\`bash
git add {paths}
git commit -m "{type(scope): message}"
\`\`\`
```

**Rules for tasks:**

- First step of implementation tasks **must be a failing test** if TDD is required.
  Label it clearly: `**Step 1 (RED):** Write failing test`.
- Implementation step follows: `**Step 2 (GREEN):** Write minimal implementation`.
- Refactor step if needed: `**Step N (REFACTOR):** Clean up while tests stay green`.
- Every bash command must be copy-paste ready.
- Every code block must be complete (no `// ... rest of file`).
- If a file is too long for one step, split it into logical sub-steps with `- [ ]` each.
- "Expected:" is required after any `pnpm test`, `docker compose`, or server start command.
- Commit message format: `type(scope): message` — types: feat, fix, test, chore, refactor, docs.

**Minimum task coverage per release:**

| Area | Must have tasks for |
|------|---------------------|
| Infra | Docker Compose, env vars |
| DB | Schema, migrations, seed |
| Backend | Each service/module independently |
| Auth | Login flow (Telegram, JWT, etc.) |
| Payments | Payment matching, entitlement |
| Bot | Handler registration, webhook setup |
| Frontend | Layout, auth pages, feature pages |
| Tests | Unit, integration, E2E each as own task |
| CI | GitHub Actions or equivalent |

Do not bundle unrelated concerns into one task. One task = one commit.

### 4. Done Criteria

```markdown
## Done criteria for {Release}

- All {N} tasks merged
- `pnpm test` green (or equivalent)
- Manual smoke: {describe the happy path a human runs end-to-end}
- {Any release-specific KPI from the plan, e.g. "payment memo match confirmed on testnet"}
```

### 5. Deferred Items

```markdown
## Skipped in {Release} (deferred to {Next Release})

- {Feature A}
- {Feature B}
```

---

## Task Granularity Guide

| Scope | Rule |
|-------|------|
| Monorepo / infra skeleton | 1 task |
| Each Prisma model group | 1 task |
| Each NestJS module (controller + service + tests) | 1 task |
| Each Telegram command or webhook handler | 1 task |
| Each external adapter (TonAPI, STON.fi, DeDust) | 1 task |
| Payment matching + entitlement logic | 1 task |
| Each frontend page group (auth, dashboard, etc.) | 1 task |
| Unit test suite for a package | 1 task (if not already inside module task) |
| Integration test suite | 1 task |
| E2E happy path | 1 task |
| CI pipeline | 1 task |
| README + quickstart | 1 task |

Prefer finer granularity. If in doubt, split.

---

## TDD Checklist (when plan mandates test-first)

Apply to every task that produces production logic:

```
- [ ] Step 1 (RED): Write minimal failing test. Run it. Paste the failure output as a comment or note.
- [ ] Step 2 (GREEN): Write smallest implementation that passes the test.
- [ ] Step 3: Run full test suite for the module. All green.
- [ ] Step 4 (REFACTOR): Clean up. Re-run. Still green.
```

Never write implementation code before a failing test exists. Exceptions (no test needed):
- Generated wrappers (e.g. Prisma client, Tolk TS wrappers)
- Pure DTO types / Zod schemas used only for type checking
- Docker / Compose / Kubernetes YAML
- Migration SQL (covered by integration tests)
- One-off seed scripts

---

## Language Rules

- **Choose one language for the entire file.** EN or RU (or as specified by user).
- Headings, prose, Expected lines, comments in markdown → chosen language.
- Code identifiers, variable names, function names → English always.
- Git commit messages → English always (conventional commits).
- If the plan mixes languages, ask the user before proceeding.

---

## Common Mistakes to Avoid

1. **Vague steps** — "implement the service" is not a step. Write out the actual code.
2. **Missing Expected** — every terminal command needs an expected outcome.
3. **Missing commit** — every task ends with a git commit step.
4. **Bundled tasks** — auth + payments + DB in one task. Split them.
5. **Missing TDD steps** — if TDD required, no implementation step without a prior RED step.
6. **Wrong language** — mixing RU prose and EN prose in the same file.
7. **Omitting Deferred section** — always list what is explicitly NOT in this release.
8. **Invented versions** — if the input plan doesn't specify a version, use "latest stable" and note it.

---

## Reference: Conventional Commit Types

| Type | Use for |
|------|---------|
| `feat` | New feature or capability |
| `fix` | Bug fix |
| `test` | Adding or fixing tests |
| `chore` | Build, config, deps, infra |
| `refactor` | Code change without behavior change |
| `docs` | Documentation only |
| `perf` | Performance improvement |

Scope = package or app name, e.g. `feat(api)`, `test(ton)`, `chore(infra)`.
