# Full Suite for Claude Code

A complete agent + workflow package for [Claude Code](https://claude.ai/download). Drop the `.claude/` directory and `CLAUDE.md` into your project and Claude Code automatically gets:

- **109 specialized agents** — domain experts loaded as needed (PostgreSQL, Swift, Kubernetes, security review, frontend, etc.)
- **Single-model workflow** — disciplined planning, research-before-implementation, self-verification, iteration to convergence
- **Persistent memory** — knowledge that survives across sessions; per-task session state with checkpoints
- **Deep web search** — 50+ results per query with topic-aware bonus sources (arXiv, PubMed, Hacker News, Stack Overflow, …)

Everything is **plain Markdown + a couple of POSIX/PowerShell scripts**. No daemons, no subprocess spawning, no configuration server. The agents are checklists Claude reads into its own context — not separate processes.

## Quick Start

```bash
git clone https://github.com/itohnobue/full-suite-claude-code
cp -R full-suite-claude-code/.claude /path/to/your/project/
cp full-suite-claude-code/CLAUDE.md /path/to/your/project/
```

Or, if you already have a project `CLAUDE.md`, **append** the contents of this `CLAUDE.md` to it instead of overwriting.

That's it. Open your project with Claude Code and the workflow activates automatically.

> Tip: add `tmp/`, `.claude/knowledge.md`, and `.claude/session.md` to your project's `.gitignore` if you'd rather not commit memory files.

## Agents

Browse `.claude/agents/INDEX.md` for the full directory grouped by domain. Quick reference:

| Category | Count | Examples |
|----------|-------|----------|
| Language Implementation | 22 | python-pro, golang-pro, rust-pro, typescript-pro, c-pro, cpp-pro, java-pro, ruby-pro, scala-pro |
| Web Frameworks | 10 | react-pro, nextjs-pro, vue-pro, django-pro, fastapi-pro, rails-pro, spring-boot-pro |
| Architecture & Design | 9 | backend-architect, api-designer, microservices-architect, event-sourcing-architect |
| DevOps & Infrastructure | 11 | devops-engineer, kubernetes-architect, cloud-architect, terraform-pro, sre-engineer |
| Security | 6 | security-reviewer, penetration-tester, threat-modeling-pro, backend-security-coder |
| Database | 5 | postgres-pro, sql-pro, database-architect, database-optimizer, database-reviewer |
| Testing & Quality | 5 | code-reviewer, tdd-guide, test-automator, qa-pro, e2e-runner |
| AI & ML | 5 | ai-engineer, ml-engineer, mlops-engineer, prompt-engineer, llm-architect |
| Frontend & Mobile | 5 | frontend-developer, ios-pro, ui-designer, ux-designer, mobile-developer |
| Documentation | 7 | documentation-pro, technical-writer, docs-architect, tutorial-engineer |
| Incident & Troubleshooting | 4 | incident-responder, debugger, devops-troubleshooter, devops-incident-responder |
| Specialized | 20 | build-engineer, cli-developer, product-manager, web-searcher, agent-organizer, … |

**How they're used.** When Claude tackles a non-trivial subtask, it reads the matching agent's `.md` file (e.g. `.claude/agents/postgres-pro.md`), applies the checklists/anti-patterns to the current work, then discards the instructions. One agent at a time, fresh re-read every time. Hybrid tasks split into subtasks, each with the appropriate agent.

**Selection rule.** Most specialized wins — `postgres-pro` over `database-optimizer`, `swift-pro` over a generic alternative.

## Memory System

Two tiers, both stored as plain Markdown so they're trivially diffable and version-controllable:

- **Knowledge** (`.claude/knowledge.md`) — permanent. Patterns, gotchas, architecture notes, design decisions. Survives across sessions.
- **Session** (`.claude/session.md`) — temporary. Current task: plan, todos, progress, checkpoints, blockers.

```bash
# Save a discovery
.claude/tools/memory.sh add gotcha "psycopg2 needs libpq-dev on Ubuntu" --tags postgres,ubuntu

# Recall context before starting work
.claude/tools/memory.sh context "postgres connection"

# Track session progress
.claude/tools/memory.sh session add todo "Implement auth middleware" --status pending

# Checkpoint (after every significant step)
.claude/tools/memory.sh session add context "CHECKPOINT: auth middleware | DONE: route+model | NEXT: tests"
```

Multiple Claude Code instances stay separated via `-S session-name` or the `MEMORY_SESSION` env var.

## Web Search

Deep web search via `web_search.sh` — DuckDuckGo + Brave fallback, anti-bot handling, smart content extraction, sentence-level BM25 compression, cross-page deduplication, and topic-aware bonus sources:

```bash
.claude/tools/web_search.sh "React server components best practices" --tech
.claude/tools/web_search.sh "CRISPR delivery methods" --sci --med
.claude/tools/web_search.sh "Kalman filter implementations" --sci
```

| Flag | Adds sources |
|------|-------------|
| `--tech` | Hacker News, Stack Overflow, Dev.to, GitHub |
| `--sci` | arXiv, OpenAlex |
| `--med` | PubMed, Europe PMC, OpenAlex |

`CLAUDE.md` instructs Claude to route ALL internet research through this tool — no rogue `curl`, no built-in WebSearch/WebFetch — so results are consistent and rate-limit-friendly.

Dependencies are auto-installed by [uv](https://github.com/astral-sh/uv) on first run; no manual setup needed.

## Workflow Model

The included `CLAUDE.md` configures Claude Code as a **single model handling the task end-to-end** — planning, research, implementation, verification, delivery. There is no spawning, no parallel agent batches.

Key principles:

- **Quality over speed.** No deadlines. Slow, methodical work produces correct code; speed produces bugs.
- **Research before implementation.** Read the actual reference source for the equivalent feature before designing your own. Skim project structure to understand layout before deep work.
- **Self-verification.** After every non-trivial change: re-read the diff, mentally trace control flow, run build + tests, address root causes (not symptoms), reject speculation, verify framework semantics.
- **Iteration to convergence** for high-stakes work — security audits, production changes, complex refactors. Two consecutive empty review passes = done. Default cap 3 (use 5 for high-stakes).
- **Phase-by-phase delivery.** For multi-step tasks: decompose into ordered phases, brief the user, execute one at a time, never silently skip a planned phase.

Full details in [`CLAUDE.md`](CLAUDE.md).

## Requirements

- **[Claude Code](https://claude.ai/download)** — the host CLI.
- **Python 3.9+** for the memory and web-search tools. Dependencies are managed by [uv](https://github.com/astral-sh/uv) and auto-installed on first run.
- **POSIX shell** (bash/zsh) on macOS/Linux, **PowerShell or cmd** on Windows. Both wrappers are included.

No API keys or services required for the core suite. The web search tool uses public endpoints (DuckDuckGo, Brave, arXiv, etc.) — no key needed for typical use.

## License

MIT — see [LICENSE](LICENSE).
