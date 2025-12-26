## Project Overview

**Name**: Habit & Mood Coaching Platform Backend

This service is a NestJS-based backend for a habit and mood coaching platform. It provides APIs for:

- User authentication and profile management
- Subscription and billing with Stripe
- Habit creation, completion logging, and statistics
- Mood entry tracking, trends, and insights
- AI-generated daily routines with audio/text content
- Reminders for habits and routines
- Inspiration quotes, support tickets, and basic chat conversations

Everything is backed by PostgreSQL via Prisma and uses Redis, BullMQ, Firebase Storage, and Stripe integrations.

---

## Tech Stack

- Language: TypeScript (Node.js)
- Framework: NestJS 11 (REST + Swagger)
- ORM: Prisma with PostgreSQL
- Cache / Queue: Redis + BullMQ
- Auth: Passport (local, JWT, Google, Apple), JWT access/refresh tokens
- Storage: Local/S3/MinIO (via SazedStorage), Firebase Storage (GCS)
- Payments: Stripe (customers, payment methods, subscriptions, webhooks)
- Scheduling: @nestjs/schedule (Cron-based schedulers)

---

## Key Features

- **Authentication & Users**
	- Email/password registration & login
	- Google and Apple OAuth login
	- JWT access tokens + refresh tokens stored in Redis
	- Password reset, email verification via OTP, 2FA support
	- Basic profile endpoints (`/auth/me`, `/auth/update`, `/profile/me`)

- **Subscriptions & Payments**
	- Stripe customer creation and payment method handling
	- Subscription plans (`SubsPlan`) with free, trial, and paid tiers
	- Trial activation, plan listing, status lookup, and cancellation
	- Stripe webhook endpoint for subscription/payment events

- **Habits & Progress**
	- Habit CRUD with category, frequency, and duration
	- Daily completion logs (`HabitLog`) with undo support
	- "Today" view combining due habits, reminders, and completion state

- **Mood Tracking & Insights**
	- Mood entries (`MoodEntry`) with score, emotions, optional notes
	- Daily aggregates (`MoodDailyAggregate`) for averages, extremes, and top emotions
	- Trend and insight endpoints (average mood, volatility, streaks, top emotions)

- **AI Routines**
	- Onboarding preferences (`UserRoutineProfile`)
	- Daily routines (`Routine`, `RoutineItem`) combining meditation, sound healing, journaling, and podcast/video content
	- Media pulled from Firebase Storage and/or YouTube playlists
	- Routine and item status tracking with streak calculations

- **Reminders**
	- Habit- and routine-linked reminders with time, days, time zone, and window
	- Upcoming reminders for "today" for UI cards
	- Minute-level scheduler that scans due reminders and triggers notifications

- **Dashboard, Inspiration, Support & Chat**
	- Home summary for today's progress and mood
	- Profile overview metrics and achievements
	- Daily and personalized inspiration quotes
	- Support tickets and feedback endpoints
	- Basic chat conversations (conversations + messages)

---

## Configuration

### Stripe Webhook

Public webhook URL (replace `{domain_name}`):

```
http://{domain_name}/api/payment/stripe/webhook
```

For local development, with the Nest app running on port 4000:

```bash
stripe listen --forward-to localhost:4000/api/payment/stripe/webhook
```

Trigger a test payment intent event:

```bash
stripe trigger payment_intent.succeeded
```

### Environment

Copy `.env.example` to `.env` and configure according to your environment:

- Database: `DATABASE_URL`
- Redis: `REDIS_HOST`, `REDIS_PORT`, `REDIS_PASSWORD`
- JWT: `JWT_SECRET`, `JWT_EXPIRY`
- Stripe: `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`
- Firebase Storage: `FIREBASE_PROJECT_ID`, `FIREBASE_CLIENT_EMAIL`, `FIREBASE_PRIVATE_KEY`, `FIREBASE_STORAGE_BUCKET`
- Google/Apple OAuth client settings
- Mail SMTP credentials

---

## Installation

Install dependencies:

```bash
yarn install
```

---

## Setup

1. Copy `.env.example` to `.env` and update values.
2. Apply database migrations:

```bash
npx prisma migrate dev
```

3. Seed dummy data (if available):

```bash
yarn cmd seed
```

---

## Running

```bash
# development
yarn start

# watch mode
yarn start:dev

# production mode (after build)
yarn start:prod

# watch mode with swc compiler (faster)
yarn start:dev-swc
```

Run with Docker:

```bash
docker compose up
```

---

## API Documentation

Swagger UI is available at:

```
http://{domain_name}/api/docs
```

---

## Reminders API (Current)

The app includes a minute-level scheduler (via NestJS `@Cron`) that checks for due reminders and dispatches notifications. Create reminders via the API to match the client UI.

**Endpoints (prefixed with `/api` at runtime):**

- `POST /reminders/set`
	- Body fields: `reminder_time` ("HH:MM" or "HH:MM:SS"), optional `preferred_time`, `habit_id?`, `routine_id?`, `tz?`, `days?`, `name?`.
	- Auto-derives recurrence days based on habit frequency; validates time vs preferred window.

- `GET /reminders`
	- List all reminders for the authenticated user.

- `GET /reminders/upcoming`
	- Returns up to 3 upcoming reminders for "Coming Up Today" widgets.

- `GET /reminders/reminder-slots/:preferred`
	- Returns 30-minute reminder slots for a preferred window (e.g. Morning / Afternoon / Evening / Night).

- `PATCH /reminders/:id/turn-off-on`
	- Toggle a reminder's `active` flag.

- `PATCH /reminders/:id`
	- Edit reminder fields: name, time, days, tz, window, date, etc. Recomputes `scheduled_at` if needed.

- `DELETE /reminders/:id`
	- Delete a reminder.

Scheduling logic combines `time`, `days`, `tz`, and the current date to compute the next `scheduled_at`. The scheduler uses this to decide which reminders are due at each minute.
