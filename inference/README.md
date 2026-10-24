# Inference

Place prediction and submission-generation scripts here.

Planned responsibilities:
- Load saved model weights and reproduce training-time image preparation.
- Predict separate filament masks for each test image.
- Restore masks to the original 2048 × 2048 image coordinates.
- Encode masks following the organizer's pycocotools guidance.
- Write the CSV columns `filament_id` and `segmentation_rle`.
- Check unique filament IDs and round-trip mask encoding.

Save generated submissions under the ignored root `outputs/` directory.
