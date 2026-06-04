# Eliminating Child Care Deserts in New York State — Optimization Models

**IEOR E4004 — Optimization Models and Methods | Columbia Engineering | Fall 2025**  
Team: Oscar Lozachmeur, Chad Readey, Hanrui Zhang

---

## Overview

Over half of American children live in child care deserts — regions where licensed child care slots fall far short of demand. This project applies mixed-integer linear programming to help the New York State government determine the minimum funding needed to eliminate these deserts across all 1,646 ZIP codes in the state.

Three progressively realistic scenarios are modeled and solved:

| Scenario | Description | Total Cost | Status |
|---|---|---|---|
| Part 1 — Idealistic | Unlimited new builds, no spatial constraints | $430.1M | Optimal |
| Part 2 — Realistic | Distance rules + 20% expansion cap + piecewise costs | $444.9M | Optimal |
| Part 3 — Fairness | $100M budget + equity constraint across regions | — | Infeasible |

---

## Problem Definition

A ZIP code is classified as a child care desert if:
- **High-demand area** (employment >= 60% OR avg income <= $60K): available slots <= 50% of children aged 0-12
- **Normal-demand area**: available slots <= 33% of children aged 0-12

NYS policy additionally requires that slots for children under 5 cover at least two-thirds of that age group's population.

New facility options available to the state:

| Size | Total Slots | Max Slots (0-5) | Build Cost |
|---|---|---|---|
| Small | 100 | 50 | $65,000 |
| Medium | 200 | 100 | $95,000 |
| Large | 400 | 200 | $115,000 |

---

## Results

### Part 1 — Idealistic Scenario

Assumes facilities can be built anywhere with no spatial or logistical constraints.

- Total cost: **$430,139,450**
- New facilities built: 3,452 (Small: 658 / Medium: 277 / Large: 2,517)
- Expansions: 0
- ZIP codes covered: 1,646 / 1,646

New construction dominates because expansion triggers a fixed baseline cost. Building new large facilities is more cost-effective at scale.

### Part 2 — Realistic Scenario

Adds a minimum distance of 0.06 miles between facilities, a 20% cap on expansions, and piecewise convex expansion costs.

- Total cost: **$444,945,613** (+3.5% vs Part 1)
- New facilities built: 3,513 (Small: 589 / Medium: 267 / Large: 2,657)
- Expansions: 2,343
- ZIP codes covered: 1,646 / 1,646

Expansion segment breakdown: 31,563 slots at 0-10%, 8,704 slots at 10-15%, 1,373 slots at 15-20%. The 3.5% cost increase reflects real-world spatial and logistical constraints. Large facilities dominate at approximately $287 per child — the best cost-per-slot ratio.

### Part 3 — Fairness Scenario

Goal: maximize the Social Coverage Index (SCI) under a $100M budget with a fairness constraint capping the coverage gap between any two ZIP codes at 0.1.

```
SCI = (2/3) x coverage(0-5) + (1/3) x coverage(5-12)
```

Result: infeasible. Under a fairness band of 0.1 and a $100M budget, the model cannot simultaneously satisfy budget, coverage floors, and fairness. Relaxing either the budget or the fairness tolerance restores feasibility. This result is documented and interpreted in the full report.

---

## Repository Structure

```
childcare-deserts-nys-optimization/
│
├── data/
│   ├── population.csv
│   ├── avg_individual_income.csv
│   ├── employment_rate.csv
│   ├── child_care_regulated.csv
│   └── potential_locations.csv
│
├── Optimization_Final.ipynb     — Parts 1 and 2: full model implementation
├── Fairness_Problem.ipynb       — Part 3: fairness and budget constraint
├── optimization_project.pdf     — Full written report
├── .gitignore
└── README.md
```

---

## Methodology

### Part 1 — Idealistic Model

Objective: minimize total construction and expansion cost.

Key constraints: total coverage meets the desert-elimination threshold for every ZIP code; slots for children aged 0-5 cover at least 2/3 of that population; expansion is capped at min(500, 120% of current capacity); a surcharge of $100 per slot applies for children aged 0-5.

### Part 2 — Realistic Model

Same objective, with three additional constraints: a minimum distance of 0.06 miles between any two facilities; expansion limited to 20% of current capacity; piecewise convex expansion costs linearized with three auxiliary variables per facility.

| Expansion Range | Cost Structure |
|---|---|
| 0 to 10% | (20,000 + 200 x nf) x x/nf |
| 10 to 15% | (20,000 + 400 x nf) x x/nf |
| 15 to 20% | (20,000 + 1,000 x nf) x x/nf |

### Part 3 — Fairness Model

Objective: maximize SCI subject to a $100M budget, a fairness band of 0.1 across all ZIP codes, and all desert-elimination constraints. The model is infeasible — the budget and fairness requirements cannot be jointly satisfied at this scale.

---

## Tech Stack

- Python 3.12
- Gurobi 12 (gurobipy) — Mixed-Integer Linear Programming
- pandas, numpy — Data preprocessing
- scikit-learn (BallTree) — Haversine distance computation
- Jupyter Notebooks

---

## Gurobi License

This project requires a Gurobi academic license. The free trial is limited to 2,000 variables and cannot solve models at the scale of this project (1,646 ZIP codes with multiple decision variables per ZIP). Academic licenses are free for students at gurobi.com/academia. The solver was run using the Columbia University WLS academic license, which requires access to the university network.

---

## Report

Full methodology, mathematical formulations, results, and discussion are available in `optimization_project.pdf`.

---

## Limitations and Next Steps

- Cost data are uniform across the state — real costs vary by county and terrain
- Facility sizes are fixed at three options — no variable-size optimization
- Population estimates assume 50% of the 10-14 age group fall within ages 10-12
- Ongoing operational costs are excluded from the model
- Next steps: sensitivity analysis on the fairness band and budget; multi-year optimization incorporating operating costs

