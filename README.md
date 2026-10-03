# perception-cpp: a showcase

A write-up of my work on the C++ perception stack of **IIT Bombay Racing's** Formula Student driverless car. Perception turns raw **camera and LiDAR** data into a list of cones (**distance, direction, colour**) that SLAM and the planner consume.

> **Note:** this repository contains documentation and diagrams only. The code lives in the team's private repository and is not reproduced here. The figures are schematic illustrations I drew to explain the ideas. They are not logged data, and this repo contains no measured benchmarks.

![Three pipelines](assets/triptych.png)

## What this is

The stack began as a Python package. Over 2026 it was rebuilt as a C++20 / CUDA / TensorRT ROS 2 node that keeps the **same output message**, so SLAM and planning needed no changes. I did most of the C++ rewrite and its tuning. I authored about 35 commits in the package between January and June 2026.

One node runs one of **five modes**, chosen by two parameters (platform and pipeline):

| Mode | Camera L | Camera R | LiDAR | Fusion |
|---|:-:|:-:|:-:|:-:|
| LiDAR only | | | yes | |
| Mono | yes | | | |
| Dual mono | yes | yes | | |
| Fusion | yes | | yes | yes |
| Dual fusion | yes | yes | yes | yes |

![What the car senses](assets/car_senses.png)

## Contents

| Doc | Topic |
|---|---|
| [Python to C++](docs/01-python-to-cpp.md) | What the old pipeline did and why it was rebuilt |
| [Camera branch](docs/02-camera-branch.md) | Detection, IPM depth, pitch correction |
| [LiDAR branch](docs/03-lidar-branch.md) | ROI, ground removal, clustering, cone checks, colour |
| [Fusion](docs/04-fusion.md) | Uncertainty-aware matching and blending |
| [Tooling and what's next](docs/05-tooling-and-next.md) | Tuning tools and the PointPillars prototype |

## My contributions

From the commit history of the package, in rough order:

- **Jan 2026:** first working dual-fusion run on a rosbag; real-life mono depth fixes; LiDAR driver and clustering parameter tuning; package split from the Python code.
- **Feb 2026:** new camera-to-LiDAR association with uncertainty-aware matching, a fusion configuration override, and a tuning workflow for its parameters.
- **Mar-Apr 2026:** configuration moved out of headers into YAML; a static log dashboard; a ground-removal alternative explored; configs for the D1 platform tuned on recorded bags.
- **May-Jun 2026:** LiDAR-only fallback, thread and core handling in clustering, inverse-perspective-mapping (IPM) depth with dynamic pitch, and a gyroscope-based pitch tool.

## Honest status

- There are no saved latency or accuracy benchmarks for the C++ pipeline yet. Measuring them is the first next step.
- The learned LiDAR detector (PointPillars) is a prototype on recorded data, not deployed.
- Some configuration flags (for example IPM and vision pitch correction) are toggled per platform and run.

## Stack

C++20, CUDA, TensorRT, ROS 2, PCL, Open3D, OpenCV, Eigen, ONNX Runtime, YAML configuration, Python tooling for tuning.

## Contact

Ishaan Mondal, IIT Bombay (Mechanical Engineering, class of 2028). GitHub: [@IshaanM05](https://github.com/IshaanM05).
