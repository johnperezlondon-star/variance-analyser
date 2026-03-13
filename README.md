# AI Variance Analyser

> Automated budget-vs-actuals analysis with GPT-4o narrative generation — built for finance teams, designed for portfolios.

![Python](https://img.shields.io/badge/Python-3.10+-blue?style=flat-square&logo=python&logoColor=white)
![OpenAI](https://img.shields.io/badge/GPT--4o-OpenAI-412991?style=flat-square&logo=openai&logoColor=white)
![Streamlit](https://img.shields.io/badge/Streamlit-UI-FF4B4B?style=flat-square&logo=streamlit&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-Data-150458?style=flat-square&logo=pandas&logoColor=white)
![Status](https://img.shields.io/badge/Status-Portfolio%20Ready-brightgreen?style=flat-square)

---

## The business problem this solves

Every finance team spends **2–4 hours per reporting cycle** manually writing variance commentary. A controller reads the numbers, identifies the top movers, applies business context, and writes prose like:

> *"Revenue was £77k above budget driven by North region outperformance, partially offset by Marketing overspend of £13.7k related to the Q2 campaign launch."*

This tool **automates that workflow entirely**. Upload two CSV files (budget + actuals), optionally add a line of business context, and receive a full management commentary ready to drop into a board pack — in under 30 seconds.

---

## Live demo

**[View the interactive UI demo →](app_demo.html)**

The demo shows the full Streamlit interface with sample data pre-loaded and a simulated GPT-4o commentary generation.

---

## Sample output

Given this input data (summarised):

| Cost centre | Category    | Budget YTD | Actual YTD | Variance  | %      |
|-------------|-------------|-----------|-----------|----------|--------|
| Sales       | Revenue     | £980,000  | £1,057,000 | +£77,000 | +7.9%  |
| Sales       | Salaries    | £273,000  | £251,160   | +£21,840 | +8.0%  |
| Marketing   | Advertising | £57,500   | £71,200    | −£13,700 | −23.8% |
| HR          | Recruitment | £24,000   | £18,400    | +£5,600  | +23.3% |

The tool generates this commentary automatically:

> *The organisation delivered a favourable overall variance of £68,500 (+4.8%) against the half-year budget of £1.42m, reflecting stronger-than-anticipated revenue performance and the benefit of the Q2 headcount freeze across sales and support functions.*
>
> *The most significant driver of the positive outturn was Sales Revenue, which exceeded budget by £77,000 (+7.9%), consistent with the North region's outperformance noted during the period. This was further supported by a favourable salary variance of £21,840 (+8.0%) in the Sales cost centre, directly attributable to the headcount freeze which delayed two planned hires into H2...*
>
> *Looking ahead, the favourable revenue trend is expected to continue into Q3 provided the North region maintains current pipeline conversion rates...*

---

## How it works

```
budget.csv  ──┐
               ├──► Python engine (Pandas) ──► Variance table + flags
actuals.csv ──┘                                        │
                                                       ▼
context notes ─────────────────────────────► GPT-4o prompt
                                                       │
                                           ┌───────────┴────────────┐
                                           ▼                        ▼
                                   AI commentary text        Excel report (.xlsx)
```

**Three-stage pipeline:**

1. **Data layer** (`variance_engine.py`) — Pandas merges budget and actuals on `cost_centre` + `category`, calculates absolute and percentage variances, and flags material items (>5% or >£5,000 threshold, configurable).

2. **AI layer** (`ai_narrative.py`) — Passes only the flagged variances to GPT-4o with a structured prompt that specifies tone (formal finance), format (3-paragraph board pack prose), and scope. Temperature set to 0.3 for consistent, professional output.

3. **Output layer** — Produces a formatted Excel workbook (Summary + Detail tabs with conditional formatting) and the written commentary, surfaced via a Streamlit UI.

---

## Project structure

```
variance-analyser/
├── data/
│   ├── budget.csv          # Sample budget data (6 cost centres, 6 months)
│   └── actuals.csv         # Sample actuals data (matching structure)
├── outputs/                # Generated reports land here
├── variance_engine.py      # Data processing: merge, variance calc, flagging, Excel export
├── ai_narrative.py         # GPT-4o integration: prompt engineering + commentary generation
├── app.py                  # Streamlit UI: file upload, table display, commentary trigger
├── requirements.txt
├── .env.example            # API key template (never commit your actual .env)
└── README.md
```

---

## Quickstart

### 1. Clone and install

```bash
git clone https://github.com/YOUR_USERNAME/variance-analyser.git
cd variance-analyser
pip install -r requirements.txt
```

### 2. Add your OpenAI API key

```bash
cp .env.example .env
# Edit .env and add: OPENAI_API_KEY=sk-...
```

### 3. Run the app

```bash
streamlit run app.py
```

The app opens at `http://localhost:8501`. Upload the sample CSV files from `/data` to see it in action immediately.

---

## Data format

Both `budget.csv` and `actuals.csv` must share the same column structure:

```csv
cost_centre,category,january,february,march,april,may,june
Sales,Revenue,150000,160000,155000,170000,165000,180000
Sales,Salaries,45000,45000,45000,46000,46000,46000
Operations,Overhead,12000,12000,12500,12500,13000,13000
Marketing,Advertising,8000,10000,9000,11000,9500,10000
```

- Column names for months can be any consistent label (`jan`, `january`, `2024-01` etc.)
- Cost centre and category names must match exactly between the two files
- Numeric values should be whole numbers (no currency symbols)

---

## Key technical decisions

**Why only pass flagged items to GPT?**
Sending the full dataset to the model is wasteful and produces unfocused commentary. By pre-filtering to material variances (>5% or >£5k), the AI writes about what matters, costs ~80% less per API call, and produces tighter prose.

**Why temperature 0.3?**
Finance commentary needs to be consistent and professional. Low temperature prevents the model from being creative with numbers or introducing confident-sounding errors. It still writes fluently — it just doesn't improvise.

**Why Pandas melt (wide to long)?**
The input format (one column per month) is natural for humans to prepare in Excel, but variance calculations are much cleaner on long-format data. Melting on load means the rest of the codebase is simple and month-agnostic.

---

## Configurable thresholds

In `variance_engine.py`, the materiality thresholds are easily adjusted:

```python
# Default: flag if >5% variance OR >£5,000 absolute
MATERIALITY_PCT = 5.0      # percentage threshold
MATERIALITY_ABS = 5000     # absolute value threshold (in your currency)
```

---

## requirements.txt

```
pandas>=2.0
openpyxl>=3.1
openai>=1.0
streamlit>=1.28
python-dotenv>=1.0
```

---

## Roadmap / planned features

- [ ] Multi-period trend view (rolling 12 months)
- [ ] Departmental drill-down with month-level detail
- [ ] PDF board pack generator (WeasyPrint)
- [ ] Anomaly detection on historical actuals using statistical outlier flagging
- [ ] Slack / Teams notification for material variances on scheduled runs
- [ ] Support for P&L, balance sheet, and cash flow statement formats

---

## Skills demonstrated

| Area | What this project shows |
|------|------------------------|
| Finance domain | Variance analysis, management accounts, materiality thresholds, board pack commentary |
| Python / Pandas | CSV ingestion, wide-to-long transformation, aggregation, conditional flagging |
| AI / Prompt engineering | GPT-4o integration, structured prompt design, output consistency via temperature |
| Data output | Excel workbook generation with openpyxl, conditional formatting |
| UI | Streamlit app with file upload, dynamic table rendering, download button |
| Code quality | Modular structure, separation of concerns, `.env` for secrets, documented functions |

---

## About the author

Built as part of a portfolio demonstrating AI applied to finance automation. Background in financial accounting and management reporting — this tool addresses a real pain point I observed in month-end close processes.

*Connect on [LinkedIn](#) · See more projects on [GitHub](#)*

---

## Licence

MIT — free to use, adapt, and build on. If this helps you in your own projects, a star on the repo is appreciated.
