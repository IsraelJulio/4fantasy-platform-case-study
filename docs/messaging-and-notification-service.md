# Messaging and Notification Service

<img width="628" height="442" alt="image" src="https://github.com/user-attachments/assets/8cd0c19b-42db-4298-baf7-2bbde1acd0dd" />


## Introduction

4Fantasy includes an internal messaging and notification service designed to communicate with users in a flexible, reliable and scalable way.

The platform supports sending messages to:

- All users
- Specific users
- Users from a specific competition context
- Users related to a specific team, league, round or event
- Targeted groups based on business rules

This feature was designed to support both operational communication and user engagement inside the platform.

The production implementation is private because the project is intended for commercial use. This document describes the high-level design and technical approach without exposing private source code, credentials, notification tokens or sensitive implementation details.

---

## Messaging Goals

The messaging and notification service was designed with the following goals:

- Allow administrators to send announcements to all users or selected users.
- Allow the system to send automated messages based on platform events.
- Keep an internal history of messages for each user.
- Support unread message tracking.
- Support push notifications through an external provider.
- Process notification delivery asynchronously.
- Avoid slowing down user-facing requests.
- Keep push notification credentials secure on the backend.
- Support future real-time updates in the Angular UI.
- Improve user engagement and communication inside the platform.

---

## Main Use Cases

The messaging service can be used for scenarios such as:

| Use Case | Description |
|---|---|
| Round updates | Notify users when a round opens, closes or is calculated. |
| Lineup reminders | Remind users to select or update their lineup before the round closes. |
| Matchup results | Notify users about wins, losses or draws in direct confrontations. |
| Achievement unlocks | Inform users when they unlock a new achievement or badge. |
| League updates | Send updates related to league rankings or participation. |
| Admin announcements | Allow administrators to send messages to all users or selected users. |
| Team-related updates | Notify users about events connected to their fantasy team or supported real team. |
| System alerts | Communicate important platform updates or maintenance notices. |

---

## High-Level Messaging Flow

```txt
Admin or System Event
 |
 v
Message Creation
 |
 v
Recipient Selection
 |
 v
Internal Message Storage
 |
 v
Background Processing
 |
 v
Push Notification Dispatch
 |
 v
User Notification Center
```

This flow separates message creation from message delivery, making the system more reliable and easier to maintain.

---

## Technical Design

The messaging service is designed with a clear separation between internal messages and external push notifications.

| Component | Responsibility |
|---|---|
| Internal Message Entity | Stores the message content, title, type and metadata. |
| Message Recipient Entity | Stores which users should receive each message. |
| Application Service | Coordinates message creation and recipient selection. |
| Background Job | Processes message delivery asynchronously. |
| Push Notification Provider | Sends push notifications to external devices. |
| Angular Notification UI | Displays messages, unread counters and notification history. |

This separation allows the platform to keep a reliable internal record of messages while also supporting external notification channels.

---

## Technologies Used

| Technology | Usage |
|---|---|
| ASP.NET Core | Backend API and messaging endpoints. |
| C# | Business logic and message processing. |
| Entity Framework Core | Persistence of messages and recipients. |
| PostgreSQL | Storage for internal messages, recipients and delivery status. |
| Angular | Notification center and user interface. |
| TypeScript | Frontend logic and typed API communication. |
| OneSignal | Push notification delivery to users' devices. |
| Hangfire | Background processing for asynchronous notification dispatch. |
| REST APIs | Communication between frontend and backend. |
| SignalR | Planned real-time updates for unread counters and instant notification feedback. |

---

## Internal Messaging Model

The messaging structure can be represented as:

```txt
Message
 |
 ├── Title
 ├── Content
 ├── Type
 ├── CreatedAt
 ├── CreatedBy
 └── Recipients
      |
      ├── UserId
      ├── IsRead
      ├── ReadAt
      ├── DeliveryStatus
      └── SentAt
```

This model allows the platform to track:

- Who received the message
- Whether the message was read
- When the message was sent
- Delivery status
- User-specific notification history

---

## Suggested Entities

The implementation can be represented using entities similar to the following:

```txt
InternalMessage
- Id
- Title
- Content
- MessageType
- CreatedAt
- CreatedByUserId
- Metadata
- IsSystemGenerated
```

```txt
InternalMessageRecipient
- Id
- MessageId
- UserId
- IsRead
- ReadAt
- DeliveryStatus
- SentAt
- ErrorMessage
```

```txt
PushNotificationLog
- Id
- MessageId
- UserId
- Provider
- ExternalNotificationId
- Status
- SentAt
- ErrorMessage
```

These entities help separate the message itself, the recipients and the external push delivery tracking.

---

## Recipient Targeting Strategy

Messages can be delivered using different recipient strategies.

```txt
All Users
Specific Users
Users by League
Users by Championship
Users by Fantasy Team
Users by Business Rule
```

This makes the system flexible enough to support both manual admin announcements and automated system-generated notifications.

Examples:

| Targeting Strategy | Example |
|---|---|
| All Users | Send a platform announcement. |
| Specific Users | Send a message to selected users. |
| Users by League | Notify all members of a specific league. |
| Users by Championship | Notify all users participating in a championship. |
| Users by Fantasy Team | Notify users related to a specific fantasy team. |
| Business Rule | Notify users who forgot to set their lineup. |

---

## Background Processing

Notification delivery can be processed asynchronously to avoid slowing down user-facing requests.

Example flow:

```txt
Message Created
 |
 v
Recipients Generated
 |
 v
Background Job Queued
 |
 v
Push Notifications Sent
 |
 v
Delivery Status Updated
```

Using background jobs improves reliability and keeps the API responsive, especially when a message needs to be sent to many users.

### Why Hangfire Fits This Use Case

Hangfire can be used to:

- Queue notification jobs.
- Retry failed deliveries.
- Process messages outside the HTTP request lifecycle.
- Track background job execution.
- Improve reliability for bulk message sending.
- Avoid blocking admin actions while notifications are being sent.

---

## Push Notification Strategy

OneSignal is used as the external push notification provider.

The backend is responsible for communicating with OneSignal. The frontend should never expose OneSignal private credentials.

Recommended flow:

```txt
Internal Message Created
 |
 v
Backend Determines Recipients
 |
 v
Backend Sends Push Request to OneSignal
 |
 v
OneSignal Delivers Notification
 |
 v
Delivery Result Stored
```

Benefits:

- Push logic remains centralized in the backend.
- Credentials are protected.
- Delivery can be logged.
- Failed deliveries can be retried.
- The internal message history remains independent from the push provider.

---

## Real-Time Notification Strategy

The platform can use real-time communication to improve the user experience.

A planned real-time flow may include:

```txt
New Message Created
 |
 v
Unread Counter Updated
 |
 v
Angular Topbar Notification Badge Updated
 |
 v
User Opens Notification Center
 |
 v
Message Marked as Read
```

SignalR can be used in the future to update unread counters and notification badges without requiring a page refresh.

---

## Angular Notification UI

The Angular frontend can include a notification area responsible for displaying user messages.

Possible UI elements:

| UI Element | Purpose |
|---|---|
| Topbar notification icon | Shows unread message count. |
| Notification dropdown | Shows recent messages. |
| Notification center page | Shows full message history. |
| Read/unread status | Helps users identify new messages. |
| Toast feedback | Displays immediate alerts for important events. |
| Empty state | Shows a friendly message when there are no notifications. |

Suggested Angular structure:

```txt
features/notifications/
 |
 ├── pages/
 |    └── notification-center/
 |
 ├── components/
 |    ├── notification-bell/
 |    ├── notification-list/
 |    ├── notification-item/
 |    └── unread-badge/
 |
 └── services/
      └── notification.service.ts
```

---

## Suggested API Endpoints

Possible API endpoints:

```txt
GET  /api/notifications/my
GET  /api/notifications/my/unread-count
PUT  /api/notifications/{messageId}/read
PUT  /api/notifications/read-all
POST /api/admin/messages
POST /api/admin/messages/send-to-users
POST /api/admin/messages/send-to-all
```

Admin endpoints should be protected by authorization rules.

---

## Example Message Payload

```json
{
  "title": "Round 5 is now open",
  "content": "You can now update your lineup for the next round.",
  "messageType": "RoundUpdate",
  "targetType": "ChampionshipUsers",
  "targetId": 1
}
```

---

## Example User Notification Response

```json
{
  "items": [
    {
      "id": 1001,
      "title": "Achievement unlocked",
      "content": "Your team unlocked the Captain Star achievement.",
      "messageType": "Achievement",
      "isRead": false,
      "createdAt": "2026-05-01T12:00:00Z"
    }
  ],
  "unreadCount": 1
}
```

---

## Delivery Status

Delivery status can be tracked for each recipient.

Example statuses:

```txt
Pending
Queued
Sent
Failed
Read
```

Tracking delivery status helps with:

- Debugging failed notifications
- Understanding message reach
- Supporting retries
- Improving admin visibility
- Keeping message history reliable

---

## Why This Feature Matters

The messaging service is important because fantasy sports platforms depend on timely communication.

Users need to know when:

- A round is open
- A lineup deadline is close
- A matchup result is available
- A ranking changed
- A new achievement was unlocked
- An admin announcement was published

From an engineering perspective, this feature demonstrates:

- Backend business workflow design
- Asynchronous processing
- Push notification integration
- User targeting logic
- Database modeling for message delivery
- Frontend notification UX
- Separation between internal messages and external delivery channels
- Product thinking focused on user engagement

---

## Privacy and Safety Considerations

The public case study does not expose real user data, notification tokens or production credentials.

The messaging service should follow these principles:

- Never expose OneSignal credentials publicly.
- Never store secrets in frontend code.
- Use backend services to trigger push notifications.
- Store only the necessary delivery metadata.
- Avoid exposing private user data in notifications.
- Allow future support for notification preferences.
- Keep admin messaging protected by authorization rules.
- Use mock data in all public documentation and screenshots.

---

## Public Case Study Scope

This document intentionally provides a high-level overview of the messaging and notification service.

It is designed to demonstrate:

- Messaging architecture
- User targeting strategy
- Push notification integration
- Background job processing
- Frontend notification UX
- Security and privacy awareness
- Product-oriented communication design

It does not expose:

- Production source code
- OneSignal private keys
- Real notification tokens
- Real user data
- Internal credentials
- Complete implementation details
- Sensitive commercial rules

---

## Summary

The 4Fantasy messaging and notification service allows the platform to communicate with all users, selected users or targeted groups based on business rules.

The architecture separates internal message storage from external push notification delivery, using ASP.NET Core, Entity Framework Core, PostgreSQL, Hangfire, OneSignal, Angular and TypeScript.

This feature improves user engagement, supports admin communication and demonstrates important full stack engineering concepts such as asynchronous processing, push integration, user targeting, notification history and real-time UI planning.
