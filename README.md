# Eliminating Child Care Deserts in New York State — Optimization Models

> **IEOR E4004 — Optimization Models and Methods | Columbia Engineering | Fall 2025**  
> Team: Oscar Lozac'hmeur, Chad Readey, Hanrui Zhang

---

## Overview

Over half of American children live in **child care deserts** — regions where licensed child care slots fall far short of demand. This project uses **mixed-integer optimization** (via Gurobi) to help the New York State government determine the minimum funding needed to eliminate these deserts across all 1,646 ZIP codes in the state.

Three progressively realistic scenarios are modeled:

| Scenario | Description | Result |
|---|---|---|
| **Part 1 — Idealistic** | No spatial constraints; unlimited new builds | $430.1M feasible |
| **Part 2 — Realistic** | Distance rules, 20% expansion cap, piecewise costs | $444.9M feasible |
| **Part 3 — Fairness** | $100M budget + equity constraint across regions | Infeasible (as expected) |

---

## Problem Definition

A ZIP code is classified as a **child care desert** if:
- **High-demand area** (employment ≥ 60% OR avg income ≤ $60K): available slots ≤ 50% of children aged 0–12
- **Normal-demand area**: available slots ≤ 33% of children aged 0–12

Additionally, NYS policy mandates that slots for children **under 5** must cover at least **2/3 of that population**.

### Facility Options

| Size | Total Slots | Max Slots (0–5) | Build Cost |
|---|---|---|---|
| Small | 100 | 50 | $65,000 |
| Medium | 200 | 100 | $95,000 |
| Large | 400 | 200 | $115,000 |

---

## Repo Structure

```
childcare-deserts-nys/
│
├── data/
│   ├── population.csv
│   ├── avg_individual_income.csv
│   ├── employment_rate.csv
│   ├── child_care_regulated.csv
│   └── potential_locations.csv
│
├── notebooks/
│   ├── Part1_Idealistic.ipynb        ← Minimum cost, no spatial constraints
│   ├── Part2_Realistic.ipynb         ← Piecewise costs + distance rules
│   ├── Part3_Fairness.ipynb          ← Fairness + budget constraint
│   └── Optimization_Final.ipynb      ← Consolidated final notebook
│
├── src/
│   ├── partie_1.py                   ← Refactored Part 1 model
│   └── fairness_problem.py           ← Fairness scenario model
│
├── report/
│   └── optimization_project.pdf      ← Full written report
│
├── requirements.txt
├── .gitignore
└── README.md
```

---

## Methodology

### Part 1 — Idealistic Model

**Objective:** Minimize total construction + expansion cost.

**Decision variables:**
- `x[z, s]` — number of new facilities of size `s` to build in ZIP `z`
- `x[f]` — slots added via expansion at facility `f`

**Key constraints:**
- Coverage ≥ threshold for all ZIP codes (desert elimination)
- 0–5 age policy: slots ≥ 2/3 × population(0–5)
- Expansion ≤ min(500, 120% × current capacity)

**Result:** $430.1M total — 3,452 new facilities (mostly Large), 0 expansions.  
New construction dominates because expansion triggers a fixed baseline cost.

---

### Part 2 — Realistic Model

Same objective, with added constraints:

- **Distance rule:** No two facilities (new or existing) within 0.06 miles of each other
- **Expansion cap:** Max 20% of current capacity
- **Piecewise convex expansion costs:**

| Expansion Range | Unit Cost Formula |
|---|---|
| 0–10% | `(20,000 + 200·n_f) · x/n_f` |
| 10–15% | `(20,000 + 400·n_f) · x/n_f` |
| 15–20% | `(20,000 + 1,000·n_f) · x/n_f` |

The piecewise function is linearized using 3 auxiliary variables per facility.

**Result:** $444.9M — 3,513 new facilities + 2,343 expansions. Cost increase of ~3.5% vs Part 1 reflects real-world spatial and logistical friction.

---

### Part 3 — Fairness Model

**Objective:** Maximize Social Coverage Index (SCI) under $100M budget.

```
SCI = (2/3) × coverage(0–5) + (1/3) × coverage(5–12)
```

**Fairness constraint:** The gap in child care availability between any two ZIP codes ≤ 0.1.

**Result:** Model is infeasible under these conditions. The $100M budget is insufficient to simultaneously satisfy desert elimination, the 0–5 policy floor, and the fairness band (δ = 0.10). Relaxing either the budget or the fairness tolerance restores feasibility.

---

## Key Results

```
Part 1 (Ideal)    → Total Cost: $430,139,450 | 3,452 new facilities | 0 expansions
Part 2 (Realistic)→ Total Cost: $444,945,613 | 3,513 new facilities | 2,343 expansions
Part 3 (Fairness) → Infeasible under $100M + δ=0.10 fairness constraint
```

**Coverage achieved:** All 1,646 ZIP codes meet desert-elimination and 0–5 policy requirements in Parts 1 and 2.

---

## Tech Stack

- **Python 3.12**
- **Gurobi 11** (`gurobipy`) — Mixed-Integer Linear Programming solver
- **pandas** — Data loading and preprocessing
- **numpy** — Numerical computations
- **Jupyter Notebooks** — Exploratory modeling


---

## Report

The full written report (methodology, formulations, results, limitations) is available in [`report/optimization_project.pdf`](report/optimization_project.pdf).

---

## Limitations & Next Steps

- Cost data are uniform — real costs vary by county and terrain
- Fixed facility sizes (S, M, L) — no variable-size optimization
- Population estimates assume 50% of the 10–14 age group are ages 10–12
- Excludes ongoing operational expenses
- Next steps: sensitivity analysis on δ and budget; multi-year optimization with operating costs

