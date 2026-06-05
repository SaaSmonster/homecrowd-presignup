# HomeCrowd — Pre-Launch Landing Page

A single-screen email/phone capture page for the HomeCrowd validation test. One file
(`index.html`), no backend, no build step. Signups write straight into a Google Sheet.

## Go live in ~10 minutes

### 1. Set up the Google Sheet

1. Create a Google Sheet named **`HomeCrowd Signups`**.
2. Rename the first tab to **`Signups`**.
3. Add this header row in row 1: `timestamp | email | phone | source`
   (A1 `timestamp`, B1 `email`, C1 `phone`, D1 `source`).

### 2. Add the Apps Script

1. In the Sheet: **Extensions → Apps Script**.
2. Delete anything in `Code.gs` and paste this:

   ```javascript
   function doPost(e) {
     var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('Signups');
     var data = JSON.parse(e.postData.contents);
     sheet.appendRow([new Date(), data.email || '', data.phone || '', data.source || '']);
     return ContentService
       .createTextOutput(JSON.stringify({ result: 'success' }))
       .setMimeType(ContentService.MimeType.JSON);
   }
   ```

3. **Deploy → New deployment** → gear icon → **Web app**.
4. **Execute as: Me**, **Who has access: Anyone**.
5. **Deploy**, authorize, and **copy the Web app URL** (ends in `/exec`).

### 3. Wire the URL into the page

In `index.html`, near the bottom `<script>` block, replace the placeholder:

```js
const SCRIPT_URL = "PASTE_YOUR_APPS_SCRIPT_EXEC_URL_HERE";
```

### 4. Background

A looping `assets/background.mp4` is already in place and plays automatically (muted, loops,
behind the navy overlay). To swap it, just replace that file. To use a GIF or still image
instead, set `--bg-gif: url("assets/background.gif");` in `:root`. If no media loads, a
navy→blue gradient shows.

### 5. Publish

Upload the whole folder (`index.html` + `assets/`) to any static host (Netlify drop,
Vercel, GitHub Pages, Cloudflare Pages — all free). That's your Instagram link. Do a test
submit and confirm a row appears in the `Signups` tab.

---

## Things you can tune (edit values in `index.html`)

- **Brand colors** — the `:root` block at the top of the `<style>`. One place for everything.
- **Button color** — `--accent` (fill) and `--accent-text` (label). Currently a warm
  amber/gold with navy text. A few ready-to-paste alternatives are listed in a comment right
  above those variables (electric mint, clean white, bright cyan-blue). Once your background
  is in, a blue/cyan can work too — just check it still contrasts the background.
- **Countdown box** — the `COUNTDOWN` object in the `<script>`. Runs on a repeating cycle:
  it reads `START_SPOTS` (37), slides down to `MIN_SPOTS` (8) over `CYCLE_MS` (45 min), then
  resets to 37 and repeats. It's clock-based, so a refresh always shows the cycle-accurate
  number, and it visibly ticks down while a visitor watches. **Heads up: this is
  *manufactured* urgency, NOT tied to real signups.** Change `CYCLE_MS` to make the reset
  faster/slower. To remove it, delete the `<div class="counter">…</div>` block in the HTML.
- **Desktop layout** — at ≥900px wide the page switches to a two-column layout (hero text
  left, form panel right); below that it's the single-column mobile layout. Breakpoint is the
  `@media (min-width: 900px)` block in the `<style>`.
- **Top flag bar** — light-gray strip; edit the `FLAGS` array in the `<script>` (two-letter
  country codes; flags from flagcdn.com). Scroll speed is the `42s` in `.flagbar-track`.
  Strip color is `--flag-bg`; the blue divider under the wordmark is the `.brandbar`
  `border-bottom`.
- **Video darkness** — the `.bg-overlay` rule. Raise the `rgba(...)` alphas to darken.
- **Background** — replace `assets/background.mp4` to change the video, or set
  `--bg-gif: url("assets/background.gif");` in `:root` for a GIF/still.
- **Source tag** — `SOURCE = "ig-launch"` in the `<script>`. Change per campaign to track
  where signups came from (lands in the sheet's `source` column).

## Fonts

- **Wordmark "HomeCrowd": Regarn Black**, self-hosted from `assets/fonts/Regarn-Black.otf`
  (wired via an `@font-face` rule at the top of the `<style>` block — no external font CDN).
  ⚠️ **Licensing:** Regarn's free distribution is **personal-use only**. This is fine for a
  short validation test, but for a sustained commercial launch confirm you hold the commercial
  license (Craft Supply Co. / Creative Fabrica). If you'd rather sidestep the question, swap
  the wordmark to a free display font — say the word.
- **Everything else: Space Grotesk** (Google Fonts, free for commercial use).

## How it behaves

- **Email** required + format-checked; **phone** optional (collected with a visible consent +
  STOP opt-out line — no texts are sent by this page).
- A hidden **honeypot** field silently drops bot submissions.
- If the network call fails, the visitor **still sees the confirmation** and the error is
  logged to the console — so no signup is lost to a flaky connection. Dedupe the sheet later.
- No redirects, no personal data in the URL (everything is sent in the POST body).

## Notes

- No "World Cup" / "FIFA" wording anywhere — those are policed marks. Copy uses "the
  tournament" / "the matches". Keep it that way if you edit the copy.
- Phone numbers are collected for later use only. **Do not** wire up SMS sending until a
  compliant sender is in place.
