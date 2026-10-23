# 3D Reconstruction with Structured Light

Laboratory 08 of the course **Computational Imaging** (Skoltech, 2026).

A camera–projector pair is calibrated and used to scan an object with a moving dark–light edge. Every projected edge is a **plane of light**, every camera pixel on the edge is a **ray**; their intersection is a 3D point. Sweeping the edge across the object produces a point cloud, whose accuracy is then compared with a theoretical prediction.

## Method

| Step | What is done | Main functions |
|---|---|---|
| Patterns | 100 vertical dark–light edge patterns for the 1280 × 800 projector panel, with the panel coordinates of both ends of each edge | `generate_slset` |
| Edge detection | Per-pixel normalization `v = (photo − black) / (white − black)`; in every image row the strongest crossing of `v = 0.5` is located with sub-pixel linear interpolation | `detect_edge` |
| Camera rays | Undistortion, then `d = Rᵀ K⁻¹ x̃` and `C = −Rᵀ t` | `pixel_to_ray` |
| Planes of light | Two projector rays through the ends of an edge give the plane normal `n = d₁ × d₂` | `reconstruct` |
| Triangulation | Ray–plane intersection `λ = n·(C_p − C_c) / (n·d)` | `intersect` |
| Camera calibration | Checkerboard plate (10 mm squares) with a three-dot marker as origin; Zhang's method, poses with outlying reprojection error removed | `cv.calibrateCamera` |
| Projector calibration | Projector treated as an inverse camera: a projected 9 × 6 dot grid is located on the calibrated plate (dot photo divided by white photo), dot rays intersected with the plate give 3D positions, paired with the known panel coordinates | `detect_projected_dots`, `cv.calibrateCamera` |
| Analysis | Baseline and object size against tape and caliper; least-squares plane (SVD) on a flat board; edge localization error from a cubic fit; comparison with `δz ≈ z² δx / (f b)` | `fit_plane` |

The functions `generate_slset`, `detect_edge`, `pixel_to_ray`, `intersect` and `fit_plane` use NumPy array operations only.

## Setup

- Camera: Nikon D5600 with AF-S NIKKOR 50 mm f/1.8G (3.9 µm pixels), photos processed at 1620 × 1080 px
- Projector: Optoma ML750e, 1280 × 800
- Camera–projector distance (baseline) 0.18 m, object distance 0.57 m
- 20 calibration poses; object and flat board scanned with 35 edges (patterns 63–97)

## Results

| Quantity | Result | Reference |
|---|---|---|
| Camera focal length / principal point | 3 892 px / (801, 467) px, RMS 0.76 px (17 poses) | ≈ 3 790 px expected (50 mm focused at 0.58 m) |
| Projector focal length / principal point | 2 200 px / (622, 1052) px, RMS 0.52 px (7 poses) | – |
| Baseline from calibration | 183.2 mm (std 1.4 mm over poses) | 180 mm (tape) |
| Object size from point cloud | 97.0 × 123.9 mm | 104 × 116 mm (caliper) |
| Depth noise on a flat board (RMS) | 0.41 mm | 0.34 mm predicted |
| Edge localization error | 0.74 px | 0.5 px assumed in the prediction |

The residual map of the flat board follows the printed checkerboard on the board: the edge is detected with a slightly different bias on dark and light squares, which accounts for most of the edge error.

## Repository structure

```
L08-solution.ipynb            main notebook: acquisition, calibration, reconstruction, analysis
L08-explained.ipynb           step-by-step examples of the methods (synthetic scan, noise test)
prelab/                       handwritten pre-lab derivation (triangulation, depth resolution)
setup-act/                    own setup
  calibration-dataset/          20 poses: dot-pattern photo and white-frame photo
  calibration-data/             calibration result (camera and projector K, distortion, R, t)
  reconstruction-dataset/       object scan: white, black, 35 edge photos
  plane-dataset/                flat-board scan: white, black, 35 edge photos
  structured-light-patterns/    projected images
  panel_upper_points.npy        panel coordinates of the edge ends
  panel_lower_points.npy
setup-ref-1/                  reference setup provided with the course (calibration data and panel points;
                              its scan photos are not included in this repository)
```

## Running

```bash
python -m venv .venv
.venv\Scripts\activate            # Windows  (Linux/macOS: source .venv/bin/activate)
pip install numpy opencv-python matplotlib ipykernel
```

Open `L08-solution.ipynb` with the `.venv` kernel and run all cells (about 5 minutes). The notebook writes the calibration to `setup-act/calibration-data/` and uses it for the reconstruction.
