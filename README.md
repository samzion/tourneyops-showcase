# TourneyOps

**Sports tournament management platform — built for organisers who are done with spreadsheets.**

> A live, Nigerian-owned SaaS platform that replaces the manual coordination behind sports competitions with a single, rules-based system. Currently serving 100+ registered users across 5+ active organisations.

[![Status](https://img.shields.io/badge/Status-Live%20%26%20Active-brightgreen?style=flat-square)](https://tourneyops.com)
[![Platform](https://img.shields.io/badge/Platform-Web-blue?style=flat-square)](https://tourneyops.com)
[![Backend](https://img.shields.io/badge/Backend-Java%2017%20%7C%20Spring%20Boot%203.5-6DB33F?style=flat-square&logo=spring)](https://spring.io/projects/spring-boot)
[![Database](https://img.shields.io/badge/Database-PostgreSQL-4169E1?style=flat-square&logo=postgresql)](https://www.postgresql.org/)
[![Frontend](https://img.shields.io/badge/Frontend-Next.js-000000?style=flat-square&logo=nextdotjs)](https://nextjs.org/)
[![Auth](https://img.shields.io/badge/Auth-JWT%20%7C%20Spring%20Security-orange?style=flat-square)](https://spring.io/projects/spring-security)
[![Deployment](https://img.shields.io/badge/Deployment-Docker-2496ED?style=flat-square&logo=docker)](https://www.docker.com/)

---

## Live Platform

**[tourneyops.com](https://tourneyops.com)** — currently in active iteration based on organiser feedback.

> *This repository is a public engineering showcase. The source code is proprietary. All implementation details shared here represent high-level design decisions and are intended to demonstrate system design thinking, domain expertise, and engineering maturity.*

---

## Table of Contents

- [The Problem](#the-problem)
- [The Solution](#the-solution)
- [Who It Serves](#who-it-serves)
- [Key Features](#key-features)
- [System Architecture](#system-architecture)
- [Domain Model](#domain-model)
- [Tournament Format Engines](#tournament-format-engines)
- [API Design](#api-design)
- [Technology Stack](#technology-stack)
- [Engineering Decisions](#engineering-decisions)
- [Challenges & Lessons Learned](#challenges--lessons-learned)
- [Business Impact](#business-impact)
- [Roadmap](#roadmap)
- [Screenshots](#screenshots)
- [Contact](#contact)

---

## The Problem

Sports competition management in Nigeria — and across much of Africa — is still largely manual.

Tournament organisers coordinate registrations through Google Forms. Bracket draws happen in WhatsApp groups, often by hand. Scheduling conflicts are discovered the day of the match. Results are recorded in notebooks or Excel sheets. Standings are calculated manually and shared as screenshots.

This creates three compounding problems:

**Subjectivity** — draw decisions, scheduling choices, and eligibility calls are made by individuals, not enforced by rules. This creates disputes.

**Fragmentation** — registration, scheduling, results, and standings live in different tools. Organisers spend more time coordinating than managing.

**Invisibility** — players have no reliable way to see their own statistics, track their history across tournaments, or compare their performance over time.

TourneyOps was built to solve all three.

---

## The Solution

TourneyOps gives tournament organisers a single platform to manage the full competition lifecycle — from registration through to final standings — with every decision driven by pre-configured rules, not manual judgement.

At its core, TourneyOps is a **tournament orchestration engine**. It understands the operational logic of seven distinct competition formats and automates the decisions that previously required manual intervention: who plays whom, in what order, on which venue, with which umpire, and what happens when a match ends.

The platform is designed to be **sport-agnostic by architecture**. Football and Table Tennis are live. The domain model was deliberately built to accommodate additional sports without requiring structural changes.

---

## Who It Serves

TourneyOps is a multi-stakeholder platform. Six distinct user roles interact with the system, each with a different operational context:

| Role | Primary Responsibility | Key Platform Interactions |
|------|----------------------|--------------------------|
| **Organiser** | Runs the tournament end-to-end | Creates seasons, configures formats, manages registration, publishes results |
| **Player** | Participates as an individual | Registers for tournaments, views schedule, tracks statistics |
| **Team** | Participates as a group | Manages roster, views fixtures, tracks standings |
| **Coach** | Manages team preparation | Views team schedule, tracks squad statistics |
| **Umpire** | Officiates matches | Receives assignments, submits match events and scores |
| **Club** | Manages a sports organisation | Oversees multiple teams and participants |

The permission model is **role-additive** — a single user account can hold multiple roles simultaneously, reflecting real-world scenarios where a coach also registers as a player, or a club administrator also organises tournaments.

---

## Key Features

### Tournament Management
- Create and configure tournament seasons with flexible parameters: format, sport type, participation type (individual/team), registration rules, and scheduling constraints
- Support for both **street (casual)** and **formal** competition structures
- Open and close registration with configurable eligibility enforcement
- Publish seasons and control visibility to participants

### Seven Tournament Format Engines
The core differentiator. Each format has a dedicated orchestration engine that handles bracket generation, round progression, and edge cases specific to that format:

| Format | Description |
|--------|-------------|
| **League** | Round-robin fixture generation with configurable points system and tiebreaker rules |
| **Knockout** | Single-elimination bracket with seeding support and bye handling |
| **Group-to-Knockout** | Group stage feeding into knockout rounds with configurable qualification criteria |
| **Challenger Rotation** | Rotating challenger system for continuous play formats |
| **Double Elimination** | Winner and loser bracket management with correct finals routing |
| **Fixed Matches with Bye Recovery** | Pre-determined fixture list with automatic bye recovery when participants are missing |
| **Gauntlet Staircase** | Progressive difficulty format with configurable advancement rules |

### Match Operations
- Automated fixture generation per format rules
- Conflict-aware scheduling: validates venue availability, umpire assignments, and participant eligibility simultaneously
- Match lifecycle management: `SCHEDULED → IN_PROGRESS → COMPLETED`
- Match stoppage and resumption with timestamp tracking
- Sport-specific event recording (goals, assists, cards, points, sets)

### Participant & Team Management
- Player profiles with sport-specific technical attributes stored as flexible JSONB (positions, play styles, grip types, racket specifications)
- Biological attribute tracking: height, weight, hand/foot dominance, blood type
- Team roster management with join requests, invitation flows, and role assignments
- Coach and umpire profiles with certification tracking

### Venue & Umpire Management
- Venue registration with GPS coordinates
- Multi-provider geocoding for venue discovery (Nominatim, Google Maps, Mapbox) with automatic provider fallback
- Umpire certification level tracking and match assignment

### Notifications
- Async email and in-app notifications via event-driven architecture
- Notification triggers: registration confirmation, match scheduling, score updates, season events
- Severity-based notification levels
- Notification logic fully decoupled from core business operations

### Statistics & Analytics
- Tournament standings updated in real time as results are submitted
- Cross-season player statistics: cumulative performance across multiple tournament cycles
- Head-to-head comparison between participants
- Combined stage standings for multi-phase competitions

---

## System Architecture

TourneyOps is structured as a **modular monolith** — a single deployable unit internally organized into self-contained domain modules, each with its own entities, repositories, services, and controllers. This was a deliberate architectural choice over microservices at this stage, prioritising deployment simplicity and development velocity without sacrificing internal modularity.

```
┌─────────────────────────────────────────────────────────────┐
│                     Next.js Frontend                        │
│           Dashboard · Fixtures · Registrations              │
│              Standings · Statistics · Admin                 │
└──────────────────────────┬──────────────────────────────────┘
                           │ HTTPS / REST API
┌──────────────────────────▼──────────────────────────────────┐
│                  Spring Boot Backend                        │
│                                                             │
│  ┌─────────────┐  ┌──────────────┐  ┌──────────────────┐   │
│  │ Auth Module │  │ Season Module│  │  Match Module    │   │
│  │ JWT / BCrypt│  │ Registration │  │  Orchestration   │   │
│  └─────────────┘  │ Eligibility  │  │  Score Tracking  │   │
│                   └──────────────┘  └──────────────────┘   │
│  ┌─────────────┐  ┌──────────────┐  ┌──────────────────┐   │
│  │  Participant│  │  Venue &     │  │  Notification    │   │
│  │  Team Module│  │  Umpire Mgmt │  │  Engine (Async)  │   │
│  └─────────────┘  └──────────────┘  └──────────────────┘   │
│  ┌─────────────┐  ┌──────────────┐  ┌──────────────────┐   │
│  │  Statistics │  │  Geocoding   │  │  Format Engines  │   │
│  │  Dashboard  │  │  (Fallback)  │  │  × 7 formats     │   │
│  └─────────────┘  └──────────────┘  └──────────────────┘   │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │          Global Exception Handler (AOP)              │   │
│  └─────────────────────────────────────────────────────┘   │
└──────────────────────────┬──────────────────────────────────┘
                           │ JPA / Liquibase
┌──────────────────────────▼──────────────────────────────────┐
│                     PostgreSQL                              │
│         35+ tables · JSONB columns · 35+ migrations        │
└─────────────────────────────────────────────────────────────┘

External Services:
┌──────────────┐  ┌──────────────────────────────────────┐
│  Zepto Mail  │  │  Geocoding Providers (with fallback)  │
│  (Email)     │  │  Nominatim → Google Maps → Mapbox    │
└──────────────┘  └──────────────────────────────────────┘
```
![TourneyOps System Architecture](docs/architecture/system-overview.png)
---

## Domain Model

The domain model reflects the real operational complexity of sports tournament management. Key design decisions:

**Entities (35+ total):**

```
User ──< UserRole >── Role
User ──< Player
User ──< Coach
User ──< Umpire
User ──< Club
User ──< Organiser

Player ──< PlayerSport (JSONB attributes per sport)

Club ──< Team ──< TeamMember

TournamentSeason ──< TournamentParticipant
TournamentSeason ──< TournamentStage ──< TournamentGroup
TournamentSeason ──< TournamentMatch ──< MatchEvent

TournamentMatch >── Venue
TournamentMatch >── Umpire
```

**Key design decisions in the domain model:**

**JSONB for sport-specific attributes** — Different sports have fundamentally different player profiles. A football player has positions and play styles. A table tennis player has grip types and racket specifications. Rather than creating sport-specific tables that would require schema changes for every new sport, sport attributes are stored as validated JSONB, keeping the model extensible while maintaining queryability.

**Role-additive user model** — A single user account can hold multiple roles simultaneously. This reflects real-world usage (a coach who also plays, a club admin who also organises) and avoids duplicate user records.

**Stage and group abstraction** — Tournament stages (group phase, knockout phase, etc.) and groups within stages are first-class entities, enabling the system to correctly model multi-phase competitions with different rules at each phase.

---

## Tournament Format Engines

The orchestration layer is the most technically complex part of the system. Each of the seven format engines implements a common interface but handles substantially different domain logic.

The challenge is not simply generating fixtures. It is handling progression:
- Which participants advance to the next round?
- What happens when a participant has a bye?
- How are tiebreakers resolved within a group?
- How does a double elimination loser bracket merge back into the winner bracket for the final?

Each engine is responsible for:
1. Validating that the tournament configuration is valid for that format
2. Generating the initial fixture set
3. Advancing participants to subsequent rounds based on results
4. Handling edge cases (byes, walkovers, disqualifications)
5. Determining when the competition is complete and who won

```
TournamentOrchestratorEngine (interface)
    ├── LeagueOrchestratorEngine
    ├── KnockoutOrchestratorEngine
    ├── GroupToKnockoutOrchestratorEngine
    ├── ChallengerRotationOrchestratorEngine
    ├── DoubleEliminationOrchestratorEngine
    ├── FixedMatchesWithByeRecoveryOrchestratorEngine
    └── GauntletStaircaseOrchestratorEngine
```

---

## API Design

The API follows REST conventions with a consistent response envelope across all endpoints.

**Base URL:** `https://api.tourneyops.com/api/v1`

**Response envelope:**
```json
{
  "success": true,
  "message": "Season retrieved successfully",
  "data": { ... },
  "errors": null,
  "timestamp": "2026-06-03T10:30:00Z"
}
```

**Authentication:** JWT Bearer token. Stateless. Tokens include role claims for server-side authorization.

**API Modules (122 endpoints across 21 controllers):**

| Module | Base Path | Key Operations |
|--------|-----------|----------------|
| Authentication | `/auth` | Register, login, email verification, password reset |
| Tournament Seasons | `/tournament-seasons` | CRUD, open/close registration, publish |
| Orchestration | `/tournament-orchestrator` | Generate fixtures, advance rounds |
| Matches | `/tournament-matches` | Schedule, record events, complete |
| Participants | `/tournament-participants` | Register, manage eligibility |
| Players | `/players` | Profile management, sport specializations |
| Teams | `/teams` | Create, manage roster, join requests |
| Team Members | `/team-members` | Add, remove, assign roles |
| Umpires | `/umpires` | Profiles, certifications, assignment |
| Venues | `/venues` | Register, geocode, availability |
| Series | `/series` | Multi-season series management |
| Divisions | `/divisions` | Division configuration within seasons |
| Notifications | `/notifications` | Fetch, mark read, preferences |
| Dashboard | `/dashboard` | Aggregated stats per role |
| Cross-Season Stats | `/cross-season-stats` | Historical analytics |

---

## Technology Stack

### Backend
| Layer | Technology | Version | Rationale |
|-------|-----------|---------|-----------|
| Language | Java | 17 | LTS release; strong typing; enterprise ecosystem |
| Framework | Spring Boot | 3.5.6 | Production-grade; mature ecosystem; Spring Security integration |
| Security | Spring Security + JJWT | 0.12.6 | Stateless JWT auth; BCrypt password hashing |
| ORM | Spring Data JPA / Hibernate | Latest | Clean repository pattern; native query support where needed |
| Database Migrations | Liquibase | 4.27.0 | Versioned, auditable schema changes; safe across environments |
| Build | Maven | 3.x | Standard Java build tooling |
| Containerization | Docker | Multi-stage | Consistent deployment; environment isolation |
| Email | Zepto Mail API | v1.1 | Transactional email delivery |
| Geocoding | Nominatim / Google Maps / Mapbox | — | Multi-provider with fallback strategy |

### Database
| Component | Technology | Notes |
|-----------|-----------|-------|
| Primary Store | PostgreSQL | 15+; JSONB for flexible sport attributes; strong ACID guarantees |
| Schema Management | Liquibase | 35+ migrations; versioned; environment-safe |
| Query Approach | JPA + native queries | ORM for standard operations; native SQL for complex analytics |

### Frontend
| Component | Technology | Notes |
|-----------|-----------|-------|
| Framework | Next.js 14 | SSR for performance; React ecosystem |
| State Management | React state / Context | Proportional to complexity |
| HTTP Client | Axios | Interceptors for auth token injection |

---

## Engineering Decisions

**Why a modular monolith over microservices?**

At current scale, the operational overhead of microservices (distributed tracing, service discovery, network latency between services, complex deployment pipelines) would exceed the benefits. The modular monolith gives us clean internal boundaries today with a clear migration path to services later if specific modules (e.g. notifications, statistics) need independent scaling.

**Why JSONB for sport-specific player attributes?**

The alternative — a separate table per sport — would require a schema migration every time a new sport is added. JSONB keeps the model extensible while remaining queryable. Validation of sport-specific fields happens at the service layer, not the database constraint level, which is an acceptable trade-off for the flexibility gained.

**Why Liquibase over Hibernate DDL auto?**

`spring.jpa.hibernate.ddl-auto=update` is a liability in any production system — it can silently drop columns, misinterpret renames, and behaves differently across database versions. Every schema change in TourneyOps is a versioned, reviewable Liquibase changeset that can be safely replayed across dev, staging, and production.

**Why event-driven notifications?**

Coupling notification logic to business transactions would make the core domain logic harder to test, slower to execute, and brittle to email service outages. Spring application events let notification triggers fire after a transaction commits, run asynchronously, and fail independently of the business operation that triggered them.

**Why multi-provider geocoding?**

A single geocoding provider creates a single point of failure for venue discovery. The fallback chain (Nominatim → Google Maps → Mapbox) ensures the feature remains operational even if one provider is unavailable or rate-limited, with no user-visible degradation.

---

## Challenges & Lessons Learned

### 1. Domain complexity precedes code complexity

The hardest part of building TourneyOps was not writing the code — it was understanding the domain well enough to model it correctly before writing anything. Tournament management has subtleties that only become apparent when you talk to real organisers: the difference between a bye and a walkover, how tiebreakers cascade through group stages, what happens when an umpire cancels on match day.

**Lesson:** Time spent on domain modeling before implementation is not delay — it is the work. A wrong entity relationship discovered at migration 35 is far more expensive than one discovered before migration 1.

### 2. Production is a different environment

Early in the platform's operation, a critical user flow was needed for a live tournament before the code had been written. The decision was made to make a targeted, documented database change directly to keep the competition running, then implement the proper code flow with full edge case coverage afterwards.

**Lesson:** Production systems require pragmatic engineering judgment — knowing when a controlled manual intervention is correct, and always closing the loop with the proper implementation. The key is documentation and follow-through.

### 3. Scheduling conflicts are a constraint satisfaction problem

The conflict-aware scheduling system went through multiple iterations. The initial approach — checking venue, umpire, and participant availability as separate sequential queries — had race conditions under concurrent requests. The correct approach was to handle all constraint validation within a single transaction boundary.

**Lesson:** Concurrent write scenarios in booking/scheduling systems require careful transaction design. Optimistic locking and transaction-scoped constraint checks are not optional.

### 4. The role-additive user model saved significant re-work

An early design had separate user tables per role (PlayerUser, CoachUser, etc.). This was refactored to a single User entity with a role junction table after it became clear that real users frequently hold multiple roles. The refactor was painful mid-development but correct.

**Lesson:** Model the real world, not the idealized version. Users are not always cleanly one thing.

---

## Business Impact

TourneyOps addresses a real operational problem in an underserved market. The value proposition is measurable:

**For organisers:**
- Bracket draws that previously took 2–3 hours of manual work now take minutes
- Scheduling conflicts identified before matches are published rather than on match day
- Results and standings published automatically, eliminating manual spreadsheet updates

**For participants:**
- Single source of truth for fixtures, results, and standings
- Performance statistics tracked across multiple tournament seasons
- Match notifications without relying on organiser WhatsApp broadcasts

**Market context:**
- Sports tournament management software exists for Western markets but almost none of it is designed for, priced for, or hosted in the African market
- TourneyOps is Nigerian-owned, Nigerian-hosted, and designed around how Nigerian sports organisations actually operate

**Current traction:**
- 100+ registered users
- 5+ active organisations
- 5+ completed tournaments
- Football and Table Tennis fully operational
- Active UX iteration based on organiser feedback

---

## Roadmap

### Near-term (active development)
- [ ] Additional sports: Basketball, Badminton, Volleyball
- [ ] Mobile-responsive UX improvements based on current organiser feedback
- [ ] Advanced statistics: player ratings, performance trends
- [ ] Tournament discovery and public registration pages

### Medium-term
- [ ] Mobile application (iOS + Android)
- [ ] Organiser billing and subscription management
- [ ] Live match scoring interface for umpires
- [ ] Spectator-facing live standings and fixture tracker

### Long-term
- [ ] Multi-country expansion beyond Nigeria
- [ ] API access for third-party integration (sports media, betting, broadcast)
- [ ] Talent identification: data-driven player profiles for scouts and academies

---

## Screenshots

> *Screenshots coming — the sections below describe what to expect.*

| Screen | Description |
|--------|-------------|
| **Organiser Dashboard** | Overview of active seasons, upcoming matches, registration counts, and platform activity |
| **Season Setup** | Format selection, date configuration, eligibility rules, sport selection |
| **Fixture View** | Generated bracket or round-robin grid with match dates, venues, and umpire assignments |
| **Match Detail** | Live event recording interface — goals, cards, points — with running score |
| **Standings Table** | Real-time standings with points, goal difference, head-to-head tiebreakers |
| **Player Profile** | Sport-specific attributes, match history, cross-season statistics |
| **Registration Flow** | Player/team registration with eligibility validation |

---

## Repository Structure

```
tourneyops-showcase/
│
├── README.md                          ← This file
├── docs/
│   ├── architecture/
│   │   ├── system-overview.png        ← High-level architecture diagram
│   │   ├── domain-model.png           ← Entity relationship overview
│   │   └── format-engines.png        ← Tournament engine hierarchy
│   ├── api/
│   │   └── api-overview.md           ← API modules and endpoint summary
│   ├── design/
│   │   ├── scheduling-logic.md       ← Conflict detection design
│   │   └── notification-architecture.md ← Event-driven notification design
│   └── decisions/
│       └── adr-001-modular-monolith.md ← Architecture Decision Record
│
└── screenshots/
    ├── dashboard.png
    ├── fixture-view.png
    ├── match-scoring.png
    └── standings.png
```

---

## Contact

**Samson Kayode** — Software Engineer & Co-Founder, TourneyOps

- **Email:** kayodesamson4@gmail.com
- **LinkedIn:** [linkedin.com/in/kayodesamson](https://linkedin.com/in/kayodesamson)
- **GitHub:** [github.com/samzion](https://github.com/samzion)
- **Platform:** [tourneyops.com](https://tourneyops.com)

Open to backend engineering roles, technical co-founder conversations, and product partnerships. Available immediately.

---

*TourneyOps is co-built with [Emmanuel Kayode](https://github.com/tobi007).*
