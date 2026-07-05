# NEW LIFE CHAMPION LEAGUE U18 — Codex Working Rules

## Product goal
Build a reliable competition reporting and statistics system for NEW LIFE CHAMPION LEAGUE U18. The system must let staff enter a match report once and derive standings, team statistics, player statistics, scorers, assists, cards, suspensions, appearances, lineups, substitutions, and match summaries from that single source of truth.

## Non-negotiable rules
- Inspect the existing repository, framework, database, and naming conventions before changing architecture.
- Reuse existing tables and types where possible. Do not create duplicate data-entry flows or duplicate derived-stat tables without a clear technical reason.
- Treat official match data as immutable audit-sensitive data. Preserve edit history or at minimum updated_at/updated_by metadata.
- Do not silently guess missing match facts. Support explicit statuses such as draft, pending verification, confirmed, and corrected.
- Validate that player totals reconcile with match and team totals.
- Make scoring rules and tie-break rules configurable in competition settings rather than hard-coding disputed values.
- Yellow-card suspension threshold must be configurable; current expected threshold is 2 accumulated yellow cards = 1-match suspension.
- Support Thai text correctly throughout the UI and database.
- Keep the public website read-only. Restrict data-entry and corrections to authorized admin users.
- Do not expose service-role keys, secrets, or private player documents to the client.

## Required workflow
1. Admin opens a scheduled match.
2. Admin records or confirms both team sheets.
3. Admin records match events by minute and stoppage time.
4. Admin enters/derives half-time and full-time result.
5. System runs reconciliation and validation.
6. Admin confirms the report.
7. Confirmed data updates all public standings and statistics automatically.
8. A correction creates an auditable revision and recalculates affected outputs.

## Data scope
- Competition/season settings
- Teams and registered players
- Fixtures and match status
- Starting XI, substitutes, captain, goalkeeper
- Goals, own goals, assists, penalties, missed penalties
- Yellow cards, second-yellow red cards, straight red cards
- Substitutions in/out
- Player of the match
- Referee crew and optional notes
- Team and player cumulative statistics
- Suspensions and eligibility warnings

## Validation requirements
- Full-time score must equal counted goal events, with own goals attributed correctly.
- A player event must reference a player eligible for one of the two teams, unless explicitly recorded as an unresolved event.
- The same player cannot be in both teams or duplicated in one lineup.
- Starting XI count, bench limit, and substitution limits must follow configurable competition settings.
- A substituted-out player cannot re-enter unless the competition setting explicitly allows it.
- Card totals, appearances, starts, minutes, and goals must be reproducible from raw match records.
- Confirmed matches should not be editable without a correction flow.

## Engineering expectations
- Add database migrations safely and idempotently.
- Add seed/demo data only when clearly separated from production data.
- Add unit tests for standings, tie-breaks, cards, suspensions, and match reconciliation.
- Add a concise README section describing setup, migrations, admin workflow, and deployment.
- Run lint, typecheck, tests, and build before completing the task.
- Do not merge directly to main. Work in the current branch and leave a clear PR summary.