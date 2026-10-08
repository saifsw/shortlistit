# Shortlist — MVP Product Requirements

## Overview

Shortlist is a web app for saving products you're thinking of buying, from any store, in one place. Each product gets a list of competing listings, so you can compare prices at a glance and track where each one stands, from 'considering' to 'purchased'. Related products, like the parts of a PC build, can be gathered into a group. The MVP ships as a mobile-first Next.js web app backed by a Laravel API; a mobile app follows later.

## Problem

When you shop for something, the options end up scattered across browser tabs, screenshots, and chat messages. It's hard to remember which listing was cheapest, which ones you already ruled out, and which one you ordered.

## Goals (MVP)

- Save a listing in under 15 seconds by pasting a link.
- Compare every listing for a product and spot the cheapest one immediately.
- Record the price a seller agreed to after negotiating, and have it drive the comparison.
- Group related products, like the parts of a PC build, and see a running total.
- Track each listing's status from 'Considering' to 'Purchased'.

## Non-goals (MVP)

- Automatic price tracking or price-drop alerts
- Price history charts
- Browser extension or share-sheet saving
- Shared or collaborative lists
- Automatic currency conversion
- Product search or deal discovery
- Mobile app (planned after the web MVP)

## Target user

An individual shopper comparing several options across stores (local and international) before buying. The first user is the builder; broader audience comes after the MVP proves useful.

## Key concepts

- **Group** — a set of related products you're shopping for together (for example a 'Gaming PC' group holding a list for the CPU, GPU, RAM, and so on). Optional: a list can also stand alone. Has a name and an optional note.
- **List** — one product you're shopping for, holding competing listings from different stores (for example 'Graphics card' or 'Standing desk'). Has a name and an optional note. Belongs to at most one group.
- **Listing** — one specific offer: title, listed price, optional negotiated price, currency, store, URL, image, notes, and status.
- **Status** — where a listing stands. Fixed set for the MVP: **Considering** (default), **Shortlisted**, **Purchased**, **Rejected**.
- **Account** — a user signed in with Google. Their groups, lists, and listings are private to that account and stored on the backend.

## Functional requirements

### 1. Sign in and account

- Sign in with Google before accessing saved data. On first sign-in, create an account; returning users see their existing data.
- Keep users signed in across browser visits until they sign out or their session expires. Provide sign-out from the Account tab.
- Scope every group, list, listing, and link-parsing request to the signed-in account where applicable. Users cannot read or change another account's data.

### 2. Lists

- Create, rename, and delete a list. Deleting asks for confirmation and removes the list's listings.
- The home screen shows all groups and standalone lists. A list shows its listing count and its lowest active price.

### 3. Groups

- Create, rename, and delete a group. Deleting asks for confirmation and whether to keep its lists as standalone lists or delete them too.
- A group holds any number of lists, for example a 'Gaming PC' group with lists for CPU, GPU, RAM, motherboard, storage, power supply, and case. Each list keeps its own competing listings.
- Create a list inside a group, or move an existing list into or out of one. A list belongs to at most one group, and lists can still stand alone.
- The group screen shows each list with its selected listing, that listing's price, and its status. The selected listing is the Purchased one if there is one, otherwise the Shortlisted one (the cheapest if several), otherwise the cheapest Considering one.
- The group shows an estimated total (the sum of the selected prices), the amount already spent (Purchased listings), and progress, for example '3 of 7 parts purchased'.
- Totals use effective prices (see Negotiated price). A group that mixes currencies shows a total per currency instead of converting.

### 4. Add a listing

**By link**

- The user pastes a URL into the add field.
- The app fetches page metadata (title, image, price, store name from the domain) and pre-fills the form.
- The user can edit any field before saving.
- If the fetch fails or no price is found, the form still opens with the URL and store pre-filled and the rest blank. Saving is never blocked.

**Manually**

- Fields: title (required), listed price (required), negotiated price (optional), currency (default JOD), store, URL (optional), image (optional), notes (optional).

### 5. Update status

- Change a listing's status from a quick action on the list screen or from the listing detail screen.
- New listings start as Considering.
- Filter a list by status.
- Rejected listings are hidden from the comparison by default, with a toggle to show them. Purchased listings stay visible but look distinct.

### 6. Negotiated price

- Every listing has a listed price (what the seller advertises) and an optional negotiated price (what the seller agreed to after haggling).
- Set, change, or clear the negotiated price from a quick action on the card or from the listing detail screen, at any status.
- Optionally add a short note with it, for example 'agreed over WhatsApp, cash on pickup'.
- The effective price is the negotiated price if set, otherwise the listed price. Sorting, the 'Lowest' badge, differences from the lowest, and group totals all use the effective price.
- A card with a negotiated price shows it prominently, with the listed price struck through and the amount saved.
- The negotiated price uses the listing's currency.

### 7. Compare prices

- Listings in a list sort by effective price (the negotiated price if set, otherwise the listed price), lowest first, by default.
- The lowest-priced active listing (Considering or Shortlisted) gets a 'Lowest' badge.
- Every other card shows its difference from the lowest price (for example '+JOD 12').
- If a list mixes currencies, show a warning and don't rank across currencies. Conversion is post-MVP.

### 8. Edit and delete

- Any listing field can be edited. Changing the listed or negotiated price updates a 'price last updated' date shown on the card.
- Deleting a listing offers undo.

## Web app navigation and screens

- Design for narrow phone viewports first, then adapt to tablets and desktop. Keep primary actions and price comparisons usable with touch and one hand; use stacked listing cards on phones and take advantage of wider layouts on desktop.
- On phones, use a persistent bottom tab bar with **Home**, **Groups**, and **Account**. On wider screens, adapt the same destinations to a side navigation rail or sidebar. Detail screens retain a clear route back to their parent tab.
- **Home tab** — all groups and standalone lists, with a prominent create action. This is the default destination after sign-in.
- **Groups tab** — browse and create groups. **Group detail** shows the lists in a group, each with its selected listing and price, plus estimated total, amount spent, and progress.
- **Account tab** — show the signed-in Google account and provide sign-out.
- **List detail** — listing cards, sort and status filter, add action. It opens from Home or a group.
- **Add listing** — a mobile-friendly sheet or page with two paths: paste a link, or enter manually.
- **Listing detail** — all fields, status control, negotiated price, open original link, edit, delete.

## Delivery plan

1. **MVP:** responsive Next.js web app and Laravel backend, including Google sign-in and persistent account data.
2. **Later:** mobile app using the same account data and Laravel API. Mobile-specific interaction and sign-in details will be defined before that phase.

## Technical considerations

- **Repository:** use a Turborepo monorepo. Place the Next.js app in `apps/web` and the Laravel backend in `apps/api`. Turborepo coordinates workspace tasks; Composer manages the backend's PHP dependencies.
- **API boundary:** the Laravel backend owns business rules, persistence, authorization, and link parsing. Expose a versioned API with documented contracts so the later mobile app can use the same backend.
- **Authentication:** use Google OAuth for sign-in. Laravel owns account creation and authenticated API access; use secure, HTTP-only browser sessions for the web app. The later mobile app will need a mobile-appropriate OAuth flow.
- **Link parsing:** Laravel reads Open Graph tags and JSON-LD product data from the page. Parsing runs server-side because browsers cannot reliably fetch arbitrary store pages.
- **Expect failures:** many stores block scrapers or load prices with JavaScript. The manual fallback is a core path, not an edge case.
- **Prices:** store as decimals with an ISO currency code. JOD uses 3 decimal places.
- **Storage:** persist account-scoped data in the backend database so it is available across browser sessions and to the later mobile app.

## Success metrics

- Median time to add a listing is under 15 seconds.
- At least 60% of pasted links auto-fill a title (target to validate).
- The app is used to make at least one real purchase decision end to end.

## Risks

- **Unreliable link parsing:** mitigated by the always-available manual fallback.
- **Scope creep into price tracking and mobile app development:** deferred until the web core flow is proven.
- **Stale prices:** mitigated by showing when each price was last updated.

## Open questions

1. Is the rule for a group's selected listing right (Purchased, else Shortlisted, else cheapest), or should you explicitly mark one pick per list?
2. Are the four statuses enough, or should users create their own?
3. Which stores matter most? This decides which links to optimize parsing for first.

## Post-MVP ideas

Mobile app, share-sheet saving, price refresh and drop alerts, price history, currency conversion, shared lists, browser extension.
