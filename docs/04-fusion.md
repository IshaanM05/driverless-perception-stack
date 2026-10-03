# Camera-LiDAR fusion

The camera knows **colour** and direction well but is unsure of distance. The LiDAR knows **position** precisely but has no reliable colour. Fusion keeps the best of each.

## Matching: why uncertainty, not metres

![Gate in metres vs in units of uncertainty](../assets/gate.png)

A fixed circle treats every direction the same, so it can accept a wrong candidate that is merely close. The camera's error is stretched along its viewing ray, so its gate should be too. Closeness is measured as a **Mahalanobis distance**: in units of uncertainty, not metres. A chi-square test on that distance decides whether a LiDAR cluster belongs to a camera cone.

## Blending

![Fusing two beliefs](../assets/blend.png)

Two uncertain estimates of the same quantity combine into one that is more certain than either, and it sits closer to the sharper sensor. This is the weighted, Kalman-style blend. The fused cone takes **colour from the camera and position mostly from the LiDAR**.

## How a match is decided

1. Camera cones are converted to the LiDAR's frame using the sensors' mounting offsets.
2. Each camera cone gets a **search area** sized by its own uncertainty, with a minimum so close cones are not missed.
3. LiDAR candidates inside that area are scored by Mahalanobis distance, and a chi-square test accepts or rejects them.
4. If two camera cones want the same LiDAR point, the pairing is resolved so each LiDAR point is used once.

## Uncertainty model

- **Camera:** distance error grows with range, direction error is small.
- **LiDAR:** small error in both.
- Both are converted from distance-and-direction form into the same plane so the two ellipses can be compared and combined.

## Status of each cone

| Status | Meaning |
|---|---|
| Fused | Camera and LiDAR agree; position is blended |
| Camera only | No LiDAR match; kept only if the camera-only fallback is enabled |
| Rejected | Failed the gate |
| Duplicate | Same cone reported twice; one copy removed |

Duplicates are removed by keeping the fused copy over a camera-only one.

## Design choices

- **Colour always comes from the camera.** The LiDAR is trusted for position only.
- The same code supports a plain nearest-match option, which makes it easy to compare against the uncertainty-aware version on the same recording.

## Tuning

The sensor uncertainty model and gate parameters were tuned offline:

1. Record LiDAR-derived ground truth.
2. Record camera detections of the same scene.
3. Sweep settings with cross-validation and keep the best.

The tuning tool also has a live dashboard. Its ground truth comes from the LiDAR rather than from hand annotation.
