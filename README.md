# Financial Floor Decision Lab

Most finance tools answer: *How much do you have?*

This answers: *When do I become financially safe — and how do life decisions change that timeline?*

---

## The Core Idea

Financial safety isn't a number. It's an age.

**Financial Floor Age** — the first year where projected invested assets reach the Safe Layer Target,
the point where passive income covers target spending without depleting principal.

```
Safe Layer Target    = (Target Annual Spend − Social Security) / Withdrawal Rate
Financial Floor Age  = first year where Invested Assets ≥ Safe Layer Target
```

Once the floor is defined, every life decision becomes a question:
does this move the date, and by how much?

---

## Architecture — IS → BS → CF

All calculations follow accounting-first logic. No formulas without a financial statement home.

```
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
```

This structure enforces internal consistency — every metric traces back to a posted number,
not a floating assumption. It also maps directly to a multi-user database schema.
Every row in the ledger carries `household_id`, making it ready for Postgres/Supabase migration.

---

## Two Files, Two Purposes

### `financial_floor_v5.xlsx` — Working prototype

Single-user, formula-driven. No Python, no macros. Error count: 0. Formula count: 2,186.

- Financial Floor Age confirmed at 51 (vs. expected 51 ✓)
- Scenario comparison across 4 scenarios × 40-year projection
- CF waterfall: surplus → retirement → goals → residual cash = $0
- All views derived from a single projection engine

**Sample output — age 38, $4,800/mo net, $147,952 invested:**

| Metric | Value |
|---|---|
| Safe Layer Target | $1,050,000 |
| Safe Layer Progress | 14.1% |
| Financial Floor Age | 51 |
| Years to Stability | 13 |
| Survival Months | 6.4 |

**Scenario comparison:**

| Scenario | Floor Age | Delta |
|---|---|---|
| Baseline | 51 | — |
| Career break 1yr | 51 | +0 |
| Grad school 2yr | 52 | +1 |
| Family planning | 51 | +0 |

Most decisions don't move the floor. A few do. That's the insight.

---

### `financial_floor_ledger_v2.xlsx` — Multi-user schema design

Ledger-first architecture modeled after ERP posting logic.

```
Event driver → Posting rules → Ledger postings
                                      ↓
                    IS / CF / BS as derived views, not source tables
```

- Chart of accounts with DR/CR normal signs
- `household_id` on every fact row — multi-tenant ready
- IS, CF, BS derived from ledger queries, not hardcoded calculations

**Known limitation:** Sample data covers periods 0–1 only.
The posting generator (recurring event expansion + compounding) is the V3 build target.

---

## Why Two Layers?

The v5 Excel prototype answers the personal finance question now.
The ledger schema answers the product architecture question for later.

```
Current (Excel):   1 user, formula-driven, no persistence
V3 target (DB):    N users, posting-engine-driven, append-only ledger
                   household_id + profile_id + scenario_id → all facts
```

The schema is already normalized for that migration.
Adding `user_id` to the master tables is the only structural change required.

---

## Scenario Types

| Type | IS Impact | BS Impact | Example |
|---|---|---|---|
| `baseline` | None | None | Normal projection |
| `flow_shock` | Income / expense delta | Indirect via CF | Career break, family planning |
| `stock_shock` | None | Direct cash reduction | Travel fund, medical cost |
| `hybrid` | Flow + stock | Both | Grad school (income drop + tuition) |

**Key design decisions:**
- Cash is a survival buffer only — excluded from compounding calculations
- 401k modeled as payroll-deducted, paused during `career_break` / `grad_school` scenarios
- Scenarios are parallel branches from the same opening balance, not sequential overwrites

---

## Metric Definitions

| Metric | Formula |
|---|---|
| `surplus` | monthly_income − monthly_expense |
| `starting_invested_assets` | brokerage + retirement_accounts *(cash excluded)* |
| `safe_layer_target` | (target_monthly_spend × 12 − social_security × 12) / withdrawal_rate |
| `safe_layer_progress` | starting_invested_assets / safe_layer_target |
| `survival_months` | cash / monthly_expense |
| `floor_age` | current_age + first projection year where assets ≥ safe_layer_target |
| `years_to_stability` | floor_age − current_age |

---

## Dashboard

### Page 1 — Where Am I Today
![Where Am I](images/page1_where_am_i.png)

### Page 2 — What If
![What If](images/page2_what_if.png)

Looker Studio report (sample data):
[Financial Floor — Executive View](https://lookerstudio.google.com/reporting/c8a3698c-821c-4670-a1e2-48ac1da1e6ba)

---

## Repository Structure

```
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
```

---

## Roadmap

**V1 (done):** Single-user Excel prototype with working Floor Age calculation
**V2 (done):** Ledger-first multi-user schema design
**V3 (next):** Python posting generator — recurring event expansion + compounding
**V4:** Postgres migration, API layer, Looker Studio live connection at scale

---

## Stack

`Excel` `Power BI` `Looker Studio` `GitHub`

---

## Related

- [`freight-canonical-model`](https://github.com/lhksun/freight-canonical-model) — Canonical freight data architecture
- [`ops-finance-decision-layer`](https://github.com/lhksun/ops-finance-decision-layer) — Sales segmentation and commercial analytics
- [`gap-trading-engine`](https://github.com/lhksun/gap-trading-engine) — Rule-based intraday trading system
