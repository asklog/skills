---
name: asklog-session-info
description: What asklog can tell you about sessions, agents, workspaces, tabs, panes, accounts and Chief roles, which command returns each field, and jq one-liners to query them. Load when you need to know what is running, where, in which state, or who started it.
---

# What asklog knows, and where to read it

All of these are read-only and safe to run at any time.

## Sessions: `asklog ls --json`

An array; one object per session.

| Field | What it says |
|---|---|
| `id` | exact session id; the handle for `read`, `send`, `kill` |
| `title` | the tab title (not unique) |
| `cwd` | the folder the session started in |
| `workspaceId` | the workspace it belongs to, or `null` |
| `pid` | the session's own shell process |
| `alive` | `false` once its process has exited |
| `origin` | `live`, or `restored` after a daemon crash (scrollback recovered, new shell) |
| `agentStatus`, `statusSeq`, `statusChangedAt` | the agent's state (`working`, `blocked`, `done`, `idle`, `unknown`), a change counter, and when it last changed |
| `detectedAgent` | the agent CLI the daemon sees in the foreground right now (for example `claude`, `codex`), or `null` |
| `agent` | how it was launched with `--agent`: `profile`, and `resume` (`available` means `asklog resume` can bring it back) |
| `agentMode` | `normal` or `yolo` for a session started with `--agent`; `null` for a plain shell |
| `agentAccount` | the stored account it runs on (`provider`, `id`, `label`) |
| `chat` | present when the app can show the session as a chat |
| `chiefBinding` | present on Chief, Lead and Worker sessions: `role` is `worker` for a Worker and `chief` for Chief and Leads (a Lead has a non-null `parentDispatchId`), plus `taskId`, `dispatchId`, `generation` |
| `cols`, `rows`, `createdAt` | size and age |

```sh
asklog ls --json | jq -r '.[] | select(.agentStatus == "blocked") | "\(.id) \(.title) \(.cwd)"'   # who needs an answer
asklog ls --json | jq -r '.[] | select(.detectedAgent != null) | "\(.id) \(.detectedAgent) \(.agentStatus)"'
asklog ls --json | jq -r --arg d "$PWD" '.[] | select(.cwd == $d) | .id'                             # sessions in this folder
asklog ls --json | jq -r '.[] | select(.chiefBinding.role == "worker") | "\(.id) \(.chiefBinding.taskId)"'
```

## Workspaces: `asklog workspaces --json`

`id`, `name`, `folderPath`, `createdAt`, `lastActivityAt`, `archivedAt`. Commands that take `--workspace` accept the exact name or the id; prefer the id when names repeat.

## Tabs and panes: `asklog layout list --json`

`{workspaces: [{id, name, snapshot: {screen: {tabs, activeTabId, ...}}}]}`. Each tab has `layout.root`, a tree of `{kind: "split", direction, children, ratios}` and `{kind: "pane", id, content}`, plus `layout.focused` (the focused pane id). A pane's `content` is `{kind: "session", session: "<id>"}` for a terminal; other kinds are non-terminal panes, and `null` is an empty pane.

```sh
# Every pane with its session, per workspace:
asklog layout list --json | jq -r '.workspaces[] | .name as $w | .. | objects | select(.kind? == "pane") | "\($w) \(.id) \(.content.session? // "-")"'
# One workspace only:
asklog layout list --workspace <name-or-id> --json
```

`asklog layout info <tab-or-pane-id> --json` returns one pane (or tab) with its `tabId`, `focused`, and the full session object (`null` once `asklog ls` no longer lists that session), so it is the quickest way from a pane id to everything about what runs in it.

## Accounts and machine

- `asklog agents list --json` - stored Claude and Codex accounts, which one new sessions start on, and how much of each rate limit is used.
- `asklog whoami --json` - which asklog account this machine is signed in to.
- `asklog reach --json` - whether the machine is reachable from outside its own network.
- `asklog devices --json` - devices that can reach this machine.

These name accounts and devices, and `whoami --json` carries the account email and the host's public key: read the fields you need and never paste the raw output anywhere.

## Chief

- `asklog chief config --json` - every Chief setting, its value, the layer it comes from and who set it. `asklog chief config keys` explains each key.
- Chief, Lead and Worker sessions show up in `asklog ls` like any other; Worker sessions carry `chiefBinding` and run in `<repo>/.asklog-worktrees/<task-id>/` by default.

## Screen contents

`asklog read <id> --lines n` is the only way to see output. There is no history API beyond the scrollback that `read` returns.
