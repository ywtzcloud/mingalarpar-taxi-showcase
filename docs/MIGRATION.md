# Migration

Mingalarpar follows the path proven with [Call Me Taxi](https://github.com/ywtzcloud/callme-taxi-showcase/blob/main/docs/MIGRATION.md): **move the rules, don't rewrite them**, then improve on the owned platform.

## Steps and verification

| # | Step | Verified by |
|---|---|---|
| 1 | Inventory app, server functions and local project | normalised source diff; deployed vs repository hashes |
| 2 | Harden the new server | firewall rules, fail2ban status, time zone, swap |
| 3 | Read-only export (least-privilege key, hidden prompt) | per-collection counts against the source; 879/879 files downloaded with zero missing |
| 4 | Evidence check of every scheduled and event service | production data (e.g. how many orders were auto-frozen in the last 30 days) and execution history, before deciding what to port |
| 5 | Deploy the platform with Mingalarpar rules; import | automated tests on real PostgreSQL; record reconciliation |
| 6 | Live dispatch drills and load test | synthetic drivers with real users' push tokens set aside; p95 per operation |
| 7 | Testing app (separate package and Firebase project) on the new backend | field testing on phones |
| 8 | Cutover: final export → import → switch | smoke test; monitoring and backups active from the first minute |

## Export results

- 39 collections and **233,044 records**. Managed platforms often cap reported totals at 5,000, so exported counts above that are expected and were checked by paging to the end.
- **879 media files** (64 MB).
- **9 deployed services**, each with its settings and the *names* of its environment variables. Secret values are never exported.

## Principles

- Nothing in the old system is changed during the move. Every export call is a read.
- Every step prints what it did, and a person reviews it before the next step.
- Secrets are typed into hidden prompts on the server, never pasted into chat or committed.
- Real users are never disturbed by tests.
