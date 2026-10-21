# Pseudocode

## Notation

| Symbol | Meaning |
|---|---|
| q, q̇ | joint angles and velocities |
| J | Jacobian |
| Λ | operational-space inertia |
| g(q) | gravity torque |
| b | joint viscous damping |
| K, ζ | task stiffness and damping ratio |
| K_e | surface contact stiffness |
| M_v, B_v | virtual mass and damping |
| F_d | desired force, 5 N |
| series(a, b) | stiffness of two springs in series: product over sum |

## Trajectory

```text
PATH(word, shape)
    strokes ← letters of word as polylines
    curve ← samples of shape placed below the word
    fit everything inside the reachable area of the table
    return strokes, curve

TIMING(path)
    duration ← length / writing speed
    progress ← minimum-jerk profile from 0 to length over duration
    return position and velocity along path at every 1 ms
```

## Impedance torque

```text
TORQUE(x_d, ẋ_d, K, ζ)
    e ← pose error between tool and x_d
    ė ← J q̇ minus ẋ_d
    D ← 2 ζ √(K Λ)
    F ← −K e − D ė
    τ ← Jᵀ F, then add g(q) and b q̇
    return τ
```

## Contact stiffness

```text
CALIBRATE
    press the table and hold
    K_e ← steady force / penetration depth
    return K_e
```

## Force control along the surface normal

```text
IMPEDANCE_SETPOINT
    δ ← F_d / series(K_z, K_e)
    return surface height minus δ

ADMITTANCE_SETPOINT(F)
    ẍc ← ((F − F_d) − B_v ẋc) / M_v
    increase ẋc by ẍc Δt
    increase xc by ẋc Δt
    return surface height minus offset, raised by xc
```

## Writing

```text
WRITE(controller)
    K_e ← CALIBRATE
    for each stroke in PATH
        lift, move above the stroke start, lower until contact
        for each sample of TIMING(stroke)
            z_d ← setpoint of controller from the last measured force
            τ ← TORQUE((x_d, y_d, z_d), velocity, K, ζ)
            apply τ, step 1 ms, measure force
    return RMSE of force against F_d, path error, mean squared jerk
```

## Experiments

```text
UNEVEN
    add 3 mm ridges every 6 cm, run WRITE for both controllers

ZETA
    for ζ in 0.3 to 5
        run WRITE for both controllers
        record force RMSE and touchdown overshoot

FREE_SPACE
    for joint friction in 0 to 2 N·m
        trace a circle in the air with impedance and with a stiff position loop
        record mean path error

CURSIVE
    letters from a single-stroke script font, split at sharp turns, smoothed by corner cutting
    run WRITE for Hope and राम
```
