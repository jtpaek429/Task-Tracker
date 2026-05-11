# mytodo.page

A keyboard-first personal task tracker. Tasks auto-sort into urgency buckets, sync across devices via Supabase, and stay out of your way.

**Live site → [mytodo.page](https://mytodo.page)**

### What it does
- Automatically routes tasks into five sections — Due Soon (Today / Tomorrow), Upcoming, Future, Unscheduled, and Done — based on due date
- Parses natural language dates ("next friday", "apr 25") so you never touch a date picker
- Supports priority levels (P1–P3) that sort tasks within each section
- Multi-select tasks with ⌘+click for bulk date, priority, or delete actions
- Full undo/redo across every mutation — create, edit, delete, complete, and bulk actions
- Full-text search, drag-and-drop rescheduling, and a complete keyboard shortcut system (`C` new task, `/` search, `⌘Z` undo)

### Stack
Vanilla JS · Supabase (Postgres + Auth) · Vercel · Resend

### Key decisions
- No framework, no build step — the entire frontend is a single `index.html`. Kept it simple on purpose; the interesting problems were product and UX, not bundler config
- Supabase Row Level Security enforces per-user data isolation at the DB layer, so the anon key is safe to ship in client code
- localStorage as an optimistic cache — mutations write locally first for instant render, then sync to Supabase async
- Done section shows only today's completions so it doesn't become a graveyard; older completed tasks are hidden until revisited
- Mobile layout is a fully separate UX: bottom-sheet modal, floating `+` FAB, always-visible edit/delete buttons, drag-and-drop disabled on touch
