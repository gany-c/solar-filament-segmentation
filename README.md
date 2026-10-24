# Solar Filament Segmentation

Workspace for the Kaggle Solar Filament Segmentation Challenge 2026.

## Project structure

```text
resources/       Competition references, documentation, and notes
analysis/        Exploratory notebooks and validation analysis
pre-processing/  Image preparation and annotation-to-mask conversion
training/        Training configurations, scripts, and experiment records
inference/       Prediction scripts and submission generation
```

## Workflow

1. Review the competition references in `resources/`.
2. Explore images and annotations in `analysis/`.
3. Implement data preparation in `pre-processing/`.
4. Train and evaluate models in `training/`.
5. Generate masks and the submission CSV in `inference/`.

This repository is a scaffold. It does not yet contain a model or executable pipeline.
Keep raw competition data, generated masks, model weights, and credentials out of Git.
Record dependencies and package versions as the implementation takes shape.

## Data and validation

Keep the organizer's downloaded directory structure intact outside this repository,
or under an ignored `data/` directory. Keep repeated annotations of the same
underlying solar image together when making local training and validation splits.

## Competition

- Overview: https://www.kaggle.com/competitions/filament-segmentation-2026
- Data: https://www.kaggle.com/competitions/filament-segmentation-2026/data
- Rules: https://www.kaggle.com/competitions/filament-segmentation-2026/rules

Consult the current official pages for requirements. Do not upload competition data
to a public repository. No license for project code has been selected yet.
