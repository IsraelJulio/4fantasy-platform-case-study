# Frontend Structure

## Introduction

4Fantasy uses an Angular frontend built with TypeScript to provide a responsive and interactive fantasy sports experience for users and administrators.

The frontend is responsible for presenting fantasy teams, player selection, leagues, rankings, matchups, round summaries, achievements and admin workflows in a clear and mobile-friendly interface.

The production source code is private because the project is intended for commercial use. This document provides a public high-level overview of the frontend structure, organization principles, feature modules, UI responsibilities and architectural decisions without exposing private implementation details.

---

## Frontend Goals

- Provide a responsive experience for desktop and mobile users.
- Organize the application by business features.
- Keep components reusable and maintainable.
- Communicate with the backend through typed Angular services.
- Separate regular user features from admin workflows.
- Support loading states and good perceived performance.
- Keep UI logic separated from API communication.
- Provide a polished fantasy sports product experience.
- Support future evolution into a commercial platform.

---

## Technology Stack

| Area | Technology |
|---|---|
| Framework | Angular |
| Language | TypeScript |
| UI Library | PrimeNG |
| Styling | SCSS |
| API Communication | Angular HttpClient |
| Routing | Angular Router |
| Forms | Reactive Forms / Template Forms |
| State Handling | Component state and shared services |
| Build / Hosting | Angular build, Vercel or similar frontend hosting |

---

## High-Level Frontend Architecture

```txt
Angular Application
 |
 ├── Core
 |    ├── Services
 |    ├── Guards
 |    ├── Interceptors
 |    └── Models
 |
 ├── Shared
 |    ├── Components
 |    ├── Pipes
 |    ├── Directives
 |    └── Utilities
 |
 ├── Features
 |    ├── Auth
 |    ├── Home
 |    ├── Team
 |    ├── Lineup
 |    ├── League
 |    ├── Ranking
 |    ├── Round Summary
 |    ├── Matchups
 |    ├── Achievements
 |    └── Admin
 |
 └── Layout
      ├── Topbar
      ├── Sidebar
      ├── Shell
      └── Navigation
```

---

## Suggested Folder Structure

```txt
src/app/
 |
 ├── core/
 |    ├── auth/
 |    ├── guards/
 |    ├── interceptors/
 |    ├── models/
 |    ├── services/
 |    └── constants/
 |
 ├── shared/
 |    ├── components/
 |    ├── pipes/
 |    ├── directives/
 |    ├── utils/
 |    └── types/
 |
 ├── features/
 |    ├── auth/
 |    ├── home/
 |    ├── team/
 |    ├── lineup/
 |    ├── league/
 |    ├── ranking/
 |    ├── round-summary/
 |    ├── matchups/
 |    ├── achievements/
 |    └── admin/
 |
 ├── layout/
 |    ├── app-shell/
 |    ├── topbar/
 |    ├── sidebar/
 |    └── footer/
 |
 ├── assets/
 |
 ├── environments/
 |
 └── app-routing.module.ts
```

---

## Core Module

The `core` area contains services and utilities that are used globally across the application.

Typical responsibilities:

- Authentication service.
- User session handling.
- API configuration.
- HTTP interceptors.
- Route guards.
- Global models.
- Error handling helpers.
- Application constants.

Example structure:

```txt
core/
 |
 ├── auth/
 |    ├── auth.service.ts
 |    ├── token.service.ts
 |    └── user-session.service.ts
 |
 ├── guards/
 |    ├── auth.guard.ts
 |    └── admin.guard.ts
 |
 ├── interceptors/
 |    ├── auth.interceptor.ts
 |    └── error.interceptor.ts
 |
 ├── services/
 |    ├── api.service.ts
 |    ├── notification.service.ts
 |    └── loading.service.ts
 |
 └── models/
      ├── user.model.ts
      └── api-response.model.ts
```

---

## Authentication Flow

```txt
Login Form
 |
 v
AuthService
 |
 v
API Authentication Endpoint
 |
 v
Token Storage
 |
 v
Auth Interceptor
 |
 v
Protected API Calls
```

| Part | Responsibility |
|---|---|
| AuthService | Handles login, logout and current user retrieval. |
| TokenService | Stores and retrieves access tokens. |
| AuthGuard | Protects authenticated routes. |
| AdminGuard | Protects admin routes. |
| AuthInterceptor | Adds token to HTTP requests. |
| ErrorInterceptor | Handles unauthorized responses and global API errors. |

---

## Shared Module

Typical shared components:

```txt
shared/components/
 |
 ├── player-card/
 ├── team-card/
 ├── ranking-table/
 ├── loading-skeleton/
 ├── empty-state/
 ├── confirmation-dialog/
 ├── score-badge/
 ├── achievement-card/
 └── responsive-table/
```

Shared components should be generic enough to be reused by multiple features.

---

## Feature Modules

Feature folders represent business areas of the platform.

Suggested pattern:

```txt
features/feature-name/
 |
 ├── pages/
 ├── components/
 ├── services/
 ├── models/
 └── feature-routing.module.ts
```

---

## Auth Feature

Responsible for login, registration and account access.

Main responsibilities:

- Login screen.
- Register screen.
- Form validation.
- Authentication feedback.
- Redirect after login.

---

## Home Feature

Responsible for the user dashboard.

Example structure:

```txt
features/home/
 |
 ├── pages/
 |    └── home-dashboard/
 |
 ├── components/
 |    ├── current-round-card/
 |    ├── my-team-summary/
 |    ├── current-matchup-card/
 |    ├── league-position-card/
 |    └── recent-achievements/
 |
 └── services/
      └── home-dashboard.service.ts
```

The home screen should consume a read-optimized endpoint:

```txt
GET /api/dashboard/home
```

---

## Team Feature

Main responsibilities:

- Show fantasy team profile.
- Show team score.
- Show historical performance.
- Show achievements.
- Allow team customization where applicable.

---

## Lineup Feature

Responsible for player selection and round lineup.

Main responsibilities:

- Select players.
- Filter players by position.
- Show selected lineup.
- Select captain.
- Validate formation rules.
- Save lineup before round closes.
- Prevent editing after round closure.

---

## League Feature

Main responsibilities:

- List public leagues.
- Create league.
- Join league.
- Show league details.
- Show league members.
- Navigate to rankings and matchups.

---

## Ranking Feature

Main responsibilities:

- Display team positions.
- Show wins, draws, losses and scores.
- Support round filters.
- Support league-specific ranking.
- Highlight the current user's team.
- Use pagination or virtual scrolling for large rankings.

---

## Round Summary Feature

Main responsibilities:

- Show current or historical round summary.
- Display best player.
- Display highest team score.
- Display average score.
- Show player and team performance.
- Consume snapshot data when available.

---

## Matchups Feature

Main responsibilities:

- Show current user's matchup.
- Show league matchups.
- Compare scores.
- Show winner or draw.
- Display historical matchup results.

---

## Achievements Feature

Main responsibilities:

- Show available achievements.
- Show unlocked achievements.
- Highlight recent achievements.
- Improve user engagement.

---

## Admin Feature

The admin area is separated from regular user features.

Example structure:

```txt
features/admin/
 |
 ├── pages/
 |    ├── admin-dashboard/
 |    ├── championship-admin/
 |    ├── player-admin/
 |    ├── real-team-admin/
 |    ├── real-match-admin/
 |    ├── round-admin/
 |    └── scoring-admin/
 |
 ├── components/
 |    ├── admin-table/
 |    ├── admin-form/
 |    ├── match-event-editor/
 |    ├── scoring-review-panel/
 |    └── round-status-panel/
 |
 └── services/
      ├── admin-player.service.ts
      ├── admin-match.service.ts
      └── admin-round.service.ts
```

Admin routes should be protected by an admin guard.

---

## Layout Structure

Responsibilities:

- Provide consistent page structure.
- Display navigation.
- Display user session information.
- Display notifications or unread messages.
- Support desktop and mobile navigation.
- Keep feature pages focused on business content.

---

## Routing Strategy

Example routes:

```txt
/login
/register
/home
/my-team
/lineup
/leagues
/leagues/:id
/leagues/:id/ranking
/rounds/:roundNumber/summary
/matchups/my-current
/achievements
/admin
/admin/players
/admin/real-matches
/admin/rounds
```

| Route Type | Guard |
|---|---|
| Public routes | No guard |
| User routes | AuthGuard |
| Admin routes | AdminGuard |
| Unknown routes | NotFound page |

---

## Service Layer

Services should:

- Call backend endpoints.
- Return typed observables.
- Keep API URLs centralized.
- Avoid duplicating HTTP logic.
- Keep components focused on UI behavior.

```txt
Component
 |
 | handles UI state and user actions
 v
Service
 |
 | handles API communication
 v
Backend API
```

---

## UI State Strategy

Typical state pattern:

```txt
isLoading
data
errorMessage
emptyState
```

| State | UI Behavior |
|---|---|
| Loading | Show skeleton or spinner. |
| Success with data | Render the content. |
| Success with empty data | Show empty state message. |
| Error | Show friendly error message and retry option. |
| Unauthorized | Redirect to login. |
| Forbidden | Show access denied message. |

---

## PrimeNG Usage

| PrimeNG Component | Usage |
|---|---|
| Table | Rankings, admin lists, player lists. |
| Card | Team cards, player cards, achievement cards. |
| Dialog | Confirmations and forms. |
| Dropdown | Filters and selections. |
| Toast | Success and error feedback. |
| Toolbar | Admin and page actions. |
| TabView | Grouped content. |
| Avatar | Player and team images. |
| Badge | Scores, status and notifications. |
| Skeleton | Loading states. |
| Chart | Score and performance visualization. |

---

## Frontend Performance Considerations

- Use read-optimized endpoints.
- Avoid repeated API calls for the same screen.
- Cache stable reference data in services when appropriate.
- Use lazy-loaded modules for larger feature areas.
- Avoid rendering very large lists without pagination.
- Use trackBy in repeated lists.
- Optimize images.
- Avoid heavy calculations inside templates.
- Keep components focused and lightweight.

---

## Public Case Study Scope

This document intentionally provides a high-level frontend structure. It demonstrates Angular architecture thinking, feature-based organization, UI/UX responsibility separation, API integration strategy, mobile-first product thinking, admin/user workflow separation, maintainability and scalability.

It does not expose production source code, private component implementation, real user data, sensitive business rules, internal commercial assets or private environment configuration.

---

## Summary

The 4Fantasy frontend is structured as a modular Angular application focused on feature organization, responsive UI, API integration, reusable components and maintainability.

The frontend supports user-facing fantasy sports workflows such as team management, lineup selection, league rankings, round summaries, matchups and achievements, while also providing separated admin workflows for competition management.
