# MitsuCare — Business Presentation Brief

**AI-Powered Predictive Maintenance for Mitsubishi Vehicles**

---

## 1. EXECUTIVE SUMMARY

**MitsuCare** is an intelligent predictive maintenance system that detects vehicle faults **before warning lights appear**, delivering structured diagnostics to workshop technicians and plain-language alerts to vehicle owners in real time.

**Problem:** Most vehicles break down from issues that could have been prevented. Owners miss early warning signs; workshops lack early visibility into developing problems.

**Solution:** An end-to-end AI system using:

- 30 days of OBD sensor telemetry (Track 1 — technicians)
- 12-hour daily monitoring (Track 2 — owners)
- Machine Learning classification (8 fault types / 4 risk levels)
- LLM-powered diagnostic briefs grounded in Standard Operating Procedures (RAG)
- Real-time push alerts to vehicle owners in Bahasa Indonesia

**Outcome:** Reduced vehicle downtime, lower repair costs, improved customer satisfaction, higher technician efficiency.

---

## 2. THE PROBLEM

### Current State

- **Reactive Maintenance:** Vehicles fail, owners call for roadside assistance or breakdown
- **Owner Blindness:** No early signals; repairs are unexpected and costly
- **Technician Delays:** Diagnosis takes time; customers wait for answers
- **Workshop Inefficiency:** Technicians lack structured data before physical inspection
- **Lost Revenue:** Preventive service opportunities missed; emergency repairs are more expensive

### Market Pain Points

- Average automotive breakdown costs: **IDR 500K–5M+** per incident (parts + labor + towing)
- 40%+ of failures are preventable with early detection
- Fleet operators lose productivity due to unexpected breakdowns
- Technicians spend hours on diagnostics without clear direction

### Target Markets

- **Mitsubishi Dealerships & Authorized Workshops** (>300 locations in Indonesia)
- **Commercial Fleet Operators** (taxi, ride-share, delivery)
- **Insurance & Financing Companies** (risk reduction, claim prevention)
- **Individual Vehicle Owners** (Outlander, Eclipse Cross, Xpander owners)

---

## 3. THE SOLUTION — MITSUCARE ARCHITECTURE

### Three-Tier System

```
┌─────────────────────────────────────────────────────────────┐
│              Vehicle IoT Sensors (OBD2)                     │
│    12 sensors × every 2 hours (07:00–19:00 WIB)            │
└────────────────┬────────────────────────────────────────────┘
                 │
        ┌────────┴────────┐
        ▼                 ▼
   Track 1          Track 2
   TECHNICIAN       VEHICLE OWNER
   ────────────     ──────────────
   30-day avg       12-hour avg
   8 fault classes  4 risk levels
        │                 │
        ▼                 ▼
   ML Classifier    ML Classifier
        │                 │
        ▼                 ▼
   LLM + RAG         LLM + RAG
   (English)         (Bahasa Indonesia)
        │                 │
        ▼                 ▼
   Technician       Owner Push Alert
   Fault Brief      (Mobile App)
   (Tablet App)     + Follow-up Chat
   + Follow-up Chat
```

### Key Components

| Component         | Technology                   | Purpose                                                       |
| ----------------- | ---------------------------- | ------------------------------------------------------------- |
| **Backend API**   | FastAPI (Python)             | Exposes prediction endpoints, handles requests from both apps |
| **Data Storage**  | PostgreSQL                   | Stores vehicle profiles, sensor readings, predictions         |
| **ML Models**     | scikit-learn (SVM + LogReg)  | Classifies fault/risk from 12 sensor dimensions               |
| **RAG Pipeline**  | LangChain + ChromaDB         | Retrieves relevant SOP procedures for LLM context             |
| **LLM**           | Google Gemini 2.0 Flash      | Generates structured briefs + plain-language alerts           |
| **Technician UI** | Streamlit (tablet-optimized) | Fault brief, inspection steps, follow-up chat                 |
| **Owner UI**      | Streamlit (mobile-optimized) | Risk alert, actionable steps, follow-up chat in Bahasa        |
| **Monitoring**    | Prometheus + Grafana         | Real-time system health, query rates, LLM latency             |
| **Deployment**    | Docker + Nginx               | Containerized, scalable, load-balanced across domains         |

---

## 4. TRACK 1 — TECHNICIAN FAULT DIAGNOSIS

### Input

- **Vehicle Telemetry:** 30 days of continuous OBD sensor data
- **Sensors Monitored:** O₂, MAF, Throttle Position, RPM, Cam Advance, Knock, Coolant, Oil Pressure, MAP, EGR, Battery, Fuel Temperature
- **Processing:** Aggregated into daily averages; anomalies flagged

### Output

- **Fault Class:** One of 8 types
  - Normal (no action)
  - Battery Degradation
  - Brake System Issue
  - Cooling System Problem
  - Engine Misfire
  - Alternator Failure
  - Oil Pressure Issue
  - Transmission Problem

- **Technician Fault Brief:** Structured report including
  - Fault summary (plain language)
  - Sensor readings vs. SOP thresholds
  - Step-by-step inspection procedures (from SOP)
  - Priority level (critical/high/medium)
  - Recommended action (do not drive / inspect within X days / schedule)

### Business Value

- ✅ Technicians spend 40% less time on initial diagnosis
- ✅ Reduces misdiagnosis via SOP-grounded procedures
- ✅ Enables proactive workshop scheduling
- ✅ Higher first-time-fix rate

---

## 5. TRACK 2 — VEHICLE OWNER ALERT

### Input

- **Daily Telemetry:** 12-hour window (07:00–19:00 WIB)
- **Sensors Monitored:** Same 12 plus TPMS, ambient temperature, humidity, brake events, speed
- **Processing:** Daily risk classification

### Output

- **Risk Class:** One of 4 levels
  - No Risk (Tidak Ada Risiko) — no alert
  - Low Risk (Risiko Rendah) — monitor, informational
  - Medium Risk (Risiko Sedang) — schedule maintenance
  - High Risk (Risiko Tinggi) — urgent, do not drive

- **Owner Push Alert:** Plain-language Bahasa Indonesia
  - Urgency label (e.g., "⚠️ Perhatian")
  - What's happening (in simple language, no sensor jargon)
  - Recommended action (e.g., "Kunjungi bengkel dalam 3 hari")
  - Option to chat with AI for more details

### Business Value

- ✅ Owners aware of problems before emergencies
- ✅ Increased customer loyalty via proactive care
- ✅ Higher preventive maintenance bookings
- ✅ Reduced roadside breakdowns (insurance/brand reputation)

---

## 6. TECHNOLOGY STACK & ARCHITECTURE

### Microservices Deployment

```
├── PostgreSQL Database (vehicle profiles, telemetry, predictions)
├── FastAPI Backend (prediction endpoints, orchestration)
├── Streamlit App — Technician (port 8501)
├── Streamlit App — Owner (port 8502)
├── Prometheus (metrics collection)
├── Grafana (dashboards & analytics)
├── Nginx (reverse proxy, domain routing)
```

### ML Pipeline

- **Data Input:** 12 sensor readings (numeric)
- **Feature Engineering:** Standardization, aggregation over time windows
- **Classification:**
  - **Track 1:** Support Vector Machine (SVM) — 8 classes
  - **Track 2:** Logistic Regression — 4 classes
- **Accuracy:** Validated on historical Mitsubishi dataset (600+ Track 1 samples, 140+ Track 2 samples)

### RAG Pipeline

1. **Document Loading:** Two SOP markdown files (Track 1 + Track 2)
2. **Chunking:** 500-char segments with 50-char overlap (preserves context)
3. **Embedding:** Google text-embedding-004 (semantic search)
4. **Vector Store:** ChromaDB (persistent, local)
5. **Retrieval:** Max Marginal Relevance (MMR) search, top-4 results
6. **LLM Output:** Gemini 2.0 Flash generates structured briefs

### Guardrails & Safety

- Input validation (plate number, sensor bounds)
- Output validation (brief structure, alert tone)
- Context grounding (LLM only uses retrieved SOP context, not general knowledge)
- Error handling (graceful degradation if LLM unavailable)

---

## 7. BUSINESS MODEL & REVENUE STREAMS

### B2B — Dealerships & Workshops

- **Subscription:** Per-workshop tier
  - Starter: ≤5 active vehicles, dashboards — IDR 2M/month
  - Professional: ≤50 vehicles, analytics, API access — IDR 5M/month
  - Enterprise: Unlimited, custom integrations — IDR 15M+/month

### B2C — Vehicle Owners (Direct)

- **Mobile App Subscription:** IDR 49K/month (push alerts + chat support)
- **Freemium:** Limited alerts; upgrade for full features

### B2B2C — Insurance & Financing

- **Risk Scoring API:** Per-query pricing
  - Insurance underwriting: IDR 500/vehicle/month
  - Financing risk assessment: IDR 1K/vehicle for approval decision

### Professional Services

- Implementation & integration: IDR 5M–50M
- Custom SOP documentation: IDR 2M–10M
- Training for technician teams: IDR 1M–5M

---

## 8. GO-TO-MARKET STRATEGY

### Phase 1 (Months 1–3): Pilot Program

- Partner with 1–2 Mitsubishi dealerships in Jakarta
- Deploy technician + owner apps with 50–100 vehicles
- Collect feedback, refine SOPs, validate accuracy
- **Success Metric:** >85% technician satisfaction, >75% owner engagement

### Phase 2 (Months 4–9): Regional Expansion

- Expand to 10–15 workshops across Java
- Add fleet operator partners (taxi, delivery)
- Launch B2C owner app on Play Store
- **Target:** 1,000+ active vehicles

### Phase 3 (Months 10–18): National Scale

- Nationwide dealer network integration
- Insurance company partnerships
- Financing company integrations
- **Target:** 10,000+ active vehicles, 3–5 regional service centers

---

## 9. COMPETITIVE ADVANTAGES

| Advantage                    | Why It Matters                                                 |
| ---------------------------- | -------------------------------------------------------------- |
| **AI-Grounded in SOPs**      | Not a black-box model; technicians trust explained diagnostics |
| **Dual-Track Design**        | Serves both B2B (workshops) and B2C (owners) in one system     |
| **Bahasa Indonesia Support** | Culturally localized; competitors focus on English only        |
| **RAG + LLM**                | Generates _human-like_ briefs; not just raw predictions        |
| **Containerized Deployment** | Easy integration into existing dealer IT; no massive migration |
| **Real-Time Monitoring**     | Prometheus + Grafana dashboards; ops visibility built-in       |

---

## 10. FINANCIAL PROJECTIONS (18 Months)

### Revenue Forecast

- **Month 1–3:** IDR 0 (pilot, no revenue)
- **Month 4–6:** IDR 20M (10 workshops × 2M)
- **Month 7–9:** IDR 80M (15 workshops × 5M avg + owner subs)
- **Month 10–18:** IDR 500M+ (expansion + B2B2C deals)

### Cost Structure

- **Development:** 2–3 engineers, DevOps specialist
- **Infrastructure:** AWS/Google Cloud (IDR 5M–10M/month)
- **Google API (LLM/Embeddings):** IDR 1M–3M/month (usage-based)
- **Support & Operations:** 1–2 support staff

### Break-Even

- **Estimated:** Month 10–12 (assuming 500–800 active paying vehicles)

---

## 11. RISK MITIGATION

| Risk                            | Mitigation                                                            |
| ------------------------------- | --------------------------------------------------------------------- |
| **Low adoption by technicians** | Pilot extensively; invest in training & documentation                 |
| **Privacy concerns (OBD data)** | Clear data governance, local storage option, GDPR-compliant           |
| **LLM hallucination**           | RAG-grounded context + output validators catch errors                 |
| **High infrastructure costs**   | On-premises deployment option for large partners                      |
| **Competitor entry**            | First-mover advantage in Indonesia; build deep workshop relationships |

---

## 12. SUCCESS METRICS & KPIs

### Product

- Average technician diagnostic time reduction: **>40%**
- Owner push alert engagement rate: **>60%**
- Preventive booking increase (workshop): **+35%**
- False positive rate (misdiagnosis): **<5%**

### Business

- Monthly Recurring Revenue (MRR): Target IDR 50M by month 12
- Active vehicle count: Target 5,000+ by month 18
- Customer retention: Target >90%
- Net Promoter Score (NPS): Target >70

### Technical

- API uptime: >99.5%
- LLM response time: <5 seconds (p95)
- Model accuracy (F1 score): >0.85 on both tracks

---

## 13. TEAM & EXPERTISE

### Required Roles

- **Lead Engineer** (AI/ML + Backend)
- **DevOps/Infrastructure Engineer**
- **Product Manager** (automotive domain knowledge preferred)
- **Sales & Business Development**
- **Customer Success & Implementation**

### Technical Debt & Maintenance

- Continuous SOP updates as new fault types emerge
- ML model retraining (quarterly) with new data
- LLM prompt optimization (ongoing)

---

## 14. CONCLUSION

MitsuCare transforms vehicle maintenance from **reactive crisis management** to **proactive health optimization**. By combining predictive ML, RAG-grounded LLMs, and dual-track UX, we deliver:

1. **For Workshops:** Faster, more accurate diagnostics → higher efficiency → more bookings
2. **For Owners:** Early warning + peace of mind → lower repair costs → brand loyalty
3. **For Mitsubishi:** Increased customer lifetime value, reduced warranty claims, brand differentiation

**Time to Market:** 6 months to MVP, 18 months to profitability  
**Initial Investment:** IDR 500M–800M (team + infrastructure + pilot)  
**Projected ROI:** 3–5x over 3 years

---

## APPENDIX: TECHNICAL SPECIFICATIONS

### Supported Vehicles

- Mitsubishi Outlander (2020+)
- Mitsubishi Eclipse Cross (2020+)
- Mitsubishi Xpander (2020+)
- Roadmap: All OBD2-compliant Mitsubishi models

### System Requirements

- OBD2 Bluetooth/WiFi dongle (driver-side dashboard)
- Technician: Tablet (7–10 inch, Android/iOS, Streamlit web app)
- Owner: Smartphone (iOS/Android native app, or web-based)
- Backend: 1–2 CPU, 4–8 GB RAM (scales horizontally)

### Integration Points

- **Vehicle Telematics:** OBD2 → IoT gateway → PostgreSQL
- **Workshop Systems:** API integration with existing diagnostic tools
- **Mobile Push:** Firebase Cloud Messaging (FCM) for owner alerts
- **Payment:** Stripe, GCash, DANA for B2C subscriptions

### Security

- End-to-end encrypted telemetry
- API key authentication (workshop + owner apps)
- Role-based access control (technician vs. owner vs. admin)
- Data retention policy (12 months sensor history, then anonymize)

---

**Document Version:** 1.0  
**Last Updated:** June 2026  
**Next Review:** September 2026
