# OctoFit Tracker architecture

OctoFit Tracker is a three-tier application for a school fitness program. The starter code includes API contracts and intentionally incomplete implementation points so you can use GitHub Copilot agent mode in a test-driven loop.

![Three-tier OctoFit architecture](../.github/images/octofit-architecture.svg)

## Tier boundaries

- **Presentation tier**: React 19, Vite, React Router, Bootstrap, and a shared API client in `octofit-tracker/frontend/src`.
- **Logic tier**: Express and TypeScript routes/services in `octofit-tracker/backend/src` expose `/api/*` behavior on port `8000`.
- **Data tier**: Mongoose models and seed data in `octofit-tracker/backend/src/models` and `src/scripts/seed.ts` persist to MongoDB on port `27017`.

## Ports and environment rules

- MongoDB: `27017`
- Express API: `8000`
- Vite frontend: `5173`
- The browser calls relative `/api` paths. Vite proxies them to `http://localhost:8000`, so the unauthenticated API port stays private in Codespaces. `VITE_API_BASE_URL` can override the base URL.

## API contract

| Capability | Method and path | Expected behavior |
| --- | --- | --- |
| Activity tracking | `GET /api/activities`, `POST /api/activities` | List and create valid activity records. POST accepts `userId`, `teamId`, `type`, and `minutes`; `userId` and `teamId` must be well-formed IDs for an existing user and team; otherwise respond with a deliberate 4xx (never a 500 from a cast error). A created activity stores the submitted `user`, `team`, `type`, and `minutes` plus computed `points` and `occurredAt`. |
| Teams | `GET /api/teams`, `POST /api/teams` | List and create teams with members. Created and listed teams keep the submitted `name` and `mascot`. Reject a team without a name with `400`. |
| Leaderboard | `GET /api/leaderboard` | Return teams ordered by the sum of their activity points, highest first. Each entry has `teamName`, `totalPoints`, and a 1-based `rank`. |
| Workout suggestions | `GET /api/workouts/suggestions/:userEmail` | Recommend workouts from recent activity patterns for the matching seeded user's email address. The first suggestion's `focus` matches that user's goal or recent activity, with a `reason` explaining why. An email with no matching user returns a deliberate 4xx (for example, `404`). |

## Implementation decisions

Record your plan here before editing code.

- Presentation tier decision: Implement route components in `octofit-tracker/frontend/src` that use the shared API client, keep loading, success, empty, and error state in the presentation layer, and render accessible Bootstrap UI.
- Logic tier decision: Implement Express route handlers and explicit TypeScript service functions under `octofit-tracker/backend/src` for `/api/activities`, `/api/teams`, `/api/leaderboard`, and `/api/workouts/suggestions/:userEmail`.
- Data tier decision: Complete the Mongoose schemas in `octofit-tracker/backend/src/models` with required relationships and fields, while keeping seed data in `octofit-tracker/backend/src/scripts/seed.ts` stable and idempotent.
- Activity tracking approach: Validate submitted `userId`, `teamId`, `type`, and `minutes`; reject malformed or missing references with deliberate 4xx responses; create activities with linked users and teams, positive computed points, and an `occurredAt` date.
- Team management approach: Require a team name and mascot, default `members` to an empty array, preserve submitted members, and return clear 400 responses for invalid team input.
- Leaderboard approach: Aggregate activity points by team, sort totals highest first, and assign one-based ranks in team leaderboard entries.
- Workout suggestions approach: Find the user by email, infer a focus from recent activity or the user's fitness goal, match a seeded workout, and include a reason that explains the recommendation; return a 4xx response for unknown email addresses.

## Test plan

Record the commands you run and what each failure means.

- Data-tier command: `npm run test:data --prefix octofit-tracker/backend`
  - Failure here means Mongoose data contracts or idempotent seed behavior are incomplete. Required validation failures currently cover user identity and goals, team fields and membership defaults, activity relationships and metrics, and workout fields.
- API command: `npm run test:api --prefix octofit-tracker/backend`
  - Failure here means route or service behavior is incomplete. The contracts require activity and team creation/listing with deliberate 4xx validation, leaderboard aggregation and ranking, and email-based workout recommendations with 4xx handling for unknown users.
- Frontend command: `npm test --prefix octofit-tracker/frontend`
  - Failure here means presentation behavior is incomplete. The contracts require route data to render, the Mona workout request to use the specific suggestions email endpoint, and accessible loading, empty, success, and API error states across all four capabilities.
- Backend build command: `npm run build --prefix octofit-tracker/backend`
  - A failure here means the TypeScript Express implementation does not compile or has type errors.
- Frontend build command: `npm run build --prefix octofit-tracker/frontend`
  - A failure here means the React/Vite implementation does not build or has type errors.
