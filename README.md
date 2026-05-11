# mytodo.page

A keyboard-first personal task tracker. Tasks auto-sort into urgency buckets using natural language processing and sync across devices. 

**Live site → [mytodo.page](https://mytodo.page)**

<img width="1437" height="859" alt="image" src="https://github.com/user-attachments/assets/00a4d02a-1cc3-4fe2-9458-b859ac3fc1c5" />

### What it does
- Automatically routes tasks into five sections: Due Soon (Today / Tomorrow), Upcoming, Future, Unscheduled, and Done, based on due date
- Parses natural language dates ("next friday", "apr 25") 
- Supports priority levels (P1–P3) that sort tasks within each section
- Multi-select tasks with ⌘+click for bulk date, priority, or delete actions
- Full undo/redo across every mutation — create, edit, delete, complete, and bulk actions
- Full-text search, drag-and-drop rescheduling, and a complete keyboard shortcut system (`C` new task, `/` search, `⌘Z` undo)
<img width="551" height="514" alt="image" src="https://github.com/user-attachments/assets/7c076e08-1653-476a-b4cc-ca5aae518054" />

### Why I built this
I took on this project as a learning exercise on how to prototype & build a full stack app using Claude Code. I wanted to build something that would solve an actual problem that I experience, and ultimately landed on task tracking because I feel like I always have dozens of things to track across a number of different contexts & urgencies. I wanted to build a tool that would help me keep track of those things and focus on the things that mattered most & when they mattered most.

### Stack
Vanilla JS · Supabase (Postgres + Auth) · Vercel · Resend

### Key decisions
- No framework or build step needed, entire frontend is a single index.html. Wanted to keep things fast and simple to focus on the product & UX.
- localStorage as an optimistic cache — mutations write locally first for instant render, then sync to Supabase async
- Done section shows only today's completions so it doesn't become a graveyard; older completed tasks are hidden until revisited
- v1 mobile layout: bottom-sheet modal, floating `+`, always-visible edit/delete buttons, drag-and-drop disabled on touch
