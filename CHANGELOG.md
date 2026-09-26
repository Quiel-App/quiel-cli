# Changelog

What changed in `@quiel/cli`, newest first. Русская версия — [CHANGELOG.ru.md](https://github.com/Quiel-App/quiel-cli/blob/main/CHANGELOG.ru.md).

Some entries are marked **Platform** — those changed on the Quiel server and reached you without upgrading anything. They are listed because they change what your agent does, and a client-only changelog would leave that unexplained.

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
