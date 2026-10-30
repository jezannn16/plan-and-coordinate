# Multi-Agent Coordination Mode

Converts file-by-file release plans (written-plans format) into one
coordination **AGENTS-STATUS.md** for **N agents** (N≥1), where each
agent has a number and owns specific tasks. Board:

- distributes tasks to agents based on file exclusivity
- blocks phases with a chain of dependencies
- forces free agents to **wait in the active cycle** (not stopped)
  until the task appears
- requires git-commit on **every** action (claim, progress, done, wait,
  signal coordination)

---

## Three modes of use

### Mode 1 - Board Generation (Generator)

The user has expanded plan files (R1.0.md, R1.1.md, R2.md, ...) and he
wants to get a fresh `AGENTS-STATUS.md`.

→ Go to **Generator Workflow**.

### Mode 2 - Agent Runtime

A specific agent works on the board: captures a task, updates the status,
makes a commit or enters a wait-loop.

→ Go to **Agent Runtime Protocol**.

### Mode 3 - Wait-loop (special Runtime submode)

The agent is free, but there are no parallel tasks for it → does not turn off, but
spins the loop `git pull → check board → sleep` until the task appears.

→ Go to **Wait-Loop Protocol**.

---

## Generator Workflow

### Step 0 - Ask how many agents

**This is a MANDATORY first step.** Before any plan readings, ask:

> *"How many developers/agents will be working on the project? Enter a number
> N (for example 2, 3, 4). Then everyone will choose their number (Agent-1, Agent-2, ...)."*

If the user says:
- "one" / "1" / "I'm alone" → N=1 (the board is still useful for tracking;
  single-agent mode with linear order of tasks is supported)
- "two" / "2" / "me and a friend" → N=2
- "three" / "3" → N=3
- ...
- “a lot” / “I don’t know” → ask for clarification: “Minimum 2, maximum 8. How much?”

**Limit**: N from 1 to 8. More - degradation of coordination, offer to break it up
to sub-teams.

After receiving N, ask **second required question**:

> *"Okay, N={N}. Each agent must choose their own permanent number.
> Which agent am I? Agent-1 / Agent-2 / … / Agent-{N}?"*

Record which agent is the current speaker (this affects `🔄` captures
and highlighting on the board). By default, first to launch = Agent-1.

Names `Agent-1` ... `Agent-N` - standard. If the user wants
human names (“Vanya”, “Masha”, “Pasha”) - support, but on the board
Always show the number next to your name: `Masha (Agent-2)`.

### Step 1 - Read all plan files

Read every expanded-release file from the user. For every task
extract:

- task number and title
- files it touches (block `Files:`)
- dependencies (which should be done first - explicit references or ordinal ones)
- money-critical / security-critical marking if available

### Step 2 — Split tasks into phases

Group tasks within each release into **consecutive phases**, where
within the phase, tasks can go **parallel**. Task enters the phase when
ALL of his dependencies are in earlier phases.

### Step 3 - Distribute owners to N agents

Assign each task an owner (one of Agent-1 ... Agent-N) or mark
SHARED (several agents working together - a rare case for a pair
programming in the money-critical part).

**Distribution rules:**

1. **File exclusivity within a phase.** In one parallel phase the same
   the same file (path) belongs to exactly one agent. No intersections.
2. **Equal load.** Within the parallel phase, the number of tasks per agent
   should differ by no more than 1. If there are 5 tasks in a phase and N=3, this is
   2 + 2 + 1, not 4 + 1 + 0.
3. **Vertical specialization if possible.** If the project has
   obvious verticals (data/comms/infra/frontend) - stick to one
   verticals per agent within the release. But this is secondary to file exclusivity.
4. **If N=1.** All tasks on Agent-1 in linear order. The phases become
   formality (one chain), wait-loop is not needed.
5. **If N > number of parallel tasks in the phase.** Extra agents are included in
   **wait-loop** during this phase (see Wait-Loop Protocol below). NOT fractions
   artificially make tasks so that everyone is busy - this creates conflicts.
6. **Money-critical / Security-critical tasks.** Marked with `💰 MONEY-CRITICAL`
   or `🛡️ SECURITY-CRITICAL`, go to their own separate phase, the owner is one
   agent, concurrency is prohibited within this phase.

If unclear, ask the user about specific priorities (“who
is stronger in TS - Agent-1 or Agent-2?", "Who is responsible for billing?").

### Step 4 – Write AGENTS-STATUS.md

Use the **Board Template** below **in its entirety**. Don't skip sections.

### Step 5 - Save and Present

Save the file as `AGENTS-STATUS.md` in your working directory.
**Make your first commit right away:**

```bash
git add AGENTS-STATUS.md
git commit -m "chore(agents): initial board for N={N} agents across {K} releases"
git push
```

Then present the file via `present_files`.

---

## Board Template

````markdown
# 🚦 AGENTS-STATUS.md — {Project Name}

**N agents:** {N} · **Creation date:** YYYY-MM-DD

| # | Agent | Name | Current status |
|---|-------|-----|----------------|
| 1 | Agent-1 | {name or '—'} | 💤 idle |
| 2 | Agent-2 | {name or '—'} | 💤 idle |
| ... | ... | ... | ... |

<!--
═══════════════════════════════════════════════════════════════════════
TASK STATUSES:
  ⬜ not started 🔄 working NOW ✅ ready and committed
  ⛔ blocked (dependency awaiting)
  ⏸️ requires coordination (see section below)

PHASE/RELEASE STATUS:
  🔒 LOCKED  →  🏃 ACTIVE  →  ✅ DONE

AGENT STATUSES (in the table above and in individual sections):
  💤 idle - no active task, in wait-loop
  🔄 working - captured the task (there is a line with 🔄 in the board)
  ⏳ blocked - waiting for another agent (coordination)
  ✅ done-phase - has finished all its tasks of the current phase, is waiting for the rest
  🛑 stopped - intentionally left (rare; norm: 💤, not 🛑)

═══════════════════════════════════════════════════════════════════════
RULES FOR AGENT (DO NOT VIOLATE):

  1. `git pull --rebase` before ANY action.
  2. Captured task → status ⬜→🔄 + indicate your files → commit immediately:
     "status: 🔄 Task {N} captured by Agent-{X}"
  3. After each significant change to a file inside a task (created a file,
     passed the RED test, passed the GREEN test, refactor done) - a separate commit.
     Every meaningful action = git commit + push.
  4. Done task → status 🔄→✅ + commit hash → commit:
     "status: ✅ Task {N} done by Agent-{X} [{hash}]"
  5. Done phase → unlock next phase → commit:
     "status: ✅ Phase {X} DONE — Phase {Y} unlocked"
  6. Done release → unlock next release → commit.
  7. DO NOT touch files with someone else's 🔄.
  8. Need a file from someone else's zone → section "Need coordination" below.
     Set yourself a status ⏸️ and continue the wait-loop.
  9. If there are no available tasks → **WAIT-LOOP** (see below),
     DO NOT switch off.

═══════════════════════════════════════════════════════════════════════
WAIT-LOOP (when there are no tasks for me):

  while True:
    git pull --rebase
    check the board:
      is there ⬜ with my owner and phase 🏃 ACTIVE?  → grab, get out of the loop.
      are all my tasks in the ✅ phase?  → coordinate phase transition.
      otherwise → status: 💤 idle, sleep 60s, continue.
    every 5th cycle (≈5 min) → commit updating your status
    on the board (timestamp) so that others can see “I’m alive”:
      "status: 💤 Agent-{X} alive — waiting for phase {Y}"

  DO NOT leave the process under any circumstances. The cycle spins until it becomes obvious
  user commands "stop" or until all releases are closed.
═══════════════════════════════════════════════════════════════════════
-->

---

## 📊 General progress

```
{Release A}   ░░░░░░░░░░░░░░░  0/{N_tasks}  🏃 ACTIVE
{Release B}   ░░░░░░░░░░░░░░░  0/{N_tasks}  🔒 LOCKED
...
──────────────────────────────────────────────────────
TOTAL 0/{TOTAL}
```

---

# 🚀 {Release Name}
**Status:** 🏃 ACTIVE · Thread: `release/{slug}`

**Verticals:**
- **Agent-1** — {one-line description of area of responsibility}
- **Agent-2** — {one-line description}
- **Agent-3** — {one-line description}
- ...

## {Release} · Phase A — {Title} (Tasks {N}–{M})
**Type:** {SEQUENTIAL | PARALLEL} · Branch: `{slug}/phase-a` · **Status: 🏃 ACTIVE**

> {Who works in this phase. Who's in the wait-loop.}

| # | Problem | Owner | Status | Files (temporarily 🔄 - do not touch) | Commit |
|---|--------|-------|--------|----------------------------------|--------|
| {N} | {Title} | Agent-1 | ⬜ | `{file1}` `{file2}` | — |
| {N+1} | {Title} | Agent-2 | ⬜ | `{file3}` | — |

**Who is in the wait-loop at this phase:** {if there is - a list of agents who
waiting for the next phase to unlock}

---

## {Release} · Phase B — {Title} ⚡ IN PARALLEL (Tasks {N}–{M})
**Type:** PARALLEL · **Status: 🔒 LOCKED** (waiting for Phase A)

| # | Problem | Owner | Branch | Status | Files | Commit |
|---|--------|-------|-------|--------|-------|--------|
| {N} | {Title} | Agent-1 | `{slug}/{branch1}` | ⛔ | `{files}` | — |
| {M} | {Title} | Agent-2 | `{slug}/{branch2}` | ⛔ | `{files}` | — |
| {K} | {Title} | Agent-3 | `{slug}/{branch3}` | ⛔ | `{files}` | — |

---

✅ **Done criteria {Release}:** {KPI from plan}

---
````

Repeat the release block for each release. All releases after the first one begin
with `🔒 LOCKED (waiting for {Prev Release} ✅ DONE)`.

Be sure to add at the end of the board:

````markdown
## 🤝 Coordination needed

<!--
  If an agent needs a file/type/contract from a foreign zone, add it here.
  Format: - [ ] R{N} Task {M}: what you need → @Agent-{X}
  When the request is closed, check the checkbox and commit.
-->

- [ ] _(still empty)_

---

## 🗒️ Agent activity log

<!--
  A quick log of who is doing what NOW. Updated for every
  change of status. Helps to see the entire swarm in one line.
-->

| When (UTC) | Who | What | Commit |
|-------------|-----|-----|--------|
| YYYY-MM-DDTHH:MM | — | board initialized | {hash} |

---

## ⚡ Concurrency map (cheat sheet)

```
{Release A}  Phase A(Agent-1)  ──→  Phase B(⚡ Agent-1, Agent-2, Agent-3)  ──→  Phase C(Agent-2)
              T1: Agent-1            T3: Agent-1                                  T7: Agent-2
              T2: Agent-1            T4: Agent-2
                                     T5: Agent-3
                                     T6: Agent-3
{Release B}  Phase A(⚡ all)  ──→  ...
```
````

---

## Phase Design Rules

These rules make the board safe for parallel work.

**Dependency rule.** A phase starts only when all tasks of the previous one are
✅. If task X reads the output of task Y, X goes into phase later than Y.

**Ownership exclusivity.** Within the parallel phase, each file path
belongs to exactly one agent. Two agents never touch the same one
same file in one phase.

**N-aware branching.**
- Sequential phases with one owner: `release/r1.0/phase-a`
- Parallel phases - a separate branch for the agent:
  ```
  release/r1.0/agent-1-db
  release/r1.0/agent-2-shared
  release/r1.0/agent-3-bot
  ```

**Money/Security-critical marking.** `💰 MONEY-CRITICAL` or
`🛡️ SECURITY-CRITICAL` tasks - in their phase, one agent, no parallelism
inside. Pair programming allowed (SHARED owner) - both agents
They look, one writes.

**Lock cascade.** Each release after R1.0 starts
`🔒 LOCKED (waiting for {prev} ✅ DONE)`. Each phase after Phase A -
`🔒 LOCKED (waiting for Phase {prev})`.

**Wait-loop is the default.** If there are no free tasks for the agent in the phase -
it does NOT turn off, it is in `💤 idle` and spins the wait-loop (see below).

---

## Agent Runtime Protocol

When an agent works on the board, it follows this protocol **line by line**.
These rules are in the template comments (above), and again here for the orchestrator.

### Before the session starts

```bash
git pull --rebase origin main
```

Open `AGENTS-STATUS.md`. Find your number `Agent-{X}` in the table above.
Find your tasks in the current phase of the current active release.

### Capture task (claim)

1. Find the task line with your name in Owner and status `⬜`.
2. Change `⬜` → `🔄`.
3. In the "Files" column, enter your list (if not already listed).
4. Set yourself the status `🔄 working` in the top table of agents.
5. Add a line to **Activity Log**:
   `{timestamp} | Agent-{X} | claim T{N}: {short title} | —`
6. **Commit immediately, before any code edits:**

```bash
git add AGENTS-STATUS.md
git commit -m "status: 🔄 Task {N} captured by Agent-{X}"
git push
```

If push is rejected (race condition: someone got ahead) - `git pull --rebase`,
check if someone else has taken over the task, select the next one ⬜.

### While running (intermediate commits)

**Every meaningful action = git commit + push.** This is the rule.

| Action | Commit message |
|----------|-------------------|
| Created a test file RED | `test({scope}): RED — failing test for {N}` |
| Passed RED → GREEN | `feat({scope}): GREEN — minimal impl for T{N}` |
| Refactor green→green | `refactor({scope}): T{N} — cleanup` |
| Changed migration | `chore(db): T{N} — migration {desc}` |
| Updated config/env | `chore({scope}): T{N} — config update` |
| Corrected types | `chore(types): T{N} — fix types` |
| Document/comments | `docs({scope}): T{N} — notes` |

Never accumulate “many changes into one commit by the end of the task.”
Atomicity of commits is the main protection against loss of work.

### Completing the task (done)

1. The final commit of the task:
```bash
git add {all task files}
git commit -m "{type}({scope}): T{N} — {one-line summary}"
git push
# write a short hash (git rev-parse --short HEAD)
```

2. Open `AGENTS-STATUS.md`:
   - Task status `🔄` → `✅`
   - Insert a short hash in the "Commit" column in the format `[abc1234]`
   - In **Activity Log** add the line:
     `{timestamp} | Agent-{X} | done T{N} | {hash}`
   - Update the release progress bar (✅ count / total)
   - Give yourself a status. Options:
     * There is another ⬜ with my Owner in the same phase → `🔄 working` after
       capturing the next task
     * In this phase all my tasks are ✅, but the phase is not all ✅ → `✅ done-phase`
       (wait for the others, enter the wait-loop)
     * All my tasks in all active phases ✅ → `💤 idle`, wait-loop

3. Board update commit:
```bash
git add AGENTS-STATUS.md
git commit -m "status: ✅ Task {N} done by Agent-{X} [{hash}]"
git push
```

### End of phase

Check: all phase lines are `✅`? If yes:
1. Change the phase status `🏃 ACTIVE` → `✅ DONE`.
2. Find the next phase. Change it `🔒 LOCKED` → `🏃 ACTIVE`. Change it
   statuses of her tasks `⛔` → `⬜` (now available for capture).
3. If the next phase is parallel, all agents with tasks in it can
   start at the same time. Those who do not have tasks in it → wait-loop.
4. Commit:
```bash
git add AGENTS-STATUS.md
git commit -m "status: ✅ Phase {X} DONE — Phase {Y} unlocked"
git push
```

### Completion of release

All release phases `✅ DONE`. Then:
1. Change the release status `🏃 ACTIVE` → `✅ DONE`.
2. Unlock the next release: `🔒 LOCKED` → `🏃 ACTIVE`, unlock
   its Phase A.
3. Update the general progress bar.
4. Commit:
```bash
git add AGENTS-STATUS.md
git commit -m "status: ✅ {Release} DONE — {Next Release} unlocked"
git push
```

### Conflict-avoidance

**Never** edit a file that has someone else's `🔄` on its board.

If you need a file from someone else’s zone, add an entry to “Need coordination”:

```markdown
- [ ] R1.0 T5 (Agent-2 is needed from Agent-1): export `JwtPayload` from
      `apps/api/src/auth/` to `packages/shared/`. → @Agent-1
```

Set yourself the status `⏸️ blocked` and **continue wait-loop** (don't switch off).
Agent-1, reading the board on the next pull, either does what is asked (and
closes the checkbox) or responds with an alternative in the same section.

### Progress bar format

```
R1.0 Paid Alpha   ████████░░░░░░░  8/15  🏃 ACTIVE
```

Update after every `✅`. We only count ✅ tasks. 15 characters
in total. `█` = done, `░` = todo.

---

## Wait-Loop Protocol

This is a **separate runtime submode** in which the agent enters when it has
no task available. The main rule: **don't switch off**.

### What does "no task available" mean?

- All my tasks are in the current active phase - ✅
- The current active phase is not yet complete ✅ (other agents are working)
- Next phase `🔒 LOCKED`

### Cycle

```
loop:
  1. git pull --rebase
  2. read AGENTS-STATUS.md
  3. find everything ⬜ with my Owner in phases with status 🏃 ACTIVE
  4. if there is → exit the loop, go to Runtime "Task Capture"
  5. otherwise → check if all the tasks of my current phase are ✅?
     - if everything ✅ And I can unlock the next phase (logically
       Phase transition) → try to seize the "Phase unlocker" role:
         a. pull again, make sure that after my previous done
            no one unlocked
         b. if the phase is still 🏃 ACTIVE and all tasks ✅ → unlock
            next (see "End of phase")
  6. set your status in the top table:
       💤 idle - no conflicts, just waiting
       ⏸️ blocked — I have a request in “Need coordination”
       ✅ done-phase — all my tasks for this phase are done, I’m waiting for the rest
  7. sleep 60 seconds
  8. every ~5 minutes (every 5th loop iteration):
       update the timestamp of your status in the board and commit:
         "status: 💤 Agent-{X} alive @ {timestamp}"
  9. goto loop
```

### When to exit the wait-loop

**Automatic:**
- Appeared ⬜ for me → Runtime "Task Capture"
- The user gave an explicit command "stop" / "stop" / "exit"
- All board releases → `✅ DONE` (work completed; final commit
  state and can be completed)

**DO NOT go out:**
- Just "boring"
- No work for 10 minutes (that's normal, keep waiting)
- Network error when pulling (retry after 60s)
- Other people's commits to the board (that's normal, continue)

### Heartbeat commit

Every ~5 minutes (5th loop cycle) make a heartbeat commit to the journal
activities:

```bash
git add AGENTS-STATUS.md
git commit -m "status: 💤 Agent-{X} alive @ {ISO timestamp}"
git push
```

This lets the other agents see that you're alive and still watching.
at the board. Without heartbeat, after 30 minutes other agents must count
you `🛑 stopped` (but do not cancel your `🔄` tasks automatically - ask
at the user).

### Single-agent mode (N=1)

When N=1 wait-loop is practically not needed (there are no other participants to wait).
But if, according to dependency logic, the task requires an external event (for example,
passing CI after push) - Agent-1 can also go into a short wait-loop,
check CI, update the board. This is a valid pattern.

---

## Concurrency map

The map at the end of the board is a compact ASCII chart of dependencies for all releases,
so that all agents can see “what after what” and “who is where” on one screen.

Example for N=3:
```
R1.0  PhA(Agent-1)   ──→  PhB(⚡ all)         ──→  PhC(Agent-2)       ──→  PhD(⚡ Agent-1,3)
                          T3: Agent-1                T7: Agent-2             T10: Agent-1
                          T4: Agent-2                T8: Agent-2             T11: Agent-3
                          T5: Agent-3                                        (Agent-2 💤)
                          T6: Agent-3
                          (Agent-1 second slot)

R1.1  PhA(Agent-2)   ──→  PhB(⚡ all)         ──→  ...
```

Mark `(Agent-X 💤)` if in the parallel phase the agent does not have a task and he
will be in wait-loop.

---

## Common mistakes

1. **Overlapping files in parallel phase** - two agents in the Owner for one
   way. Before committing the board in Generator, check each parallel phase.
2. **Forgot git pull at the beginning of the action** - the status is moving apart.
3. **Captured a task without commit `🔄`** - other agents do not see that the task
   busy, they can start working in parallel.
4. **A lot of changes have accumulated into one commit** - loss of atomicity. On
   Each meaningful action is a separate commit.
5. **Turned off in wait-loop** - this is **forbidden**. Wait-loop spins until
   an explicit "stop" from the user or closing all releases.
6. **Unlocked the next phase too early** - only when EVERYONE is ✅ in
   current phase, not only our own.
7. **Do not mark coordination** - if you need a file from someone else’s zone, section
   “Need coordination” is a must; You cannot edit the file directly.
8. **Progress bar is not updated** - update after each ✅.
9. **Heartbeat skipped** - without it, others think you're stuck.
10. **Not recorded in the activity log** - every capture/done/heartbeat
    must be a log line.

---

## Connection with other skills

```
project-architect → ARCHITECTURE.md + ROADMAP.md + ADRs
        ↓
plan expansion mode → R1.0.md, R1.1.md, R2.md, ... (file-by-file tasks)
        ↓
plan-and-coordinate ← you are here
        ↓
AGENTS-STATUS.md (N agents)
        ↓
N agents work in parallel according to the protocol
       (with git-commit-per-action and wait-loop)
```
