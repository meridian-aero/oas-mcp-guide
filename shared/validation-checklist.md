# Post-Analysis Validation Checklist

Check these after every analysis. The MCP server performs automated validation, but you should also sanity-check the physics.

## 1. Envelope Validation

Every tool response includes a `validation` block:

```json
{
  "validation": {
    "passed": true,
    "findings": [...]
  }
}
```

**Always check `validation.passed`** before trusting results. If `false`, read `findings` for specific issues.

## 2. Aerodynamic Sanity Checks

| Check | Expected Range | Red Flag |
|---|---|---|
| CL at positive alpha | > 0 | CL < 0 at alpha > 0 |
| CD | > 0 | CD < 0 (solver failure) |
| L/D | 5–40 for conventional wings | L/D > 50 or < 3 |
| CM | Negative for stable configs | Not a hard rule — depends on CG |

## 3. Structural Sanity Checks

| Check | Meaning | Action |
|---|---|---|
| `failure > 1.0` | **Structural failure** — stress exceeds allowable | Increase thickness or reduce load |
| `failure < 0` | Safe — margin available | OK |
| `failure ≈ 0` | Right at the allowable | Tight design; verify safety factor |
| `|L_equals_W| > 0.1` | Trim not achieved | Adjust alpha or W0 |

**Critical**: `failure > 1.0` means structural failure, NOT `failure > 0`. The failure metric is a Von Mises stress ratio minus 1: values between 0 and 1 are feasible but with reduced margin.

## 4. Optimization Checks

| Check | What to Look For |
|---|---|
| `success` field | `true` = optimizer converged; `false` = hit iteration limit or failed |
| Objective change | Should decrease monotonically (check opt_history) |
| Constraint satisfaction | All constraints within tolerance of their targets |
| DV bounds | Check if any DV hit its upper or lower bound (may indicate bounds too tight) |

## 5. Common Gotchas

- **`load_factor` scales trim weight**, not aerodynamic loads. To increase structural loads, increase alpha.
- **Stale cache**: If you change `load_factor` between runs without calling `reset`, the cached problem may not update.
- **DV name mismatch**: Incorrect DV names fail silently — the optimizer runs but doesn't change the intended variable.
- **`num_y` must be ODD**: Even values raise an error.
