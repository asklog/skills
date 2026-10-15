---
name: asklog-panes
description: Work with asklog panes - find your own session and pane, split a pane, start a shell command or another agent beside you, read another pane's output, send it text, wait for it, and close it safely. Load whenever you are asked to open, run or check something "in a pane", "beside", "on the right" or "below".
---

# asklog panes

In asklog a **workspace** has **tabs**, a tab is a tree of **splits**, and each leaf is a **pane** that usually shows one **session** (a shell or an agent). Pane ids are UUIDs; session ids are short hex strings. Everything below uses exact ids. See `asklog-cli` for the base commands.

## 1. Find your own session and pane

asklog sets no "current session" variable. Match your process ancestry against session pids:

```sh
pids=" "; p=$$
while [ "$p" -gt 1 ]; do pids="$pids$p "; p=$(ps -o ppid= -p "$p" | tr -d ' '); done
self=$(asklog ls --json | jq -r --arg pids "$pids" '.[] | select(.pid as $x | $pids | contains(" \($x) ")) | .id')

# Your workspace id and pane id:
asklog layout list --json | jq -r --arg s "$self" \
  '.workspaces[] | .id as $w | .. | objects | select(.kind? == "pane" and .content.session? == $s) | "\($w) \(.id)"'
```

If `self` is empty you are not running inside an asklog session; start what you need with `asklog new --workspace <ws> --json` instead of splitting.

## 2. Open something beside you

```sh
# A shell command in a new pane to the right (or `down`):
asklog layout split "$pane" right --command 'npm run dev' --cwd "$PWD" --title dev --json | jq '{sessionId, paneId}'

# Another agent below you:
asklog layout split "$pane" down --agent codex --cwd "$PWD" --json | jq '{sessionId, paneId}'

# A new tab in a workspace:
asklog layout new-tab --workspace <name-or-id> --command 'htop' --json | jq '{sessionId, paneId}'
```

- Record the returned `sessionId`; it is the only handle you use from now on.
- `--command` is typed into a fresh shell in that pane, so the shell stays after the command ends.
- `--agent` and `--command` are exclusive. `--cwd` defaults to the workspace folder.
- Pass `--request-id <uuid>` and reuse it on a retry: a repeated request returns the same pane instead of making a second one.
- `asklog new --workspace <ws> --place right --beside <session>` also works, but placement is best effort (only a window that is open and showing that workspace honours it). `layout split` is exact.
- Inside a Chief or Lead folder, add `--capability "$ASKLOG_CHIEF_CAPABILITY"` to `asklog layout` commands (see `asklog-chief`).

## 3. Read another pane

```sh
asklog read "$sid" --lines 40
```

To find a session that is already in a pane, read the pane's `content.session` from `asklog layout info <pane-id> --json`, or match on `cwd`/`title` in `asklog ls --json` and **confirm with the person** if more than one matches.

## 4. Send it text

```sh
asklog send "$sid" --text 'make test'
asklog send "$sid" --key enter
```

`--text` types without submitting; `--key enter` submits. Clear a half-typed line first with `--key ctrl+u`; interrupt with `--key ctrl+c`. To hand a task to an agent and wait for its answer, use the `asklog-drive-agent` skill.

## 5. Wait for it

- **An agent:** poll `agentStatus` and `statusSeq` (see `asklog-drive-agent`).
- **A plain shell:** `agentStatus` stays `unknown`, so wait for output. Append a sentinel to the command and poll `read`:

```sh
asklog send "$sid" --text 'make test; echo "__done $?"' && asklog send "$sid" --key enter
until asklog read "$sid" --lines 5 | grep -qE '^__done [0-9]+$'; do sleep 2; done
asklog read "$sid" --lines 60
```

Always bound a wait with a timeout and say what you saw if it expires.

## 6. Close it

```sh
asklog kill "$sid"
```

Killing a session closes its pane. Kill only sessions you created and recorded. Never kill by title, by `grep`, or by "the one that looks idle", and never kill the person's own sessions or your own.
