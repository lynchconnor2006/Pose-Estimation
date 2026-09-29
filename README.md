# Pose-Estimation
# Automated Biomechanical Analysis of Lacrosse Athletes Using Deep Learning Pose Estimation

A computer vision pipeline that screens lacrosse athletes for injury-risk movement patterns using standard smartphone video, no marker-based motion capture equipment required. The system fine-tunes a YOLOv8x-pose model to track 21 anatomical keypoints, then applies a custom post-processing and inverse kinematics pipeline to compute 15 biomechanical metrics per frame.

## Overview

Marker-based motion capture is the traditional gold standard for biomechanical movement analysis, but its cost and infrastructure requirements put it out of reach for most athletic programs. This project asks whether a single standard camera and a fine-tuned pose estimation model can close that gap for a specific, high-value use case: screening lacrosse athletes for gait asymmetries linked to ACL injury risk.

Beyond building the pipeline, this project also identifies and characterizes a methodological artifact specific to single-camera video: velocity-dependent measures (e.g., stance power, ground contact time) can show large apparent bilateral asymmetry purely as a function of camera angle, even when the underlying movement is symmetric, while purely geometric measures (e.g., joint angle, vertical displacement) do not show this bias. This distinction is central to interpreting the pipeline's output correctly.

## Key Features

- **21-keypoint pose estimation** — YOLOv8x-pose fine-tuned via transfer learning on manually annotated lacrosse practice footage, including detailed bilateral foot anatomy (toe, heel, ankle) for gait-strike analysis
- **Post-processing pipeline**:
  - Confidence-weighted gap interpolation for short sequences of missed detections
  - Savitzky-Golay filtering for trajectory smoothing (preserves rapid direction changes better than a standard Butterworth filter)
  - Dual-reference coordinate normalization (ground plane + body centerline)
- **Inverse kinematics** — joint angle, angular velocity/acceleration, and force/power proxy calculations from smoothed keypoint trajectories
- **15 biomechanical metrics per frame** — including stance width, knee valgus, knee lift height, foot-ground contact state, and bilateral asymmetry indices
- **Movement-phase classification** — automatically distinguishes steady-state running from cutting/direction-change movements, since bilateral asymmetry is expected and healthy during cutting but potentially injury-relevant during steady running
- **Camera-bias-aware bilateral analysis** — separates genuinely comparable geometric measures from velocity-dependent measures that require dual-side capture to interpret reliably

## Repository Structure

```
.
├── data/
│   └── annotations/              # CVAT keypoint annotations (not included -- see Data Availability)
├── training/
│   └── train_yolov8_pose.py      # Model fine-tuning script
├── pipeline/
│   ├── keypoint_smoothing.py     # Post-processing: interpolation, Savitzky-Golay filtering, normalization
│   ├── inverse_kinematics.py     # Joint angle / velocity / power / impulse calculations
│   ├── movement_classification.py # Steady-running vs. cutting classification
│   └── metrics.py                # 15 biomechanical metric calculations + bilateral asymmetry
├── analysis/
│   └── bilateral_analysis.py     # Right/left comparison, geometric vs. velocity-dependent breakdown
├── figures/                      # Output visualizations (skeleton overlays, time-series plots)
├── requirements.txt
└── README.md
```

> **Note:** file and folder names above are placeholders reflecting the pipeline stages described in the accompanying paper/poster. Update this section to match your actual repository structure.

## Installation

```bash
git clone https://github.com/<your-username>/<repo-name>.git
cd <repo-name>
pip install -r requirements.txt
```

Core dependencies:
- `ultralytics` (YOLOv8x-pose)
- `opencv-python`
- `numpy`, `scipy` (filtering, interpolation, inverse kinematics)
- `pandas` (metric tabulation)
- `matplotlib` (figure generation)

## Usage

**Fine-tune the pose estimation model:**
```bash
python training/train_yolov8_pose.py --data data/annotations --epochs 100
```

**Run the full pipeline on a video:**
```bash
python pipeline/keypoint_smoothing.py --input path/to/video.mp4 --output results/
python pipeline/inverse_kinematics.py --input results/keypoints.csv --output results/metrics.csv
```

**Generate bilateral asymmetry analysis:**
```bash
python analysis/bilateral_analysis.py --input results/metrics.csv --output results/bilateral_report.csv
```

> Update these commands to reflect actual CLI arguments and entry points once finalized.

## Methodology Summary

1. **Data collection**: 550 practice-session frames manually annotated in CVAT with 21 keypoints per athlete.
2. **Model training**: YOLOv8x-pose fine-tuned over 100 epochs (cosine LR schedule, rotation/color/flip augmentation), achieving 99.5% mAP@50.
3. **Post-processing**: confidence-weighted interpolation, Savitzky-Golay smoothing, dual-reference normalization.
4. **Metric calculation**: 15 biomechanical measures per frame via inverse kinematics, plus steady-running vs. cutting classification.
5. **Bilateral analysis**: right/left comparison restricted to steady-running frames, with explicit separation of geometric vs. velocity-dependent measures to account for single-side camera bias.

Full methodological detail, results, and the camera-bias analysis are documented in the accompanying paper (`/paper`) and conference poster (`/poster`).

## Results Summary

- **Detection**: >95% joint detection rate overall; 95–97% for pelvis/hip/knee (highest-confidence joints); ~12mm average positional accuracy
- **Movement classification**: 60% steady running / 40% cutting in validation footage
- **Bilateral asymmetry (steady running)**: geometric measures within 1–2% (symmetric); velocity-dependent measures showed 50–58% apparent asymmetry, attributed to single-side camera perspective bias rather than a true movement deficit

## Data Availability

Training data includes identifiable footage of student-athletes and is not publicly released. All results reported in the accompanying paper were generated from a separate validation video for which full disclosure rights are held. Code is provided to allow the pipeline to be applied to your own footage.

## Limitations & Future Work

- Current results are based on single-side (single-camera) video; velocity-dependent measures require dual-side capture and averaging to correct for the camera-angle bias identified in this work. The pipeline architecture already supports this extension.
- Tracks one athlete at a time; multi-athlete tracking for full-practice/full-game screening is a planned extension.
- Computes 2D keypoint-based approximations, not true 3D joint angles; full 3D reconstruction would require calibrated multi-camera triangulation.

## Citation

If you use this pipeline or reference this work, please cite:

> Lynch, C., & Rafique, H. Automated Biomechanical Analysis of Lacrosse Athletes Using Deep Learning Pose Estimation. Syracuse University, Sport Analytics.


## Acknowledgments

Hassan Rafique, Assistant Professor of Sport Analytics at Syracuse University, is a co-author on the accompanying research paper.

## Contact

Connor Lynch — Sport Analytics, Syracuse University
