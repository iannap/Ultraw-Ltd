# Ultraw — project memory

Handoff written 2026-08-25. Everything below was established in a Cowork session with Ian Popper.
Drop this file at the root of the `Ultraw-Ltd` repo so Claude Code picks it up automatically.

---

## What Ultraw is

A **multi-vendor online marketplace for San Pedro, Belize** (Ambergris Caye), operated by Ultraw Ltd.
Owner: Ian Popper (`iananthonypopper@gmail.com`). Domain: **ultraw.bz**.

Model is Rappi-style goods delivery, **not** food delivery.

**Launch categories:** groceries & minimarkets · retail & general goods · pharmacy & essentials
**Explicitly out of launch scope:** restaurants and prepared food

Plan is to prove demand in San Pedro on the web, then publish a native app.

**Design references the owner named:** Rappi's homepage (search-first, category tiles), Uber Eats' flow.

---

## Stack

| Layer | Choice | Notes |
|---|---|---|
| Host | **EasyWP** (Namecheap managed WordPress) | **Not WP Engine** — see naming trap below |
| CMS | WordPress | |
| Commerce | WooCommerce | Store country Belize, currency **BZD** |
| Marketplace | **Dokan** (Lite for now) | Motta has native Dokan support |
| Theme | **Motta 1.6.5** by UIX Themes + its official child theme | ThemeForest, GPL |
| Builder | **Elementor** + **Motta Addons** (~60 widgets) | Motta's widgets require both |
| Repo | `github.com/iannap/Ultraw-Ltd` | |

### Motta package contents
The ThemeForest download (`Motta-Package/`) contains:
- `motta.zip` — parent theme
- `motta-child.zip` — official child theme (only `style.css` + `functions.php`)
- `Plugins/motta-addons.zip` — companion plugin, required for all Elementor widgets
- `Plugins/soo-demo-importer.zip` — demo importer

### Motta files worth knowing
- `inc/dynamic-css.php` — generates the CSS custom properties from Customizer options. This is why branding is a **setting, not a code change**.
- `inc/vendors/dokan.php` — 351 lines wiring Dokan vendor data into product pages, quickviews and store headers. Adapters also exist for WCFM and MarketKing.
- `inc/mobile/navigation-bar.php` + `template-parts/mobile/` — the built-in mobile bottom nav and compact header. This is the app-like shell; use it rather than building one.
- `dokan/store-lists-filter.php`, `dokan/vendor-store-info.php` — Dokan template overrides.

### Motta design tokens (defaults)
```
--mt-color__primary: #3449ca      <- Motta's blue, replace with Ultraw's
--mt-font__main: "Outfit"
--mt-border__radius: 2px
```
These come from `:root` in `style.css` and are overridden at runtime by
`primary_color` / `primary_color_custom_color` theme options.

---

## Hard rules

1. **Never edit the Motta parent theme.** All custom CSS and PHP goes in `motta-child`.
   A Motta update overwrites the parent wholesale.
2. **Branding is done in the Customizer, not in code.** Setting the primary colour regenerates
   the entire derived palette (dark, darker, gray, grayer, grayest, box-shadow) via
   `inc/dynamic-css.php`. Hardcoding hex values fights the theme.
3. **EasyWP SFTP uses port 22.** The abandoned workflow used **2222**, which is WP Engine's port.
   Any future deploy pipeline must use 22 and the `fs-alpha.easywp.com` style host.
4. **The owner is new to git, GitHub and deploys, and works almost entirely from an iPhone.**
   Give one step at a time. Screenshots are the normal feedback channel. Expect the GitHub
   *mobile app* to be a dead end — it has no repository Settings and no Actions "Run workflow";
   mobile Safari on github.com does.

---

## The naming trap

Everything from the earlier attempt is named "WP Engine" — the workflow file
(`deploy-wpengine.yml`), the job names, the GitHub secrets (`WPENGINE_SFTP_HOST`,
`WPENGINE_SFTP_USER`, `WPENGINE_SFTP_PASSWORD`). **The site was never on WP Engine.**
It is and always was EasyWP. The `ewp.live` staging domain is EasyWP's, not WP Engine's.
Rename this on the fresh start rather than inheriting the confusion.

---

## Current state (2026-08-25)

- **ultraw.bz is LIVE, PUBLIC, and serving Motta's demo content.** Verified by fetching the
  homepage: blog posts titled "Robot Vacuums and Mops Keep Your Floors Sparkling Clean" and
  "Apple unveils new iMac and iPad Pro with M1 chips", categories for Fashion / Wearable
  Technology / Mother & Kids, lorem ipsum in places, under an "Ultraw Ltd" logo.
  **Highest-priority fix: enable coming-soon mode and purge demo content.**
- Motta, the child theme, WooCommerce, Dokan and supporting plugins are reported installed.
- Elementor 4.2.3 detected on the live site.
- No real vendors, no real products, no completed test order, no payment gateway.
- Earlier staging install: `ultraw-new-116f609.ingress-alpha.ewp.live` — treat as abandoned.

### Repos
- `iannap/Ultraw-Ltd` — the live one. Fresh.
- `iannap/Ultraw` ("Online Store", JavaScript) — **to be archived**
- `iannap/Ultraw-V1` — **to be archived**. Contains `deploy-wpengine.yml` on branch
  `claude/ultraw-fresh-start-k8mh97`, plus the three SFTP secrets. That workflow's only run
  failed in 10s because the secrets were empty at the time. The secrets were added afterwards
  but it was never successfully re-run. **It was never needed** — there is no custom code to
  deploy yet.

---

## Decisions already made

| Decision | Choice | Rationale |
|---|---|---|
| Dispatch & live tracking | **Deferred until after MVP** | Run 20+ orders manually first; the manual runbook becomes the spec for `ultraw-dispatch` |
| Launch categories | Groceries, retail, pharmacy | No restaurants — avoids prep-time coordination and live courier tracking on day one |
| Old repos | Archive, don't delete | History preserved, repo list stays clean |
| Deploy pipeline | **Parked** | Not worth it until there's custom `motta-child` code worth versioning |
| Marketplace plugin | Dokan | Natively supported by Motta |

---

## Open blockers

1. **Payments — this is the project-defining risk.**
   **Stripe does not operate in Belize.** Stripe supports 46 countries and, in all of Latin
   America, only Brazil and Mexico. Most WooCommerce documentation assumes Stripe and will not
   apply. Unresolved options to investigate:
   - Atlantic Bank and Belize Bank merchant services / online gateway
   - First Atlantic Commerce (serves much of the Caribbean)
   - **Cash on delivery** — a legitimate MVP answer for an island market, costs nothing to start
   Must be settled *before* recruiting vendors. A marketplace that can't take money is a catalogue.

2. **Pharmacy regulation in Belize** — confirm what may be listed and delivered online before
   putting a pharmacy live. Needs a real answer, not a guess.

3. **Dokan Lite vs Pro** — Lite covers basic vendors. Pro adds vendor staff, advanced
   commission structures and better withdrawals. Start Lite, upgrade on demand.

4. **EasyWP capacity** — managed plans are sized for ordinary sites. A marketplace with many
   vendors and large image catalogues is heavier. Watch load times once real catalogues land.

---

## Build phases

Live tracker (checkboxes persist across sessions and devices):
https://claude.ai/code/artifact/05b088b7-1d89-4530-a665-924cbd5cfbb1

**Phase 0 — Clear the decks.** Archive `Ultraw` and `Ultraw-V1`. Confirm `Ultraw-Ltd` is the
only active repo. Retire the old staging install.

**Phase 1 — Foundation.** Coming-soon mode ON first. WooCommerce. Belize/BZD. Permalinks to
Post name (early — changing later breaks links). Elementor. Motta parent uploaded but not
activated; **activate the child**. Motta Addons. Dokan Lite. Demo import then strip back.

**Phase 2 — Make it Ultraw.** Primary colour in Customizer. Logo + favicon. Typeface.
Rappi-style homepage: search-first, category tiles, store carousels. Enable Motta's mobile
bottom nav. All custom code in `motta-child` only.

**Phase 3 — One real vendor.** Dokan setup wizard. **Settle payments.** Recruit one San Pedro
minimarket or retail shop. 20–30 real products with real photos and prices. Place a real
end-to-end order with real money. Have the vendor complete one withdrawal. Until money reaches
a vendor, nothing is proven.

**Phase 4 — Delivery, simple version.** 2–4 San Pedro zones in WooCommerce shipping. Flat fee
per zone. Delivery windows and cut-offs. A written manual dispatch runbook. 20+ real orders
through it.

**Phase 5 — Launch.** Point ultraw.bz properly. SSL forced. Terms, Privacy, Refunds and a
Vendor Agreement (commission, payout timing, liability for bad orders). Test checkout on a real
phone with a real card. Coming-soon off.

**Later, deliberately.** `ultraw-dispatch` (driver assignment, live GPS). Native app.
Restaurants. GitHub Actions deploy pipeline.

---

## Access and security

- There is **no programmatic admin access** to ultraw.bz. wp-admin credentials were offered in
  chat and declined — no tool here logs into authenticated admin panels, and scripting a
  headless browser around that is not an acceptable substitute.
- **The admin password `Cinematic#21` was posted in chat and is reused across at least two
  accounts. It must be rotated, and 2FA enabled.** If it hasn't been done, say so again.
- **Legitimate path to real access:** install **Jetpack** on ultraw.bz, connect it to a
  WordPress.com account, and enable abilities at `wordpress.com/me/mcp`. The WordPress.com
  connector reaches self-hosted Jetpack-connected sites over proper authorization. Narrower
  than full admin but covers pages and content. Not yet set up.
- Default working mode until then: write real code and exact click-by-click steps; the owner
  executes them on his phone.

---

## How to work with Ian

- One step at a time. Do not dump a ten-step list — give step one, wait for the screenshot.
- He will send screenshots rather than paste text. Read them carefully; twice already the fix
  was spotting that a value had been typed into the wrong field, or that he was in the GitHub
  app instead of a browser.
- Be direct about blockers rather than working around them quietly. The Stripe finding and the
  live demo content were both things he did not know and needed to hear plainly.
- He describes the project as feeling like "a bag of trash" when there are too many
  half-finished threads. Reducing to one repo, one site, one next action is the antidote.
