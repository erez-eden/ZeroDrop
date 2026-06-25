# PRD: ZeroDrop — Master Gateway Aggregator (MVP Prototype)

## 1. Introduction / Overview

ZeroDrop is a **single-contract payment infrastructure platform** for mid-market eCommerce operators in Israel. The merchant signs one agreement with ZeroDrop — no individual gateway contracts, no per-gateway monthly fees, no managing separate API keys or settlement cycles.

ZeroDrop acts as the **master merchant-of-record aggregator**, natively connected to a pre-integrated network of local payment gateways (Hyp, CreditGuard, Tranzila, Grow). During checkout, ZeroDrop dynamically routes each transaction through the healthiest available gateway with sub-400ms latency. If a gateway fails mid-session, card token metadata is preserved and seamlessly handed off to the next gateway — the customer never re-enters their card.

The merchant pays a tiered fee: **1% of transaction amount** for standard routing (cleared on first attempt), and **2% for insurance-covered transactions** (rescued by retry or declined and covered by ZeroDrop insurance). No monthly minimum, no hidden gateway markups.

**North Star KPI:** Achieve a **0% checkout failure rate** due to payment network crashes during major regional traffic spikes (e.g., ShoppingIL), resulting in a measurable conversion lift at the payment step versus historical single-gateway baselines.

**This is a frontend prototype.** Login, checkout data, store connections, gateway health, billing, and ledger are all simulated — no real payment processing or backend integration exists.

**Prototype user:** Adam Kogan (mid-market eCommerce operator, Shopify store).

---

## 2. Goals

| # | Goal | Measurement |
|---|------|-------------|
| G1 | Demonstrate one-click onboarding — merchant sees a fully operational multi-gateway network immediately | Simulated gateway network active on first login, no setup required |
| G2 | Show dynamic failover with transparent token handoff | Routing pipeline shows sub-400ms gateway-to-gateway transitions |
| G3 | Provide a live operations dashboard with real-time checkout visibility | Simulated data stream with routing path traces |
| G4 | Display Rescued GMV — revenue saved by failover routing | Aggregated value of transactions where fallback succeeded |
| G5 | Show one consolidated bill — tiered 1% / 2% fee | Single invoice view, no per-gateway line items |
| G6 | Display auto-insurance coverage and payout projections | Billing view shows Covered by Insurance total and Payout Due with end-of-next-month date |
| G7 | Let the operator configure preferred gateway and routing | Settings panel with routing params, insurance limits, and security toggles — persisted to localStorage |
| G8 | **North Star:** Demonstrate zero-failure routing during a simulated ShoppingIL spike | Flash Sale mode: burst checkouts at 200ms intervals, auto-rerouting around down gateways |

---

## 3. User Stories

### US-1: Operator sees instant redundancy out of the box
*As Adam Kogan, a Shopify merchant who just signed up for ZeroDrop, I want to log in and immediately see all four payment gateways active and routing transactions — no manual gateway setup, no API keys, no contracts to chase.*

**Acceptance criteria:**
- On first login, all four gateways (Hyp, CreditGuard, Tranzila, Grow) are pre-connected and healthy.
- The dashboard shows live transactions routing through multiple gateways immediately.
- No "connect gateway" or "enter API key" workflow exists.

### US-2: Operator monitors live checkouts and routing paths
*As Adam Kogan, I want to watch each checkout fly through the gateway routing pipeline in real time, so I can verify failover is instantaneous and invisible to customers.*

**Acceptance criteria:**
- Pipeline animation shows packets flowing from Cart → gateway nodes → Outcome.
- Each gateway flashes green (pass) or red (fail) as packets transit.
- Feed shows status: Cleared / Rescued / Declined with retry lap count.
- Sub-400ms transitions are visually fast (animation compressed to ~500ms per hop for demo).

### US-3: Operator views rescued revenue, insurance coverage, and consolidated bill
*As Adam Kogan, I want to see how much revenue ZeroDrop saved by rerouting, what's covered by insurance, and see exactly what I owe — one number, one bill — without digging through per-gateway settlement reports.*

**Acceptance criteria:**
- Dashboard metric cards: Transactions, Total Volume, Success Rate (%), Rescued count, Covered by Insurance (₪ auto-paid next month).
- GMV & Billing view shows 4-card summary: Total GMV, Covered by Insurance, Total Fees, Payout Due.
- 13-month stacked bar chart shows GMV breakdown (Cleared / Rescued / Lost) with current month highlighted.
- The 2% fee applies to rescued (auto-covered) and declined (insurance-claimable) transactions; 1% applies to standard cleared transactions.

### US-4: Operator configures preferred gateway
*As Adam Kogan, I want to tell ZeroDrop which gateway to try first, because I may have a preference based on local market performance.*

**Acceptance criteria:**
- Settings panel lists all four pre-integrated gateways.
- Operator selects preferred gateway from dropdown.
- Routing always tries preferred first, then falls back by health.
- Preference persists via localStorage.

### US-5: Operator views auto-insurance coverage
*As Adam Kogan, I want to understand what's automatically covered by ZeroDrop's insurance, how much is being paid out each month, and see insurance status per transaction — without filing any claims.*

**Acceptance criteria:**
- Rescued and declined transactions are automatically covered by insurance. No manual claim filing needed.
- Payouts are automatically processed by the last business day of the following month.
- The Billing page shows "Covered by Insurance" total (rescued GMV), "Payout Due" total (rescued + declined), and the payout date.
- Per-transaction details (modal) show ★ Covered or ★ Insured badge with payout timeline.

### US-6: Operator watches automatic failover under real conditions
*As Adam Kogan, I want to see ZeroDrop handle gateway degradation and outages automatically, and test extreme traffic spikes to verify reliability.*

**Acceptance criteria:**
- Gateway health changes automatically every 6s — healthy → degraded → down → recovery with varying probabilities.
- No manual outage buttons needed — gateways randomly go down and recover on their own.
- "Flash Sale" button dumps checkouts at 200ms intervals until manually stopped. Simulates ShoppingIL traffic bursts.
- Routing visual pipeline handles automatic gateway outages transparently — packets reroute in real time.

### US-7: Operator configures routing strategy and features
*As Adam Kogan, I want to configure how ZeroDrop routes and optimizes my transactions, including timeout thresholds, retry strategy, and AI-powered acceptance optimization.*

**Acceptance criteria:**
- Settings page provides 6 cards: Account & Store, Integration, Routing Configuration, Insurance & Billing, Security, Notifications.
- Routing Configuration: preferred gateway, timeout (ms), backoff delay (ms), retry lap count — all persisted to localStorage.
- Acceptance Boost toggle: AI-optimized gateway timing/formats, reduces failure rates ~50%. Visible "🚀 Boost Active" badge on dashboard when enabled.
- Insurance & Billing shows tiered fee rates (1%/2%), coverage limit, and auto-payout schedule.
- Security: tokenization status (always on), 3D Secure enforcement toggle.
- Notifications: configurable toggles for gateway alerts and rescue event alerts.

---

## 4. Functional Requirements

### FR-1: Simulated Authentication
- Login screen with pre-filled credentials for Adam Kogan.
- Any password accepted. Redirects to dashboard.
- No real auth backend.

### FR-2: Checkout Simulation Engine
- Simulated checkout stream at ~1-3 second intervals.
- Each simulated checkout includes:
  - `orderId` (e.g., `#SHOP-2841`)
  - `customerName`
  - `amount₪` (random between 50₪ and 2,500₪)
  - `preferredGateway` (the operator's chosen gateway)
  - `timestamp`
- Every checkout routes through the retry-loop engine (FR-3).
- Stream is toggleable (pause/resume).
- **All four gateways are pre-integrated** — no merchant setup required.

### FR-10: Flash Sale Simulation Mode
- "Flash Sale" button dumps 22 checkouts in 5 seconds.
- Pipeline and activity feed handle surge without lag.
- Target: demonstrate 0% unrecoverable during the burst.
- Cyan-accented button in dashboard header.

### FR-3: Dynamic Health Routing with Retry Loop & Token Preservation
- System maintains `healthy`, `degraded`, or `down` status per gateway.
- **Token handoff:** When a gateway fails mid-session, raw card token metadata is securely preserved and passed to the next gateway. Customer never re-enters payment details. The handoff must be sub-400ms (simulated as instant in prototype, with visual staging).
- Routing: up to 3 laps with increasing backoff:
  - **Lap 1:** immediate pass. Preferred → healthiest remaining.
  - **Lap 2:** +500ms backoff. Excludes hard-fail gateways.
  - **Lap 3:** +1000ms backoff. Final attempt.
- All 3 laps exhausted → "Declined (unrecoverable)" → logged to ledger.

### FR-4: Gateway Health Monitor (Simulated)
- Four gateways: Hyp, CreditGuard, Tranzila, Grow.
- Displayed as health cards with: status, latency, 1h uptime %.
- Health transitions simulated with random probabilities every 6 seconds:
  - Healthy → Degraded (6%), Healthy → Down (2% sudden outage)
  - Degraded → Down (10%), Degraded → Healthy (22% auto-recovery)
  - Down → Degraded (18%), Down → Healthy (4% direct recovery)
- All transitions and recoveries are automatic — no manual intervention.

### FR-5: Live Operations Dashboard (Primary View)
- Landing page after login.
- Top bar: "🚀 Boost Active" badge (when enabled), Flash Sale button, stream pause/resume.
- 5-card metric bar: Transactions, Total Volume, Success Rate, Rescued, Covered by Insurance.
- Cart card → Gateway row → Completed Transactions feed with animated routing packets.
- Stats update live as transactions flow through the system.

### FR-6: GMV & Billing View (Secondary View)
- **4-card metric summary:**
  - Total GMV (month-to-date volume)
  - Covered by Insurance (rescued amount, auto-covered)
  - Total Fees (1% standard / 2% insurance, consolidated)
  - Payout Due (insured amount: rescued + declined, due end of next month)
- **13-month stacked bar chart:** Cleared (green), Rescued (cyan), Declined/Lost (red). Current month highlighted ★. Previous 12 months simulated.

### FR-7: Insurance Coverage (Auto-Paid)
- All rescued and declined transactions are automatically covered by ZeroDrop's insurance.
- No manual claim filing — payouts are automatically processed by the last business day of the following month.
- Insurance coverage limit configurable in Settings (default: ₪50,000 per transaction).
- Payout date is dynamically computed as end of next month and displayed in billing modals.
- Fee is 2% for all insurance-covered transactions (rescued or declined), 1% for standard cleared.

### FR-7a: Transaction Detail Modal
- Clicking "View →" on any Completed Transactions row opens a modal.
- Modal shows: Order ID, Customer, Amount, Status (with ★ Covered / ★ Insured badge), Final Gateway, Routes Tried, Laps, Processing Fee (with rate), Insurance payout timeline, Timestamp.

### FR-8: Gateway Preference & Settings Page
- 6-card settings page in 2-column grid.
- Cards: Account & Store (profile), Integration (Shopify status, API key, webhook), Routing Configuration (preferred gateway, timeout, backoff, retry laps), Insurance & Billing (coverage limit, fee rates, payout schedule), Security (tokenization, 3D Secure toggle), Notifications (gateway down alert, rescue alert toggles).
- All settings persisted to localStorage.
- Preferred gateway persisted separately.

### FR-8a: Acceptance Boost (Authorization Optimization)
- Toggle in Settings → Routing Configuration.
- When enabled, reduces gateway failure rates by 50% across all retry laps (simulated AI optimization of transaction timing and formatting per gateway).
- "🚀 Boost Active" badge appears in dashboard topbar when enabled.
- State persisted to localStorage (`zdAcceptBoost`).

### FR-9: Navigation & Layout
- Sidebar: Dashboard, GMV & Billing, Settings (3-item nav).
- Sidebar footer: Adam Kogan, Operator.
- Top bar: page title, Boost indicator, Flash Sale button, stream pause/resume.

### FR-11: Consolidated Billing (One Invoice)
- The system must display one unified bill with tiered fee structure (1% standard / 2% insurance).
- No per-gateway fees, no hidden markups, no subscription.
- The bill updates live as transactions flow through the system.
- Billing card in GMV & Billing view. Settings view explains: "One contract. One bill. 1% standard / 2% insurance."

---

## 5. Non-Goals (Out of Scope)

| # | Non-Goal | Reason |
|---|----------|--------|
| NG1 | Real payment processing | Prototype — all transactions simulated |
| NG2 | Actual Shopify/WooCommerce plugin | Integration surface mocked |
| NG3 | Real gateway API connections | Gateway health simulated |
| NG4 | Real authentication | Single hardcoded user |
| NG5 | Real card token handling / PCI compliance | Token preservation described but not implemented in prototype |
| NG6 | Mobile-responsive layout | Desktop-first prototype |
| NG7 | Multi-merchant / tenant support | Single merchant (Adam Kogan) |
| NG8 | Historical persistence beyond session | Data resets on refresh |
| NG9 | Per-gateway settlement reports | Merchant sees one consolidated bill by design |
| NG10 | Manual gateway connection / API key entry | All gateways pre-integrated by ZeroDrop |

---

## 6. Design Considerations

All UI follows `DESIGN.md` at the project root.

### Key design tokens:
- **Background:** `#FFFFFF`
- **Surface:** `#F8FAFC`
- **Primary:** `#0F172A` (deep ink)
- **Secondary/Accent:** `#0EA5E9` (sky blue), `#22D3EE` (cyan)
- **Typography:** Inter (body/headings), JetBrains Mono (metrics, tags, timestamps)
- **Radius:** 4px (badges), 8px (buttons), 12px (cards)

### Layout:
```
┌──────────────────────────────────────────────┐
│ SIDEBAR (260px)  │  TOP BAR (sticky)         │
│                   │  🚀 Boost   ⚡Flash  ⏸   │
│ - Dashboard ●     │                           │
│ - GMV & Billing   │  ┌──────────────────────┐ │
│ - Settings        │  │ 5-card Metric Bar    │ │
│                   │  │ (TXNs, Vol, Rate,    │ │
│                   │  │  Rescued, Covered)   │ │
│                   │  └──────────────────────┘ │
│                   │                           │
│                   │  ┌────┐                   │
│                   │  │Cart│  ← checkout       │
│                   │  └──┬─┘                   │
│  👤 Adam Kogan    │  ↓  ↓  ↓  ↓              │
│     Operator      │  GW1→GW2→GW3→GW4         │
│                   │  (animated packets)       │
│                   │                           │
│                   │  ┌──────────────────────┐ │
│                   │  │ Completed            │ │
│                   │  │ Transactions Feed    │ │
│                   │  └──────────────────────┘ │
└──────────────────────────────────────────────┘
```

### Status badge colors:
- Cleared: green `#F0FDF4` / `#15803D`
- Rescued/Covered: cyan `#ECFEFF` / `#0891B2`
- Declined: red `#FEF2F2` / `#B91C1C`

---

## 7. Technical Considerations

| # | Consideration | Details |
|---|---------------|---------|
| T1 | Single HTML file | Self-contained prototype, inline CSS/JS, no server required. |
| T2 | Simulated data | setInterval/setTimeout for checkouts and gateway health. |
| T3 | State management | JavaScript arrays/objects for checkouts, gateways, ledger. |
| T4 | localStorage | Preferred gateway, routing config (timeout, backoff, laps), coverage limit, notification toggles, acceptance boost, 3D Secure enforcement. |
| T5 | No frameworks | Vanilla HTML/CSS/JS. No build tooling. |
| T6 | Google Fonts | Inter (400, 500, 600, 700) + JetBrains Mono (600, 700). |
| T7 | Browser target | Latest Chrome, Firefox, Safari. |

---

## 8. Success Metrics

| # | Metric | Target |
|---|--------|--------|
| M1 | Checkout simulation error-free | 100+ simulated transactions in demo |
| M2 | Failover routing visually traceable | Every checkout shows routing hops and outcome |
| M3 | Automatic gateway outages trigger rerouting | Down gateways skipped within one health tick (~6s), traffic shifts to healthy gateways |
| M4 | Insurance tracking accurate | Rescued count matches filtered checkouts; Payout Due = rescued + declined GMV |
| M5 | Covered by Insurance GMV correct | Sum matches rescued checkouts, displayed on dashboard + billing |
| M6 | Single HTML file | Opens in any browser without server |
| M7 | **Consolidated bill correct** | Tiered fees (1% / 2%) summed correctly: cleared at 1%, rescued + declined at 2% |
| M8 | **Acceptance Boost impact visible** | Toggle ON → success rate rises measurably, rescued rate drops |
| M9 | All settings persisted | Preferred gateway, routing config, boost toggle, notifications survive browser refresh |

---

## 9. Open Questions

| # | Question | Decision |
|---|----------|----------|
| Q1 | Dark mode toggle? | No — light mode per DESIGN.md |
| Q2 | Configurable stream speed? | No — fixed ~1-3s interval |
| Q3 | Flash Sale mode? | Yes — FR-10. Toggle on/off, runs until manually stopped. |
| Q4 | Manual claim filing? | **No — auto-paid insurance.** All insured transactions (rescued + declined) are auto-settled by end of next month. Zero operator action needed. |
| Q5 | **Pricing model?** | **Tiered percentage: 1% for standard cleared transactions, 2% for insurance-covered transactions (rescued + declined).** |
| Q6 | **Gateway integration model?** | **Pre-integrated aggregator.** Merchant signs one contract with ZeroDrop. All four gateways active out-of-the-box. |
| Q7 | Manual gateway outage? | **No — automatic random outages.** Health transitions simulated every 6s. Gateways degrade and recover on their own. |
| Q8 | Acceptance Boost? | **Yes — FR-8a.** AI-optimized gateway timing/formats reduces failure rates ~50%. Toggle in Settings.

---

*PRD format: [snarktank/ai-dev-tasks create-prd.md](https://github.com/snarktank/ai-dev-tasks/blob/main/create-prd.md)*
*Design system: `/DESIGN.md`*
*Updated: June 2026 — v3 — Auto-insurance model, Acceptance Boost, automatic outages*
