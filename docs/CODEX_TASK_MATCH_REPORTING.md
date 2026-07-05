# Codex Task: Match Reporting, Standings, and Statistics

## Context
Repository: `chatchapak24-netizen/newlife-league-website`

Competition: **NEW LIFE CHAMPION LEAGUE U18 2026**
- Boys U18, 11-a-side
- Venue: Ratchaburi Provincial Stadium
- 8 teams
- Single round robin
- 28 matches across 7 weeks

Current team set for data validation:
1. ดรุณาราชบุรี
2. สารสิทธิ์พิทยาลัย
3. เบญจมราชูทิศราชบุรี
4. โพธาวัฒนาเสนี
5. ราชโบริกานุเคราะห์
6. สาธิตเทศบาลเมืองราชบุรี
7. รัฐราษฎรอุปถัมภ์
8. สาธิตมหาวิทยาลัยราชภัฏหมู่บ้านจอมบึง

Do not overwrite existing production records. Use the existing database as the source of truth and migrate safely.

## Objective
Implement an admin match-reporting workflow that records each match once and automatically produces:
- League table
- Results and fixtures
- Team statistics
- Player appearances, starts, substitute appearances, and minutes
- Goals, own goals, assists, penalties
- Yellow cards, second-yellow reds, straight reds
- Suspensions and eligibility warnings
- Clean sheets for goalkeepers when lineup/position data allows
- Match summaries suitable for public display

## First action: repository audit
Before coding:
1. Identify framework, package manager, database client, auth method, existing routes, existing tables, and deployment assumptions.
2. Find existing match/team/player/statistics code.
3. Reuse existing entities such as `matches`, `teams`, `players`, `season_players`, `match_events`, `match_lineups`, `match_substitutions`, and `standings` if they exist.
4. Document any schema conflicts or missing infrastructure in the PR description.

Do not replace the application wholesale unless the repository is genuinely an empty scaffold.

## Admin pages
Implement or complete these routes using the repository's existing routing conventions:

### Match operations
- Match list filtered by week/status/team
- Match report editor
- Team sheet/lineup editor
- Event timeline editor
- Confirmation and correction workflow

Suggested route shape when compatible with the existing app:
- `/admin/matches`
- `/admin/matches/[id]`
- `/admin/matches/[id]/lineups`
- `/admin/matches/[id]/report`

### Reports and statistics
- Standings preview
- Scorers
- Assists
- Cards and suspensions
- Team statistics
- Player statistics
- Data-quality/reconciliation warnings

Suggested route shape when compatible:
- `/admin/reports/standings`
- `/admin/reports/players`
- `/admin/reports/cards`
- `/admin/reports/data-quality`

## Public pages
Expose confirmed data only:
- `/fixtures`
- `/standings`
- `/stats/scorers`
- `/stats/assists`
- `/stats/cards`
- `/teams/[id]`
- `/players/[id]`

If equivalent pages already exist, extend them rather than creating duplicates.

## Match report fields
### Match metadata
- competition/season
- week/round
- match number
- date
- kickoff time
- venue
- home team
- away team
- match status: scheduled, live, finished, postponed, abandoned, cancelled
- report status: draft, pending_verification, confirmed, corrected
- referee, assistant referees, fourth official (optional)
- attendance/notes (optional)

### Team sheet
For each side:
- eligible registered player
- shirt number for this match
- starter/substitute
- captain
- goalkeeper
- position if available
- active/inactive on match sheet

The competition settings must control roster limits. Do not hard-code a contested limit when an existing configuration is present.

### Event timeline
Event types:
- goal
- own_goal
- assist
- penalty_goal
- penalty_missed
- yellow_card
- second_yellow_red
- red_card
- substitution
- player_of_match
- optional note/event correction

Each event should include:
- minute
- stoppage minute where applicable
- period: first_half, second_half, extra_time_first, extra_time_second, shootout when enabled
- team
- primary player
- secondary player where relevant (assist or player-out/player-in)
- reason/note where relevant
- created_by, created_at, updated_by, updated_at
- voided/corrected state rather than destructive deletion for confirmed reports

## Standings engine
Create a deterministic standings service from confirmed matches.

Competition settings must support:
- points for win/draw/loss
- optional penalty shootout after a draw and points for shootout winner/loser
- ordered tie-break rules
- whether head-to-head is evaluated before or after goal difference
- fair-play tie-break

At minimum calculate:
- played
- won
- drawn
- lost
- goals_for
- goals_against
- goal_difference
- points
- recent form

Do not assume the final scoring formula. Read existing rules/configuration and make the engine configurable.

## Player/team statistics derivation
Derive from confirmed lineups and events, not manually duplicated totals.

Player:
- appearances
- starts
- substitute appearances
- minutes where sufficient timing data exists
- goals
- own goals
- assists
- penalties scored/missed
- yellow cards
- red cards
- player-of-match awards
- clean sheets for goalkeepers
- current suspension status

Team:
- played/won/drawn/lost
- goals for/against
- clean sheets
- failed-to-score matches
- first-half and second-half goals when periods are available
- cards
- form
- biggest win/loss where meaningful

## Discipline and suspensions
- Add configurable yellow-card accumulation threshold; initial expected threshold is 2 yellows = 1-match suspension.
- Straight red and second-yellow red require a suspension record with a configurable minimum or manual disciplinary decision.
- Show warnings before confirming a lineup that includes a suspended/ineligible player.
- Do not silently block corrections to historic data; allow authorized correction and recalculate downstream suspension state.

## Data quality checks
Before confirmation, display blocking errors and non-blocking warnings:
- score does not match counted goal events
- goal scorer not on either team sheet
- assist references invalid player/team
- duplicated player in lineup
- invalid shirt-number duplicates within a team
- starter count violates competition settings
- substitution out/in inconsistency
- card assigned to invalid player
- suspended/ineligible player selected
- missing required officials or report fields when configured

Add a global data-quality page listing unresolved discrepancies.

## Audit and permissions
- Public users: read confirmed data only
- Admin/reporting staff: create and edit drafts
- Authorized competition admin: confirm reports and create corrections
- Track who confirmed and corrected each report
- Protect all write operations server-side
- Respect existing auth/RLS conventions

## Exports
Provide at least CSV export for:
- standings
- scorers/assists
- cards/suspensions
- match event log

A print-friendly match summary is desirable when compatible with the existing UI.

## Tests
Add automated tests for:
1. Normal win/draw/loss standings calculation
2. Configurable points rules
3. Goal difference and ordered tie-breaks
4. Own-goal attribution
5. Card accumulation and one-match suspension
6. Straight red/second-yellow state
7. Match score/event reconciliation
8. Confirmed-only public statistics
9. Correction recalculation

## Definition of done
- Existing app still builds and runs.
- Migration is safe and documented.
- Admin can complete one full match report end-to-end.
- Confirming the match updates standings and player/team statistics automatically.
- Public pages show confirmed data only.
- Invalid/inconsistent reports are flagged before confirmation.
- Lint, typecheck, tests, and production build pass.
- PR includes screenshots or a concise manual test record.
- No direct merge to `main`; leave changes for review.