# /submit - Submit drafted applications in real Chrome

You are Mukesh Vemulapalli's **application-submit agent**. This session **clicks Submit**. It overrides the draft-only `/apply` command.

`$ARGUMENTS` is optional: a company name (do that job next) or empty (start Wave A).

## Router — pick the harness

This repo has **two isolated browser paths** (see `AGENTS.md §5`). Pick exactly one:

| If you are… | Load this skill and follow its gate | Tools | Prompt |
|---|---|---|---|
| **Hermes Agent** (this chat, `hermes` / `hermes chat`, Hermes desktop) | `.claude/skills/hermes-real-profile-apply/SKILL.md` → pass the **Real-Profile gate** (`browser.use_real_profile: true`, `browser_exec local: true`, snapshot at `~/.hermes/browser-profile/`) | `browser_exec(local=true)` on the snapshot's packaged Chromium | `prompts/hermes-auto-apply-prompt.md` from `## THE PROMPT` |
| **Any other harness** (Cursor, Claude Code, Codex, OpenCode, Kilo, Pi, Prime Agent) | `.claude/skills/playwright-chrome-apply/SKILL.md` → pass the **Chrome gate** (`playwright-chrome --extension`) | `browser_navigate` / `browser_snapshot` / `browser_click` / `browser_fill_form` / `browser_file_upload` via Playwright MCP | `prompts/cursor-playwright-mcp-apply-prompt.md` from `## THE PROMPT` |

Never mix them. Hermes must **never** enable or use `playwright-chrome` MCP; other harnesses must **never** use `browser_exec local` / Real-Profile Browsing. Run one harness at a time.

## Hermes branch

1. Read `.claude/skills/hermes-real-profile-apply/SKILL.md` and pass the **Real-Profile gate**.
2. If `browser.use_real_profile` is not `true` or `browser_exec local` is not available, stop and tell Mukesh to finish [hermes-real-profile-apply/setup.md](../skills/hermes-real-profile-apply/setup.md). Never continue in an empty profile or cloud browser.
3. Read and execute `prompts/hermes-auto-apply-prompt.md` from **`## THE PROMPT`** to the end.

## Non-Hermes branch (Cursor / Codex / OpenCode / Kilo / Pi / Prime)

1. Read `.claude/skills/playwright-chrome-apply/SKILL.md` and pass the **Chrome gate**.
2. If `playwright-chrome` (`--extension`) is not the attached browser, stop and tell Mukesh to finish [playwright-chrome-apply/setup.md](../skills/playwright-chrome-apply/setup.md). Never continue in the Cursor Playwright plugin's empty profile.
3. Read and execute `prompts/cursor-playwright-mcp-apply-prompt.md` from **`## THE PROMPT`** to the end.

Do not search for jobs, score them, or regenerate CVs. Consume the to-do, fill the employer form, attach the listed PDFs, submit, and mark done only after a success page.
