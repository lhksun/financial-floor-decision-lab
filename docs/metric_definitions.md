# Metric Definitions

## Primary KPIs

### Financial Floor Age
> The age at which projected invested assets first reach the Safe Layer Target.

```
floor_age = current_age + first projection year where invested_assets ≥ safe_layer_target
```

This is the central output of the model. Everything else is context for this number.

### Years to Stability
```
years_to_stability = floor_age − current_age
```

### Safe Layer Target
```
safe_layer_target = (target_annual_spend − social_security_annual) / withdrawal_rate

Default: withdrawal_rate = 0.04 (4% rule)
```

The amount of invested assets needed to sustain target spending indefinitely without principal depletion.

### Safe Layer Progress
```
safe_layer_progress = liquid_invested_assets / safe_layer_target
```

Current position as a percentage of the floor target.

---

## Secondary KPIs

### Survival Months
```
survival_months = cash / monthly_expense
```

How long cash alone covers expenses. Minimum threshold: 6 months.  
Note: cash is excluded from compounding and does not count toward safe layer progress.

### Investment Rate
```
investment_rate = (discretionary_invest + mandatory_401k) × 12 / annual_gross_income
```

### Monthly Free Cash Flow (Surplus)
```
surplus = monthly_net_income − monthly_expense
```

The hard constraint. No allocation can exceed this.

---

## Asset Definitions

| Asset | Included in Safe Layer | Compounded | Notes |
|---|---|---|---|
| Cash | No | No | Survival buffer only |
| Brokerage | Yes | Yes | Taxable invested assets |
| Retirement (401k + IRA) | Yes | Yes | Tax-advantaged |
| Goal Funds | No | No | Earmarked, excluded from floor calc |

```
starting_invested_assets = brokerage + retirement_accounts
```

Cash is deliberately excluded from the safe layer calculation. Including it would inflate progress and obscure the distinction between liquid buffer and long-term capital.

---

## Scenario Impact Metrics

### Delta vs Baseline
```
delta_years = scenario_floor_age − baseline_floor_age
```

Positive = scenario delays financial floor.  
Zero = no meaningful impact on timeline.  
Negative = scenario accelerates floor (e.g., income increase scenario).

---

## Feasibility Thresholds (V1)

A scenario is flagged as **feasible** if all conditions hold throughout the projection:

| Condition | Threshold |
|---|---|
| Cash balance | ≥ 0 at all periods |
| Survival months | ≥ 6 at all periods |
| Invested assets | ≥ safe_layer_target × 0.25 |
| Floor delay | ≤ 3 years vs baseline |

Failing any single condition marks the scenario as not feasible at that start offset.
