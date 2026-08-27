# Belize payments & logistics — research notes

Written 2026-08-27, in response to Ian's framing of the core problem: Amazon, DoorDash and
Uber's marketplace products don't operate in Belize, and the only thing that actually works
today is Facebook Marketplace. This doc is the research behind why, and what Ultraw can
actually do about it. It expands "Open blockers" in `CLAUDE.md` — read that first for
project context.

---

## Why the majors haven't come

It isn't that Belize is unsolvable. It's that it isn't worth their engineering budget.

- Belize's total population is ~400,000 — smaller than a single mid-size US suburb. San
  Pedro itself has ~20,000 residents (2024 estimate), though San Pedro + Caye Caulker
  together generate roughly 70% of Belize's tourism revenue, so the real addressable
  demand is residents *plus* a large, seasonal, higher-spending tourist/expat population.
- Uber Eats, DoorDash and Amazon's marketplace stacks assume they can plug into rails that
  already exist everywhere they operate: Stripe-class card processing, an existing gig-work
  courier supply, addressable postal addressing. None of that is true here by default —
  **Stripe does not support Belize at all**, and there's no existing large gig-courier fleet
  to recruit from.
- Building bespoke payment-rail integrations and a local delivery fleet for a market this
  size is not a decision a company the size of Amazon or Uber makes. It's exactly the kind
  of unglamorous, high-friction, low-headline-value integration work a small local operator
  *can* justify doing — the same gap Jumia filled across Africa starting ~2012.
- Conclusion: the absence of majors is a market-size signal, not a feasibility verdict. The
  rails exist in Belize (see below) — nobody has bothered wiring a multi-vendor marketplace
  on top of them yet, at least not in San Pedro.

---

## Payments

### The hard constraint
Stripe supports 46 countries; in Latin America that's Brazil and Mexico only. Nearly every
WooCommerce/Dokan payments tutorial assumes Stripe and does not apply here. This is real and
already logged as the project's top blocker — but it is not a dead end. There are several
live paths, layered by how fast they can go live vs. who they serve.

### Path 1 — PayPal Business (fastest, do this first)
PayPal issues business accounts to Belize entities today. A PayPal Business account can:
- Take a "Pay with PayPal" button or full checkout on the site, which accepts major cards
  **from buyers who don't have a PayPal account** — this is the part that matters, because
  it means foreign tourist/expat credit cards work without Ultraw ever holding a local card
  merchant agreement.
- Send PayPal.me links or invoices for manual/phone orders — useful as a bridge before
  checkout is even built out.
- Settle without going through Belize's bank underwriting process at all.

WooCommerce ships an official PayPal gateway plugin — this is close to a same-day
integration, not a project. **This is the single fastest way to get Ultraw taking real
money**, and it specifically covers the tourist/expat segment that already prepays online for
vacation-rental groceries (see "Existing local competitors" below).

Gap: PayPal skews toward buyers who already have foreign cards — i.e. tourists/expats, not
necessarily unbanked San Pedro locals. It's a fast unlock, not the whole answer.

### Path 2 — Local bank merchant gateway (start the application now, in parallel)
- **Atlantic Bank** offers a real payment gateway product (POS + e-commerce + e-Packaging),
  settling locally, with fraud screening and CVV2. atlabank.com/payment_gateway.php
- **Belize Bank** offers merchant services (POS, card acceptance) via a standard
  branch-visit → account review → terminal provisioning flow, historically ~10 business days
  for physical POS; the online gateway timeline needs to be confirmed directly with a
  Merchant Officer since it isn't published.
- **First Atlantic Commerce (FAC, now under the PowerTranz brand)** is the underlying gateway
  technology many Caribbean acquiring banks use, and it settles in BZD. In practice this is
  reached *through* a bank relationship (Atlantic Bank or Belize Bank), not signed up for
  directly — treat FAC as the plumbing behind Path 2, not a separate fourth option.

This path is slower (bank underwriting, business account, compliance review) but is the one
that eventually gives Ultraw normal local card acceptance with BZD settlement. Given the lead
time, **the application should start now**, independent of Phase 3 sequencing — it can run in
the background while cash-on-delivery and PayPal cover launch.

### Path 3 — Local digital wallets (second wave, once volume justifies the dev time)
Two real, Central-Bank-recognized wallets exist and are already used by real Belizean
consumers:
- **E-kyash** (Belize Bank's wallet) — no bank account required to receive, P2P and
  merchant QR payments, bill pay, and Belize Bank explicitly documents third-party merchant
  file integration. Already networked into 1,500+ merchants. This is the wallet most likely
  to reach *unbanked* San Pedro locals, which cards and PayPal don't.
- **DigiWallet** (Digicell/Belize Telemedia's telco wallet) — largest user base in the
  country (~190,000 mobile customers), ~90% network coverage, built-in e-commerce/merchant
  payment support.

**Naming trap to flag, same shape as the WP Engine/EasyWP one already in `CLAUDE.md`:**
searching "DigiWallet WooCommerce plugin" mostly surfaces a *Dutch* payment processor also
called DigiWallet (digiwallet.nl / e-plugins.nl), unrelated to Belize's DigiWallet. Do not
install that plugin thinking it connects to the Belizean wallet — it doesn't. Belize
DigiWallet integration, if pursued, will need direct outreach to Digicell/Belize Telemedia
for real merchant API docs.

Real proof this stack works in production: **Runman**, a Belizean-built delivery app (see
below), already lets customers pay via DigiWallet, E-kyash, or card inside its checkout. That
de-risks Path 3 — it's not theoretical, a Belizean company has shipped it.

### Path 4 — Cash on delivery (ship on day one, don't treat as a fallback)
COD isn't a stopgap here, it's the payment method that already built this category's only
working precedent: Jumia used cash-on-delivery deliberately across Africa specifically
because card/bank penetration was low, and it was "primarily responsible" for their early
order volume. Belize's own informal delivery culture (Runman's motorcycle "runmen," Facebook
Marketplace trades) already runs almost entirely on cash-in-hand or pay-on-pickup trust.
Launching with COD alongside PayPal costs nothing and matches how San Pedro already
transacts.

### Horizon — Central Bank's Instant Payment System
The Central Bank of Belize has a live legal framework for this already (National Payment
System Act 2017, Electronic Transactions Act) and has contracted Montran Corporation, with
World Bank backing, to build a national **Instant Payment System**. This is not usable yet,
but it means Belize is actively building the FedNow/PIX-style rail that would eventually make
bank-to-bank checkout cheap and instant. Worth a watch-item, not a plan dependency.

### Recommended sequencing
1. **Now:** Cash on delivery + PayPal Business checkout. Both can be live before Phase 3's
   "one real vendor" milestone.
2. **In parallel, slow-cook:** Start the Atlantic Bank or Belize Bank merchant gateway
   application. Long lead time, so start it even though it won't be launch-blocking.
3. **Once order volume justifies engineering time:** Add E-kyash as a custom WooCommerce
   gateway — it's the best answer for unbanked locals and has real merchant-network effects
   already.
4. **Don't touch:** the Dutch "DigiWallet for WooCommerce" plugin — wrong DigiWallet.

---

## Logistics

### The geography is actually an advantage, not the Jumia problem
Jumia's continental logistics nightmare was about *distance* — poor roads and inter-city
freight across a continent. San Pedro doesn't have that problem: it's a small island town,
golf-cart-scale, with short last-mile distances even at the far ends of town. The delivery
problem here isn't "how do we move goods 400km," it's "how do we get a handful of drivers
covering a few square miles reliably." That's a solvable operations problem, not an
infrastructure one.

### Existing precedent, already proving both halves of the model
- **Runman** — a real Belizean on-demand delivery app (food + "shopping"), built by
  Belizeans, already live in Belize City, Belmopan, Ladyville and 8 Miles. It is *not* in
  San Pedro/Ambergris Caye today. Its driver model is motorcycle-based, which fits mainland
  city streets but doesn't map directly onto San Pedro's golf-cart geography. Treat this as
  proof the demand + payment model works in this country, a competitive watch-item for if
  they expand to the island, and a UX/ops reference rather than a direct blueprint.
- **San Pedro's own single-vendor delivery operators** already exist and are already making
  money serving this exact demand: San Pedro Provisioning Company, Coconut Carts Belize,
  Mata Grande, Lagniappe Provisioning, and Public Supermarket (free delivery). All of them
  run isolated order-taking (mostly Facebook/website forms aimed at tourists prepping
  vacation rentals before arrival) with their own ad hoc delivery. This is direct market
  evidence for Ultraw's core thesis: the demand is proven, it's just fragmented across single
  stores with no shared checkout, no shared delivery fleet, and no way to buy from more than
  one of them in one order. That fragmentation *is* the opening — same relationship Instacart
  or Rappi has to individual grocery chains.
- Golf carts are already the default local delivery vehicle on the island for other trades
  (every cart rental company on Ambergris Caye advertises "free delivery" of the cart
  itself) — the vehicle and the driver pool to adapt already exist locally.

### Recommended approach — matches the phases already in CLAUDE.md, don't skip ahead
1. **Don't build fleet tech first.** Phase 4 already calls for a written manual dispatch
   runbook and 20+ real orders before any `ultraw-dispatch` code — that sequencing is right
   and this research doesn't change it.
2. **Recruit delivery capacity from what already exists on the island**, not from scratch:
   the vacation-rental grocery-prep operators already employ or contract people who deliver
   groceries by golf cart daily. Either bring them on as Ultraw vendors directly (they get
   an aggregated storefront + shared checkout instead of a solo Facebook page) or hire from
   their delivery pool once Ultraw needs its own drivers.
3. **Design the eventual driver-assignment system (`ultraw-dispatch`, "Later, deliberately")
   around golf carts, not motorcycles** — Runman's motorcycle model is the wrong template for
   this island's geometry and vehicle norms.
4. **Watch Runman's expansion**, not as a threat to react to yet, but because if they reach
   San Pedro first with an already-proven payment stack (DigiWallet/E-kyash/card), that
   changes the competitive picture for Ultraw's Phase 3–4 window.

---

## Net read on the original question

"Why hasn't anyone come, and can Ultraw actually solve it" — yes, and the pieces are less
missing than they looked:

- Belize *does* have functioning digital payment rails (bank gateways, two real wallets,
  PayPal access, a Central-Bank-blessed legal framework, and an Instant Payment System in
  development) — no major platform has bothered wiring a multi-vendor marketplace checkout
  on top of them because the market is small by their standards, not because it's impossible.
- San Pedro's logistics problem is a short-distance operations problem, not a continental
  infrastructure gap, and the delivery workforce (golf-cart operators already serving
  individual grocery/provisioning businesses) already exists to draw from.
- The fastest real unlock for Ultraw specifically is: **COD + PayPal now, bank gateway
  application started now in parallel, wallets added once volume justifies it** — which
  keeps Phase 3 ("one real vendor, settle payments, one real end-to-end order") achievable
  without waiting on anything that isn't already live today.

## Sources
- [Accepting Payments in Belize: PSPs, Compliance & Fees — PayAtlas](https://payatlas.com/countries/belize-bz)
- [First Atlantic Commerce Merchant Account Guide](https://cartdna.com/en/resources/blog/how-to-open-fac-merchnat-account)
- [Atlantic Bank — Payment Gateway](https://www.atlabank.com/payment_gateway.php)
- [Belize Bank — Merchant Services](https://www.belizebank.com/merchant-services/)
- [Belize Bank — E-kyash Integration](https://www.belizebank.com/ekyash-integration/)
- [E-kyash — A digital payments solution](https://www.e-kyash.com/)
- [DigiWallet Belize — Merchant](https://www.digiwallet.bz/merchant/)
- [PayPal Belize — Payment Solutions for Your Business](https://www.paypal.com/bz/webapps/mpp/merchant)
- [Central Bank of Belize — National Payment System Act](https://www.centralbank.org.bz/about-the-bank/laws-and-regulations/national-payment-system-act)
- [Central Bank of Belize — Instant Payment System announcement](https://www.centralbank.org.bz/detail-pages/news/2026/04/21/central-bank-of-belize-announces-development-of-an-instant-payment-system)
- [Run Man Belize](https://runman.co/)
- [7 News Belize — Runman coverage](https://www.7newsbelize.com/sstory.php?nid=74773)
- [San Pedro Provisioning Company](https://sanpedroprovisioningcompany.com/)
- [San Pedro City: Population Boom — Ambergris Today](https://ambergristoday.com/blog/2026/07/08/san-pedro-city-population-boom/)
- [The 70% Powerhouse: San Pedro and Caye Caulker tourism — Travel And Tour World](https://www.travelandtourworld.com/news/article/the-70-powerhouse-why-san-pedro-and-caye-caulker-are-redefining-belize-tourism-in-2026/)
- [Jumia and the Amazon of Africa: Lessons for E-Commerce — Oban International](https://obaninternational.com/blog/jumia-and-the-amazon-of-africa-lessons-for-e-commerce/)
