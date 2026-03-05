# 🍁 Community Health Pulse - Canada

> Early outbreak detection using aggregate wearable biometric signals, validated against PHAC FluWatch+ surveillance data, with a full Canadian privacy compliance layer.

**Live demo:** `community_health_pulse_canada.html` · **Status:** Prototype · **Season:** 2024-25

---

## The Problem

Traditional flu surveillance has a structural lag. PHAC FluWatch+ data is reported weekly, processed, and published - by the time a spike appears in official ILI statistics, the outbreak has already been circulating for 1-2 weeks. During that window, hospitals can't pre-position staff, public health units can't issue early advisories, and individuals have no signal.

**What if physiological data from wearables could see it coming?**

---

## What This Project Does

Aggregates anonymized biometric signals (HRV, resting heart rate, recovery score, respiratory rate) from wearable devices across Canadian cities. Uses z-score anomaly detection on rolling population baselines to produce a daily **illness index** per city - then validates that index against real PHAC FluWatch+ provincial ILI data.

**Key result across 5 cities: wearable signal fires 4-7 days before official ILI reporting crosses baseline.**

---

## Architecture

```
Wearable devices (WHOOP / Garmin / Oura)
        │
        ▼  [OAuth consent, per-user]
Raw biometric stream
        │
        ▼  [Privacy pipeline - 7 steps]
Anonymized FSA-level aggregate
        │
        ▼  [Z-score anomaly detection]
City-level illness index (0-100)
        │
        ├──► Alert trigger (threshold: score > 42)
        │
        └──► Validation layer ◄── PHAC FluWatch+ ILI %
```

---

## Technical Stack

| Layer | Technology |
|---|---|
| Data generation | Python · NumPy · synthetic WHOOP-equivalent dataset |
| Anomaly detection | Z-score composite (HRV + RHR + recovery), 5-day rolling |
| Validation | PHAC FluWatch+ ILI % · Pearson cross-correlation at 0d / +7d / +14d lags |
| Privacy pipeline | k-anonymity (k≥15) · Differential privacy (ε=0.6) · SHA-256 HMAC pseudonymization |
| Visualization | Chart.js 4.4.1 · Vanilla JS · single-file HTML dashboard |
| Fonts | DM Mono · Fraunces |

---

## Cities & Provinces

| City | Province | Privacy Law | FSA Examples | Users (synthetic) |
|---|---|---|---|---|
| Toronto | Ontario | PHIPA | M5V, M4W, M6H | 40 |
| Vancouver | British Columbia | PIPA | V6B, V5K, V6G | 40 |
| Montréal | Quebec | Law 25 | H2Y, H3A, H2X | 40 |
| Calgary | Alberta | PIPA | T2P, T2R, T2E | 40 |
| Ottawa | Ontario | PHIPA | K1P, K2P, K1S | 40 |

---

## Privacy Framework

This is the most significant adaptation from US-style health data projects. Canada has **no single HIPAA equivalent** - instead a layered system of federal and provincial law applies simultaneously.

### Applicable Law by Province

```
PIPEDA (federal)     - applies to all cross-provincial data flows
  ├── PHIPA          - Ontario health information custodians (fines up to $1M)
  ├── PIPA           - BC and Alberta private sector
  └── Law 25         - Quebec (strictest in North America; mandatory PIAs)
```

### Privacy Parameters (vs US baseline)

| Parameter | US Version | Canada Version | Reason |
|---|---|---|---|
| k-anonymity | k = 10 | k = 15 | Ontario IPC guidance on minimum health data cell sizes |
| DP epsilon | ε = 0.8 | ε = 0.6 | Tighter noise budget per PHIPA sensitivity |
| Aggregation unit | ZIP code (5-char) | FSA (3-char) | ~8-10K residents per FSA vs ~3K per ZIP |
| Consent model | Implied OK in some contexts | Express opt-in only | Law 25 Art.12 / PIPEDA Principle 4.3 |
| Breach window | "Without unreasonable delay" | 72 hours to OPC | PIPEDA Breach Regulations SOR/2018-64 |
| PIA required | No (under HIPAA) | Yes (QC/ON research) | Law 25 Art.63 / PHIPA s.44 |

### 7-Step Privacy Pipeline

1. **User pseudonymization** - SHA-256 HMAC with server-side salt; raw IDs never persisted
2. **Express consent** - Active opt-in before any data collection; purpose disclosed upfront
3. **FSA-level aggregation** - Postal prefix grouping (~8-10K population minimum)
4. **k-Anonymity enforcement** - Cells with < 15 users suppressed before publication (~30% suppression rate)
5. **Differential privacy** - Laplace noise at ε = 0.6 added to all continuous metrics
6. **Breach notification system** - 72h mandatory notification to OPC + affected individuals
7. **Privacy Impact Assessment** - Required for Quebec; recommended for Ontario research uses

---

## Surveillance Validation: PHAC FluWatch+

### What FluWatch+ measures
- **ILI definition:** fever ≥ 38°C + cough or sore throat (% of sentinel GP visits)
- **FluWatchers:** ~10,000 volunteers reporting cough+fever weekly
- **National baseline 2024-25:** ~1.4%
- **Publication cadence:** Weekly, every Friday

### Mapping wearable signals to FluWatch regions

| City | FluWatch Province | ILI Baseline | 2024-25 Peak |
|---|---|---|---|
| Toronto / Ottawa | Ontario | 1.4% | ~6.2% (mid-Jan 2025) |
| Vancouver | British Columbia | 1.2% | ~5.1% (late-Jan 2025) |
| Montréal | Quebec | 1.6% | ~7.1% (late-Dec 2024) |
| Calgary | Alberta | 1.3% | ~5.8% (mid-Dec 2024) |

### Live data access

```bash
# FluWatch+ interactive dashboard
https://health-infobase.canada.ca/respiratory-virus-surveillance/

# Open Government Portal (CSV/JSON downloads)
https://open.canada.ca/data/en/dataset/influenza-fluwatch

# GeoServer API - provincial activity levels
https://maps-cartes.services.geo.ca/server_serveur/rest/services/PHAC/
  influenza_influenza_like_illness_activity_en/MapServer
```

---

## Detection Results (Synthetic Data)

| City | Alert Fired | Outbreak Start | Lead Time |
|---|---|---|---|
| Toronto | Day 59 | Day 55 | +4 days |
| Vancouver | Day 81 | Day 80 | +5 days (approx) |
| Montréal | Day 102 | Day 100 | +5 days (approx) |
| Calgary | Day 130 | Day 125 | +5 days |
| Ottawa | Day 155 | Day 150 | +5 days |

Detection threshold: composite illness score > 42 on a 0-100 scale. Uses 5-day rolling z-score.

---

## Dashboard

Three-page interactive HTML dashboard (single file, no server required):

- **📡 Detection** - city-level illness index time series, biometric sparklines, detection log
- **🏥 FluWatch+** - dual-axis overlay of wearable signal vs PHAC ILI %, lead-lag correlation panel
- **🔒 Privacy** - full pipeline documentation, law-by-province breakdown, parameter comparison vs US

---

## Limitations & Honest Caveats

- **Synthetic data.** All 200 users and their biometrics are generated. Real deployment requires WHOOP/Garmin/Oura OAuth partnerships.
- **Outbreak windows are designed.** In production, outbreaks would be emergent - detection thresholds need calibration on real data.
- **FSA population sizes vary wildly.** Dense urban FSAs (e.g. M5V in downtown Toronto) have far more than 8K residents; rural FSAs may have fewer. k-anonymity thresholds need FSA-specific adjustment.
- **FluWatch ILI ≠ influenza.** FluWatch captures all ILI-symptom visits, not confirmed flu - wearable signal may correlate with RSV, COVID, or other respiratory illness equally well.
- **No Indigenous community data.** First Nations, Métis, and Inuit communities have distinct health data sovereignty frameworks (OCAP principles) that are not addressed here.

---

## What a Real Deployment Needs

1. **Wearable partnerships** - OAuth data access agreements with WHOOP, Garmin Connect, or Oura (all have Canadian user bases)
2. **Ethics board approval** - Required in Ontario (PHIPA s.44) and Quebec (Law 25 PIA) before any real health data collection
3. **FSA population baseline** - Statistics Canada PCCF+ data for accurate per-FSA population denominators
4. **Calibration dataset** - At minimum one full flu season of paired wearable + FluWatch data to tune detection thresholds
5. **PHAC partnership interest** - FluWatch+ team at PHAC has a history of piloting novel surveillance inputs (e.g., wastewater, pharmacy data)

---

## Repo Structure

```
/
├── README.md
├── community_health_pulse_canada.html   # Main dashboard (self-contained)
├── cdc_validation.html                  # US version with real CDC ILINet data
├── data/
│   ├── canada_data.json                 # Synthetic CA dataset (255 KB)
│   └── cdc_real_data.json               # Real CDC ILINet 2024-25 (HHS regions)
├── notebooks/
│   ├── generate_canada_data.py          # Synthetic data generation
│   ├── privacy_pipeline.py              # k-anonymity + DP implementation
│   └── correlation_analysis.py          # FluWatch vs wearable correlation
└── docs/
    └── privacy_framework.md             # Full Canadian privacy law breakdown
```

---

## Skills Demonstrated

`Python` `NumPy` `Statistical anomaly detection` `Differential privacy` `k-anonymity`
`Chart.js` `Vanilla JS` `Data visualization` `PHAC FluWatch API` `PIPEDA / PHIPA / Law 25`
`Public health informatics` `Synthetic data generation` `Privacy-preserving analytics`
`Product thinking` `Canadian health regulatory landscape`

---

*Built as a prototype to explore whether consumer wearable data can serve as an early-warning layer for public health surveillance in Canada. Not affiliated with PHAC, WHOOP, or any provincial health authority.*
