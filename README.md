# Converso – Real-Time AI Teaching Platform

Converso is a full-stack Next.js app where users can create AI companions and run real-time voice learning sessions.

## Features

- Real-time voice tutoring sessions powered by Vapi
- Companion builder with subject, topic, voice, style, and duration
- Companion library with subject/topic filtering
- Bookmark companions and track recent session history
- Personal dashboard (`/my-journey`) with:
	- bookmarked companions
	- recent sessions
	- user-created companions
- Clerk authentication + subscription gating via Clerk Pricing Table
- Supabase-backed persistence for companions, sessions, and bookmarks
- Sentry instrumentation configured for Next.js

## Tech Stack

- Next.js 16 (App Router, React 19)
- TypeScript
- Tailwind CSS
- Clerk (auth + billing UI)
- Supabase (database)
- Vapi Web SDK (voice sessions)
- Sentry (monitoring)

## Routes

- `/` – home (popular companions + recent sessions)
- `/companions` – companion library
- `/companions/new` – create companion
- `/companions/[id]` – live companion session
- `/my-journey` – profile, bookmarks, history
- `/subscription` – Clerk pricing table

## Prerequisites

- Node.js 20+
- npm 10+
- Clerk account/project
- Supabase project
- Vapi account + web token

## Environment Variables

Create `.env.local` in the project root:

```env
# Clerk
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=
CLERK_SECRET_KEY=
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_IN_FALLBACK_REDIRECT_URL=/
NEXT_PUBLIC_CLERK_SIGN_UP_FALLBACK_REDIRECT_URL=/

# Supabase
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_DEFAULT_KEY=

# Vapi
NEXT_PUBLIC_VAPI_WEB_TOKEN=

# Sentry (optional for local)
SENTRY_AUTH_TOKEN=
```

## Installation & Local Run

```bash
npm install
npm run dev
```

App runs at `http://localhost:3000`.

## Database Setup (Supabase)

Run the following SQL in the Supabase SQL Editor.

```sql
create extension if not exists "pgcrypto";

create table if not exists public.companions (
	id uuid primary key default gen_random_uuid(),
	name text not null,
	subject text not null,
	topic text not null,
	voice text not null,
	style text not null,
	duration integer not null,
	author text not null,
	bookmarked boolean not null default false,
	created_at timestamptz not null default now()
);

create table if not exists public.session_history (
	id uuid primary key default gen_random_uuid(),
	companion_id uuid not null references public.companions(id) on delete cascade,
	user_id text not null,
	created_at timestamptz not null default now()
);

create table if not exists public.bookmarks (
	id uuid primary key default gen_random_uuid(),
	companion_id uuid not null references public.companions(id) on delete cascade,
	user_id text not null,
	created_at timestamptz not null default now(),
	unique (companion_id, user_id)
);
```

## Build & Production

```bash
npm run build
npm run start
```

## Troubleshooting

### 1) `Could not find the table 'public.bookmarks' in the schema cache`

Cause: the `bookmarks` table is missing in Supabase.

Fix: create the table using the SQL in the **Database Setup** section.

### 2) Webpack `.pack.gz` cache restore errors on Windows/OneDrive

Cause: filesystem cache lock/corruption in `.next/dev/cache/webpack`.

Fixes:
- delete `.next` and restart
- keep the project outside synced OneDrive folders when possible

### 3) Session errors during voice call

Check:
- valid `NEXT_PUBLIC_VAPI_WEB_TOKEN`
- provider config/credits (Vapi/OpenAI/Deepgram/11labs)
- only one active session per user/tab during testing

## Project Structure

```text
app/
	companions/
	my-journey/
	subscription/
components/
lib/
	actions/
	supabase.ts
	vapi.sdk.ts
constants/
types/
```

## Notes

- This app uses Clerk server auth inside server actions.
- Bookmark/session data in UI depends on Supabase relational queries.
- Sentry files are included and can be enabled/adjusted per environment.
