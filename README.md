# mytodo.page

A clean, keyboard-first personal task tracker. Organize tasks by urgency, sync across devices, and get out of your own way.

**Live site → [mytodo.page](https://mytodo.page)**

---

## Features

**Task organization**
- Five sections: Due Soon, Upcoming, Future, Unscheduled, and Done — tasks route automatically based on due date
- Due Soon splits into Today / Tomorrow sub-groups
- Priority levels (P1 High, P2 Medium, P3 Low) sort tasks within each section
- Drag and drop to reschedule tasks between sections

**Input**
- Natural language date parsing — type "today", "next friday", "apr 25"
- Preset pills: Today, Tomorrow, Upcoming (+3 days), Future (+7 days), Unscheduled
- Multi-select with ⌘+click; bulk update date, priority, or delete

**Keyboard shortcuts**
| Key | Action |
|-----|--------|
| `C` | New task |
| `/` | Search |
| `?` | Show shortcuts |
| `⌘Z` / `⌘⇧Z` | Undo / Redo |
| `⌘⌫` | Delete selected |
| `Enter` (modal) | Save |
| `⌘↩` (modal) | Save + create another |
| `T/M/U/F/S` (modal) | Date presets |
| `1/2/3/0` (modal) | Priority |

**Other**
- Full-text search across all tasks
- Undo/redo for every mutation (create, edit, delete, complete, bulk actions)
- Auth: email/password + Google OAuth
- Account deletion (removes all data)
- Responsive — mobile layout with bottom-sheet modal and floating action button

---

## Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Vanilla JS, CSS Grid — single `index.html`, no build step |
| Auth | Supabase Auth (email/password + Google OAuth) |
| Database | Supabase Postgres with Row Level Security |
| Hosting | Vercel (auto-deploys from `main`) |
| Email | Resend (transactional auth emails) |
| Domain | mytodo.page via Vercel |

---

## Running locally

No build tools required. Clone the repo and open `index.html` in a browser directly, or serve it with any static server:

```bash
npx serve .
```

The app connects to the live Supabase backend. You can sign up for a real account or point it at your own Supabase project by swapping `SUPABASE_URL` and `SUPABASE_ANON_KEY` at the top of the `<script>` block in `index.html`.

---

## Project structure

```
index.html          # Entire frontend — markup, styles, and JS
privacy.html        # Privacy policy
terms.html          # Terms of service
supabase/
  functions/
    delete-account/ # Edge Function: deletes user tasks + auth record
```

---

## Data model

```sql
create table tasks (
  id           text primary key,
  user_id      uuid references auth.users not null,
  title        text not null,
  description  text,
  due          text,
  completed    boolean default false,
  completed_at timestamptz,
  created_at   timestamptz default now(),
  priority     integer   -- 1 High · 2 Medium · 3 Low · null = none
);
```

Row Level Security ensures users can only access their own tasks.
