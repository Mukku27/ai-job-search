---
framework_version: 1.0.0
---

# Agent Guidelines: AI Job Search

This workspace is structured to manage job search activities, scraper tools, CVs, cover letters, and interview preparation.

## Thin-Pointer Design (Single Source of Truth)

To prevent duplication and configuration drift across different AI agent frameworks (Claude Code, Google Antigravity, Codex, Cursor, Gemini CLI, etc.), this workspace uses a unified thin-pointer design. All agent runtimes should load the canonical specifications and candidate profiles from the files and directories below:

1. **Personal Candidate Profile:**
   - The candidate profile, contact details, education, and target preferences are defined in [CLAUDE.md](CLAUDE.md) and the individual profile methodology files under [.claude/skills/job-application-assistant/](.claude/skills/job-application-assistant/) (specifically `01-*.md` etc.).
2. **Canonical Workflow Specifications:**
   - The step-by-step instructions and triggers for tasks (setup, scrape, rank, apply, upskill, interview) are defined in the [.claude/](.claude/) directory (specifically under `.claude/skills/` and `.claude/commands/`).
   - Do not duplicate these rules or specifications. Treat `.claude/` files as the single source of truth.
3. **Portal Search Skills:**
   - Job-portal search CLIs live under [.agents/skills/](.agents/skills/) in the portable Agent Skills format (with a `SKILL.md` per portal). Codex and Antigravity discover these automatically; the `/scrape` workflow in [.claude/skills/job-scraper/](.claude/skills/job-scraper/) orchestrates them.
4. **Human prose (BeHuman):**
   - Natural-sounding writing for hiring answers, cover letters, and interview prep lives in [.claude/skills/behuman/](.claude/skills/behuman/). Apply it whenever drafting application Q&A or other candidate-facing prose.
5. **Browser submit (real Chrome — two isolated paths):**
   - **Hermes Agent only — Real-Profile Browsing (v0.20.6+):** `/submit` fills and submits already-drafted kits via Hermes' managed snapshot of Mukesh's default Chromium profile (`~/.hermes/browser-profile/`) driven by packaged Chromium through `browser_exec(local=true)`. Canonical files: [.claude/commands/submit.md](.claude/commands/submit.md) (Hermes branch) and [.claude/skills/hermes-real-profile-apply/](.claude/skills/hermes-real-profile-apply/) (setup: `browser.use_real_profile: true` in `~/.hermes/config.yaml` or Desktop Settings → Browser). Sequential prompt: [prompts/hermes-auto-apply-prompt.md](prompts/hermes-auto-apply-prompt.md). Never use Playwright MCP `--extension` inside Hermes.
   - **All other harnesses (Cursor, Claude Code, Codex, OpenCode, Kilo, Pi, Prime Agent) — Playwright MCP `--extension`:** same `/submit` queue but via `playwright-chrome` MCP (`npx @playwright/mcp@latest --extension`) attached to the live Chrome tab. Canonical files: [.claude/skills/playwright-chrome-apply/](.claude/skills/playwright-chrome-apply/) and MCP config [.cursor/mcp.json](.cursor/mcp.json); OpenCode: [opencode.jsonc](opencode.jsonc) and `~/.config/opencode/opencode.jsonc`; Kilo: `.kilo/kilo.jsonc`; Pi: `.pi/mcp.json`; Prime: `~/.prime/agent/settings.json`. Sequential prompt: [prompts/opencode-playwright-mcp-apply-prompt.md](prompts/opencode-playwright-mcp-apply-prompt.md).
   - Do not use browser-use MCP, a cloud browser, or an empty Playwright profile for LinkedIn/ATS sessions. Run **one harness at a time** — Hermes Real-Profile and Playwright extension must not contend for the same profile.
