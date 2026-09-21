---
name: eventhub-domain
description: EventHub application domain knowledge — business rules, API endpoints, data models, user flows, UI selectors, and error scenarios. Use when writing tests, reviewing code, creating scenarios, or answering questions about how EventHub works.
user-invocable: false
---

# EventHub Domain Knowledge

## Overview
EventHub is a full-stack event ticket booking platform. Users browse events, book tickets, and manage bookings. Each user operates in an isolated sandbox with limits on events and bookings.

## Tech Stack
- **Frontend**: Next.js 14 (App Router), React 18, TypeScript, Tailwind CSS, React Query v5
- **Backend**: Express.js 4.21, Prisma ORM 5.22, MySQL 8+
- **Auth**: JWT (7-day expiry), bcryptjs password hashing
- **Testing**: Playwright (E2E), Chromium only
- **API Docs**: Swagger UI at `/api/docs`

## Architecture
```
frontend/ (Next.js 14)           backend/ (Express.js)
├── app/                         ├── src/
│   ├── page.tsx (Home)          │   ├── routes/       (HTTP layer)
│   ├── login/                   │   ├── controllers/  (Request handling)
│   ├── register/                │   ├── services/     (Business logic)
│   ├── events/                  │   ├── repositories/ (Data access)
│   │   ├── page.tsx (List)      │   ├── validators/   (Input validation)
│   │   └── [id]/page.tsx        │   ├── middleware/    (Auth, errors)
│   ├── bookings/                │   └── utils/        (Error classes)
│   │   ├── page.tsx (List)      ├── prisma/
│   │   └── [id]/page.tsx        │   ├── schema.prisma
│   └── admin/                   │   └── seed.js
│       ├── events/page.tsx      └── app.js
│       └── bookings/page.tsx
├── components/
├── lib/
│   ├── api/ (client, endpoints)
│   └── hooks/ (useAuth, useEvents, useBookings)
└── types/
```

## Data Models

### User
| Field | Type | Notes |
|---|---|---|
| id | Int (PK) | Auto-increment |
| email | String | Unique |
| password | String | bcrypt hashed |
| events | Event[] | User's created events |
| bookings | Booking[] | User's bookings |

### Event
| Field | Type | Notes |
|---|---|---|
| id | Int (PK) | Auto-increment |
| title | String | Required |
| description | Text | Optional |
| category | String | Conference/Concert/Sports/Workshop/Festival |
| venue | String | Required |
| city | String | Bangalore/Mumbai/Hyderabad/Delhi/Chennai |
| eventDate | DateTime | Must be future date |
| price | Decimal | Per ticket, >= 0 |
| totalSeats | Int | >= 1 |
| availableSeats | Int | Dynamic for user events |
| isStatic | Boolean | true = seeded event, immutable |
| userId | Int? | null for static events |

### Booking
| Field | Type | Notes |
|---|---|---|
| id | Int (PK) | Auto-increment |
| eventId | Int (FK) | Cascade delete with Event |
| userId | Int (FK) | Cascade delete with User |
| customerName | String | Min 2 chars |
| customerEmail | String | Valid email format |
| customerPhone | String | Min 10 digits |
| quantity | Int | 1-10 tickets |
| totalPrice | Decimal | price x quantity |
| status | String | Always "confirmed" |
| bookingRef | String | Unique, format: `[FIRST_LETTER]-[6_RANDOM]` |

## Detailed Knowledge (Sub-Files)

Load these based on what the current task needs:

- **Business rules & validation logic** → read `./business-rules.md`
- **API endpoints & error codes** → read `./api-reference.md`
- **UI selectors for test automation** → read `./ui-selectors.md`
- **User flows, test scenarios & test data** → read `./user-flows.md`

# EventHub API Reference

## Authentication (No Auth Required)
| Method | Endpoint          | Body                    | Response                          |
|--------|-------------------|-------------------------|-----------------------------------|
| POST   | /api/auth/register| { email, password }     | { token, user: { id, email } }    |
| POST   | /api/auth/login   | { email, password }     | { token, user: { id, email } }    |
| GET    | /api/auth/me      | -                       | { user: { userId, email } }       |

## Events (Bearer Token Required)
| Method | Endpoint        | Query/Body                         | Response                     |
|--------|-----------------|------------------------------------|------------------------------|
| GET    | /api/events     | ?search, category, city, page, limit | { data: Event[], pagination }|
| GET    | /api/events/:id | -                                  | { data: Event }              |
| POST   | /api/events     | CreateEventInput                   | { data: Event }              |
| PUT    | /api/events/:id | UpdateEventInput                   | { data: Event }              |
| DELETE | /api/events/:id | -                                  | { message }                  |

## Bookings (Bearer Token Required)
| Method | Endpoint              | Query/Body                    | Response                      |
|--------|-----------------------|-------------------------------|-------------------------------|
| GET    | /api/bookings         | ?eventId, status, page, limit | { data: Booking[], pagination}|
| GET    | /api/bookings/:id     | -                             | { data: Booking }             |
| GET    | /api/bookings/ref/:ref| -                             | { data: Booking }             |
| POST   | /api/bookings         | CreateBookingInput            | { data: Booking }             |
| DELETE | /api/bookings         | -                             | Clear all user bookings       |
| DELETE | /api/bookings/:id     | -                             | Cancel single booking         |

## Error Scenarios
| Scenario                  | HTTP Code | Message                              |
|---------------------------|-----------|--------------------------------------|
| Invalid login credentials | 401       | Invalid credentials                  |
| Duplicate email register  | 409       | Email already registered             |
| Missing auth token        | 401       | Unauthorized                         |
| Cross-user booking access | 403       | Forbidden / Access Denied            |
| Edit static event         | 403       | Cannot modify static events          |
| Insufficient seats        | 400       | Insufficient seats available         |
| Invalid event date (past) | 400       | Event date must be in the future     |
| Missing required fields   | 400       | Validation error details             |

