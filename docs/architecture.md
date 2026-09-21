# Architecture and Scope

[Back to README](../README.md) · Design baseline: 2026-09-20

This document describes the target architecture, not implementation status. See [Technical Decisions](technical-decisions.md) for defaults and [Roadmap](roadmap.md) for delivery boundaries.

## 1. What the platform should deliver

RoboForge does not primarily deliver a `.usd` file. Its outputs are **trusted evaluation reports, validated training data, and complete experiment records that make both reproducible and auditable**.

Long-term value comes from domain tasks, robot/control integration, real-world calibration, data quality, and reproducible model iteration. Better upstream generators should provide better candidate assets without forcing the core platform to change.

The initial scope is rigid-body manipulation plus a limited set of articulated tasks. G2 is the full reference implementation; simulated Franka is the portability check. Mobile manipulation, deformables, additional robots, and additional physics backends are added only when justified.

## 2. Layers and ownership

![Architecture](assets/architecture.svg)

| Layer | Core objects | RoboForge responsibility | Prefer to reuse |
| --- | --- | --- | --- |
| Experiment definition | `ExperimentSpec` | Pin task, robot, policy, controller, timing, seeds, split and scorer version | YAML / Pydantic |
| Domain contracts | `RobotPackage`, `TaskDefinition`, `TaskBinding`, `PolicyAdapter` | Decouple embodiments, tasks and policies | Arena registration and task components |
| Execution | `ExecutionProfile`, `Worker` | Lockstep / timed execution, lifecycle, timeout and action isolation | Isaac Lab; ROS 2 / vendor SDK |
| Evidence and data | `Trace`, `EvidenceProvider`, `DatasetManifest` | Recording, scoring, calibration, QA and lineage | MCAP, Parquet, LeRobot |
| Workflow | `JobSpec`, `ArtifactRef` | Compose evaluation, production, training and release jobs; stay out of the control loop | Local containers; later OSMO |
| Upstream content | `AssetManifest`, candidate tasks | Ingest and validate; never trust untested generated content | Genie, SAGE, Blender, etc. |

**Reuse frameworks; own contracts.** Core types should not inherit an upstream simulator's entire object model. Arena / Isaac details stay behind adapters so upgrades primarily affect adapters rather than hardware interfaces and stored data.

## 3. Minimum experiment definition

An experiment binds immutable references to at least:

| Field | Meaning |
| --- | --- |
| `task` / `binding` | Task semantics plus concrete robot, scene and object bindings |
| `robot_model` / `robot_instance` | Model, variant and instance calibration; pure simulation may use an explicit virtual instance |
| `policy` / `controller` | Weights, preprocessing/postprocessing, action transforms, IK / WBC and controller configuration |
| `execution` | Clock, plant, policy/controller deployment and sensor sources |
| `world` / `assets` | Scene, physics, materials, references and asset versions |
| `metrics` / `evidence` | Criteria, evidence sources, thresholds and insufficient-evidence handling |
| `initialization` / `split` | Initial-state distribution, seeds, development / regression / holdout membership |
| `runtime` | Code commit, dependency lock, container, hardware and driver information |

Model weights alone are not the evaluated system: policy preprocessing, action handling, and controllers are part of the evaluation target.

## 4. Time and deployment are orthogonal

`clock = lockstep | wall_paced` describes how time advances.

`plant = simulated | physical` describes what is being controlled. Policies and controllers may run locally, in remote software processes, or on target compute; sensors may be simulated or explicitly declared physical inputs.

SIL / HIL are named deployment configurations, not physics engines. A physical plant cannot arbitrarily stop time. Invalid combinations must be rejected before execution. See [Execution Model](execution-model.md).

In v0, HIL specifically means **the actual target compute device runs deployment software while controlling a simulated robot in closed loop**. It does not claim that physical motors, drives, or mechanics are in the loop.

## 5. Same experiment contract, not necessarily the same loop

A lockstep worker can batch homogeneous environments on GPU. A wall-paced worker prioritizes resource isolation and end-to-end deadlines. They share experiment, action, and trace contracts but do not need the same scheduler loop.

Simulation, policy inference, and ROS / SDK components should run in separate processes or containers. Training is a separate job: RoboForge exchanges datasets and model artifacts without becoming the training framework.

SIL/HIL should preserve the lockstep experiment's scene and control definitions whenever possible. If a separate Genie runtime is used, it is a different runtime and requires paired validation; it must not be represented as merely another mode label.

## 6. Upstream generation and asset validation

Recommended flow:

```text
generation / reconstruction / import → candidate asset → static checks → physics and task tests
                                     → versioned registry → scene binding → evaluation / data production
```

Visuals, collision, physics and semantics are managed separately. Preserve source URDF / CAD / Blender assets and import recipes; Isaac consumes versioned USD runtime artifacts. Do not rely on arbitrary format round-trips to preserve semantics.

Before promotion, validate scale, coordinates, hierarchy, dependencies, collision, articulation, stability and task usability. Model-inferred mass, friction and similar properties are priors to calibrate, not measurements. Neural visual backgrounds and interactive foreground objects may use different representations, but sensor, occlusion and collision consistency must be tested.

## 7. Where agents belong

Frontier models may assist with configuration generation, candidate scenes, tooling, failure diagnosis and repair patches. Model provider, version, tool permissions and cost should remain replaceable and traceable.

Agents must not modify frozen scoring rules, holdout sets, or release gates during official runs. Candidate assets and repairs are promoted only after tests and review. Commands to physical robots pass through a separately approved hardware boundary; a general code-execution tool must not bypass it.

## 8. Proposed implementation layout

This is an implementation proposal, **not a claim that these directories or packages already exist**:

```text
src/roboforge/
  contracts/        # experiment, action, task, evidence and trace schemas
  runtime/          # lockstep / wall-paced / lifecycle
  evaluation/       # scorers, reports and validity statistics
  data/             # traces, QA and export
  adapters/         # Isaac / Arena / policy transport / ROS
robot_packages/     # g2, franka; model and instance separated
experiments/        # versioned task and run configurations
integrations/       # external collection, generation, training and job adapters
```

Start as a modular project rather than a fleet of microservices. Adding a robot or policy should add packages/adapters rather than change the core execution loop.

## 9. Non-goals

The first version will not build a new physics engine, generic training framework, task-planning language, cluster scheduler, or full web product. It will not promise automatic policy transfer across embodiments, equivalence across physics engines, bitwise reproducibility, hard real-time guarantees, or automatic conversion of arbitrary scenes into training-ready environments.

Software watchdogs are not emergency stops or safety certification. Physical-robot experiments still require site-specific risk assessment, vendor safety mechanisms, and controlled testing.
