# Real-time 2D → 3D Reconstruction Software

&#x20;&#x20;

> Reconstruct 3D geometry from a single webcam stream in (near) real-time using a monocular depth model (GLPN) + Open3D for RGBD → point cloud → surface reconstruction. Includes tools for sampling, visualization, and simple augmentation.

---

## Features

* Real-time webcam capture → monocular depth estimation using `vinvino02/glpn-nyu` (Hugging Face `transformers`).
* Convert predicted depth + color → Open3D RGBD → per-frame point cloud.
* Accumulate frames, denoise, and perform Poisson surface reconstruction to produce `.obj` meshes.
* Sample meshes to point clouds (`pcloud.npy`) and produce simple rotation-based augmentation (`aug_pclouds.npy`).
* Optional visualization via PyVista / Open3D, and PyTorch-ready dataset examples.

---

## Repository layout

```
.
├─ main.py                  # simple single-frame GLPN demo (improved version)
├─ pcloud.py                # sample mesh -> point cloud (.npy/.ply) + visualization
├─ rotation.py              # augmentation example & DataLoader usage
├─ realtime_pipeline.py     # integrated capture -> accumulate -> reconstruct -> sample pipeline
├─ meshes/                  # output meshes (mesh_0000.obj ...)
├─ data/                    # saved camera frames (optional)
├─ pcloud.npy               # sampled point cloud (created by pipeline)
├─ aug_pclouds.npy          # augmented dataset (created by pipeline)
└─ README.md
```

---

## Quick start

1. Clone the repo and `cd` into it:

```bash
git clone <your-repo-url>
cd Real-time-2D-to-3D-Reconstruction-Software
```

2. (recommended) Create and activate a virtual environment:

```bash
python -m venv .venv
source .venv/bin/activate   # macOS / Linux
.venv\Scripts\activate      # Windows
```

3. Install required packages (CPU example):

```bash
pip install numpy pillow opencv-python transformers matplotlib open3d pyvista
```

> **Important:** install `torch` separately according to your platform and CUDA version (recommended for speed). See the official PyTorch installation guide for the correct command for your system.

4. Run the integrated pipeline:

```bash
python realtime_pipeline.py
```

* Press `q` in any OpenCV window to quit.
* Meshes will be saved to `./meshes/mesh_XXXX.obj`.
* The sampled point cloud will be saved as `pcloud.npy` and augmented dataset as `aug_pclouds.npy`.

5. Optional: visualize / sample a saved mesh with `pcloud.py`:

```bash
python pcloud.py --mesh ./meshes/mesh_0000.obj --samples 4096 --save-ply
```

6. Optional: run augmentation / DataLoader demo:

```bash
python rotation.py
```

---

## Configuration & tuning

Key variables you may want to adjust (in `realtime_pipeline.py`, `main.py`, or the integrated script):

* `DEPTH_METERS_SCALE` — maps normalized GLPN output to meters. Default is \~5.0; calibrate with a reference object for real metric scale.
* `MERGE_EVERY` — number of frames to accumulate before merging/meshing. Larger → more complete reconstruction but slower.
* `VOXEL_SIZE` — voxel downsampling size (meters) before meshing. Larger = faster, lower detail.
* `POISSON_DEPTH` — Poisson reconstruction depth (6–8 recommended). Higher → more detail, more memory/time.
* `OUTLIER_NB`, `OUTLIER_STD` — outlier removal parameters (statistical outlier removal).
* `SAMPLES_FOR_PY` — number of points to sample from mesh when creating `pcloud.npy`.

---

## Performance tips

* **Use GPU**: move PyTorch model to CUDA for orders-of-magnitude faster inference. Install `torch` with CUDA support for best performance.
* **Mixed precision**: enabling `torch.autocast()` (on CUDA) speeds up inference on modern GPUs.
* **Lower Poisson depth** and **increase voxel size** to reduce memory/time during reconstruction.
* **Resize input frames** to a model-friendly size (multiples of 32) to keep inference consistent.
* **Avoid blocking visualizers** every frame: use `cv2.imshow` for fast previews and open Open3D/PyVista only when needed.

---

## Troubleshooting

* **No webcam detected**: try other indices `cv2.VideoCapture(0/1/2)` or use a video file input.
* **Open3D/PyVista GUI issues on headless servers**: run without visualization flags and save outputs to disk.
* \`\`\*\* install problems\*\*: installation can be platform-specific; the code falls back to Open3D sampling if PCU is not available.
* **GLPN model download fails**: ensure internet access on first run so Hugging Face can download & cache the model.

---

## Example commands

Start realtime pipeline (default):

```bash
python realtime_pipeline.py
```

Sample & visualize a mesh:

```bash
python pcloud.py --mesh ./meshes/mesh_0000.obj --samples 2048 --save-ply
```

Run augmentation & DataLoader demo:

```bash
python rotation.py
```

---

## Outputs

* `./meshes/mesh_XXXX.obj` — reconstructed meshes.
* `pcloud.npy` — sampled point cloud (N × 3).
* `aug_pclouds.npy` — augmented dataset (M × N × 3) (original + rotated variants).
* `data/` — optionally saved frames.

---

## Extending the project (ideas)

* ICP-based alignment between frames for improved merging when camera/subject moves.
* Camera + metric calibration to convert GLPN relative depths to true meters.
* Replace GLPN with a lighter/faster depth model for true high-FPS real-time.
* Add a web-based UI/dashboard to change parameters live.
* Create a lazy `torch.utils.data.Dataset` to perform on-the-fly augmentation rather than storing all variants.

---

## Acknowledgements

* GLPN model: `vinvino02/glpn-nyu` (used via Hugging Face `transformers`).
* Open3D for point cloud and mesh processing.
* PyVista for optional visualization.

---

## Contact

Built by **Ashutosh Sharma** — feel free to open issues or PRs on GitHub for enhancements, bug reports, or questions.

---
