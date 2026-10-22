# Evaluation Notebooks

This folder contains the notebooks used for the **3500-recording independent evaluation set** of the project:

**Candidate-Conditioned Ontology-Guided Audio-Visual Scene Classification under Ambiguous Acoustic Evidence**

The evaluation stage uses only the final frozen pipeline:

```text
CLIP → AST → ontology → candidate-conditioned rules
```

No parameter tuning, grid search, PANNs comparison, or naive fusion is performed on this split.

## Dataset

These notebooks use the balanced independent evaluation subset:

```text
data_subset_3500_test_balanced/
```

with the manifest:

```text
manifests/tau_balanced_3500_test_manifest.csv
```

The evaluation split contains **3500 recordings**, with **350 recordings per scene class**.

## Notebook Overview

| Notebook | Purpose |
|---|---|
| `01_CLIP_eval_3500.ipynb` | Run the frozen CLIP visual baseline on the 3500-recording evaluation set. |
| `02_AST_eval_3500.ipynb` | Extract AST AudioSet events using the same direct AST implementation used during development. |
| `03_CLIP_AST_ontology_eval_3500.ipynb` | Apply the frozen basic ontology to the CLIP Top-3 candidates using AST evidence. |
| `04_CLIP_AST_ontology_candidate_eval_3500.ipynb` | Apply the frozen candidate-conditioned transport, park, and metro-station rules on top of the ontology output. |

## Recommended Execution Order

Run the notebooks in the following order:

```text
1. 01_CLIP_eval_3500.ipynb
2. 02_AST_eval_3500.ipynb
3. 03_CLIP_AST_ontology_eval_3500.ipynb
4. 04_CLIP_AST_ontology_candidate_eval_3500.ipynb
```

Each stage uses the outputs produced by the previous stage.

## Main Evaluation Results

| Method | Accuracy (%) | Macro-F1 |
|---|---:|---:|
| CLIP | 77.03 | 0.7675 |
| CLIP + AST + ontology | 77.80 | 0.7751 |
| CLIP + AST + ontology + candidate rules | **78.97** | **0.7858** |

The ontology improves the CLIP baseline while providing an explicit reasoning path from detected acoustic events to semantic concepts and candidate scenes. The candidate-conditioned stage provides further selective corrections within visually ambiguous candidate groups.

## Frozen Basic-Ontology Configuration

The evaluation uses the ontology configuration selected during development:

```python
MIN_EVENT_CONFIDENCE = 0.08
ONTOLOGY_LAMBDA = 0.40
CLIP_PROTECTION_MARGIN = 0.20
MIN_ONTOLOGY_EVIDENCE = 0.15
MIN_ONTOLOGY_ADVANTAGE = 0.05
STRONG_ONTOLOGY_EVIDENCE = 0.60

NORMALIZATION_MODE = "sum"
CLIP_PROTECTION_MODE = "candidate"
```

Ontology evidence is based on:

```text
event confidence × temporal persistence × semantic strength
```

and is restricted to the CLIP Top-3 candidate scenes.

## Candidate-Conditioned Rules

The final stage considers three frozen candidate groups:

```text
{bus, metro, tram}

{park, pedestrian street, street traffic}

{metro, metro station, tram}
```

Candidate rules are applied only when the basic ontology has **not already changed the original CLIP Top-1 prediction**. Any correction already made by the ontology is preserved.

## AST Extraction

AST uses the same frozen configuration as development:

```text
Model: MIT/ast-finetuned-audioset-10-10-0.4593
Sampling rate: 16 kHz mono
Chunk duration: 3 seconds
Top-K events per chunk: 8
Final partial chunk: zero-padded
Activation: sigmoid
```

The implementation uses:

```python
ASTFeatureExtractor
ASTForAudioClassification
```

from Hugging Face `transformers`, using direct model inference rather than the `pipeline("audio-classification")` API.

AST events are aggregated per recording using:

```text
max_score
mean_score
count
num_chunks
persistence = count / num_chunks
```

## CLIP Baseline

The CLIP evaluation uses:

```text
Model: openai/clip-vit-base-patch32
Frames per video: 8
Sampling: centers of equal temporal intervals
Scene prompts: 10 fixed TAU prompts
Aggregation: mean frame-level class probabilities
```

## Evaluation Protocol

The 3500-recording split is used only for independent evaluation.

Therefore:

- no ontology parameters are tuned on this set;
- no candidate-rule thresholds are changed;
- no grid search is performed;
- PANNs is not evaluated here;
- naive CLIP+audio fusion experiments are not repeated here.

All ontology and candidate-rule settings are carried over from the 6000-recording development stage.

## Notes

- Run the notebooks from a clean kernel for final reproduction.
- Keep the original dataset folders unchanged.
- The outputs from CLIP and AST are reused by the ontology notebook.
- The candidate notebook uses the ontology output and preserves ontology corrections.
