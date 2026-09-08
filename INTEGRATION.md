# Donation page - integration contract

This repo currently serves the live donation page at the apex of
`masjidattarbiya.org` (GitHub Pages, `main` branch root). The plan is to move it
to `masjidattarbiya.org/donate` as a route inside the main masjid website, so
the apex can serve the main site.

Read this before merging. Payments are live: donors' money moves through these
links today, so a careless cutover breaks real donations.

## Non-negotiables

1. **The donation route must stay self-contained.** `index.html` has no build
   step, no framework, and no dependencies beyond two Google Fonts. Keep it that
   way when it becomes `/donate`. If the main site's framework breaks, donations
   must still work.
2. **`?thanks=` must keep working on whatever URL the page ends up on.** See
   below - this is hardwired into GoCardless in 22 places.
3. **Amounts and links are not editable by guesswork.** Every `pay.gocardless.com/BRT...`
   link in `index.html` was verified by opening its live payment page and
   confirming amount, type (one-off vs recurring), and that no unwanted Direct
   Debit mandate is bundled. Never add or swap a link without doing the same.

## The `?thanks=` contract

After a donor pays, GoCardless redirects them back to the donation page with a
query parameter. The page reads it and shows a thank-you banner:

| Parameter | Shown to donor |
|---|---|
| `?thanks=oneoff` | "Thank you for your donation..." |
| `?thanks=monthly` | "Your monthly donation is all set up..." |
| `?thanks=1` (legacy) | Neutral fallback wording |

All 22 active GoCardless templates currently redirect to
`https://masjidattarbiya.org/?thanks=oneoff` or `...?thanks=monthly` - i.e. the
**apex**, not `/donate`.

**So the cutover has a hard dependency:** the moment the apex stops serving the
donation page, every donor who completes a payment lands on the main site's
homepage instead of a thank-you, unless one of these is done:

- **Option A (preferred):** update all 22 templates to point at
  `https://masjidattarbiya.org/donate?thanks=...` - see script below.
- **Option B (safety net):** the main site's homepage detects a `?thanks=`
  parameter and redirects to `/donate?thanks=...`, preserving the value.

Doing both is cheapest insurance. Option B alone leaves a permanent oddity in
the main site; Option A alone fails for anyone mid-payment during the cutover.

## Updating the GoCardless redirects

The dashboard has **no edit option** for existing templates - only "Copy link"
and "Delete". Recreating them changes the `BRT...` share links, which would
break the site. The API can update them in place, keeping the same links:

```
GET  https://api.gocardless.com/billing_request_templates?limit=40
PUT  https://api.gocardless.com/billing_request_templates/{BRT_id}
     body: {"billing_request_templates":{"redirect_uri":"https://masjidattarbiya.org/donate?thanks=oneoff"}}
     headers: Authorization: Bearer <token>, GoCardless-Version: 2015-07-06
```

Templates whose name contains "Monthly" get `?thanks=monthly`; the rest get
`?thanks=oneoff`. Requires a read-write API token from the GoCardless dashboard
(Developers > Create > Access token). **Revoke the token immediately after** -
it is a live credential for the charity's payment account.

Note there are also 4 abandoned one-off templates (created 24 Jul 2026, shown in
the dashboard as "Pay by Bank + DD Set up") that are deliberately not linked from
the site - they wrongly bundle a recurring mandate with a one-off payment. They
should be deleted, not updated.

## Cutover order

1. Merge the donation page into the main site at `/donate` and deploy it, while
   the apex still serves the current page. Both work simultaneously.
2. Verify `/donate` end-to-end on a phone: every amount button opens the right
   GoCardless page, and `?thanks=oneoff` / `?thanks=monthly` render correctly.
3. Update the 22 GoCardless redirects to the `/donate` URL, and add the
   homepage safety-net redirect.
4. Only then point the apex at the main site.
5. Re-test a real £5 donation end to end, both one-off and monthly.

## Assets and brand

| File | Purpose |
|---|---|
| `index.html` | The whole donation page - markup, styles, and link map |
| `logo.png` | Full wordmark logo (4452x2186, transparent) |
| `favicon.png` | Square mosque emblem, 512x512, cropped from the logo |
| `apple-touch-icon.png` | 180x180 on white, for iOS home screen |
| `CNAME` | GitHub Pages custom domain |

Brand tokens sampled from the logo - the main site should match these:

- Teal `#42BAC5`, deep teal `#17808A`, ink teal `#0E4A50`
- Gold `#D4AE61`, deep gold `#B08D3F`
- Headings: Marcellus. Body: Inter.
- Prose style: plain hyphens, never em dashes.

## Charity details (must appear on the donation page)

- Attarbiya Masjid & Kowneyn Community Centre
- Registered Charity No. **1142204** - linked to the Charity Commission register
- 2 Revesby Walk, Nechells, Birmingham B7 4LG
- 07908 854187 / info@masjidatarbiya.org

The charity's registered website on the Charity Commission is still the
abandoned `kowneyn.org`. Trustees should update it to `masjidattarbiya.org` -
that listing is the strongest available defence against fake donation pages.
