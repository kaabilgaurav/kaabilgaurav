# Automation Stability Report

Completed: 2026-07-18

## What changed

| File | Change | Why |
| --- | --- | --- |
| `.github/workflows/update-profile-art.yml` | Removed the `Prepare Portrait` step. | `source-prepped.png` is tracked in the repository, so daily asset generation does not need to execute the optional background-removal pipeline. This removes the failing `rembg`/ONNX runtime path from GitHub Actions. |
| `scripts/requirements.txt` | Removed `numpy`, `rembg`, and `opencv-python-headless`; added a comment that this file defines automated-workflow dependencies. | None of these packages are imported by any remaining workflow command. Removing `rembg` also removes its transitive `onnxruntime` installation from the workflow. |
| `scripts/requirements-photo-prep.txt` | Added an optional local dependency set for `prep_photo.py`. | Keeps manual portrait preprocessing available without burdening or destabilizing the automated workflow. |
| `scripts/prep_photo.py` | Clarified that it is a manual operation and documented its optional dependency installation command. | Keeps the script maintainable after it is removed from automation. |

## Behavior preserved

- `make_ascii_svg.py` continues to read `source-prepped.png` by default, unchanged.
- The animated ASCII portrait, information card, and contribution heatmap generation remain in the workflow.
- Daily scheduled execution, manual dispatch, push execution, concurrency handling, generated-file loop prevention, and commit/push retry logic remain unchanged.
- README layout, SVG colors, appearance, and animations were not changed.

## Validation performed

- Confirmed `source-prepped.png` is tracked by Git.
- Confirmed no workflow step or shell command references `prep_photo.py`.
- Confirmed the workflow still includes daily schedule, manual dispatch, and `main` push triggers.
- Parsed the workflow YAML and checked the commit-step shell syntax with `bash -n`.
- Compiled every Python script with `py_compile`.
- Ran, without executing `prep_photo.py`:
  - `scripts/render_heatmap_svg.py`
  - `scripts/make_ascii_svg.py`
  - `scripts/make_info_card.py`
- Parsed `data/contributions.json` and all generated SVGs as valid XML.
- Verified `make_ascii_svg.py` still references `source-prepped.png`.
- Ran `git diff --check` successfully.

## Remaining risks

- If `source-photo.png` changes, a maintainer must run the manual preprocessing command before committing the updated `source-prepped.png`:

  ```bash
  pip install -r scripts/requirements-photo-prep.txt
  python scripts/prep_photo.py
  ```

- The optional manual preprocessing path still depends on `rembg` and its ONNX runtime stack. This is intentional and isolated from GitHub Actions.
- Contribution retrieval still depends on GitHub’s public contribution-calendar HTML and may fail if that markup changes or is rate-limited.
- Direct dependency versions for image processing remain unpinned in the optional local requirements file; pin them after validating a known-good local environment.
