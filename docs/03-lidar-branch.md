# LiDAR branch

![LiDAR stages](../assets/lidar0.png)

The five stages below are shown on a synthetic scene I generated to illustrate each step. It is not logged data. Rings are scan lines on the road, and cones rise above them.

| # | Stage | What it does |
|---|---|---|
| 1 | Region of interest | Keep the corridor in front of the car |
| 2 | Ground removal | Fit the road surface and remove it |
| 3 | Clustering | Nearby points become one cone candidate |
| 4 | Cone checks | Is it really a cone? |
| 5 | Colour | Blue or yellow from reflectivity |

<p>
<img src="../assets/lidar1.png" width="19%"> <img src="../assets/lidar2.png" width="19%"> <img src="../assets/lidar3.png" width="19%"> <img src="../assets/lidar4.png" width="19%">
</p>

*Panels 2 to 5: ground removed, clustered, checked, coloured.*

## Cone checks

- **Base restoration:** ground removal can eat the cone's foot, so those points are given back.
- **Shape (PCA):** the cluster's principal axes should say tall and thin.
- **Point count:** a cone at that range should have a sensible number of points.

These run in LiDAR-only mode. In fusion mode the camera supplies the cone hypothesis.

## Colour from reflectivity

Cones have a reflective band, so reflectivity against height differs by colour. Two independent opinions are combined: a small learned classifier and a hand-built curve rule. **Both must agree**, otherwise the colour stays unknown rather than guessed.

## Ground removal in more detail

The road is found by repeatedly fitting a plane to the points that remain. Each fitted plane is judged:
- A **flat plane** is ground and is removed.
- A **steep plane far from the car** is a wall or barrier and is removed too.
- A **steep plane close to the car** might be a cone, so the search stops there rather than deleting it.

The removed ground points are kept, not discarded, because the cone checks below use them to restore a cone's base.

## Clustering in more detail

Points that sit close together become one cluster. The clustering radius and minimum point count are tuned per platform, because a LiDAR with fewer beams returns sparser points on a cone. Clustering is the most CPU-heavy LiDAR stage, so I spent time on its thread and core handling to keep it from competing with the rest of the node.

## Restoring the cone base

Ground removal tends to take the lowest slice of a cone with the road. A small cylinder around the cluster's base is searched in the removed ground points and any that fall inside are given back, so the cone's measured height and shape are not understated.

## What each check protects against

| Check | Rejects |
|---|---|
| Base restoration | Under-measured cones, which would otherwise fail the height check |
| Shape (PCA) | Flat or sprawling objects that are not tall and thin |
| Point count | Clusters with too few or too many points for their range |

## Colour in more detail

Reflectivity along a cone's height forms a pattern, and blue and yellow cones give different patterns. A small learned classifier reads that pattern and a hand-built rule fits a curve to it. A colour is only accepted when both agree, and ambiguous cones stay unknown. In fusion the camera's colour is used instead.

## LiDAR-only fallback

I also added a LiDAR-only pipeline that runs without the cameras, so the car can keep producing cones if the cameras are unavailable. It is the only mode that does ground removal, the cone checks and colour all on the LiDAR side.
