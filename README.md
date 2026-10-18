# Beer Binder

Mobile-first beer-collecting app: stamp beers into a digital binder, flip Top Trumps-style cards, earn badges, compete on a demo leaderboard, and preview a printed book.

Single static file (`index.html`), no build step, no dependencies.

## Run locally
Open `index.html` in a browser, or run `python3 -m http.server` in this folder and visit http://localhost:8000.

## Deploy on GitHub
1. Create a repo, upload `index.html` and `README.md`.
2. Settings → Pages → Deploy from branch → `main` / root.

## Works now (local only)
Binder (grid and open-book views), filters, sorting, add-beer flow with camera/gallery photo, duplicate handling, manual beer submission (pending review, no points), card flip, rarity tiers, points, 18 badges, seasonal challenge, demo friends and leaderboards, compare, print-book preview with GBP price, draft saving, onboarding with demo mode. Data is saved in `localStorage`.

## Needs an external service
- Accounts, shared groups, live leaderboards, invites: backend (e.g. Supabase). Groups here are **not** synced between devices.
- Cloud photo storage.
- Photo recognition and a real beer database (the sample beers are fictional).
- Print supplier and payment provider. The checkout is a labelled demo; no orders or payments happen.

## Tuning
Scoring: `BONUS` and `PTS` constants. Badges: `BD`. Seasonal challenges: `SEASON`. Beers: `RAW`.

## Suggested Supabase schema
```sql
create table profiles(id uuid primary key references auth.users, username text unique, bio text, visibility text default 'friends');
create table beers(id uuid primary key default gen_random_uuid(), name text, brewery text, style text, country text, abv numeric, rarity smallint check(rarity between 0 and 4), status text default 'pending');
create table collected_beers(user_id uuid references profiles, beer_id uuid references beers, first_at date, primary key(user_id, beer_id));
create table memories(id uuid primary key default gen_random_uuid(), user_id uuid, beer_id uuid, on_date date, place text, companions text, note text, photo_path text);
create table groups(id uuid primary key default gen_random_uuid(), name text, invite_code text unique, owner uuid);
create table group_members(group_id uuid references groups, user_id uuid references profiles, primary key(group_id,user_id));
create table badges(id text primary key, name text, criteria jsonb);
create table user_badges(user_id uuid, badge_id text references badges, earned_at timestamptz default now(), primary key(user_id,badge_id));
create table point_events(id bigserial primary key, user_id uuid, reason text, points int, created_at timestamptz default now());
create table print_orders(id uuid primary key default gen_random_uuid(), user_id uuid, config jsonb, status text default 'draft', total_gbp numeric);
```
Enable row-level security on every table. Never put the service-role key in frontend code; award points and verify rarity server-side.

## Next steps
1. Add Supabase auth, tables and storage so groups and leaderboards sync.
2. Connect a verified beer database and optional image matching (user always confirms).
3. Integrate a print-on-demand supplier and Stripe, then replace the demo checkout.
