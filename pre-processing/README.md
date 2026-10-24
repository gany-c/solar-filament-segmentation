# Pre-processing

Place image and annotation preparation scripts here.

Planned responsibilities:
- Read grayscale images and COCO-style annotation records.
- Convert supported segmentation representations into individual masks.
- Apply image and mask transformations consistently.
- Save processed data under the ignored root `data/` directory.

Document resizing and coordinate transforms so inference can restore predictions
to the original image size. This directory name is intended for scripts; its hyphen
makes it unsuitable as a normal Python import package name.
