# Research Notes: From Scene Generation to Trusted Robot Experiments

[Back to README](../../README.md) · Research baseline: 2026-09-20

## Scope and evidence

This document consolidates the research findings and project leads that shaped RoboForge. It is intentionally not another exhaustive project directory. Deep-research reports that were not preserved verbatim are not presented here as source material.

Only a small set of primary project entry points was rechecked while preparing these notes. We have not reproduced the cited systems on local GPUs or robots. Official documentation/open code, paper results, author demos and our own future measurements should be treated as different evidence levels. “Adopt” or “candidate” below describes a RoboForge design decision, not a maturity ranking or an integration claim.

Specific dependency versions, performance numbers and community-demo success rates are not platform guarantees. Compatibility, contact behavior and data value must be established during bring-up and paired experiments.

## 1. How the direction converged

| Research stage | What survived scrutiny | Implication for RoboForge |
| --- | --- | --- |
| GPT / Blender for rich scenes | Generate → execute → observe → repair is more valuable than one-shot mesh generation | Agents orchestrate content/tools; they do not certify physical correctness |
| SAGE / SimFoundry and related systems | Scene generation and Real2Sim already have strong reusable approaches | Do not build another generic scene generator |
| Internal Isaac evaluation + data workflows | Task semantics, control, calibration and data value matter more than scene count | Make versioned experiments the center of the platform |
| G2 reference implementation | Physical hardware plus a rich reference ecosystem lowers initial integration cost | G2-first, but prove the interface with a second embodiment |
| Stop-the-world + SIL/HIL | Time progression and deployment topology are different dimensions | Two timing semantics; shared tasks/traces; separate reports |

The resulting thesis: **turn generated or reconstructed worlds into executable, validated and traceable robot experiments, then demonstrate that those experiments improve model selection and data investment.**

## 2. Primary runtime stack: adoption and boundaries

| Project / primary entry point | What is reusable | RoboForge decision / gap |
| --- | --- | --- |
| [Isaac Sim](https://docs.isaacsim.omniverse.nvidia.com/) / [Isaac Lab](https://github.com/isaac-sim/IsaacLab) | Simulation, sensors, robot-learning environments | Primary runtime; our tasks, controls and sim-real validity still require validation |
| [Isaac Lab-Arena](https://github.com/isaac-sim/IsaacLab-Arena) | Composable environments and policy-evaluation components | Prefer an adapter; do not bind internal contracts to its entire type system; follow matched dependencies |
| [Genie Sim](https://github.com/AgibotTech/genie_sim) | G2-related assets, control, interaction and collection references | Selective reuse; validate concrete robot variants and runtime modules separately |
| [Newton](https://github.com/newton-physics/newton) | Candidate GPU physics and solver capabilities | Evaluate per task later; it is not a task framework and solvers should not be assumed equivalent |
| [OSMO](https://github.com/NVIDIA/OSMO) | Heterogeneous compute and multi-stage workflows | Candidate at scale; it does not own per-control-cycle scheduling or domain scoring |

These are not mutually exclusive choices. They occupy different layers: runtime, environment/evaluation, application, physics and orchestration. The default is Lab + Arena adapter, Genie as a reference/integration source, and Newton/OSMO when concrete needs justify them.

## 3. World generation and Real2Sim as upstream suppliers

| Project / source | Why it matters | Intended relationship |
| --- | --- | --- |
| [SAGE](https://github.com/NVlabs/sage) | Task-driven scene generation, model/tool orchestration and action-generation-related code | Candidate scene/task supplier; internal execution validation remains ours |
| [SimFoundry](https://research.nvidia.com/labs/gear/simfoundry/) / [code](https://github.com/NVlabs/SimFoundry) | Video-to-interactive-scene, digital cousins, modular reconstruction | Real2Sim candidate; do not equate paper pipeline with every module being production-ready/open |
| [EmbodiedGen](https://github.com/HorizonRobotics/EmbodiedGen) | Agent-driven assets/scenes and multi-simulator deployment paths | Candidate asset backend; validate model service, output quality and Isaac adaptation separately |
| [SceneSmith](https://scenesmith.github.io/) | Rich indoor content and staged generation | Method/content reference rather than default Isaac-native path |
| [Infinigen](https://infinigen.org/) | Procedural environments and asset variation | Useful for controllable distributions and repeated production; do not invoke an LLM on every reset |
| [SimForge](https://github.com/AndrejOrsula/simforge) | Procedural simulation-asset tooling | Candidate generation plugin; unrelated to RoboForge despite the similar name |
| [NVIDIA SimReady Blender Add-on](https://github.com/NVIDIA/simready-blender-add-on) | Structured Blender → physics/USD asset path | Preserve visual/collision/joints, then validate after import |
| [Marble → Isaac tutorial](https://developer.nvidia.com/blog/simulate-robotic-environments-faster-with-nvidia-isaac-sim-and-world-labs-marble/) | Neural visual backgrounds combined with explicit collision | Candidate for navigation/perception; interactive foreground still needs explicit physics |
| [FIRE3D](https://xiahongchi.github.io/Fire3D) | Object-centric reconstruction | Future Real2Sim candidate, not a first-version hard dependency |
| [Initial Blender case study](https://www.aiformortals.co/blog/gpt-6-astra-blender-fallingwater) | Long-running agent modeling, inspection and repair | Author demo; not evidence of valid physics, articulation, reset, or robot-task behavior |

The public SimFoundry repository observed during this research still described parts of the data-generation/training path as forthcoming while exposing scene/application components. Integration decisions should therefore be made module by module rather than relying on a broad “end-to-end complete” label.

### Path comparison

| Path | Best initial use | Main risk |
| --- | --- | --- |
| Isaac-native tasks + validated asset registry | First manipulation evaluation/data path | Domain tasks and robot adaptation still require work |
| Blender / CAD → USD → Isaac | Precise structure, industrial assets, procedural variants | Material, collision, articulation and scale conversion |
| Real2Sim → digital cousins | Real workstation reproduction and failure regression | Calibration, occlusion, contact and system identification |
| Neural background + explicit foreground | Visual richness, navigation and perception | Appearance vs. depth/semantic/physics inconsistency |
| Other engine → Isaac | Reuse of a specific benchmark or asset base | Readable format does not imply equivalent runtime semantics |

## 4. Data and models: reuse entry points, not a new training stack

| Project / source | Reuse / research value | Internal requirement |
| --- | --- | --- |
| Mimic / SkillGen workflows in [Isaac Lab](https://github.com/isaac-sim/IsaacLab) | Expand a small demonstration set into executable trajectories | Task stages, object relations and execution validation remain necessary |
| [MimicGen](https://mimicgen.github.io/) / [robomimic](https://github.com/ARISE-Initiative/robomimic) | Demonstration augmentation and imitation-learning methods | Preserve native working formats; unify QA and provenance |
| [LeRobot](https://github.com/huggingface/lerobot) | Robot datasets and policy tooling | Training export and policy adapters; does not replace full traces |
| [OpenPI](https://github.com/Physical-Intelligence/openpi) | Policy serving/deployment reference | Action processing, timing and control configuration must be versioned |
| [Isaac-GR00T](https://github.com/NVIDIA/Isaac-GR00T) | Generalist robot-policy integration reference | Weights are not the whole evaluated system; do not assume G2 support |
| [MCAP](https://mcap.dev/) / [RLDS](https://github.com/google-research/rlds) | Raw recording and episode representation | Separate raw trace, tool format, training export and evaluation result |

Synthetic perception data, expert trajectories and failure traces are different products. Appearance generation/augmentation systems such as Cosmos should be integrated according to their actual I/O contract, not treated as explicit physical worlds or verified action trajectories.

## 5. Directions worth tracking

These are follow-up research directions, not reproduced integrations or confirmed compatibility claims. They are kept to avoid losing important threads without expanding the first-version scope.

| Direction | Primary entry points / leads | Question |
| --- | --- | --- |
| Sim-real evaluation validity | [SIMPLER](https://simpler-env.github.io/) | Can simulation predict meaningful G2 model improvements, regressions and bad releases? |
| Task suites and abstractions | [ManiSkill](https://github.com/haosulab/ManiSkill), [RoboCasa](https://robocasa.ai/), [LIBERO](https://libero-project.github.io/) | Which task/scoring abstractions transfer, and which are embodiment/engine-specific? |
| Semantic and long-horizon tasks | [BEHAVIOR / OmniGibson](https://behavior.stanford.edu/), [BDDL](https://github.com/StanfordVL/bddl) | Can predicates/stages be reused without building a new task language? |
| Bimanual and automated data | [RoboTwin](https://robotwin-platform.github.io/), DexMimicGen | Can they reduce G2 demonstration and repair cost? |
| Counterexamples and robustness | [Scenic](https://scenic-lang.org/), [VerifAI](https://github.com/BerkeleyLearnVerify/VerifAI) | How should we search interpretable rare failures instead of only randomizing? |
| Multiple physics backends | [MuJoCo](https://github.com/google-deepmind/mujoco), Newton, [Drake](https://drake.mit.edu/), Genesis | Which tasks justify a second engine, and should disagreement be treated as a clue rather than ground truth? |
| Sensors and contact | Replicator, tactile/force simulation, sensor timing | Which observations beyond RGB-D matter, and how are they calibrated to hardware? |
| Agentic Real2Sim / physical asset generation | Agentic Real2Sim, PhysX-Anything and related work | Which properties may be inferred, which measured, and which require validation? |

These directions do not change the initial stack. The most valuable early additions are likely **validity envelopes, timing robustness, and failure-driven data ROI**, not more simulator labels without evidence.

## 6. Research conclusions translated into engineering constraints

**Scene → experiment.** A mesh or USD that opens is only the first step; articulation, collision, task solvability, reset, control and throughput still require validation.

**Demo → evidence.** Community demos and paper metrics establish direction. Release gates come from our own tasks, repeated experiments and paired physical tests.

**Generic generation → useful data distributions.** Scale generation according to coverage, failure modes and training value. Do not put an LLM in every RL reset.

**Unified semantics, not a universal backend.** Support different timing/deployment modes without claiming that different evidence sources or physics engines produce equivalent scores.

**Replaceable models and tools.** Frontier models are useful for orchestration and cross-stack diagnosis, but the project thesis should not depend on a single model name or an unverified capability.

The next step is to execute the [roadmap](../roadmap.md) and let evidence from G2 plus a second embodiment determine which candidates become real dependencies.
