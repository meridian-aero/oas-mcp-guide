# Standard Flight Conditions

Use these as defaults when the user doesn't specify flight conditions. All values are ISA-based.

## Cruise Conditions by Aircraft Class

| Aircraft Class | Mach | Altitude (ft) | Density (kg/m3) | Velocity (m/s) | Re (1/m) | Speed of Sound (m/s) |
|---|---|---|---|---|---|---|
| UAV / small UAS | 0.3 | 5,000 | 1.05 | 100 | 3e5 | 334 |
| Regional turboprop | 0.45 | 25,000 | 0.55 | 143 | 5e5 | 318 |
| Narrow-body (A320-class) | 0.78 | 35,000 | 0.38 | 230 | 1e6 | 295 |
| Wide-body (787-class) | 0.84 | 35,000 | 0.38 | 248 | 1e6 | 295 |

## OAS Default Cruise (Wide-Body)

```
velocity=248.136, Mach_number=0.84, density=0.38, reynolds_number=1e6, speed_of_sound=295.4
```

## Maneuver Conditions

| Condition | Mach | Density | Velocity | Load Factor | Notes |
|---|---|---|---|---|---|
| Standard 2.5g maneuver | 0.64 | 0.38 | 189 | 2.5 | FAR 25.337 limit load |
| Gust (1.3g) | 0.84 | 0.38 | 248 | 1.3 | Typical cruise gust encounter |

## Multipoint `flight_points` Template

```json
[
  {
    "velocity": 248.136, "Mach_number": 0.84, "density": 0.38,
    "reynolds_number": 1e6, "speed_of_sound": 295.4, "load_factor": 1.0
  },
  {
    "velocity": 189.0, "Mach_number": 0.64, "density": 0.38,
    "reynolds_number": 1e6, "speed_of_sound": 295.4, "load_factor": 2.5
  }
]
```

Point 0 = cruise, Point 1 = maneuver (convention used by OAS multipoint tools).

## Incompressible (Low-Speed) Conditions

For low-speed / incompressible analysis set `Mach_number=0` and `with_wave=False`. OAS disables compressibility corrections at Mach=0.
