---
name: define-geometry
description: >
  Define wing geometry for an OAS analysis without running analysis.
  Use when the user wants to create a wing, set up a surface, choose mesh
  parameters, or prepare geometry for later analysis. Also use when the user
  asks "what mesh should I use" or "set up a CRM wing". Triggers on any
  mention of creating/defining a wing surface, mesh setup, or geometry definition.
argument-hint: "[wing_type] [span] [num_y]  e.g. 'CRM wing, 58m span, num_y=13'"
---

# Define Wing Geometry

Create an OAS lifting surface with validated parameters. This prepares geometry for downstream analysis or optimization — it does not run any solver.

## Workflow

### 1. Start Provenance

Call `start_session(notes="Define <wing_type> geometry")` to begin tracking.

### 2. Gather Parameters

Parse `$ARGUMENTS` for wing_type, span, num_y, and structural model. If not provided, ask the user. For detailed guidance on wing types and parameter ranges, read [wing-types-reference.md](wing-types-reference.md).

Key parameters to determine:
- **Wing type**: `rect` (clean rectangular), `CRM` (NASA Common Research Model), `uCRM_based` (updated CRM for multipoint)
- **Mesh density**: `num_y` (spanwise nodes, must be **odd**). Start with 7 for screening, 11–13 for production.
- **Structural model**: `None` (aero-only), `tube` (simple beam), `wingbox` (realistic box beam)
- **Material**: If structural, use properties from `shared/material-database.md`

### 3. Create Surface

Call `create_surface` with the gathered parameters. Example for a CRM transport wing:

```
create_surface(
    name="wing", wing_type="CRM", num_x=2, num_y=7,
    symmetry=True, with_viscous=True, CD0=0.015
)
```

For aerostructural work, add structural parameters:

```
create_surface(
    name="wing", wing_type="CRM", num_x=2, num_y=7,
    symmetry=True, with_viscous=True, CD0=0.015,
    fem_model_type="tube",
    E=70e9, G=30e9, yield_stress=500e6, mrho=3000.0
)
```

### 4. Visualize

Call `visualize(run_id=None, plot_type="planform")` to show the planform, or `visualize(run_id=None, plot_type="mesh_3d")` for a 3D view. Use `output="file"` in CLI environments.

### 5. Log Decision

```
log_decision(
    decision_type="mesh_resolution",
    reasoning="<why this wing_type, num_y, and structural model>",
    selected_action="<chosen configuration summary>"
)
```

## Quick Parameter Defaults

| Parameter | Default | Notes |
|---|---|---|
| `num_x` | 2 | Chordwise panels; 2 is sufficient for VLM |
| `num_y` | 7 | Spanwise panels; **must be odd** |
| `symmetry` | True | Model half-wing (doubles for full) |
| `CD0` | 0.015 | Parasitic drag coefficient |
| `with_viscous` | True | Include skin-friction drag |
| `with_wave` | False | Include wave drag (enable for transonic) |
| `safety_factor` | 2.5 | Applied to yield_stress |

## Control Point Ordering

All `*_cp` arrays are **root-to-tip** ordered. The server converts internally.
