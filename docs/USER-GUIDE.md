# SnapKit Python — User Guide

Complete guide to the tolerance-compressed attention allocation library.

## Table of Contents

1. [Concepts](#concepts)
2. [SnapFunction](#snapfunction)
3. [DeltaDetector](#deltadetector)
4. [AttentionBudget](#attentionbudget)
5. [ScriptLibrary](#scriptlibrary)
6. [LearningCycle](#learningcycle)
7. [SnapTopology (ADE)](#snaptopology-ade)
8. [ConstraintSheaf (Cohomology)](#constraintsheaf-cohomology)
9. [Advanced Modules](#advanced-modules)
10. [Common Patterns](#common-patterns)

---

## Concepts

### Snap Function

A snap function maps continuous values to discrete expected points. Values within `tolerance` are compressed (snapped) to the expected baseline. Values exceeding tolerance are **deltas** — information that demands attention.

The snap function is the **gatekeeper of attention**. It determines what reaches consciousness and what is safely ignored.

### Delta

A delta is any observation that exceeds snap tolerance. Deltas carry:
- **Magnitude** — how far from expected
- **Severity** — NONE / LOW / MEDIUM / HIGH / CRITICAL
- **Actionability** — can thinking change this? [0..1]
- **Urgency** — does this need attention NOW? [0..1]

### Attention Budget

Cognition is finite. The attention budget models `Σ A_i ≤ A_max` — total allocated attention cannot exceed available bandwidth. Deltas are ranked by `magnitude × actionability × urgency` and allocated proportionally.

### Scripts

Scripts are pre-learned patterns that execute automatically. When a situation snaps to a known script, cognition is freed for higher-level planning. This is expertise: speedcubing algorithms, poker basic strategy, surgical techniques.

### Learning Cycle

Expertise follows a cycle: **experience → pattern → script → automation → disruption → rebuilding**. The learning cycle tracks which phase you're in and manages cognitive load accordingly.

---

## SnapFunction

### Basic Usage

```python
from snapkit import SnapFunction, SnapTopologyType

snap = SnapFunction(
    tolerance=0.1,
    topology=SnapTopologyType.HEXAGONAL,
    baseline=0.0,
    adaptation_rate=0.01,
)

result = snap.snap(0.05)
# SnapResult(original=0.05, snapped=0.0, delta=0.05, within_tolerance=True)

result = snap.snap(0.3)
# SnapResult(original=0.3, snapped=0.3, delta=0.3, within_tolerance=False)  ← DELTA
```

### Working with SnapResult

```python
result = snap.snap(0.15)
result.original          # 0.15
result.snapped           # 0.0 (snapped to baseline)
result.delta             # 0.15
result.within_tolerance  # False (0.15 > 0.1)
result.is_delta          # True
result.tolerance         # 0.1
result.topology          # SnapTopologyType.HEXAGONAL
```

### Auto-Calibration

Set tolerance automatically from sample data:

```python
snap = SnapFunction()
data = [0.1, 0.12, 0.08, 0.11, 0.09, 0.5]
snap.calibrate(data, target_snap_rate=0.9)
# Sets baseline = mean(data) ≈ 0.167
# Sets tolerance so 90% of sample snaps
```

### Adaptive Tolerance

```python
snap = SnapFunction(tolerance=0.1)
snap.enable_adaptive_tolerance(window=50)

# Tolerance auto-adjusts:
# - High delta rate → tighter tolerance (more attention)
# - Low delta rate → looser tolerance (less attention)

snap.disable_adaptive_tolerance()  # restore base tolerance
```

### Hierarchical Snap

Snap at multiple tolerance levels simultaneously:

```python
snap = SnapFunction(baseline=0.0)
results = snap.snap_hierarchical(0.15, levels=[0.01, 0.05, 0.1, 0.2, 0.5])

# results[0]: tolerance=0.01, is_delta=True  (tight: catches micro-deltas)
# results[2]: tolerance=0.1,  is_delta=True
# results[4]: tolerance=0.5,  is_delta=False (loose: snaps it away)

profile = snap.hierarchical_profile(0.15)
# profile['transition_tolerance'] → the tolerance where snap flips to delta
```

### Complex / Eisenstein Snap

```python
snap = SnapFunction()
result = snap.snap_complex(complex(1.2, 0.7))
# Snaps to nearest Eisenstein integer on A₂ lattice
# Uses basis ω = -1/2 + i√3/2
```

### Vector and Matrix Snap

```python
import numpy as np

snap = SnapFunction(tolerance=0.1, baseline=0.0)

# Vector
values = np.array([0.05, 0.15, 0.25, 0.03])
results = snap.snap_vector(values)

# N-dimensional
matrix = np.random.rand(3, 4)
snapped = snap.snap_matrix(matrix, axis=1)
```

### Batch Processing

```python
snap = SnapFunction(tolerance=0.1)

batch_result = snap.snap_batch([0.05, 0.15, 0.25, 0.03])
# batch_result['snap_rate']     → 0.5
# batch_result['mean_delta']    → 0.12
# batch_result['max_delta']     → 0.25
```

### Rolling Window Snap

```python
snap = SnapFunction()
time_series = [0.1, 0.12, 0.08, 0.5, 0.11, 0.09, 0.48]

windows = snap.snap_rolling(time_series, window=3, stride=1)
# Each window: baseline, tolerance, snap_rate, mean_delta
```

### Serialization

```python
snap = SnapFunction(tolerance=0.1)
snap.snap(0.05)
snap.snap(0.3)

state = snap.to_dict()           # → dict
restored = SnapFunction.from_dict(state)  # → identical state
```

### Statistics

```python
snap = SnapFunction(tolerance=0.1)
for v in [0.05, 0.12, 0.3, 0.08, 0.15]:
    snap.observe(v)

snap.snap_rate      # 0.6 (3 of 5 snapped)
snap.delta_rate     # 0.4
snap.calibration    # 0.6 (= snap_rate)
snap.statistics
# {'total_observations': 5, 'snap_count': 3, 'delta_count': 2, ...}
```

---

## DeltaDetector

### Basic Multi-Stream Detection

```python
from snapkit import SnapFunction, DeltaDetector

detector = DeltaDetector()

# Add streams with independent snap functions
detector.add_stream('temperature', SnapFunction(tolerance=0.5, baseline=98.6))
detector.add_stream('heart_rate', SnapFunction(tolerance=10, baseline=72))
detector.add_stream('bp', SnapFunction(tolerance=5, baseline=120))

# Observe values
results = detector.observe({
    'temperature': 99.1,
    'heart_rate': 85,
    'bp': 135,
})
# results['temperature'].severity → DeltaSeverity.NONE (within 0.5)
# results['heart_rate'].severity  → DeltaSeverity.MEDIUM (85 - 72 = 13 > 10)
# results['bp'].severity          → DeltaSeverity.MEDIUM (135 - 120 = 15 > 5)
```

### Delta Severity

```python
from snapkit.delta import DeltaSeverity

delta = results['heart_rate']
delta.severity           # DeltaSeverity.MEDIUM
delta.magnitude          # 13.0
delta.exceeds_tolerance  # True
delta.attention_weight   # 13.0 * actionability * urgency
```

| Severity | Condition |
|----------|-----------|
| NONE | ratio ≤ 1.0 |
| LOW | ratio ≤ 1.5 |
| MEDIUM | ratio ≤ 3.0 |
| HIGH | ratio ≤ 5.0 |
| CRITICAL | ratio > 5.0 |

Where `ratio = magnitude / tolerance`.

### Prioritization

```python
# Get top-k deltas ranked by attention weight
top3 = detector.prioritize(top_k=3)
for d in top3:
    print(f"{d.stream_id}: mag={d.magnitude:.1f}, weight={d.attention_weight:.1f}")
```

### Actionability and Urgency Functions

```python
# Actionability: can thinking change this?
def market_actionability(value):
    if value > 0.8:  # Market spike — can't control
        return 0.2
    return 0.9  # Normal fluctuation — can respond

detector.add_stream(
    'market',
    SnapFunction(tolerance=0.1),
    actionability_fn=market_actionability,
    urgency_fn=lambda v: min(v * 10, 1.0),
)
```

### Delta Clustering

```python
clusters = detector.delta_clusters(n_clusters=3)
for cluster_id, deltas in clusters.items():
    print(f"Cluster {cluster_id}: {len(deltas)} deltas")
```

### Stream Statistics

```python
stats = detector.statistics
# {'num_streams': 3, 'total_observations': 10, 'total_deltas': 3, ...}
```

---

## AttentionBudget

### Basic Allocation

```python
from snapkit import AttentionBudget

budget = AttentionBudget(total_budget=100.0, strategy='actionability')

deltas = detector.prioritize(top_k=10)
allocations = budget.allocate(deltas)

for alloc in allocations:
    print(f"{alloc.delta.stream_id}: {alloc.allocated:.1f} units (priority {alloc.priority})")
    # "bp: 45.2 units (priority 1, high actionability; high urgency)"
```

### Allocation Strategies

```python
# Actionability-weighted (THE expert strategy)
budget = AttentionBudget(total_budget=100.0, strategy='actionability')

# Reactive: attend to biggest deltas regardless of actionability
budget = AttentionBudget(total_budget=100.0, strategy='reactive')

# Uniform: equal attention to all deltas
budget = AttentionBudget(total_budget=100.0, strategy='uniform')
```

### Multi-Level Attention

```python
# Macro: which streams to attend to
# Micro: which deltas within a stream
result = budget.multi_level_allocate(
    macro_deltas=top_deltas,
    micro_deltas={'stream_a': micro_a, 'stream_b': micro_b},
    macro_budget=40.0,
    micro_budget=60.0,
)
```

### Attention with Reserve

```python
# Keep 20% reserve for unexpected critical deltas
allocations = budget.allocate_with_reserve(deltas, reserve_fraction=0.2)
```

### Insights

```python
insight = budget.attention_insight()
# Detects: fixation, overload, underload, balanced
# insight['insights'][0]['type'] → 'fixation' | 'overload' | 'underload' | 'balanced'
```

---

## ScriptLibrary

### Creating Scripts

```python
from snapkit import ScriptLibrary, Script
import numpy as np

library = ScriptLibrary(match_threshold=0.85)

# Create a script manually
script = Script(
    id='basic_fold',
    name='Fold weak hand out of position',
    trigger_pattern=np.array([0.1, 0.2, 0.3]),
    response='fold',
)
library.add_script(script)

# Or learn from experience
script = library.learn(
    trigger_pattern=np.array([0.1, 0.2, 0.3]),
    response='fold',
    name='auto_fold_weak',
)
```

### Matching

```python
observation = np.array([0.12, 0.19, 0.31])

# Best match
match = library.find_best_match(observation)
if match and match.is_match:
    script = library.get(match.script_id)
    print(f"Execute: {script.response}")

# All matches (loose threshold)
all_matches = library.find_all_matches(observation)

# Conflict resolution when multiple scripts match
resolved = library.resolve_conflicts(observation)
```

### Script Composition

```python
# Chain multiple scripts into a sequence
composite = library.compose(['step_1', 'step_2', 'step_3'])
# composite.response → {'sequence': ['step_1', 'step_2', 'step_3'], 'responses': {...}}
```

### Script Inheritance

```python
# Create a specialized variant
child = library.extend('basic_fold', 'fold_weak_early_pos', response='fold_early')
```

### Script Versioning

```python
new_version = Script(id='basic_fold', ...)
library.update('basic_fold', new_version)  # archives old, activates new

history = library.version_history('basic_fold')
```

### Recording Outcomes

```python
script = library.get('basic_fold')
script.record_use(success=True, timestamp=42)
script.record_use(success=False, timestamp=43)
print(script.success_rate)  # 0.5
print(script.confidence)    # adjusted based on history
```

### Script Plans

```python
from snapkit.scripts import ScriptPlan

plan = ScriptPlan(name="Strategy", library=library)
plan.add_step('opening_move')
plan.add_step('midgame', conditions={'ahead': True}, fallback='midgame_defensive')
plan.add_step('endgame')

result = plan.execute(observation)
print(f"Progress: {plan.progress:.0%}")
```

---

## LearningCycle

### Basic Usage

```python
from snapkit import LearningCycle, SnapFunction

cycle = LearningCycle(
    snap=SnapFunction(tolerance=0.1),
    novelty_threshold=5,        # consecutive deltas before disruption
    script_creation_threshold=3, # similar deltas before creating a script
)

for value in data_stream:
    state = cycle.experience(value)
    print(f"Phase: {state.phase.value}, Load: {state.cognitive_load:.2f}")
```

### Learning Phases

| Phase | Description | Cognitive Load |
|-------|-------------|----------------|
| `DELTA_FLOOD` | No scripts, everything is novel | ≈ 1.0 |
| `SCRIPT_BURST` | Patterns emerging, rapid creation | 0.5-0.8 |
| `SMOOTH_RUNNING` | Most things snap to scripts | ≈ 0.0 |
| `DISRUPTION` | Scripts failing, deltas accumulating | rising |
| `REBUILDING` | Constructing new scripts | 0.5-0.8 |

### Experience Buffer

```python
from snapkit.learning import ExperienceBuffer

buffer = ExperienceBuffer(capacity=1000)
buffer.store(observation=0.42, delta=0.05, was_scripted=True, script_id='s1')

# Replay experiences to strengthen scripts
buffer.replay(cycle, n=32)
```

### Transfer Learning

```python
# Transfer scripts from another domain
count = cycle.transfer_knowledge(other_cycle)
print(f"Imported {count} scripts")
```

### Forgetting Curve

```python
# Scripts decay if not used (Ebbinghaus forgetting)
cycle.apply_forgetting(decay_rate=0.01)
```

---

## SnapTopology (ADE)

### Using Topologies

```python
from snapkit.topology import (
    SnapTopology, ADEType, ADE_DATA,
    binary_topology, hexagonal_topology, tetrahedral_topology,
    triality_topology, exceptional_e8,
    all_topologies, recommend_topology,
)

# Get a specific topology
topo = hexagonal_topology()  # A₂ — densest 2D packing
# SnapTopology(A2, rank=2, h=3, |Φ|=6)

# Snap a point
import numpy as np
snapped, delta = topo.snap(np.array([1.2, 0.7]))
```

### ADE Data

```python
data = ADE_DATA[ADEType.E8]
# {'rank': 8, 'dim': 8, 'roots': 240, 'coxeter': 30, 'solid': 'Dodecahedron/Icosahedron'}

for topo in all_topologies():
    print(f"{topo.name}: rank={topo.rank}, roots={topo.num_roots}, h={topo.coxeter_number}")
```

### Recommending Topologies

```python
recommend_topology(num_categories=2)   # → A₁ (binary)
recommend_topology(num_categories=4)   # → A₃ (tetrahedral)
recommend_topology(dimension=2)        # → A₂ (hexagonal, provably optimal)
recommend_topology(tensor_rank=8)      # → E₈
```

---

## ConstraintSheaf (Cohomology)

### Constraint Checking

```python
from snapkit.cohomology import ConstraintSheaf
from snapkit.topology import hexagonal_topology

sheaf = ConstraintSheaf(
    topology=hexagonal_topology(),
    tolerance=0.1,
)

sheaf.add_constraint('temp', 98.6)
sheaf.add_constraint('bp_sys', 120.0)
sheaf.add_constraint('bp_dia', 80.0)
sheaf.add_dependency('bp_sys', 'bp_dia')

report = sheaf.check_consistency()
report.globally_consistent  # True ↔ H¹ = 0
report.max_delta            # largest constraint violation
report.h1_analog            # number of constraints exceeding tolerance
```

### Eisenstein Constraint Check

```python
values = [complex(1, 0), complex(0.5, 0.3), complex(-0.2, 0.8)]
report = sheaf.check_eisenstein(values)
```

---

## Advanced Modules

### Adversarial Calibration

```python
from snapkit.adversarial import AdversarialSnap

# Detect fake deltas — values designed to trigger false attention
adv = AdversarialSnap(snap)
result = adv.check(value)
# result.is_authentic → True if delta is genuine
# result.camouflage_score → how well it mimics expected distribution
```

### Cross-Domain Transfer

```python
from snapkit.crossdomain import CrossDomainTransfer

# Transfer calibrated tolerances between domains with same topology
transfer = CrossDomainTransfer()
transfer.calibrate(source_snap, target_snap)
```

### Streaming

```python
from snapkit.streaming import StreamSnap

stream = StreamSnap(snap, window_size=100)
for value in data_stream:
    result = stream.process(value)
    if result.is_delta:
        handle_delta(result)
```

### Pipeline

```python
from snapkit.pipeline import SnapPipeline

pipeline = SnapPipeline()
pipeline.add_stage('snap', snap_stage)
pipeline.add_stage('detect', delta_stage)
pipeline.add_stage('allocate', budget_stage)
result = pipeline.process(data)
```

---

## Common Patterns

### Anomaly Detection

```python
snap = SnapFunction(tolerance=0.1)
snap.calibrate(normal_data, target_snap_rate=0.95)

for value in incoming_data:
    result = snap.observe(value)
    if result.is_delta:
        alert(f"Anomaly: {result.delta:.4f} from baseline {snap.baseline:.4f}")
```

### Multi-Stream Monitoring

```python
detector = DeltaDetector()
for name, data in streams.items():
    snap = SnapFunction(tolerance=0.1)
    snap.calibrate(data, target_snap_rate=0.9)
    detector.add_stream(name, snap)

budget = AttentionBudget(total_budget=100.0)

for tick, values in time_series:
    detector.observe(values)
    allocations = budget.allocate(detector.prioritize(top_k=5))
    # Focus attention on the most actionable deltas
```

### Adaptive Learning System

```python
cycle = LearningCycle(snap=SnapFunction(tolerance=0.1))
buffer = ExperienceBuffer(capacity=10000)

for experience in experience_stream:
    state = cycle.experience(experience)
    buffer.store(experience, delta=..., was_scripted=...)

    # Periodically replay to strengthen scripts
    if cycle.current_state.phase == LearningPhase.SMOOTH_RUNNING:
        buffer.replay(cycle, n=32)

    # Detect phase transitions
    transition = cycle.detect_phase_transition()
    if transition:
        print(f"Phase: {transition['from']} → {transition['to']}")
```

---

*Part of the Cocapn constraint theory ecosystem.*
