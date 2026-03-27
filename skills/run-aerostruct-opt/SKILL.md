---
name: run-aerostruct-opt
description: >
  Optimize wing structure for minimum fuel burn or structural mass using
  coupled aerostructural analysis. Use when the user wants to minimize fuel
  burn, find the lightest feasible wing, optimize thickness distribution,
  or do structural sizing optimization. Triggers on: "minimize fuel burn",
  "structural optimization", "size the wing", "minimum weight wing",
  "optimize thickness", "aerostruct optimization", "structural sizing".
argument-hint: "[objective] [material]  e.g. 'min fuelburn with aluminum tube'"
---

# Aerostructural Optimization

Minimize fuel burn or structural mass by varying structural and aerodynamic design variables simultaneously.

## Workflow

### 1. Start Session

```
start_session(notes="Aerostruct optimization: min <objective> with <material> <fem_type>")
```

### 2. Create Surface with Structure

**Tube spar** (faster, fewer DVs):
```
create_surface(
    name="wing", wing_type="CRM", num_x=2, num_y=7,
    symmetry=True, with_viscous=True, CD0=0.015,
    fem_model_type="tube",
    E=70e9, G=30e9, yield_stress=500e6, mrho=3000.0
)
```

**Wingbox** (more realistic, more DVs):
```
create_surface(
    name="wing", wing_type="CRM", num_x=2, num_y=7,
    symmetry=True, with_viscous=True, CD0=0.015,
    fem_model_type="wingbox",
    E=70e9, G=30e9, yield_stress=500e6, mrho=3000.0
)
```

For material properties, see `shared/material-database.md`. Log mesh decision.

### 3. Baseline Analysis

Run `run_aerostruct_analysis` to get baseline fuelburn/structural_mass for the objective scaler:

```
run_aerostruct_analysis(
    surfaces=["wing"], alpha=5.0,
    velocity=248.136, Mach_number=0.84, density=0.38,
    reynolds_number=1e6, speed_of_sound=295.4, W0=120000.0
)
```

Compute: `objective_scaler = 1.0 / baseline_fuelburn` (or `1.0 / baseline_structural_mass`).
See `shared/scaling-guide.md` for recommended values.

### 4. Log Optimization Setup

**Design variables** depend on the structural model:

Tube:
```
log_decision(
    decision_type="dv_selection",
    reasoning="twist + thickness + alpha for tube spar min-fuelburn",
    selected_action="twist (-10 to 15), thickness (0.003 to 0.25, scaler=100), alpha (-5 to 10)"
)
```

Wingbox:
```
log_decision(
    decision_type="dv_selection",
    reasoning="twist + spar_thickness + skin_thickness + alpha for wingbox min-fuelburn",
    selected_action="twist, spar_thickness (scaler=1000), skin_thickness (scaler=1000), alpha"
)
```

Constraints (same for tube and wingbox):
```
log_decision(
    decision_type="constraint_choice",
    reasoning="Trim, structural feasibility, and no spar intersection",
    selected_action="L_equals_W=0, failure<=0, thickness_intersects<=0"
)
```

### 5. Run Optimization

Tube example:
```
run_optimization(
    surfaces=["wing"],
    analysis_type="aerostruct",
    objective="fuelburn",
    design_variables=[
        {"name": "twist", "lower": -10, "upper": 15},
        {"name": "thickness", "lower": 0.003, "upper": 0.25, "scaler": 100},
        {"name": "alpha", "lower": -5, "upper": 10}
    ],
    constraints=[
        {"name": "L_equals_W", "equals": 0.0},
        {"name": "failure", "upper": 0.0},
        {"name": "thickness_intersects", "upper": 0.0}
    ],
    objective_scaler=<computed_scaler>,
    W0=120000.0, velocity=248.136, Mach_number=0.84,
    density=0.38, reynolds_number=1e6, speed_of_sound=295.4
)
```

### 6. Assess Convergence

```
log_decision(
    decision_type="convergence_assessment",
    reasoning="success=<bool>, <N> iters, fuelburn: <before> → <after> (<X>% reduction), failure=<val>",
    selected_action="<accept / re-run>",
    prior_call_id="<_provenance.call_id>"
)
```

### 7. Visualize

```
visualize(run_id="<run_id>", plot_type="opt_history", output="file")
visualize(run_id="<run_id>", plot_type="stress_distribution", output="file")
visualize(run_id="<run_id>", plot_type="failure_heatmap", output="file")
visualize(run_id="<run_id>", plot_type="deflection_profile", output="file")
```

### 8. Export Provenance

```
export_session_graph(output_path="aerostruct_opt_provenance.json")
```

## Key Design Variable Reference

| DV | Model | Bounds | Scaler |
|---|---|---|---|
| twist | all | -10 to 15 deg | 1 |
| alpha | all | -5 to 10 deg | 1 |
| thickness | tube | 0.003 to 0.25 m | 100 |
| spar_thickness | wingbox | 0.001 to 0.05 m | 1000 |
| skin_thickness | wingbox | 0.001 to 0.05 m | 1000 |
| t_over_c | all | 0.06 to 0.20 | 10 |
| chord | all | 0.5 to 3.0 m | 1 |

For constraint reference, see `shared/common-constraints.md`.
