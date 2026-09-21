# Technical Decision Record

[Back to README](../README.md) · Decision baseline: 2026-09-20

“Default” means the intended first-version design choice, not a claim of implementation, benchmark results, or deployment validation. Dependency compatibility and physical-robot parameters must be established during bring-up.

## Decision table

| ID | Default | Rationale / boundary | Re-evaluate when |
| --- | --- | --- | --- |
| D01 | Experiment-centric modular Python project + independent workers | Unify semantics without premature microservices | Real multi-team deployment requires stronger isolation |
| D02 | Isaac Lab + Isaac Sim / PhysX as primary simulation path; RTX for visual tasks | Reuse existing learning and sensor ecosystem | Calibrated tasks show another backend is materially better |
| D03 | Arena behind a replaceable adapter; Core does not inherit Arena wholesale | Reuse task/evaluation components while containing upstream change | Bring-up remains blocked; fall back to native Lab environments |
| D04 | Genie primarily supplies G2 references and standalone tools; do not merge runtimes by default | Avoid changing scene/control semantics while comparing timing | Paired tests demonstrate equivalent configuration and behavior |
| D05 | Full G2 reference + simulated Franka portability check | Prove abstraction with a second embodiment | A different physical robot becomes a concrete requirement |
| D06 | Clock and deployment topology are orthogonal; v0 HIL is compute HIL | Do not conflate time progression with hardware coverage | Drives / physical plant enter the loop |
| D07 | Lockstep worker barrier; wall-paced async inference + action queue | The timing modes optimize for different goals | Measured throughput requires more complex scheduling |
| D08 | RobotPackage + capability declaration; actions have explicit names/units | Avoid fixed joint dimensions and hidden transforms | A concrete new control mode requires extension |
| D09 | Thin TaskDefinition / Binding + Python predicates | Reuse task components instead of inventing a DSL | Multiple backends cannot share required semantics |
| D10 | Unify policy semantics, keep transport in adapters | Reuse existing model servers; avoid premature RPC standardization | Cross-language / bandwidth / latency becomes a measured bottleneck |
| D11 | ROS 2 / vendor SDK at deployment boundary; tensor paths inside batched simulation | Avoid routing GPU-local data through ROS | A deployment topology explicitly requires messaging |
| D12 | Separate traces, tool working data, training exports and results | One format should not serve conflicting purposes | Data scale/access patterns materially change |
| D13 | Object storage + PostgreSQL; local containers first | Reuse infrastructure rather than build a data platform | Multi-node orchestration justifies OSMO |
| D14 | Upstream generators produce candidates; agents cannot modify official scoring | Protect benchmark integrity, asset quality and holdouts | Automated QA/review evidence justifies narrowly scoped delegation |
| D15 | Newton is a task-level candidate, not a first-version multi-backend commitment | Do not promise engine equivalence | Contact/deformable/performance needs have real-world evidence |
| D16 | Original code/docs under MIT; external materials tracked separately | Licenses do not inherit from the host repository | Re-check on every third-party import |

## 1. Dependency baseline and upgrades

Start from a coherent dependency set recommended by Arena rather than independently combining each project's latest release. Record actual commits, submodules, lockfiles, container digests, CUDA/driver versions and hardware.

Specific branch/version numbers discussed during research are leads, **not a validated compatibility matrix in this repository**. Official entry points: [Isaac Lab](https://github.com/isaac-sim/IsaacLab), [Arena](https://github.com/isaac-sim/IsaacLab-Arena).

Maintain production and candidate runtime configurations. Promote a candidate only after G2, second-robot, sensor, reset and timing regressions. Keep the previous baseline for comparison without maintaining indefinitely diverging production stacks.

If Arena blocks the first tasks, use native environments from the same Isaac Lab baseline. Do not rewrite upstream solely to preserve an “Arena-based” label.

## 2. Engineering-stack defaults

| Area | Default | Constraint |
| --- | --- | --- |
| Core | Python 3.12, uv, Pydantic v2, YAML, pytest | Simulation/inference containers follow their own compatibility constraints |
| GPU integration node | Ubuntu 24.04 + ROS 2 Jazzy candidate | Preserve vendor-supported G2 onboard stack; lock after testing |
| Control | Vendor SDK/controllers; separate C++ process when necessary | Python main loop does not promise high-frequency hard real-time |
| Model serving | Native servers + PolicyAdapter | No specific model is claimed supported yet |
| Jobs | OCI containers + LocalExecutor; OSMO when scale requires it | Job scheduler never owns individual control cycles |
| Storage | Existing object store + PostgreSQL | Manifests/large artifacts immutable; DB primarily indexes state |
| Reporting | JSON / Parquet + static visual reports | No full web product initially |

Actual software support follows [Isaac Sim documentation](https://docs.isaacsim.omniverse.nvidia.com/), [ROS 2 Jazzy documentation](https://docs.ros.org/en/jazzy/) and dependency tests. These are target defaults, not installation instructions.

## 3. Data formats

Use MCAP or native capture files for raw time series, with large images/tensors externalized and referenced by manifests. Generation tools retain their native working formats. Prefer LeRobot v3 for training export and JSON / Parquet for evaluation results. See [Evaluation & Data](evaluation-and-data.md).

Do not repeatedly transcode video or discard original timing merely to enforce format uniformity. Schema migrations require versioned converters and tests.

## 4. Build / Adopt / Integrate

| Strategy | Scope |
| --- | --- |
| Build: must own | Domain tasks, scoring protocol, RobotPackage, timing semantics, trace contract, real-world calibration, quality gates |
| Adopt: avoid reimplementation | Simulation physics/rendering, training frameworks, storage, containers and cluster scheduling |
| Integrate: thin adapters | Arena, Genie, policy servers, Mimic / SkillGen, upstream generators, ROS / SDK |
| Defer until evidence | Multi-engine validation, complex automatic curricula, deformables, deeper hardware HIL, full UI |

Even when community tools provide similar capabilities, domain scoring and the real-world validity envelope remain internal responsibilities. Conversely, do not reimplement commodity framework features and call them platform differentiation.

## 5. Licensing and provenance

The initial repository contains original planning documents, SVG and an MIT License; it does not yet contain third-party source code, robot assets or model weights.

Future imports record source URL, commit/version, path, license, attribution, modifications and distribution scope. This includes files copied or referenced from Genie and assets/models downloaded by generators. Being installable as a dependency does not imply that all dependency materials may be redistributed under MIT.
