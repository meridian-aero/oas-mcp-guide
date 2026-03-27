---
name: compute-drag-polar
description: >
  Generate a drag polar (CL vs CD curve) and find the best L/D operating point.
  Use when the user wants a drag polar, CL-CD curve, performance envelope,
  best L/D, alpha sweep, or wants to find the optimum angle of attack.
  Triggers on: "drag polar", "CL-CD curve", "best L/D", "sweep alpha",
  "operating point", "performance map", "L/D vs alpha".
argument-hint: "[alpha_range] [Mach]  e.g. '-5 to 15 deg at Mach 0.84'"
---

# Drag Polar Generation

Sweep angle of attack to produce a CL-CD polar and identify the best operating point.

## Workflow

### 1. Start Session

```
start_session(notes="Drag polar: <wing_type> at Mach <Mach>")
```

### 2. Create Surface (if not already defined)

```
create_surface(
    name="wing", wing_type="CRM", num_x=2, num_y=7,
    symmetry=True, with_viscous=True, CD0=0.015
)
```

Log mesh decision after creation.

### 3. Compute Drag Polar

```
compute_drag_polar(
    surfaces=["wing"],
    alpha_start=-5.0, alpha_end=15.0, num_alpha=21,
    velocity=248.136, Mach_number=0.84, density=0.38, reynolds_number=1e6
)
```

Default alpha range of -5 to 15 in 21 steps covers most operating conditions. Adjust if the user specifies different limits.

### 4. Interpret Results

The response contains arrays: `alphas`, `CLs`, `CDs`, `CMs`, `L_over_Ds`, and scalar `best_L_over_D`.

Key things to report:
- **Best L/D**: value and the alpha at which it occurs
- **CL range**: usable CL range before stall indication
- **Drag breakdown at key alphas**: CDi vs CDv vs CDw percentages
- **CL at user's target alpha** (if specified)

```
log_decision(
    decision_type="result_interpretation",
    reasoning="Best L/D = <val> at alpha = <val>deg (CL=<val>). Usable range: alpha <min> to <max>.",
    selected_action="<recommend operating point or next analysis>",
    prior_call_id="<_provenance.call_id>"
)
```

### 5. Visualize

```
visualize(run_id="<run_id>", plot_type="drag_polar", output="file")
```

### 6. Optional: Stability at Operating Point

If the user cares about stability, call `compute_stability_derivatives` at the best L/D alpha:

```
compute_stability_derivatives(
    surfaces=["wing"], alpha=<best_alpha>,
    velocity=248.136, Mach_number=0.84, density=0.38, reynolds_number=1e6,
    cg=<estimated_cg>
)
```

Report CL_alpha, CM_alpha, and static margin.

### 7. Export Provenance

```
export_session_graph(output_path="drag_polar_provenance.json")
```

## Report Format

| Alpha (deg) | CL | CD | L/D | CM |
|---|---|---|---|---|
| ... | ... | ... | ... | ... |

**Best L/D**: XX.X at alpha = X.X deg (CL = X.XX)
