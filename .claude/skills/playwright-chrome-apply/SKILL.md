---
name: playwright-chrome-apply
description: >
  Submits already-drafted job applications in Mukesh's real Chrome via Playwright MCP
  --extension (logged-in LinkedIn, Google, Greenhouse, Ashby, Workday sessions).
  Use ONLY in non-Hermes harnesses (Cursor, Claude Code, Codex, OpenCode, Kilo, Pi,
  Prime Agent) when the user says submit, apply online, fill this form, upload the CV,
  Greenhouse, Ashby, Lever, Workday, Keka, Freshteam, /submit, or "run the apply queue".
  Hermes Agent must NOT use this skill — Hermes uses hermes-real-profile-apply instead.
  Never use a cloud browser or an empty Playwright profile for these sessions.
---

# Playwright Chrome apply — NON-HERMES HARNESSES ONLY

> **Scope:** Cursor, Claude Code, Codex, OpenCode / OpenCode 2, Kilo Code, Pi Agent, Prime Agent.
> **Hermes Agent must not use this skill.** Hermes uses [`hermes-real-profile-apply`](../hermes-real-profile-apply/SKILL.md) via Real-Profile Browsing (`browser_exec local`) instead — see `AGENTS.md §5`.

This skill **submits** kits that `/apply` already drafted. It does not search, score, or regenerate CVs.

Canonical loop: `.claude/commands/submit.md` (non-Hermes branch) → `prompts/cursor-playwright-mcp-apply-prompt.md` (from `## THE PROMPT`).

Chrome attach steps (one-time): [setup.md](setup.md).

If you are running inside **Hermes Agent**, stop and load `hermes-real-profile-apply` instead. Do not install or enable `playwright-chrome` MCP inside Hermes.

## Chrome gate (every session, before the first form)

1. Confirm these tools exist: `browser_navigate`, `browser_snapshot`, `browser_click`, `browser_fill_form`, `browser_file_upload`, `browser_select_option`.
2. Confirm the server is **`playwright-chrome`** (`npx @playwright/mcp@latest --extension`), not the Cursor Playwright plugin's isolated browser.
3. First navigation may open the extension tab-picker. Stop. Tell Mukesh to pick the Chrome tab (or approve the connection). Wait for `CONTINUE`.
4. If the only browser is a fresh/empty profile, **stop**. Ask Mukesh to enable `playwright-chrome` in Cursor Settings → MCP and install the Playwright Chrome extension. Never invent a second anonymous browser and pretend it is logged in.

Save Playwright snapshots with a `filename`, then inspect via context-mode. One live application form at a time.

## What to say later

- `/submit` — start Wave A from the to-do
- `/submit Dscout` — that company next
- "Submit the queue" / "fill this Greenhouse form" — same skill
