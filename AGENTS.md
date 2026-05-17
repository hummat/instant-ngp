# Repository Guidelines

## Project Structure & Module Organization

- Core C++/CUDA code lives in `src/` and public headers in `include/neural-graphics-primitives/`.
- Python bindings are in `src/python_api.cu` (module `pyngp`); higher‑level scripts live in `scripts/`.
- Runtime‑compiled kernels and cache live in `include/neural-graphics-primitives/fused_kernels/` and `rtc/`.
- Network configs are in `configs/`; sample assets are under `data/`. Do not modify `dependencies/` except when upgrading third‑party code.

## Build, Run, and Dev Commands

- Configure: `cmake . -B build -DCMAKE_BUILD_TYPE=RelWithDebInfo`
- Build C++ targets: `cmake --build build --config RelWithDebInfo -j`
- Run GUI binary: `./instant-ngp data/nerf/fox`
- Python environment: `pip install -r requirements.txt`
- Run via Python bindings: `python scripts/run.py --scene fox` (or another scene/config).

## Local Workstation Notes

- Source `/home/matthias/build_cuda.zsh` before configuring or building here. It selects CUDA 12.4 at `/mnt/work/cuda-12.4-linux`, `gcc-12`/`g++-12`, `CUDAHOSTCXX=g++-12`, OptiX 8.1, and `TORCH_CUDA_ARCH_LIST=8.9`.
- For the local RTX 4070, configure explicitly for Ada: `cmake --fresh . -B build -DCMAKE_BUILD_TYPE=RelWithDebInfo -DTCNN_CUDA_ARCHITECTURES=89`. Do not rely on GPU auto-detection; it has fallen back to `75;86` in this checkout.
- A verified clean build sequence is:
  ```bash
  source /home/matthias/build_cuda.zsh
  cmake --fresh . -B build -DCMAKE_BUILD_TYPE=RelWithDebInfo -DTCNN_CUDA_ARCHITECTURES=89
  cmake --build build --target clean
  cmake --build build --config RelWithDebInfo -j4
  ```
- If linking fails with `/usr/bin/ld: cannot find -lcudart`, inspect `/mnt/work/cuda-12.4-linux/targets/x86_64-linux/lib/libcudart.so*`. Broken `unsupported reparse tag` symlinks should be replaced with normal relative links to `libcudart.so.12.4.99`.
- The repo-local `.python-version` can make bare `python`/pyenv shims fail with `pyenv: version 'ngp' is not installed`. For utility scripts that only need OpenCV/NumPy, `/mnt/work/git/mini-mesh/.venv/bin/python` is known-good.
- `colmap` for local reconstruction work is available at `/mnt/work/git/mini-mesh/.local/mini-mesh/bin/colmap`.

## Local COLMAP/NeRF Workflow Notes

- For a folder that already has `transforms.json` and matching image paths, run the GUI directly: `./instant-ngp /path/to/dataset` or `./instant-ngp /path/to/transforms.json`.
- To convert an existing COLMAP binary model for instant-ngp, export text first:
  ```bash
  mkdir -p /path/to/data/colmap_text
  /mnt/work/git/mini-mesh/.local/mini-mesh/bin/colmap model_converter \
    --input_path /path/to/data/sparse \
    --output_path /path/to/data/colmap_text \
    --output_type TXT
  /mnt/work/git/mini-mesh/.venv/bin/python scripts/colmap2nerf.py \
    --images images_orig \
    --text colmap_text \
    --out transforms_colmap2nerf.json \
    --aabb_scale 32 \
    --appear_embed_dim 16
  ```
- Match `--images` to the image names in `colmap_text/images.txt`. Nerfstudio-style folders may have `images/frame_00001.jpg`, while raw COLMAP exports may refer to `0001.jpg` under `images_orig/`.
- For real captures, start with `aabb_scale: 32`; larger natural scenes may need `64` or `128`, while tight object captures can often use less. `--appear_embed_dim 16` writes `n_extra_learnable_dims: 16` and is useful for phone video exposure or white-balance variation.
- Old `digital-minis` history used NVLabs instant-ngp mainly as a fast preview step before Neuralangelo: generate `transforms.json` with `colmap2nerf.py`, then run `/path/to/instant-ngp/instant-ngp .` to verify the capture reconstructs before committing to slower mesh training.

## Coding Style & Naming Conventions

- C++/CUDA: C++14, follow existing style. Preserve tabs/spacing; do not mass‑reformat.
- Types use `PascalCase` (`Testbed`, `NerfNetwork`); functions and free helpers use `snake_case` (`render_frame`, `load_training_data`); members often use `m_foo`.
- CUDA kernels and device functions prefer lower_snake_case and explicit `__global__/__device__`.
- Python: 4‑space indentation, `snake_case` for functions/variables, `CamelCase` for classes. Avoid new runtime dependencies unless essential.

## Testing Guidelines

- There is no dedicated in‑tree test suite beyond vendored dependency tests. If you add tests, keep them small, fast, and opt‑in.
- Prefer targeted sanity checks (e.g. minimal training runs on `data/nerf/fox`) over long‑running benchmarks.

## Commit & Pull Request Guidelines

- Use short, descriptive commits; scope prefixes like `chore:`, `ci:`, `nix:` are common (`git log` for examples).
- Keep PRs focused: describe motivation, key changes, and user‑visible effects. Include CLI examples or screenshots when UI or behavior changes.
- Avoid large mechanical refactors or formatting‑only PRs; keep diffs minimal, especially in CUDA kernels and headers.

## Agent-Specific Instructions

- When editing as an automated agent, touch only what the task requires; avoid changing vendored code in `dependencies/`.
- Prefer local, well‑scoped changes in `src/`, `include/neural-graphics-primitives/`, `scripts/`, and `configs/`.
- Do not introduce new build steps or global config switches without clear justification in comments and PR text.
- For NeRF training logic, keep `src/testbed_nerf.cu` and `include/neural-graphics-primitives/fused_kernels/train_nerf.cuh` consistent; remember that RFL/RFLRelax paths rely on runtime‑compiled kernels when `NGP_BUILD_WITH_RTC` is on.
- For dataset or format changes, update the relevant loader (e.g. `src/nerf_loader.cu`) and, if user‑visible, mirror the behavior in `docs/nerf_dataset_tips.md`.
- To expose new options to Python, first add them to `Testbed` (`testbed.h` / `testbed.cu`), then bind in `src/python_api.cu`, and finally use them from `scripts/run.py` or notebooks.
- When changing GUI or VR behavior, guard new code with the existing `NGP_GUI`, `NGP_VULKAN`, and `NGP_OPTIX` macros so headless and reduced‑feature builds still compile.
