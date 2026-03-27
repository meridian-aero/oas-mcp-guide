---
name: run-multipoint-opt
description: >
  Run a multipoint aerostructural optimization with cruise and maneuver conditions.
  This is the most common pattern in OAS literature: size the wing structure at
  a 2.5g maneuver point while minimizing fuel burn at cruise. Use when the user
  mentions multipoint, cruise+maneuver, 2.5g sizing, structural sizing with
  maneuver loads, or realistic wing optimization with multiple flight conditions.
  Triggers on: "multipoint optimization", "cruise and maneuver", "2.5g sizing",
  "realistic structural optimization", "multipoint", "maneuver sizing".
argument-hint: "[cruise_Mach] [maneuver_LF]  e.g. 'Mach 0.84 cruise, 2.5g maneuver'"
context: fork
---

# Multipoint Aerostructural Optimization

Size wing structure at a critical maneuver condition while minimizing fuel burn at cruise. This is the standard approach in the OAS literature for realistic transport wing design.

## When to Use Multipoint vs Single-Point

- **Single-point**: Quick screening, aero-only, simple structural sizing
- **Multipoint**: Realistic structural sizing where the maneuver case sizes the structure but cruise determines fuel burn. Use for any serious structural optimization.

## Workflow

### 1. Start Session

```
start_session(notes="Multipoint optimization: cruise M=<Mach>, maneuver <LF>g")
```

### 2. Gather Parameters

Parse `$ARGUMENTS` or ask the user:
- **Cruise Mach** (default: 0.84 for wide-body transport)
- **Maneuver load factor** (default: 2.5 for FAR 25.337)
- **Wing type** (default: `uCRM_based` for multipoint — preferred over `CRM`)
- **FEM model** (default: `wingbox` — standard for multipoint in literature)
- **Material** (default: aluminum — see `shared/material-database.md`)

For flight condition details, see [flight-conditions-reference.md](flight-conditions-reference.md).

### 3. Create Surface

```
create_surface(
    name="wing", wing_type="uCRM_based", num_x=2, num_y=11,
    symmetry=True, with_viscous=True, with_wave=True, CD0=0.015,
    fem_model_type="wingbox",
    E=70e9, G=30e9, yield_stress=500e6, mrho=3000.0,
    S_ref_type="wetted",
    distributed_fuel_weight=True, struct_weight_relief=True,
    fuel_density=803.0, Wf_reserve=15000.0
)
```

Use `num_y=11` or higher for multipoint (more spanwise resolution needed). Log mesh decision.

### 4. Log Optimization Setup

```
log_decision(
    decision_type="dv_selection",
    reasoning="Multipoint wingbox: twist + spar/skin thickness + t/c + alpha for cruise+maneuver sizing",
    selected_action="twist, spar_thickness, skin_thickness, t_over_c, alpha"
)

log_decision(
    decision_type="constraint_choice",
    reasoning="Failure at maneuver point sizes structure; fuel volume constraint ensures tank feasibility",
    selected_action="failure<=0 at point 1, L_equals_W=0 at both points, fuel_vol_delta>=0, fuel_diff=0"
)
```

### 5. Run Optimization

```
run_optimization(
    surfaces=["wing"],
    analysis_type="aerostruct",
    objective="fuelburn",
    design_variables=[
        {"name": "twist", "lower": -10, "upper": 15},
        {"name": "spar_thickness", "lower": 0.001, "upper": 0.05, "scaler": 1000},
        {"name": "skin_thickness", "lower": 0.001, "upper": 0.05, "scaler": 1000},
        {"name": "t_over_c", "lower": 0.06, "upper": 0.20, "scaler": 10},
        {"name": "alpha", "lower": -5, "upper": 10}
    ],
    constraints=[
        {"name": "L_equals_W", "equals": 0.0, "point": 0},
        {"name": "L_equals_W", "equals": 0.0, "point": 1},
        {"name": "failure", "upper": 0.0, "point": 1},
        {"name": "fuel_vol_delta", "lower": 0.0},
        {"name": "fuel_diff", "equals": 0.0}
    ],
    flight_points=[
        {"velocity": 248.136, "Mach_number": 0.84, "density": 0.38, "reynolds_number": 1e6, "speed_of_sound": 295.4, "load_factor": 1.0},
        {"velocity": 189.0, "Mach_number": 0.64, "density": 0.38, "reynolds_number": 1e6, "speed_of_sound": 295.4, "load_factor": 2.5}
    ],
    objective_scaler=1e-5,
    tolerance=1e-6,
    max_iterations=400
)
```

### 6. Assess Convergence

The response contains `final_results` keyed by role: `{"cruise": {...}, "maneuver": {...}}`.

```
log_decision(
    decision_type="convergence_assessment",
    reasoning="success=<bool>, <N> iters. Cruise: fuelburn=<val>, L/D=<val>. Maneuver: failure=<val>.",
    selected_action="<accept / re-run with adjusted bounds>",
    prior_call_id="<_provenance.call_id>"
)
```

### 7. Visualize

```
visualize(run_id="<run_id>", plot_type="multipoint_comparison", output="file")
visualize(run_id="<run_id>", plot_type="stress_distribution", output="file")
visualize(run_id="<run_id>", plot_type="failure_heatmap", output="file")
```

### 8. Export and Report

```
export_session_graph(output_path="multipoint_opt_provenance.json")
```

Return a summary table to the main conversation:

| Metric | Cruise | Maneuver |
|---|---|---|
| CL | ... | ... |
| CD | ... | ... |
| L/D | ... | ... |
| Failure | ... | ... |
| Structural mass | ... | ... |
| Fuel burn | ... | — |
