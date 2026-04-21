# mytodo.page — Project Context

## What this is
A personal task tracker web app. Single `index.html` + `privacy.html`, no framework, no build tools. Deployed at https://mytodo.page via Vercel. Source: https://github.com/jtpaek429/Task-Tracker

## Stack
- **Frontend**: Vanilla JS, CSS Grid, single HTML file
- **Backend**: Supabase (free tier) — Postgres DB + auth
- **Hosting**: Vercel (auto-deploys from `main`)
- **Domain**: mytodo.page (leased through Vercel)

## Auth
- Email/password + Google OAuth (both configured in Supabase)
- Supabase anon key is safe in public code — RLS enforces per-user data isolation
- Apple OAuth skipped (requires $99/year Apple Developer account)

## Supabase setup
- Project URL and anon key are hardcoded at the top of the `<script>` block in `index.html`
- `tasks` table with RLS policy: `auth.uid() = user_id`
- Redirect URLs configured for `https://mytodo.page/**` and Vercel preview URLs

```sql
create table tasks (
  id           text primary key,
  user_id      uuid references auth.users not null,
  title        text not null,
  description  text,
  due          text,
  link         text,
  completed    boolean default false,
  completed_at timestamptz,
  created_at   timestamptz default now()
);
alter table tasks enable row level security;
create policy "own tasks only" on tasks
  for all using (auth.uid() = user_id);
```

## Data flow
- `tasks` array is the in-memory state
- Every mutation updates localStorage immediately (fast render) then calls Supabase async
- On sign-in, tasks are loaded fresh from Supabase and overwrite localStorage
- `dbUpsert(task)` / `dbDelete(id)` are the two Supabase write helpers

## Key UI decisions
- **Sections**: Due Soon (red), Upcoming (amber), Future (blue), Unscheduled (gray), Done (green)
- **Unscheduled** (formerly "Triage"): full-width, auto-collapses when empty
- **Preset pills** in task modal are colored to match their section
- **Dev mode**: generates 5 test tasks (tagged `_dev:true`) and can bulk-delete them

## Mobile (≤700px)
- Single-column stacked layout
- Floating red `+` FAB replaces header Add Task button
- Task modal becomes a bottom sheet (slides up from bottom)
- Always-visible edit/delete buttons (no hover dependency)
- Drag-and-drop disabled on touch (`hover: none` media query)
- iOS zoom fix: inputs set to `font-size: 16px`
- Tap card description to expand beyond 2-line clamp

## Parked ideas (not yet built)
- Swipe right to complete task on mobile
- Pull-to-refresh
- PWA / add to home screen support
- Haptic feedback on task completion (Android only, iOS Safari doesn't support)
- Account deletion flow (needed before opening to public users)

## Repo / branch hygiene
- Work on feature branches, open PRs against `main`
- Vercel auto-generates a preview URL for every branch push
- Use preview URLs to test on mobile before merging

## Contact / ownership
- GitHub: jtpaek429
- Contact email for privacy requests: jpaekprojects@gmail.com
