---
name: asklog-drive-agent
description: Hand a prompt to another AI agent running in an asklog session (Claude Code, Codex and others), wait until its turn ends, and read its answer, using agentStatus and statusSeq instead of guessing from the screen. Load before typing into another agent's session.
---

# Driving another agent through asklog

Use this when you start or are pointed at another agent's session and need to give it work. You need its exact session id (`asklog-panes` shows how to start one beside you and get it).

## Status words

`asklog ls --json` reports each session's `agentStatus`:

| Value | Meaning |
|---|---|
| `working` | mid-turn |
| `blocked` | stopped and waiting for an answer (a permission prompt or a question) |
| `done` | reached idle while nobody was watching, and nobody has looked since |
| `idle` | between turns |
| `unknown` | no agent is running, or nothing can tell (every plain shell) |

`statusSeq` goes up on every status change. Waiting for `statusSeq` to move **and** the status to leave `working` is what makes a wait reliable: right after you press enter the status can still read `idle` from before your prompt.

## Send a prompt and wait

```sh
sid=3f9a1c2e   # the exact session id
state() { asklog ls --json | jq -r --arg s "$sid" '.[] | select(.id == $s) | "\(.statusSeq) \(.agentStatus)"'; }

read seq0 _ <<<"$(state)"
asklog send "$sid" --key ctrl+u                      # clear anything half-typed
asklog send "$sid" --text 'Run the tests and summarise any failures.'
asklog send "$sid" --key enter

waited=0
while :; do
  read seq now <<<"$(state)"
  [ "$seq" -gt "$seq0" ] && [ "$now" != working ] && break
  sleep 3; waited=$((waited + 3))
  [ "$waited" -ge 1800 ] && { echo "still $now after ${waited}s"; break; }
done
asklog read "$sid" --lines 120
```

Do not name a shell variable `status`: zsh reserves it.

Read generously. Agent TUIs pad blank rows between the last reply and their input box and footer, so `--lines 10` can return only the footer. Ask for a marker line and match it: `asklog read "$sid" --lines 200 | grep 'DONE:' | tail -1` (the last match, since your own prompt is on the screen too).

## Before you type

- `asklog read "$sid" --lines 60` first. If the agent is `working`, wait; if a person is typing in it, do not type over them.
- If it shows a dialog (trust this folder, sign in, choose a model), `--text` will not answer it: the paste is inert to menus and there are no arrow keys. Use `--key enter` or `--key escape` only when you are sure what they select, otherwise ask the person to answer it in the app.
- Multi-line prompts are fine in one `--text`; newlines inside the paste do not submit.

## After the turn

- `blocked`: read the screen. It is usually a permission prompt or a question for the person. Answer only what you are entitled to answer; otherwise surface it.
- `done` or `idle`: read the answer with `asklog read`. `done` turns back to `idle` once someone looks at the session, so either one means the turn ended.
- To stop a runaway turn: `asklog send "$sid" --key escape` (Claude Code, Codex) or `--key ctrl+c`.

## Etiquette

- Give the other agent a complete, self-contained task; it cannot see your context.
- Ask for a recognisable final line ("end with DONE: <summary>") when you need to parse the reply.
- Kill only agents you started (`asklog kill "$sid"`), and only after you have read what you need.
