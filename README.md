# Doomsday Ready — Ultimate MCU Rewatch & Booking Hub

[![Node.js](https://img.shields.io/badge/Node.js-22%2B-green.svg)](https://nodejs.org/)
[![Next.js](https://img.shields.io/badge/Next.js-15-black.svg)](https://nextjs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.7-blue.svg)](https://www.typescriptlang.org/)
[![Tailwind CSS](https://img.shields.io/badge/TailwindCSS-3.4-38bdf8.svg)](https://tailwindcss.com/)
[![Prisma ORM](https://img.shields.io/badge/Prisma%20ORM-8.0.0--rc.24-5a67d8.svg)](https://www.prisma.io/)
[![Vitest](https://img.shields.io/badge/Vitest-5.1-yellow.svg)](https://vitest.dev/)

**Doomsday Ready** is a full-stack, production-quality web application engineered for Marvel Cinematic Universe (MCU) fans preparing for *Avengers: Doomsday* (2026). It combines timeline dependency modeling with algorithmic rewatch planning and seat allocation optimization for opening-night watch parties.

---

## Table of Contents

- [1. Project Overview](#1-project-overview)
- [2. Key Features](#2-key-features)
- [3. Technology Stack](#3-technology-stack)
- [4. Repository & Architecture](#4-repository--architecture)
- [5. Database Schema & Mermaid Diagram](#5-database-schema--mermaid-diagram)
- [6. Algorithmic Implementations & Complexity](#6-algorithmic-implementations--complexity)
  - [A. Movie Dependency Graph & Topological Sorting](#a-movie-dependency-graph--topological-sorting)
  - [B. Unbounded Coin-Change Dynamic Programming for Cinema Seat Bundles](#b-unbounded-coin-change-dynamic-programming-for-cinema-seat-bundles)
- [7. API Documentation](#7-api-documentation)
- [8. Prerequisites & Environment Setup](#8-prerequisites--environment-setup)
- [9. Installation, Database Migration & Seeding](#9-installation-database-migration--seeding)
- [10. Development & Testing Commands](#10-development--testing-commands)
- [11. Docker Compose Setup](#11-docker-compose-setup)
- [12. Screenshots & UI Walkthrough](#12-screenshots--ui-walkthrough)
- [13. Assumptions & Limitations](#13-assumptions--limitations)
- [14. AI Assistance Disclosure](#14-ai-assistance-disclosure)

---

## 1. Project Overview

With the announcement of *Avengers: Doomsday* featuring Robert Downey Jr. as Victor Von Doom, fans face an extensive library of interconnected films spanning multiple phases. Watching them in simple release order can lead to redundant viewing, while watching without prerequisites causes confusion.

**Doomsday Ready** solves two core problems:
1. **Precise Chronological Prerequisite Ordering**: Determines the exact subset of transitive prerequisites needed for any destination MCU movie and sorts them into a topologically valid, chronological rewatch order while detecting any potential circular dependencies.
2. **Opening-Night Cinema Bundle Optimization**: Calculates the absolute minimum number of seat bundles needed to accommodate a watch party group size at a given cinema venue with zero wasted seats.

---

## 2. Key Features

- **MCU Catalog Management**: Complete CRUD operations for MCU titles with title uniqueness, positive release year, runtime bounds (1–300 minutes), watched status tracking, and multi-prerequisite relationships.
- **Circular Dependency Guard**: Proactive cycle-detection engine that rejects self-dependencies and indirect circular loops (HTTP 409 `CIRCULAR_DEPENDENCY`).
- **Custom Topological Rewatch Engine**: Pure custom DFS graph traversal with recursion-stack cycle detection and deterministic tie-breaking (release year, then title). Accurately calculates total prerequisites, watched count, unwatched count, and remaining watch time.
- **Unbounded Coin-Change DP Seat Optimizer**: Pure custom dynamic programming algorithm calculating minimum seat bundles for party sizes (1–10,000) across 1–10 distinct bundle sizes (1–500), complete with optimal bundle quantities and impossible state detection.
- **Cinematic Marvel-Themed Dark UI**: Premium dark charcoal aesthetic (`#08090c`) with deep crimson accents (`#e23636`), subtle gold highlights, responsive grids, interactive progress bars, and modal forms.
- **Live Operational Dashboard**: Real-time calculated statistics (Total Movies, Watched Movies, Remaining Time, Cinema Venues) directly synchronized with backend data.

---

## 3. Technology Stack

### Frontend
- **Framework**: Next.js 15 (App Router, React 19, TypeScript).
- **Styling**: Tailwind CSS 3.4 with custom Marvel-inspired theme tokens.
- **Icons**: Lucide React.
- **HTTP Client**: Axios with centralized error handling and request abstraction.
- **State Management**: Built-in React hooks (`useState`, `useEffect`, `useCallback`, `useMemo`).

### Backend
- **Runtime**: Node.js 22+ (ES Modules, TypeScript 5.7+).
- **Framework**: Express.js 5.
- **Database**: PostgreSQL 18.
- **ORM & Contract**: Prisma 8 Release Candidate (`@prisma/orm-postgres: 8.0.0-rc.24`, `@prisma/cli-engine`, `@prisma/orm-toolchain`) using native contract emission and PostgreSQL adapter.
- **Request Validation**: Zod.
- **Architecture**: Modular layered architecture (Controllers, Services, Validators, Middlewares, Utilities).

### Testing & Quality
- **Test Runner**: Vitest 5.1.
- **Integration Testing**: Supertest 7.3.
- **Suite**: 40 automated tests (19 algorithm unit tests + 21 HTTP API integration tests).

---

## 4. Repository & Architecture

The repository enforces strict separation of concerns between client and server:

```text
doomsday-ready/
├── backend/
│   ├── prisma/
│   │   └── schema.prisma         # Legacy schema reference
│   ├── src/
│   │   ├── config/               # Environment & app configuration
│   │   ├── controllers/          # Express route controllers
│   │   ├── middlewares/          # Centralized error & 404 handlers
│   │   ├── prisma/
│   │   │   ├── contract.prisma   # Prisma 8 canonical data contract
│   │   │   ├── contract.json     # Emitted Contract IR
│   │   │   ├── contract.d.ts     # Emitted TypeScript typings
│   │   │   ├── db.ts             # Runtime database singleton client
│   │   │   └── seed.ts           # Idempotent database seeder
│   │   ├── routes/               # API route definitions
│   │   ├── services/             # Business logic layer
│   │   ├── utils/                # Graph & Dynamic Programming algorithms
│   │   ├── validators/           # Zod validation schemas
│   │   ├── app.ts                # Express app setup
│   │   └── server.ts             # Server entry point
│   ├── tests/
│   │   ├── integration/          # Supertest API endpoint tests
│   │   └── unit/                 # Pure algorithm unit tests
│   ├── prisma.config.ts          # Prisma 8 project configuration
│   ├── tsconfig.json             # TypeScript configuration
│   └── package.json
│
├── frontend/
│   ├── src/
│   │   ├── app/                  # Next.js App Router pages
│   │   │   ├── booking/          # Cinema Booking view
│   │   │   ├── movies/           # Movie Catalog view
│   │   │   ├── rewatch/          # Rewatch Plan view
│   │   │   ├── globals.css       # Custom Marvel styles & scrollbars
│   │   │   ├── layout.tsx        # Root HTML layout with Navbar & Footer
│   │   │   └── page.tsx          # Operational Dashboard
│   │   ├── components/           # Reusable UI components (Navbar, Modal, StatCard, etc.)
│   │   ├── features/             # Feature-specific components
│   │   │   ├── booking/          # Booking calculator & quote breakdown
│   │   │   ├── movies/           # Movie list, cards, and modal forms
│   │   │   └── rewatch/          # Chronological timeline & progress bar
│   │   ├── hooks/                # Custom React data hooks (useMovies, useRewatchPlan, etc.)
│   │   ├── lib/                  # Axios API client
│   │   ├── services/             # Frontend API service layer
│   │   └── types/                # Shared TypeScript contracts
│   ├── tailwind.config.ts        # Tailwind design tokens
│   ├── tsconfig.json             # TypeScript configuration
│   └── package.json
│
├── docker-compose.yml            # Multi-container orchestration
└── README.md                     # Project documentation
```

---

## 5. Database Schema & Mermaid Diagram

The database schema is fully normalized in PostgreSQL:

```mermaid
erDiagram
    Movie {
        uuid id PK "Primary Key (UUID)"
        string title "Unique movie title"
        integer releaseYear "Release year"
        integer runtime "Runtime in minutes (1-300)"
        boolean watched "Watched flag (default: false)"
        timestamp createdAt "Creation timestamp"
        timestamp updatedAt "Update timestamp"
    }

    MovieDependency {
        uuid movieId PK, FK "Movie being watched"
        uuid prerequisiteId PK, FK "Required prerequisite movie"
    }

    Cinema {
        uuid id PK "Primary Key (UUID)"
        string name "Unique cinema venue name"
        timestamp createdAt "Creation timestamp"
        timestamp updatedAt "Update timestamp"
    }

    CinemaBundle {
        uuid id PK "Primary Key (UUID)"
        uuid cinemaId FK "Foreign Key to Cinema"
        integer size "Bundle seat capacity (1-500)"
    }

    Movie ||--o{ MovieDependency : "requires (movieId)"
    Movie ||--o{ MovieDependency : "prerequisite for (prerequisiteId)"
    Cinema ||--|{ CinemaBundle : "offers (1 to 10 bundles)"
```

### Relational Rules
- `MovieDependency`: Composite primary key `[movieId, prerequisiteId]`. Cascades on deletion (`onDelete: Cascade`).
- `CinemaBundle`: Composite unique constraint `@@unique([cinemaId, size])`. Cascades on deletion (`onDelete: Cascade`). Each cinema must maintain between 1 and 10 distinct bundle sizes.

---

## 6. Algorithmic Implementations & Complexity

> **Important**: No external graph or optimization libraries (such as Graphology, NetworkX, Math.js, or coin-change packages) were used. All algorithms were designed, developed, and tested from scratch for this project.

### A. Movie Dependency Graph & Topological Sorting

- **File**: `backend/src/utils/movie-graph.ts`
- **Cycle Detection**:
  - Implemented via Depth-First Search (DFS) with a `visiting` recursion stack and `visited` memoization set.
  - Before adding new prerequisites to an existing movie $M$, the algorithm tests whether any target prerequisite already possesses a directed dependency path leading back to $M$.
  - Rejects circular configurations immediately with HTTP 409 `CIRCULAR_DEPENDENCY`.
- **Topological Rewatch Order**:
  - Traverses transitive prerequisites recursively starting at target movie $T$.
  - Post-order accumulation ensures every prerequisite is visited and pushed to the schedule *before* the movie depending on it.
  - Excludes the target movie itself from the rewatch queue.
  - Guarantees each prerequisite appears **exactly once**, even in complex diamond dependency graphs (e.g., $A \to B \to D$ and $A \to C \to D$).
  - Deterministic tie-breaking: independent prerequisites are ordered chronologically by `releaseYear` ascending, then `title` alphabetically.
- **Complexity**:
  - **Time Complexity**: $\mathcal{O}(V + E)$ where $V$ is the number of movies and $E$ is the number of prerequisite dependency relations.
  - **Space Complexity**: $\mathcal{O}(V + E)$ to maintain the adjacency graph, recursion call stack, and visited sets.

### B. Unbounded Coin-Change Dynamic Programming for Cinema Seat Bundles

- **File**: `backend/src/utils/cinema-bundles.ts`
- **Objective**: Find the minimum number of ticket bundles such that:
  $$\sum_{i} (\text{bundleSize}_i \times \text{quantity}_i) = \text{groupSize}$$
  with zero empty or unused seats purchased.
- **DP State & Recurrence**:
  - `dp[s]` stores the minimum bundle count required to seat exactly $s$ guests.
  - `previous[s]` stores the bundle size used in the optimal transition to reconstruct the solution.
  - Base case: `dp[0] = 0`, all other `dp[s] = Infinity`.
  - Recurrence:
    $$\text{dp}[s] = \min_{\text{size} \in \text{bundles}, s \ge \text{size}} (\text{dp}[s - \text{size}] + 1)$$
- **Reconstruction**: Backtracks from `groupSize` down to `0` using `previous[s]`, aggregating counts into descending bundle sizes. If `dp[groupSize] === Infinity`, returns `{ possible: false, totalBundles: 0, bundles: [] }`.
- **Complexity**:
  - **Time Complexity**: $\mathcal{O}(\text{groupSize} \times B)$ where $B$ is the number of distinct bundle sizes ($1 \le B \le 10$). For the maximum group size of 10,000, operations complete in $< 10$ milliseconds.
  - **Space Complexity**: $\mathcal{O}(\text{groupSize})$ for the `dp` and `previous` arrays.

---

## 7. API Documentation

Base URL: `http://localhost:5000/api`

### Movies Endpoints

#### 1. `GET /api/movies`
Returns all cataloged MCU movies with their prerequisite IDs and watched status.
- **Response**: `200 OK`
```json
{
  "success": true,
  "data": [
    {
      "id": "c7a8...",
      "title": "Iron Man",
      "releaseYear": 2008,
      "runtime": 126,
      "watched": true,
      "prerequisites": [],
      "createdAt": "2026-11-21T...",
      "updatedAt": "2026-11-21T..."
    }
  ]
}
```

#### 2. `POST /api/movies`
Creates a new movie.
- **Request Body**:
```json
{
  "title": "Thor: Ragnarok",
  "releaseYear": 2017,
  "runtime": 130,
  "watched": false,
  "prerequisites": []
}
```
- **Response**: `201 Created`
- **Errors**: `400 Bad Request` (validation error), `409 Conflict` (`DUPLICATE_RESOURCE` or `CIRCULAR_DEPENDENCY`).

#### 3. `PATCH /api/movies/:id`
Updates movie attributes or prerequisite dependencies.
- **Request Body** (partial):
```json
{
  "watched": true,
  "runtime": 135
}
```
- **Response**: `200 OK`
- **Errors**: `404 Not Found`, `409 Conflict` (circular dependency).

#### 4. `GET /api/movies/:id/rewatch-order`
Generates the chronological topological rewatch plan for the specified movie.
- **Response**: `200 OK`
```json
{
  "success": true,
  "data": {
    "target": {
      "id": "b91f...",
      "title": "Avengers: Doomsday"
    },
    "movies": [
      {
        "id": "...",
        "title": "Iron Man",
        "releaseYear": 2008,
        "runtime": 126,
        "watched": true
      },
      {
        "id": "...",
        "title": "Captain America: The First Avenger",
        "releaseYear": 2011,
        "runtime": 124,
        "watched": true
      }
    ],
    "totalMovies": 7,
    "watchedCount": 4,
    "unwatchedCount": 3,
    "unwatchedRuntime": 460
  }
}
```

---

### Cinemas & Booking Endpoints

#### 1. `GET /api/cinemas`
Returns all cinema venues with their available bundle seat sizes.
- **Response**: `200 OK`
```json
{
  "success": true,
  "data": [
    {
      "id": "e42a...",
      "name": "Star Cineplex",
      "bundleSizes": [2, 5],
      "createdAt": "...",
      "updatedAt": "..."
    }
  ]
}
```

#### 2. `POST /api/cinemas`
Registers a new cinema venue.
- **Request Body**:
```json
{
  "name": "Starlight IMAX",
  "bundleSizes": [3, 8, 15]
}
```
- **Response**: `201 Created`
- **Errors**: `400 Bad Request` (invalid sizes, duplicates, count outside 1-10), `409 Conflict` (`DUPLICATE_RESOURCE`).

#### 3. `GET /api/cinemas/:id/booking?groupSize=11`
Calculates the optimal seat bundle quote.
- **Query Params**: `groupSize` (integer between 1 and 10000).
- **Response (Possible)**: `200 OK`
```json
{
  "success": true,
  "data": {
    "groupSize": 11,
    "possible": true,
    "totalBundles": 4,
    "bundles": [
      { "size": 5, "quantity": 1 },
      { "size": 2, "quantity": 3 }
    ]
  }
}
```
- **Response (Impossible)**: `200 OK`
```json
{
  "success": true,
  "data": {
    "groupSize": 7,
    "possible": false,
    "totalBundles": 0,
    "bundles": []
  }
}
```

---

## 8. Prerequisites & Environment Setup

### Prerequisites
- **Node.js**: v22.0.0 or higher.
- **PostgreSQL**: v16+ (v18 recommended; native service or Docker).
- **npm** or **pnpm**.

### Environment Variables

**Backend (`backend/.env`):**
```env
PORT=5000
DATABASE_URL="postgresql://postgres:1234@localhost:5432/doomsday_ready?schema=public"
FRONTEND_URL="http://localhost:3000"
NODE_ENV="development"
```

**Frontend (`frontend/.env.local`):**
```env
NEXT_PUBLIC_API_BASE_URL="http://localhost:5000/api"
```

---

## 9. Installation, Database Migration & Seeding

### 1. Install Dependencies
```bash
# Backend
cd backend
npm install

# Frontend
cd ../frontend
npm install
```

### 2. Database Setup & Prisma 8 Contract Emit
Prisma 8 is configured with `@prisma/orm-postgres`. Ensure PostgreSQL is running and the database `doomsday_ready` exists.

```bash
cd backend

# Emit contract IR and TypeScript definitions
npm run prisma:emit

# Apply contract updates to PostgreSQL database
npm run prisma:update
```

### 3. Seed MCU Movies and Cinemas
The database seed script is completely idempotent:
```bash
cd backend
npm run seed
```

Seeded content includes:
1. *Iron Man* (2008, 126m)
2. *Captain America: The First Avenger* (2011, 124m)
3. *The Avengers* (2012, 143m) $\leftarrow$ Iron Man, Captain America
4. *Captain America: Civil War* (2016, 147m) $\leftarrow$ The Avengers
5. *Thor: Ragnarok* (2017, 130m)
6. *Avengers: Infinity War* (2018, 149m) $\leftarrow$ Civil War, Thor: Ragnarok
7. *Avengers: Endgame* (2019, 181m) $\leftarrow$ Infinity War
8. *Avengers: Doomsday* (2026, 180m) $\leftarrow$ Endgame
- Cinemas: *Star Cineplex* [2, 5], *Cinema Hall A* [1, 3, 4], *Cinema Hall B* [6, 9, 20], *Cinema Hall C* [4, 6].

---

## 10. Development & Testing Commands

### Running Locally

**Terminal 1 — Backend:**
```bash
cd backend
npm run dev
# Server listening at http://localhost:5000
```

**Terminal 2 — Frontend:**
```bash
cd frontend
npm run dev
# Next.js running at http://localhost:3000
```

### Running Automated Tests
```bash
cd backend
npm test
```
Executes all 40 Vitest unit and Supertest integration tests.

### Production Build Verification
```bash
# Frontend Next.js build
cd frontend
npm run build

# Backend TypeScript check
cd ../backend
npm run build
```

---

## 11. Docker Compose Setup

Run the entire stack (PostgreSQL + Backend + Frontend) with Docker Compose:

```bash
docker compose up --build
```

- Frontend: `http://localhost:3000`
- Backend API: `http://localhost:5000/api`
- PostgreSQL: `localhost:5432`

To shut down:
```bash
docker compose down -v
```

---

## 12. Screenshots & UI Walkthrough

| View | Description | Placeholder |
| :--- | :--- | :--- |
| **Operational Dashboard** | Live MCU metrics, preparation overview, and quick links | `[Screenshot: Dashboard View]` |
| **MCU Movie Catalog** | Movie cards, watched toggles, and modal form | `[Screenshot: Movies View]` |
| **Topological Rewatch Plan** | Ordered timeline, progress indicator, and time remaining | `[Screenshot: Rewatch Plan View]` |
| **Cinema Seat Optimizer** | Venue selector, party size input, and bundle breakdown | `[Screenshot: Booking View]` |

---

## 13. Assumptions & Limitations

1. **Deterministic Tie-Breaking**: When multiple prerequisite movies are independent in the DAG, they are sorted chronologically by `releaseYear` ascending, and then by `title` alphabetically.
2. **Cinema Optimization**: The algorithm finds the minimal bundle count strictly matching the group size. Partial bundles or oversized purchases are returned as `possible: false`.
3. **No Copyrighted Posters**: In adherence with production guidelines, all UI representations use styled vector iconography, typographic badges, and cinematic gradients rather than copyrighted Marvel movie posters.
4. **Booking Quotes**: The booking calculator outputs pricing/bundle quotes only; no payment gateway integration is attached.

---

## 14. AI Assistance Disclosure

This application was engineered using agentic AI pair programming (Google DeepMind Antigravity). AI assistance was utilized for:
- Initial project scaffolding and Prisma 8 contract configuration.
- Comprehensive test case authoring (Vitest and Supertest).
- Implementation of the custom graph DFS traversal and unbounded coin-change DP algorithms from theoretical principles without third-party dependencies.
- Strict TypeScript verification and Tailwind CSS styling.
