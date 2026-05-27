# API Overview

## Introduction

4Fantasy exposes a REST API built with ASP.NET Core to connect the Angular frontend with the backend business logic, database and platform services.

The API is responsible for supporting fantasy team management, league participation, player selection, round-based scoring, matchups, rankings, real match administration and dashboard data.

The production source code and full API implementation are private because the project is intended for commercial use. This document provides a public high-level overview of the API design, endpoint groups, request/response patterns and technical decisions without exposing sensitive implementation details.

---

## API Goals

- Provide a clear contract between the Angular frontend and the backend.
- Keep controllers focused on HTTP request/response orchestration.
- Move business logic into application services.
- Use DTOs to avoid exposing internal database entities.
- Support complex fantasy sports workflows.
- Optimize read-heavy screens with specific endpoints.
- Protect sensitive operations with authentication and authorization.
- Keep admin features separated from regular user features.
- Support future scalability and commercial evolution.

---

## Technology Stack

| Area | Technology |
|---|---|
| Backend Framework | ASP.NET Core |
| API Style | REST |
| Language | C# |
| Database Access | Entity Framework Core |
| Database | PostgreSQL |
| Documentation | Swagger / OpenAPI |
| Authentication | Token-based authentication |
| Frontend Consumer | Angular / TypeScript |
| Data Format | JSON |

---

## High-Level API Flow

```txt
Angular Component
 |
 v
Angular Service
 |
 v
HTTP Request
 |
 v
ASP.NET Core Controller
 |
 v
Application Service
 |
 v
Domain Rules
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

The API layer does not directly implement complex business logic. Instead, it delegates operations to application services.

---

## API Design Principles

| Principle | Description |
|---|---|
| Resource-oriented endpoints | Endpoints are grouped around business resources such as teams, leagues, rounds and players. |
| DTO-based responses | API responses use DTOs instead of exposing database entities directly. |
| Thin controllers | Controllers orchestrate requests and delegate business logic to services. |
| Clear separation of concerns | HTTP handling, business rules and data access are separated. |
| Consistent response structure | Responses should be predictable for frontend consumption. |
| Read-optimized endpoints | Heavy screens use specific endpoints to avoid overfetching. |
| Secure admin operations | Admin endpoints are separated and protected. |
| Mock-safe documentation | Public examples avoid real user data and production details. |

---

## Endpoint Organization

```txt
/api/auth
/api/users
/api/championships
/api/leagues
/api/teams
/api/players
/api/rounds
/api/lineups
/api/matchups
/api/rankings
/api/dashboard
/api/achievements
/api/admin
```

---

## Authentication API

Example endpoints:

```txt
POST /api/auth/register
POST /api/auth/login
POST /api/auth/refresh-token
POST /api/auth/logout
GET  /api/auth/me
```

Typical responsibilities:

- Register users.
- Authenticate users.
- Return access tokens.
- Identify the current logged-in user.
- Protect private fantasy team and league data.

Example login request:

```json
{
  "email": "user@example.com",
  "password": "StrongPassword123"
}
```

Example login response:

```json
{
  "accessToken": "mocked-access-token",
  "refreshToken": "mocked-refresh-token",
  "expiresIn": 3600,
  "user": {
    "id": 1,
    "name": "Demo User",
    "email": "user@example.com"
  }
}
```

---

## User API

```txt
GET /api/users/me
PUT /api/users/me
GET /api/users/me/teams
GET /api/users/me/leagues
```

---

## Championship API

```txt
GET  /api/championships
GET  /api/championships/{championshipId}
GET  /api/championships/{championshipId}/current-round
GET  /api/championships/{championshipId}/summary
```

Example response:

```json
{
  "id": 1,
  "name": "Demo Futsal Championship",
  "currentRoundNumber": 5,
  "isActive": true
}
```

---

## League API

```txt
GET  /api/leagues
GET  /api/leagues/public
GET  /api/leagues/{leagueId}
POST /api/leagues
POST /api/leagues/{leagueId}/join
GET  /api/leagues/{leagueId}/teams
GET  /api/leagues/{leagueId}/ranking
GET  /api/leagues/{leagueId}/matchups
```

Example ranking response:

```json
{
  "leagueId": 10,
  "roundNumber": 5,
  "ranking": [
    {
      "position": 1,
      "teamId": 101,
      "teamName": "Demo Warriors",
      "wins": 4,
      "draws": 0,
      "losses": 1,
      "roundScore": 72.5,
      "totalScore": 350.8
    }
  ]
}
```

---

## Fantasy Team API

```txt
GET  /api/teams/my
GET  /api/teams/{teamId}
POST /api/teams
PUT  /api/teams/{teamId}
GET  /api/teams/{teamId}/history
GET  /api/teams/{teamId}/achievements
```

---

## Player API

```txt
GET /api/players
GET /api/players/{playerId}
GET /api/players/by-position/{position}
GET /api/players/by-team/{realTeamId}
GET /api/players/{playerId}/history
```

Typical query parameters:

```txt
championshipId
roundNumber
position
realTeamId
search
page
pageSize
sortBy
```

---

## Lineup API

```txt
GET  /api/lineups/my-current
GET  /api/lineups/team/{teamId}/round/{roundNumber}
POST /api/lineups
PUT  /api/lineups/{lineupId}
POST /api/lineups/{lineupId}/captain
```

Example save lineup request:

```json
{
  "teamId": 101,
  "roundNumber": 5,
  "captainPlayerId": 201,
  "players": [
    {
      "playerId": 201,
      "position": "Forward"
    },
    {
      "playerId": 202,
      "position": "Goalkeeper"
    }
  ]
}
```

---

## Round API

```txt
GET /api/rounds
GET /api/rounds/current
GET /api/rounds/{roundNumber}
GET /api/rounds/{roundNumber}/summary
GET /api/rounds/{roundNumber}/players
GET /api/rounds/{roundNumber}/teams
```

---

## Matchup API

```txt
GET /api/matchups/my-current
GET /api/matchups/league/{leagueId}/round/{roundNumber}
GET /api/matchups/{matchupId}
```

---

## Ranking API

```txt
GET /api/rankings/championship/{championshipId}
GET /api/rankings/league/{leagueId}
GET /api/rankings/league/{leagueId}/round/{roundNumber}
```

---

## Dashboard API

Dashboard endpoints are optimized for read-heavy user screens.

```txt
GET /api/dashboard/home
GET /api/dashboard/round-summary
GET /api/dashboard/my-team
GET /api/dashboard/my-leagues
GET /api/dashboard/my-current-matchup
```

Benefits:

- Provide home screen summary.
- Reduce multiple frontend calls.
- Return only the data needed by the screen.
- Avoid overfetching.
- Improve perceived performance.
- Support mobile-first user experience.

---

## Achievement API

```txt
GET /api/achievements
GET /api/achievements/my-team
GET /api/achievements/team/{teamId}
```

---

## Admin API

Admin endpoints should be protected by authentication and authorization.

```txt
/api/admin/championships
/api/admin/players
/api/admin/real-teams
/api/admin/real-matches
/api/admin/real-match-events
/api/admin/rounds
/api/admin/scoring
/api/admin/achievements
```

---

## API Response Patterns

### Success Response

```json
{
  "success": true,
  "data": {
    "id": 101,
    "name": "Demo Warriors"
  }
}
```

### Error Response

```json
{
  "success": false,
  "error": {
    "code": "LINEUP_VALIDATION_FAILED",
    "message": "The selected lineup does not match the required formation."
  }
}
```

### Paginated Response

```json
{
  "items": [],
  "page": 1,
  "pageSize": 20,
  "totalItems": 100,
  "totalPages": 5
}
```

---

## DTO Strategy

DTOs avoid exposing internal database entities directly.

Benefits:

- Clear API contracts.
- Better frontend integration.
- Reduced overfetching.
- Improved security.
- Easier versioning.
- Better control over response size.

Example DTOs:

```txt
TeamSummaryDto
PlayerCardDto
LeagueRankingRowDto
RoundSummaryDto
MatchupDto
HomeDashboardDto
AchievementDto
```

---

## Validation Strategy

| Validation | Reason |
|---|---|
| User must be authenticated to create a fantasy team. | Protects private user data. |
| User cannot edit lineup after the round closes. | Preserves fairness. |
| Lineup must follow formation rules. | Protects competition rules. |
| Captain must be part of the selected lineup. | Prevents invalid score calculation. |
| Player must belong to the current championship. | Prevents cross-competition issues. |
| Admin-only endpoints require admin permissions. | Protects sensitive workflows. |

---

## Performance Considerations

Recommended practices:

- Use DTO projections.
- Use pagination for large collections.
- Use `AsNoTracking` for read-only queries.
- Avoid returning full entity graphs.
- Avoid unnecessary joins.
- Use indexes for frequent filters.
- Use snapshot tables for closed rounds.
- Cache stable data when appropriate.
- Separate write operations from read-optimized endpoints.

---

## Security Considerations

Security practices include:

- Token-based authentication.
- Authorization for admin endpoints.
- Environment variables for secrets.
- No credentials in source code.
- No real user data in public examples.
- No production tokens in screenshots.
- Input validation.
- Protected write operations.
- Safe error messages.

---

## Public Case Study Scope

This document intentionally provides a high-level API overview. It demonstrates REST API design, backend architecture thinking, DTO strategy, business rule validation, performance awareness, security awareness and commercial product maturity.

It does not expose production source code, full implementation details, sensitive algorithms, real tokens, real user data, complete endpoint contracts or private business logic.

---

## Summary

The 4Fantasy API is designed as a REST-based backend layer that connects the Angular frontend with the platform's fantasy sports business rules and PostgreSQL database.

It supports user authentication, fantasy team management, leagues, players, lineups, rounds, matchups, rankings, achievements and admin workflows.

The API architecture emphasizes clear contracts, DTO-based responses, thin controllers, application services, validation, security and performance-oriented read endpoints.
