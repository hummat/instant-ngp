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
