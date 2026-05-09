## Temporary Files

You can use the `tmp/` subfolder in the current project folder to save any temporary files if needed.
This is useful for storing intermediate results, reports, or data during multi-step workflows.

---

## Workflow Model

**You do all the work yourself.** There is no multi-agent orchestration, no spawning, no parallel agent batches. You are a single model handling the task end-to-end: planning, research, implementation, verification, delivery.

The `.claude/agents/*.md` files are **domain guidance for yourself** — specialized checklists and anti-patterns you load into your own working context for one subtask at a time. They are NOT separate processes.

**Quality over speed — ALWAYS.** Never rush, never cut corners, never try to finish faster. Slow, thorough, methodical work produces quality. Speed produces bugs. Prefer more research, more verification, more iteration over a faster turnaround. There is no deadline. The only measure of success is correct, production-ready work.

---

## Agents (Domain Guidance)

Agents folder: `.claude/agents/`. Each `.md` is a domain-specific instruction set (PostgreSQL, security review, Swift, frontend, etc.) you read before tackling a non-trivial subtask, apply, then discard.

**Rules:**
- Before any non-trivial subtask: select the best agent and read its `.md` file (fresh re-read every time)
- Load ONE agent at a time
- DO NOT use the Task / Agent tool to spawn agents — read the `.md` inline and apply it yourself
- Agent instructions are TEMPORARY — apply to the current subtask only, discard after

**Discovery:** Glob `.claude/agents/*.md` to list. Consult `.claude/agents/INDEX.md` for the full directory grouped by domain. Grep by keyword for ad-hoc lookup.

**Categories** (rough counts — see INDEX.md for the authoritative directory):

| Category | Count | Examples |
|----------|-------|----------|
| Language Implementation | 22 | python-pro, golang-pro, rust-pro, typescript-pro |
| Web Frameworks | 10 | react-pro, nextjs-pro, django-pro, fastapi-pro |
| Architecture & Design | 9 | backend-architect, api-designer, microservices-architect |
| DevOps & Infrastructure | 11 | devops-engineer, kubernetes-architect, cloud-architect |
| Security | 6 | security-reviewer, penetration-tester, threat-modeling-pro |
| Database | 5 | postgres-pro, sql-pro, database-architect |
| Testing & Quality | 5 | code-reviewer, tdd-guide, test-automator |
| AI & ML | 5 | ai-engineer, ml-engineer, prompt-engineer |
| Frontend & Mobile | 5 | frontend-developer, ios-pro, ui-designer |
| Documentation | 7 | documentation-pro, technical-writer, docs-architect |
| Incident & Troubleshooting | 4 | incident-responder, debugger, devops-troubleshooter |
| Specialized | 20 | build-engineer, cli-developer, product-manager, web-searcher, etc. |

**Selection:** Most specialized wins — postgres-pro over database-optimizer, swift-pro over a generic alternative. For hybrid tasks, split into subtasks and load a different agent per subtask.

**Subtask flow:**
1. Read the agent `.md`
2. Apply its checklists/anti-patterns to the current subtask
3. Complete the work fully and self-verify
4. Save discoveries to knowledge if non-trivial
5. Discard the agent instructions
6. Move to the next subtask (with a fresh agent if it's a different domain)

After all subtasks: compose a single report for the user.

---

## Request Workflow

For non-trivial requests:

1. **Recall context:** `./.claude/tools/memory.sh context "<keywords>"` — extract keywords from the request (entities, technologies, services, error types). MANDATORY for non-trivial tasks.
2. **Continuation check:** `memory.sh search "CONTINUATION"` — if a prior session left work in progress, resume it (see Session Continuation).
3. **Plan:** For multi-step work, decompose into ordered phases. Save the plan: `memory.sh session add plan "..."`. Brief the user on the phases before executing — do not wait for the user to ask.
4. **Research before implementation:** When introducing or modifying a component, read the ACTUAL reference source for the equivalent feature first — your mental model may have misinterpreted, oversimplified, or missed fields/logic. Skim files (structure, function names, imports, sizes) to understand the project layout before deep work. Invest time in preparation — careful research produces better results than rushed implementation.
5. **Execute phase by phase:** For each phase, identify the relevant agent `.md` (if any), apply its guidance, do the work, self-verify before moving on. Update session state (`memory.sh session update`) as phases complete. **Before starting an implementation phase**, run a quick build/test sanity check (~30 seconds) to confirm the environment works — catching a broken baseline early prevents sinking time into changes on top of it.
6. **Self-verify** every non-trivial change (see Self-Verification section).
7. **Deliver:** Before declaring done, re-check the phase plan — confirm every planned phase is complete, or explicitly mark it SKIPPED with a reason. Never silently drop a planned phase. Then summarize what changed and save discoveries to knowledge if non-trivial.

For simple single-domain tasks (no discovery needed, no architectural decisions), handle directly without the formal phase structure.

---

## Self-Verification

After completing a non-trivial change, verify your own work before declaring done. Uncritical "looks good" finishes ship bugs.

**Always:**
- **Read the changed files** — confirm the edit landed where intended and matches the goal
- **Trace execution mentally** — for control flow inside closures, callbacks, and async code, confirm `return` / `throw` / `break` / `continue` targets the intended scope, not a surrounding one
- **Run build + tests** when applicable; report failures honestly. Do not claim done if the build is broken
- **Address root causes, not symptoms** — if the problem describes a structural issue (missing handle, unmanaged lifecycle, absent synchronization), don't paper over it with a guard, try/catch, or retry. The fix must match the problem's root cause
- **Preserve design rationale** — if you remove or modify a comment explaining WHY code is structured a certain way, account for that concern in the new code. A removed design comment whose rationale isn't addressed in the replacement is a regression signal
- **Reject speculation** — don't add guards for hypothetical future changes. If a failure requires a future code change to manifest, it's not a real bug. Exception: documented TODOs/FIXMEs that explicitly mention the planned change
- **Verify framework semantics** — if a fix relies on framework-specific behavior (lifecycle, reactivity, scheduling, state management), confirm the rule before relying on it. If you can't confirm, say so instead of assuming

**For high-stakes work** (security audits, production changes, complex refactors): do multiple review passes until you stop finding new issues. Two consecutive empty passes = done. Default cap is 3 iterations (use 5 for very high-stakes security/production audits). If the cap is hit without convergence, summarize what's known, note "convergence not reached", and proceed.

**Before claiming something is missing or broken** — grep for existing guards, handlers, or implementations first. The codebase often already handles a case you assumed it didn't.

---

## Web Research

For any internet search:

1. Read agent instructions: `.claude/agents/web-searcher.md`
2. **ALL internet research must go through `web_search.sh`** — no exceptions. This means: no built-in WebSearch tool, no WebFetch tool, no `curl` against APIs, no manual GitHub API calls, no `wget`, nothing else. Every time you need information from the internet, use `./.claude/tools/web_search.sh "query"` (or `.claude/tools/web_search.bat` on Windows)
   - **One query per call** — run each query as a separate `web_search.sh` invocation. Never combine multiple queries into a single call. Run calls **sequentially** (one after another, not in parallel) to avoid hitting API rate limits
   - **Always use default options** — never add `-s`, `--max-results`, or any result-limiting flags. Let the tool use its built-in defaults
   - **Scientific queries: ALWAYS add `--sci`** for CS, physics, math, engineering, materials science, astronomy, or any non-medical academic topic. Enables: arXiv + OpenAlex.
   - **Medical queries: ALWAYS add `--med`** for medicine, clinical trials, pharmacology, biomedical, genetics, neuroscience, epidemiology, or any health/life science topic. Enables: PubMed + Europe PMC + OpenAlex.
   - **Tech queries: ALWAYS add `--tech`** for software development, DevOps, IT infrastructure, programming, startups, or any tech industry topic. Enables: Hacker News + Stack Overflow + Dev.to + GitHub.
   - **Both flags together (`--sci --med`)** for interdisciplinary queries (e.g., computational biology, bioinformatics, medical imaging AI). Use both when the topic spans science AND medicine.
   - **MANDATORY**: These flags MUST be used for ALL queries matching the above descriptions. Never omit them for relevant queries. When in doubt, add the flag — it never hurts.
3. Synthesize results into a report

**Note**: Always use forward slashes (`/`) in paths for agent tool run, even on Windows.
Dependencies handled automatically via uv.

---

## Memory System

**NEVER use MEMORY.md for anything.** MEMORY.md is Claude Code's auto-memory system and is completely separate from this project's memory system. Do not read, write, or reference MEMORY.md. Use only `knowledge.md` and `session.md` via the `memory.sh` tool.

Two-tier: **Knowledge** (`knowledge.md`) permanent, **Session** (`session.md`) temporary.

| Question | Use |
|----------|-----|
| Will this help in future sessions? | **Knowledge** |
| Current task only? | **Session** |
| Discovered a gotcha/pattern/config? | **Knowledge** |
| Tracking todos/progress/blockers? | **Session** |

### Knowledge

```bash
memory.sh add <category> "<content>" [--tags a,b,c]
```

| Category | Save When |
|----------|-----------|
| `architecture` | System design, service connections, ports |
| `gotcha` | Bugs, pitfalls, non-obvious behavior |
| `pattern` | Code conventions, recurring structures |
| `config` | Environment settings, credentials |
| `entity` | Important classes, functions, APIs |
| `decision` | Why choices were made |
| `discovery` | New findings about codebase |
| `todo` | Long-term tasks to remember |
| `reference` | Useful links, documentation |
| `context` | Background info, project context |

**Tags:** Cross-cutting concerns (e.g., `--tags redis,production,auth`). **Skip:** Trivial, easily grep-able, duplicates.

**After tasks:** State "**Memories saved:** [list]" or "**Memories saved:** None"

**Other:** `search "<query>"`, `list [--category CAT]`, `delete <id>`, `stats`

### Session

Tracks current task. Persists until cleared.

**Categories:** `plan`, `todo`, `progress`, `note`, `context`, `decision`, `blocker`. **Statuses:** `pending` → `in_progress` → `completed` | `blocked`.

```bash
memory.sh session add todo "Task" --status pending
memory.sh session show                    # View current
memory.sh session update <id> --status completed
memory.sh session delete <id>
memory.sh session clear                   # Current only
memory.sh session clear --all             # ALL sessions
```

### Multi-Session

Multiple CLI instances work without conflicts. Resolution: `-S` flag > `MEMORY_SESSION` env > `.claude/current_session` file > `"default"`.

```bash
memory.sh session use feature-auth        # Switch session
memory.sh -S other session add todo "..." # One-off
memory.sh session sessions                # List all
```

---

## Checkpoints & Recovery

For long tasks vulnerable to context compaction, save state to disk so you can resume cleanly.

**Save after every significant step:**
```bash
memory.sh session add context "CHECKPOINT: [task] | DONE: [steps] | NEXT: [remaining] | FILES: [key files] | BUILD/TEST: [commands]"
```
Keep one active checkpoint at a time (delete the previous before adding new). Under 500 chars.

**Compaction recovery — MANDATORY sequence (do ALL steps, no skipping):**
1. **Re-read CLAUDE.md in full and STRICTLY follow it** — ALWAYS, no exceptions, no partial reads. This is the #1 cause of workflow deviation after compaction
2. `memory.sh session show` — restore session state and the latest checkpoint
3. Read any plan or continuation files in `tmp/` (e.g. `tmp/plan.md`, `tmp/continuation.md`)
4. Resume from the next unfinished step

Do not rely on the conversation summary alone — read the checkpoint files.

| Checkpoint | Recovery |
|-----------|----------|
| Plan written | Read `tmp/plan.md` → start phase 1 |
| Phase N in progress | Read session checkpoint → continue from NEXT step |
| Phase N done | Read session state → start phase N+1 |
| Continuation written | Read `tmp/continuation.md` → resume next phase |

---

## Session Continuation

For tasks exceeding a single session:

1. Complete the current phase fully
2. Write `tmp/continuation.md`: original task, plan, completed phases, next phase, decisions, modified files, blockers
3. `memory.sh add context "CONTINUATION: [summary]" --tags continuation`
4. Tell the user what's done and what continues

**Pickup:**
1. `memory.sh search "CONTINUATION"` — find the entry
2. Read `tmp/continuation.md` — restore plan and state
3. Continue from the next phase. Never re-do completed prior work
4. On final phase: clean up `tmp/continuation.md` and remove the memory entry

If a task will likely need many phases or span many sessions, plan explicit session splits up front using this protocol — long sessions degrade from compaction pressure.

---

## Skills (Workflows)

Add project-specific workflows as skills in `.claude/skills/<skill-name>/SKILL.md`. Each skill is invoked with `/<skill-name>`. This is where you place repetitive multi-step procedures (release, deploy, audit, etc.) so they can be triggered as a single command.
