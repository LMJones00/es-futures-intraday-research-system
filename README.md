# 📈 ES Futures Intraday Research System

> A multi-year quantitative research project spanning custom data collection via the IBKR API, systematic intraday feature engineering, and backtesting across 2,400+ observations — built to identify high-probability trading signals on the S&P 500 E-mini futures (ES) contract.

---

## 📌 Project Overview

This project explores a core hypothesis: **specific intraday price levels in the first minutes of market open can predict end-of-day direction on the ES futures contract.**

The system was built bottom-up — starting with manual observation, then automating data collection, cleaning, feature engineering, and backtesting — culminating in a systematic signal framework now being integrated with an AI-powered news intelligence pipeline.

**Instrument:** ES (S&P 500 E-mini Futures) — CME  
**Data resolution:** 1-minute bars + sub-second bars (1s resolution at open)  
**Dataset size:** 2,400+ intraday observations across multiple trading sessions  
**Outcome variable:** Day direction at 4:00pm and 4:15pm close vs. 9:30am open

---

## 🗂️ Repository Structure

```
├── gettingData.ipynb        # IBKR API connection, data ingestion, timezone handling
├── barsToCleaning.ipynb     # Raw bar data validation and preparation
├── cleaningToClean.ipynb    # Deduplication, filtering, outlier handling
├── preparing.ipynb          # Feature engineering, indicator construction
├── data/
│   └── NADEX_DATASET.xlsx   # Multi-sheet analytical framework (sanitised)
└── README.md
```

---

## 🔧 Pipeline: Four-Stage Data System

### Stage 1 — `gettingData.ipynb`: IBKR API Ingestion

Connects to Interactive Brokers TWS via `ib_insync`, pulls historical bar data for the ES futures contract, and handles all timezone, deduplication, and incremental save logic.

**Key features:**
- Pulls **1-minute bars** across full trading sessions (configurable duration)
- Pulls **sub-second bars** (1-second resolution) for the critical 9:30–10:00am open window
- Handles IBKR pacing violations with built-in `sleep()` delays
- Filters weekends and converts to US/Eastern timezone automatically
- Saves incrementally — new data appended to existing CSV without overwriting
- Targets 50+ specific intraday timestamps identified as high-signal moments

**Technologies:** `ib_insync`, `pandas`, `pytz`, `nest_asyncio`

---

### Stage 2 — `barsToCleaning.ipynb`: Raw Validation

Validates raw bar data from the IBKR output before it enters the analytical pipeline.

- Checks for missing timestamps and data gaps
- Validates OHLCV column integrity
- Flags anomalous volume or price spikes
- Prepares data for the cleaning stage

---

### Stage 3 — `cleaningToClean.ipynb`: Data Cleaning

Applies systematic cleaning rules to produce a reliable analytical dataset.

- Removes duplicate entries and near-duplicate rows
- Handles missing values using forward-fill where appropriate
- Filters to relevant trading hours only
- Produces clean, analysis-ready CSV output

---

### Stage 4 — `preparing.ipynb`: Feature Engineering

Constructs the analytical features used in backtesting and modelling.

**Features engineered include:**
- `Indicator Price` — price at each specific intraday timestamp
- `Initial Direction` — whether the indicator price is above or below the 9:30am open
- `Day Direction` — whether the market closed above or below open (at 4pm and 4:15pm)
- `Philosophy Result` — does the initial direction correctly predict day direction?
- `Win/Loss` — binary outcome per observation
- Price distance metrics: open-to-close diff, open-to-mark diff (rounded, standard deviation intervals)
- Win/loss streaks per indicator

---

## 📊 Analytical Framework (NADEX_DATASET.xlsx)

The dataset is structured across multiple sheets, each serving a distinct analytical purpose:

| Sheet | Purpose |
|-------|---------|
| `Daily Data` | Core observation table — 2,400+ rows of intraday price snapshots with outcomes |
| `GPT` | AI-assisted analysis layer |
| `Metrics` | Computed price distance statistics (open/close/mark diffs, standard deviations) |
| `415pm Results` | Win rate breakdown per indicator, per day of week, at 4:15pm close |
| `415pm Philo Results` | Philosophy-based directional prediction results |
| `Opp Strat Results` | Counter-strategy testing (opposite direction trades) |
| `Top Indicators` | Ranked list of highest win-rate indicators overall and by day of week |
| `Weekly Data` | Aggregated weekly performance tracking |

### Core Research Question

> *"Does the price at a specific intraday timestamp, relative to the 9:30am open, reliably predict whether the market will close higher or lower at 4pm/4:15pm?"*

Each row in `Daily Data` represents one observation: a specific timestamp's price on a specific trading day, with the day's final outcome recorded. The `Top Indicators` sheet surfaces which timestamps have the highest consistent win rates — the foundation of the trading strategy.

---

## 📐 Key Metrics Tracked

| Metric | Description |
|--------|-------------|
| `Win Rate %` | % of times initial direction correctly predicted day close |
| `Win/Loss by Day` | Breakdown by Monday–Friday to identify day-of-week effects |
| `Streak` | Consecutive win/loss runs per indicator |
| `Open-Close Diff` | Absolute point distance between open and close |
| `Open-Mark Diff` | Distance between open and the mark price |
| `StdDev Intervals` | Price movement in standard deviation bands |

---

## 🔗 Integration Roadmap

This project feeds directly into the [AI Market Intelligence Pipeline](https://github.com/LMJones00/ai-market-intelligence-pipeline-n8n-workflow):

```
Phase 1 (Complete) ──► Manual observation + Excel backtesting framework
Phase 2 (Complete) ──► Automated IBKR data collection pipeline
Phase 3 (Active)   ──► AI news signal integration (n8n pipeline)
Phase 4 (Planned)  ──► Combined model: price signals + news sentiment → direction prediction
Phase 5 (Planned)  ──► Real-time dashboard + automated trade alerts
```

The goal: combine the validated intraday price signals from this dataset with macro news sentiment scores from the n8n pipeline to build a multi-factor direction prediction model.

---

## 🧠 Design Decisions

**Why sub-second resolution at open?**  
The first 30 seconds of market open show the highest volatility and the strongest directional signals. 1-minute bars obscure this — 1-second bars capture it.

**Why these specific timestamps?**  
The 50+ timestamps were identified through manual observation across hundreds of trading sessions before being systematised. Each represents a price level where directional commitment tends to be confirmed or reversed.

**Why NADEX / binary-style analysis?**  
The binary (above/below open) framing maps directly to binary options structure — a simplified but powerful way to test directional edge before applying to continuous instruments.

---

## 🔧 Tech Stack

| Tool | Role |
|------|------|
| **Python** | Data collection, cleaning, feature engineering |
| **ib_insync** | IBKR TWS API wrapper |
| **pandas** | Data manipulation and analysis |
| **pytz / datetime** | Timezone handling |
| **Excel (multi-sheet)** | Backtesting framework and win-rate analysis |
| **Scikit-Learn / XGBoost** | Predictive modelling (in development) |

---

## ⚠️ Notes

- IBKR TWS must be running locally for data collection scripts to execute
- `lastTradeDateOrContractMonth` must be updated per contract expiry cycle
- Dataset provided is sanitised — some date ranges omitted for brevity
- This is a research project, not financial advice

---

## 👤 Author

**Luke M. Jones**  
MBA Candidate — AI Strategy & Project Management, Anglia Ruskin University, Cambridge UK  
[AI Market Intelligence Pipeline →](https://github.com/LMJones00/ai-market-intelligence-pipeline-n8n-workflow)  
[LinkedIn](https://linkedin.com/in/lukemjones1) | LMJSplash@gmail.com

---

*Active research project. See integration roadmap above for next phases.*
