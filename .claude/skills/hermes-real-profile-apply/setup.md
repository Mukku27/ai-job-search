# One-time: enable Hermes Real-Profile Browsing (Hermes Agent only)

Goal: Hermes drives a **snapshot of Mukesh's real Chromium profile** (LinkedIn / Google / ATS cookies) via its packaged Chromium — no Playwright extension needed.

> Other agents (Cursor, Claude Code, Codex, OpenCode, Kilo, Pi, Prime) do **not** use this. They stay on Playwright MCP `--extension` via `.claude/skills/playwright-chrome-apply/setup.md`.

## 1. Requirements

- **Hermes Agent v0.20.6+** (commits `1f4d095fd8`, `830e4a29be` — Aug 27 2026). Check: `hermes --version`
- **Chromium-family default browser:** Chrome, Edge, Brave, or Chromium. Firefox as OS default fails closed.
- Only the **active/default profile** is snapshotted — other Chrome profiles are ignored.

## 2. Enable Real-Profile Browsing

### Option A — Config file (all surfaces)

Add to `~/.hermes/config.yaml`:

```yaml
browser:
  use_real_profile: true
```

Optional — let Hermes *offer* to close a locked browser after your approval (never auto-closes):

```yaml
browser:
  use_real_profile: true
  real_profile_autoclose: true
```

Restart Hermes after editing.

### Option B — Desktop app

**Settings → Browser → Use My Real Browser Profile** (toggle on).

## 3. How it syncs

| When | What happens |
|---|---|
| First browser session after enabling | Hermes reads `Local State → profile.last_used`, copies cookies/logins/preferences to `~/.hermes/browser-profile/<browser>/`, launches packaged Chromium on the snapshot |
| Every new session thereafter | Cookies and logins are **copied again** — logins you do in your live Chrome show up in the next agent session |
| You toggle it off | Hermes **deletes** the whole snapshot store on next browser use — credentials do not linger |

Snapshot path is a non-default directory — this is what keeps it legal under Chrome 136+ default-profile debugging block and avoids profile-lock fights.

## 4. Verify

```bash
# Config is on
cat ~/.hermes/config.yaml | grep -A2 browser:

# Snapshot exists after first browser use
ls -la ~/.hermes/browser-profile/

# In Hermes, browser_exec now exposes the `local` argument (schema-gated — only when consent is on)
# Ask the agent: "list browser tools" — you should see browser_exec with local: true
```

Open a Hermes session and ask it to navigate to `https://linkedin.com` or `https://greenhouse.io` — you should be logged in as Mukesh without typing credentials.

## 5. Windows caveat

Windows holds cookie/login DBs with an exclusive (deny-all) lock while Chrome/Edge/Brave is running — Hermes **cannot copy while the browser is open**. It fails fast with "fully quit the browser and retry".

- Quit Chrome fully, including tray/background instances: `chrome://settings/system` → turn off "Continue running background apps when Google Chrome is closed".
- With `real_profile_autoclose: true`, Hermes stops and **asks you** first; only on your `yes` does it run `hermes browser close-profile` (kills the profile's process tree — unsaved tabs lost), then retries. If still locked after that, it stays blocked — no loop.
- **macOS and Linux:** copying works while the browser is running — no quit needed.

## 6. Security framing

> This is **consent-gated convenience, not an isolation boundary.** A page the agent visits runs with your real logins. Enable it when you want the agent acting as you; leave it off otherwise. Off by default for exactly this reason.

Source: [Browse as You: Real-Profile Browsing in Hermes Agent](https://hermes-agent-lab.com/blog/hermes-real-profile-browsing-guide/) and v0.20.6 release notes.

## 7. Confirm inside Hermes

Ask the agent:

```
Are we on Real-Profile Browsing? Show browser.use_real_profile and snapshot path.
```

You should see `use_real_profile: true` and `~/.hermes/browser-profile/<browser>/`.

## 8. Fallback

Do **not** install Playwright MCP `--extension` inside Hermes — the two bridges contend for the profile. If Real-Profile Browsing is unavailable (old Hermes, Firefox default, locked profile you cannot quit), stop and fix the gate above. Never invent an empty browser and pretend it is logged in.
