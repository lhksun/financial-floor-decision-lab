# Data Model

## v5 Prototype Schema (Excel)

Single-user. All calculations in Excel formulas.

### Sheet Map

| Sheet | Role | Looker Reads? |
|---|---|---|
| `input` | All user inputs. Only sheet users edit. | No |
| `calc_is` | IS layer: income, expense, surplus | Yes |
| `calc_cf` | CF layer: allocation waterfall | Yes |
| `calc_bs` | BS layer: asset state | Yes |
| `calc_metrics` | KPI outputs: floor_age, progress | Yes |
| `projection` | 4 scenarios × 40 years = 160 rows | Yes |
| `scenarios_def` | Scenario reference table | No |

### Projection Grain
One row = one scenario × one projection year

```
projection_key = (scenario_id, projection_year_index)
```

### Scenario Definitions

| ID | Name | Type | pause_401k |
|---|---|---|---|
| SCN000 | Baseline | baseline | FALSE |
| SCN001 | Career Break 1yr | flow_shock | TRUE |
| SCN002 | Grad School 2yr | hybrid | TRUE |
| SCN003 | Family Planning | flow_shock | FALSE |

---

## v2 Ledger Schema (Multi-User Design)

Ledger-first architecture. IS, CF, BS are derived views.

### Entity Hierarchy

```
households
  └── household_profiles (versioned snapshots)
        └── scenario_master (parallel branches)
              └── ledger_postings (fact table)

account_master      ← chart of accounts
event_driver        ← business event definitions
posting_rules       ← event class → debit/credit mapping
scenario_adjustments ← delta layer for scenario branches
policy_master       ← allocation policy per household
goal_master         ← goal definitions
```

### Canonical Tables (DB-ready)

```sql
households          (household_id PK, ...)
household_profiles  (profile_id PK, household_id FK, ...)
account_master      (account_code PK, account_class, normal_sign, ...)
scenario_master     (scenario_id PK, household_id FK, parent_scenario_id, ...)
event_driver        (event_id PK, household_id FK, scenario_id FK, recurring_flag, ...)
posting_rules       (posting_rule_id PK, event_class, debit_account, credit_account, ...)
scenario_adjustments (adjustment_id PK, scenario_id FK, target_event_id FK, ...)
policy_master       (policy_id PK, household_id FK, policy_key, policy_value_num, ...)
goal_master         (goal_id PK, household_id FK, ...)
ledger_postings     (posting_id PK, household_id FK, scenario_id FK,
                     period_index, account_code FK, dr_cr, amount, signed_amount, ...)
```

### Ledger Posting Structure

Each business event generates two ledger rows (double-entry):

```
event: Employment Income, period 1, $57,600
→ P0007: DR 1000 (Cash)     $57,600   cash_flow_group=Operating
→ P0008: CR 4000 (Income)   $57,600   cash_flow_group=Operating

event: Retirement Allocation, period 1, $9,600
→ P0011: DR 1200 (Retirement) $9,600  cash_flow_group=Investing
→ P0012: CR 1000 (Cash)       $9,600  cash_flow_group=Investing
```

### Chart of Accounts

| Code | Name | Class | Statement | Normal Sign |
|---|---|---|---|---|
| 1000 | Cash | asset | BS | DR |
| 1100 | Brokerage | asset | BS | DR |
| 1200 | Retirement Assets | asset | BS | DR |
| 1300 | Goal Fund | asset | BS | DR |
| 2000 | Liabilities | liability | BS | CR |
| 3000 | Opening Equity | equity | BS | CR |
| 4000 | Employment Income | revenue | IS | CR |
| 5000 | Living Expense | expense | IS | DR |

### View Derivation

```sql
-- IS view
SELECT scenario_id, period_index,
  SUM(CASE WHEN account_class='revenue' THEN signed_amount ELSE 0 END) AS income,
  SUM(CASE WHEN account_class='expense' THEN signed_amount ELSE 0 END) AS expense
FROM ledger_postings
GROUP BY scenario_id, period_index

-- BS view (cumulative)
SELECT scenario_id, period_index,
  SUM(CASE WHEN account_code='1000' AND period_index <= ? THEN signed_amount END) AS cash_end,
  SUM(CASE WHEN account_code='1100' AND period_index <= ? THEN signed_amount END) AS brokerage_end
FROM ledger_postings
GROUP BY scenario_id, period_index

-- CF view (cash account movements by group)
SELECT scenario_id, period_index, cash_flow_group,
  SUM(signed_amount) AS cf_amount
FROM ledger_postings
WHERE account_code = '1000'
GROUP BY scenario_id, period_index, cash_flow_group
```

### Multi-User Extension

To support multiple users, add `household_id` as a filter to all views:

```sql
WHERE household_id = :current_user_household_id
```

No schema changes required. The FK chain already exists:
`users → households → household_profiles → ledger_postings`

---

## V3 Roadmap: Posting Generator

The gap between v2 schema and a live engine is one component: a posting generator that expands `event_driver` × `scenario_adjustments` × `posting_rules` into `ledger_postings` automatically.

Planned as a Python script:

```
Input:  event_driver, scenario_adjustments, posting_rules, household_profiles
Output: ledger_postings (all periods, all scenarios, recurring expansion + compounding)
```

When complete, all views and KPIs become fully computed from source data.
