# Configuration and platforms

The pipeline is driven by YAML configuration, not constants in code. Moving parameters out of headers into configuration was one of the larger refactors, because it let the same binary run on different vehicles and simulators.

## Platforms

One configuration block per platform. Each carries its own topics, camera calibration, sensor mounting and tuning.

| Platform | Purpose |
|---|---|
| `bot` | Small prototype vehicle |
| `d1` | The competition car |
| `ads_dv` | Autonomous-driving test vehicle |
| `eufs` | EUFS simulator |
| `carmaker` | CarMaker simulator |

## What is tunable

**Camera side**
- Camera topics, calibration and mounting (height, yaw, offsets from the car origin).
- Detector model and confidence cut-offs.
- Box validation limits (shape, size).
- IPM options and pitch handling.
- Merge distance for cones seen by both cameras.

**LiDAR side**
- ROI box and the exclusion zone around the car body.
- Ground removal method and settings.
- Clustering settings.
- Which cone checks are enabled, and their limits.
- Colour classifier model and confidence.
- A set of relaxed overrides used when the LiDAR runs inside fusion.

**Fusion**
- Association method (uncertainty-aware or a simple nearest match, for comparison).
- Sensor uncertainty model for camera and LiDAR.
- Gate threshold and search radius.
- Whether a camera-only cone is kept when no LiDAR match exists.

## Why per-platform

Sensor height, mounting angle and LiDAR model differ between vehicles, and so does the noise. Settings that work on one platform are wrong on another. Tuning each platform on its own recorded data, rather than sharing one global setting, was necessary for the car and the simulators to behave sensibly.

## Tuning workflow

Parameters are tuned on **recorded bags**, then confirmed on the vehicle. For fusion, an offline tool sweeps settings against LiDAR-derived ground truth (see [Fusion](04-fusion.md)).
