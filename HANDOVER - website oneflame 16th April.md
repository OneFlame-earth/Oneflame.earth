# ONE FLAME — MASTER HANDOVER DOCUMENT
**Date:** April 2026  
**Author:** Claude (Anthropic)  
**For:** Next AI session continuing this project

---

## WHO YOU ARE TALKING TO

Michael C. British entrepreneur based in Pitesti, Romania. Owner of Better Connections Ltd (energy consultancy, Congleton UK). Founder of One Flame (oneflame.earth). Works from iPhone 16 Pro Max. Communication: direct, fast, British English. No em dashes. No semicolons. No hashtags. No filler. No excessive explanation.

---

## WHAT ONE FLAME IS

A global peace initiative. People pay £1.78 (currently still £0.79 on Stripe — price change is LAST thing to do) to place a named candle on a live world map. The candle burns for 7 days. Relighting is free forever — the candle is theirs permanently. One fee. For life.

**The five pillars:** Peace, Hope, Health, Love, For the children.

**No end date.** The World Cup and FIFA references have been completely removed from all pages. This is a permanent campaign.

**TikTok:** @oneflame.earth — 1,442 followers, 3.2M+ views. Bio link should point to `https://oneflame.earth/link.html`.

---

## TECH STACK

| Item | Value |
|------|-------|
| Hosting | GitHub Pages — repo `OneFlame-earth/Oneflame.earth` |
| Branch for dev | `staging` (merges to `main` to go live) |
| Database | Supabase — `https://rytjvdptbjnlhayclcvu.supabase.co` |
| Supabase anon key | `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InJ5dGp2ZHB0YmpubGhheWNsY3Z1Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzUzMzA1MjksImV4cCI6MjA5MDkwNjUyOX0.y6qVpN-tJKafDVY_yUk0K6yQ9xXYy5xfIhFoJvqWf48` |
| Payments | Stripe — payment link `https://buy.stripe.com/cNi9AU0At8DecDL7rPeME04` (currently £0.79) |
| Stripe webhook | `smooth-function` edge function — handles `checkout.session.completed`, sets `payment_confirmed=true`, resets `expires_at` |
| Stripe success URL | `https://oneflame.earth/welcome.html?code={CHECKOUT_SESSION_CLIENT_REFERENCE_ID}` |
| Domain | oneflame.earth (Namecheap) |
| Email | hello@oneflame.earth |
| Map library | D3 v7 + TopoJSON — Natural Earth projection |
| Fonts | Playfair Display, Source Serif 4, Inter (Google Fonts) |

---

## SUPABASE TABLE: `candles`

### Columns confirmed to exist:
- `id` (uuid)
- `code` (text) — 5-char alphanumeric, e.g. 9AMBJ
- `nickname` (text)
- `latitude` (float)
- `longitude` (float)
- `location_name` (text)
- `intention` (text)
- `payment_confirmed` (boolean)
- `stripe_session_id` (text)
- `created_at` (timestamptz)
- `expires_at` (timestamptz)
- `email` (text) — **added April 2026**
- `pin` (text) — **added April 2026**

### RLS Policies (confirmed in Supabase):
- SELECT: `payment_confirmed = true` for public
- INSERT: `payment_confirmed = false` for anon
- UPDATE: `USING (true) WITH CHECK (true)` for anon — **may need fixing, see OUTSTANDING ISSUES**

---

## FOUNDING CANDLE

**Code: 9AMBJ**  
**Name: Jem**  
**Location: Congleton, Cheshire, United Kingdom**  
This is the only real external purchase. Permanent. Never expires. Special treatment across all pages — green star badge, "Founding candle" label, never shown as expired, sits at top of flames.html above all other candles.

---

## ALL PAGES — CURRENT STATUS

All pages have been rebuilt in a new warm/light design (cream, parchment, amber). The map remains dark. The home page transitions dark-to-light on scroll. All files exist as `_new.html` — rename by removing `_new` when pushing to GitHub.

| File | Status | Notes |
|------|--------|-------|
| `index_new.html` → `index.html` | ✅ Done | Home page. Dark hero, amber-to-cream transition, 5 pillars, founder message, how it works, CTA |
| `map_new.html` → `map.html` | ✅ Done | Natural Earth projection, dark navy ocean, hover tooltips, ghost dots (tiny cream), PIN relight, zoom-to-candle with retry |
| `flames_new.html` → `flames.html` | ✅ Done | Warm cream, Jem at top with founding badge, search, map/cert/share per row |
| `receipts_new.html` → `receipts.html` | ✅ Done | "Our Story" — founder message prominent, pillars, charity receipts |
| `certificate_new.html` → `certificate.html` | ✅ Done | Search by code, warm parchment certificate card, print/share |
| `countries_new.html` → `countries.html` | ✅ Done | Ranked list with progress bars, gold/silver/bronze top 3 |
| `accounts_new.html` → `accounts.html` | ✅ Done | Full financial transparency, receipt PDF links, correct milestones, charities page link |
| `charities_new.html` → `charities.html` | ✅ Done | Cost waterfall + 89 charities from Excel, link buttons per row for future receipts |
| `terms_new.html` → `terms.html` | ✅ Done | Light warm design |
| `privacy_new.html` → `privacy.html` | ✅ Done | Light warm design |
| `cookies_new.html` → `cookies.html` | ✅ Done | Light warm design |
| `welcome_new.html` → `welcome.html` | ⚠️ See issues | Post-payment page — PATCH still unreliable |
| `link_new.html` → `link.html` | ✅ Done | TikTok entry — detects TikTok browser, shows instructions or button |

### PDFs to include in repo root:
- `RNLI_receipt.pdf`
- `TeamRubicon_receipt.pdf`

---

## OUTSTANDING ISSUES — PRIORITY ORDER

### 1. CRITICAL: welcome.html PATCH not saving details (UNSOLVED)

**Symptom:** After payment, user fills in name/country/intention/PIN on welcome.html. Clicking save appears to work but the candle stays Anonymous with placeholder ocean coordinates in Supabase.

**What we know:**
- Payment confirms correctly (payment_confirmed = true)
- The candle row EXISTS in Supabase after payment
- The `pin` and `email` columns have been added
- The RLS UPDATE policy shows `USING(true) WITH CHECK(true)` — should allow updates
- The PATCH returns `Prefer: return=representation` now — will log to console
- Both full PATCH and core-fields fallback PATCH are attempted

**Most likely remaining cause:**
The Supabase RLS UPDATE policy `USING(true)` may not be effective because the anon key's SELECT policy (`payment_confirmed = true`) gates what rows PATCH can "see". If the candle reaches welcome.html before `payment_confirmed` is set to true by the webhook, the row is invisible to the anon key's UPDATE.

**Fix to try next:**
Run this SQL in Supabase:
```sql
DROP POLICY IF EXISTS "Anyone can update own candle" ON candles;
CREATE POLICY "Anyone can update own candle"
ON candles FOR UPDATE TO anon
USING (true)
WITH CHECK (true);
```

Then check browser console on welcome.html after saving — the new code logs `[welcome] PATCH status:` and the full response body. Share that log output to diagnose further.

**Alternative fix if RLS persists:**
Use the Supabase service role key (not the anon key) for the PATCH only. The service role bypasses RLS entirely. This is safe for this operation because the PATCH only updates the row matching the exact code the user just paid for.

Service role key location: Supabase Dashboard → Settings → API → `service_role` key.
Replace only the PATCH fetch headers with: `'Authorization': 'Bearer SERVICE_ROLE_KEY'`

### 2. Map zoom after welcome.html

**Symptom:** After completing the welcome.html form, tapping "See my candle" opens map.html but doesn't zoom to the candle.

**Fix already in map_new.html:** `zoomToCandle` now retries up to 5 times at 2-second intervals. Should work once PATCH issue is resolved and coordinates are saved correctly.

### 3. charities.html not in burger nav on all pages

The burger nav on all pages doesn't include charities.html yet. Needs adding to the nav drawer `<ul>` in every HTML file:
```html
<li><a href="charities.html" class="nav-link">💝 Charities</a></li>
```

### 4. Price change (LAST — do after everything else works)

- Site shows £1.78 everywhere
- Stripe payment link still charges £0.79
- When ready: create new Stripe payment link at £1.78, update `STRIPE_LINK` constant in `map_new.html`, `link_new.html`, `index_new.html` and `map_new.html`
- Update Stripe product description: "One fee. For life. Relight anytime free. Your candle with certificate. £1.78."

### 5. Languages (LAST — do after price change)

All pages are English only. 32 language versions to be added last. The old index.html had the full translation system — copy the T object and setLang() function from the old codebase.

---

## PAYMENT FLOW — HOW IT SHOULD WORK

```
link.html 
  → generates code (5-char)
  → inserts placeholder row (Anonymous, random ocean coords, payment_confirmed=false)
  → redirects to Stripe with ?client_reference_id=CODE
  
Stripe (Apple Pay / card)
  → webhook fires smooth-function
  → smooth-function sets payment_confirmed=true, resets expires_at to now+7days
  → Stripe redirects to welcome.html?code=CODE
  
welcome.html
  → polls Supabase every 5s until payment_confirmed=true
  → shows "continue anyway" after 20s if still waiting
  → shows details form: nickname, country, hometown, intention, PIN, email
  → on save: PATCHes candles row with all fields
  → on success: shows done screen with map/cert/share links
  
map.html?candle=CODE
  → zooms to candle location
  → shows detail panel
```

---

## CANDLE DATA SITUATION

All test candles should be deleted. Only `9AMBJ` (Jem, Congleton) and real paying customers should remain.

**SQL to clean test data:**
```sql
DELETE FROM candles 
WHERE nickname ILIKE '%michael%'
OR (nickname = 'Anonymous' AND location_name = '')
AND code NOT IN ('9AMBJ');
```

**Do NOT delete:**
- 9AMBJ (Jem — founding candle)
- Any row with a real location_name and non-Anonymous nickname from real paying customers

---

## CHARITY MILESTONES

| Milestone | Charity | Amount | Status |
|-----------|---------|--------|--------|
| Launch | RNLI Cornwall | £50 | ✅ Paid — receipt: `RNLI_receipt.pdf` |
| Launch | Team Rubicon | $67.78 (≈£50) | ✅ Paid — receipt: `TeamRubicon_receipt.pdf` |
| 1,000 candles | RNLI | £125 | Upcoming |
| 1,000 candles | Team Rubicon | £125 | Upcoming |
| 10,000 candles | TBD by Michael | TBD | Upcoming |

Future charities: full list of 89 organisations in `charities.html` and `Charities_-_Master.xlsx`.

---

## DAILY PUSH TO MAIN (instructions for Michael)

To push staging changes to main each day:
1. Go to GitHub repo `OneFlame-earth/Oneflame.earth`
2. Click **Pull requests** → **New pull request**
3. Base: `main` ← Compare: `staging`
4. Click **Create pull request** → **Merge pull request** → **Confirm merge**
5. GitHub Pages rebuilds automatically — live in ~60 seconds

---

## DESIGN SYSTEM

| Token | Value | Use |
|-------|-------|-----|
| `--amber` | `#C8963E` | Primary gold — buttons, links, accents |
| `--amber-lt` | `#E8B86D` | Lighter gold — hover states, italic text |
| `--ember` | `#E05A1B` | Orange-red — share button, urgency |
| `--coal` | `#0D0A06` | Near-black — map background, dark hero |
| `--cream` | `#FBF7F0` | Warm white — light page backgrounds |
| `--parchment` | `#F5ECD8` | Warm yellow-white — cards, stat strips |
| `--text-dk` | `#1C1208` | Near-black — body text on light pages |
| `--text-mid` | `#5C4A2E` | Warm brown — secondary text |
| `--text-lt` | `#9A8570` | Muted — labels, metadata |
| `--green` | `#3D7A5A` | Paid/confirmed — Jem, receipt badges |

**Fonts:** Playfair Display (headings, italic), Source Serif 4 (body text), Inter (UI elements)

**Map colours:** Ocean `#0D1B2A` (deep navy), Land `#1C2814` (dark forest green), Borders `#3A5028`

---

## WHAT WORKS WELL

- Map: Natural Earth projection, hover tooltips, ghost dots, PIN-protected relight popup, city labels at deep zoom only (k≥25), zoom-to-candle from flames.html
- Flames.html: Jem at top with founding badge, search, all buttons work
- All light pages: warm cream/parchment design, readable, human
- Payment flow up to welcome.html: confirmed working
- Charity transparency: real receipts linked, correct milestones
- TikTok detection on link.html: working

## WHAT DOES NOT WORK YET

- welcome.html PATCH (saving nickname/country/pin after payment) — primary outstanding issue
- Map zoom after welcome.html form completion — depends on PATCH fix
- charities.html not in burger nav on all pages
- Price not yet £1.78 on Stripe
- Languages not yet added

---

*End of handover. Good luck.*
