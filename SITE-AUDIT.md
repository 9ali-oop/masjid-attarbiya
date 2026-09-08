# Attarbiya Masjid website - code audit

Audit of `zakadinho-create/masjid-website` (the new main site), 8 September 2026.

## Method and coverage - read this first

The site was reviewed across nine dimensions by independent reviewers, each finding then
re-checked against the actual files before being reported.

- **Fully reviewed and independently verified:** donations and payments.
- **Reviewed, with every key claim re-verified by hand for this report:** factual accuracy,
  launch readiness, accessibility, forms and privacy.
- **NOT reviewed - still outstanding:** responsive/mobile CSS, SEO and social sharing,
  code architecture and maintainability, design-system coherence. These stopped partway
  through on a usage limit. They are not "clean"; they are simply unexamined.

Every finding below was confirmed against the real files. Nothing here is speculative.

## Verdict

This is a real, competent website - nine pages, a working mobile nav, a hero slideshow,
real photography, real event posters, correct social links, and the correct address and
charity number on every page, with no framework or build step to maintain. The palette is
already sampled from the masjid's logo, so it agrees with the live donation page.

It is not launch-ready, but the gap is content and correctness, not craftsmanship.
Roughly a day of focused work closes it.

---

## Blockers - fix before this goes anywhere near the public

### 1. The masjid's phone number cannot be dialled, on all nine pages

`07305 154 7675` is **12 digits**; a UK mobile is 11. The links built from it,
`tel:+4473051547675` and `https://wa.me/4473051547675`, are malformed and will fail.
The live donation site uses `07908 854187`, which is valid.

**Someone must confirm with the masjid which number is correct**, then find-and-replace
across the site. This is the highest-priority item: a donation page whose "call us if in
doubt" number does not work actively undermines the anti-fraud message it sits beside.

### 2. Internal notes to the client are published as visible page text

Nine yellow `placeholder-note` boxes and two blue `draft-note` boxes render to the public on
about, donate, index, madrasah and prayer-times. They include, verbatim:

- "The descriptions below are realistic placeholder text based on how similar Birmingham
  masjids (Green Lane Masjid, Birmingham Qur'an Academy, Masjid Al-Sunnah) describe these
  services." (services.html)
- "the weekly schedule, ages, fees, and teacher names - is placeholder" (madrasah.html)
- "Also using the leaflet's phone number ... the live masjidattarbiya.org site currently
  shows a different number" (donate.html, directly under the anti-fraud trust box)

Publishing text that names other Birmingham masjids as the source of your service copy, and
admits your teacher names are invented, would be genuinely damaging. All must go.

Also public: 25 `draft-content` spans, 6 `needs-content` cells, and 3 literal `[Day / time]`
placeholders sitting inside tables.

### 3. Prayer times are hardcoded and already wrong

index.html publishes a fixed table under the heading **"Today's Prayer Times"**, captured on
9 August 2026. It is stale by roughly an hour now and drifts further every day. The masjid's
previous website became notorious for exactly this failure. prayer-times.html already has the
right approach - a live Masjidbox embed - so the homepage table should be replaced with that
embed or removed.

### 4. Donors pay and see nothing

All 22 GoCardless templates redirect to `masjidattarbiya.org/?thanks=oneoff|monthly`. Nothing
in this site reads that parameter: a search across every HTML and JS file for `thanks`,
`URLSearchParams` or `location.search` returns zero matches. A donor who completes a Direct
Debit lands on an unchanged page with no confirmation. Some will assume it failed and pay twice.

Worse, all 18 payment links carry `target="_blank"`, so even once handled the thank-you would
appear in an orphaned tab. Fix: drop `target="_blank"`, and port the live page's `?thanks=`
handler into `js/main.js`, which every page already loads.

### 5. Two payment-grid errors

- The retired **£2 one-off** tier is still live, using deprecated link
  `BRT01KWFDFAFWT0RAGGFVMPPXKBRY`. Management removed £1 and £2. Delete it.
- The **£50 one-off** tier is missing entirely; the grid jumps £20 to £100. Add
  `BRT01KWFDHPC07KDAG8CJSTEDBAWA`.

With both fixed, each tab holds exactly nine tiers, £5 to £2,000, and the three-column grid
balances. Everything else checks out: all nine monthly links are correct for their labels, and
**none of the four faulty mandate-bundling templates appear anywhere** - which was the single
thing most worth getting right.

### 6. Children's personal data is collected with no privacy notice

`registration.html` embeds a live Google Form collecting children's full names, dates of birth,
home addresses and medical/allergy information. `contact.html` posts to a third-party endpoint,
`https://formspree.io/f/xqpzjjee`.

There is **not one word** about privacy, data protection, GDPR, cookies or safeguarding anywhere
in the site. For a registered charity handling children's health data that is a legal exposure,
not a polish item. Needed before launch:

- A privacy notice page, linked from the footer and from both forms.
- A recorded owner for that Formspree account and that Google Form: which trustee controls them,
  and who can retrieve the data if that person is unavailable.

---

## High priority

- **Brand colours fail accessibility contrast.** Gold `#d4ae61` on white measures **2.09:1** and
  teal `#42bac5` **2.32:1**, against the 4.5:1 standard. Fix without losing the brand: use deep
  teal `#17808a` (4.68:1) for teal *text*; his own `#1a7078` (5.78:1) already passes. Keep the
  bright teal and gold for fills, borders and backgrounds only.
  *For fairness, the live donation page shares this weakness: white on gold `#B08D3F` is 3.12:1,
  passing only because that button text is large and bold. Worth tightening on both.*
- **index.html has no `<h1>`.** Confirmed: every other page has exactly one; the homepage has none.
- **The hero slideshow auto-advances every 6 seconds** with no pause control, and steals keyboard
  focus when it turns. Add a pause control and respect `prefers-reduced-motion`.
- **No `:focus` styles anywhere** (zero occurrences in the stylesheet), and **no `<main>` landmark
  or skip link** on any page. Keyboard users cannot see where they are.
- **The donate Monthly/One-off tabs carry no ARIA state**, so a screen-reader donor gets no signal
  about which giving type is selected.
- **Expired content presented as current:** the madrasah course is headed "This Summer" and ended
  two weeks ago; the events page is headed "Past & Upcoming" with every listing in the past.

## Medium

- The "Direct Debit ... cancel anytime" fineprint also shows under the one-off tab, where it is
  misleading. Split it per panel.
- services.html skips from `h1` to `h3`.
- Footer body text sits at 3.71:1 on the teal footer; the copyright line at 2.91:1.
- The homepage donation call-to-action reads "Set Up Direct Debit", framing giving as monthly-only.

## Not yet examined

Responsive and mobile CSS, SEO and WhatsApp link previews, code architecture and duplication, and
design-system coherence with the donation page. Worth a second pass, particularly the SEO
metadata: WhatsApp is the masjid's main distribution channel, and a missing `og:image` means
shared links paste as bare text with no preview card.

Two things are already visible without that pass. Several images exceed 1MB (the largest is
1.1MB), which will be slow on mobile data. And the header and footer are copy-pasted into all ten
pages, so a single phone-number change means ten separate edits - which is precisely how the wrong
number came to be everywhere.

## What is genuinely good

Worth stating plainly, because the list above is long. Every one of the 34 images has meaningful
alt text. The contact form labels are correctly associated. The mobile nav toggle handles
`aria-expanded` properly. The donate tab panels use the real `hidden` attribute rather than a
CSS-only hide. Every `target="_blank"` already carries `rel="noopener"`. No secrets are committed,
and there are no `innerHTML` or `eval` sinks. The address, charity number, email and social links
are byte-identical across all nine pages.

That is careful work, and the foundation is sound.
