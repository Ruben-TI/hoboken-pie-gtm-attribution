# GTM/GA4 Conversion Tracking Build Without Toast Pro (Hoboken Pie)

## What It Is

A Google Tag Manager and GA4 conversion tracking build for Hoboken Pie, an Austin pizza shop pivoting 70% of its ad budget from catering leads to online retail orders. The core challenge: Toast (the client's online ordering platform) locks GTM/pixel injection behind a "Pro" paywall the client didn't have. Every standard tracking method was unavailable, so a working conversion signal had to be engineered around that gap.

---

## The Problem

The client wanted a budget split: 70% retail (online pickup/delivery via Toast) and 30% catering (lead form on the client's own site). But retail orders complete entirely on `toasttab.com`, a domain outside GTM's reach without the paid tier. Three standard tracking routes were checked and closed off in sequence:

- **Toast's native Google Ads integration.** Available, but it's a managed black-box service that would have taken campaign control away from the account.
- **GTM container injection into Toast's checkout pages.** The settings field exists, but it's gated behind Toast's Digital Storefront Pro subscription.
- **Custom post-checkout redirect** (bounce the customer back to a `hobokenpie.com/thank-you` page GTM could see). Also paywalled, confirmed via Toast's own support AI.

With the "real" purchase event unreachable, the account had no success signal to optimize against for the retail side of the budget.

---

## The Solution & Impact

**Proxy conversion strategy (retail).** Built a `retail_order_start` event that fires on the "Order Online" button click itself, the last moment still inside GTM's reach before the user leaves for Toast, instead of waiting for a purchase confirmation that was structurally unavailable.

- **Trigger debugging via the "Red X" method.** The first version of the trigger didn't fire. Rather than guess, the fix came from reading GTM Preview's own trigger-condition breakdown to find which exact condition was failing. First it was a URL pattern mismatch (`toasttab.com/hoboken-pies` vs. the actual `order.toasttab.com/online/hoboken-pies`), then a leftover `/menu` page-path restriction that excluded homepage clicks. Both were root-caused from the debug data, not trial and error.
- **GA4 Key Event registration bypass.** GA4 normally takes 24 to 48 hours to surface a new event for import into Google Ads. Manually registering `retail_order_start` as a Key Event made it importable right away, avoiding a two-day delay on a live budget pivot.
- **Conversion value modeling.** Set the proxy event's value to $14 (the entry-level pie price) instead of a default $1, giving Smart Bidding a realistic revenue signal to bid against despite not having real order totals from Toast.

**Enhanced Conversions (catering).** Built a DOM Element variable to capture the catering form's email field, mapped it into a User-Provided Data variable, and attached it to the catering conversion tag. This hardens attribution for high-value leads against cookie loss, using only the access available on the client's own site with no Toast dependency.

**Goal-category segmentation.** Categorized `retail_order_start` as **Purchase** (Primary) and the catering event as **Submit lead form** (Primary) in separate campaigns with isolated goal settings, so the 70% retail budget and 30% catering budget couldn't bid against each other's signals.

**Measured funnel outcome (catering side, GA4).** `scroll_catering_menu` then `view_catering_menu` then `begin_catering_checkout`: a real engagement funnel, not just a click count. Month over month, the catering campaign went from 29 conversions, $24.50 per conversion, and roughly $7,250 in estimated revenue in March, to 52 conversions, $19.73 per conversion, and roughly $13,000 in estimated revenue in April. Cost per conversion dropped 20%.

---

## Tech Stack

`Google Tag Manager` · `Google Analytics 4` · `Google Ads` · `Enhanced Conversions` · `GA4 Key Events` · `Toast (online ordering platform)`

---

## Demo / Screenshots

📊 Monthly reporting dashboards: March 2026 and April 2026 (catering campaign performance, funnel events, device/day-of-week breakdowns)
📊 [March 2026 Dashboard](./Hoboken_Pie%20Google%20Ads%20March%202026.pdf)
📊 [April 2026 Dashboard](./Hoboken_Pie_Dashboard%20Google%20Ads%20April%202026.pdf)


> *Dashboards include: conversion trend, cost-per-conversion, search term/keyword performance, device split, day-of-week performance, and the GA4 event funnel (menu scroll, menu view, checkout begin).*

---

## How It Works

1. **Diagnose the tracking gap.** Confirmed GTM injection, native Toast integration, and redirect workarounds were all paywalled before committing to a proxy strategy.
2. **Build the proxy trigger.** GTM Click trigger scoped to Toast checkout URLs, explicitly excluding `/catering` paths to avoid cross-contaminating the two budgets.
3. **Debug systematically.** Used GTM Preview's per-condition pass/fail view to isolate exactly which trigger condition was blocking the tag, instead of rebuilding from scratch.
4. **Register and import.** Manually created the GA4 Key Event to skip the standard 24 to 48 hour propagation delay, then imported both `retail_order_start` and the catering lead event into Google Ads as separate Primary conversions.
5. **Value and categorize.** Set a realistic proxy value ($14) and correct goal category (Purchase) so Smart Bidding treated the signal appropriately.
6. **Harden the catering signal.** Built Enhanced Conversions via a DOM Element variable capturing the catering form's email, independent of the Toast paywall.
7. **Isolate budgets.** Campaign-level goal settings ensured the 70% retail campaign only optimized against `retail_order_start`, and the 30% catering campaign only against the lead event.

---

## What I Learned / Key Decisions

- **A blocked "ideal" solution isn't a dead end. It's a design constraint.** Three standard tracking methods were paywalled in sequence. Instead of recommending the client pay for Toast Pro, the fix was moving the conversion event earlier in the funnel, to the last point still inside GTM's reach.
- **Debug from evidence, not guesses.** The trigger not firing had two separate root causes (URL mismatch, then a stale page-path condition), both found by reading GTM's own per-condition debug output instead of rebuilding the trigger blind.
- **A proxy signal needs a realistic value to be useful.** A $1 default conversion value doesn't give Smart Bidding a meaningful revenue signal. Pricing the proxy at the entry-level product price ($14) gave the algorithm something real to optimize against.
- **Goal isolation prevents budget bleed.** Without campaign-specific goal settings, a shared account-level conversion list would have let the 70% retail budget get credit for (or bid against) 30% catering leads, and vice versa.

---

## Setup & Usage

This documents a live client tracking implementation, not a standalone script. The methodology is replicable for any business using a third-party ordering platform that restricts tag injection:

1. Confirm what tracking access is actually available on the third-party platform before assuming a redirect or pixel injection will work.
2. If the platform blocks GTM, move the conversion event to the last controllable touchpoint (e.g., an outbound click) instead of waiting for a confirmation page you can't see.
3. Manually register new events as GA4 Key Events if you can't wait out the standard propagation delay.
4. Set a realistic proxy conversion value. Don't leave it at a $1 default.
5. Use campaign-specific goal settings whenever a single account runs multiple budget-isolated campaigns.

---

*Part of a series documenting real agency work in paid search, reporting, and marketing operations.*

---

**Portfolio slot:** Growth/Ops Tool, Tag Management & Attribution Engineering
**One-line description:** Built a working conversion tracking system for a client whose ordering platform blocked every standard tagging method. Used a debugged proxy event, GA4 Key Event registration, and Enhanced Conversions to hit a 20% drop in cost-per-conversion month over month.
