# PRD: ZeroDrop Payment Failover Router & Dashboard (MVP Prototype)

## 1. Introduction / Overview

ZeroDrop is a payment failover router and checkout insurance infrastructure built for mid-market eCommerce operators. During high-volume sales events (e.g., ShoppingIL), payment gateways can fail under load — dropping entire carts and costing merchants thousands in lost revenue.

The ZeroDrop MVP prototype intercepts checkout payment submissions, monitors gateway health in real time, and dynamically reroutes failed transactions to the healthiest available fallback gateway with sub-400ms latency. If no gateway can process the transaction, the lost revenue is logged to a Reconciliation Ledger for insurance claims. An operator dashboard provides live operational visibility and rescued-GMV metrics.

**This is a frontend prototype.** Login, checkout data, store connections, and gateway health data are all simulated — no real payment processing or backend integration exists.

**Prototype user:** Adam Kogan (mid-market eCommerce operator running Shopify store).

---

## 2. Goals

| # | Goal | Measurement |
|---|------|-------------|
| G1 | Demonstrate a checkout plugin intercepting a payment and routing to the healthiest gateway | Simulated checkout flow with visible routing decision |
| G2 | Show dynamic failover when the primary gateway is unhealthy | Simulated gateway health toggle that triggers reroute |
| G3 | Provide a live operations dashboard showing active checkouts, gateway statuses, and latency | Dashboard with real-time simulated data stream |
| G4 | Display rescued GMV — revenue saved by the failover router | Aggregated dollar value of transactions where fallback succeeded |
| G5 | Log unrecoverable failures to a Reconciliation Ledger for insurance claims | Ledger table showing date, amount, customer, and gateway logs |
| G6 | Let the operator configure their preferred gateway | Settings panel that reorders routing preference |

---

## 3. User Stories

### US-1: Operator monitors live checkouts in real time
*As Adam Kogan, an eCommerce operator running a Shopify store during a flash sale, I want to see a live stream of incoming checkout attempts, their routing decisions, and outcomes, so I can confirm the failover router is working and no revenue is being dropped.*

**Acceptance criteria:**
- A live activity feed shows each checkout attempt with: timestamp, customer name, cart amount, primary gateway attempted, fallback gateway used (if any), status (Cleared / Rescued / Failed).
- New entries appear automatically (simulated at ~1-3 second intervals).
- The feed auto-scrolls to show latest entries.
- Each entry is color-coded: green = Cleared, cyan = Rescued, red = Failed.

### US-2: Operator sees gateway health status at a glance
*As Adam Kogan, I want to see the real-time health status of all connected payment gateways so I can identify outages before they affect customers.*

**Acceptance criteria:**
- A card or panel shows each gateway (Stripe, Adyen, Braintree, Checkout.com) with:
  - Status indicator (green = healthy, amber = degraded, red = down)
  - Current latency in ms
  - Uptime % over the last hour
- Health status updates dynamically (simulated).
- The operator's preferred gateway is visually marked (star or highlight).

### US-3: Operator views rescued revenue and ROI
*As Adam Kogan, I want to see how much revenue ZeroDrop has saved by rerouting failed transactions and what my insurance coverage looks like, so I can justify the investment to my leadership.*

**Acceptance criteria:**
- Metric cards display:
  - Total GMV Processed (all checkouts)
  - Rescued GMV (transactions saved by fallback)
  - Claimable Lost GMV (unrecoverable failures logged for insurance)
  - Active Insurance Coverage Amount
- A simple bar chart shows rescued vs. lost GMV by day (simulated weekly data).

### US-4: Operator configures preferred gateway
*As Adam Kogan, I want to tell ZeroDrop which payment gateway is my primary preference, so routing decisions respect my existing relationships and negotiated rates.*

**Acceptance criteria:**
- A settings panel lists all gateways.
- Operator can drag or select a preferred gateway.
- Preferred gateway is always tried first (if healthy).
- Remaining gateways are ordered by dynamic health metrics.

### US-5: Operator reviews the Reconciliation Ledger
*As Adam Kogan, I want to see a monthly ledger of all transactions where ALL gateways failed, so I can file insurance claims and understand my exposure.*

**Acceptance criteria:**
- A table lists unrecoverable transactions with: date, order ID, customer, cart amount, all gateway responses, and claim status.
- Monthly total of claimable lost GMV is displayed.
- Each row shows claim status (Pending / Filed / Settled).

### US-6: Operator simulates a gateway outage to test failover
*As Adam Kogan, I want to trigger a simulated gateway outage from the dashboard, so I can verify the failover router responds correctly during demos.*

**Acceptance criteria:**
- A "Simulate Outage" button on each gateway card.
- Clicking it marks the gateway as "down" (red) and incoming checkouts immediately reroute.
- A "Recover" button restores the gateway to healthy.
- This is clearly labeled as a simulation/demo feature.

---

## 4. Functional Requirements

### FR-1: Simulated Authentication
- The system must display a login screen with pre-filled credentials for Adam Kogan.
- Login must accept any password and redirect to the operator dashboard.
- No real authentication backend is required.

### FR-2: Checkout Simulation Engine
- The system must simulate a stream of eCommerce checkout attempts at ~1-3 second intervals.
- Each simulated checkout must include:
  - `orderId` (e.g., `#SHOP-2841`)
  - `customerName` (e.g., `Sarah Chen`)
  - `amount` (random between $15 and $850)
  - `primaryGateway` (the operator's preferred gateway)
  - `timestamp`
- The system must route each checkout through the failover logic (see FR-3).
- The simulated stream must be toggleable (pause/resume) from the dashboard.

### FR-10: Flash Sale Simulation Mode
- The system must provide a "Flash Sale" button in the dashboard header.
- When clicked, the button must dump 20+ checkout attempts within 5 seconds to simulate a high-volume sales event.
- The activity feed must handle the surge — displaying all entries without lag or dropping rows.
- The routing engine must process each checkout individually through the failover logic.
- The button must be visually distinct (cyan accent or outlined) and labeled "Simulate Flash Sale."
- After the burst, the simulation must return to the normal ~1-3s interval.

### FR-3: Dynamic Health Routing Engine with Retry Loop (Simulated)
- The system must maintain `healthy`, `degraded`, or `down` status for each gateway.
- The routing algorithm must execute in **laps** — up to 3 full passes across all available gateways with increasing backoff before declaring a transaction unrecoverable.
- **Lap 1 (immediate):**
  1. Try the operator's preferred gateway first.
  2. If the preferred gateway is `down`, skip to the healthiest remaining gateway (lowest latency).
  3. If the first attempted gateway fails (simulated ~15-25% chance), try the next healthiest.
  4. If any gateway succeeds in Lap 1, the transaction is marked "Cleared" and processing stops.
- **Lap 2 (retry, 500ms backoff):**
  5. If Lap 1 exhausted all gateways unsuccessfully, pause 500ms then retry from step 1.
  6. Exclude any gateway that returned a permanent-style failure in Lap 1 (simulated ~5% chance per gateway).
  7. If any gateway succeeds in Lap 2, mark as "Rescued (retry)."
- **Lap 3 (final retry, 1000ms backoff):**
  8. If Lap 2 also failed, pause 1000ms and make a final pass through remaining gateways.
  9. If Lap 3 succeeds, mark as "Rescued (retry ×2)."
  10. If Lap 3 fails — all gateways exhausted across all laps — mark as "Declined (unrecoverable)" and log to the Reconciliation Ledger.
- The routing decision and lap count must be visible in the live activity feed for each transaction.

### FR-4: Gateway Health Monitor (Simulated)
- The system must display 4 payment gateways: Stripe, Adyen, Braintree, Checkout.com.
- Each gateway must have simulated health data:
  - Status: randomly transitions between healthy/degraded/down with healthy as default.
  - Latency: random value between 80ms and 600ms (updates periodically).
  - Uptime %: calculated from simulated uptime history.
- Health data must update every 5-10 seconds.

### FR-5: Live Operations Dashboard (Primary View)
- The dashboard must include:
  - Live checkout activity feed (real-time, auto-scrolling, latest-first).
  - Gateway health status panel (4 cards).
  - Simulated outage controls (toggle per gateway).
  - Stream pause/resume button.
  - Flash Sale simulation button (see FR-10).
- This is the landing page after login.

### FR-6: Rescued GMV & Billing View (Secondary View)
- The dashboard must include a secondary view with:
  - 5 metric cards: Total GMV, Rescued GMV, Claimable Lost GMV, Your Fee (1%), Insurance Coverage.
  - **Your Fee (1%):** Calculated as `Total GMV handled × 1%`. This is ZeroDrop's only charge — no subscription fee. Shown with a distinct accent (cyan) to separate it from operational metrics.
  - A weekly bar chart comparing rescued vs. lost GMV.
  - A donut chart showing gateway success rate distribution.
- The view must be accessible via sidebar or tab navigation.

### FR-7: Reconciliation Ledger
- The system must log every transaction where ALL gateways failed across all retry laps.
- The ledger table must include: date, order ID, customer name, amount, fee (1%), attempted gateways, retry laps attempted, final error, claim status.
- **Fee column:** Each row shows `amount × 1%` — even failed transactions generate a fee entry for transparency (though claims offset unrecoverable losses).
- The ledger must show a running monthly total of claimable lost GMV.
- Claim status must default to "Pending" and be manually changeable to "Filed" or "Settled" (simulated).

### FR-8: Gateway Preference Settings
- The system must allow the operator to set a preferred gateway.
- The preference must be persisted in browser localStorage for the prototype.
- The routing engine must read and respect this preference.

### FR-9: Navigation & Layout
- The system must have a sidebar with navigation items: Dashboard (live ops), GMV & Billing, Reconciliation Ledger, Settings.
- The active user (Adam Kogan) must be displayed in the sidebar footer.
- The header must show the page title and a simulated notification bell.

---

## 5. Non-Goals (Out of Scope)

| # | Non-Goal | Reason |
|---|----------|--------|
| NG1 | Real payment processing | Prototype only; all transactions are simulated |
| NG2 | Actual Shopify/WooCommerce plugin development | Integration surface is mocked |
| NG3 | Real gateway API connections | Gateway health is simulated |
| NG4 | Real authentication / user management | Single hardcoded user for prototype |
| NG5 | Automated insurance claim filing | Claim workflow is simulated |
| NG6 | Mobile-responsive layout | Desktop-first prototype; responsive is nice-to-have but not required |
| NG7 | Multi-merchant / tenant support | Single merchant (Adam Kogan) only |
| NG8 | Historical data persistence beyond page session | Simulated data resets on page refresh |

---

## 6. Design Considerations

All UI must follow the ZeroDrop DESIGN.md specification at the project root.

### Key design tokens to use:
- **Background:** `#FFFFFF` (white)
- **Surface:** `#F8FAFC` (light gray)
- **Primary:** `#0F172A` (deep ink)
- **Secondary/Accent:** `#0EA5E9` (sky blue — interactive) and `#22D3EE` (cyan — live indicators)
- **Typography:** Inter (body/headings), JetBrains Mono (metrics, tags, timestamps)
- **Radius:** 4px (tags/badges), 8px (buttons), 12px (cards)
- **Spacing:** 16px gaps, 24px card padding

### Layout structure:
```
┌─────────────────────────────────────────────────┐
│ SIDEBAR (260px)  │  TOP BAR (sticky)            │
│                   │                              │
│ - Dashboard ●     │  ┌──────────┐ ┌──────────┐  │
│ - GMV & Billing   │  │ Metric   │ │ Metric   │  │
│ - Ledger          │  │ Card     │ │ Card     │  │
│ - Settings        │  └──────────┘ └──────────┘  │
│                   │                              │
│                   │  ┌───────────────────────┐   │
│                   │  │ Activity Feed         │   │
│                   │  │ (live checkout stream)│   │
│                   │  └───────────────────────┘   │
│                   │                              │
│                   │  ┌──────────┐ ┌──────────┐   │
│  👤 Adam Kogan    │  │ Gateway  │ │ Gateway  │   │
│     Operator      │  │ Health   │ │ Health   │   │
│                   │  └──────────┘ └──────────┘   │
└─────────────────────────────────────────────────┘
```

### Routing decision visualization:
- Activity feed entries show the routing path with lap indicators:
  - `Stripe ✅` in green for healthy first-attempt transactions
  - `Stripe → (fail) → Adyen ✅ [L1]` in cyan for Lap-1 rescues
  - `Stripe → Adyen → (fail) → Checkout.com ✅ [L2]` in amber-cyan for Lap-2 rescues
  - `Stripe → Adyen → Braintree → Checkout.com → (retry)... ❌ [L3 exhausted]` in red for fully unrecoverable

### Status badge colors:
- Cleared: green tint `#F0FDF4` / text `#15803D`
- Rescued: cyan tint `#ECFEFF` / text `#0891B2`
- Failed: red tint `#FEF2F2` / text `#B91C1C`

---

## 7. Technical Considerations

| # | Consideration | Details |
|---|---------------|---------|
| T1 | Single HTML file | The prototype should be a single, self-contained HTML file with inline CSS and JS for easy demoing. |
| T2 | Simulated data | Use `setInterval` / `setTimeout` to generate checkout events and update gateway health. |
| T3 | State management | Maintain checkout history, gateway states, and ledger entries in JavaScript arrays/objects. |
| T4 | localStorage | Persist only the preferred gateway setting across sessions. |
| T5 | No frameworks | Vanilla HTML/CSS/JS. No React, Vue, or build tooling needed for the prototype. |
| T6 | Google Fonts | Load Inter (weights 400, 500, 600, 700) and JetBrains Mono (weight 600) from Google Fonts CDN. |
| T7 | Browser target | Latest Chrome, Firefox, Safari. No IE11 support needed. |

---

## 8. Success Metrics

For the prototype phase, success is measured by demo readiness, not production metrics:

| # | Metric | Target |
|---|--------|--------|
| M1 | Checkout simulation runs without errors | 100+ simulated checkouts in a demo session |
| M2 | Failover routing is visually traceable | Every checkout in the feed shows routing hops and outcome |
| M3 | Gateway outage simulation works | Toggling a gateway to "down" causes immediate rerouting of new checkouts |
| M4 | Reconciliation Ledger accuracy | 100% of failed checkouts appear in the ledger with correct amounts |
| M5 | Dashboard displays rescued GMV correctly | Aggregated rescued GMV matches sum of rescued checkouts in the feed |
| M6 | Prototype loads from a single HTML file | File can be opened in any modern browser without a server |

---

## 9. Open Questions

| # | Question | Decision |
|---|----------|----------|
| Q1 | Should the prototype include a "dark mode" toggle? | **No** — light mode only per DESIGN.md tokens. |
| Q2 | Should the simulated checkout stream have configurable speed? | **No** — fixed ~1-3s interval. |
| Q3 | Should there be a simulated "Flash Sale" mode? | **Yes** — button that dumps 20+ checkouts in 5 seconds, implemented as FR-10. |
| Q4 | Should the insurance claim flow include a mock "File Claim" button? | **Yes** — manual claim status toggle per FR-7. |

---

*PRD created following the [snarktank/ai-dev-tasks create-prd.md](https://github.com/snarktank/ai-dev-tasks/blob/main/create-prd.md) format.*
*Design system reference: `/DESIGN.md` (ZeroDrop design tokens).*
