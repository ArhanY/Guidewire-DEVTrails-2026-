# GigShield
**AI-Powered WhatsApp Insurance for India's Gig Workers**

> Guidewire DEVTrails 2026 · Phase 1 Submission  
> Persona: Food Delivery (Zomato / Swiggy) · Platform: WhatsApp · Pricing: Weekly · Coverage: Income Loss Only

---

## The Problem

India has 12 million food delivery riders. A typical Zomato or Swiggy partner earns Rs.12,000–20,000 per month working 8–10 hours daily. Their income is entirely on-road time. When it rains 94mm in Andheri on a Tuesday morning, Arjun earns Rs.0 for 5 hours. No insurance product in India covers this.

Existing products fail: annual premiums don't match weekly earnings; claims need app logins a rider has no time for; distribution apps are never opened between shifts.

---

## The Solution

GigShield has no worker-facing app or website. The entire rider experience lives in WhatsApp. When rain hits: *"Rs.643 is being sent to your UPI."* He did nothing. He never opened an app. He never filed a claim.

This is a distribution strategy. WhatsApp is open on every rider's phone 6-8 hrs/day. A message that arrives requires zero behaviour change.

---

## Three Actors

| Actor | Interface | Role |
|---|---|---|
| Gig Worker | WhatsApp only | Registers once. Replies 1/2 every Monday. Receives payouts. Never files a claim. |
| AI Backend | Invisible server | Polls 9 disruption APIs every 5 min. Validates fraud. Fires UPI payout. 24/7. |
| Insurance Admin | React web dashboard | Loss ratio, live claims, fraud queue, Trust Score chart, 72hr forecast. |

---

## Persona Scenarios

**Scenario A — Arjun Kumar · Zomato · Andheri West, Mumbai · Trust Score 74 (Gold)**
Tuesday 7:45am. IMD: 94mm/hr rainfall. GigShield detects alert, validates, fraud check score 14/100, Razorpay fires. 8:23am: Rs.643 sent to arjun.kumar@paytm. He did nothing.

**Scenario B — Priya Mehta · Swiggy · Bandra, Mumbai · Trust Score 61 (Silver)**
Maharashtra bandh, 80% zone impact. Zero orders for 6hrs confirmed. Rs.512 payout sent before Priya sees any news notification.

**Scenario C — Ravi Sharma · Zomato · Delhi NCR · Trust Score 81 (Platinum)**
CPCB AQI 418 (threshold 400). Platform suspends deliveries. Rs.480 for 6 suspended hours. Ravi's Platinum premium: Rs.29/week — 16.5x return.

---

## Weekly Premium Model

```
Weekly Premium = Base Rate x Zone Risk Multiplier x Trust Score Discount x Season Factor
```

| Trust Score | Tier | Discount | Range |
|---|---|---|---|
| 0-40 | Bronze | 0% | Rs.45-72 |
| 41-65 | Silver | 10% | Rs.41-62 |
| 66-80 | Gold | 20% | Rs.33-52 |
| 81-100 | Platinum | 30% | Rs.27-45 |

| Plan | Premium | Daily Cap | Triggers |
|---|---|---|---|
| Basic Shield | Rs.27-52 | Rs.400 | 5 |
| Full Shield | Rs.41-72 | Rs.800 | 9 |
| Max Shield | Rs.55-95 | Rs.1,200 | 9 + extended hours |

Season factor: +15% monsoon (Jun-Sep) · +10% winter fog NCR (Nov-Jan) · +8% pre-monsoon heat (Apr-May)
Auto-renewal: opt-out — renews every Monday unless rider texts PAUSE

---

## 9 Parametric Triggers

| # | Trigger | Source | Threshold | Payout |
|---|---|---|---|---|
| 1 | Extreme rainfall | IMD + OpenWeatherMap | >80mm/hr for 2+ hrs | 100% |
| 2 | Flood zone alert | IMD + Civic API | Zone advisory issued | 100% |
| 3 | Extreme heat | IMD Temperature | >44C for 4+ hrs | 80% |
| 4 | Severe AQI | CPCB AQI mock | AQI >400 in zone | 60% |
| 5 | Dense fog | IMD Visibility | <50m for 3+ hrs | 70% |
| 6 | Cyclone / storm | IMD Cyclone Tracker | Red alert | 100% |
| 7 | City bandh / strike | Civic alert mock | 70%+ zone impact | 100% |
| 8 | Platform suspension | Zomato/Swiggy mock | Area suspension | 90% |
| 9 | Curfew / Sec 144 | Govt. notification mock | Official order | 100% |

Payout = min(hours_lost x hourly_rate x trigger_%, daily_cap)
Claim SLA: under 90 seconds. Zero rider action required.

---

## WhatsApp Command Reference

| Command | Function |
|---|---|
| PLAN | Personalised weekly pricing options |
| 1 / 2 / 3 | Activate Basic / Full / Max Shield |
| STATUS | Live weather + AQI in your zone |
| HISTORY | All payouts — last 4 weeks |
| SCORE | Trust Score breakdown + how to improve |
| PAUSE | Skip next week (no penalty) |
| RENEW | Re-activate same plan |
| HELP | Full commands in Hindi + English |

---

## AI/ML Plan

**Module 1 — Trust Score (XGBoost, weekly batch)**
Inputs: zone stability 25%, hours consistency 20%, claim history 25%, platform rating 15%, login behaviour 15%
Output: score 0-100 per rider, every Sunday night

**Module 2 — Dynamic Premium (rule-based + ML coefficients)**
Formula runs every Monday using latest Trust Score → personalised premium sent to rider

**Module 3 — Fraud Detection: Isolation Forest (real-time per claim)**
6 signals: GPS zone match, delivery activity, claim frequency, plan upgrade timing, Trust Score, cluster detection
Thresholds: <70 auto-approve · 70-84 delayed+admin · >=85 blocked

| Fraud Pattern | Score | Decision |
|---|---|---|
| GPS zone mismatch (>20km) | 85-95 | Auto-block |
| Active deliveries during claim | 80-90 | Auto-block |
| Duplicate same zone/week | — | Hardcoded block |
| Pre-event upgrade (<6hrs) | 60-75 | Delayed |
| Cluster fraud (5+ riders) | 75-85 | Admin flagged |

**Module 4 — 72-Hour Forecast (Prophet time-series)**
Trained on 5 years IMD data + bandh records. Output: risk heatmap by zone shown in insurer dashboard. Updated every 6 hours.

---

## Tech Stack

| Layer | Technology | Cost |
|---|---|---|
| Worker interface | Meta WhatsApp Business Cloud API | Free (1000 conv/month) |
| Bot server | Node.js + Express | Free (Railway) |
| Database | PostgreSQL via Supabase | Free tier |
| Disruption Mesh | Node.js cron + OpenWeatherMap | Free tier |
| Fraud detection | Python FastAPI + Isolation Forest | Free (Railway) |
| Trust Score ML | Python + XGBoost + Prophet | Free (open source) |
| Payments | Razorpay Test Mode | Free |
| Insurer dashboard | React + Recharts + ShadCN UI | Free (Vercel) |
| Dev tunneling | ngrok | Free tier |

---

## Database Schema

| Table | Key Columns |
|---|---|
| riders | phone, partner_id, upi_id, zone_id, trust_score, tier |
| policies | rider_id, plan, week_start, week_end, premium_paid, payout_used, payout_cap, status |
| disruptions | trigger_type, zone_id, severity, started_at, ended_at, hours_duration |
| claims | rider_id, disruption_id, payout_amount, fraud_score, fraud_flags, status, paid_at |
| trust_scores | rider_id, week, score, zone_factor, hours_factor, claim_factor, rating_factor |
| payments | claim_id, razorpay_order_id, upi_id, amount, status, initiated_at, confirmed_at |

---

## 6-Week Plan

| Phase | Dates | Deliverables |
|---|---|---|
| Phase 1 | Mar 4-20 | README, scenarios, triggers, premium model, tech stack, 2-min video |
| Phase 2 Wk 3 | Mar 21-28 | WhatsApp bot, registration, Supabase schema, Monday cron, policy activation |
| Phase 2 Wk 4 | Mar 29-Apr 4 | 5 live triggers, ACE engine, Razorpay test payout, full E2E demo loop |
| Phase 3 Wk 5 | Apr 5-11 | Isolation Forest microservice, GPS check, Trust Score XGBoost, cluster detection |
| Phase 3 Wk 6 | Apr 12-17 | React insurer dashboard, 72hr forecast, Hindi bot, 5-min demo video, pitch deck |

---

## Business Viability

| Metric | Value |
|---|---|
| Total addressable market | Rs.32,448 crore/year |
| Year 1 target | ~180,000 riders (5% Tier-1 cities) |
| Projected revenue Year 1 | Rs.48.7 crore (~$5.8M) |
| Target loss ratio | 58-65% |
| Combined ratio target | 85-90% (profitable) |
| WhatsApp API cost | Rs.0.20 per rider/week (0.4% of premium) |
| Break-even | ~14,000 active subscribers |
| Distribution | WhatsApp link in Zomato/Swiggy partner app — Zero CAC |
| Regulatory path | IRDAI Sandbox for parametric microinsurance |

---

## PDF Constraints Compliance

| Requirement | Status |
|---|---|
| Exclude health, life, accident, vehicle | COMPLIANT — all 9 triggers are income-loss events only |
| Weekly pricing model | COMPLIANT — Mon-Sun cycle, weekly premium and payout cap |
| Food delivery persona only (Zomato/Swiggy) | COMPLIANT — Partner ID validation at registration |
| Loss of income ONLY | COMPLIANT — payout = hours_lost x hourly_rate x trigger_% |
| Automated coverage and payouts | COMPLIANT — zero rider action, end-to-end automatic |
| AI-powered risk assessment | COMPLIANT — XGBoost Trust Score + dynamic premium |
| Intelligent fraud detection | COMPLIANT — Isolation Forest, 6 signals, real-time |
| Real-time parametric trigger monitoring | COMPLIANT — 9 triggers every 5 minutes |
| Integration (APIs, payments) | COMPLIANT — OpenWeatherMap, CPCB mock, Razorpay, WhatsApp API |
| Insurer analytics dashboard | COMPLIANT — React dashboard: loss ratio, claims, fraud queue, forecast |

---

*GigShield — Income Loss Only · No health/life/accident/vehicle payouts · Weekly pricing · Zomato/Swiggy food delivery*
