# Common Constraint Sets

Standard constraint configurations by optimization type. Copy and adapt these — don't use them blindly.

## Aero-Only Optimization

```json
[{"name": "CL", "equals": 0.5}]
```

The CL target forces the optimizer to maintain lift while minimizing drag. Adjust the target to match the design operating point.

## Single-Point Aerostructural Optimization

```json
[
  {"name": "L_equals_W", "equals": 0.0},
  {"name": "failure", "upper": 0.0},
  {"name": "thickness_intersects", "upper": 0.0}
]
```

- **L_equals_W = 0**: Trim constraint (lift equals weight)
- **failure <= 0**: Structural safety. `failure > 1.0` means structural failure; the optimizer keeps it below 0 for margin
- **thickness_intersects <= 0**: Prevents upper and lower spar caps from intersecting (wingbox only; harmless for tube)

## Multipoint Aerostructural Optimization

```json
[
  {"name": "L_equals_W", "equals": 0.0, "point": 0},
  {"name": "L_equals_W", "equals": 0.0, "point": 1},
  {"name": "failure", "upper": 0.0, "point": 1},
  {"name": "fuel_vol_delta", "lower": 0.0},
  {"name": "fuel_diff", "equals": 0.0}
]
```

- Constraints with `"point": N` target a specific flight point (0=cruise, 1=maneuver)
- **failure at point 1** (maneuver): sizes the structure at the critical load condition
- **fuel_vol_delta >= 0**: wing volume must accommodate the fuel
- **fuel_diff = 0**: fuel mass consistency between cruise and maneuver points

## Available Constraint Names

| Name | Type | Applicable To |
|---|---|---|
| `CL` | aero | aero, aerostruct |
| `CD` | aero | aero, aerostruct |
| `CM` | aero | aero, aerostruct |
| `S_ref` | aero | aero, aerostruct |
| `failure` | structural | aerostruct only |
| `thickness_intersects` | structural | aerostruct only |
| `L_equals_W` | trim | aerostruct only |
| `fuel_vol_delta` | fuel | multipoint only |
| `fuel_diff` | fuel | multipoint only |
