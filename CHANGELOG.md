# Changelog

What changed in `@quiel/cli`, newest first. Русская версия — [CHANGELOG.ru.md](https://github.com/Quiel-App/quiel-cli/blob/main/CHANGELOG.ru.md).

Some entries are marked **Platform** — those changed on the Quiel server and reached you without upgrading anything. They are listed because they change what your agent does, and a client-only changelog would leave that unexplained.

## 0.12.2 — 2026-10-07

### Fixed

- **The task waiter no longer dies after 10 minutes** ([#4](https://github.com/Quiel-App/quiel-cli/issues/4)). Claude Code applies a hook's `timeout` to `asyncRewake` hooks (600 seconds by default); only `async: true` hooks are exempt. We had read it the other way round, so `quiel wait-for-task` lived ten minutes, Claude Code killed it silently, and a task queued later woke nobody. `quiel init` now gives the waiter a timeout longer than its 8-hour wait, and the waiter leaves a minute early on its own. **Run `quiel init` again after updating.** Until you do, the waiter wakes the session for a short turn every nine minutes to restart itself, and asks you to run `quiel init`.
- **A dead waiter's lock no longer blocks the next one.** The lock was released only if the waiter exited normally, and on Windows a killed waiter's process number can go to another program. A lock the waiter has not refreshed for three minutes now counts as abandoned. The old shared `wait-for-task.lock` from before 0.11.1 is cleaned up.

### Added

- **The waiter keeps a log**: `~/.quiel/wait-for-task-state-<agentId>.log` records the start, connection failures (once, not on every poll), signals and why it exited. No tokens and no file contents. Before this, a waiter that died left nothing to go on.

### Platform

- **A Live chat survives a short connection drop.** It used to close as "the agent went offline" the moment the agent's socket dropped, even though the client reconnected seconds later. Now it waits a minute and a half for the agent to come back.

## 0.12.1 — 2026-10-07

### Added

- **A freshly started session tells you how to get going.** An agent in auto mode starts listening for tasks and Live chats only after its first turn: Claude Code's task waiter starts at the end of a turn, and Codex enters its `next_task` loop on a message. Right after start nobody could wake it, and a Live chat went unanswered. Now, when a session starts in auto mode, the terminal shows what to type: `/quiel-auto` for Claude Code (or start the session with `claude "/quiel-auto"`), "call next_task" for Codex.

### Fixed

- **Limits on the Claude desktop app are explained instead of missing** ([#3](https://github.com/Quiel-App/quiel-cli/issues/3)). Claude Code reports subscription limits only to its status line, and the desktop app's Code tab never calls it, so no numbers ever came from there and it looked broken. The client now tells the platform where Claude Code runs (`CLAUDE_CODE_ENTRYPOINT`, just that word). The sidebar says "limits: unavailable on desktop", and `quiel status` shows a "Subscription limits" line with the reason. Hitting the limit is still reported from the desktop app.

### Platform

- **The Live chat shows the agent's status**: whether it is online (last signal) and when it last asked for work. If it is online but not listening, the chat says why and what to type.

## 0.12.0 — 2026-10-07

### Added

- **Live: a person can chat with your agent.** On the platform's new Live page a teammate picks a free agent in auto mode and asks it about the project — quick questions, help with tasks. The agent answers in the chat and may quote code. While the chat is open the agent does not take tasks and works **read-only**: it cannot edit files, run anything but reading commands (`ls`, `cat`, `rg`, `git log`/`diff`/`show` and similar), read `.env` and keys, or change tasks. When asked to do work, it saves a draft task to the backlog for the person to start. New tools: `live_wait`, `live_reply`, `live_create_draft`. Works for Claude Code and Codex.
- **Run `quiel init` again after updating.** The read-only guard needs the `PreToolUse` hook to see more tools than before (`MultiEdit`, `NotebookEdit`, `Read`, `Grep`, Codex's `apply_patch`, Quiel's own tools). Until you do, the platform keeps your agent out of Live and says why. The old hook entry is replaced, not duplicated.

### Fixed

- **A Codex agent no longer goes quiet when Claude Code settings sit in the same project.** The `Stop` hook saw the Claude Code waiter in `.claude/settings.json` and let Codex stop, though that waiter only wakes Claude Code. Codex now keeps polling for work, as it should.

## 0.11.1 — 2026-10-07

### Fixed

- **An agent in auto mode wakes up again when a task appears** ([#2](https://github.com/Quiel-App/quiel-cli/issues/2)). `quiel wait-for-task` read a shared `~/.quiel/state.json` instead of the agent's own `state-<agentId>.json`. That file does not exist, so the waiter saw manual mode and exited at once without asking the platform, and the agent sat idle until a person wrote to it. It has been broken since the waiter appeared in 0.6.0. Nothing to set up: update the client and restart the Claude Code session.
- **Two projects on one computer no longer block each other's waiter.** The lock file was shared by the whole machine, so the second agent saw "a waiter is already running" and never woke up. Each agent now has its own lock.
- **`quiel wait-for-task` says why it is not waiting.** When run by hand, it prints the reason (manual mode, a task is already in progress, project not connected). Claude Code writes this line only to its debug log, so it does not disturb the model.

## 0.11.0 — 2026-10-05

### Added

- **Codex: the platform sees your subscription limits too.** Codex hooks do not carry the 5-hour and weekly windows, and its status line takes no custom command, but Codex writes them to its own session log. At the end of every turn the `Stop` hook reads the latest entry and passes the platform the same two percentages and reset times as Claude Code does. Only the numbers leave your machine, not a line of the session. Nothing to set up: update the client and restart the Codex session. Not yet checked on a live Codex session.

## 0.10.1 — 2026-09-29

### Fixed

- **Running `quiel init` again no longer asks for everything from scratch.** It used to ignore both `.quiel.json` and the token saved in `~/.quiel/credentials.json` and ask for the platform URL, the token and the rest again — and since a token is shown only once, in practice that meant issuing a new one. Now, in an already connected directory, it shows what is set up and asks a single question; Enter keeps everything, including the token. Answer "n" to change something: each question then shows the current value, and Enter keeps it. A token saved for a different platform address is never reused. Without a terminal, the saved values are used as they are; a flag changes only its own field.

## 0.10.0 — 2026-09-29

### Added

- **The platform sees your subscription limit before it runs out.** Claude Code reports how much of the 5-hour and weekly windows is used only to its status line — hooks do not carry it. `quiel init` now sets `quiel statusline` as the project's status line: it passes the platform the two percentages and their reset times, and prints an ordinary line. **Your own status line is not lost** — if one was set, it is wrapped (`quiel statusline --then '…'`) and called with the same input, so it looks exactly as before; one set in your user settings is called too. Only the numbers leave your machine — not the directory, the model or the session cost. Needs a Pro or Max subscription; the numbers appear after the first model reply in a session. Run `quiel init` again to set it up.
- **Taking a task back no longer loses the work.** When the platform takes a task away from the agent — a person pressed "Return to the queue" — the client commits and pushes what it has to the task branch and tells the platform where it is, so the next agent continues from it. Work that was already committed but never pushed is pushed too; before, a clean working tree meant "nothing to save" and those commits stayed on your machine. The same applies when the agent runs out of its limit.
- **The agent is told why there is no work.** When its 5-hour window is 90% used, `next_task` says until what time no new tasks will come, instead of an empty queue it would keep asking about.

### Platform

- **Two bars under every agent in the sidebar — 5h and week**, with the percentage and reset time: green up to 70%, amber up to 90%, red beyond. Agents that cannot report limits say so instead of showing an empty space.
- **At 90% of the 5-hour window the agent gets no new tasks** until the window resets; it finishes the current one. On by default, switchable in the project settings. The agent's owner gets one notification per window with a link to the current task — handing it to another agent is a button, never automatic.
- **A task taken back from a working agent waits for its branch.** For up to a minute, or until the branch arrives, it is not given to anyone else — otherwise a free agent grabbed it first and started from scratch.

## 0.9.0 — 2026-09-29

### Added

- **The agent can see the project it plans.** Two new tools. `get_project` returns the organisation's roles — with how many people and agents in this project hold each one — the project's milestones, and its members with their agents. `create_milestone` creates a milestone (planning roles only, like `plan_task`). Until now an agent decomposing a repository had no way to learn any of this: in a real project it invented the roles `pm`, `backend`, `devops`, `mobile` and `web` for 45 tasks, none of which existed, and emulated milestones with `ms:*` labels because `plan_task` took a `milestoneId` it could never look up.
- **Warnings from the server now reach the model.** `create_task` and `plan_task` print the server's notes under **ВНИМАНИЕ** instead of dropping them.
- **The agent is told where roles come from.** The instructions `quiel init` writes into `CLAUDE.md` / `AGENTS.md` now say to take roles, milestones and members from `get_project` rather than invent them — so the agent goes there before its first attempt, not after a refusal. Run `quiel init` again to refresh the section; the rest of the file is left as it is.

### Platform

- **A task can no longer be addressed to a role that does not exist.** `create_task` and `plan_task` now refuse an unknown `roleKey`, and the refusal lists the roles that do exist — so an agent on any client version corrects itself on the first retry. A role that exists but that nobody in the project holds is accepted with a warning: the task will wait for someone to take the role.
- **Follow-ups with an unknown role no longer turn into orphan tasks.** Rejecting a whole `report` over one follow-up would throw away finished work, so the role is cleared instead, the follow-up reaches the person without one, and the agent is told which roles were cleared.
- **`plan_task` checks `milestoneId` and `assigneeId`.** Both used to be written as given — including a milestone from another project or a member who is not in this one.

## 0.8.0 — 2026-09-26

### Added

- **Tasks know which repository they write to.** A project can hold several repositories, and until now a task did not say which one it belonged to — the platform took the first it found, so a front-end task's pull request could quietly go to the back-end repository. The task card now has a repository field (shown only when there is more than one), `create_task` and `plan_task` take a `repoUrl`, and the client reports which repository its own checkout is — so the dispatcher stops handing an agent work it physically cannot do. **Only the address leaves your machine, never the contents.** An agent that reports nothing keeps taking tasks exactly as before.

- **The agent hears about a red CI instead of guessing.** Two things it could not see before. First: `testResult: PASSED` is the agent's word about a *local* run — if the GitHub checks on the pull request's head are red or still running, the delivery response now says so, with the names of the failing checks. It is a warning, not a refusal; the hard block stays a project setting. Second: when the repository's *default* branch is broken, the assignment and `get_context` say plainly that this is not a consequence of the agent's own work and not its job to fix inside the task — because in a real project `main` was red for days and agents kept going to fix someone else's problem.
- **The agent is told when a dependency's code may not be in the main branch.** A task counts as done by its status, not by where its code ended up — and in a real project a pull request was merged into an already-merged branch, so the code never reached `main` while the task closed as done and its dependants went to work on nothing. The dependency is still released by status (the opposite rule would hang every project that works without pull requests), but the assignment and `get_context` now carry a line naming the suspect dependency, and `report` warns when your own pull request is aimed at another task's branch instead of the default one.
- **The agent is told when a task has blown its budget.** A project can now set a task budget in live tokens. It is a warning, not a ceiling: the platform never stops the work. The card gets a mark, the project managers get one notification, and `report` asks the agent to say where the spend went — the task turned out larger than it looked, or it got stuck going in circles.

### Platform

Changes on the Quiel server — they reach you without upgrading the client.

- **A broken main branch is visible in the project.** GitHub sends a check event for every run, including the ones on `main` — and the platform used to throw those away, because they belong to no task. Now the project overview carries a banner naming the failing checks, with links to their logs and a button that opens a pre-filled "fix the CI" task. Judged by the branch's latest commit, so a failure that has since been fixed raises nothing.
- **A task can be sent back automatically when CI goes red after delivery.** Checks usually finish after the report: it arrives seconds after the push, the run takes minutes. A new project switch — off by default — lets the platform watch the pull request's head and, when the checks complete as failures, return the task to the agent the way a reviewer would, naming what failed. It spends an attempt, like any return, so three red deliveries in a row end in "Failed".
- **A dependency that may not have reached the main branch is marked in the card.** Next to the dependency, not in a separate panel: "done" and "the code is not in main" are two claims about the same row, and the second has to argue with the first where it stands. Only when the platform knows — pull requests exist, their base branch is known, and none was merged into the default one.
- **When a base pull request is merged, the platform reminds you to retarget the ones stacked on it.** GitHub only retargets them when the base branch is deleted; otherwise the merge goes into an already-merged branch and the code never reaches the default one. A note lands in the task's feed and the project managers get one notification listing what to retarget.
- **What a task cost, in money and by attempt.** The card now shows money next to the tokens wherever there is something to compute it from — rates set, spend from a metered agent — and, once a task has been sent back from review, a per-attempt breakdown: a single total never says whether the rework cost more than the original work. The project's Costs screen adds a split by task type, counted across every task of the period rather than the ten most expensive.

## 0.7.0 — 2026-09-25

### Added

- **`report` takes structured follow-ups.** After almost every task something is left over, and agents already list it in the report — as prose: "Follow-up: write tools — M2-07; OAuth discovery — M4-04…". Prose cannot be turned into a task with a button, and a week later nobody finds it. The new `followUps` field takes a list instead: each entry has a title, a type, a priority and a `kind` — `blocking` ("what is done cannot count as done without this") or `later`. A person turns them into tasks with one click, and the new task is linked to the original automatically.

- **The client reports how your agent's work is paid for.** Quiel shows what a task cost in tokens, but tokens only become money if it knows who pays: on a subscription the marginal cost of a task is zero. The client now works this out from its own environment and sends one word — `SUBSCRIPTION`, `API` or `CLOUD`. **No key, no variable name and no variable value leaves your machine**, only the conclusion. It is a guess, not a fact — Claude Code asks you to approve an API key before it overrides your subscription, and you may have declined — so the agent panel lets you correct it, and your correction survives reconnects.

### Platform

Changes on the Quiel server — they reach you without upgrading the client.

- **Red CI is visible where the decision is made.** The platform saw GitHub checks and showed them in a card above, but the accept button had no idea: tasks were accepted with a red CI because the agent's report said `testResult: PASSED` — its words, usually about a local run. A warning now stands next to the button. A warning, not a block: the hard block is a project setting and is off by default on purpose.
- **You can see whether a task's code reached the main branch.** A task counts as done by its status, not by where its code ended up. In a real project a pull request was opened on top of another task's branch; the base PR was merged into `main` first, this one into an already-merged branch, and the code never reached `main` while the task closed as done. The card now says when a merge went somewhere else — or when the PR was closed without merging.
- **The repository for a pull request is chosen, not guessed.** With several GitHub repositories on a project the platform took the first one, so a front-end task's PR could quietly go to the back-end repository. It now refuses and says so instead of tossing a coin.
- **A specification living in the repository is finally visible.** A document can now be added by **path** — `docs/SPEC.md` — instead of text or a file. The platform stores the address, not the contents: the agent opens the file on its own machine. Before this, `get_context` answered "no documents" and told the agent to ask a human about a file it could have opened.
- **The "Now" panel for an agent.** "In progress" used to mean three different things: a subagent writing code for thirty minutes, a hung session, a lost lease. The panel tells them apart: what the agent is doing, how long the work has been running, whether it has gone silent, and what it ran last.
- **What the agent said last is in the task card.** Progress notes lived in the feed tab, while the decision "keep waiting or step in" is made looking at the card.
- **Task progress in words.** Branch created, tests were run, code pushed, pull request opened — and when. Read from the command log, so it says what the agent *ran*, not what succeeded; a denied command is not a milestone.
- **The daily summary now also says what happened in the project.** Delivered over the day with pull request links, waiting for acceptance, stuck, queued per role. It goes to organisation owners and admins and to project managers.
- **A task says how long it took.** Three numbers, not one: time in progress, time waiting for a human, and the total. Derived from status transitions, not from what an agent reports about its own session.
- **A "Costs" section for the project.** Totals for a day, a week or a month, the ten most expensive tasks with their working time, and a breakdown by agent. With token rates set in the organisation settings, spend from metered agents is also shown in money.
- **Follow-ups from a report turn into tasks with one button.** A list under the result: tick what you want and create, or create all at once. A created task lands in the backlog, carries a line saying where it came from, and is linked to the original. For a partial delivery, the remainder becomes its own blocking task instead of sending the whole task back for rework.

## 0.6.4 — 2026-09-25

### Added

- **A changelog.** Until now there was none — no file, no GitHub releases, no page — so upgrading told you nothing about what you were upgrading. It now ships with the package, appears on the showcase repository and becomes the body of each release. You can subscribe: **Watch → Releases** on [Quiel-App/quiel-cli](https://github.com/Quiel-App/quiel-cli) sends mail and offers a feed; npm only ever shows the latest version.

Nothing else changed in the client. Releases 0.6.0, 0.6.2 and 0.6.3 have been written up retroactively — that is the history this file starts from.

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
