---
name: physical-analysis-report
description: >
  Generate a comprehensive analysis report from one or more OAS runs.
  Use when the user wants a formal report, summary document, or detailed
  write-up of their analysis results. Triggers on: "write a report",
  "summary of results", "formal analysis report", "document the study",
  "generate report", "write up the analysis".
argument-hint: "[run_ids or 'all']  e.g. 'report on last 3 runs'"
---

# Physical Analysis Report

Generate a structured technical report from OAS analysis results.

## Workflow

### 1. Identify Runs

Accept:
- Specific run_ids from `$ARGUMENTS`
- "last N runs" → call `list_artifacts()` and take the N most recent
- "all from session" → call `list_artifacts()` filtered by session

### 2. Retrieve Data

For each run:
```
get_artifact(run_id="<id>")
get_detailed_results(run_id="<id>", detail_level="standard")
```

### 3. Generate Report Sections

#### Executive Summary
2-3 sentences: what was analyzed, key finding, recommendation.

#### Configuration
| Parameter | Value |
|---|---|
| Wing type | CRM / rect / uCRM_based |
| Span (m) | ... |
| num_y | ... |
| FEM model | tube / wingbox / none |
| Material | aluminum / titanium / composite |

#### Flight Conditions
| Parameter | Value |
|---|---|
| Mach | ... |
| Altitude proxy (density) | ... |
| Velocity (m/s) | ... |
| Alpha (deg) | ... |
| Load factor | ... |

#### Results

**Aerodynamic Performance:**

| Metric | Value |
|---|---|
| CL | ... |
| CD | ... |
| L/D | ... |
| CM | ... |
| Drag breakdown (CDi/CDv/CDw) | ... |

**Structural Performance** (if aerostruct):

| Metric | Value |
|---|---|
| Structural mass (kg) | ... |
| Fuel burn (kg) | ... |
| Failure metric | ... |
| Max deflection | ... |

**Optimization Results** (if optimization):

| Metric | Value |
|---|---|
| Converged | Yes/No |
| Iterations | ... |
| Objective improvement | ...% |
| DV changes | ... |

#### Visualizations

Generate key plots:
```
visualize(run_id="<id>", plot_type="lift_distribution", output="file")
visualize(run_id="<id>", plot_type="drag_polar", output="file")       # if drag polar run
visualize(run_id="<id>", plot_type="stress_distribution", output="file")  # if aerostruct
visualize(run_id="<id>", plot_type="opt_history", output="file")       # if optimization
```

#### Validation Status

Report the `validation.passed` status and any findings/warnings.

#### Provenance

Reference the session_id and exported provenance graph file.

### 4. Export

```
export_session_graph(output_path="analysis_report_provenance.json")
```
