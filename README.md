# Benchmark Instances and Results — E-VSP with Policy-Guided Battery Management

This repository contains the benchmark instances and computational results associated with the paper:

Ricard, L., Desaulniers, G., Lodi, A. & Rousseau, L.-M. (2026). Chance-constrained battery management for electric bus scheduling, *European Journal of Operational Research*.

---

## Problem description

The instances model an **Electric Vehicle Scheduling Problem (E-VSP)**. Energy consumption per trip is **stochastic**, modelled as a discrete probability distribution. The objective is to find a minimum-cost assignment of vehicles to trips while satisfying operational and battery constraints under uncertainty.

---

## Repository structure

```
.
├── instances/
│   ├── shared/
│   │   ├── variations.csv       # Time-of-day variation periods
│   │   └── travel_data.csv      # Deadhead travel times and energy consumption
│   ├── I1/
│   │   ├── locations.csv        # Network locations (4 nodes)
│   │   └── trips_{1..5}.csv     # 5 simulated service trip sets
│   ├── I2/
│   │   ├── locations.csv
│   │   └── trips_{1..5}.csv
│   └── I3/
│       ├── locations.csv
│       └── trips_{1..5}.csv
└── results/
    └── all_results.csv          # Full computational results (all instances × all configurations)
```

All three instance families (I1, I2, I3) share the same **network topology** (`shared/`) but differ in the number and structure of service trips.

---

## File formats

### `instances/shared/variations.csv`

Defines the time-of-day **variation periods** used to look up travel times and energy consumption.

| Column | Type | Description |
|---|---|---|
| `variation_ID` | int | Identifier of the variation period |
| `start_time` | int | Period start, in minutes from midnight |
| `end_time` | int | Period end, in minutes from midnight |

To determine which variation applies to a trip departing at time *t*, find the row where `start_time ≤ t ≤ end_time`.

---

### `instances/shared/travel_data.csv`

**Deadhead** travel parameters for each ordered pair of locations.

| Column | Type | Description |
|---|---|---|
| `from_loc` | str | Origin location ID |
| `to_loc` | str | Destination location ID |
| `travel_time_min` | dict | Travel time (minutes), keyed by `variation_ID` |
| `energy_consumption_pct` | dict | Energy consumed (% of battery capacity), keyed by `variation_ID` |

Both `travel_time_min` and `energy_consumption_pct` are stored as Python-style dictionaries `{variation_id: value}`. Use the variation ID matching the scheduled departure time of the preceding trip (see `variations.csv`).

Battery capacity is **300 kWh**; multiply `energy_consumption_pct / 100` by 300 to obtain kWh.

---

### `instances/I{1,2,3}/locations.csv`

Network **nodes** with their type and capacity attributes.

| Column | Type | Description |
|---|---|---|
| `location_id` | str | Unique location identifier |
| `type` | str | One of `terminal`, `depot`, `charging_station` |
| `depot_capacity` | int | Max vehicles that can be stored (depots only) |
| `charging_capacity` | int | Max vehicles that can charge simultaneously (charging stations only) |

Unused capacity fields are left blank.

---

### `instances/I{1,2,3}/trips_{1..5}.csv`

**Service trips** to be covered. Each family contains 5 independently simulated instances.

| Column | Type | Description |
|---|---|---|
| `trip_id` | str | Unique trip identifier |
| `start_loc` | str | Origin location ID |
| `start_time` | int | Scheduled departure, minutes from midnight |
| `end_loc` | str | Destination location ID |
| `end_time` | int | Scheduled arrival, minutes from midnight |
| `distance_km` | float | Trip distance in kilometres |
| `energy_min_pct` | int | Minimum discrete energy consumption (% of battery) |
| `energy_max_pct` | int | Maximum discrete energy consumption (% of battery) |
| `energy_probabilities` | str | Semicolon-separated probabilities for each integer energy level from `energy_min_pct` to `energy_max_pct` (inclusive) |

The energy consumption of trip *i* is a discrete random variable on `{energy_min_pct, energy_min_pct+1, ..., energy_max_pct}`. The *k*-th probability in `energy_probabilities` corresponds to the *k*-th integer in that range. Probabilities sum to 1.

**Example:** `energy_min_pct=2`, `energy_max_pct=4`, `energy_probabilities=0.2;0.5;0.3` means P(2%) = 0.2, P(3%) = 0.5, P(4%) = 0.3.

---

### `results/all_results.csv`

Computational results for all instances × model types × parameter configurations.

| Column | Description |
|---|---|
| `test` | Instance identifier, e.g. `I1_1` (family I1, simulation 1) |
| `type` | Model type: `worst-case`, `optimistic`, or `stochastic` |
| `epsilon` | Chance constraint threshold ε (stochastic model only; 0 for worst-case/optimistic) |
| `sigma^min` | Minimum authorized SoC |
| `sigma^low` | Lower bound on the recommended SoC range |
| `sigma^up` | Upper bound on the recommended SoC range |
| `sigma^max` | Maximum authorized SoC |
| `UB` | Best upper bound (primal solution value) |
| `LB` | Best lower bound (dual bound) |
| `gap(%)` | Optimality gap: `(UB - LB) / LB × 100` |
| `#Ebs` | Number of electric buses used in the solution |
| `BBn` | Number of branch-and-bound nodes explored |
| `CPU_total` | Total CPU time (seconds) |
| `CPU_root` | CPU time at the root node (seconds) |
| `CPU_pricing` | CPU time spent solving the pricing subproblems (seconds) |
