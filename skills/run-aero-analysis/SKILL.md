---
name: run-aero-analysis
description: >
  Run a VLM aerodynamic analysis of a wing using OpenAeroStruct.
  Use when the user asks to analyze a wing, check lift and drag, compute CL,
  evaluate aerodynamic performance, or run a single-point analysis.
  Triggers on: "analyze wing", "what's the L/D", "compute CL",
  "aero analysis", "check drag", "evaluate this wing", "run VLM".
argument-hint: "[wing_type] [alpha] [Mach]  e.g. 'CRM at alpha=3, Mach 0.84'"
---

# Aerodynamic Analysis

Run a single-point VLM aerodynamic analysis and interpret the results.

## Workflow

### 1. Start Session

```
start_session(notes="Aero analysis: <wing_type> at alpha=<alpha>, Mach=<Mach>")
```

### 2. Create Surface

Parse `$ARGUMENTS` for wing_type, alpha, and Mach. If not specified, use sensible defaults.

```
create_surface(
    name="wing", wing_type="CRM", num_x=2, num_y=7,
    symmetry=True, with_viscous=True, CD0=0.015
)
```

Enable `with_wave=True` for transonic (Mach > 0.6) analysis. For flight conditions, consult `shared/standard-flight-conditions.md`.

### 3. Log Mesh Decision

```
log_decision(
    decision_type="mesh_resolution",
    reasoning="<why this wing_type and mesh density>",
    selected_action="wing_type=CRM, num_x=2, num_y=7"
)
```

### 4. Run Analysis

```
run_aero_analysis(
    surfaces=["wing"], alpha=5.0,
    velocity=248.136, Mach_number=0.84, density=0.38, reynolds_number=1e6
)
```

### 5. Check Results

From the response envelope:
1. **Check `validation.passed`** — if false, read `validation.findings` for issues
2. Read `summary.narrative` for a plain-English summary
3. Note `summary.flags` (e.g., `tip_loaded`, `induced_drag_dominant`)
4. Extract key metrics from `results`: CL, CD, CM, L/D

### 6. Log Interpretation

```
log_decision(
    decision_type="result_interpretation",
    reasoning="CL=<val>, CD=<val>, L/D=<val> — <what this means>",
    selected_action="<next step: drag polar, optimization, or done>",
    prior_call_id="<_provenance.call_id from the analysis>"
)
```

### 7. Visualize

Call `visualize(run_id="<run_id>", plot_type="lift_distribution")` to see the spanwise Cl distribution.

In CLI environments, use `output="file"` or set `configure_session(visualization_output="file")` first.

### 8. Export Provenance

```
export_session_graph(output_path="aero_analysis_provenance.json")
```

### 9. Report

Present a summary table:

| Metric | Value |
|---|---|
| CL | ... |
| CD | ... |
| L/D | ... |
| CM | ... |
| CDi / CD_total | ...% |
| CDv / CD_total | ...% |

Note any validation warnings or unusual flags.
