# Bloom Planner — Cloud Sync Setup

This version includes optional Supabase account sign-in and per-user cloud sync. It still saves locally first. Cloud sync begins only after you explicitly upload or download and enable sync on that device.

## 1. Create a Supabase project
1. Visit https://supabase.com and create a project. Keep your database password private.
2. In the Supabase SQL Editor, run the SQL below.
3. In Project Settings / API, copy your Project URL and the publishable key (or legacy anon key). Never use a service-role/secret key in the planner.

```sql
create table if not exists public.bloom_data (
  user_id uuid primary key references auth.users(id) on delete cascade,
  payload jsonb not null default '{}'::jsonb,
  updated_at timestamptz not null default now()
);

alter table public.bloom_data enable row level security;
revoke all on table public.bloom_data from anon, authenticated;
grant select, insert, update on table public.bloom_data to authenticated;

create policy "Users can read their own Bloom data"
on public.bloom_data for select
to authenticated using (auth.uid() = user_id);

create policy "Users can insert their own Bloom data"
on public.bloom_data for insert
to authenticated with check (auth.uid() = user_id);

create policy "Users can update their own Bloom data"
on public.bloom_data for update
to authenticated using (auth.uid() = user_id) with check (auth.uid() = user_id);
```

If you rerun this SQL after creating policies, Supabase may report that a policy already exists; that is okay.

## 2. Test cloud sync before hosting
1. Open `index.html` on your PC.
2. Go to **My Progress → Private cloud sync**.
3. Enter your Supabase Project URL and publishable/anon key, then choose **Save connection settings**.
4. Create an account or sign in. If email confirmation is enabled, confirm the email first.
5. Export a local backup first. On the PC, choose **Upload this device to cloud**. This overwrites any existing cloud copy for that account.

## 3. Host on GitHub Pages (HTTPS)
1. Create/sign in to GitHub at https://github.com.
2. Create a new private or public repository named `bloom-planner` (GitHub Pages availability can depend on account/repository settings).
3. Upload the files inside the `bloom_planner_deadlines` folder to the repository root: `index.html`, `manifest.json`, `sw.js`, `icon-192.png`, `icon-512.png`, and `README.md`.
4. Open repository **Settings → Pages**. Choose **Deploy from a branch**, select the default branch and `/ (root)`, and save.
5. Wait for GitHub Pages to publish, then open the HTTPS URL it provides. Test sign-in and cloud sync there.

## 4. Add it to iPhone
1. Open the published HTTPS URL in Safari on your iPhone.
2. Tap **Share** → **Add to Home Screen** → **Add**.
3. Open Bloom from the new Home Screen icon.
4. In **My Progress → Private cloud sync**, enter the same Supabase URL/key, sign in to the same account, and choose **Download cloud data**. Download replaces the current device's planner data, so export a backup first.

## Privacy and synchronization notes
- Supabase stores the planner JSON in a row owned by your authenticated account. Row Level Security policies restrict access to the matching user ID.
- The publishable/anon key is intended for browser use; security depends on correct Row Level Security. Never paste a service-role key into the planner or publish it.
- After you upload on the first device and download on the second, changes made through the planner's save actions sync automatically while signed in and online. Keep the app open briefly after edits so pending sync can finish.
- This is a simple single-record sync model, not a conflict-resolution system. If you edit on both devices while offline or at the same time, the last successful upload may replace newer data from the other device. Export backups regularly and avoid simultaneous offline editing.
- Supabase email confirmation settings may require you to confirm your email before signing in.
- GitHub Pages hosts the app files publicly even if the repository is private in some account plans; do not put private data or secrets in the repository. Your planner entries are stored in Supabase, not in the hosted HTML.
