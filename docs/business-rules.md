# Business Rules

## Introduction

4Fantasy is a fantasy sports platform based on real-world competitions, fantasy teams, leagues, rounds, player selection, scoring, matchups, rankings and achievements.

This document describes the main business rules of the platform at a high level. The goal is to demonstrate the complexity of the domain and the reasoning behind the product without exposing sensitive commercial formulas or private implementation details.

The production source code and complete business logic are private because the project is intended for commercial use.

---

## Business Rule Goals

- Keep fantasy competitions fair and consistent.
- Preserve historical round results.
- Connect real sports events to fantasy scoring.
- Support league-specific rankings and matchups.
- Prevent invalid lineup changes after round closure.
- Provide a clear scoring experience for users.
- Support gamification through achievements.
- Allow administrators to manage competitions safely.
- Keep sensitive formulas abstracted in public documentation.

---

## Domain Context

The platform connects two main worlds:

| Context | Description |
|---|---|
| Real Sports Context | Real teams, real players, real matches and match events. |
| Fantasy Context | Fantasy teams, user lineups, leagues, matchups, scores and rankings. |

```txt
Real Match Event
 |
 v
Player Round Score
 |
 v
Fantasy Team Score
 |
 v
Matchup Result
 |
 v
League Ranking
```

---

## Main Business Concepts

| Concept | Description |
|---|---|
| Championship | A competition season or tournament. |
| Round | A specific stage of the championship. |
| Real Team | A real-world sports team. |
| Player | A real athlete available for fantasy selection. |
| Fantasy Team | A user-created team in the fantasy platform. |
| League | A group of fantasy teams competing together. |
| Lineup | The selected players for a fantasy team in a round. |
| Captain | A selected player who may receive a score multiplier. |
| Matchup | A direct confrontation between two fantasy teams. |
| Ranking | The ordered classification of teams in a league or championship. |
| Achievement | A badge or award unlocked based on performance. |

---

## Championship Rules

### Championship Creation

- A championship can contain multiple rounds.
- A championship can contain multiple leagues.
- A championship can contain real teams and players.
- A championship can have one active/current round.
- A championship can be active or inactive.
- Fantasy teams belong to a specific championship.
- Rankings and scores should not be mixed between different championships.

### Current Round Rule

Each active championship should have a current round.

- The current round defines which lineup can be edited.
- The current round defines which real matches are relevant.
- The current round controls whether the market is open or closed.
- Historical rounds should remain available for read-only visualization.

---

## Round Rules

### Round Status

```txt
Open
In Progress
Closed
Calculated
```

| Status | Meaning |
|---|---|
| Open | Users can edit their lineups. |
| In Progress | Real matches may be happening and scoring can change. |
| Closed | Users can no longer edit lineups. |
| Calculated | Final scores, rankings and snapshots have been generated. |

### Lineup Lock Rule

- If the round is open, users may select or replace players.
- If the round is closed, users cannot change the lineup.
- If the round is in progress, lineup changes should be blocked.
- Historical lineups must remain unchanged after closure.

### Historical Round Rule

After a round is closed and calculated:

- Final player scores should be preserved.
- Final fantasy team scores should be preserved.
- Final matchup results should be preserved.
- Final league rankings should be preserved.
- Historical data should be read-only for regular users.

---

## Fantasy Team Rules

### Fantasy Team Creation

- A user can create a fantasy team for a championship.
- A fantasy team must have a name.
- A fantasy team can have a shield or logo.
- A fantasy team belongs to one user.
- A fantasy team belongs to one championship.
- A fantasy team may participate in one or more leagues.

### Fantasy Team Uniqueness

A user should not have duplicated fantasy teams inside the same championship.

### Fantasy Team Participation in Leagues

- A fantasy team can join public leagues.
- A fantasy team can join private leagues when allowed.
- A fantasy team can participate in league-specific rankings.
- A fantasy team can participate in league-specific matchups.
- Leaving or joining a league should respect championship and round rules.

---

## League Rules

### League Creation

- A league belongs to a championship.
- A league can be public or private.
- A league contains fantasy teams.
- A league has its own ranking.
- A league may have its own matchup schedule.

### League Membership

- Only fantasy teams from the same championship can join a league.
- A fantasy team should not be duplicated in the same league.
- League membership should be used when calculating league rankings and matchups.

### League Ranking

Possible criteria:

```txt
- Wins
- Draws
- Losses
- Total fantasy score
- Round score
- Tie-breaker rules
```

Example order:

```txt
1. Most wins
2. Highest total score
3. Highest round score
4. Additional tie-breaker
```

---

## Player Rules

### Player Availability

- Players belong to a real team.
- Players belong to a championship.
- Players can be active or inactive.
- Only active players should be available for lineup selection.
- Player position should be used to validate formations.

Example positions:

```txt
Goalkeeper
Defender
Winger
Pivot
Coach
```

### Player Price and Market Value

- A player can have a fantasy price.
- The platform may use price to control fantasy team budget.
- Player price may change based on performance.
- Price rules should be consistent and transparent to users.

Public documentation should not expose private commercial pricing formulas in full detail.

---

## Lineup Rules

### Lineup Creation

- A fantasy team can have one lineup per round.
- A lineup belongs to a fantasy team.
- A lineup belongs to a round.
- A lineup can include selected players.
- A lineup can include one captain.
- A lineup must follow formation rules.

### Formation Validation

Example formation:

```txt
1 Coach
1 Goalkeeper
2 Wingers
1 Pivot
```

Rules:

- Required positions must be filled.
- The same player cannot be selected twice.
- Selected players must be active.
- Selected players must belong to the same championship.
- Selected players must be available before the round closes.

### Captain Rule

- The captain must be part of the selected lineup.
- Only one player can be captain.
- The captain may receive a score multiplier.
- Captain scoring should be applied consistently.
- Captain selection should be locked when the round closes.

Example:

```txt
Player Score: 10.0
Captain Multiplier: 2x
Captain Final Score: 20.0
```

---

## Scoring Rules

### Player Score Calculation

Player scores are based on real match events.

Examples:

```txt
Goal
Assist
Save
Yellow Card
Red Card
Own Goal
Participation
```

Rules:

- Each event can affect player score.
- Positive actions may increase score.
- Negative actions may decrease score.
- The score belongs to a player and a round.
- Scores should be recalculated when admin updates match events before final calculation.

### Fantasy Team Score Calculation

```txt
Selected Players
 |
 v
Player Round Scores
 |
 v
Captain Multiplier
 |
 v
Fantasy Team Round Score
```

Rules:

- Only selected lineup players count toward the fantasy team score.
- Captain multiplier is applied to the selected captain.
- Inactive or unselected players should not count.
- Final team score should be preserved after round calculation.
- Historical scores should not be unexpectedly modified.

---

## Matchup Rules

- A matchup belongs to a league.
- A matchup belongs to a round.
- A matchup includes two fantasy teams.
- Both teams must belong to the same league.
- Matchup results are based on round scores.
- The team with the highest round score wins.
- If both teams have the same score, the result is a draw.
- Matchup results can affect league ranking.
- Matchup results should be preserved after round calculation.

---

## Ranking Rules

Possible ranking factors:

| Factor | Description |
|---|---|
| Wins | Number of matchup victories. |
| Draws | Number of tied matchups. |
| Losses | Number of defeats. |
| Total Score | Accumulated fantasy score. |
| Round Score | Score in the current round. |
| Tie-breaker | Additional criteria when teams are tied. |

Example ranking priority:

```txt
1. Wins
2. Total Score
3. Round Score
4. Tie-breaker
```

---

## Achievement Rules

Examples:

```txt
Highest score of the round
Captain with high score
Team without negative players
Comeback victory
Best goalkeeper performance
Consistent scorer
```

Rules:

- An achievement belongs to a fantasy team or player.
- Achievements may be round-specific.
- The same achievement should not be duplicated incorrectly.
- Unlocking should occur after the relevant score calculation.
- Achievement history should be preserved.

---

## Admin Business Rules

### Admin Match Event Management

- Only authorized admins can manage real match events.
- Match events affect player scores.
- Updating events may require score recalculation.
- Admin actions should be logged.
- Finalized rounds should require extra care before changes.

### Round Closure

```txt
Admin closes round
 |
v
Lineups are locked
 |
v
Real match events are reviewed
 |
v
Player scores are calculated
 |
v
Fantasy team scores are calculated
 |
v
Matchup results are calculated
 |
v
Rankings are updated
 |
v
Snapshots are generated
```

### Recalculation Rule

- Recalculation should be admin-controlled.
- Recalculation should update affected player scores.
- Recalculation should update affected team scores.
- Recalculation should update affected matchups and rankings.
- Recalculation should regenerate affected snapshots.
- Recalculation actions should be logged.

---

## Notification Rules

Examples:

```txt
Round opened
Round closed
Lineup reminder
Matchup result
Achievement unlocked
Team victory
League ranking update
```

Rules:

- Notifications should be relevant to the user.
- Notifications should not expose private data from other users.
- Notification delivery may be processed asynchronously.
- Users may have notification preferences in future versions.

---

## Data Integrity Rules

| Rule | Reason |
|---|---|
| A fantasy team must belong to one user. | Maintains ownership. |
| A fantasy team must belong to one championship. | Prevents cross-season conflicts. |
| A league must belong to one championship. | Keeps rankings isolated. |
| A lineup must belong to one fantasy team and one round. | Preserves historical selections. |
| A player score must belong to one player and one round. | Preserves scoring consistency. |
| A matchup must belong to one league and one round. | Keeps confrontations organized. |
| A ranking snapshot must belong to a league and round. | Supports historical ranking. |
| Real user data must not be exposed publicly. | Protects privacy. |

---

## Business Rule Validation Examples

```txt
The selected lineup does not match the required formation.
The captain must be part of the selected lineup.
The round is already closed and the lineup cannot be changed.
The selected player is not available for this championship.
This fantasy team is already registered in this league.
Only administrators can update match events.
This round has already been calculated.
```

---

## Public Case Study Scope

This document intentionally describes the business rules at a high level. It demonstrates domain understanding, business rule modeling, fantasy sports logic, fairness and consistency concerns, historical data preservation, admin workflow design and commercial product thinking.

It does not expose full private scoring formulas, production source code, complete commercial rules, real user data, sensitive business strategies or internal implementation details.

---

## Summary

The 4Fantasy business rules connect real sports events to fantasy team performance through rounds, lineups, scoring, matchups, rankings and achievements.

The platform must preserve fairness, protect historical results, validate user actions, support admin workflows and provide a consistent fantasy sports experience.

These rules make 4Fantasy a domain-rich application with real architectural and product complexity, going far beyond a simple CRUD system.
