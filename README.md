# asklog skills

Agent skills for working with [asklog](https://asklog.ai): the desktop app, daemon and `asklog` CLI that run terminal sessions and AI coding agents in panes, reachable from your phone and browser.

Each skill is a folder with a `SKILL.md` in the open [Agent Skills](https://agentskills.io) format, so it works with Claude Code, Codex and any other agent that reads skills. They are short on purpose: the commands and rules an agent needs, checked against the real CLI.

| Skill | Use it to |
|---|---|
| [`asklog-cli`](skills/asklog-cli/SKILL.md) | run the core commands (`ls`, `read`, `send`, `new`, `kill`, `layout`), read JSON output, avoid the common mistakes |
| [`asklog-panes`](skills/asklog-panes/SKILL.md) | find your own pane, split it, start a command or agent beside you, read and type into another pane, wait for it, close it safely |
| [`asklog-drive-agent`](skills/asklog-drive-agent/SKILL.md) | hand a prompt to another agent in a session and wait for its turn to end using `agentStatus` and `statusSeq` |
| [`asklog-session-info`](skills/asklog-session-info/SKILL.md) | know what asklog reports about sessions, agents, workspaces, tabs, panes and accounts, with `jq` one-liners |
| [`asklog-chief`](skills/asklog-chief/SKILL.md) | work with or as Chief, Leads and Workers: talking to Chief, starting a Lead, settings, and the `asklog_chief` tool's requests, dispatch, steering and reports |

The examples use `jq` and a bash or zsh shell.

## Install

With the [`skills`](https://www.npmjs.com/package/skills) CLI, which installs into the right folder for each agent it detects:

```sh
npx skills add asklog/skills                 # into the current project
npx skills add asklog/skills -g              # for your user, in every project
npx skills add asklog/skills --skill asklog-panes   # just one
npx skills add asklog/skills --list          # see what is here
```

Or by hand: clone this repository and copy or symlink the folders under `skills/` into your agent's skills folder.

```sh
git clone https://github.com/asklog/skills.git ~/src/asklog-skills

# Claude Code, for your user (or .claude/skills inside one project):
mkdir -p ~/.claude/skills
ln -s ~/src/asklog-skills/skills/asklog-* ~/.claude/skills/

# Codex and other agents that read .agents/skills, inside one project:
mkdir -p .agents/skills
ln -s ~/src/asklog-skills/skills/asklog-* .agents/skills/
```

Symlinks pick up updates with a `git pull`; copies do not.

Inside asklog's own Chief and Lead folders, asklog already writes its built-in `asklog-settings`, `asklog-layout`, `asklog-landing` and `asklog-machines` skills. The skills here use different names, so the two sets sit side by side.

## Contributing

Keep skills short and grounded: document only commands that exist in the released `asklog` CLI (`asklog --help` is the reference), test every snippet against a running daemon, and use exact session ids in every example.
