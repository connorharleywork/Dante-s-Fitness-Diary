# Dante's Fitness Diary

Premium mobile-first v4 fitness diary for Dante, deployed as a static GitHub Pages app.

## Live app

GitHub Pages should serve `index.html` at:

<https://connorharleywork.github.io/Dante-s-Fitness-Diary/>

Because this repo uses a plain static app, GitHub Pages displays the app directly instead of the README.

## Supabase setup

1. Create a Supabase project.
2. Enable Email/Password in **Authentication → Providers**.
3. Create the diary table:

```sql
create table if not exists public.diary_profiles (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null unique references auth.users(id) on delete cascade,
  diary_data jsonb not null default '{}'::jsonb,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);

alter table public.diary_profiles enable row level security;

create policy "Users can read own diary"
  on public.diary_profiles for select
  using (auth.uid() = user_id);

create policy "Users can insert own diary"
  on public.diary_profiles for insert
  with check (auth.uid() = user_id);

create policy "Users can update own diary"
  on public.diary_profiles for update
  using (auth.uid() = user_id)
  with check (auth.uid() = user_id);
```

4. `index.html` contains the public Supabase URL and anon public key used by the static app.

Only use the Supabase anon public key in the frontend. Never commit a service role key, database password, JWT secret, real backup JSON, or personal data.

### Supabase Authentication URL Configuration

In the Supabase dashboard, check **Authentication → URL Configuration** and set these values so confirmation and reset emails return to the GitHub Pages app instead of localhost:

- **Site URL:** `https://connorharleywork.github.io/Dante-s-Fitness-Diary/`
- **Redirect URLs:**
  - `https://connorharleywork.github.io/Dante-s-Fitness-Diary/`
  - `https://connorharleywork.github.io/Dante-s-Fitness-Diary/**`
  - `https://connorharleywork.github.io/Dante-s-Fitness-Diary/?verified=true`
  - `https://connorharleywork.github.io/Dante-s-Fitness-Diary/?reset=true`

The frontend sign-up flow uses the `?verified=true` redirect, and the password reset flow uses the `?reset=true` redirect.

## Deployment

No build step is required.

1. Commit changes to the default publishing branch.
2. In GitHub, open **Settings → Pages**.
3. Select the branch and `/ (root)` folder.
4. Visit the GitHub Pages URL above.

The app includes a v4 cache/version marker and no service worker, which helps avoid stale cached versions in Huawei Chrome and other mobile browsers.

## Data model notes

- Current app data version: `4`.
- Supabase stores the full diary object in `public.diary_profiles.diary_data` as JSONB.
- `localStorage` is retained as an offline fallback draft cache.
- Supabase auth persists the browser session in `localStorage`, so users stay logged in until they explicitly log out or clear browser data.
- JSON export/import remains available as a safety backup.
- v3 imports are automatically migrated to v4.
- Legacy v3 meal arrays are preserved in `legacyMealLogs` and shown only in a collapsed old meal history section.
- Water tracking is hidden and ignored in new v4 workflows; old imported water values are not used.

## Suggested future improvements

- Add a tiny printable coach report for weekly client check-ins.
- Add optional exercise template presets for Dante's most common lifts.
- Add a manual "sync now" button for reassurance after imports.
- Add screenshot-based visual regression checks before major UI changes.
