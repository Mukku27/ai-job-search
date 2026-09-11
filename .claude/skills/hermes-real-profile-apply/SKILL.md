---
name: hermes-real-profile-apply
description: >
  Submits already-drafted job applications in Hermes Agent via Real-Profile Browsing
  (v0.20.6+) — Hermes copies your default Chromium profile into ~/.hermes/browser-profile/
  and drives it with packaged Chromium via browser_exec local. Use ONLY inside Hermes Agent
  when the user says submit, apply online, fill this form, upload CV, Greenhouse, Ashby,
  Lever, Workday, Keka, Freshteam, /submit, or "run the apply queue" from Hermes.
  Never use Playwright MCP --extension inside Hermes; that path is for Cursor/Codex/OpenCode/Kilo/Pi/Prime.
---

# Hermes Real-Profile Apply

This skill **submits** kits that `/apply` already drafted — **Hermes Agent only**.
It uses **Real-Profile Browsing** (Hermes v0.20.6+, Aug 27 2026), not Playwright MCP.

Canonical loop: `.claude/commands/submit.md` (Hermes branch) → `prompts/hermes-auto-apply-prompt.md` (from `## THE PROMPT`).

Other harnesses (Cursor, Claude Code, Codex, OpenCode, Kilo, Pi, Prime Agent) continue to use
`.claude/skills/playwright-chrome-apply/` via Playwright MCP `--extension`. Do not mix the two.

## How Real-Profile Browsing works

1. Hermes reads your OS default Chromium browser's `Local State → profile.last_used` (Chrome/Edge/Brave/Chromium).
2. Copies that **active profile only** (cookies, logins, preferences) into a managed snapshot at `~/.hermes/browser-profile/<browser>/`.
3. Launches its **packaged Chromium** on that snapshot and hands the CDP endpoint to `browser_exec` / Browser Use CLI.
4. On every new Hermes browser session, auth is **re-synced** from your real profile — logins you did in your live Chrome appear in the next agent session.

**Snapshot, not live profile:** Hermes never opens your live profile directory directly. This avoids profile-lock fights, sidesteps Chrome 136+'s default-profile debugging block, and makes consent revocable (toggle off → snapshot deleted on next browser use). See [Real-Profile Browsing guide](https://hermes-agent-lab.com/blog/hermes-real-profile-browsing-guide/).

## Setup (one-time, Hermes only)

Setup steps live in [setup.md](setup.md). In short:

```yaml
# ~/.hermes/config.yaml
browser:
  use_real_profile: true          # off by default, consent-gated
  # real_profile_autoclose: true  # optional — agent offers to close locked browser after your approval
```

Desktop: **Settings → Browser → Use My Real Browser Profile**.

After enabling, `browser_exec` gains a schema-gated `local` argument that forces a real-profile local session even under a cloud backend. The model cannot use `local` unless you have consented.

## Pre-flight gate (every Hermes session, before the first form)

1. Confirm Hermes version is **v0.20.6+** (`hermes --version` or `/version`). If older, stop and tell Mukesh to upgrade.
2. Confirm `browser.use_real_profile: true` is set (`cat ~/.hermes/config.yaml` or Settings → Browser toggle).
3. Confirm the active tool is **`browser_exec` with `local: true`** (or Browser Use CLI on the snapshot), not `browser_navigate` from Playwright MCP and not `computer_use`.
4. Confirm snapshot exists: `ls ~/.hermes/browser-profile/` should show `<browser>/` after first sync.
5. If on **Windows** and the browser is open, Hermes will fail fast with "fully quit the browser and retry" — this is expected (exclusive file lock on cookie DB). Quit Chrome fully (including tray/background apps at `chrome://settings/system` → "Continue running background apps") and retry. With `real_profile_autoclose: true`, Hermes will *offer* to run `hermes browser close-profile` after your explicit approval — never auto-closes.
6. If the only browser available is `Firefox` (non-Chromium default), it fails closed with a clear message — switch OS default to Chrome/Edge/Brave.

If any gate fails, **stop**. Tell Mukesh to finish [setup.md](setup.md). Never fall back to an empty Playwright profile or a cloud browser for LinkedIn/ATS and pretend it is logged in.

## What to do after the gate

- Read and execute `prompts/hermes-auto-apply-prompt.md` from `## THE PROMPT` to the end.
- Use `browser_exec(local=true)` for every navigation, form fill, and file upload. One live application form at a time.
- For file uploads, pass absolute paths under the workspace (e.g. `/Users/mukku27/Desktop/ai-job-search/cv/main_*.pdf`).
- On login/CAPTCHA/OTP/2FA/“verify you are human”: stop, tell Mukesh what to do in his **live Chrome** (the source of the snapshot), wait for `CONTINUE`, then start a **new browser session** so the snapshot re-syncs the fresh auth.

## Scope

- **This skill:** Hermes Agent + Real-Profile Browsing + `browser_exec local`
- **Not this skill:** Cursor/Codex/OpenCode/Kilo/Pi/Prime → use `playwright-chrome-apply` + Playwright MCP `--extension`
- Never run both harnesses against the same snapshot at the same time.

## What to say later

- `/submit` inside Hermes — start Wave A from the to-do via Real-Profile Browsing
- `/submit Dscout` — that company next
- "Submit the queue" / "fill this Greenhouse form" — same skill (Hermes branch)
