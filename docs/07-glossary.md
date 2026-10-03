# Glossary

| Term | Meaning |
|---|---|
| **Cone list** | The output: distance, direction and colour of each cone |
| **Bearing / direction** | Angle of a cone relative to the car's forward axis |
| **ROI** | Region of interest: the part of space we keep (the corridor ahead) |
| **YOLO** | A real-time object detector that draws boxes around cones |
| **TensorRT** | NVIDIA's runtime for running neural networks fast on the GPU |
| **IPM** | Inverse perspective mapping: intersect the camera ray with the road plane to get distance |
| **Pitch** | The car's nose-up or nose-down tilt |
| **Ground contact** | The point where a cone meets the road; the bottom of its box |
| **Ground removal** | Fitting and removing the road surface from a point cloud |
| **Clustering** | Grouping nearby points into objects |
| **PCA** | Principal component analysis: finds the main axes of a cluster to test its shape |
| **Reflectivity / intensity** | How strongly a surface returns the LiDAR pulse; cones have a reflective band |
| **Mahalanobis distance** | Distance measured in units of uncertainty rather than metres |
| **Chi-square gate** | A statistical test deciding whether a candidate is close enough to count as a match |
| **Kalman-style blend** | Weighted average of two estimates, leaning on the more certain one |
| **Mono** | Camera-only pipeline |
| **Fusion** | Camera and LiDAR combined |
| **Pose compensation** | Correcting cone positions for the car's motion between sensing and publishing |
| **Rosbag** | A recording of sensor data that can be replayed offline |
| **PointPillars** | A learned detector that turns a point cloud into a 2D image of columns, then finds objects |
| **SLAM** | Simultaneous localisation and mapping: builds the track map and the car's pose |
