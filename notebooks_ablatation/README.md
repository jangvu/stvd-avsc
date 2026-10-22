# Ablation Notebooks

This folder contains the ablation studies used to analyze the contribution of the main components of the proposed audio-visual scene classification framework.

The experiments are organized across both the **6000-recording development set** and the **3500-recording independent evaluation set**.

## Notebook Overview

| Notebook | Purpose |
|---|---|
| `Ablation_ontology_6000.ipynb` | Ablates temporal-persistence weighting and the ontology-level CLIP protection gate on the 6000-recording development set. |
| `Ablation_ontology_3500.ipynb` | Repeats the ontology ablations on the independent 3500-recording evaluation set using the frozen configuration. |
| `Ablation_ontology_candidate_6000.ipynb` | Evaluates the contribution of the candidate-conditioned transport, park, and metro-station rule families on the development set. |
| `Ablation_ontology_candidate_3500.ipynb` | Repeats the candidate-rule ablations on the independent evaluation set. |

## Ablation Design

Two groups of ablations are evaluated.

### 1. Basic ontology ablations

The basic ontology is evaluated under three configurations:

```text
Full ontology
Without persistence
Without CLIP protection
```

These experiments test the role of:

- temporal-persistence weighting in acoustic evidence;
- the ontology-level CLIP protection gate.

Persistence is intended to emphasize acoustic events that recur over time and is expected to be more useful for longer or acoustically complex recordings. The CLIP protection gate acts as a conservative safeguard against weak acoustic evidence overriding a visually confident prediction.

### 2. Candidate-rule ablations

The candidate-conditioned stage is evaluated under:

```text
Full candidate
Without transport rules
Without park rule
Without station rules
Ontology only
```

These experiments isolate the contribution of the three semantic candidate-rule families:

```text
Transport:
{bus, metro, tram}

Park:
{park, pedestrian street, street traffic}

Station:
{metro, metro station, tram}
```

## Ablation Results

| Configuration | Development (%) | Evaluation (%) |
|---|---:|---:|
| Full ontology | 79.73 | 77.80 |
| w/o persistence | 79.85 | 77.80 |
| w/o CLIP protection | 79.70 | 77.74 |
| Full candidate | **81.12** | **78.97** |
| w/o transport rules | 80.17 | 78.14 |
| w/o park rule | 81.10 | 78.97 |
| w/o station rules | 80.70 | 78.63 |

The ontology ablations show that removing persistence has little effect under the present dataset conditions, while removing the CLIP protection gate causes a small but consistent reduction in accuracy. The candidate-rule ablations show that the transport rules provide the largest contribution, followed by the station rules, while the park rule has only a minor effect on the current dataset.

## Development and Evaluation Protocol

The ablation notebooks use the same frozen ontology and candidate-rule logic as the main experiments.

The development notebooks operate on:

```text
manifests/tau_balanced_6000_experiment_manifest.csv
```

The evaluation notebooks operate on:

```text
manifests/tau_balanced_3500_test_manifest.csv
```

The evaluation split is used only for independent validation. No threshold tuning or rule redesign is performed on the 3500-recording set.

## Important Reproducibility Note

For ontology ablations, the **Full ontology** configuration should reproduce the corresponding main ontology result before the ablation variants are interpreted.

Expected reference accuracies are:

```text
Development: 79.73%
Evaluation:  77.80%
```

If the full configuration does not reproduce these values, the notebook inputs, execution state, ontology version, or configuration should be checked before using the ablation results.

## Notes

- Run each notebook from a clean kernel when reproducing final results.
- Keep the same CLIP and AST result files used by the corresponding main experiment.
- Ablation notebooks should only disable the tested component; the remaining ontology and candidate logic must remain unchanged.
- No PANNs ablation is performed.
- No additional rule tuning is performed on the evaluation set.
