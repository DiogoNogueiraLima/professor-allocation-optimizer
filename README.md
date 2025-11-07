# Professor Allocation Optimizer

This repository implements and documents a solution to the **Balanced Academic Curriculum Problem (BACP)**, catalogued as [Problem 030 on CSPLib](https://www.csplib.org/Problems/prob030/). The goal is to assign every mandatory course in a curriculum to one of the available academic periods while keeping the workload balanced and honouring institutional rules such as prerequisite ordering and per-period load limits. All experiments, diagnostics, and plots live in `notebook.ipynb`.

## Problem overview

Given:

- a set of courses, each with a contact-hour load and optional prerequisites;
- a fixed number of academic periods (semesters/terms);
- institutional policies limiting the minimum/maximum number of hours and courses allowed per period.

Find an assignment of courses to periods that:

1. Respects **hard constraints**
   - courses can only be scheduled after every prerequisite has been completed;
   - each period must host between the allowed minimum and maximum number of courses;
   - the cumulative workload per period must remain within the allowed hour range.
2. Minimises **workload imbalance**.

This mirrors the CSPLib definition of the BACP, where the cost function combines balance objectives with heavy penalties for any infeasible assignment.

## Cost calculation

For any schedule we:

1. Convert it to list/dict form to keep period indices consistent with the dataset order.
2. Aggregate total hours and course counts per period and compute the population variance of the hours as the base cost (balanced workloads ⇒ lower cost).
3. Add a large penalty whenever a period violates the hour or course bounds. Each unit above/below the bound is multiplied by a penalty factor (largest course load × number of courses).
4. Add another penalty whenever a prerequisite appears in the same or a later period than its dependent course.

Feasible assignments therefore dominate infeasible ones, while the variance term keeps the per-period loads as even as possible. Section **1.3** in the notebook documents the exact formulas.

## Repository contents

| Path | Description |
| ---- | ----------- |
| `data.json` | Instance used in the experiments (46 courses, 8 periods, 33 prerequisite relations). Contains `num_periods`, hour/course limits, a course list, the corresponding `course_hours`, and the prerequisite tuples. |
| `notebook.ipynb` | Notebook describing the problem, loading the dataset, running the optimisation methods, and rendering the schedules as agenda-like tables for easy inspection. |

## Solutions explored

The notebook currently compares three heuristic approaches. Each section explains the logic, shows pseudo-code or parameter tables, and plots convergence + best schedules.

### 3. Hill Climbing

- Random initial schedule plus two neighbourhood operators (single-course reassignment and pairwise swap).
- Strictly greedy acceptance keeps any improving neighbour, and multiple seeds (15 by default) expose how sensitive the procedure is to the starting point.
- Results: reaches feasible schedules quickly but can stagnate when prerequisites force tight bottlenecks.

### 4. Genetic Algorithm (GA)

- Uses tournament selection, one-point crossover, point-wise mutation, and elitism.
- A hyperparameter sweep precedes the multi-seed evaluation so the best `(pop_size, num_generations, crossover_rate)` combination feeds the robustness analysis.
- Schedule outputs are rendered both for the sweep winner and for the best seed, making it easy to compare alternative timetables.

### 5. Firefly-based metaheuristic

- Implements the modified discrete firefly algorithm described in Schinas et al., with brightness computed as `1 / (1 + cost)` and Hamming distance governing attractiveness.
- After each relocation we apply a sigmoid-based scatter step that randomly reassigns a subset of courses, keeping the swarm diverse.
- Once the search is warmed up (iteration > 2) a lightweight local search (swap/reassign moves) refines the incumbent best firefly, mirroring the “local search mechanism” from Algorithm 1.
- Results: convergence is smoother than hill climbing and the GA, often producing different schedules.

## Results summary

- **Constraint satisfaction**: For this experiment the firefly search was the only one that returned violation-free schedules; hill climbing and GA leaved a minor violation when forced into tight prerequisite chains.
- **Schedule inspection**: the notebook prints per-period tables (columns = periods, rows = course slots with `course - hours`) so domain experts can validate the final assignments visually.
- **Next steps** (outlined in notebook Section 4.8): extend the hyperparameter sweep (mutation rate, elitism), add local repair heuristics, and benchmark against exact/other metaheuristic baselines.

## Running the experiments

1. **Install dependencies** (Matplotlib is required for the plots used in the notebooks):
   ```bash
   pip install matplotlib pandas
   ```
2. **Launch Jupyter** from the repository root:
   ```bash
   jupyter notebook
   ```
3. **Open `notebook.ipynb`** and run the cells sequentially. The notebook will:
   - load `data.json` and print the instance summary plus cost-calculation details;
   - execute hill climbing, GA (with tuning + multi-seed diagnostics), and the firefly search;
   - render convergence plots and the agenda-style tables for every best schedule.

## References

- *Balanced Academic Curriculum Problem* — CSPLib Problem 030: <https://www.csplib.org/Problems/prob030/>
- Schinas, P. et al. (2023). *Balancing Academic Curriculum Problem Solution: A Discrete Firefly-Based Approach*.
- De Werra, D. (1985). *An introduction to timetabling*. European Journal of Operational Research, 19(2), 151–162.

The implementation follows the spirit of the CSPLib formulation while focusing on heuristic solvers implemented in Python notebooks for experimentation, comparison, and visual analysis.
