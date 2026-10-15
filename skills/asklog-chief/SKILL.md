---
name: asklog-chief
description: Work with or inside asklog's orchestrator - Chief, Leads and Workers. How to find and talk to Chief, start a Lead, read team status and settings from the CLI, and, when you are Chief, a Lead or a Worker, the asklog_chief tool calls and conventions (requests, dispatch, steering, reports, acknowledgements). Load when Chief, a Lead, a Worker, a task brief with taskId/dispatchId/generation, or the asklog_chief tool is involved.
---

# Chief, Leads and Workers

Chief is asklog's built-in orchestrator: the person says what they want, Chief turns it into tasks and starts a **Worker** for each in an isolated copy of the repository, watches it, checks what it reports against the forge, and brings back a reviewable pull request.

| Role | How many | Does | Talks to the person |
|---|---|---|---|
| Chief | one per machine | takes requests, dispatches and supervises Workers | yes |
| Lead | optional, per workspace, each with a focus (review, backend, ...) | supervises its workspace's Workers; reads the repo, never writes it | yes |
| Worker | one per task | does the work in `<repo>/.asklog-worktrees/<task-id>/` on branch `asklog/<task-id>`, pushes only that branch, opens a PR | never |

The daemon is the only writer of Chief's state. Agents act through the `asklog_chief` tool with a capability scoped to their role and task. Nothing is merged without the person's word (`merge.requirePersonWord` is locked on by default).

## From outside Chief (any agent or shell)

- **Find Chief's sessions:** `asklog ls --json | jq '.[] | select(.chiefBinding != null) | {id, title, role: .chiefBinding.role, task: .chiefBinding.taskId, agentStatus}'`. Workers have `role: "worker"`; Chief and Leads have `role: "chief"` (a Lead's `parentDispatchId` is set). Titles read `Chief`, `Lead · <name>`, `Worker · <task-id>`.
- **Watch a Worker:** `asklog read <id> --lines 60` and its `agentStatus`. Do not type into a Worker; steering goes through its supervisor.
- **Talk to Chief:** `asklog chief` attaches to its conversation (starting one if none is live). It is interactive: from a tool call, use `asklog read`/`asklog send` on Chief's session id instead (see `asklog-drive-agent`). Chief files a request from the person's own words in that conversation, so type only what the person asked you to relay, verbatim.
- **Start a Lead:** `asklog lead <workspace> [name]` attaches to that workspace's Lead and starts it if it is idle; with several Leads it lists the names. Leads are created in the app.
- **Settings:** `asklog chief config [--project <workspace>] [--json]` shows every key, its value and its source; `asklog chief config keys` explains them. `set`, `reset`, `lock`, `unlock` and `instructions` change them, and the CLI acts with the **person's** authority and is bound by no lock. Change settings only on the person's explicit word, and never `agent.mode.chief` or `agent.mode.workers` on your own.
- Chief is set up (switched on, given an agent account) in the asklog app, not from the CLI.

## As Chief or a Lead

You run in Chief's or a Lead's folder; its `CLAUDE.md`/`AGENTS.md` is your charter and outranks other instruction files. The folder also carries built-in skills (`asklog-settings`, `asklog-layout`, `asklog-landing`, and in Chief's folder on an account with more than one machine, `asklog-machines`): load those for config, layout, merge, cleanup and promote.

Every change takes a fresh UUID `requestId`; reuse it when you retry the same change.

| Op | Use |
|---|---|
| `op=briefing` | where things stand now; read it when unsure |
| `op=inbox`, then `op=acknowledge` with `deliveryId` | messages waiting for you; acknowledge each `[asklog delivery <uuid>]` |
| `op=request` with `quote`, `workspaceId` or `projectId`, `brief` | file what the person asked **in this conversation**: `quote` is at least 8 characters copied exactly from their message (asklog records their whole message); with `brief` the Worker starts at once. `kind=research` for a report instead of a PR; `lead:{name}` opens a Lead they asked for |
| `op=dispatch` with `taskId`, `brief` | start a queued task; optional `agent` (`claude`, `codex`, `grok`, `cursor`, `opencode`, `gemini`, `pi`, `agy`), `accountId`, `dependsOn`, `leadId` |
| `op=steer` with `taskId`, `text` | a durable message to a running Worker you supervise; it counts as read only when the Worker acknowledges it |
| `op=person action=name` | record what the person likes to be called |

Conventions: file only what the person asked for, in their words, and never widen it. Implementation goes to a Worker, never done by Chief or a Lead. Start nothing new without the person's word. A Worker's result counts only after asklog verifies it with the forge; "cannot tell" means stop and say so. Bring merge, cleanup and decisions back to the person as the one action they can take.

Layout commands from these folders need the scoped capability: `asklog layout list --capability "$ASKLOG_CHIEF_CAPABILITY"`, or `op=layout` with `request:{action: list|info|open-tab|split, ...}`.

## As a Worker

Your brief names your `taskId`, `dispatchId` and `generation`. Every report is `op=report` with those three, a fresh UUID `requestId`, `kind` and `text`:

1. `kind=ack` first, saying you read the brief.
2. `kind=progress` sparingly.
3. `kind=question` for something only the person can decide (`decisionId` revises one you asked), `kind=blocked` when you cannot continue, `kind=paused` with `until` (ISO time) for an outside wait that will clear on its own.
4. `kind=result` when done. A build task: validate, commit, push **only your branch**, open a PR, then `result:{url, head, validation}` with the full PR URL, the 40-character HEAD and the commands you ran with what they showed. A research task: `report:{source, summary}` with the report's path inside your copy; never push or open a PR.

Between steps and before reporting a result, read `op=inbox` and acknowledge every delivery. After a context compaction, re-read `op=briefing`. If your CLI has no MCP tool, the brief gives a shell command that takes the same JSON on stdin; use it and never read the capability file yourself.

Never merge, push a base branch, delete work, or talk to the person directly.
