# Changelog

What changed in `@quiel/cli`, newest first. Русская версия — [CHANGELOG.ru.md](https://github.com/Quiel-App/quiel-cli/blob/main/CHANGELOG.ru.md).

Some entries are marked **Platform** — those changed on the Quiel server and reached you without upgrading anything. They are listed because they change what your agent does, and a client-only changelog would leave that unexplained.

## 0.6.3 — 2026-09-24

### Added

- **`link_tasks` and `unlink_tasks` take a list.** After breaking down a specification there are dozens of links, and one call now costs one model turn instead of forty. Each link is applied on its own: a refusal on one does not cancel the rest, and the reply reports every line. At most 100 per call — a longer list almost certainly was not read before it was sent.
- **`get_context(include:['related'])` returns both sides of a link and its type.** Before, an agent saw only the tasks it depends on — never that its own work was holding something up — and could not tell a hard `BLOCKS` from a "see also" `RELATES`. Both `kind` and `direction` are now always present.

### Fixed

- **`report` no longer advises repeating a call that cannot succeed.** When the task was no longer yours, the reply said "the task is still yours — you can safely repeat report". Repeating gave the same refusal, while the stop hook kept demanding the work be submitted: the agent could neither finish nor stop. Now `report`, `release_task` and `log` recognise that refusal, drop the task from local state and say what to do with the work instead.
- **The `report` tool description says what each status does.** It never explained that `partial` and `failed` end the task and release the lease, so a model could pick one without knowing the consequence.

### Platform

- **`report(partial)` now sends the task to review instead of parking it.** It used to land in "waiting for a human", which has no route to "done" — a finished task with two caveats could not be accepted at all. Review is where "accept or send back" already lives. Auto-acceptance does not apply to a partial submission.
- **What is left to do is visible.** `nextSteps` from a partial report is shown under the result, above the accept button, instead of vanishing into the event payload.
- **An agent is told when it loses a task, whatever the reason.** Only an expired lease produced a warning before; a task taken back by a subscription limit or by a person in the interface was removed silently, and the agent kept working on it.
- **The owner is told when the queue stands still while an agent is free.** That is what an agent without its task watcher looks like — usually `quiel init` was not re-run after an upgrade.

### Upgrading

`npm i -g @quiel/cli`, then reconnect the MCP server (`/mcp` in Claude Code). Re-running `quiel init` is not needed: nothing changed in the files it writes.

## 0.6.2 — 2026-09-24

### Fixed

- **An agent upgraded from 0.6.0 without re-running `init` went silent instead of idling.** `npm i -g` replaces the binary and does not touch `.claude/settings.json`, so the watcher added in 0.6.0 was never installed — and the stop hook let the agent stop anyway. The hook now reads the configuration and only allows idling when the watcher is actually there; without it the old polling behaviour returns, and the human is told to run `init`.
- **A server refusal no longer traps the agent in a loop.** The hook could not tell "there is no work" from "I could not ask", and kept demanding a task that could not be fetched.
- **`waitSec` documented 300 seconds while the code used 90.** A model builds its behaviour from the description, so the gap cost more than a typo. The ceiling is now 110 seconds: at 600 there was a window where the server had already handed out a task while the call was killed by a client timeout, leaving the task in progress with a lease nobody was holding.
- **`parseRoles` split on commas only** — roles separated by spaces were read as one.

### Platform

- **A task that still has a lease is no longer a dispatch candidate.** One orphaned lease row made the dispatcher fail on a unique index and answer `internal_error` to *every* task in the project.

## 0.6.1

Never published. The version was bumped in the repository and superseded by 0.6.2 before release.

## 0.6.0 — 2026-09-24

### Added

- **The agent idles instead of asking for work in a loop, and is woken when work appears.** Polling cost a model turn every minute and a half — over a night of an empty queue that is dozens of empty calls and a bloated context. Waiting costs a socket and a few bytes. Requires `quiel init` to be re-run: the watcher is a hook, and upgrading the package does not write hooks.
- **Planning tools for an agent with the manager role**: `link_tasks` / `unlink_tasks` for dependencies, `plan_task` to edit any task of the project without holding its lease, and `dependsOn` in `create_task` so links can be set at creation time.

## Earlier versions

No changelog was kept before 0.6.0. Versions 0.1.0 through 0.5.3 were released while the platform was still being built and had no outside users.
