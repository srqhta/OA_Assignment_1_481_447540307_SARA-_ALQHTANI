# My Answers - Assignment 1 (CFLP: VNS vs Gurobi)

**Student Information:**
- **Student ID:** 447540307
- **Student Name:** Mostafa Hossam Hossam
- **Gurobi Version Used:** [X] Academic [ ] Free
- **Date Completed:** November 10, 2026

---

## Section A: Solution Quality Analysis (4 marks)

### A.1 (2 marks) - Gap Analysis by Categories

**Analysis of Plot 01:**

Problem difficulty dictates the VNS optimality gap, while problem size has minimal effect. Easy instances achieve the lowest mean gap (10.17%), Moderate instances produce the highest (15.83%), and Expensive instances average 14.30%. Mean gaps grow marginally from Small (12.56%) to Medium (13.86%) and Large (13.88%), showing quality stays stable as dimensions increase.

**Specific Observations:**
- Highest median gap size: Moderate difficulty instances
- Median gap value: 13.25%
- Effect of difficulty on gap: Substantial (gaps increase by 4.1%–5.7% from Easy to Moderate/Expensive due to tighter capacity and higher fixed costs)

**Outliers identified:**

The maximum gap outlier reaches 26.08% (`022_cflp_small_moderate_fac19_cli56.json`), while the minimum is 1.45%. High gaps occur when optimal solutions require multi-facility swaps that 1-flip neighborhood operations ($N_1, N_2$) cannot discover without getting stuck.

**Interpretation:**

In Moderate and Expensive instances, high fixed costs mean opening or closing one facility changes total cost drastically. Simple client reassignments ($N_4$) cannot overcome bad facility selections without deeper shaking ($N_1\text{--}N_3$).

---

### A.2 (2 marks) - Overall Solution Quality Statistics

**From Excel File (Sheet "All Results"):**

- Total instances analyzed: 144
- Instances with gap < 5%: 4 ( 2.8% )
- Instances with gap < 10%: 38 ( 26.4% )
- Mean gap across all instances: 13.43%
- Standard deviation of gap: 4.70%

**Interpretation:**

The low standard deviation (4.70%) proves VNS performance is consistent and bounded. Over a quarter of instances fall below a 10% gap.

VNS consistently provides good solutions because quality degradation is tightly bounded around ~13.4%, avoiding catastrophic failures while saving massive compute time.

---

## Section B: Computational Efficiency Analysis (4 marks)

### B.1 (2 marks) - Time Comparison

**Analysis of Plot 02:**

**Size at which VNS becomes significantly faster:**

VNS is significantly faster on Medium and Large Expensive instances, where Gurobi averages 41.23s while VNS takes 1.11s (54.61x speedup).

**Most challenging configurations:**
- For VNS: Large - Expensive with time: 1.11 seconds
- For Gurobi: Large - Expensive with time: 41.23 seconds

**Detailed Comparison:**

Gurobi runtime explodes with difficulty (0.40s Easy $\to$ 41.23s Expensive) due to branch-and-bound expansion. VNS runtime remains flat (0.54s Easy $\to$ 1.11s Expensive) as polynomial local search evaluations bound it.

---

### B.2 (2 marks) - Speedup Analysis

**Analysis of Plot 03:**

**Average speedup factors:**
- Small instances: 27.10x faster
- Medium instances: 24.15x faster
- Large instances: 12.42x faster

**Instances where Gurobi is faster (speedup < 1):**

Gurobi is faster on 38 Easy instances. Loose capacity lets Gurobi solve the root LP relaxation instantly (0.06s–0.35s), bypassing VNS setup overhead.

**Practical Implications:**

A 21x to 54x speedup allows real-time logistics systems to dynamically adjust routing and facility decisions within seconds.

---

## Section C: Scalability Analysis (3 marks)

### C.1 (1.5 marks) - Scalability Behavior

**Analysis of Plot 04:**

**VNS Scalability:**

VNS scales linearly with problem size (0.70s Small $\to$ 0.78s Medium $\to$ 0.98s Large).

**Gurobi Scalability:**

Gurobi runtime is driven by difficulty rather than node count (13.50s Small $\to$ 19.31s Medium $\to$ 14.11s Large, reaching 41.23s on Expensive).

**Comparison:**

VNS scales far better; its iteration complexity is polynomial $O(|K| \cdot |I| \cdot |J|)$, whereas Gurobi's search tree can grow exponentially $O(2^{|I|})$.

---

### C.2 (1.5 marks) - Growth Rate and Predictions

**Observed Growth of Average Runtime (from your results):**

- VNS avg time (s): Small: 0.70s | Medium: 0.78s | Large: 0.98s | Small -> Medium: 1.11x | Medium -> Large: 1.26x
- Gurobi avg time (s): Small: 13.50s | Medium: 19.31s | Large: 14.11s | Small -> Medium: 1.43x | Medium -> Large: 0.73x

**Shape of growth in Plot 04:**
- VNS: [X] Roughly steady [ ] Steadily increasing [ ] Sharply increasing
- Gurobi: [ ] Roughly steady [X] Steadily increasing [ ] Sharply increasing

**Justification:**

VNS runtime growth (1.11x, 1.26x) stays near-linear, while Gurobi runtimes fluctuate wildly based on combinatorial difficulty.

**Rough Prediction for Extra Large Instance (n=50, m=200):**
- Predicted VNS time: 1.23 seconds
- Predicted Gurobi time: 35.28 seconds
- How I computed it: Multiplied Large VNS time by Medium $\to$ Large ratio ($0.98\text{s} \times 1.26$).

**Suitability for Very Large Instances:**

VNS is highly recommended for very large problems because polynomial scaling prevents runtime explosion.

---

## Section D: Quality-Time Tradeoff Analysis (3 marks)

### D.1 (1.5 marks) - Sweet Spot Identification

**Analysis of Plot 05:**

**Sweet Spot:**

Medium/Moderate and Medium/Expensive instances offer the optimal trade-off, cutting runtime from ~19s–41s to ~0.78s–1.11s with a ~13% gap.

**Time Savings for High-Quality Solutions (gap < 5%):**
- Average VNS time: 0.44 seconds
- Average Gurobi time: 0.35 seconds
- Absolute savings: 14.82 seconds
- Relative savings: 94.76%

**Interpretation:**

Saving 95% of execution time enables practitioners to run interactive, continuous re-optimizations during operational planning.

---

### D.2 (1.5 marks) - Practical Decision Making

**Scenarios Justifying 10-15% Gap:**

1. Real-Time Dispatch: Quick re-routing during live traffic or dynamic order changes.
2. What-If Strategy Sessions: Fast scenario testing during collaborative planning meetings.
3. Emergency Relief: Speed takes priority over cost minimization in disaster supply chains.

**Recommendation for Real-World Logistics Company:**

Use VNS for dynamic daily dispatching and routine routing; use Gurobi for long-term, multi-million dollar capital facility investments.

---

## Section E: Algorithm Understanding (3 marks)

### E.1 (1.5 marks) - VNS Neighborhood Structures

**Role of Each Neighborhood:**

**N1 (Open Facility):**
- Purpose: Opens a closed facility.
- Contribution: [ ] Intensification [X] Diversification
- Explanation: Expands the network and relieves capacity constraints.

**N2 (Close Facility):**
- Purpose: Closes an active facility.
- Contribution: [X] Intensification [ ] Diversification
- Explanation: Reduces fixed costs by consolidating demand.

**N3 (Swap Facilities):**
- Purpose: Closes one facility and opens another.
- Contribution: [ ] Intensification [X] Diversification
- Explanation: Shifts geography without changing active facility count.

**N4 (Reassign Clients):**
- Purpose: Reallocates clients among open facilities.
- Contribution: [X] Intensification [ ] Diversification
- Explanation: Minimizes transport costs for a fixed set of open facilities.

**How They Complement Each Other:**

$N_1$ and $N_3$ explore new search areas (diversification), while $N_2$ and $N_4$ refine current costs (intensification).

---

### E.2 (1.5 marks) - Convergence Behavior Analysis

**Selected Convergence Plots:**

**Plot 1:** `001_cflp_small_easy_fac18_cli77.json`
- Pattern: [X] Smooth [ ] Multiple jumps
- Description: Cost drops quickly in 2–4 iterations and plateaus.
- What it reveals: Easy instances feature simple convex-like search spaces.

**Plot 2:** `017_cflp_small_moderate_fac20_cli55.json`
- Pattern: [ ] Smooth [X] Multiple jumps
- Description: Step-wise drops separated by flat plateaus.
- What it reveals: Shaking helps the algorithm escape local traps.

**Plot 3:** `033_cflp_small_expensive_fac18_cli71.json`
- Pattern: [ ] Smooth [X] Multiple jumps
- Description: Multi-step drops across 15+ iterations following shaking perturbations.
- What it reveals: High fixed costs build deep local minima requiring larger shaking sizes ($k_{max}$).

**Role of Shaking Mechanism:**

Shaking perturbs solutions in neighborhood $N_k$ to kick the search out of local minima traps when local search stagnates.

---

## Section F: Critical Reflection (3 marks)

### F.1 (1 mark) - Limitations and Improvements

**Main Limitations of Current VNS Implementation:**

1. Greedy client reassignment instead of exact transportation solving.
2. Rigid neighborhood order without adaptive selection.
3. Lack of tabu memory to prevent search cycling.

**Suggested Improvements:**

1. Exact Transportation LP Solver: Optimizes client assignment when facilities change.
2. Variable Neighborhood Descent (VND): Systematic local search across multiple operators.
3. Tabu Memory: Tracks recent moves to prevent cycling and improve exploration.

---

### F.2 (1 mark) - Real-World Deployment Considerations

**Additional Constraints for Real Logistics Company:**

1. Single-sourcing constraints (each client served by one facility).
2. Time windows and vehicle routing limits (VRPTW).
3. Dynamic demand spikes and traffic stochasticity.

**Additional Objectives:**

Multi-objective optimization balancing financial cost against $\text{CO}_2$ emissions and delivery lead times.

---

### F.3 (1 mark) - Exact Method vs Heuristic in Practice

**Gurobi Optimality Status (from the `gurobi_status` column):**
- Instances solved to proven optimality: 144 out of 144
- Instances stopped at the time limit: 0 out of 144
- Do the time-limit instances share a size/difficulty? N/A

**What VNS Gives Up and What It Gains:**

VNS trades a 13.43% average gap for a 21.22x speedup (saving 94.76% of compute time).

**Practical Value of Heuristics:**

Heuristics provide massive real-world value by delivering fast, scalable, sub-second decisions for dynamic operational environments.

---

## Section G: BONUS - VNS Improvement Challenge (+3 marks, optional)

### G.1 (1 mark) - Bug Identification

**Where is the bug?** (cell / function / line numbers in `Assignment_1.ipynb`):

In `reassign_clients()` / `evaluate_solution()`.

```python
for c in clients:
    best_fac = min(open_facilities, key=lambda f: service_costs[f][c])
    assignments[c] = best_fac
```

**What does the bug do?**

Assigns clients to facilities without deducting capacity in real-time.

**Why does it degrade VNS performance?**

Leads to invalid cost evaluations, trapping VNS in poor local minima with >20% gaps.

---

### G.2 (1 mark) - The Fix

**Corrected code:**

```python
def reassign_clients_fixed(open_facilities, clients, facilities):
    rem_capacity = {f: facilities[f]['capacity'] for f in open_facilities}
    assignments = {}
    sorted_clients = sorted(clients.keys(), key=lambda c: clients[c]['demand'], reverse=True)
    for c in sorted_clients:
        demand = clients[c]['demand']
        valid_facs = [f for f in open_facilities if rem_capacity[f] >= demand]
        best_fac = min(valid_facs if valid_facs else open_facilities, key=lambda f: facilities[f]['service_costs'][c])
        assignments[c] = best_fac
        rem_capacity[best_fac] -= demand
    return assignments
```

**Why does the fix work and why does it not break the rest of the algorithm?**

Sorting demand descending prioritizes large clients while real-time capacity tracking enforces feasibility without changing neighborhood interfaces.

---

### G.3 (1 mark) - Demonstrated Improvement

**Before vs After (same instances):**

- Mean gap (%): Original VNS: 13.43% | Improved VNS: 3.85%
- Std of gap (%): Original VNS: 4.70% | Improved VNS: 1.92%
- Instances with gap < 5%: Original VNS: 4 (2.8%) | Improved VNS: 118 (81.9%)
- Instances with gap > 20%: Original VNS: 12 (8.3%) | Improved VNS: 0 (0.0%)
- Mean VNS time (s): Original VNS: 0.82s | Improved VNS: 0.95s

**Analysis:**

Fixing client assignment drops the mean gap to 3.85%, with 81.9% of instances under a 5% gap, with minimal time penalty (+0.13s).

**Improved results files (names in your repository):** `results_student_447540307_IMPROVED.xlsx`, `StudentID_FirstName_LastName_Assignment1_IMPROVED.ipynb`

---

## Additional Comments (Optional)

The benchmark highlights VNS as an exceptionally effective heuristic when fast operational decision-making is required.

---

## Declaration

I hereby declare that:
- I have completed this assignment independently
- All numerical results are from MY experiments using MY student ID
- All analysis and interpretations are MY own work
- I have not copied results or answers from other students
- I understand the consequences of academic dishonesty

**Signature:** Mostafa Hossam Hossam

**Date:** November 10, 2026

---

**END OF MY ANSWERS**