---
name: stability-assessment
description: >
  Assess static longitudinal stability of a wing configuration.
  Computes CL_alpha, CM_alpha, static margin, and neutral point location.
  Use when the user asks about stability, static margin, trim, neutral point,
  or whether a wing/aircraft is stable. Triggers on: "is it stable",
  "static margin", "stability derivatives", "neutral point", "CL_alpha",
  "trim", "longitudinal stability", "pitch stability".
argument-hint: "[cg_location]  e.g. 'check stability with CG at 25% MAC'"
---

# Static Stability Assessment

Evaluate the longitudinal stability characteristics of a wing configuration.

## Workflow

### 1. Start Session

```
start_session(notes="Stability assessment: <wing_type>")
```

### 2. Create Surface (if needed)

Use the user's geometry, or create a default:

```
create_surface(
    name="wing", wing_type="CRM", num_x=2, num_y=7,
    symmetry=True, with_viscous=True, CD0=0.015
)
```

### 3. Baseline Aero Analysis

Run a single-point analysis at the design alpha to get baseline CL and CM:

```
run_aero_analysis(
    surfaces=["wing"], alpha=5.0,
    velocity=248.136, Mach_number=0.84, density=0.38, reynolds_number=1e6
)
```

### 4. Compute Stability Derivatives

```
compute_stability_derivatives(
    surfaces=["wing"], alpha=<design_alpha>,
    velocity=248.136, Mach_number=0.84, density=0.38, reynolds_number=1e6,
    cg=<cg_location>
)
```

The `cg` parameter is the CG location as an x-coordinate in the mesh frame. For a CRM wing, 25% MAC is approximately x ≈ 5–6 m from the nose. If the user doesn't specify CG, estimate 25% of the mean aerodynamic chord.

### 5. Interpret Results

| Derivative | Stable Config | Meaning |
|---|---|---|
| CL_alpha | > 0 | Lift increases with alpha (normal) |
| CM_alpha | < 0 | Nose-down pitching moment with alpha increase (stable) |
| Static margin | > 0 | CG ahead of neutral point (stable) |

Report:
- **CL_alpha**: lift curve slope (typical: 4–6 per radian for finite wings)
- **CM_alpha**: pitch stiffness (must be negative for stability)
- **Static margin**: percentage of MAC; 5–15% is typical for transport aircraft
- **Neutral point**: x-location where CM_alpha = 0

```
log_decision(
    decision_type="result_interpretation",
    reasoning="CL_alpha=<val>, CM_alpha=<val>, static margin=<val>% MAC — <stable/unstable>",
    selected_action="<recommendation: CG within limits / needs tail / adjust sweep>",
    prior_call_id="<_provenance.call_id>"
)
```

### 6. Export Provenance

```
export_session_graph(output_path="stability_provenance.json")
```

## Stability Guidance

- **CM_alpha < 0**: Statically stable in pitch. The wing produces a restoring moment.
- **CM_alpha > 0**: Statically unstable. Needs a tail, active control, or CG shift.
- **CM_alpha ≈ 0**: Neutrally stable. CG is at the neutral point.

A wing-only configuration (no tail) is inherently limited in pitch stability. Note this when reporting results.
