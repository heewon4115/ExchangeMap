# Exchange Map

Simple static prototype for testing whether international students click map pins, country filters, student review cards, and the review button.

## Open locally

Double-click `index.html` to open it in your browser. The map uses Leaflet and OpenStreetMap, so the browser needs internet access to load map tiles.

## Add Google Analytics 4

Open `index.html` and find this line near the top of the JavaScript:

```js
const GA_MEASUREMENT_ID = 'G-XXXXXXXXXX';
```

Replace `G-XXXXXXXXXX` with your real GA4 Measurement ID, for example:

```js
const GA_MEASUREMENT_ID = 'G-ABC1234567';
```

If you leave the placeholder value, the page still works. Events are printed to the browser console instead of being sent to GA4.

Google Analytics status is not shown anywhere in the UI. It is only in the code and browser console so normal users do not see analytics/testing details.

## Connect Supabase community posts

Community tips, comments, and recommendation counts can be shared publicly through Supabase. Open `index.html` and check these constants near the top of the JavaScript:

```js
const SUPABASE_URL = 'https://your-project-ref.supabase.co';
const SUPABASE_PUBLISHABLE_KEY = 'sb_publishable_...';
```

The publishable key is safe to use in the browser when Row Level Security policies are enabled. Do not put a Supabase `service_role` key in this static site.

Run this SQL in Supabase SQL Editor before testing shared community posts:

```sql
create table community_tips (
  id uuid primary key default gen_random_uuid(),
  board text not null default 'info',
  author text not null default 'Anonymous student',
  title text not null,
  body text not null,
  recommends integer not null default 0,
  delete_code_hash text,
  created_at timestamptz not null default now()
);

create table community_comments (
  id uuid primary key default gen_random_uuid(),
  tip_id uuid references community_tips(id) on delete cascade,
  author text not null default 'Anonymous student',
  body text not null,
  created_at timestamptz not null default now()
);

create table if not exists community_post_comments (
  id uuid primary key default gen_random_uuid(),
  post_id text not null,
  board text not null default 'info',
  author text not null default 'Anonymous',
  body text not null,
  created_at timestamptz not null default now()
);

create table if not exists community_posts (
  id text primary key,
  board text not null default 'info',
  title text not null,
  body text not null default '',
  author text not null default 'Exchange Map',
  source text not null default 'app',
  recommends integer not null default 0,
  created_at timestamptz not null default now(),
  last_seen_at timestamptz not null default now()
);

alter table community_tips enable row level security;
alter table community_comments enable row level security;
alter table community_post_comments enable row level security;
alter table community_posts enable row level security;

create policy "Anyone can read tips"
on community_tips for select
using (true);

create policy "Anyone can create tips"
on community_tips for insert
with check (true);

create policy "Anyone can recommend tips"
on community_tips for update
using (true)
with check (true);

drop policy if exists "Anyone can delete tips with password filter"
on community_tips;
create policy "Anyone can delete tips with password filter"
on community_tips for delete
using (true);

create policy "Anyone can read comments"
on community_comments for select
using (true);

create policy "Anyone can create comments"
on community_comments for insert
with check (true);

drop policy if exists "Anyone can read board post comments" on community_post_comments;
create policy "Anyone can read board post comments"
on community_post_comments for select
using (true);

drop policy if exists "Anyone can create board post comments" on community_post_comments;
create policy "Anyone can create board post comments"
on community_post_comments for insert
with check (true);

drop policy if exists "Anyone can read board posts" on community_posts;
create policy "Anyone can read board posts"
on community_posts for select
using (true);

drop policy if exists "Anyone can create board posts" on community_posts;
create policy "Anyone can create board posts"
on community_posts for insert
with check (true);

drop policy if exists "Anyone can update board posts" on community_posts;
create policy "Anyone can update board posts"
on community_posts for update
using (true)
with check (true);

create or replace function delete_community_tip(
  p_tip_id uuid,
  p_delete_code_hash text
)
returns boolean
language plpgsql
security definer
set search_path = public
as $$
begin
  delete from community_tips
  where id = p_tip_id
    and delete_code_hash = p_delete_code_hash;

  return found;
end;
$$;

grant execute on function delete_community_tip(uuid, text) to anon, authenticated;
```

If you already created the Supabase tables before board post comments were added, run this once:

```sql
create table if not exists community_posts (
  id text primary key,
  board text not null default 'info',
  title text not null,
  body text not null default '',
  author text not null default 'Exchange Map',
  source text not null default 'app',
  recommends integer not null default 0,
  created_at timestamptz not null default now(),
  last_seen_at timestamptz not null default now()
);

create table if not exists community_post_comments (
  id uuid primary key default gen_random_uuid(),
  post_id text not null,
  board text not null default 'info',
  author text not null default 'Anonymous',
  body text not null,
  created_at timestamptz not null default now()
);

alter table community_post_comments enable row level security;
alter table community_posts enable row level security;

drop policy if exists "Anyone can read board posts" on community_posts;
create policy "Anyone can read board posts"
on community_posts for select
using (true);

drop policy if exists "Anyone can create board posts" on community_posts;
create policy "Anyone can create board posts"
on community_posts for insert
with check (true);

drop policy if exists "Anyone can update board posts" on community_posts;
create policy "Anyone can update board posts"
on community_posts for update
using (true)
with check (true);

drop policy if exists "Anyone can read board post comments" on community_post_comments;
create policy "Anyone can read board post comments"
on community_post_comments for select
using (true);

drop policy if exists "Anyone can create board post comments" on community_post_comments;
create policy "Anyone can create board post comments"
on community_post_comments for insert
with check (true);
```

If you already created the table before adding deletion passwords, run this once:

```sql
alter table community_tips
add column if not exists delete_code_hash text;

drop policy if exists "Anyone can delete tips with password filter"
on community_tips;
create policy "Anyone can delete tips with password filter"
on community_tips for delete
using (true);

create or replace function delete_community_tip(
  p_tip_id uuid,
  p_delete_code_hash text
)
returns boolean
language plpgsql
security definer
set search_path = public
as $$
begin
  delete from community_tips
  where id = p_tip_id
    and delete_code_hash = p_delete_code_hash;

  return found;
end;
$$;

grant execute on function delete_community_tip(uuid, text) to anon, authenticated;
```

If Supabase is not ready or the tables are missing, the app falls back to browser `localStorage`.

## Deploy without Vercel

Use Netlify Drop:

1. Go to `https://app.netlify.com/drop`.
2. Drag the project folder containing `index.html` and `README.md` onto the page.
3. Netlify will give you a public website URL.
4. Open the URL and test the map, filters, pins, and review button.

## Deploy with Vercel

You can also deploy this as a static site on Vercel:

1. Create a GitHub repository.
2. Upload `index.html`, `README.md`, `package.json`, `vercel.json`, and `.gitignore`.
3. Go to `https://vercel.com/new`.
4. Import the GitHub repository.
5. Use these settings:
   - Framework Preset: Other
   - Build Command: leave empty, or use `npm run build`
   - Output Directory: `.`
6. Click Deploy.

After each update, push the changed files to GitHub and Vercel will redeploy automatically.

## Upload to GitHub without Git

If Git is not installed, use GitHub's website:

1. Go to `https://github.com/new`.
2. Repository name: `exchange-map`.
3. Choose Public or Private.
4. Create the repository.
5. Click `uploading an existing file`.
6. Upload these files:
   - `index.html`
   - `README.md`
   - `package.json`
   - `vercel.json`
   - `.gitignore`
7. Commit the files.

## GA events tracked

All events also include common context so GA4 Realtime and Explorations are easier to read:

- `event_group`: map, filter, review, community, navigation, or prototype.
- `analytics_area`: same broad area as `event_group`, useful as a GA4 custom dimension.
- `analytics_action`: click, open, close, select, submit, copy, delete, view, or interact.
- `analytics_object`: review, map_marker, place, filter, language, community_post, community_tip, community_board, location, screen, address, or prototype.
- `funnel_step`: simple prototype journey step such as `01_landing`, `03_discovery`, `04_place_interest`, or `05_engagement`.
- `engagement_type`: browse, preference, create, comment, recommend, copy, or delete.
- `user_intent`: easier human-readable intent such as `place_interest`, `review_intent`, `community_contribution`, or `preference_signal`.
- `conversion_candidate`: yes/no flag for important actions worth reviewing as possible conversion events.
- `context_summary`: short combined label like `map/click/map_marker`.
- `event_category`, `event_label`, `content_group`, `screen_name`, and `item_category`: GA4-friendly report fields derived from the same event context.
- `report_dimension`: compact combined field for quick Explore tables.
- `activity_label`: readable event label.
- `current_tab`, `ui_language`, `country_filter`, `category_filter`, `review_language_filter`, `community_board`.
- `selected_place_id`, `selected_place_name`, and `selected_place_category` when a place is selected.
- `active_filters`: compact snapshot of the current map category, country, feature, and review language filters.
- `visitor_id`, `session_id`, and `event_index` for prototype-level behavior tracing.
- `ga_connected`: yes/no, useful while testing before GA is connected.

Recommended GA4 custom dimensions to register for easier reports:

- `analytics_area`
- `analytics_action`
- `analytics_object`
- `funnel_step`
- `engagement_type`
- `user_intent`
- `conversion_candidate`
- `community_board`
- `selected_place_category`
- `active_filters`
- `report_dimension`

In GA4 Realtime or Explore, start by grouping events by `analytics_area`, then filter by `funnel_step` or `engagement_type` to quickly see what users are doing.

For DebugView testing, the prototype sends `debug_mode: yes` with each event. You can turn this off in `index.html` by changing:

```js
const GA_DEBUG_MODE = false;
```

- `prototype_view`: fires when the page loads with page title and current URL.
- `qr_landing`: fires when the URL has UTM parameters.
- `tab_click` and `screen_view`: fire when users move between Map, Community, and My Log.
- `search_filter_toggle`: fires when the map search/filter panel opens or closes.
- `category_filter_click`: fires when a category filter is clicked.
- `language_select`: fires when a language button is clicked.
- `ui_language_select`: fires when ENG, CH, or KOR is clicked.
- `country_filter_click`: fires when a country filter is clicked.
- `marker_click`: fires when a map marker is clicked.
- `map_clear_click`: fires when the map is tapped to clear the selected place or close filters.
- `place_list_click`: fires when the selected place card is opened as a detail page.
- `place_detail_open`: fires when the detail page opens.
- `review_panel_open`: fires when the place detail/review panel opens.
- `review_cta_click`: fires when the Leave a review button is clicked.
- `review_recommend_click`: fires when a student recommendation button on a review is clicked.
- `review_submit`: fires when the prototype review modal is submitted.
- `review_modal_close`: fires when the review modal is closed.
- `community_board_click`: fires when users switch between information, social, travel, and Q&A boards.
- `community_tip_submit`: fires when users share a tip in the Community tab.
- `community_tip_recommend_click`: fires when users recommend a community tip.
- `community_tip_comment_submit`: fires when users comment on a community tip.
- `community_tip_delete_click`: fires when users try to delete a community tip.
- `community_post_recommend_click`: fires when users recommend a built-in Community board or guide post.
- `address_copy_click`: fires when users copy a place address.
- `current_location_prompt` and `current_location_click`: fire for location permission/result testing.
- `add_record_click`: fires when users go to add a new record.
- `feature_placeholder_open`: fires when users open a feature that is not implemented yet.
- `feature_placeholder_back_click`: fires when users go back from a not-yet-built feature page.
- `feature_expectation_click`: fires when users tap the thumbs-up button for a not-yet-built feature. This is the best event for checking which future features people want.

## Add real places, pins, and reviews

Open `index.html` and find the `places` array. Each item in that array creates one map pin and one place detail page.

To add a real place:

1. Copy one existing place object inside the `places` array.
2. Change `id`, `name`, `category`, `latitude`, `longitude`, `address`, `openingHours`, `phone`, `website`, `badges`, `tags`, and `reviews`.
3. Use Google Maps, Naver Map, Kakao Map, or OpenStreetMap to find the latitude and longitude.
4. Keep `id` unique, for example `snu-dental-clinic`.

To add a real review, add an item inside that place's `reviews` array:

```js
{
  id: 'unique-review-id',
  name: 'Linh Tran',
  country: 'Vietnam',
  school: 'Seoul National University',
  language: 'English',
  rating: 4.5,
  date: 'May 2026',
  recommends: 0,
  text: 'Short practical review from a student.'
}
```

Use `school` for the Korean university where the student is studying now, not their original home university.

This prototype saves submitted place reviews in the current browser using `localStorage`. Community tips, community comments, community recommendation counts, and deletion password hashes are saved to Supabase when the database tables are available, with `localStorage` as a fallback.

## Example QR URL

Add this to the end of your website URL when making a QR code:

```text
?utm_source=poster&utm_medium=qr&utm_campaign=dorm_test_20260529
```
