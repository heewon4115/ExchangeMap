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

- `prototype_view`: fires when the page loads with page title and current URL.
- `qr_landing`: fires when the URL has UTM parameters.
- `language_select`: fires when a language button is clicked.
- `country_filter_click`: fires when a country filter is clicked.
- `marker_click`: fires when a map marker is clicked.
- `place_list_click`: fires when the selected place card is opened as a detail page.
- `review_panel_open`: fires when the place detail/review panel opens.
- `review_cta_click`: fires when the Leave a review button is clicked.
- `review_recommend_click`: fires when a student recommendation button on a review is clicked.
- `review_submit`: fires when the prototype review modal is submitted.

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

This prototype saves recommendation clicks only while the page is open. For real permanent reviews and recommendation counts, you will later need a small database such as Firebase or Supabase.

## Example QR URL

Add this to the end of your website URL when making a QR code:

```text
?utm_source=poster&utm_medium=qr&utm_campaign=dorm_test_20260529
```
