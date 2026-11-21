# Structure from Motion on an NTU campus tower (AI6121)
 
An incremental structure-from-motion (SfM) pipeline written step by step in Python (OpenCV, NumPy, SciPy), run in Google Colab. It reconstructs a coloured sparse point cloud and the camera poses of a campus tower from our own smartphone photos, refines them with bundle adjustment, and is benchmarked against COLMAP. A second scene (the CCDS signage pillar) tests whether the pipeline generalises.
 
[![Open main project in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Mrpopo911/AI6121-Sfm/blob/main/AI6121_SfM_project.ipynb)

[![Open generalization notebook in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Mrpopo911/AI6121-Sfm/blob/main/AI6121_SfM_generalization_test.ipynb)

 
## Results at a glance
 
| Scene | Photos | Cameras registered | 3D points | Mean reprojection error (after BA) | Agreement with COLMAP |
|---|---|---|---|---|---|
| Tower, Closer | 55 | 55 / 55 | 10,314 | 0.401 px | centres 1.57% of ring radius, rotation 1.07° median |
| Tower, Normal | 66 | 66 / 66 | 12,770 | 0.443 px | centres 1.79%, rotation 1.19° median |
| Pillar (new scene) | 35 | 35 / 35 | 7,776 | 0.435 px | centres 1.3%, rotation 0.5° median |
 
Bundle adjustment lowers the mean reprojection error from 0.517 → 0.401 px (Closer), 0.522 → 0.443 px (Normal) and 0.57 → 0.435 px (pillar). Errors are truncated at 4 px by outlier removal, so read them together with the point counts. COLMAP is used only as a reference, never inside the pipeline.
 
<!-- TODO: add 1-2 images here, e.g. figures/closer_views.png and figures/pillar_views.png -->
 
## Repository contents
 
| File | What it is |
|---|---|
| `AI6121_SfM_project.ipynb` | Main notebook: the full pipeline (Stages 1–7) on the two tower photo sets, plus the feature comparison, parameter tests, SOM side experiment, COLMAP benchmark and renders |
| `AI6121_SfM_generalization.ipynb` | The same pipeline run on the pillar photos (no building mask, 2,500-keypoint cap, closed ring) <!-- TODO: confirm the file name --> |
| `AI6121_SfM_Report.pdf` | Project report |
| `figures/`, `results/` | Rendered point-cloud views and result tables (CSV) <!-- TODO: optional --> |
 
The photos themselves are not stored in this repository <!-- TODO: or say where they are -->.
 
## Pipeline
 
1. **Acquisition and preprocessing:** quality check (blur, exposure, clipping, near-duplicates), undistortion with a shared intrinsic matrix (f = 1126.75 px), CLAHE contrast normalisation.
2. **Features:** GrabCut building mask, RootSIFT keypoints (SIFT and ORB were compared; see the report).
3. **Matching:** ratio test plus mutual check, fundamental-matrix RANSAC verification, a stricter rule for far-apart pairs (the tower's four sides look alike), match graph and tracks.
4. **Initialisation:** essential matrix, our own pose selection and DLT triangulation, best of ten candidate start pairs.
5. **Registration:** next photo = the one seeing most points; PnP with RANSAC; flat-wall, neighbour-rotation and capture-order checks; new-point triangulation and outlier removal.
6. **Bundle adjustment:** `scipy.optimize.least_squares` with a sparse Jacobian and Huber loss; local BA after each photo, then global BA with focal-length refinement.
7. **Post-processing and output:** upright frame, coloured PLY point cloud, camera poses, CSV tables and figures.

## Data

The photos and the saved intermediate results are in a shared Google Drive folder, not in this repo:

**[Download / copy the data folder]((https://drive.google.com/drive/folders/1QnUE3rwkOHMiHfmdDlIAV5TpwK9iHikq?usp=sharing))**

1. Open the link and copy the folder `AI6121_SfM` to your own Drive, so that it sits at `My Drive/AI6121_SfM`
   (right-click the folder, "Make a copy", then move the copy to My Drive and rename it if needed).
2. It contains `input_images.zip` (tower photos), `set2.zip` (pillar photos) and `SfM_backup/` (saved Stage 3-4 results, lets you skip the 10-minute matching stage).
3. Open a notebook in Colab, choose **Run all**, and approve the Drive permission.

## How to run (Google Colab)
 
1. Put the photos in Google Drive in the folder `My Drive/AI6121_SfM/`:
   * `input_images.zip` with `Closer_scenes/` and `Normal_scenes/` inside (for the tower notebook)
   * `set2.zip` with the pillar photos (for the generalisation notebook, in `AI6121_SfM/generalization/`)
   * optionally the saved Stage 3–4 results (`SfM_backup/`), so Stages 3–4 can be skipped
2. Open the notebook in Colab (badge above), choose **Run all**, and approve the Drive permission once. Nothing needs to be uploaded or downloaded by hand.
3. Settings in the first code cell:
| Flag | Meaning |
|---|---|
| `RECOMPUTE` | `False` loads the saved Stage 3–4 results from Drive; `True` recomputes them (about 10 min) and saves them |
| `RUN_SOM` | run the self-organizing-map pair-selection experiment |
| `RUN_COLMAP` | build the COLMAP reference models with `pycolmap` (a few minutes per scene) |
| `RUN_FEATURE_COMPARISON` | rerun the SIFT / RootSIFT / ORB comparison |
 
The generalisation notebook also has `USE_MASK` and `LOOP_FLAGS` (closed ring or open arc per scene).
 
4. Results are written to the same Drive folder: `SfM_stage6/` (models, point clouds, tables), `figures/`, `som_addon/`, and the CSV reports (`ba_comparison.csv`, `parameter_tests.csv`, `feature_comparison.csv`, `colmap_comparison.csv`).
The pipeline runs on a standard Colab CPU runtime; no GPU is needed.
 
## What we implemented and what we called
 
We did not call any function that implements SfM. Library functions are used only as generic building blocks: OpenCV (`SIFT_create`, `BFMatcher`, `findFundamentalMat`, `findEssentialMat`, `solvePnPRansac`, `grabCut`, ...) and SciPy (`least_squares`, sparse matrices, rotations). The full list is in Section 2.5 of the report. The two-view pose selection, triangulation, view ordering, registration checks, track building and the bundle-adjustment residuals and schedule are our own code.
 
## Limitations
 
* Sparse output only (about 2.5–3× fewer points than COLMAP); no dense reconstruction.
* Thresholds were tuned on one tower and one phone camera; only one further object was tested.
* The pillar run uses no mask, so a large share of its points lies on the surroundings.
* The SOM pair-selection experiment is a side result: it does not beat capture-order proximity on our ordered photo sets.
## Use of generative AI and external tools
 
<!-- TODO: make this match the declaration in the report exactly -->
* Generative AI (Claude, Anthropic) was used for [TEAM: confirm] writing and debugging parts of the notebook code and drafting the report. All code was run and checked by the team.
* COLMAP / pycolmap: used only as an external reference (starting focal length and the benchmark), not inside the pipeline.
* Libraries: OpenCV, NumPy, SciPy, pandas, Matplotlib.
## Team
 
<!-- TODO: names (matric numbers are in the report, not needed here) -->
Pratham
Harini
Hari
 
## References
 
See the reference list in the report (SIFT, RootSIFT, ORB, RANSAC, COLMAP, bundle adjustment and others).
 
