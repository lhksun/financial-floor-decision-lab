# Financial Floor Decision Lab

Most finance tools answer: How much do you have?
This answers: When do I become financially safe — and how do life decisions change that timeline?

The Core Idea
Financial safety isn't a number. It's an age.
Financial Floor Age — the first year where projected invested assets reach the Safe Layer Target,
the point where passive income covers target spending without depleting principal.
Safe Layer Target    = (Target Annual Spend − Social Security) / Withdrawal Rate
Financial Floor Age  = first year where Invested Assets ≥ Safe Layer Target
Once the floor is defined, every life decision becomes a question:
does this move the date, and by how much?

Architecture — IS → BS → CF
All calculations follow accounting-first logic. No formulas without a financial statement home.
Income Statement (IS)
What is the surplus this period?
Income − Expenses = Surplus (Net Income)
        ↓
Balance Sheet (BS)                         ← IS dependency
Surplus flows into retained earnings.
Invested assets compound forward.
Cash remains a survival buffer only.
        ↓
Cash Flow Statement (CF)                   ← BS dependency
How did cash actually move?
Operating · Investing · Financing activities
        ↓
Metrics
Floor Age · Years to Stability · Safe Layer Progress · Survival Months
This structure enforces internal consistency — every metric traces back to a posted number,
not a floating assumption. It also maps directly to a multi-user database schema.
Every row in the ledger carries household_id, making it ready for Postgres/Supabase migration.

Two Files, Two Purposes
financial_floor_v5.xlsx — Working prototype
Single-user, formula-driven. No Python, no macros. Error count: 0. Formula count: 2,186.

Financial Floor Age confirmed at 51 (vs. expected 51 ✓)
Scenario comparison across 4 scenarios × 40-year projection
CF waterfall: surplus → retirement → goals → residual cash = $0
All views derived from a single projection engine

Sample output — age 38, $4,800/mo net, $147,952 invested:
MetricValueSafe Layer Target$1,050,000Safe Layer Progress14.1%Financial Floor Age51Years to Stability13Survival Months6.4
Scenario comparison:
ScenarioFloor AgeDeltaBaseline51—Career break 1yr51+0Grad school 2yr52+1Family planning51+0
Most decisions don't move the floor. A few do. That's the insight.

financial_floor_ledger_v2.xlsx — Multi-user schema design
Ledger-first architecture modeled after ERP posting logic.
Event driver → Posting rules → Ledger postings
                                      ↓
                    IS / CF / BS as derived views, not source tables

Chart of accounts with DR/CR normal signs
household_id on every fact row — multi-tenant ready
IS, CF, BS derived from ledger queries, not hardcoded calculations

Known limitation: Sample data covers periods 0–1 only.
The posting generator (recurring event expansion + compounding) is the V3 build target.

Why Two Layers?
The v5 Excel prototype answers the personal finance question now.
The ledger schema answers the product architecture question for later.
Current (Excel):   1 user, formula-driven, no persistence
V3 target (DB):    N users, posting-engine-driven, append-only ledger
                   household_id + profile_id + scenario_id → all facts
The schema is already normalized for that migration.
Adding user_id to the master tables is the only structural change required.

Scenario Types
TypeIS ImpactBS ImpactExamplebaselineNoneNoneNormal projectionflow_shockIncome / expense deltaIndirect via CFCareer break, family planningstock_shockNoneDirect cash reductionTravel fund, medical costhybridFlow + stockBothGrad school (income drop + tuition)
Key design decisions:

Cash is a survival buffer only — excluded from compounding calculations
401k modeled as payroll-deducted, paused during career_break / grad_school scenarios
Scenarios are parallel branches from the same opening balance, not sequential overwrites


Metric Definitions
MetricFormulasurplusmonthly_income − monthly_expensestarting_invested_assetsbrokerage + retirement_accounts (cash excluded)safe_layer_target(target_monthly_spend × 12 − social_security × 12) / withdrawal_ratesafe_layer_progressstarting_invested_assets / safe_layer_targetsurvival_monthscash / monthly_expensefloor_agecurrent_age + first projection year where assets ≥ safe_layer_targetyears_to_stabilityfloor_age − current_age

Dashboard
Page 1 — Where Am I Today
이미지 표시
Page 2 — What If
이미지 표시
Looker Studio report (sample data):
Financial Floor — Executive View

Repository Structure
financial-floor-decision-lab/
├── README.md
├── data/
│   ├── financial_floor_v5.xlsx          ← working single-user prototype
│   └── financial_floor_ledger_v2.xlsx   ← multi-user ledger-first schema design
├── docs/
│   ├── architecture.md                  ← IS → BS → CF design principles
│   ├── metric_definitions.md            ← KPI formulas and definitions
│   └── data_model.md                    ← schema and table relationships
└── images/
    ├── page1_where_am_i.png
    └── page2_what_if.png

Roadmap
V1 (done): Single-user Excel prototype with working Floor Age calculation
V2 (done): Ledger-first multi-user schema design
V3 (next): Python posting generator — recurring event expansion + compounding
V4: Postgres migration, API layer, Looker Studio live connection at scale

Stack
Excel Power BI Looker Studio GitHub

Related

freight-canonical-model — Canonical freight data architecture
ops-finance-decision-layer — Sales segmentation and commercial analytics
gap-trading-engine — Rule-based intraday trading system
---

*Built as a portfolio project demonstrating constraint-based financial modeling, multi-layer data architecture, and BI tool integration (Power BI + Looker Studio).*
