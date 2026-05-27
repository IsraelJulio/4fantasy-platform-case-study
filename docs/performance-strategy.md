# Performance Strategy

## Introduction

4Fantasy is a fantasy sports platform with multiple read-heavy screens, round-based calculations, league rankings, fantasy matchups, historical data and admin workflows.

Because fantasy sports platforms can generate many repeated reads during active rounds, rankings and result updates, performance is a key architectural concern.

This document describes the high-level performance strategy used in the platform, including read optimization, snapshot-based data, query improvements, indexing, API design and frontend performance considerations.

The production implementation is private because the project is intended for commercial use. This document provides a public technical overview without exposing sensitive implementation details.

---

## Performance Goals

- Reduce response time on high-traffic screens.
- Avoid unnecessary recalculation of historical data.
- Reduce duplicated database queries.
- Improve the mobile user experience.
- Keep API responses small and focused.
- Use read-optimized endpoints for complex screens.
- Preserve consistency of closed rounds.
- Support future commercial growth.
- Make the platform easier to monitor and maintain.

---

## Main Performance Challenges

| Challenge | Why It Matters |
|---|---|
| Home dashboard loading | Users expect fast access to team, round, matchup and league information. |
| League rankings | Rankings may require aggregation across many fantasy teams. |
| Round summaries | Round data combines players, teams, scores, matchups and statistics. |
| Historical data | Closed rounds should not be recalculated on every request. |
| Matchup details | Each user wants quick access to their current confrontation. |
| Mobile performance | The application must feel fast on mobile devices. |
| Admin workflows | Admin screens may load matches, events, players and scoring data. |
| Repeated reads | Many users may open the same ranking or round summary repeatedly. |

---

## Read vs Write Characteristics

| Area | Read Frequency | Write Frequency |
|---|---:|---:|
| Home dashboard | High | Low |
| League ranking | High | Medium |
| Round summary | High | Medium |
| Player cards | High | Low |
| Matchup details | High | Low |
| Admin match events | Medium | Medium |
| Lineup selection | Medium | Medium |
| Historical round data | High | Very low after round closure |

---

## High-Level Strategy

```txt
Optimized Queries
 |
DTO Projections
 |
Read-Specific Services
 |
Snapshot Tables
 |
Indexes
 |
Frontend Loading Strategy
 |
Monitoring and Iteration
```

---

## Read-Optimized API Endpoints

Some screens should not depend on many independent API calls.

The home screen may need:

```txt
- Current round
- User fantasy team
- Current lineup status
- Current matchup
- League summary
- Ranking position
- Round score
- Recent achievements
```

Instead of making the frontend call multiple endpoints separately, the backend can expose:

```txt
GET /api/dashboard/home
```

Benefits:

- Fewer HTTP requests.
- Less duplicated frontend orchestration.
- Smaller payloads.
- Better mobile performance.
- Easier backend optimization.
- Improved perceived speed.

---

## Example Home Dashboard Read Model

```txt
HomeDashboardDto
- CurrentRound
- MarketStatus
- MyTeamSummary
- MyCurrentMatchup
- LeagueSummary
- RoundScore
- RankingPosition
- RecentAchievements
```

---

## DTO Projection Strategy

One of the most important performance practices is avoiding full entity graph loading.

```txt
Database Query
 |
Projection
 |
DTO
 |
API Response
```

Benefits:

- Less memory usage.
- Smaller SQL result sets.
- Reduced serialization cost.
- Lower risk of circular references.
- Faster frontend responses.
- Clear API contracts.

---

## Avoiding Overfetching

Overfetching happens when the API returns more data than the screen needs.

Preferred approach:

```txt
Return LeagueRankingRowDto
 |
Includes only:
- Position
- Team name
- Shield URL
- Wins
- Draws
- Losses
- Round score
- Total score
```

---

## Snapshot Strategy

Some data becomes stable after a round is closed.

Examples:

- Final player score.
- Final fantasy team score.
- Final matchup result.
- Final league ranking.
- Final round summary.
- Final achievements unlocked in that round.

```txt
Live Calculation
 |
Round Closed
 |
Generate Snapshot
 |
Read Snapshot for Historical Requests
```

---

## Snapshot-Based Read Flow

```txt
Request Historical Round Data
 |
Check if Round is Closed
 |
If Snapshot Exists
 |       |
 |       v
 |   Return Snapshot
 |
If Snapshot Does Not Exist
 |
Calculate from Live Tables
 |
Optionally Generate Snapshot
 |
Return Result
```

---

## Potential Snapshot Tables

```txt
ChampionshipRankingSnapshot
LeagueRankingSnapshot
LeagueResultSnapshot
RoundTeamSnapshot
RoundPlayerSnapshot
RealMatchSnapshot
RoundSummarySnapshot
```

Common snapshot metadata:

```txt
- Id
- ChampionshipId
- LeagueId
- RoundNumber
- Data
- GeneratedAt
- SnapshotVersion
- CalculationSource
- IsValid
```

---

## Snapshot Benefits

| Benefit | Explanation |
|---|---|
| Faster historical pages | Closed rounds can be loaded without recalculating everything. |
| Lower database load | Expensive aggregations are reduced. |
| Consistent results | Historical rankings remain stable after round closure. |
| Easier debugging | Snapshot version and generation date help investigate issues. |
| Better UX | Users get faster round summaries and rankings. |
| Scalability | The system handles repeated reads more efficiently. |

---

## Live Data vs Snapshot Data

| Scenario | Data Source |
|---|---|
| Current round open | Live data |
| Current round in progress | Live data or partially calculated data |
| Round recently closed | Live calculation followed by snapshot generation |
| Historical round | Snapshot data |
| Snapshot invalidated | Recalculate and regenerate snapshot |
| Admin recalculation | Live recalculation and snapshot refresh |

---

## Indexing Strategy

| Table | Suggested Index | Reason |
|---|---|---|
| FantasyTeam | `(UserId, ChampionshipId)` | Find user's team quickly. |
| LeagueFantasyTeam | `(LeagueId, FantasyTeamId)` | League membership lookup. |
| TeamRound | `(FantasyTeamId, RoundNumber)` | Team history by round. |
| TeamRound | `(LeagueId, RoundNumber)` | League round calculations. |
| PlayerRound | `(PlayerId, RoundNumber)` | Player history by round. |
| PlayerRound | `(RoundId)` | Round scoring lookup. |
| Matchup | `(LeagueId, RoundNumber)` | League matchups by round. |
| Matchup | `(TeamAId, TeamBId, RoundNumber)` | Direct confrontation lookup. |
| LeagueRankingSnapshot | `(LeagueId, RoundNumber)` | Historical ranking reads. |
| RoundTeamSnapshot | `(FantasyTeamId, RoundNumber)` | Historical team results. |
| RealMatch | `(ChampionshipId, RoundId)` | Admin and round match lookup. |
| RealMatchEvent | `(RealMatchId)` | Events by match. |

---

## Query Optimization Practices

- Use `AsNoTracking` for read-only queries.
- Use projections instead of loading full entities.
- Avoid unnecessary `Include` chains.
- Avoid returning entity graphs directly.
- Use pagination for large lists.
- Use filtering on the database side.
- Avoid repeated queries inside loops.
- Use aggregate queries carefully.
- Add indexes for common filters.
- Review generated SQL for heavy queries.
- Separate read services from write services where needed.

---

## Avoiding N+1 Queries

Problem:

```txt
Load 50 teams
 |
For each team, query round score
 |
50 additional queries
```

Preferred approach:

```txt
Load all required team scores in one query
 |
Group by team
 |
Project into DTO
```

---

## Pagination Strategy

Large datasets should be paginated.

Examples:

- Player list.
- League list.
- Ranking list for large leagues.
- Admin match events.
- User notifications.
- Historical rounds.

---

## Caching Strategy

Potential cache candidates:

| Data | Cache Suitability |
|---|---|
| Public league list | Medium |
| Real team list | High |
| Player catalog | Medium |
| Closed round summary | High |
| Historical rankings | High |
| Current user dashboard | Low to Medium |
| Admin mutable data | Low |

Caching should be used with clear invalidation rules.

---

## Frontend Performance Strategy

- Use screen-specific API calls.
- Avoid unnecessary repeated HTTP requests.
- Use loading skeletons for perceived performance.
- Use lazy-loaded feature modules where possible.
- Use pagination or virtual scrolling for large lists.
- Avoid rendering large lists all at once on mobile.
- Reuse shared components.
- Keep image sizes optimized.
- Use responsive layouts.
- Cache stable dropdown/reference data in services when appropriate.

---

## Admin Performance

Admin screens may include real matches, match events, player lists, scoring data, round status and recalculation actions.

Recommended practices:

- Use filters by championship and round.
- Use pagination.
- Use search with server-side filtering.
- Avoid loading all events at once.
- Separate admin write actions from user read endpoints.
- Provide confirmation for expensive recalculations.
- Log admin recalculation actions.

---

## Background Processing

Some operations can be handled outside direct user requests.

```txt
Admin closes round
 |
Background job starts
 |
Calculate player scores
 |
Calculate team scores
 |
Calculate matchups
 |
Generate rankings
 |
Generate snapshots
 |
Notify users
```

---

## Observability

Recommended practices:

- Log slow API requests.
- Log data source used: live or snapshot.
- Log snapshot generation duration.
- Track dashboard response time.
- Track ranking query duration.
- Track admin recalculation duration.
- Track error rates.
- Add health check endpoints.

Example log fields:

```txt
RequestId
Endpoint
UserId
ChampionshipId
LeagueId
RoundNumber
DataSource
DurationMs
StatusCode
```

---

## Performance Anti-Patterns to Avoid

| Anti-Pattern | Impact |
|---|---|
| Returning full entity graphs | Large payloads and slow serialization. |
| Using many unnecessary Includes | Expensive SQL queries. |
| Recalculating closed rounds every time | Slow historical pages. |
| Running queries inside loops | N+1 query problem. |
| Loading all players without pagination | Slow UI and high memory usage. |
| Making many API calls for one screen | Poor mobile experience. |
| No indexes on frequent filters | Slow database queries. |
| Caching without invalidation | Incorrect data. |
| Exposing admin-heavy endpoints to regular screens | Security and performance risks. |

---

## Public Case Study Scope

This document intentionally provides a high-level performance strategy. It demonstrates awareness of real-world performance problems, backend read optimization, database indexing strategy, snapshot-based thinking, DTO design, mobile-first performance concerns, observability and maintainability.

It does not expose production source code, private performance benchmarks, real user traffic data, sensitive business algorithms, database credentials or complete internal implementation.

---

## Summary

The 4Fantasy performance strategy focuses on reducing unnecessary recalculation, optimizing read-heavy screens, using DTO projections, applying indexes, avoiding overfetching, using snapshots for historical data and improving the mobile user experience.
