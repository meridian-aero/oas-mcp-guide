---
name: export-for-integration
description: >
  Export OAS results in formats suitable for downstream tools: Aviary (mission
  analysis), OpenConcept (thermal/propulsion), AeroSandbox (higher-fidelity
  aero), SUAVE (vehicle design), or RCAIDE. Use when the user wants to pass
  OAS results to another tool, create an integration interface, or export a
  parameterized model. Triggers on: "export for Aviary", "OpenConcept integration",
  "pass to AeroSandbox", "SUAVE input", "export results", "integration output",
  "downstream tool", "multi-tool workflow".
argument-hint: "[target_tool] [run_id]  e.g. 'export latest for Aviary'"
---

# Export for Integration

Package OAS results into formats that downstream tools can consume. This is the bridge between OAS and other analysis frameworks.

## Workflow

### 1. Retrieve Run Data

```
get_artifact(run_id="<id>")
get_detailed_results(run_id="<id>", detail_level="full")
```

### 2. Select Target Format

Based on `$ARGUMENTS` or ask the user. For format details, see [integration-schemas.md](integration-schemas.md).

### 3. Generate Export

Extract relevant data from the OAS results and format according to the target tool's schema. Write the export file(s) to disk.

### 4. Document the Export

Log what was exported, for which tool, and any assumptions or limitations:

```
log_decision(
    decision_type="result_interpretation",
    reasoning="Exported <metrics> from run <id> for <tool>. Assumptions: <list>",
    selected_action="Export file written to <path>"
)
```

## Supported Target Tools

### Aviary (NASA Mission Analysis)

OAS provides wing-level data that feeds into Aviary's aircraft-level mission optimization:
- Wing geometry (span, AR, taper, sweep, t/c distribution)
- Drag polars as CL-CD tables
- Structural mass for wing weight estimation
- Export format: CSV tables + JSON metadata

### OpenConcept (Thermal/Propulsion/Mission)

OAS provides aerodynamic lookup tables:
- CL vs alpha table
- CD vs CL table (drag polar)
- Structural mass
- Export format: NumPy-loadable CSV

### AeroSandbox (Higher-Fidelity Aero)

OAS provides the low-fidelity baseline for refinement:
- Mesh coordinates
- Pressure coefficient (Cp) distribution
- Wing geometry parameters
- Export format: JSON with arrays

### SUAVE (Vehicle Design)

OAS provides vehicle geometry + performance:
- Wing planform geometry
- Aerodynamic coefficients at design point
- Structural mass breakdown
- Export format: SUAVE-compatible Python dict (JSON)

### RCAIDE

- Standardized JSON with geometry + performance
- Export format: JSON per RCAIDE import schema

### Generic CSV

For any tool not listed above:
- Tabular results with headers
- One row per analysis point (useful for drag polar data)

## Standardized Export Envelope

All exports include this metadata wrapper:

```json
{
  "export_version": "1.0",
  "source": "OpenAeroStruct",
  "timestamp": "<ISO8601>",
  "run_id": "<id>",
  "geometry": { ... },
  "aerodynamics": { ... },
  "structures": { ... },
  "mission": { ... }
}
```
