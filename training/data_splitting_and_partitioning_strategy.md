# Data Splitting & Partitioning Strategy

## Executive Summary
To evaluate our U-Net model accurately on the MAGFiLO solar filament dataset, data partitioning must be structured to prevent **data leakage**. Splitting images randomly without accounting for repeated annotations, temporal proximity, or duplicate observations can inflate validation metrics. The split should measure performance on observations sufficiently independent of the training data.

---

## Key Partitioning Principles

### 1. Split at the Image Level (Not Instance Level)
* **The Reality:** The saved EDA reports 707 training image files, 1,154 image/annotator records in the JSON, and 8,199 filament annotations. The excess image records arise from independent annotations of the same source image; multiple filaments are separate object annotations.
* **The Leakage Risk:** Splitting by annotation records would place different filament masks from the *same* solar image into both training and validation sets.
* **Action Required:** Link filament annotations to their JSON image records using `image_id`, then group those records by the underlying source `file_name`. Different annotators may have different image IDs for the same photograph. Assign each source image, all annotator versions, and all associated instance masks to one fold.

---

### 2. Temporal Leakage Prevention (Time-Block Grouping)
* **The Risk:** Observations close together in time can show the same evolving filament structures, including observations from different GONG stations. Keeping repeated filenames together alone does not prevent this leakage. Shared solar structures, rather than identical atmospheric noise, are the primary concern.
* **Action Required:** Parse capture timestamps from filenames (`YYYYMMDDHHMMSSII`; GONG timestamps are UTC). Group nearby observations across stations into observation sessions or time blocks, and assign each group wholly to one fold. Treat an entire observing day as an initial candidate, not a proven independence interval.
* **Boundary Check:** Adjacent days can still contain the same filaments. Inspect the actual observation cadence and nearest train/validation timestamps before selecting block length. Consider a time gap around validation blocks, excluding nearby training observations, where needed. No final block length or gap has been selected.
* **Verification:** Report the closest train/validation time separation and inspect similar observations across split boundaries. Nonoverlapping date blocks alone do not guarantee independent examples.

---

### 3. Source Filename and Duplicate Grouping
* **The Risk:** Filename aliases or duplicate files can represent the same source observation. Equal timestamps from different stations identify related views, not necessarily identical photographs.
* **Action Required:** Use the source filename as the initial grouping key. Check file hashes for exact duplicates and inspect suspected near-duplicates if available. Merge verified duplicates into the same group even when their filenames differ. Handle related views from different stations through the temporal grouping above.
* **Scope:** Reprocessed versions or filename variants are possibilities to check, not properties established by the current EDA.

---

### 4. Station-Stratified Cross-Validation
* **The Reality:** GONG utilizes six distinct ground stations (`Bh`, `Ch`, `Lh`, `Mh`, `Th`, `Uh`), each possessing slight hardware variations in contrast, H-alpha bandpass, and optical artifacts.
* **Action Required:** After source-image, duplicate and temporal groups are fixed, seek a similar station distribution across folds where feasible. A time block may contain several stations. Do not break groups to force equal station counts; report any imbalance. Station balancing alone does not prevent temporal leakage.

---

## Inversion, Flipping and Training Augmentation

### Orientation of the source images
The Sun has north and south rotation poles. Standard GONG H-alpha processing places solar north at the top and east on the left, so south is down and west is right. Visible N/S labels are not required for this convention. Confirm that the competition JPEGs retain this orientation before interpreting image position scientifically.

### Geometric flips and rotations
For the current task of predicting individual filament masks, the following are candidate training augmentations, not yet validated improvements:

| Transformation | Effect | Target handling |
| --- | --- | --- |
| Horizontal mirror | Reverses left/right and east/west orientation | Mirror the image and each individual mask identically |
| Vertical flip | Reverses top/bottom and north/south orientation | Flip the image and each individual mask identically |
| 180-degree rotation | Equivalent to both flips together | Rotate the image and each individual mask identically |

These operations preserve pixel values and avoid interpolation when implemented as array flips or a quarter-turn rotation by 180 degrees. They create additional views of existing observations, not new independent solar observations. Preserve individual mask identities rather than replacing instance targets with a union mask. If boxes, polygons, spines or coordinate features are used, transform those consistently too.

### Brightness inversion is a different operation
An intensity inversion such as `255 - image` makes dark filaments bright. It does not change spatial orientation and does not match the usual appearance of the supplied H-alpha observations. Leave it out of the initial augmentation experiment unless a separate validation comparison supports its use.

### Directional labels and scientific interpretation
A single mirror reflection reverses geometric handedness. Do not carry Left/Right chirality labels unchanged into a future chirality task without checking their definitions and the required label transformation. A 180-degree rotation preserves geometric handedness. The current competition focuses on segmentation masks, so chirality classification is outside this augmentation plan.

Flips also change hemisphere and directional relationships. This is acceptable as a segmentation experiment but may remove useful positional patterns. Test performance on original, untouched validation images before deciding whether to retain flips.

### Order of operations and evaluation
1. Establish source-image, duplicate and temporal groups, then freeze the train/validation split.
2. Apply random paired image/mask transformations only to training examples, preferably during loading. Any saved augmented variants inherit their source image's fold.
3. Compare a baseline against a flip-augmented run using the same split and evaluation procedure. Check small filaments, merged objects and fragmentation with the organizer's instance-level evaluator.
4. Keep validation images unaugmented for this comparison. Any future test-time augmentation requires inverse-transforming predictions before combining them and evaluating that procedure separately.

---

## Summary Pipeline Checklist
- [ ] Link annotations by `image_id`, then group all annotator records by source `file_name`.
- [ ] Check exact duplicates and merge their groups.
- [ ] Parse UTC capture times and build temporal groups across stations.
- [ ] Choose block length and any time gap after inspecting cadence and cross-boundary similarity.
- [ ] Balance station representation where possible without breaking groups.
- [ ] Audit source-image overlap and nearest train/validation timestamps.
- [ ] Freeze the split before creating or applying augmentations.
- [ ] Transform each training image and its individual masks identically.
- [ ] Rerun corrected EDA morphology cells before using their statistics.
- [ ] Compare augmentations on untouched validation observations before adopting them.

## References
- [GONG H-alpha processing, orientation and timestamp conventions](https://nso.edu/data/nisp-data/h-alpha/)
- [Competition data and annotation format](https://www.kaggle.com/competitions/filament-segmentation-2026/data)
- [Competition task and evaluation](https://www.kaggle.com/competitions/filament-segmentation-2026)
- [MAGFiLO dataset paper, including chirality annotations](https://www.nature.com/articles/s41597-024-03876-y)
