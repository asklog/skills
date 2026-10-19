---
name: asklog-layout
description: Organise asklog's tabs and panes from the CLI - rename, reorder, close and focus tabs, move terminals between tabs or beside each other, group terminals (such as Chief's Workers) into one tab, close, rename and resize panes, and read or replace a whole arrangement. Load before rearranging anything in the asklog window; never do it with computer use.
---

# Organising tabs and panes

A **workspace** has **tabs**; a tab is a tree of **splits** whose leaves are **panes**; a pane shows one **session** (a shell or an agent), a file, or nothing. The arrangement belongs to the machine, so every asklog window, the phone and the browser see the same one, and a change you make here shows up everywhere at once.

**Arrange the window only with these commands.** Never use computer use, screenshots, clicks, keystrokes or AppleScript on the asklog app to move, split, close or resize anything. If a command is refused, report the refusal and what the person can do; do not fall back to driving the app.

Every command takes exact ids from `asklog layout list --json` (or `asklog layout info <id> --json`) and prints JSON with `--json`. Needs an asklog version that has these verbs (`asklog layout --help` lists them); an older machine answers that it cannot edit layouts yet.

## Read

```sh
asklog layout list --json                       # every workspace, its tabs and panes
asklog layout list --workspace myproj --json    # one workspace
asklog layout info <tab-or-pane-id> --json      # one tab or pane, with its session
```

Tab and pane ids are UUIDs; session ids are short hex. In `list`, each workspace has `tabs: [{id, title, active, panes: [{id, tabId, content, title, focused, session}]}]`.

## Tabs

| Do | Command |
|---|---|
| Name a tab | `asklog layout rename-tab <tab-id> Review` (`--clear` goes back to the derived name) |
| Reorder | `asklog layout move-tab <tab-id> 0` (0 is first) |
| Bring to front / focus a pane | `asklog layout focus <tab-or-pane-id>` |
| Close | `asklog layout close-tab <tab-id>` |
| Close and end its terminals | `asklog layout close-tab <tab-id> --kill` |
| New tab with something running | `asklog layout new-tab --workspace myproj --agent codex` or `--command 'npm run dev'` |

Closing a tab or pane **hides** its terminals: they keep running and are listed under "Not in a tab". `--kill` ends the processes too; only use it for terminals you started, or on the person's word.

## Panes

| Do | Command |
|---|---|
| Start something beside a pane | `asklog layout split <pane-id> right\|down\|left\|up --command '...'` |
| Move a terminal into another tab | `asklog layout move <pane-id> --to-tab <tab-id>` |
| Move it beside another pane | `asklog layout move <pane-id> --beside <pane-id> --side down` |
| Give it a tab of its own | `asklog layout move <pane-id> --new-tab --title Logs` |
| Put back a terminal that is in no tab | `asklog layout move --session <session-id> --new-tab` (or `--to-tab`, `--beside`) |
| Gather terminals into one tab | `asklog layout group --workspace myproj <session-id> <session-id> --title Build` |
| Gather Chief's Workers | `asklog layout group --workspace myproj --workers` (tab "Workers") |
| Close a pane | `asklog layout close-pane <pane-id>` (`--kill` also ends its terminal) |
| Name a terminal | `asklog layout rename-pane <pane-id> api` (`--clear` to drop the name) |
| Resize | `asklog layout resize <pane-id> 60%` (its share of the split it is in, 5%-95%) |
| Equal sizes | `asklog layout equalize <tab-id>` |

`group` arranges the terminals as a grid in one tab: the first tab with that name, else a tab already holding one of them, else a new one. Moving a terminal out of a tab that then holds nothing removes that tab.

## The whole arrangement

```sh
asklog layout get --workspace myproj --json > layout.json     # {workspaceId, revision, screen}
# edit layout.json (tab order, titles, the split tree)
asklog layout apply --workspace myproj --file layout.json     # exit 1 if it changed since
```

`apply` replaces the arrangement only if nothing changed since the revision you read; otherwise read it again and reapply. Prefer the small commands above; use `apply` for a large reorganisation.

## Chief's Workers

When Chief starts Workers they all open in one "Workers" tab of their workspace, as a grid, so the person sees the whole team at once (on the canvas too). Chief's setting `workers.layout` chooses `shared-tab` (default) or `own-tab`. To tidy Workers that ended up in separate tabs, run `group --workers`.

## From Chief's or a Lead's folder

Add `--capability "$ASKLOG_CHIEF_CAPABILITY"` to every `asklog layout` command, or call the `asklog_chief` tool with `op=layout` and `request:{action, ...}`: `action` is the command name (`new-tab` is `open-tab`), the ids are `tabId`, `paneId`, `targetId`, `sessionId`, `toTabId`, `besidePaneId`, and the rest are `title`, `side`, `index`, `share`, `kill`, `newTab`, `workers`, `sessionIds`, `expectedRevision`, `screen`. A Lead may arrange only its own workspace.
