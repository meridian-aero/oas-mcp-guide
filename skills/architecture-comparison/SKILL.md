---
name: architecture-comparison
description: >
  Compare aircraft configurations: conventional tube-and-wing, truss-braced wing
  (TBW), high-aspect-ratio, or multi-surface designs. Use when the user wants to
  compare architectures, evaluate TBW vs conventional, study high AR effects,
  or set up a multi-surface analysis. Triggers on: "truss-braced wing", "TBW",
  "compare configurations", "high aspect ratio trade", "BWB", "strut-braced",
  "conventional vs TBW", "architecture study", "configuration comparison".
argument-hint: "[configs]  e.g. 'conventional vs TBW at AR 10, 13.5, 17'"
context: fork
---

# Architecture Comparison

Compare different aircraft configurations by running the same analysis across each architecture and compiling results.

## Supported Architectures

| Architecture | OAS Representation | Notes |
|---|---|---|
| Conventional tube-and-wing | Single `create_surface(wing_type="CRM")` | Baseline for most comparisons |
| High-aspect-ratio wing | Single surface with increased `span` | AR 12–18 typical for studies |
| Truss-braced wing (TBW) | Two surfaces: wing + strut | Strut as second `create_surface` with `offset` |
| Blended wing body (BWB) | Single wide-chord surface with high taper | **Limitation**: OAS has no fuselage model; wing aero only |

For pre-built templates, see [architecture-templates.md](architecture-templates.md).

## Workflow

### 1. Start Session

```
start_session(notes="Architecture comparison: <list of configs>")
```

### 2. Gather Configuration Details

Parse `$ARGUMENTS` or ask the user:
- Which architectures to compare?
- What aspect ratios? (e.g., 9, 13.5, 17 for an AR trade)
- For TBW: strut span fraction (default: 60%), strut chord, vertical offset
- Common mission: W0, cruise Mach, load factor
- Analysis type: aero or aerostruct?

### 3. Run Each Architecture

For each configuration:

```
reset()  # Clear state between configurations

# Create surface(s) for this architecture
create_surface(name="wing", ...)
# For TBW, also:
create_surface(name="strut", wing_type="rect", span=20, offset=[25, -10, -3], ...)

# Run analysis
run_aerostruct_analysis(surfaces=["wing"], ...)  # or ["wing", "strut"] for TBW

log_decision(
    decision_type="result_interpretation",
    reasoning="<architecture>: L/D=<>, struct_mass=<>, fuelburn=<>",
    selected_action="continue to next architecture",
    prior_call_id="<_provenance.call_id>"
)
```

### 4. Build Comparison Table

| Architecture | AR | L/D | struct_mass (kg) | fuelburn (kg) | failure | Notes |
|---|---|---|---|---|---|---|
| Conventional | 9.0 | ... | ... | ... | ... | Baseline |
| High-AR | 13.5 | ... | ... | ... | ... | +X% L/D, +Y% mass |
| TBW | 17.0 | ... | ... | ... | ... | Strut at 60% span |

### 5. Analyze Trade-Offs

Identify:
- Which architecture wins on each metric
- Where diminishing returns occur (especially for increasing AR)
- Structural penalties of high AR (increased bending loads → heavier structure)
- Whether TBW strut support offsets the structural penalty

```
log_decision(
    decision_type="result_interpretation",
    reasoning="<cross-architecture analysis and trade-off summary>",
    selected_action="<recommended architecture and rationale>"
)
```

### 6. Export

```
export_session_graph(output_path="architecture_comparison_provenance.json")
```

## Important Limitations

- **BWB**: OAS models BWB as a wing surface only — no fuselage aerodynamics. Use for relative comparisons, not absolute BWB performance.
- **TBW strut**: The strut is modeled as a separate lifting surface. OAS does not model strut-wing interference or strut buckling explicitly — the strut provides lift relief and structural support through the VLM coupling.
- **Multi-body**: OAS cannot model physically separate aircraft (e.g., two small airplanes). Each surface shares a single flight condition.
