# Engineering case study

## 1. Finding a silently failing production service

**Context.** During the move to the new platform, every server function that was *actually deployed* was exported, hashed and compared with the repository copy. The repository is not proof of what runs.

**Finding.**
- The deployed code matched the repository, except for two lines in the push service.
- An extra three services existed that were not in the repository's function list: the expiry scheduler, the expiry watchdog and a top-up identity guard.
- The top-up guard hard-coded an **old project id** in its API calls. The platform's per-execution key belongs to the current project, so every call it made was rejected with 401. The guard had never been able to remove a single orphan deposit.

**Response.**
- The finding was recorded in the migration notes.
- Design decision: the guard's purpose moves into server-side validation on the deposit write itself. A request without a driver identity is refused instead of being cleaned up afterwards.
- This removes a whole class of "event handler failed silently" risks.

**Lesson.** Inventory what runs, not what the repository says, and read error paths, not just happy paths.

## 2. Two configuration sources disagreed

**Finding.**
- The developer's local configuration and a fallback constant in two services named a different project from the one production uses.
- Nothing visibly broke, because the production runtime injected the right value, but any manual run or new environment would have hit the wrong project.

**Response.**
- A single source of truth per environment, injected at deploy time.
- No project identifiers as code fallbacks.
- The export tooling takes the project id explicitly and checks it before reading anything: it lists the databases and record counts and stops on a mismatch.

## 3. Keeping two diverged apps maintainable

**Finding.** A normalised diff against the sister app showed:
- 57 identical files;
- 203 diverged files, about 25,000 changed lines;
- 71 Mingalarpar-only files;
- 56 Call Me-only files.

Copying fixes file by file would silently revert product features.

**Response.**
- The platform (backend) is shared.
- In the app, each platform-level fix is re-applied as an intentional, reviewed commit: session fencing, server clock, realtime reconnect, accept lock, offer fan-out handling.
- Product modules stay separate.
- A written migration log records every decision so work can be resumed or reviewed at any time.

## 4. Race-safe automation instead of "check then write"

**Problem.** Automations such as auto-freeze and staff-window release compete with humans who accept or release the same order at the same moment.

**Design.**
1. Read the order inside the transaction.
2. Stage the change.
3. Commit, and treat a conflict as "someone else acted first". Never retry blindly.
4. Never infer success from a lost acknowledgement; re-read the committed state instead.

Bounded sweeps of 5 pages × 100 orders keep a watchdog's cost flat however much history accumulates.

## 5. Project hygiene

The local project had grown to **4.5 GB**, almost all of it generated audit reports, backups and build outputs from earlier tooling.

A report-first clean-up script:
- listed everything with its size;
- deleted only regenerable caches;
- *moved* everything else to an archive (nothing deleted);
- never touched tracked files, signing keys or local configuration;
- packed uncommitted source changes so they could be preserved as a commit first.

Result: **48 MB** working tree, 4.1 GB archived.
