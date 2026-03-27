---
name: run-aero-opt
description: >
  Optimize wing aerodynamics: minimize drag at fixed CL, maximize L/D, or
  optimize twist/planform shape. Use when the user wants to minimize CD,
  optimize twist, find the best wing shape at a given CL, or do any
  aero-only optimization. Triggers on: "optimize for drag", "minimize CD",
  "best twist for CL=0.5", "aero optimization", "maximize L/D",
  "optimize planform", "reduce drag".
argument-hint: "[target_CL] [DVs]  e.g. 'CL=0.5 with twist and alpha'"
---

# Aero-Only Optimization

Minimize CD (or another aero objective) subject to a CL constraint by varying aerodynamic design variables.

## Workflow

### 1. Start Session

```
start_session(notes="Aero optimization: min <objective> at CL=<target>")
```

### 2. Create Surface

Aero-only — no `fem_model_type` needed:

```
create_surface(
    name="wing", wing_type="CRM", num_x=2, num_y=7,
    symmetry=True, with_viscous=True, CD0=0.015
)
```

Log mesh decision.

### 3. Baseline Analysis

Run a baseline `run_aero_analysis` to:
- Establish initial CD for computing `objective_scaler`
- Verify the surface is set up correctly

```
run_aero_analysis(
    surfaces=["wing"], alpha=5.0,
    velocity=248.136, Mach_number=0.84, density=0.38, reynolds_number=1e6
)
```

Compute scaler: `objective_scaler = 1.0 / baseline_CD`. See `shared/scaling-guide.md` for details.

Log result interpretation with `prior_call_id`.

### 4. Log Optimization Setup

```
log_decision(
    decision_type="dv_selection",
    reasoning="Twist + alpha provide the most drag reduction for aero-only problems",
    selected_action="twist (3 cp, -10 to 10 deg), alpha (-5 to 15 deg)"
)

log_decision(
    decision_type="constraint_choice",
    reasoning="CL=<target> ensures lift is maintained while drag is minimized",
    selected_action="CL equals <target>"
)
```

### 5. Run Optimization

```
run_optimization(
    surfaces=["wing"],
    analysis_type="aero",
    objective="CD",
    design_variables=[
        {"name": "twist", "lower": -10, "upper": 10, "n_cp": 3},
        {"name": "alpha", "lower": -5, "upper": 15}
    ],
    constraints=[{"name": "CL", "equals": 0.5}],
    objective_scaler=<computed_scaler>,
    Mach_number=0.84, density=0.38, velocity=248.136, reynolds_number=1e6
)
```

### 6. Assess Convergence

Check `results.success` and `results.optimization_history`:

```
log_decision(
    decision_type="convergence_assessment",
    reasoning="success=<bool>, <N> iterations, CD reduced by <X>%",
    selected_action="<accept / re-run with tighter tolerance>",
    prior_call_id="<_provenance.call_id>"
)
```

### 7. Visualize

```
visualize(run_id="<run_id>", plot_type="opt_history", output="file")
visualize(run_id="<run_id>", plot_type="opt_dv_evolution", output="file")
visualize(run_id="<run_id>", plot_type="opt_comparison", output="file")
```

### 8. Export Provenance

```
export_session_graph(output_path="aero_opt_provenance.json")
```

## Design Variable Reference

| DV | Description | Typical Bounds |
|---|---|---|
| `twist` | Spanwise twist distribution | -10 to 15 deg |
| `alpha` | Angle of attack | -5 to 15 deg |
| `chord` | Spanwise chord distribution | 0.5 to 3.0 m |
| `sweep` | Wing sweep angle | 0 to 35 deg |
| `taper` | Taper ratio | 0.2 to 1.0 |
| `t_over_c` | Thickness-to-chord ratio | 0.06 to 0.20 |

For constraint reference, see `shared/common-constraints.md`.
