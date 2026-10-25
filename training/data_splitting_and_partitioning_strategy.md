# Model Training and Data Partitioning Strategy

## Executive Summary
To train and evaluate an instance-segmentation model accurately on the MAGFiLO solar filament dataset, data partitioning must be structured to prevent **data leakage**. Splitting images randomly without accounting for repeated annotations, temporal proximity, or duplicate observations can inflate validation metrics. The split should measure performance on observations sufficiently independent of the training data.

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

These operations preserve pixel values and avoid interpolation when implemented as array flips or a 180-degree rotation implemented as two quarter turns. They create additional views of existing observations, not new independent solar observations. Preserve individual mask identities rather than replacing instance targets with a union mask. If boxes, polygons, spines or coordinate features are used, transform those consistently too.

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

## Training Algorithm

The proposed baseline is Mask R-CNN with a pretrained ResNet-50 feature-pyramid backbone. The settings below are provisional starting points to validate, not conclusions established by the EDA.

1. **Prepare the dataset**
   - Index images, timestamps, stations, and annotations.
   - Link annotations through `image_id`, then group records by source filename.
   - Verify image dimensions, mask alignment, and duplicate files.
   - Preserve separate filament masks and use one foreground class: filament.

2. **Create five validation folds**
   - Keep repeated images and all their annotator records in the same fold.
   - Group observations by date across stations.
   - Inspect nearby observations across fold boundaries.
     - Merge related temporal groups or introduce exclusion gaps where needed.
   - Balance stations without breaking groups.
   - Save and freeze the fold assignments.

3. **Handle multiple annotators**
   - Sample unique photographs uniformly during training.
   - Randomly select one complete annotation set for each sampled photograph.
   - Keep alternative annotations separate.
   - Evaluate predictions using the organizer's treatment of multiple annotation sets.

4. **Preprocess images**
   - Read images as grayscale and scale intensities to `[0,1]`.
   - Repeat the grayscale channel for a pretrained three-channel backbone.
   - Apply the backbone's expected normalization once.
   - Start with full images resized to `1024 × 1024`.
     - Preserve coordinate mappings to the original `2048 × 2048` resolution.
     - Check whether small masks disappear during resizing.
   - Initially omit brightness inversion, CLAHE, and fixed-radius disk masking.

5. **Augment training examples**
   - Apply horizontal and vertical flips independently with probability `0.5`.
   - Transform images and all corresponding instance masks identically.
   - Test mild brightness and contrast changes separately.
   - Keep validation images unchanged.
   - Initially avoid aggressive cropping, erasing, and distortion.

6. **Initialize the model**
   - Use Mask R-CNN with a pretrained ResNet-50 feature-pyramid backbone.
   - Configure prediction heads for background and filament.
   - Derive training boxes from individual segmentation masks.
   - Inspect small-object detection and elongated-mask quality before changing proposal or mask-head settings.

7. **Train an initial baseline**
   - Start with one validation fold.
   - Use these provisional settings:
     - Optimizer: AdamW.
     - Backbone learning rate: `1e-5`.
     - Prediction-head learning rate: `1e-4`.
     - Weight decay: `1e-4`.
     - Effective batch size: `4`, using gradient accumulation if necessary.
     - Schedule: short warm-up followed by cosine decay.
     - Maximum duration: `50` epochs.
     - Early stopping: `10` epochs without improved validation PQ.
   - Use standard classification, proposal, box-regression, and instance-mask losses.
   - Save checkpoints and experiment settings.

8. **Validate after each epoch**
   - Predict complete validation images.
   - Restore masks to original image coordinates.
   - Calculate Panoptic Quality using the organizer's evaluator.
   - Inspect:
     - Missed filaments and false detections.
     - Fragmentation and over-merging.
     - Thin branches and boundary quality.
   - Report results by station, object size, and elongation.
   - Select checkpoints by validation PQ and record inference time.

9. **Tune post-processing**
   - Tune detection-confidence and mask-probability thresholds.
   - Check whether suppression removes neighboring elongated objects.
   - Avoid automatic small-component removal or disconnected-mask splitting unless validation supports it.

10. **Compare controlled experiments**
    - Change one setting at a time:
      - Flips versus no flips.
      - Simple scaling versus alternative contrast normalization.
      - `1024 × 1024` versus higher-resolution inputs.
      - Standard mask loss versus an additional Dice loss.
    - Retain changes that improve validation performance consistently.

11. **Complete cross-validation**
    - Train the selected configuration across all five folds.
    - Report mean PQ, fold variation, and inference time.
    - Separate threshold tuning from final performance assessment where practical.

12. **Train and export the final model**
    - Choose settings and training duration from cross-validation.
    - Retrain on all eligible training images.
    - Save model weights, preprocessing settings, dependency versions, and reproduction instructions.
    - Generate test predictions.
      - Restore masks to `2048 × 2048`.
      - Encode individual masks as RLE.
      - Assign unique filament IDs.
      - Verify encoding round trips and submission format.

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

- [Torchvision Mask R-CNN model documentation](https://docs.pytorch.org/vision/main/models/generated/torchvision.models.detection.maskrcnn_resnet50_fpn.html)
