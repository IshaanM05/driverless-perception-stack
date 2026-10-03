# Camera branch

```
Image -> resize (keep shape) -> YOLO on TensorRT (both cameras together)
      -> discard implausible boxes -> IPM distance -> car frame -> merge left + right
```

## Detection

A YOLO model runs through TensorRT. Left and right frames are processed as one batch. Boxes with implausible shapes or sizes are dropped before depth is estimated.

## IPM: inverse perspective mapping

Treat the road as a flat plane. The camera ray through the **bottom of the cone's box** (where it touches the ground) meets that plane at the cone's position.

![IPM geometry](../assets/ipm.png)

Distance is camera height divided by the tangent of the total angle below the horizon (camera pitch plus the ray's angle from the optical axis). It does not depend on how big the cone looks. If the car's pitch is ignored, the same pixel lands on a different ground point (grey dashed ray in the figure).

Assumptions to be honest about: flat ground and a known camera height.

## Pitch correction

The car pitches under braking and acceleration. For each frame, the expected size of a cone (from geometry and the current pitch) is compared with its measured box. The median disagreement over visible cones is limited and smoothed, and the updated pitch feeds the next IPM computation.

![Depth sensitivity of the old method](../assets/depth_curve.png)

*Why size-based depth was weak: the same pixel error costs far more distance at range.*

## Box validation

Before any distance is computed, each detection must look like a cone:
- The box shape must be taller than wide, like a cone.
- The box must be neither tiny (noise) nor huge (a false positive).
- Boxes cut off by the image edge are treated with care, because a clipped box misplaces the ground contact.

Rejecting bad boxes here is cheaper and safer than trying to correct a bad range later.

## Distance, direction and the car's frame

1. **Direction** comes straight from the cone's horizontal position in the image and the camera's focal length.
2. **Range** is the ground distance from IPM, corrected for how far off the optical axis the cone sits.
3. The camera's own **yaw** is added, because the cameras are angled outward.
4. The result is shifted by the camera's **mounting offsets** to the car's origin, then expressed again as distance and direction.

Cones that fail IPM (for example a ray that points above the horizon, or a non-finite result) are dropped rather than clamped to a guess.

## Two cameras

In dual modes the left and right results are concatenated. Cones that land close to each other are treated as the same physical cone seen twice, and one copy is kept. Cones seen by only one camera are kept as they are.

## Camera-only limits

Colour is dependable, but distance still comes from geometry that assumes flat ground and a known camera height. That is why fusion with LiDAR exists: it fixes distance where geometry is weakest.

## Output

Each cone leaves this branch as distance, direction and colour in the car's frame.
