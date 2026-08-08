# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A collection of standalone, static HTML **cash-on-delivery (COD) affiliate landing pages** for the Polish market (all copy is in Polish, "płatność za pobraniem" = pay on delivery). Each directory is one self-contained single-product funnel: a long-form sales page with an order form that POSTs the lead to the ISL affiliate network, followed by a thank-you page that fires the conversion event.

There is **no build system, framework, package manager, or test suite** — just HTML/CSS/JS files served as-is. The site is deployed under `https://www.ossaward.org/offer-lp/` (see the absolute `thankyoupage` URL in `fastmower_pl`).

## Running locally

Serve the repo root with any static server so relative `css/`, `images/`, `media/`, and `js/` paths resolve, then open a page:

```bash
python3 -m http.server 8000
# then visit http://localhost:8000/fastmower_pl/  or  /blower_pl/
```

Note: the order form and phone-validation POST to `offers.islaffiliate.com`; those network calls only succeed from an allowed deployed origin, not localhost.

## The funnels

- `fastmower_pl/` — "Fast Mower" cordless lawn mower. Built from a **Mobirise** export: Bootstrap grid + many `css/style_N.css` / `mbr-additional.css` stylesheets. Uses the **shared** `thanks/index.html` as its thank-you page (absolute URL).
- `blower_pl/` — "Powerful Blower" leaf blower. A **WordPress/Elementor "canvas" template** export: a single bundled `css/all.css`, Elementor class soup, inline `<style>` overrides hiding theme chrome. Uses its own local `thank-you.html`.
- `thanks/index.html` — shared generic thank-you page; fires the Google Ads conversion on load.

## The order flow (shared pattern — most important thing to understand)

Every landing page contains the same core order form; only the hidden IDs differ:

```html
<form class="tm-order-form" action="https://offers.islaffiliate.com/forms/html/" method="post">
  name, tel, street-address        <!-- the only three visible fields -->
  <input name="uid" ...>           <!-- affiliate/session UID (same across pages) -->
  <input name="offer" ...>         <!-- per-page offer ID -->
  <input name="lp" ...>            <!-- per-page landing-page ID -->
  <input name="thankyoupage" ...>  <!-- where ISL redirects after submit -->
  <input name="_key" ...>          <!-- per-page signing key -->
  <script src="https://offers.islaffiliate.com/forms/html/js-v2/" async></script>
</form>
```

The `js-v2` remote script wires up submission. When editing/cloning a page, the `offer`, `lp`, and `_key` hidden inputs are the identity of that offer — do not copy them between pages.

Two thank-you strategies exist, and they differ in how order data flows:

- **fastmower_pl**: `thankyoupage` is a static absolute URL to `thanks/index.html`. That page just fires `gtag('event','conversion', { send_to: 'AW-18339059189/...' })` on load.
- **blower_pl**: JS (`updateThankyouURL()` near the bottom of `index.html`) live-builds `thank-you.html?name=...&tel=...&address=...` from the form inputs on every keystroke. `thank-you.html` then reads those query params to render an order summary. It also does inline phone validation via a POST to `offers.islaffiliate.com/forms/validation/phone.php`.

## Analytics / tracking (differs per page — keep them separate)

- `fastmower_pl` + shared `thanks/`: Google Ads gtag **AW-18339059189** (conversion label `CIXcCOvqid4cEPWr36hE`).
- `blower_pl`: Google Tag Manager **GTM-NSB3W8QQ**, GA4 **G-N71T4WQ878**, and an ISL tracking pixel (`offers.islaffiliate.com/pixel/?offer=4093...`).

When cloning a funnel for a new offer, these IDs plus the form's `offer`/`lp`/`_key` are the fields that must be updated for tracking and payout to attribute correctly.

## Editing conventions

- Each funnel is fully self-contained under its own directory with local `css/`, `images/`, `media/`, `js/`. Keep asset paths relative.
- `blower_pl/index.html` is Elementor-generated markup — expect verbose auto-generated class names and IDs; edit the visible copy/inputs rather than trying to refactor the structure.
- All user-facing copy is Polish; preserve language and the COD framing when editing.
