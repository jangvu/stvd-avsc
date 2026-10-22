# STVD-AVSC

**Candidate-Conditioned Ontology-Guided Audio-Visual Scene Classification under Ambiguous Acoustic Evidence**

This repository contains the code, manifests, and experimental results used to reproduce a training-free audio-visual scene-classification framework based on frozen CLIP and AST models.

The method uses CLIP for visual scene prediction, AST for acoustic event extraction, an explicit acoustic ontology for semantic reasoning, and candidate-conditioned rules for selected ambiguous scene groups.

## Repository Structure

```text
stvd-avsc/
├── manifests/
│   └── dataset manifests
├── notebooks/
│   └── 6000-recording development experiments
├── notebooks_ablatation/
│   └── ontology and candidate-rule ablations
├── notebooks_eva/
│   └── 3500-recording independent evaluation
├── results/
│   └── experimental outputs and summaries
├── Acoustic_ontology.ipynb
└── README.md
```

Each notebook folder contains its own `README.md` with the execution order, experiment details, and relevant settings.

## Experimental Protocol

Two balanced subsets of the TAU Audio-Visual Urban Scenes dataset are used:

| Split | Recordings | Per class | Purpose |
|---|---:|---:|---|
| Development | 6000 | 600 | Baselines, ontology/rule design, parameter analysis |
| Independent evaluation | 3500 | 350 | Frozen final evaluation |

The overall workflow is:

```text
6000 development set
CLIP / PANNs / AST comparison
        ↓
ontology and candidate-rule design
        ↓
freeze method
        ↓
3500 independent evaluation
CLIP → AST → ontology → candidate rules
```

No task-specific neural-network fine-tuning is performed.

## Main Results

| Dataset | CLIP | Ontology | Proposed |
|---|---:|---:|---:|
| Development | 78.93% | 79.73% | **81.12%** |
| Evaluation | 77.03% | 77.80% | **78.97%** |

The proposed method improves accuracy by **+2.18 percentage points** on the development set and **+1.94 percentage points** on the independent evaluation set.

## Main Components

- **CLIP** — visual baseline and Top-3 candidate generator.
- **AST** — acoustic event extractor.
- **Acoustic ontology** — maps detected events to semantic scene evidence.
- **Candidate-conditioned rules** — selectively refine ambiguous candidate groups while preserving accepted ontology corrections.

The ontology also provides a traceable reasoning path:

```text
acoustic event → semantic concept → candidate scene → final decision
```

## Reproduction

Start with the README inside the relevant folder:

- `notebooks/` for development experiments;
- `notebooks_ablatation/` for ablation studies;
- `notebooks_eva/` for independent evaluation.

The raw TAU audio/video data are not redistributed in this repository. Dataset paths are referenced through the manifests in `manifests/`.

## Repository
