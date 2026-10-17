---
name: asklog-cli
description: Core asklog CLI commands for agents - list sessions, read a session's screen, type and press keys in it, start and kill sessions, workspaces and layout, JSON output and common pitfalls. Load before running any `asklog` command.
---

# asklog CLI

asklog runs terminal sessions and AI coding agents in panes on a machine, served by a local daemon and reachable from the desktop app, phone and browser. The `asklog` command talks to that daemon (default `127.0.0.1:7481`, override with `--host`/`--port`). Run `asklog --help` for the full, version-matched reference; this skill is the part an agent needs.

## Core commands

| Goal | Command |
|---|---|
| List sessions | `asklog ls --json` |
| List workspaces (ids, names, folders) | `asklog workspaces --json` |
| Read a session's screen | `asklog read <session> --lines 40` |
| Type text (not submitted) | `asklog send <session> --text '...'` |
| Press a key | `asklog send <session> --key enter` (also `escape`, `ctrl+c`, `ctrl+u`) |
| Start a shell session | `asklog new --workspace <name-or-id> --cwd <dir> --title <name> --json` |
| Start an agent session | `asklog new --agent claude-code --workspace <ws> --json` |
| Kill a session | `asklog kill <session>` |
| Tabs and panes | `asklog layout list --json`, `asklog layout info <pane-id> --json` |
| Split a pane and start something in it | `asklog layout split <pane-id> right\|down --command '...' --json` |
| Put an agent back on its conversation after a crash | `asklog resume <session>` |
| Stored Claude/Codex accounts and rate-limit use | `asklog agents list --json` |
| Chief, asklog's orchestrator | `asklog chief`, `asklog chief config` |

Agent profiles for `--agent`: `claude-code`, `codex`, `grok`, `cursor`, `opencode`, `gemini`, `pi`, `agy`. `--yolo` starts the agent with no permission prompts; `--normal` (the default) keeps them.

`asklog attach <session>` is interactive (detach with Ctrl-]); an agent should use `read` and `send` instead.

## JSON output

Pass `--json` and parse it (for example with `jq`) instead of scraping tables.

- `asklog ls --json` is an array of sessions: `id`, `title`, `cwd`, `workspaceId`, `pid`, `alive`, `origin`, `agentStatus`, `statusSeq`, `detectedAgent`, `agent`, and more. See the `asklog-session-info` skill.
- `asklog new --json` prints the new session, so take its `id` from there.
- `asklog layout ...` always prints JSON; `--json` makes it one line. `split` and `new-tab` answer `{ok, sessionId, paneId, snapshot}`.

```sh
id=$(asklog new --workspace myproj --title build --json | jq -r .id)
asklog layout split "$pane" right --command 'npm test' --json | jq '{sessionId, paneId}'
```

## Pitfalls

- **Session ids are exact.** They are short hex strings like `3f9a1c2e`. No prefix matching, no title matching. Get the id from `asklog ls --json`, `asklog new --json` or a layout answer and keep it. Never pick a session to kill by grepping titles: titles repeat (`zsh`, `Claude Code`) and other people's sessions share the machine.
- **`--text` never submits.** Follow it with `asklog send <id> --key enter`. The text travels as one bracketed paste: newlines and tabs are fine, but control characters (including `\r` and escape) are refused with exit code 2.
- **Only four keys exist:** `enter`, `escape`, `ctrl+c`, `ctrl+u`. There are no arrow keys, so an arrow-key menu (for example an agent's "do you trust this folder?" dialog) cannot be answered through `send`. Start agents in folders the agent already trusts, or ask the person to answer in the app.
- **`read` shows the screen, not a log.** It returns scrollback plus the visible screen. Trailing blank rows are dropped, then `--lines n` keeps the last n rows, blank or not, so the blank rows an agent TUI pads between its reply and its input box count against n. Use `--format ansi` only when you need colours.
- **A session with no `--workspace` belongs to no workspace** (`workspaceId: null`) and lands in the "Not in a workspace" group instead of the workspace's own tabs. Pass `--workspace`, or use `asklog layout split` / `new-tab`, when the person should see it next to their project.
- **`restored`** in `asklog ls` means the daemon died and came back: the scrollback was recovered but the shell is new. `asklog resume <id>` reattaches a Claude or Codex agent to its conversation.
- **`send` to an exited session fails** on purpose; `read` still works while the daemon lists it.
- **Killing a session closes its pane.** Kill only sessions you started, by the id you recorded.
- The command was called `tl` before; that alias still works but prints a notice.
