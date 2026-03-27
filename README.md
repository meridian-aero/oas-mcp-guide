# oas-mcp-guide

A Claude Code plugin providing guided workflows for OpenAeroStruct aerostructural analysis and optimization via MCP tools. The plugin bundles the connection to the hosted OAS MCP server — no local installation of OpenAeroStruct required.

## Installation

```
/plugin install oas-mcp-guide
```

Or for local development:

```bash
claude --plugin-dir ./oas-mcp-guide
```

Installing the plugin automatically configures the MCP server connection to `https://mcp.lakesideai.dev/mcp`. The first time you use an OAS tool, your browser will open for login.

## Getting Started

### Credentials

You need an account on the OAS MCP server. Log in at: https://auth.lakesideai.dev

### Claude Code (CLI)

1. Install the plugin (see above)
2. Start a Claude Code session
3. The first time you call an OAS tool, your browser opens for login
4. After login, all 24 OAS tools are available automatically

### claude.ai (web)

1. Go to https://claude.ai → Settings → Integrations → Add MCP Server
2. Enter URL: `https://mcp.lakesideai.dev/mcp`
3. Sign in when redirected

### Viewing Dashboards and Plots

After running an analysis, you get a `run_id`. View results at:

- **Dashboard**: `https://mcp.lakesideai.dev/dashboard?run_id=<run_id>`
- **Provenance**: `https://mcp.lakesideai.dev/viewer?session_id=<session_id>`

You'll be prompted to log in with the same credentials. You can only see your own results.

## Skills

### Core Analysis
| Skill | Description |
|---|---|
| `define-geometry` | Create wing surfaces with validated parameters |
| `run-aero-analysis` | Single-point VLM aerodynamic analysis |
| `compute-drag-polar` | CL-CD sweep to find best L/D |
| `stability-assessment` | Static stability derivatives and margin |
| `run-aerostruct-analysis` | Coupled aero+structural analysis |

### Optimization
| Skill | Description |
|---|---|
| `run-aero-opt` | Minimize drag at fixed lift (aero-only) |
| `run-aerostruct-opt` | Minimize fuel burn or structural mass |
| `run-multipoint-opt` | Cruise + maneuver multipoint sizing |

### Trade Studies
| Skill | Description |
|---|---|
| `parameter-sweep` | Vary any parameter and track metrics |
| `mesh-convergence-study` | Find appropriate mesh density |
| `material-trade-study` | Compare Al vs Ti vs composite |
| `architecture-comparison` | TBW vs conventional vs HAR vs BWB |

### Output & Integration
| Skill | Description |
|---|---|
| `configuration-comparison` | Side-by-side run comparison |
| `physical-analysis-report` | Formal technical report |
| `export-for-integration` | Export for Aviary, OpenConcept, AeroSandbox, SUAVE, RCAIDE |
| `workflow-builder` | Interactive study design and planning |

## Shared Resources

The `shared/` directory contains reference material used across all skills:
- `provenance-protocol.md` — decision logging workflow
- `standard-flight-conditions.md` — flight conditions by aircraft class
- `material-database.md` — material properties for structural analysis
- `common-constraints.md` — standard constraint sets
- `scaling-guide.md` — objective and DV scaling for optimization
- `validation-checklist.md` — post-analysis sanity checks
- `known-issues.md` — gotchas and workarounds

## Key Constraints

- `num_y` must be **odd** (3, 5, 7, 9, ...)
- Structural analysis requires `fem_model_type` + material properties
- `failure > 1.0` = structural failure (not > 0)
- `load_factor` scales trim weight, not aerodynamic loads
- All `*_cp` arrays are root-to-tip ordered
