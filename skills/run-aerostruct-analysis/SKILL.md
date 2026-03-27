---
name: run-aerostruct-analysis
description: >
  Run a coupled aerostructural analysis with structural sizing and stress.
  Also handles structural-only questions since OAS always couples aero and
  structures. Use when the user wants structural mass, failure metrics,
  fuel burn estimates, deflection, stress distribution, or combined
  aero+structural results. Triggers on: "structural analysis", "fuel burn",
  "stress", "aerostructural", "deflection", "structural mass",
  "does the wing fail", "structural sizing", "weight estimate".
argument-hint: "[material] [W0] [load_factor]  e.g. 'aluminum W0=120000 at 2.5g'"
---

# Aerostructural Analysis

Run a coupled VLM + beam FEM analysis to get aerodynamic performance and structural response simultaneously.

## Prerequisites

The surface **must** be created with a structural model. See `shared/material-database.md` for property values.

## Workflow

### 1. Start Session

```
start_session(notes="Aerostruct analysis: <material>, W0=<W0>, LF=<LF>")
```

### 2. Create Surface with Structure

Parse `$ARGUMENTS` for material choice, W0, and load_factor.

**Tube spar** (simpler, faster):
```
create_surface(
    name="wing", wing_type="CRM", num_x=2, num_y=7,
    symmetry=True, with_viscous=True, CD0=0.015,
    fem_model_type="tube",
    E=70e9, G=30e9, yield_stress=500e6, mrho=3000.0
)
```

**Wingbox** (realistic, for detailed sizing):
```
create_surface(
    name="wing", wing_type="CRM", num_x=2, num_y=7,
    symmetry=True, with_viscous=True, CD0=0.015,
    fem_model_type="wingbox",
    E=70e9, G=30e9, yield_stress=500e6, mrho=3000.0
)
```

For other materials, look up E, G, yield_stress, mrho in `shared/material-database.md`.

### 3. Log Mesh Decision

```
log_decision(
    decision_type="mesh_resolution",
    reasoning="<why this mesh, structural model, and material>",
    selected_action="<configuration summary>"
)
```

### 4. Run Analysis

```
run_aerostruct_analysis(
    surfaces=["wing"], alpha=5.0,
    velocity=248.136, Mach_number=0.84, density=0.38,
    reynolds_number=1e6, speed_of_sound=295.4,
    W0=120000.0, load_factor=1.0
)
```

For flight conditions by aircraft class, see `shared/standard-flight-conditions.md`.

### 5. Interpret Results

**Critical failure interpretation** — this is the most common source of confusion:

| failure value | Meaning | Action |
|---|---|---|
| `failure < 0` | Safe — margin available | OK |
| `0 < failure < 1` | Feasible but reduced margin | May want to increase thickness |
| **`failure > 1.0`** | **Structural failure** | Increase thickness, change material, or reduce load |

Other key results:
- **structural_mass**: total structural weight in kg
- **fuelburn**: mission fuel burn in kg (uses Breguet range equation)
- **L_equals_W residual**: should be near 0 for trim; if |L_equals_W| > 0.1, alpha or W0 may need adjustment
- **Von Mises stress distribution**: highest near root (element index ny-2)

```
log_decision(
    decision_type="result_interpretation",
    reasoning="failure=<val> (<safe/failed>), struct_mass=<val>kg, fuelburn=<val>kg, L_equals_W=<val>",
    selected_action="<next step>",
    prior_call_id="<_provenance.call_id>"
)
```

### 6. Visualize

```
visualize(run_id="<run_id>", plot_type="stress_distribution", output="file")
visualize(run_id="<run_id>", plot_type="deflection_profile", output="file")
visualize(run_id="<run_id>", plot_type="weight_breakdown", output="file")
```

### 7. Export Provenance

```
export_session_graph(output_path="aerostruct_analysis_provenance.json")
```

## Report Format

| Metric | Value |
|---|---|
| CL | ... |
| CD | ... |
| L/D | ... |
| Structural mass (kg) | ... |
| Fuel burn (kg) | ... |
| Failure metric | ... (safe/failed) |
| Max deflection | ... |
| L_equals_W residual | ... |

## Important Notes

- **`load_factor` scales the trim weight** (L = load_factor × W), not the aerodynamic loads. To increase structural loads for a maneuver case, increase both `load_factor` and `alpha`.
- **Failure threshold**: `failure > 1.0` = structural failure. Values between 0 and 1 are feasible but with reduced safety margin.
- See `shared/validation-checklist.md` for full post-analysis checks.
