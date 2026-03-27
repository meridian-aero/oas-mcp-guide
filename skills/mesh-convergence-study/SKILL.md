---
name: mesh-convergence-study
description: >
  Run a mesh convergence (grid independence) study to determine the right num_y
  for an analysis. Use when the user wants to validate mesh quality, check if
  results are mesh-independent, or find the appropriate mesh density. Triggers
  on: "mesh convergence", "grid independence", "is num_y=7 enough",
  "mesh sensitivity", "how fine should my mesh be", "grid study".
argument-hint: "[wing_type] [analysis_type]  e.g. 'CRM aero convergence study'"
context: fork
---

# Mesh Convergence Study

Systematically increase mesh density to find where results stop changing, then recommend an appropriate `num_y`.

## Why This Matters

VLM results are mesh-dependent. A too-coarse mesh gives inaccurate lift distribution; a too-fine mesh wastes computation time. A convergence study finds the sweet spot.

## Workflow

### 1. Start Session

```
start_session(notes="Mesh convergence study: <wing_type> <analysis_type>")
```

### 2. Define Sweep

Standard num_y sweep: `[3, 5, 7, 9, 11, 13, 15, 21]` (all odd, as required by OAS).

For aerostruct, you may want to start at 5 (3 is very coarse for structural results).

### 3. Execute Sweep

For each num_y value:

```
reset()  # Required: clears cached OpenMDAO problem

create_surface(
    name="wing", wing_type="<type>", num_x=2, num_y=<current_num_y>,
    symmetry=True, with_viscous=True, CD0=0.015
    # Add fem_model_type and material props if aerostruct
)

run_aero_analysis(surfaces=["wing"], ...)
# or run_aerostruct_analysis(surfaces=["wing"], ...)

log_decision(
    decision_type="result_interpretation",
    reasoning="num_y=<N>: CL=<>, CD=<>, L/D=<>",
    selected_action="continue to next mesh density",
    prior_call_id="<_provenance.call_id>"
)
```

### 4. Build Convergence Table

| num_y | CL | CD | L/D | structural_mass | Delta CL (%) | Delta CD (%) |
|---|---|---|---|---|---|---|
| 3 | ... | ... | ... | ... | — | — |
| 5 | ... | ... | ... | ... | ... | ... |
| 7 | ... | ... | ... | ... | ... | ... |
| ... | | | | | | |

Compute percentage change from the previous mesh: `Delta = |value_N - value_{N-1}| / |value_{N-1}| × 100%`.

### 5. Determine Convergence

The mesh is converged when the change in key metrics (CL, CD) between successive refinements is below a threshold:
- **< 1%**: Acceptable for screening studies
- **< 0.5%**: Good for production analysis
- **< 0.1%**: High-fidelity (rarely needed for VLM)

```
log_decision(
    decision_type="mesh_resolution",
    reasoning="CD converges to within <X>% at num_y=<N>. Further refinement changes CD by only <Y>%.",
    selected_action="Recommend num_y=<N> for this wing configuration"
)
```

### 6. Export and Report

```
export_session_graph(output_path="mesh_convergence_provenance.json")
```

Return the convergence table and recommendation to the main conversation.

## Typical Results

For a CRM transport wing with aero-only analysis:
- num_y=5: ~2-5% error in CD relative to fine mesh
- num_y=7: ~1-2% error — good for screening
- num_y=11: <0.5% error — good for production
- num_y=15+: diminishing returns for VLM

Aerostructural results (structural mass) typically converge more slowly — consider using num_y=13+ for structural sizing.
