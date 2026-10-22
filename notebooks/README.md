# Development Notebooks

This folder contains the notebooks used for the **6000-recording development set** of the project:

**Candidate-Conditioned Ontology-Guided Audio-Visual Scene Classification under Ambiguous Acoustic Evidence**

The notebooks cover the visual baseline, acoustic event extraction, naive audio-visual fusion, ontology-guided reasoning, candidate-conditioned rules, and development-set parameter analysis.

## Dataset

These notebooks use the balanced TAU development subset:

```text
data_subset_6000/
```

with the manifest:

```text
manifests/tau_balanced_6000_experiment_manifest.csv
```

The development split contains **6000 recordings**, with **600 recordings per scene class**.

## Notebook Overview

| Notebook | Purpose |
|---|---|
| `CLIP_dev_6000.ipynb` | Extract the CLIP visual baseline using 8 sampled video frames and the 10 fixed TAU scene prompts. |
| `PANNs_dev_6000.ipynb` | Extract PANNs acoustic events from 3-second audio chunks for comparison with AST. |
| `AST_dev_6000.ipynb` | Extract AST AudioSet events using the direct `ASTFeatureExtractor` + `ASTForAudioClassification` implementation. |
| `01_CLIP_PANNs_mapping_6000.ipynb` | Evaluate naive CLIP + PANNs event-to-scene mapping and fusion. |
| `02_CLIP_AST_mapping_6000.ipynb` | Evaluate naive CLIP + AST event-to-scene mapping and fusion. |
| `03_CLIP_AST_ontology_6000.ipynb` | Apply the basic ontology to CLIP Top-3 candidates using AST confidence, persistence, semantic strength, and CLIP protection. |
| `04_CLIP_AST_ontology_candidate_6000.ipynb` | Apply the candidate-conditioned transport, park, and metro-station rules on top of the basic ontology. |
| `05_CLIP_AST_ontology_grid_search_6000.ipynb` | Development-set parameter study for the ontology configuration. |

## Recommended Execution Order

Run the notebooks in the following order:

```text
1. CLIP_dev_6000.ipynb
2. AST_dev_6000.ipynb
3. PANNs_dev_6000.ipynb

4. 01_CLIP_PANNs_mapping_6000.ipynb
5. 02_CLIP_AST_mapping_6000.ipynb

6. 03_CLIP_AST_ontology_6000.ipynb
7. 04_CLIP_AST_ontology_candidate_6000.ipynb
8. 05_CLIP_AST_ontology_grid_search_6000.ipynb
```

PANNs is included only as a development-stage acoustic comparison. The final evaluation pipeline uses CLIP, AST, the ontology, and the candidate-conditioned rules.

## Main Development Results

| Method | Accuracy (%) | Macro-F1 |
|---|---:|---:|
| CLIP | 78.93 | 0.7877 |
| CLIP + AST naive fusion | 79.42 | 0.7925 |
| CLIP + AST + ontology | 79.73 | 0.7960 |
| CLIP + AST + ontology + candidate rules | **81.12** | **0.8085** |

The ontology improves scene classification while also providing a traceable reasoning path from detected acoustic events to semantic concepts and candidate scenes. Candidate-conditioned rules provide additional selective corrections within visually ambiguous scene groups.

## Frozen Basic-Ontology Configuration

The main ontology configuration used after development is:

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

The ontology uses acoustic evidence of the form:

```text
event confidence × temporal persistence × semantic strength
```

and restricts ontology reasoning to the CLIP Top-3 candidate scenes.

## Candidate-Conditioned Rules

The final candidate stage focuses on three visually ambiguous groups:

```text
{bus, metro, tram}

{park, pedestrian street, street traffic}

{metro, metro station, tram}
```

Candidate rules are applied only when the basic ontology has **not already changed the original CLIP Top-1 prediction**. Existing ontology corrections are therefore preserved.

## AST Extraction

AST uses:

```text
Model: MIT/ast-finetuned-audioset-10-10-0.4593
Sampling rate: 16 kHz mono
Chunk duration: 3 seconds
Top-K events per chunk: 8
Final partial chunk: zero-padded
Activation: sigmoid
```

The implementation uses the direct Hugging Face model interface:

```python
ASTFeatureExtractor
ASTForAudioClassification
```

rather than the `pipeline("audio-classification")` API.

For each recording, AST events are aggregated using:

```text
max_score
mean_score
count
num_chunks
persistence = count / num_chunks
```

## CLIP Baseline

The CLIP baseline uses:

```text
Model: openai/clip-vit-base-patch32
Frames per video: 8
Sampling: centers of equal temporal intervals
Scene prompts: 10 fixed TAU prompts
Aggregation: mean frame-level class probabilities
```

## Notes

- These notebooks belong to the **development stage only**.
- Parameter selection, ontology construction, candidate-rule design, and sensitivity analysis are performed on the 6000-recording split.
- The independent 3500-recording evaluation is kept in the separate evaluation notebook folder.
- Raw dataset folders are not modified by these notebooks.
- Run notebooks from a clean kernel when reproducing final results.
