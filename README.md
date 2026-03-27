# oas-mcp-guide

A Claude Code plugin providing guided workflows for OpenAeroStruct aerostructural analysis and optimization via MCP tools. The plugin bundles the connection to the hosted OAS MCP server — no local installation of OpenAeroStruct required.

## Getting Started

### Prerequisites

- An account on the OAS MCP server. Sign up / log in at: https://auth.lakesideai.dev if you don't have one.

### Option 1: Claude Code (CLI / Desktop / IDE)

1. **Clone this repo**

   ```bash
   git clone https://github.com/meridian-aero/oas-mcp-guide.git
   ```

2. **Load the plugin**

   ```bash
   claude --plugin-dir ./oas-mcp-guide
   ```

   This configures the MCP server connection automatically — no extra setup needed.

3. **Start using skills** — e.g. `/oas-mcp-guide:define-geometry`, `/oas-mcp-guide:run-aero-analysis`, etc.

4. **Log in** — the first time you call an OAS tool your browser will open for authentication. After login all 24 OAS tools are available.

### Option 2: claude.ai (Web)

Claude.ai does not support plugins directly, but you can connect the MCP server and use the underlying tools:

1. Open https://claude.ai and go to **Settings → Connectors → Add MCP Server** (click the **+** button)
2. Enter the server URL: `https://mcp.lakesideai.dev/mcp`
3. Sign in when redirected to the auth page
4. After connecting, the OAS tools appear under the MCP tools list. You can enable individual tools (skills) from the connector panel — click the connector, then toggle on the tools you need (e.g. `create_surface`, `run_aero_analysis`, `run_optimization`, etc.)

5. **(Optional) Add guided skills** — claude.ai connects the MCP tools but doesn't automatically load the plugin's guided workflow skills. To add them manually, go to **Settings → Customize → Skills → +** and paste the contents of any `skills/*/SKILL.md` file from this repo. Each skill bundles prompts, validation, and multi-step orchestration on top of the raw tools. For the full experience with all 16 skills loaded automatically, use Claude Code (Option 1).

### Viewing Dashboards and Plots

After running an analysis, ask for plots at a URL, log in to view.

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
