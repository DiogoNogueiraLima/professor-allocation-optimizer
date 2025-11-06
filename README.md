# Professor Allocation Optimizer

This repository implements and documents a solution to the **Balanced Academic Curriculum Problem (BACP)**, catalogued as [Problem 030 on CSPLib](https://www.csplib.org/Problems/prob030/). The goal is to assign every mandatory course in a curriculum to one of the available academic periods while keeping the workload balanced and honouring institutional rules such as prerequisite ordering and per-period load limits.

## Problem Overview

Given:

- a set of courses, each with a contact-hour load and optional prerequisites;
- a fixed number of academic periods (semesters/terms);
- institutional policies limiting the minimum/maximum number of hours and courses allowed per period.

Find an assignment of courses to periods that:

1. Respects **hard constraints**
   - courses can only be scheduled after every prerequisite has been completed;
   - each period must host between the allowed minimum and maximum number of courses;
   - the cumulative workload per period must remain within the allowed hour range.
2. Minimises **workload imbalance**, typically measured as the variance of total hours per period.

This mirrors the CSPLib definition of the BACP, where the cost function combines balance objectives with heavy penalties for any infeasible assignment.

## Repository Contents

| Path | Description |
| ---- | ----------- |
| `data.json` | Instance used in the experiments (46 courses, 8 periods, 33 prerequisite relations). Contains `num_periods`, hour/course limits, a course list, the corresponding `course_hours`, and the prerequisite tuples. |
| `notebook.ipynb` | Main notebook (English) describing the problem, loading the dataset, and running both optimisation methods. Includes extensive markdown sections that explain modelling decisions and metrics. |

## Solution Approach

The notebook tackles the BACP with two metaheuristics:

1. **Hill Climbing**
   - Starts from a random schedule (dictionary mapping `course -> period`).
   - Uses two neighbourhood operators: (a) reassign a single course to a different period, and (b) swap the periods of two courses.
   - Ensures every generated neighbour differs from the current state to avoid stagnation.
   - Accepts strictly improving neighbours and records the best found cost. Multiple seeds are executed to mitigate local minima.

2. **Genetic Algorithm**
   - Maintains a population of candidate schedules encoded as lists matching the course order.
   - Uses tournament selection, one-point crossover, and point-wise mutation to explore the search space.
   - Applies elitism so the best individual survives each generation.
   - Evaluates several seeds and hyperparameter configurations (population size, number of generations, crossover rate) to gauge robustness.

Both methods rely on the same **penalty-based cost function**: workload variance serves as the balance measure, while violations of hour/course bounds or prerequisite ordering add a penalty proportional to the heaviest course load times the number of courses. This ensures that feasible schedules always outrank infeasible ones.

## Running the Experiments

1. **Install dependencies** (Matplotlib is required for the plots used in the notebooks):
   ```bash
   pip install matplotlib
   ```
2. **Launch Jupyter** from the repository root:
   ```bash
   jupyter notebook
   ```
3. **Open `notebook.ipynb`** and run the cells sequentially. The notebook will:
   - load `data.json` and print the instance summary;
   - execute the hill-climbing runs across multiple seeds and plot best costs;
   - execute the genetic algorithm runs, including the hyperparameter sweep and corresponding plots;
   - print any constraint violations detected in the best solutions.

## References

- *Balanced Academic Curriculum Problem* — CSPLib Problem 030: <https://www.csplib.org/Problems/prob030/>
- De Werra, D. (1985). *An introduction to timetabling*. European Journal of Operational Research, 19(2), 151–162. (Original source describing the curriculum-balancing context.)

The implementation in this repository follows the spirit of the CSPLib formulation while focusing on heuristic solvers implemented in Python notebooks for experimentation and analysis.
