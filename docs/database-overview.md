# Database Overview

## Introduction

4Fantasy uses a relational database model designed to support a fantasy sports platform with championships, leagues, fantasy teams, real teams, players, rounds, matchups, scoring, rankings and historical results.

The production database schema is private because the project is intended for commercial use. This document presents a public, high-level overview of the database design, main entities, relationships and performance considerations without exposing sensitive implementation details.

The main database technology used by the platform is PostgreSQL, with Entity Framework Core as the ORM layer.

---

## Database Goals

- Represent a real-world fantasy sports domain.
- Preserve historical round data.
- Support league-specific rankings.
- Store fantasy team lineups by round.
- Store real match events used for player scoring.
- Support direct matchups between fantasy teams.
- Enable admin workflows for managing competitions.
- Improve performance for read-heavy screens.
- Keep the model extensible for future commercial features.
- Avoid exposing sensitive production details in the public case study.

---

## Technology Stack

| Area | Technology |
|---|---|
| Database | PostgreSQL |
| ORM | Entity Framework Core |
| Query Language | LINQ / SQL |
| Migrations | EF Core Migrations |
| Backend | ASP.NET Core |
| Data Access Pattern | Service-oriented data access with optimized read queries |
| Public Documentation | High-level schema only |

---

## High-Level Domain Model

```txt
Championship
 |
 ├── League
 |    |
 |    ├── Fantasy Team
 |    |    |
 |    |    ├── Team Round
 |    |    └── Matchups
 |    |
 |    └── League Ranking
 |
 ├── Round
 |    |
 |    ├── Player Round Score
 |    └── Team Round Score
 |
 ├── Real Team
 |    |
 |    └── Player
 |
 └── Real Match
      |
      └── Real Match Event
```

This structure allows the platform to manage both the real sports side and the fantasy competition side.

---

## Core Entity Groups

| Group | Purpose |
|---|---|
| User and Authentication | Stores users and access-related information. |
| Championship Management | Defines competitions and their configuration. |
| League Management | Organizes users into private or public fantasy leagues. |
| Team Management | Stores fantasy teams created by users. |
| Real Sports Data | Stores real teams, players, real matches and events. |
| Round Management | Controls the active round and historical round data. |
| Scoring | Stores player and fantasy team performance by round. |
| Matchups | Stores direct confrontations between fantasy teams. |
| Rankings | Stores classification and ordering rules. |
| Achievements | Stores badges or awards unlocked by users or teams. |
| Snapshots | Stores precomputed read models for performance. |
| Admin Data | Supports back-office workflows and data maintenance. |

---

## Main Entities

### User

```txt
User
- Id
- Name
- Email
- PasswordHash
- CreatedAt
- UpdatedAt
```

Responsibilities:

- Own fantasy teams.
- Join leagues.
- Access user-specific dashboard data.
- Interact with the platform as a fantasy sports manager.

Sensitive authentication details are not exposed in this case study.

### Championship

```txt
Championship
- Id
- Name
- Description
- CurrentRoundNumber
- IsActive
- CreatedAt
- UpdatedAt
```

Responsibilities:

- Group rounds, leagues, real teams, players and competition rules.
- Define the current competition context.
- Allow multiple competitions to exist independently.

### League

```txt
League
- Id
- ChampionshipId
- Name
- Description
- IsPublic
- CreatedByUserId
- CreatedAt
- UpdatedAt
```

Responsibilities:

- Group fantasy teams.
- Generate league-specific rankings.
- Support private or public competitions.
- Store matchup and classification context.

Relationship:

```txt
Championship 1---N League
User 1---N League
```

### Fantasy Team

```txt
FantasyTeam
- Id
- UserId
- ChampionshipId
- Name
- ShieldUrl
- TotalScore
- Budget
- CreatedAt
- UpdatedAt
```

Responsibilities:

- Store user identity inside a competition.
- Participate in leagues.
- Select players for each round.
- Accumulate score over time.
- Unlock achievements.

Relationship:

```txt
User 1---N FantasyTeam
Championship 1---N FantasyTeam
League N---N FantasyTeam
```

### League Fantasy Team

```txt
LeagueFantasyTeam
- Id
- LeagueId
- FantasyTeamId
- JoinedAt
```

Responsibilities:

- Allow a fantasy team to participate in one or more leagues.
- Support league-specific rankings.
- Support league-specific matchups.

### Real Team

```txt
RealTeam
- Id
- ChampionshipId
- Name
- ShortName
- LogoUrl
- CreatedAt
- UpdatedAt
```

Responsibilities:

- Group real athletes.
- Participate in real matches.
- Allow users to identify player origin.

### Player

```txt
Player
- Id
- RealTeamId
- ChampionshipId
- Name
- Position
- ImageUrl
- Price
- AverageScore
- IsActive
- CreatedAt
- UpdatedAt
```

Responsibilities:

- Be selected by fantasy teams.
- Receive performance events.
- Generate round scores.
- Contribute to fantasy team scoring.

### Round

```txt
Round
- Id
- ChampionshipId
- Number
- StartDate
- EndDate
- IsOpen
- IsClosed
- CreatedAt
- UpdatedAt
```

Responsibilities:

- Control when users can select players.
- Group real matches.
- Group player scoring.
- Group fantasy team scoring.
- Define historical performance.

### Team Round

```txt
TeamRound
- Id
- FantasyTeamId
- LeagueId
- RoundId
- RoundNumber
- LineupData
- CaptainPlayerId
- TotalScore
- CreatedAt
- UpdatedAt
```

Responsibilities:

- Store selected lineup.
- Store captain selection.
- Store calculated team score.
- Preserve historical round data.
- Support league-specific results.

### Player Round

```txt
PlayerRound
- Id
- PlayerId
- RoundId
- RoundNumber
- Score
- Goals
- Assists
- YellowCards
- RedCards
- CreatedAt
- UpdatedAt
```

Responsibilities:

- Store player performance.
- Support scoring calculations.
- Provide historical player statistics.
- Feed fantasy team score calculation.

### Real Match

```txt
RealMatch
- Id
- ChampionshipId
- RoundId
- HomeTeamId
- AwayTeamId
- MatchDate
- HomeScore
- AwayScore
- Status
- CreatedAt
- UpdatedAt
```

Responsibilities:

- Store real match data.
- Group match events.
- Provide source data for player scoring.
- Support admin workflows.

### Real Match Event

```txt
RealMatchEvent
- Id
- RealMatchId
- PlayerId
- EventType
- Minute
- PointsImpact
- CreatedAt
- UpdatedAt
```

Example event types:

```txt
Goal
Assist
YellowCard
RedCard
Save
OwnGoal
```

Responsibilities:

- Record real match events.
- Feed scoring calculations.
- Allow admins to manage match events.
- Support auditability of player scores.

### Matchup

```txt
Matchup
- Id
- LeagueId
- RoundId
- RoundNumber
- TeamAId
- TeamBId
- TeamAScore
- TeamBScore
- WinnerTeamId
- CreatedAt
- UpdatedAt
```

Responsibilities:

- Store head-to-head fantasy team matchups.
- Compare round scores.
- Determine winner, loser or draw.
- Feed league-specific ranking logic.

### Achievement

```txt
Achievement
- Id
- Code
- Name
- Description
- ImageUrl
- Criteria
- CreatedAt
- UpdatedAt
```

Responsibilities:

- Define achievement rules.
- Support gamification.
- Improve user engagement.

### Fantasy Team Achievement

```txt
FantasyTeamAchievement
- Id
- FantasyTeamId
- AchievementId
- RoundNumber
- UnlockedAt
```

Responsibilities:

- Store unlocked achievements.
- Preserve when and why an achievement was earned.
- Support profile and gamification screens.

---

## Ranking and Classification Data

League rankings are calculated using fantasy team performance and matchup results.

Typical ranking criteria may include:

```txt
- Total wins
- Total score
- Round score
- Draws
- Losses
- Tie-breaker rules
```

Example ranking read model:

```txt
LeagueRankingSnapshot
- Id
- LeagueId
- ChampionshipId
- RoundNumber
- FantasyTeamId
- Position
- Wins
- Draws
- Losses
- TotalScore
- GeneratedAt
- SnapshotVersion
- IsValid
```

---

## Snapshot Tables

Snapshots are used to improve performance and preserve historical consistency.

Potential snapshot tables:

```txt
ChampionshipRankingSnapshot
LeagueRankingSnapshot
LeagueResultSnapshot
RoundTeamSnapshot
RoundPlayerSnapshot
RealMatchSnapshot
```

Common snapshot fields:

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

## Data Consistency Rules

| Rule | Reason |
|---|---|
| A fantasy team belongs to a user and a championship. | Prevents cross-competition data conflicts. |
| A league belongs to one championship. | Keeps ranking context consistent. |
| A team round belongs to a specific round. | Preserves historical lineup and scoring. |
| A player round belongs to a player and a round. | Preserves individual performance. |
| A matchup belongs to a league and round. | Keeps confrontations league-specific. |
| Historical data should not change unexpectedly after round closure. | Protects user trust and ranking consistency. |
| Public screenshots must use mock data. | Protects privacy and commercial data. |

---

## Indexing Strategy

| Table | Suggested Index |
|---|---|
| FantasyTeam | `(UserId, ChampionshipId)` |
| League | `(ChampionshipId)` |
| LeagueFantasyTeam | `(LeagueId, FantasyTeamId)` |
| TeamRound | `(FantasyTeamId, RoundNumber)` |
| TeamRound | `(LeagueId, RoundNumber)` |
| PlayerRound | `(PlayerId, RoundNumber)` |
| PlayerRound | `(RoundId)` |
| RealMatch | `(ChampionshipId, RoundId)` |
| RealMatchEvent | `(RealMatchId)` |
| Matchup | `(LeagueId, RoundNumber)` |
| Matchup | `(TeamAId, TeamBId, RoundNumber)` |
| LeagueRankingSnapshot | `(LeagueId, RoundNumber)` |
| RoundTeamSnapshot | `(FantasyTeamId, RoundNumber)` |

---

## Query Performance Considerations

Recommended practices:

- Use read-optimized DTO projections.
- Avoid loading unnecessary navigation properties.
- Use pagination when listing large datasets.
- Use `AsNoTracking` for read-only queries.
- Avoid recalculating closed rounds on every request.
- Use snapshot tables for historical data.
- Add database indexes for common filters.
- Avoid returning full entity graphs to the frontend.

---

## Public Case Study Scope

This document intentionally provides a simplified database overview.

It is designed to show domain modeling capability, relational database thinking, awareness of performance issues, historical data strategy, commercial product awareness and data privacy responsibility.

It does not expose production source code, complete schema, private scoring algorithms, real data or secrets.

---

## Summary

The 4Fantasy database model is designed to support a complex fantasy sports domain with users, fantasy teams, real teams, players, rounds, scoring, matchups, rankings and achievements.

PostgreSQL provides relational consistency, while Entity Framework Core supports database access, migrations and application development productivity.

The database design also includes performance-oriented strategies such as indexes, optimized read models and snapshot tables for historical data.
