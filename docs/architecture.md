# Architecture — IS → CF → BS Design

## Core Principle

IS, CF, BS are not three separate spreadsheets.  
They are three layers of a single calculation pipeline.

```
IS  =  현실 제약    (얼마 남는가)
CF  =  의사결정    (남은 돈을 어디에 쓰는가)
BS  =  결과 상태   (자산이 어떻게 변했는가)
```

## Layer Definitions

### Layer 1: Income Statement (IS)
**What it answers:** What is the hard constraint on cash flow?

```
surplus = monthly_income − monthly_expense
```

- Goal allocations are NOT in IS
- One-time costs are NOT in IS
- Surplus is the ceiling for all downstream allocations

### Layer 2: Cash Flow (CF) — Allocation Waterfall
**What it answers:** How is surplus distributed?

Priority order (fixed):
1. Mandatory 401k (payroll-deducted, outside surplus)
2. Discretionary retirement investment (from surplus)
3. Goal funding (from surplus remainder)
4. Residual → cash buffer

Rules:
- `surplus < 0` → all discretionary allocations = 0
- `surplus = 0` → goal funding = 0, only mandatory 401k continues
- Goals compete for remaining surplus after retirement allocation

### Layer 3: Balance Sheet (BS)
**What it answers:** What is the resulting asset state?

```
cash_end       = cash_begin + residual_cf
brokerage_end  = brokerage_begin × (1+r) + discretionary_invest
retirement_end = retirement_begin × (1+r) + mandatory_401k + discretionary_retire
```

Key rules:
- Cash is a survival buffer — no compounding
- `starting_invested_assets = brokerage + retirement` (cash excluded)
- BS changes only through CF — no direct manipulation

## Scenario Architecture

All scenarios share the same opening balance.  
Each scenario is a parallel branch, not a sequential override.

```
Opening Balance (shared)
        │
        ├── SCN000: Baseline
        ├── SCN001: Career Break  →  flow_shock: income = 0 for 1yr
        ├── SCN002: Grad School   →  hybrid: income↓ + one_time_cost
        └── SCN003: Family Plan   →  flow_shock: expense↑ for 3yr
```

## Portfolio Growth Formula

```
FV = PV × (1 + r) + C × ((1 + r) − 1) / r

PV = assets at start of year
r  = expected annual return (default 0.07)
C  = annual contribution
```

Applied annually per account, cash excluded.

## Multi-User DB Mapping

Every fact table in the ledger-first schema carries:

```sql
household_id   -- tenant scope
profile_id     -- versioned planning snapshot
scenario_id    -- parallel branch identifier
period_index   -- projection year (0 = opening)
```

Adding user authentication maps directly to `household_id`.  
No schema restructuring required for multi-tenant deployment.
