# Bill Splitter — project context

## What this is
A single-file, client-side bill splitter for groups of friends dining out in
Singapore. Hosted on GitHub Pages — no backend, no login, no persistence.
Everything lives in `index.html` (HTML + CSS + vanilla JS, no build step, no
third-party requests). Fonts are self-hosted in `fonts/` (the one deliberate
exception to "single file"): Special Elite 400, Courier Prime 400+700, and a
variable-weight Inter, loaded via @font-face — no Google Fonts calls.

## Design identity
"Receipt printer" aesthetic: paper-cream palette, Special Elite (typewriter)
display face, Courier Prime for amounts, stamp-red accent (#B23A2F), torn
perforated edge under the results receipt. Keep this identity when adding UI.

## Current features
- Add/remove people (chips) and items (name + cost)
- Each item defaults to "shared by everyone currently added"; per-item
  checkboxes to mark who consumed it
- Charges: Service (10%), GST (9%), Other — each toggleable, editable;
  Other switches between % of subtotal and flat $ via a unit toggle button
- Discount (toggleable): % or flat $, applied to the subtotal BEFORE service
  charge and GST (SG receipt stacking); flat discounts clamp at the subtotal
- Per-item discounts: "+ discount on this item" link on each item row opens a
  % / flat $ editor; item cost shows strikethrough original + net price. Net
  item cost feeds the subtotal, the itemized split, and the charge stack; the
  receipt shows "Item discounts" and "Discount" as separate lines, and
  discounted items are tagged (e.g. "Naan (÷2, −50%)") in person rows
- Venue presets: Restaurant (10% svc + GST), Café/fast food (GST only,
  prices GST-inclusive), Hawker (nett). A preset sets the toggles below it;
  the highlight is re-derived from toggle state, so manual flips un-highlight
- "Prices already include GST" checkbox: GST extracted as rate/(100+rate),
  shown on the receipt for info only, NOT added to the total
- Treat someone (🎁 on their chip): they pay $0 — equal mode divides the
  total among payers only; itemized redistributes their share equally
  among payers
- GST is computed on (subtotal + service charge) — correct SG stacking; keep this
- Two split modes:
  - Equal: grand total / headcount
  - By item: item cost split among its consumers; service/GST/other pro-rated
    by each person's share of the food subtotal (NOT split evenly — keep this)
- Itemized mode has a rounding-cent correction so shares sum exactly to total
- Itemized mode: items ticked by nobody are excluded from subtotal, charges
  AND total (with a receipt note) — previously their cost stayed in the total
  and the rounding correction silently dumped it on the biggest share
- Copy-summary-as-text button (clipboard API with execCommand fallback)
- Person rows in results: name + amount on same line, items below in grey

## Known issues (from code review, not yet fixed)
1. Equal-split mode lacks the rounding-cent correction that itemized mode has
   ($100.01 / 3 shows 3 × $33.34 = $100.02)
2. People added AFTER an item exists are not auto-ticked on that item;
   consider auto-include or per-item select all/none shortcut
3. No item quantities (4× beer = one lump sum or four lines)
4. No editing of items/prices — delete and re-add only
5. Rate inputs accept negative values via typing (min attr only blocks spinner)
6. Flat "Other charge" is added after GST; SG venues usually charge GST
   (and often svc) on fees like corkage/cake-cutting — decide stacking

## Agreed backlog (Singapore dining context), in priority order
1. "Who paid?" selector + transfer summary output
   ("Transfer $23.40 to Wei Ming" per person — matches the SG PayNow
   settlement pattern; include in the copy-to-text output)
2. ~~Discount line~~ — DONE (bill-level + per-item, see Current features)
3. Equal-split rounding fix (known issue #1)

## Considered and deferred
- Cash rounding to nearest $0.05 (off by default if added)
- Item quantities; weighted shares within an item ("Jon drank double")
- Live "++" total preview while adding items

## Explicitly rejected — do not add
- PayNow QR generation (requires user's mobile number in-page; dynamic
  payment QRs from webpages are a scam-pattern users are warned against)
- Multi-currency (SG-only tool, keep `$`)
- Login / persistence of any kind

## Conventions
- Keep everything in one index.html — single-file is a feature (drop into a
  repo, enable Pages, done). Sole exception: the four .woff2 files in fonts/,
  which must be deployed alongside index.html
- Vanilla JS only, no frameworks, no build step
- All money math: work in floats, round at display, keep sum-to-total
  corrections so displayed shares always reconcile with the printed total
- Currency format: `$` + 2dp via the fmt() helper
