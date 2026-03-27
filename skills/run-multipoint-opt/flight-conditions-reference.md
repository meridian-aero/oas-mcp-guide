# Flight Conditions Reference for Multipoint Optimization

## Standard Two-Point Configuration (Transport Aircraft)

### Point 0: Cruise
```json
{
  "velocity": 248.136,
  "Mach_number": 0.84,
  "density": 0.38,
  "reynolds_number": 1e6,
  "speed_of_sound": 295.4,
  "load_factor": 1.0
}
```

### Point 1: Maneuver (2.5g)
```json
{
  "velocity": 189.0,
  "Mach_number": 0.64,
  "density": 0.38,
  "reynolds_number": 1e6,
  "speed_of_sound": 295.4,
  "load_factor": 2.5
}
```

## Required Fields in Each Flight Point

Every dict in `flight_points` must contain **all six** fields:
- `velocity` (m/s)
- `Mach_number`
- `density` (kg/m3)
- `reynolds_number` (1/m)
- `speed_of_sound` (m/s)
- `load_factor`

Missing any field raises a validation error.

## Why Maneuver Mach is Lower

The 2.5g maneuver condition uses a lower Mach (0.64 vs 0.84) because:
- At higher load factor, the aircraft flies at a higher CL (higher alpha)
- To avoid buffet/stall, the maneuver is flown at a lower speed
- FAR 25.337 specifies the maneuver speed (VA) is lower than cruise speed

## Three-Point Configuration (Advanced)

For studies requiring a gust load condition:

```json
[
  {"velocity": 248.136, "Mach_number": 0.84, "density": 0.38, "reynolds_number": 1e6, "speed_of_sound": 295.4, "load_factor": 1.0},
  {"velocity": 189.0, "Mach_number": 0.64, "density": 0.38, "reynolds_number": 1e6, "speed_of_sound": 295.4, "load_factor": 2.5},
  {"velocity": 248.136, "Mach_number": 0.84, "density": 0.38, "reynolds_number": 1e6, "speed_of_sound": 295.4, "load_factor": 1.3}
]
```

Point 0 = cruise, Point 1 = maneuver, Point 2 = gust.

## Narrow-Body (A320-Class) Variant

```json
[
  {"velocity": 230.0, "Mach_number": 0.78, "density": 0.38, "reynolds_number": 1e6, "speed_of_sound": 295.4, "load_factor": 1.0},
  {"velocity": 175.0, "Mach_number": 0.59, "density": 0.38, "reynolds_number": 1e6, "speed_of_sound": 295.4, "load_factor": 2.5}
]
```

## Additional Multipoint Parameters

| Parameter | Default | Notes |
|---|---|---|
| `W0_without_point_masses` | 143000.0 kg | Empty weight + reserve fuel (not total W0) |
| `point_masses` | None | Engine masses, e.g. `[[10000]]` |
| `point_mass_locations` | None | Engine positions, e.g. `[[25, -10, 0]]` |
