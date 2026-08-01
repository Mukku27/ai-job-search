# IDE/CLI-Native LLM Architecture — Reverse Engineering & Porting Guide

> Extracted from the `ai-job-search` repository. Language-agnostic. Intended to be handed to another LLM (or human) to refactor a different project onto the same pattern: use the coding tool's built-in LLM instead of OpenAI / Anthropic / Gemini / OpenRouter HTTP APIs.

---

## 0. One-sentence verdict

**This repository does not call an LLM.** It is a configuration + prompt + tool pack that the host coding agent (Claude Code, Cursor, Codex, Gemini CLI, etc.) loads, then the *host* runs the model, streams tokens, executes tools, and manages auth. Application code is inverted: the LLM calls your tools; your code never calls the LLM.

---

## 1. What this repo actually is

| Layer | Role | Speaks to LLM? |
|-------|------|----------------|
| Host runtime (Claude Code / Cursor / …) | Agent loop, model I/O, tool execution, auth, streaming UI | **Yes** (owns the connection) |
| Project instructions (`CLAUDE.md`, `AGENTS.md`) | Persistent system/context injected by host on session start | Indirect (host injects) |
| Skills (`.claude/skills/**/SKILL.md`, `.agents/skills/**/SKILL.md`) | Triggered procedure packs + tool allowlists | Indirect |
| Slash commands (`.claude/commands/*.md`) | Named workflows; `$ARGUMENTS` substituted by host | Indirect |
| Custom agents (`.claude/agents/*.md`) | Sub-agent personas; host spawns via Agent tool | Indirect |
| Thin pointers (`.cursor/commands/*.md`) | One-liner adapters that `Read` canonical `.claude/commands/*` | Indirect |
| Deterministic CLIs (Bun TS under `.agents/skills/*/cli`) | Fetch/parse job boards; print JSON/text to stdout | **No** |
| Helper scripts (`salary_lookup.py`, `tools/verify_pdf.py`, …) | Domain utilities invoked via Bash by the agent | **No** |
| Durable state files (CSV, JSON, `.tex`, profile markdown) | Conversation-external memory | **No** |

There is **no** `openai` / `anthropic` / `litellm` / `ChatCompletion` client in application logic. The only `subprocess` uses are for PDF verification / LaTeX / local CLIs — never for invoking a model.

---

## 2. How the repository "communicates" with the native LLM

### 2.1 Mechanism: host-in-the-loop (not SDK / not HTTP from the app)

```
┌─────────────────────────────────────────────────────────────────┐
│  USER                                                            │
│   types: /apply https://jobs.example/123                         │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│  HOST CODING AGENT (Claude Code, Cursor Agent, Codex, …)         │
│  • Already authenticated (subscription or user-managed API key)  │
│  • Owns: model selection, streaming, retries, tool runtime       │
│  • Loads: CLAUDE.md / AGENTS.md / skills / command markdown      │
│  • Substitutes: $ARGUMENTS ← user-provided URL/text              │
│  • Runs agent loop: think → tool_call → observe → … → answer     │
└───────────┬─────────────────────────────┬───────────────────────┘
            │                             │
            │ tool calls                  │ final assistant message
            ▼                             ▼
┌───────────────────────────┐   ┌─────────────────────────────────┐
│  HOST-PROVIDED TOOLS      │   │  USER-VISIBLE OUTPUT            │
│  Read, Write, Edit, Glob, │   │  Fit score, checklists, paths   │
│  Grep, Bash, WebFetch,    │   └─────────────────────────────────┘
│  WebSearch, Agent,        │
│  AskUserQuestion, …       │
└───────────┬───────────────┘
            │ Bash(...)
            ▼
┌───────────────────────────┐
│  REPO DETERMINISTIC CODE  │
│  bun run …/cli.ts search  │
│  python salary_lookup.py  │
│  lualatex / xelatex       │
│  pdftotext                │
└───────────────────────────┘
```

**Not used by this repo for LLM I/O:**

- Application-side Anthropic/OpenAI/Gemini SDKs
- Direct HTTPS to `api.openai.com` / `api.anthropic.com` / OpenRouter
- Local OpenAI-compatible servers (Ollama, etc.) as the primary brain
- Custom IPC sockets or gRPC to a model
- MCP as the *LLM transport* (MCP may exist in the *host* for other tools; this repo's LLM path is the host agent itself)

**What *is* used:**

1. **Filesystem contracts** the host knows how to load (skills, commands, `CLAUDE.md`, `AGENTS.md`).
2. **Host tool surface** declared in skill/command frontmatter (`allowed-tools`).
3. **Subprocess tools** (Bash → Bun/Python) for non-LLM work.
4. **Host Agent tool** for multi-agent (drafter parent + reviewer child).

### 2.2 Mental model: invert the call stack

| Traditional API app | This architecture |
|---------------------|-------------------|
| `main()` calls `llm.chat(messages)` | User/host calls LLM; LLM calls `Bash`/`Read`/`Write` |
| You own request/response JSON | Host owns request/response; you own *prompts + tools* |
| You stream tokens into your UI | Host streams into its chat UI |
| You store API keys | Host stores credentials; repo ships none |

---

## 3. Prompt, context, files, images, and tool-call plumbing

### 3.1 Context injection layers (outer → inner)

1. **Host global** — model, user settings, subscription tier (outside repo).
2. **Project instructions** — `CLAUDE.md` (Claude Code convention) + `AGENTS.md` (portable thin pointer / multi-runtime entry).
3. **Skills** — YAML/MD frontmatter + body. Host matches trigger phrases or explicit `Skill(name)` permission and injects skill text when relevant.
4. **Slash command body** — entire `.claude/commands/<name>.md` becomes the active workflow prompt; `$ARGUMENTS` is the user suffix after `/name`.
5. **On-demand file reads** — agent uses `Read`/`Glob`/`Grep` to pull profile files, templates, trackers into context.
6. **Inline sub-agent prompts** — parent agent embeds draft text in the child Agent prompt (fresh context; no shared memory).
7. **Tool results** — stdout/stderr of Bash, WebFetch HTML/markdown, WebSearch snippets — appended as tool-result messages by the host.

### 3.2 How each modality is passed

| Modality | How it reaches the model |
|----------|---------------------------|
| System / standing instructions | Host auto-loads `CLAUDE.md` / `AGENTS.md` |
| Workflow procedure | Slash command markdown or skill body |
| User input | Chat message and/or `$ARGUMENTS` |
| Files | Agent `Read` tool (path → text/bytes in tool result) |
| Images / PDF pages | Host `Read` on PDF (vision-capable models inspect pages); not a separate vision API in-repo |
| Structured data from portals | Bun CLI prints JSON; agent parses from Bash tool result |
| Tool calls | Host parses model tool-call blocks; executes; returns results |
| Multi-agent handoff | Parent uses **Agent** tool; child gets a self-contained prompt string |

### 3.3 Skill frontmatter contract (portable + Claude-native)

Example pattern from portal skills:

```yaml
---
name: linkedin-search
version: 1.0.0
description: >
  Trigger phrases and when to use…
context: fork          # optional: run skill in forked context
enabled: true
allowed-tools: Bash(bun run .agents/skills/linkedin-search/cli/src/cli.ts *)
---
```

Orchestrator skills add broader host tools:

```yaml
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(...), WebFetch, WebSearch, Agent, AskUserQuestion
```

**Design principle:** declare the *minimum* tool surface. Permissions are also mirrored in `.claude/settings.json` for auto-approval of safe Bash patterns.

### 3.4 Thin-pointer adapters (multi-IDE)

Canonical workflows live once under `.claude/commands/`. Cursor (and similar) get stub commands:

```text
Read and follow the full workflow in `.claude/commands/apply.md` exactly, in order.
Wherever that file refers to `$ARGUMENTS`, use the job posting I provide in this message…
```

**Principle:** never duplicate the long procedure; point at it. Same idea in `AGENTS.md` for Codex / Antigravity / Gemini CLI discovery of `.agents/skills/`.

### 3.5 Custom sub-agents

`.claude/agents/<name>.md` = persona + methodology + optional `model:` hint. Invoked when the host/Agent router matches description, or when a command tells the parent to spawn `general-purpose` / named agents.

---

## 4. Streaming, parsing, and returning results

### 4.1 Streaming

**Owned entirely by the host.** The repo never implements SSE/WebSocket parsers, token buffers, or partial JSON stream coalescing for model output.

### 4.2 Parsing

Two different "parsers" exist:

1. **Host:** parses model tool-call envelopes (whatever format the host uses internally).
2. **Agent-as-parser (prompted):** workflows instruct the model to interpret CLI JSON, fill tables, apply structured reviewer JSON (`old_string` / `new_string` edits).

Deterministic CLIs should emit **stable, machine-friendly stdout** (JSON + exit codes) so the model can parse reliably. Retries for flaky networks belong in the CLI (backoff), not in an app-level LLM client.

### 4.3 Returning to the "application"

There is no application server waiting for a completion. "Return" means:

1. **Side effects on disk** — Write/Edit created `cv/main_<company>.tex`, updated `job_search_tracker.csv`, etc.
2. **Chat message to the user** — fit scores, checklists, file paths, next-step suggestions (format specified in the command markdown).
3. **AskUserQuestion gates** — workflow pauses until the user confirms (e.g. proceed after fit evaluation).

---

## 5. Authentication without repo-managed API keys

| Concern | Who handles it |
|---------|----------------|
| Claude / Anthropic credentials | Claude Code login: Pro/Team subscription **or** user-configured Anthropic API key in the CLI — **not in this repo** |
| Cursor model access | Cursor account / subscription |
| Codex / Gemini CLI | Their respective CLIs' auth |
| Secrets in git | Forbidden; `.gitignore` + `tools/security_guards.py` protect personal data and permission widening |
| Repo `settings.json` | **Tool permissions only** — allowlisted Bash/Skill entries, zero keys |

```json
{
  "permissions": {
    "allow": [
      "Skill(job-application-assistant)",
      "Bash(bun run:*)",
      "Bash(python salary_lookup.py:*)",
      "Bash(python3 salary_lookup.py:*)",
      "Bash(pdftotext:*)"
    ]
  }
}
```

**Principle:** the product is a *workspace the user opens inside an already-authenticated agent*. Shipping API keys would be the wrong abstraction and a supply-chain hazard (every fork would execute your permission grants).

---

## 6. IDE/CLI-specific vs generic abstractions

### 6.1 Host-specific (expect adapters per runtime)

| Artifact | Purpose |
|----------|---------|
| `.claude/settings.json` | Claude Code permission allowlist |
| `.claude/commands/*.md` | Claude Code slash commands (reference implementations) |
| `.claude/skills/**` | Claude Code skills + methodology packs |
| `.claude/agents/*.md` | Claude Code custom agents |
| `CLAUDE.md` | Claude Code project memory / instructions |
| Host tool names (`AskUserQuestion`, `Agent`, `WebFetch`, …) | Depend on host capability set |
| `.cursor/commands/*.md` | Thin pointers into `.claude/commands` |

### 6.2 Generic / portable (reuse across languages & hosts)

| Artifact | Purpose |
|----------|---------|
| `AGENTS.md` + `framework_version` | Cross-runtime entry + upgrade tracking |
| `.agents/skills/*/SKILL.md` | Portable Agent Skills discovery format |
| `.agents/skills/*/cli` | Deterministic TypeScript/Bun tools (any agent that can Bash) |
| Domain methodology markdown (`01-`…`07-`) | Rubrics, style guides, templates — pure content |
| Durable state schemas (CSV columns, `seen_jobs.json`) | Application memory |
| LaTeX / document templates | Output artifacts |
| Security allowlists & CI guards | Supply-chain policy as code |

### 6.3 The thin-pointer design principle

```
┌──────────────┐     points to      ┌────────────────────────────┐
│ Cursor cmd   │ ─────────────────► │ .claude/commands/apply.md  │
│ Codex AGENTS │ ─────────────────► │ .claude/skills/…           │
│ Gemini fork  │ ─────────────────► │ same canonical specs       │
└──────────────┘                    └────────────────────────────┘
         Never copy-paste the long workflow into each runtime tree.
```

Upstream policy: Claude Code is the *reference* runtime; other runtimes keep thin pointers or live in forks rather than duplicating untested parallel command trees.

---

## 7. Complete request/response lifecycle (example: `/apply`)

```
1. PRECONDITION
   User has Claude Code (or Cursor) installed and authenticated.
   Repo cloned; optional Bun deps installed for portal CLIs.

2. SESSION START
   Host opens workspace → injects CLAUDE.md (+ AGENTS.md as applicable)
   → indexes/discovers skills under .claude/skills and .agents/skills.

3. USER REQUEST
   User: /apply https://company.com/jobs/ai-engineer
   Host loads .claude/commands/apply.md
   Host sets $ARGUMENTS = "https://company.com/jobs/ai-engineer"

4. STEP 0 — INGEST
   Model → WebFetch(url) → host returns page text
   Model extracts company, role, location, language

5. STEP 1 — EVALUATE (drafter = same parent agent)
   Model → Read(evaluation framework + profile)
   Optional: Bash(python salary_lookup.py "Company" --json)
   Model → user-facing fit report
   Model → AskUserQuestion("proceed?") 
   If no → STOP (response is chat only)

6. STEP 2 — DRAFT
   Model → Read(style + templates + example .tex)
   Model → Write(cv/main_<company>.tex)
   Model → Write(cover_letters/cover_<company>_<role>.tex)
   Keep draft text in working memory (prompted: do not re-Read)

7. STEP 3 — REVIEW (child agent)
   Model → Agent(tool): spawn general-purpose reviewer
   Prompt includes INLINE job posting + CV + cover letter text
   Child → WebSearch/WebFetch company research
   Child → returns Part A JSON edits + Part B narrative critique
   Host returns child output as tool result to parent

8. STEP 4 — REVISE
   Parent → Edit(file, old_string, new_string) for Part A
   Parent applies Part B with judgment; re-verifies company claims via WebFetch

9. STEP 5 — VERIFY (deterministic + vision)
   Bash(lualatex …); Bash(xelatex …)
   Read(PDF) visual layout checks
   Optional: Bash(pdftotext …) ATS text-layer checks
   Loop Edit → compile until checklist passes

10. STEP 6 — PRESENT
    Chat: verification checklist, key decisions, file paths, next steps
    Durable artifacts remain on disk for later /interview /outcome
```

**Conversation state:** host session transcript (ephemeral across sessions unless the host has memory plugins).  
**Application state:** files on disk (tracker CSV, seen_jobs JSON, profile markdown, generated TeX/PDF).

---

## 8. Error handling, retries, sessions, context, conversation state

### 8.1 Error handling & retries

| Failure | Strategy in this architecture |
|---------|--------------------------------|
| Model/provider outage | Host responsibility (user retries in UI) |
| Tool/Bash failure | Command markdown: fix and re-run; or graceful skip (salary tool, pdftotext) |
| Portal HTTP flake | CLI-level backoff/retry inside Bun helpers |
| Bad LaTeX / layout | Explicit iterate-until-clean loop in the workflow |
| Fabrication risk | Hard prompt rules + reviewer must not invent skills |
| Permission denied | Host prompts user; settings.json pre-approves only narrow Bash |

There is **no** app-level exponential retry wrapper around `chat.completions`.

### 8.2 Session management

- **Host session** = one chat thread with tool history.
- **Workflow session** = one slash-command execution; gates via AskUserQuestion.
- **Cross-session continuity** = files (`job_search_tracker.csv`, `seen_jobs.json`, profile under `.claude/skills/...`), not a vector DB of chat turns (unless the *host* adds memory — e.g. claude-mem — which is orthogonal).

### 8.3 Context management (prompted techniques)

Encoded in `/apply` and skills:

- Do not re-Read files already in context.
- Pass large drafts **inline** to sub-agents (fresh context cannot see parent files unless Read).
- Scope child file reads to essentials.
- Keep durable facts in markdown/CSV so new sessions reload cheaply.
- Token-efficiency rules are part of the *product*, not a separate RAG service.

### 8.4 Conversation state shapes

1. **Ephemeral:** host message list (system + skills + user + assistant + tool_results).
2. **Structured durable:** CSV/JSON schemas with documented columns.
3. **Document durable:** profile methodology files + generated application artifacts.
4. **Dedup state:** `seen_jobs.json` for scrape idempotency.

---

## 9. Architecture building blocks to recreate (language-agnostic)

Implement these *concepts* in any language. Language choice only matters for deterministic tools.

### 9.1 Component checklist

1. **Host Agent Runtime** (buy/use, don't build): Claude Code, Cursor Agent, Codex CLI, Gemini CLI, etc.
2. **Project Instruction File**: standing identity + constraints (`CLAUDE.md` / `AGENTS.md`).
3. **Canonical Workflow Specs**: ordered markdown procedures with explicit tool steps.
4. **Skill Packs**: trigger description + `allowed-tools` + procedure body.
5. **Runtime Adapters**: thin pointers per IDE (1–5 lines each).
6. **Deterministic Tool Binaries**: CLI programs that speak JSON on stdout; idempotent; no LLM inside.
7. **Permission Manifest**: least-privilege allowlist checked in CI.
8. **Durable State Store**: files/SQLite — anything the agent can Read/Write.
9. **Verification Loops**: compile/test/lint commands the agent must run before declaring done.
10. **Multi-Agent Pattern** (optional): parent orchestrator + child Agent with self-contained prompt + structured return schema.

### 9.2 Interfaces (pseudo-IDL)

```text
InstructionDoc {
  path: "CLAUDE.md" | "AGENTS.md"
  body: markdown
  framework_version?: semver
}

Skill {
  name: string
  description: string          // also used as trigger index
  allowed_tools: ToolPattern[] // e.g. Bash(bun run … *)
  body: markdown               // procedure
  portable_path?: ".agents/skills/<name>/SKILL.md"
}

Command {
  name: string                 // /apply
  path: ".claude/commands/apply.md"
  args_placeholder: "$ARGUMENTS"
  body: markdown               // ordered steps
}

ThinPointer {
  runtime: "cursor" | "codex" | …
  path: ".cursor/commands/apply.md"
  target: Command.path
  arg_mapping: "map user suffix → $ARGUMENTS"
}

DeterministicTool {
  invoke: "bun run …/cli.ts search …" | "python tool.py --json"
  stdin?: bytes
  stdout: JSON | text
  exit_code: 0 | non-zero
  retries: implemented INSIDE the tool
}

HostTools (provided by runtime, not by you) {
  Read, Write, Edit, Glob, Grep, Bash,
  WebFetch, WebSearch, Agent, AskUserQuestion, …
}

DurableState {
  files: [tracker.csv, seen.json, profile.md, artifacts…]
}
```

### 9.3 Invariants

1. **No LLM client in domain code.**
2. **All model I/O through the host.**
3. **Single source of truth for workflows** (thin pointers elsewhere).
4. **Tools are deterministic; reasoning is in the agent.**
5. **Auth is outside the repo.**
6. **Side effects are files (or explicit host questions), not hidden API writes.**
7. **Permissions are allowlisted and CI-enforced.**

---

## 10. Comparison: IDE-native vs traditional external LLM API

### 10.1 What changes

| Area | API integration (OpenAI/Anthropic/OpenRouter/…) | IDE/CLI-native (this pattern) |
|------|--------------------------------------------------|--------------------------------|
| Entry point | Your server/CLI calls `POST /v1/chat/completions` | User chats in host; host calls model |
| Auth | `Authorization: Bearer …` in your env | Host subscription / host-managed key |
| Streaming | You parse SSE; push to your UI | Host UI streams |
| Tool calling | You implement tool loop (or Agents SDK) | Host tool loop; you declare allowlists |
| System prompt | String in code or DB | Markdown files the host auto-loads |
| Deployment | Your process must be running | User opens folder in agent |
| Multi-tenancy | Your backend | Each user runs locally with their seat |
| Model routing | Your code picks model/provider | Host settings / agent frontmatter `model:` |
| Observability | Your logs of prompts/completions | Host transcripts; optional host plugins |
| Cost control | Your metering | User's subscription / their API bill |

### 10.2 What stays the same

- Domain rubrics, evaluation criteria, style guides
- Deterministic fetch/parse/transform tools
- Output schemas (JSON from CLIs, document templates)
- Human-in-the-loop confirmation points
- Verification (tests, compile, lint) as gates
- Need for clear prompts and structured tool contracts
- Separation of "reason" vs "act"

### 10.3 Reuse vs replace when migrating

**Reuse as-is (or nearly):**

- Business rules / scoring rubrics → methodology markdown
- HTTP scrapers / DB access / PDF compile → DeterministicTool CLIs
- Templates (LaTeX, HTML, email) → files agent Writes into
- Test suites for CLIs
- Data models / CSV/JSON schemas

**Replace:**

| Remove from API project | Replace with |
|-------------------------|--------------|
| `OpenAI`/`Anthropic` SDK wrappers | Host agent runtime |
| API key vault for the *model* | Host login (keep secrets for *your* third-party APIs if tools need them) |
| Custom agent loop / LangGraph-as-orchestrator for the main UX | Slash commands + skills |
| Streaming UI for tokens | Host chat UI |
| Prompt strings buried in Python/TS | Versioned `.md` workflows |
| Server session store for chat | Host session + durable files |
| Provider retry/fallback matrix | Host + graceful tool skips |

**Keep if tools need them (not for the brain):**

- API keys for job boards, salary APIs, email, etc. — still in env for CLIs
- MCP servers for external systems — optional; host may connect them; still not the LLM transport

### 10.4 Migration playbook (API project → IDE-native)

1. **Draw the boundary**  
   List every `llm.chat` / chain call. Each becomes either: (a) a slash-command step, or (b) a skill the host triggers, or (c) deleted because the host tool already does it (web fetch, edit file).

2. **Extract deterministic work**  
   Anything that does not need an LLM (HTTP scrape, SQL, compile, validate) → CLI with JSON stdout. Ensure exit codes and errors are clear.

3. **Promote prompts to files**  
   System + few-shot + procedure → `AGENTS.md` / `CLAUDE.md` + `commands/*.md` + `skills/*/SKILL.md`.

4. **Declare tools, don't implement the loop**  
   Map your function-tools to host tools (`Bash`, `Read`, …) or keep custom CLIs behind Bash. Set `allowed-tools` narrowly.

5. **Encode the agent loop in markdown**  
   Numbered steps, stop conditions, AskUserQuestion gates, verification checklists — the procedure *is* the orchestrator.

6. **Add thin pointers** for each target IDE so you do not fork the procedure text.

7. **Move auth** off the app; document "install Claude Code / Cursor and sign in".

8. **Persist state in files** the agent can Read/Write; stop relying on in-memory conversation for business truth.

9. **Add permission CI** so forks cannot silently widen `Bash(*)`.

10. **Delete the LLM SDK** from the main path. If you must keep an API fallback, isolate it behind an optional adapter — do not mix it into skill logic.

### 10.5 When *not* to use this architecture

- Multi-tenant SaaS where *your* servers must call models for many users
- Hard real-time latency SLAs without a human in the IDE
- Headless CI that must run the LLM without an interactive host (unless you use the host's headless CLI mode *and* accept that auth still belongs to that CLI)
- Products that must hide the agent UX and ship a custom end-user chat widget (you'll likely need APIs again for that surface)

Hybrid is valid: IDE-native for power-user workflows; API for productized customer-facing chat.

---

## 11. Minimal skeleton to recreate in any language

```text
my-agent-app/
  AGENTS.md                 # thin pointer + framework_version
  CLAUDE.md                 # project instructions (or HOST.md)
  .claude/
    settings.json           # permission allowlist
    commands/
      do-work.md            # /do-work procedure
    skills/
      main-skill/
        SKILL.md            # triggers + allowed-tools + steps
        01-domain-rules.md  # pure content
    agents/
      critic.md             # optional sub-agent persona
  .cursor/commands/
    do-work.md              # "Read .claude/commands/do-work.md…"
  .agents/skills/
    my-portal/
      SKILL.md              # portable discovery
      cli/                  # ANY language: Go/Rust/Python/TS
        main.go             # prints JSON; retries; no LLM
  state/
    tracker.csv
  tools/
    security_guards.*       # CI: permissions must match allowlist
```

**Runtime behavior (all languages):**

```text
user opens folder in host agent
user runs /do-work <args>
host injects do-work.md
model calls Bash(your-cli) and Read/Write as specified
model writes artifacts + replies in chat
```

---

## 12. Mapping this repo's pieces (quick index)

| Path | Role |
|------|------|
| `AGENTS.md` | Cross-runtime thin pointer + `framework_version` |
| `CLAUDE.md` | Candidate profile + standing rules |
| `.claude/settings.json` | Auto-approve narrow tools |
| `.claude/commands/*.md` | `/setup`, `/scrape`, `/rank`, `/apply`, `/interview`, … |
| `.claude/skills/job-application-assistant/` | Fit/CV/cover/interview methodology |
| `.claude/skills/job-scraper/` | Scrape orchestration skill |
| `.claude/skills/upskill/` | Gap analysis skill |
| `.claude/agents/gemini-research-expert.md` | Research sub-agent persona |
| `.agents/skills/*/SKILL.md` + `cli/` | Portable portal tools |
| `.cursor/commands/*.md` | Cursor thin pointers |
| `tools/security_guards.py` | Permission/gitignore/lifecycle CI guards |
| `salary_lookup.py`, LaTeX, `pdftotext` | Deterministic helpers |

---

## 13. Prompt you can give another LLM to port a project

Copy-paste:

> Refactor this project to the IDE/CLI-native LLM architecture described in `docs/IDE-NATIVE-LLM-ARCHITECTURE.md`.  
> **Hard requirements:**  
> 1) Remove direct calls to OpenAI/Anthropic/Gemini/OpenRouter (or equivalent) from the main UX path.  
> 2) Encode orchestration as markdown slash-commands and skills with `allowed-tools`.  
> 3) Move all non-reasoning work into deterministic CLIs that print JSON.  
> 4) Put standing instructions in `AGENTS.md` / `CLAUDE.md`.  
> 5) Add thin pointers for at least one secondary host (e.g. `.cursor/commands`).  
> 6) Ensure zero model API keys in the repo; document host login for auth.  
> 7) Persist business state in files the agent can Read/Write.  
> 8) Add a permission allowlist + CI guard analogous to `security_guards`.  
> Preserve existing domain logic and CLI behaviors; only replace the LLM transport and agent loop.

---

## 14. Summary diagram

```text
                 ┌──────────────────────────┐
                 │   Host Coding Agent      │
                 │   (auth + model + tools) │
                 └────────────┬─────────────┘
                              │ loads
                 ┌────────────▼─────────────┐
                 │  Instructions + Skills   │
                 │  + Commands (markdown)   │
                 └────────────┬─────────────┘
                              │ tool calls
          ┌───────────────────┼───────────────────┐
          ▼                   ▼                   ▼
   Host builtins        Deterministic CLIs    Sub-agents
   Read/Web/Ask         (any language)        (Agent tool)
          │                   │
          └─────────┬─────────┘
                    ▼
            Durable files / user chat
```

**Bottom line:** treat the coding IDE/CLI as your LLM platform SDK. Your repository becomes the product configuration for that platform — not a client of a model HTTP API.
