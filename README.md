# LIBERO Instruction Robustness

A lightweight research repository for studying **language robustness and instruction generalization of Vision-Language-Action (VLA) models** using the LIBERO benchmark.

The main goal is to evaluate how different linguistic expressions of the **same manipulation task** affect VLA policy performance.

This repository currently focuses on LIBERO and LingBot-VLA, while keeping the dataset and evaluation interfaces lightweight enough to support other VLA models in the future.

---

## 1. Research Goal

Standard LIBERO tasks provide a canonical language instruction associated with each manipulation task.

For example:

```text
Original:
put the black bowl on the plate
```

We construct alternative instructions while preserving the original task semantics:

```text
place the black bowl on the plate

move the black bowl onto the plate

set the black bowl down on the plate
```

The key experimental principle is:

```text
task             unchanged
scene            unchanged
objects          unchanged
goal             unchanged
initial state    unchanged
policy           unchanged
evaluation setup unchanged

instruction      changed
```

This allows us to measure the effect of language variation on VLA performance as independently as possible.

---

## 2. Main Research Questions

This project mainly investigates:

* How sensitive are VLA policies to different phrasings of the same instruction?
* How much does instruction paraphrasing affect manipulation success rate?
* Which types of language variation cause the largest performance degradation?
* Can instruction augmentation during fine-tuning improve language robustness?
* How well do robustness improvements generalize across different LIBERO task suites?

---

## 3. LIBERO Benchmarks

The initial experiments focus on the standard LIBERO task suites:

```text
libero_spatial
libero_object
libero_goal
libero_10
```

The first development target is:

```text
libero_spatial
```

The remaining suites will be added after the complete dataset and evaluation pipeline has been verified.

LIBERO evaluation is simulation-based.

The policy receives:

```text
RGB observation
+
robot state
+
language instruction
```

and predicts robot actions that are executed inside the LIBERO environment.

Performance is primarily measured using **task success rate**.

---

## 4. Repository Scope

This repository is intentionally lightweight.

It does **not** contain:

* the full LIBERO dataset;
* LIBERO simulation assets;
* LingBot-VLA model weights;
* the complete LingBot-VLA repository;
* large demonstration datasets;
* large rollout videos or model checkpoints.

These resources remain on the cloud server.

This repository contains only the code and metadata required for our own research experiments.

---

## 5. Project Layout

The planned structure is:

```text
libero-instruction-robustness/
├── README.md
│
├── configs/
│   └── ...
│
├── data/
│   ├── official_tasks/
│   │   └── ...
│   │
│   └── instruction_variants/
│       └── ...
│
├── scripts/
│   ├── ...
│   └── ...
│
├── src/
│   └── ...
│
├── results/
│   ├── summaries/
│   └── raw/
│
├── .gitignore
└── requirements.txt
```

The exact structure may evolve as the implementation develops.

Keep the repository small and avoid unnecessary abstractions or duplicated upstream code.

---

## 6. Official LIBERO Task Metadata

The complete LIBERO dataset is not required for local instruction-dataset development.

Instead, canonical task metadata is exported from the LIBERO installation on the cloud server.

For example:

```text
data/official_tasks/libero_spatial.jsonl
```

A task record may contain:

```json
{
  "suite": "libero_spatial",
  "task_id": 0,
  "task_name": "...",
  "instruction": "..."
}
```

These files are treated as the authoritative source for the original LIBERO instructions.

Do not manually invent or modify canonical LIBERO instructions.

---

## 7. Custom Instruction Dataset

Our own dataset stores alternative language instructions while referencing the original LIBERO task through:

```text
suite + task_id
```

Example:

```json
{
  "suite": "libero_spatial",
  "task_id": 0,
  "variant_id": 1,
  "variant_type": "paraphrase",
  "instruction": "..."
}
```

The custom dataset does not duplicate:

```text
RGB images
robot states
actions
simulation assets
demonstration trajectories
```

These remain part of the upstream LIBERO environment and datasets.

---

## 8. Instruction Variation Types

The dataset is designed to support multiple categories of language variation.

Possible categories include:

```text
original
synonym
syntactic_paraphrase
multiword_substitution
polite_expression
instruction_reordering
irrelevant_context
other
```

These categories are research metadata rather than fixed implementation constraints.

New categories can be added without modifying the core evaluation pipeline.

---

## 9. Semantic Preservation

Instruction variants must preserve the original manipulation goal.

For example:

```text
Original:
put the black bowl on the left plate
```

Valid:

```text
place the black bowl onto the left plate
```

Invalid:

```text
place the black bowl onto the right plate
```

The following task properties must remain unchanged:

* target object;
* target receptacle;
* spatial relation;
* manipulation goal;
* task ordering;
* object identity.

Instruction augmentation must not accidentally create a different task.

---

## 10. Evaluation Protocol

For fair comparison, different language variants of the same task should use identical evaluation conditions.

For each comparison, keep the following fixed:

```text
LIBERO task
initial state
random seed
policy checkpoint
observation preprocessing
action configuration
maximum episode length
simulation configuration
```

Only the instruction text should change.

Conceptually:

```text
for each task:
    for each initial state:
        evaluate original instruction
        evaluate instruction variant 1
        evaluate instruction variant 2
        ...
```

This paired evaluation design reduces variance caused by different environment initializations.

---

## 11. Evaluation Metrics

The main metric is:

### Success Rate

```text
Success Rate = successful episodes / total episodes
```

Results should be reported at several levels.

### Overall

```text
overall success rate
```

### By task

```text
task 0
task 1
...
```

### By instruction type

```text
original
synonym
paraphrase
irrelevant_context
...
```

### Performance Difference

For an augmented instruction category:

```text
Delta SR = augmented SR - original SR
```

This provides a simple measurement of language-induced performance degradation or improvement.

---

## 12. Local Development

Local development focuses on:

```text
official task metadata
        ↓
instruction generation
        ↓
dataset validation
        ↓
custom JSONL dataset
```

The local machine does not need:

```text
LIBERO simulator
LingBot-VLA checkpoints
large demonstration datasets
GPU inference
```

This keeps the local Codex workspace lightweight.

---

## 13. Cloud Evaluation

The complete evaluation is performed on the cloud server.

Recommended directory layout:

```text
coding/
├── lingbot-vla-v2/
│
├── datasets/
│   └── libero/
│
└── libero-instruction-robustness/
```

Responsibilities are separated as:

```text
lingbot-vla-v2/
    model implementation

datasets/libero/
    upstream LIBERO data

libero-instruction-robustness/
    our dataset and experiment code
```

The custom repository should not modify the upstream LingBot-VLA project unless explicitly necessary.

---

## 14. VLA Integration

The evaluation code should remain mostly independent from model internals.

Conceptually:

```text
LIBERO environment
        │
        ↓
observation + instruction
        │
        ↓
policy adapter
        │
        ↓
VLA model
        │
        ↓
action
        │
        ↓
LIBERO environment
```

LingBot-VLA will initially be used as the target policy.

The exact inference interface must be connected according to the actual LingBot-VLA implementation on the cloud server.

Do not fabricate model APIs when the corresponding source code has not been inspected.

---

## 15. Development Principles

Code in this repository should remain:

* concise;
* readable;
* research-oriented;
* easy to modify;
* easy to debug;
* reproducible.

Prefer simple Python implementations over unnecessary framework abstractions.

Avoid:

* excessive object-oriented design;
* duplicated LIBERO code;
* duplicated LingBot-VLA code;
* large utility frameworks;
* unnecessary dependencies;
* hardcoded machine-specific paths.

---

## 16. Large Files

The following files should not be committed to Git:

```text
LIBERO demonstrations
simulation assets
model checkpoints
large rollout logs
rollout videos
Hugging Face caches
W&B caches
temporary files
```

Small experiment metadata and summarized results may be tracked in Git.

---

## 17. Planned Workflow

The project will be developed incrementally.

### Phase 1 — Task Metadata

* Export official LIBERO task instructions.
* Verify task IDs and suite mappings.
* Store canonical metadata locally.

### Phase 2 — Instruction Dataset

* Construct instruction variants.
* Add augmentation categories.
* Validate task consistency.

### Phase 3 — LIBERO Evaluation

* Load identical LIBERO initial states.
* Replace only the task instruction.
* Execute VLA rollouts.
* Record success/failure results.

### Phase 4 — LingBot-VLA Integration

* Connect the evaluation pipeline to LingBot-VLA.
* Verify observation preprocessing.
* Verify action representation.
* Verify action chunk execution.

### Phase 5 — Experiments

Compare:

```text
official instructions
vs.
custom instruction variants
```

and later:

```text
baseline fine-tuning
vs.
instruction-augmented fine-tuning
```

across multiple LIBERO suites.

---

## 18. Current Status

The project is currently under development.

Current priority:

```text
LIBERO-Spatial
        ↓
official instruction extraction
        ↓
custom instruction dataset
        ↓
paired evaluation pipeline
        ↓
LingBot-VLA integration
```

Implementation details and executable commands will be added as the project structure becomes stable.
