> **Not for Hermes Agent.** Hermes uses Real-Profile Browsing — see `.claude/skills/hermes-real-profile-apply/setup.md`. This file is for Cursor / Claude Code / Codex / OpenCode / Kilo / Pi / Prime only.

# One-time: attach Playwright MCP to real Chrome

Goal: Cursor drives the Chrome profile Mukesh already uses (LinkedIn / Google / ATS cookies).

## 1. Install the Chrome extension

Chrome Web Store: [Playwright Extension](https://chromewebstore.google.com/detail/playwright-extension/mmlmfjhmonkocbjadbfplnigmagldckm) (`mmlmfjhmonkocbjadbfplnigmagldckm`).

Pin it. Leave Chrome open on the profile that is logged into LinkedIn / Google.

## 2. Enable the MCP server

This repo ships:

```json
{
  "mcpServers": {
    "playwright-chrome": {
      "command": "npx",
      "args": ["-y", "@playwright/mcp@latest", "--extension"]
    }
  }
}
```

- Cursor: `.cursor/mcp.json` (also copied into `~/.cursor/mcp.json`)
- Claude Code / Codex: `.mcp.json`
- OpenCode and OpenCode 2: `~/.config/opencode/opencode.jsonc` and this repo’s `opencode.jsonc` (`playwright-chrome`, `--extension`). Restart after editing.
- Kilo Code: `~/.config/kilo/kilo.jsonc` (and `~/.config/kilo/opencode.jsonc`) plus this repo’s `.kilo/kilo.jsonc`
- Pi Agent: `~/.pi/agent/mcp.json` plus this repo’s `.pi/mcp.json`. If `/mcp` shows no servers, run `pi install npm:pi-mcp-extension` once
- Prime Agent: `~/.prime/agent/settings.json` → `mcpServers.playwright-chrome` (stdio). Project `.prime` MCP entries are ignored; this must stay in the user settings file. In-session: `await mcp.call_tool("playwright-chrome", "<tool>", args)`

Restart the harness after editing. Optional: `PLAYWRIGHT_MCP_EXTENSION_TOKEN` in that server’s environment (do not commit the token). Run only one of these harnesses against Chrome at a time.

In Cursor: **Settings → MCP → enable `playwright-chrome`**. Reload the window if the toggle is missing. Approve the first connection in Chrome (extension popup or tab picker).

Optional: copy `PLAYWRIGHT_MCP_EXTENSION_TOKEN` from the extension status page into the server `env` to skip the approval dialog. Do not invent a token.

## 3. Fallback if `--extension` is unavailable

```json
"args": ["-y", "@playwright/mcp@latest", "--channel", "chrome"]
```

If Chrome is already open and this fails with a profile lock, close extra Chrome instances **or** log in once in the MCP window. Do not start a second anonymous browser.

## 4. Confirm

Ask the agent to list tabs. You should see Mukesh's existing Chrome tabs, not an empty Playwright profile.
