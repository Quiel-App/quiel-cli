# @quiel/cli

**Русская версия — [README.ru.md](https://github.com/Quiel-App/quiel-cli/blob/main/README.ru.md).**

[![npm](https://img.shields.io/npm/v/@quiel/cli)](https://www.npmjs.com/package/@quiel/cli)

The Quiel client: a CLI **and** an MCP server that connects your coding agent — Claude Code, Codex, Cursor or OpenCode — to a project on a [Quiel](https://quiel.app) platform.

Quiel is a hosted product that orchestrates a team and the coding agents its members run: tasks live in one graph, each person connects their own agent, and in autonomous mode the agent picks up tasks for its role, does the work, pushes to git and reports back — while people watch it happen and stay in the loop.

This package is the part that runs **on your machine**.

## What it does

- **Gives your agent the platform's tools over MCP** — take the next task, read project context and documents, read the files a human attached to the task, ask a human (and ask them to attach a file), request approval for a dangerous action, submit work.
- **Installs hooks into your agent** so a shell command is checked against the project's trust profile *before* it runs. A command that matches a stop pattern is refused; a command that needs a human gets one.
- **Keeps the connection and the task lease alive**, so the platform knows the task is being worked on and not abandoned.
- **Counts the tokens each task costs** and reports the totals, so a project can be looked at afterwards.

It never sends the contents of your files anywhere — only file names, paths and commands. The token counters are numbers only: the client reads them from the agent's own transcript on your machine and sends the totals, never the conversation.

Files go the other way only: a human attaches them in the browser, the agent downloads and reads them. The agent never uploads files of its own.

## Requirements

- Node.js 22 or newer
- An account on a Quiel platform, membership in a project, and at least one role there
- An agent token — issued once, on the project's **Agents** page

## Install

```bash
npm i -g @quiel/cli
quiel --version
```

## Quick start

Run this in your project directory:

```bash
quiel init
```

It asks which agent you use (offering the one it found in the directory first), the platform address, your token, the project key, the agent id and its roles. If you already know the agent, skip the question:

```bash
quiel init --agent codex
```

Then **restart your agent's session** — this is required, not a suggestion: the MCP server only starts when a session starts. Without a restart the agent will not show up, no matter how many times you run `init`.

```bash
quiel status --check   # connection, mode, current task
quiel connect          # hold the connection to the platform
```

To let the agent pick up tasks by itself:

```bash
quiel mode auto
```

## Supported agents

| Agent | Maker | `--agent` | State |
|---|---|---|---|
| Claude Code | Anthropic | `claude-code` | Verified by a live run |
| Codex | OpenAI | `codex` | Beta |
| Cursor | Anysphere | `cursor` | Beta |
| OpenCode | SST | `opencode` | Beta |

One client covers all of them. The platform's tools are MCP — an industry standard — and behave the same everywhere; only the hook format and the configuration files differ, and the client writes the right ones for the agent you picked.

### What "beta" means here

Support for Codex, Cursor and OpenCode is built from each platform's official documentation and covered by tests, but it has **not been verified by a live run yet**. In practice: the tools will show up, but check the protection yourself — ask the agent to run a command you know is forbidden and make sure it does not go through. Do not assume stop patterns are protecting you until you have seen a refusal with your own eyes.

`quiel init` and `quiel status` both say so out loud for these agents.

Two gaps worth knowing about, and neither is an omission on our side — the platform simply does not report the event, so there is nothing to intercept:

|  | Claude Code | Codex | Cursor | OpenCode |
|---|---|---|---|---|
| Platform tools (MCP) | yes | yes | yes | yes |
| Blocking dangerous commands | yes | yes | yes | yes |
| Path-based file protection | yes | yes | **no** | yes |
| Autonomous mode | yes | yes | yes | yes |
| Moving to "rate limited" on its own | yes | **no** | **no** | yes |

**OpenCode, specifically:** it had a bug where tool calls *from a subagent* did not trigger plugins at all ([sst/opencode#5894](https://github.com/anomalyco/opencode/issues/5894) — closed, but the fixed version is not named). Quiel's own instructions tell the agent to delegate work to a subagent, so test the refusal through a subagent.

## What `quiel init` writes

Everything goes into your project directory, so it is visible in `git status` and reviewable:

| Agent | Files |
|---|---|
| Claude Code | `.claude/settings.json`, `.mcp.json`, a section in `CLAUDE.md`, `.claude/commands/quiel-auto.md` |
| Codex | `.codex/hooks.json`, `.codex/config.toml`, a section in `AGENTS.md` |
| Cursor | `.cursor/hooks.json`, `.cursor/mcp.json`, a section in `AGENTS.md` |
| OpenCode | `opencode.json`, `.opencode/plugins/quiel.js`, a section in `AGENTS.md` |

Plus one line in `.gitignore` for the directory the agent works in.

**Your token goes into none of them.** It is stored separately in `~/.quiel/credentials.json` with `0600` permissions, outside the repository. `.quiel.json` — the file that *is* committed — holds only the platform address, the project key, the agent id, its roles and which agent program you chose.

Existing configuration is merged, not overwritten: other MCP servers, other hooks and your own comments survive. Running `init` again is the normal way to reconnect an agent, and it does not duplicate anything.

## Commands

| Command | What it does |
|---|---|
| `quiel init` | Connect this project: write `.quiel.json`, the agent's configuration and the instructions section |
| `quiel connect` | Hold the connection to the platform |
| `quiel status [--check]` | Show the project, the configured agent program, mode and current task |
| `quiel mode auto\|manual` | Switch between autonomous and manual |
| `quiel mcp` | Run the MCP server over stdio (your agent starts this itself) |
| `quiel hook <name> [--format …]` | Run a hook (your agent's configuration calls this) |

## Reporting a problem

Issues are open: **https://github.com/Quiel-App/quiel-cli/issues**

Please include the output of:

```bash
quiel --version
quiel status
```

`quiel status` does **not** print your token. It does print your platform address, project key and agent id — redact them if that matters to you.

## About this repository

This repository is the documentation and the issue tracker for `@quiel/cli`. Quiel is a commercial hosted product: the client is distributed as a built package under a proprietary licence, and its source is not published here. What the client does to your machine is documented above — and every file it writes lands in your project directory, where you can read it.

## Licence

Proprietary (`UNLICENSED`). Copyright © Quiel. All rights reserved.
