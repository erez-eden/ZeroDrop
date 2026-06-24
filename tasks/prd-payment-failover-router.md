# PRD: ZeroDrop — Master Gateway Aggregator (MVP Prototype)

## 1. Introduction / Overview

ZeroDrop is a **single-contract payment infrastructure platform** for mid-market eCommerce operators in Israel. The merchant signs one agreement with ZeroDrop — no individual gateway contracts, no per-gateway monthly fees, no managing separate API keys or settlement cycles.

ZeroDrop acts as the **master merchant-of-record aggregator**, natively connected to a pre-integrated network of local payment gateways (Hyp, CreditGuard, Tranzila, Grow). During checkout, ZeroDrop dynamically routes each transaction through the healthiest available gateway with sub-400ms latency. If a gateway fails mid-session, card token metadata is preserved and seamlessly handed off to the next gateway — the customer never re-enters their card.

The merchant pays one simple, predictable fee: **40₪ per transaction**, covering all gateway costs, routing, failover, and settlement. No percentage of GMV, no monthly minimum, no hidden gateway markups.

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
| G5 | Show one consolidated bill — 40₪ × transaction count | Single invoice view, no per-gateway line items |
| G6 | Log unrecoverable failures to a Reconciliation Ledger | Ledger table with claim status and per-transaction fee |
| G7 | Let the operator configure preferred gateway | Settings panel persisted to localStorage |
| G8 | **North Star:** Demonstrate zero-failure routing during a simulated ShoppingIL spike | Flash Sale mode: 22 checkouts in 5 seconds, 0% unrecoverable rate |

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

### US-3: Operator views rescued revenue and pays one bill
*As Adam Kogan, I want to see how much revenue ZeroDrop saved by rerouting, and see exactly what I owe — one number, one bill — without digging through per-gateway settlement reports.*

**Acceptance criteria:**
- Metric cards display: Total TXNs, Rescued GMV, Total Bill (40₪ × TXNs), Claimable Lost.
- A consolidated bill summary shows: `342 transactions × 40₪ = 13,680₪`.
- No per-gateway fee breakdown — the merchant sees one fee, one bill.
- A bar chart shows rescued vs. lost GMV by day.

### US-4: Operator configures preferred gateway
*As Adam Kogan, I want to tell ZeroDrop which gateway to try first, because I may have a preference based on local market performance.*

**Acceptance criteria:**
- Settings panel lists all four pre-integrated gateways.
- Operator selects preferred gateway from dropdown.
- Routing always tries preferred first, then falls back by health.
- Preference persists via localStorage.

### US-5: Operator reviews the Reconciliation Ledger
*As Adam Kogan, I want to see every transaction where all gateways failed (all retry laps exhausted), so I can understand my exposure and file claims.*

**Acceptance criteria:**
- Ledger table: date, order ID, customer, amount, fee (40₪), gateways attempted, retry laps, claim status.
- Running monthly total of claimable lost GMV.
- Claim status toggles: Pending → Filed → Settled.

### US-6: Operator simulates outages and flash sales
*As Adam Kogan, I want to trigger gateway outages and ShoppingIL-style traffic spikes to verify ZeroDrop holds up under extreme conditions.*

**Acceptance criteria:**
- "Simulate Outage" button per gateway makes it go down and reroutes traffic instantly.
- "Recover" restores it.
- "Flash Sale" button dumps 22 checkouts in 5 seconds. Target: 0% unrecoverable.

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
- Health transitions simulated randomly; updates every 6 seconds.
- Operator can force outage and recover.

### FR-5: Live Operations Dashboard (Primary View)
- Landing page after login.
- Sections: gateway health cards, routing pipeline (animated), live checkout stream, outage/recover controls, Flash Sale button, stream pause/resume.

### FR-6: GMV & Billing View (Secondary View)
- **Metric cards:**
  - Total Transactions (count of all checkouts)
  - Rescued GMV (value saved by failover)
  - **Total Bill:** `transaction count × 40₪` — the merchant's one consolidated charge. No per-gateway breakdown.
  - Claimable Lost GMV
  - Insurance Coverage
- Weekly bar chart: rescued vs. lost GMV.
- Donut chart: routing outcomes (Cleared / Rescued / Lost).

### FR-7: Reconciliation Ledger
- Logs every transaction where all gateways + all retry laps failed.
- Columns: date, order ID, customer, amount₪, fee (40₪), gateways attempted, retry laps, claim status.
- Running monthly total of claimable lost GMV.
- Claim status: Pending → Filed → Settled (manual toggle).

### FR-8: Gateway Preference Settings
- Operator selects preferred gateway from pre-integrated list.
- Persisted to localStorage.
- Routing engine reads and honors this preference.

### FR-9: Navigation & Layout
- Sidebar: Dashboard, GMV & Billing, Ledger, Settings.
- Sidebar footer: Adam Kogan, Operator.
- Top bar: page title, Flash Sale button, stream pause/resume.

### FR-11: Consolidated Billing (One Invoice)
- The system must display one unified bill: `transactionCount × 40₪`.
- No per-gateway fees, no hidden markups, no subscription.
- The bill updates live as transactions flow through the system.
- Billing card in GMV & Billing view. Settings view explains: "One contract. One bill. 40₪/txn."

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
│                   │                           │
│ - Dashboard ●     │  ┌──────────┐ ┌───────┐  │
│ - GMV & Billing   │  │ Metric   │ │Metric │  │
│ - Ledger          │  │ Card     │ │Card   │  │
│ - Settings        │  └──────────┘ └───────┘  │
│                   │                           │
│                   │  ┌────────────────────┐   │
│                   │  │ Routing Pipeline   │   │
│                   │  │ (animated packets) │   │
│                   │  └────────────────────┘   │
│                   │                           │
│                   │  ┌────────────────────┐   │
│  👤 Adam Kogan    │  │ Activity Feed      │   │
│     Operator      │  └────────────────────┘   │
└──────────────────────────────────────────────┘
```

### Status badge colors:
- Cleared: green `#F0FDF4` / `#15803D`
- Rescued: cyan `#ECFEFF` / `#0891B2`
- Declined: red `#FEF2F2` / `#B91C1C`

---

## 7. Technical Considerations

| # | Consideration | Details |
|---|---------------|---------|
| T1 | Single HTML file | Self-contained prototype, inline CSS/JS, no server required. |
| T2 | Simulated data | setInterval/setTimeout for checkouts and gateway health. |
| T3 | State management | JavaScript arrays/objects for checkouts, gateways, ledger. |
| T4 | localStorage | Preferred gateway setting only. |
| T5 | No frameworks | Vanilla HTML/CSS/JS. No build tooling. |
| T6 | Google Fonts | Inter (400, 500, 600, 700) + JetBrains Mono (600, 700). |
| T7 | Browser target | Latest Chrome, Firefox, Safari. |

---

## 8. Success Metrics

| # | Metric | Target |
|---|--------|--------|
| M1 | Checkout simulation error-free | 100+ simulated transactions in demo |
| M2 | Failover routing visually traceable | Every checkout shows routing hops and outcome |
| M3 | Gateway outage simulation | Toggling gateway down reroutes traffic immediately |
| M4 | Reconciliation Ledger accuracy | 100% of fully-failed TXNs in ledger |
| M5 | Rescued GMV correct | Sum matches rescued checkouts |
| M6 | Single HTML file | Opens in any browser without server |
| M7 | **Consolidated bill correct** | `TXNs × 40₪` matches displayed total |
| M8 | **0% failure during Flash Sale** | 22 checkouts in 5s, zero unrecoverable |

---

## 9. Open Questions

| # | Question | Decision |
|---|----------|----------|
| Q1 | Dark mode toggle? | No — light mode per DESIGN.md |
| Q2 | Configurable stream speed? | No — fixed ~1-3s interval |
| Q3 | Flash Sale mode? | Yes — FR-10 |
| Q4 | Manual claim status toggle? | Yes — FR-7 |
| Q5 | **Pricing model?** | **40₪ flat per transaction.** No subscription, no percentage, no per-gateway fees. |
| Q6 | **Gateway integration model?** | **Pre-integrated aggregator.** Merchant signs one contract with ZeroDrop. All four gateways active out-of-the-box. |

---

*PRD format: [snarktank/ai-dev-tasks create-prd.md](https://github.com/snarktank/ai-dev-tasks/blob/main/create-prd.md)*
*Design system: `/DESIGN.md`*
*Updated: 2026 — v2 — Master Gateway Aggregator pivot*
