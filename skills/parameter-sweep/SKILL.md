---
name: parameter-sweep
description: >
  Run a parametric sweep across any OAS parameter (aspect ratio, taper, sweep,
  thickness, twist, Mach, alpha) and compile results into a trade study.
  Use when the user wants to understand how a parameter affects performance,
  run a sensitivity study, or generate trade plots. Triggers on: "sweep",
  "trade study", "parameter study", "sensitivity analysis", "how does AR affect",
  "vary taper from", "parametric study", "design of experiments".
argument-hint: "[parameter] [range]  e.g. 'sweep aspect ratio from 6 to 18 in 5 steps'"
context: fork
---

# Parametric Sweep

Systematically vary a parameter and record performance metrics to understand sensitivities and identify optimal regions.

## Workflow

### 1. Start Session

```
start_session(notes="Parameter sweep: <parameter> from <min> to <max>")
```

### 2. Define Sweep

Parse `$ARGUMENTS` or ask the user:
- **Parameter to sweep**: aspect ratio (via `span`), `taper`, `sweep`, `alpha`, `Mach_number`, `thickness_cp`, etc.
- **Range**: min, max, number of steps
- **Analysis type**: aero or aerostruct
- **Fixed conditions**: flight conditions, constraints

If the user says "sweep AR from 6 to 18", translate to span values: `span = AR * mean_chord`.

### 3. Execute Sweep

For each parameter value:

```
reset()  # Clear cached problem — required between geometry changes

create_surface(name="wing", ..., <varied_parameter>=<value>)

# Run appropriate analysis
run_aero_analysis(surfaces=["wing"], ...)  # or run_aerostruct_analysis

pin_run(run_id="<run_id>")  # Preserve for later comparison

log_decision(
    decision_type="result_interpretation",
    reasoning="At <param>=<value>: CL=<>, CD=<>, L/D=<>",
    selected_action="continue sweep",
    prior_call_id="<_provenance.call_id>"
)
```

**Important**: Call `reset()` before each iteration because `create_surface` with changed geometry on a cached problem may produce stale results.

### 4. Compile Results

Build a table of all sweep points:

| <Parameter> | CL | CD | L/D | structural_mass | fuelburn |
|---|---|---|---|---|---|
| value_1 | ... | ... | ... | ... | ... |
| value_2 | ... | ... | ... | ... | ... |

### 5. Analyze Trends

Identify:
- **Optimal value**: where the primary metric (L/D, fuelburn) is best
- **Diminishing returns**: where improvement plateaus
- **Trade-offs**: e.g., higher AR improves L/D but increases structural mass
- **Pareto front**: if multiple objectives, which designs are Pareto-optimal

```
log_decision(
    decision_type="result_interpretation",
    reasoning="<trend analysis and optimal point identification>",
    selected_action="<recommended design point and rationale>"
)
```

### 6. Export and Report

```
export_session_graph(output_path="parameter_sweep_provenance.json")
```

Return the results table and analysis to the main conversation.

## Common Sweep Parameters

| Parameter | How to Vary | Notes |
|---|---|---|
| Aspect ratio | Change `span` (AR = span²/S_ref) | Keep root_chord fixed |
| Taper ratio | Change `taper` | 0.2 to 1.0 |
| Sweep angle | Change `sweep` | 0 to 35 deg |
| Thickness | Change `thickness_cp` uniformly | Structural sizing |
| Alpha | Change `alpha` in analysis call | Fixed geometry |
| Mach number | Change `Mach_number` in analysis | Fixed geometry |
| num_y | Change `num_y` (mesh convergence) | Use the mesh-convergence-study skill instead |

## Tips

- Use **aero analysis** for purely aerodynamic sweeps (faster)
- Use **aerostruct analysis** when structural weight matters (e.g., AR sweep)
- Pin runs with `pin_run` to prevent cache eviction during long sweeps
- For 2D sweeps (two parameters), use nested loops — be mindful of total run count
