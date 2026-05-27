# Architecture Overview

## Introduction

4Fantasy is a full stack fantasy sports platform designed to manage fantasy teams, real teams, players, leagues, rounds, matchups, rankings and performance-based scoring.

This document describes the high-level architecture of the platform, including the main system components, responsibilities, data flow, backend structure, frontend structure and key architectural decisions.

The production source code is private because the project is intended for commercial use. This repository provides a public technical case study focused on architecture, product design and engineering decisions.

---

## Architecture Goals

The architecture was designed with the following goals:

- Support a real-world fantasy sports domain with complex business rules.
- Keep the backend organized, maintainable and testable.
- Separate API responsibilities from business logic.
- Support read-heavy screens such as home dashboard, league rankings and round summaries.
- Improve performance through optimized queries and snapshot-based strategies.
- Provide a responsive Angular frontend for desktop and mobile users.
- Allow future evolution into a commercial product.
- Keep sensitive implementation details and business secrets private.

---

## High-Level System Architecture

The platform follows a layered full stack architecture.

```txt
User
 |
 v
Angular Web Application
 |
 v
ASP.NET Core REST API
 |
 v
Application Services
 |
 v
Domain Rules
 |
 v
Entity Framework Core
 |
 v
PostgreSQL Database
```

### Main Components

| Component | Responsibility |
|---|---|
| Angular Web Application | Provides the user interface for players, teams, leagues, rankings, rounds and admin workflows. |
| ASP.NET Core REST API | Exposes endpoints consumed by the Angular frontend. |
| Application Services | Coordinates business operations and use cases. |
| Domain Rules | Represents fantasy sports rules such as scoring, rankings, matchups and achievements. |
| Entity Framework Core | Handles data access and database mapping. |
| PostgreSQL | Stores users, teams, leagues, rounds, players, scores, matchups and historical data. |
| Snapshot/Read Strategy | Improves performance for historical and read-heavy screens. |

---

## Backend Architecture

The backend is built with ASP.NET Core and follows a service-oriented layered structure.

The main goal is to keep controllers thin and move business logic into application services.

```txt
Backend
 |
 ├── API Layer
 |    └── Controllers
 |
 ├── Application Layer
 |    └── Services
 |
 ├── Domain Layer
 |    └── Entities and Business Rules
 |
 ├── Infrastructure Layer
 |    └── Entity Framework Core and External Integrations
 |
 └── Database
      └── PostgreSQL
```

---

## API Layer

The API layer is responsible for receiving HTTP requests and returning HTTP responses.

Typical responsibilities:

- Expose REST endpoints.
- Validate request parameters.
- Call application services.
- Return DTOs to the frontend.
- Avoid direct business rule implementation inside controllers.

Example API areas:

```txt
/api/auth
/api/championships
/api/leagues
/api/teams
/api/players
/api/rounds
/api/matchups
/api/rankings
/api/admin/matches
/api/admin/scoring
```

Controllers should remain simple and focused on request/response orchestration.

---

## Application Layer

The application layer coordinates the main use cases of the system.

Typical responsibilities:

- Create or update fantasy teams.
- Manage league participation.
- Calculate round summaries.
- Generate rankings.
- Process fantasy matchups.
- Coordinate admin workflows.
- Prepare optimized data for dashboard screens.
- Apply business rules using domain entities and services.

Example services:

```txt
ChampionshipService
LeagueService
TeamService
PlayerService
RoundService
RankingService
MatchupService
ScoringService
AchievementService
HomeDashboardReadService
```

The application layer acts as the bridge between the API layer, domain rules and infrastructure.

---

## Domain Layer

The domain layer represents the core business concepts of the platform.

Main domain concepts:

| Domain Concept | Description |
|---|---|
| Championship | Represents a competition season or tournament. |
| League | Represents a group of fantasy teams competing together. |
| Fantasy Team | Represents the user's fantasy team. |
| Real Team | Represents a real-world sports team. |
| Player | Represents an athlete available for fantasy selection. |
| Round | Represents a competition round. |
| Team Round | Stores a fantasy team's selected lineup and score for a specific round. |
| Player Round | Stores a player's performance and score in a specific round. |
| Matchup | Represents a direct confrontation between two fantasy teams. |
| Ranking | Represents the classification of teams based on score and rules. |
| Achievement | Represents badges or awards unlocked by players or teams. |

The domain layer contains the business meaning of the system and should not depend directly on infrastructure details.

---

## Infrastructure Layer

The infrastructure layer provides technical implementations required by the application.

Responsibilities include:

- Database access using Entity Framework Core.
- PostgreSQL configuration.
- Database migrations.
- External service integrations.
- File/image storage integrations.
- Notification integrations.
- Background processing.
- Logging and monitoring.

Examples of infrastructure concerns:

```txt
DbContext
Entity Configurations
Repositories
External Storage Services
Notification Services
Background Jobs
Database Migrations
```

---

## Database Architecture

PostgreSQL is used as the relational database.

The domain requires relational consistency because many entities are connected through competitions, rounds, teams, players and scores.

Simplified domain relationship:

```txt
Championship
 |
 ├── Leagues
 |    |
 |    ├── Fantasy Teams
 |    |    |
 |    |    ├── Team Rounds
 |    |    └── Matchups
 |    |
 |    └── League Rankings
 |
 ├── Rounds
 |    |
 |    ├── Player Round Scores
 |    └── Team Round Scores
 |
 ├── Real Teams
 |    |
 |    └── Players
 |
 └── Real Matches
      |
      └── Match Events
```

The database is designed to support:

- Historical round data.
- League-specific rankings.
- Fantasy team lineups.
- Real match events.
- Player scoring.
- Matchup results.
- Admin operations.
- Performance-optimized reads.

More details are described in `database-overview.md`.

---

## Frontend Architecture

The frontend is built with Angular and TypeScript.

The main frontend goals are:

- Provide a responsive user experience.
- Support both desktop and mobile layouts.
- Organize features by business domain.
- Keep components reusable and maintainable.
- Communicate with the backend through typed services.
- Provide a polished product experience for fantasy sports users.

Suggested high-level structure:

```txt
src/app/
 |
 ├── core/
 |    ├── services/
 |    ├── guards/
 |    ├── interceptors/
 |    └── models/
 |
 ├── shared/
 |    ├── components/
 |    ├── pipes/
 |    └── utils/
 |
 ├── features/
 |    ├── auth/
 |    ├── home/
 |    ├── league/
 |    ├── lineup/
 |    ├── ranking/
 |    ├── round-summary/
 |    ├── matchups/
 |    └── admin/
 |
 └── layout/
      ├── topbar/
      ├── sidebar/
      └── shell/
```

More details are described in `frontend-structure.md`.

---

## Data Flow

A typical user flow follows this process:

```txt
User Action
 |
 v
Angular Component
 |
 v
Angular Service
 |
 v
ASP.NET Core API Controller
 |
 v
Application Service
 |
 v
Domain Rule / Business Logic
 |
 v
Entity Framework Core
 |
 v
PostgreSQL
 |
 v
Response DTO
 |
 v
Angular UI Update
```

Example: viewing a league ranking.

```txt
User opens league ranking page
 |
Angular calls ranking endpoint
 |
API receives league and championship parameters
 |
Application service loads ranking data
 |
Ranking rules are applied
 |
Optimized DTO is returned
 |
Angular displays ranking table
```

---

## Read Optimization Strategy

Some screens are read-heavy and may become expensive if every request recalculates all data.

Examples:

- Home dashboard.
- League rankings.
- Round summary.
- Matchup results.
- Historical player performance.
- League classification.

To improve performance, the architecture can use a read-optimized strategy based on snapshots.

```txt
Live Data
 |
 v
Calculation Services
 |
 v
Snapshot Tables
 |
 v
Read-Optimized Endpoints
 |
 v
Angular Dashboard
```

This approach allows the system to avoid recalculating historical data that no longer changes.

Example use cases:

| Scenario | Strategy |
|---|---|
| Current round still open | Read from live data. |
| Historical round closed | Read from snapshot. |
| Ranking page with many teams | Use precomputed ranking snapshot. |
| Home dashboard | Use optimized read service. |
| Round summary | Use cached or precomputed data when possible. |

More details are described in `performance-strategy.md`.

---

## Snapshot-Based Architecture

The snapshot strategy is used to improve performance and preserve historical consistency.

Potential snapshot tables:

```txt
ChampionshipRankingSnapshot
LeagueRankingSnapshot
LeagueResultSnapshot
RoundTeamSnapshot
RoundPlayerSnapshot
RealMatchSnapshot
```

Each snapshot may include metadata such as:

```txt
RoundNumber
GeneratedAt
SnapshotVersion
CalculationSource
IsValid
```

Benefits:

- Faster historical reads.
- Reduced database load.
- Consistent historical results.
- Easier debugging of past rounds.
- Better user experience during high-traffic moments.

---

## Admin Architecture

The platform includes admin workflows for managing competition data.

Admin responsibilities may include:

- Managing championships.
- Managing players.
- Managing real teams.
- Registering real matches.
- Registering match events.
- Updating round status.
- Reviewing fantasy scoring.
- Managing achievements and business rules.

Admin features are separated from regular user features to keep permissions, UI flows and responsibilities clear.

```txt
Admin User
 |
 v
Angular Admin Panel
 |
 v
Admin API Endpoints
 |
 v
Application Services
 |
 v
PostgreSQL
```

---

## Security Considerations

The public case study does not expose production code, credentials or sensitive implementation details.

Security principles applied to the platform:

- Keep production source code private.
- Never expose environment variables.
- Never expose database credentials.
- Never expose real user data.
- Use mock data in public screenshots.
- Separate admin workflows from regular user workflows.
- Protect sensitive endpoints with authentication and authorization.
- Keep commercial business rules abstracted in public documentation.

---

## Deployment Overview

The platform can be deployed using a structure similar to:

```txt
Angular Frontend
 |
 | hosted on Vercel or similar frontend hosting
 |
ASP.NET Core API
 |
 | hosted on cloud backend provider
 |
PostgreSQL Database
 |
 | hosted on managed database provider
```

Deployment concerns:

- Environment-specific configuration.
- Secure connection strings.
- HTTPS.
- Database migrations.
- Logging.
- Monitoring.
- Backup strategy.
- API versioning.
- CI/CD pipeline.

---

## Observability and Maintainability

To support production-level maintenance, the architecture should include:

- Structured logs.
- Global exception handling.
- Request tracing.
- Performance monitoring.
- Health check endpoints.
- Clear API responses.
- Error messages that help debugging without exposing sensitive information.
- Documentation for setup and troubleshooting.

These practices help the project look more professional and production-ready.

---

## Architectural Decisions

Some important architectural decisions behind the platform:

| Decision | Reason |
|---|---|
| ASP.NET Core REST API | Reliable backend framework for enterprise web applications. |
| Angular Frontend | Strong structure for building scalable frontend applications. |
| PostgreSQL | Relational database suitable for complex domain relationships. |
| Entity Framework Core | Productive ORM with migrations and LINQ support. |
| Layered Architecture | Separates API, application logic, domain rules and infrastructure. |
| Snapshot Strategy | Improves performance for historical and read-heavy data. |
| DTO-based API responses | Prevents overexposing internal entities and improves frontend contracts. |
| Private production source code | Protects commercial product potential. |
| Public case study repository | Demonstrates technical skills without exposing intellectual property. |

---

## Why This Architecture Matters

4Fantasy is not only a CRUD application. It includes:

- Complex domain modeling.
- Round-based scoring.
- League-specific rankings.
- Fantasy matchups.
- Historical data consistency.
- Admin workflows.
- Performance-sensitive screens.
- Responsive frontend experience.
- Future commercial product considerations.

The architecture was designed to support these needs while keeping the system maintainable and prepared for future growth.

---

## Next Documents

For more details, see:

- `database-overview.md`
- `api-overview.md`
- `performance-strategy.md`
- `frontend-structure.md`
- `business-rules.md`
- `roadmap.md`
