# Robot Packages and Integration Contract

[Back to README](../README.md) · Status: interface design; neither G2 nor Franka is integrated in this repository yet

## 1. G2-first, not G2-only

G2 is the first full reference because we have physical hardware and useful assets/workflows in ecosystems such as Genie Sim. The platform core must not assume dual arms, fixed joint names, a fixed action dimension, a mobile base, or a fixed camera count.

The second package is simulated Franka using the same tabletop task family. Its purpose is to prove early that multi-robot extensibility is real rather than an abstraction on paper. It does not require a second physical robot in v0 and does not imply automatic policy transfer from G2 to Franka.

## 2. RobotPackage contract

| Section | Required content |
| --- | --- |
| Model / Variant | Kinematics, joints and frames, end tools, asset provenance and variants |
| Capabilities | Supported commands, sensors, task requirements, timing modes and deployment bindings |
| Simulation binding | Isaac assets, actuators, collision/contact, initial state and control configuration |
| Deployment binding | ROS 2 / SDK mapping, firmware constraints, communication and startup checks |
| Controller profiles | Joint control, IK / WBC, interpolation, limits, control period and validity interval |
| Instance calibration | Physical tools, zero offsets, cameras, latency, physical parameters and calibration version |
| Conformance tests | Interface, action, coordinate, reset, fault and timing validation |

Separate robot model, tool variant and physical instance. The public repository stores only non-sensitive schemas/examples; serial numbers, addresses, credentials and private calibration files are injected externally and never committed.

## 3. Unify action semantics, not vector length

Initial command types: `JointPositionTarget`, `EndEffectorPoseTarget`, `BaseVelocityTarget`, and `GripperCommand`. A robot package may accept only command types it explicitly declares.

Commands declare target names, units, coordinate frames, absolute/incremental semantics, timing and validity. Orientation conventions are fixed by contract and converted at boundaries; never depend on a library's implicit quaternion order.

The proposed internal convention is SI units, right-handed frames, explicitly named frames, and `xyzw` quaternions. Upstream conventions are converted explicitly and tested. This is a RoboForge protocol choice, not an assumption about Isaac or ROS defaults.

v0 prioritizes position and gripper control. Torque, impedance and complex whole-body control are explicit robot-package extensions. If a robot cannot satisfy task capability requirements, fail before execution rather than silently approximate.

## 4. Observations and policy adaptation

Observation schemas pin camera names, intrinsics/extrinsics, resolution, color/depth conventions, state fields, coordinates and timestamps. Policy adapters record crop/resize, normalization and action-chunk processing.

A policy running in the simulator process does not automatically gain access to hidden ground truth. Expert data generation may request privileged state, but that privilege must be labeled separately and excluded from evaluations that forbid it.

A `PolicyAdapter` should expose initialization, reset, infer, close and capability declaration. WebSocket, in-process, or other transport remains inside the adapter. Adding a model should not require changing a robot package.

## 5. G2 reuse boundary

Prefer to reference robot assets, joint/tool mappings, controllers, MoveIt / WBC and collection examples from [Genie Sim](https://github.com/AgibotTech/genie_sim). Before integration, verify the actual G2 arm, gripper, cameras, SDK and firmware.

Copying code is not equivalent to completed support. Record upstream commit, source path, license, modifications and comparison tests. Different Genie modules and third-party assets may carry different licenses; do not relicense an imported directory under MIT.

Where possible, use the same G2 scene and control definitions across lockstep, SIL and HIL. Do not assume independent Genie benchmark and ROS workflows have identical runtime semantics.

## 6. Conformance criteria

| Test | Pass condition |
| --- | --- |
| Joint and frame mapping | Names, order, units, direction and zero points are auditable; mismatches fail explicitly |
| Pose/action transforms | Forward/inverse transforms agree within tolerance; absolute/incremental semantics never mix |
| Controller | Limits, interpolation, cadence, expiry and fallback match the profile |
| Simulation asset | References resolve; collision/articulation work; rest and manipulation tests pass |
| Reset | Controller state and action buffers are isolated; late actions cannot cross epochs |
| Sensors | Actual inputs match schema, calibration and timing records |
| Deployment | Supported SIL/HIL configurations and limits have test evidence |
| Portability | Adding Franka requires only a package/binding/tests, not core-loop changes |

A task may share semantics across embodiments, but difficulty, reachability and initialization distributions must be validated per embodiment. Do not compare raw success rates for non-equivalent task instances as though they were the same experiment.

## 7. Safety boundary

Fallback behavior is robot-specific; holding the last command is not inherently safe. Hardware commands are disabled by default and require instance configuration, operational authorization and site safety conditions. RoboForge watchdogs, command limits and logs do not replace a hardware safety system.
