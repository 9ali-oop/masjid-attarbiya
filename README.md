# Attarbiya Masjid donation page (moved)

**This page has moved.** Since 7 October 2026 masjidattarbiya.org serves the main masjid website
from [masjidattarbiya-website](https://github.com/9ali-oop/masjidattarbiya-website), and the
donation page is its `donate.html`, with the same GoCardless links. This repo's `index.html` now
only forwards to https://masjidattarbiya.org/donate.html, and its `CNAME` was removed to release
the domain. The original page is in git history; `INTEGRATION.md` keeps the record of how the
payment links were verified.

The donation page for **Attarbiya Masjid & Kowneyn Community Centre**, Nechells, Birmingham.
Registered charity **1142204**.

Live: https://masjidattarbiya.org

## What this is

One HTML page with no build step and no framework. Donors choose one-off or monthly giving and an
amount, then pay through GoCardless. After paying, GoCardless sends them back with
`?thanks=oneoff` or `?thanks=monthly`, and the page shows a thank-you message.

## Files

| File | What |
|---|---|
| `index.html` | the whole page: markup, styles and the payment links |
| `logo.png`, `favicon.png`, `apple-touch-icon.png` | logo and icons |
| `CNAME` | the custom domain for GitHub Pages |
| `INTEGRATION.md` | how the page works with GoCardless, and the plan to move it to `/donate` on the main site |
| `SITE-AUDIT.md` | an audit of the main masjid website |
| `_config.yml` | keeps these notes out of the published site |

## Changing it

Commits to `main` go live through GitHub Pages within a few minutes. Donations are live, so read
`INTEGRATION.md` before changing any payment link.

The main masjid website is in [masjidattarbiya-website](https://github.com/9ali-oop/masjidattarbiya-website).
