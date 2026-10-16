# SignBridge: Indian Sign Language Recognition with Text and Speech Output

## Phase 1 closeout — 16 October 2026
An experimental **offline isolated-sign** recognition prototype. Speech is part of the eventual project title; it is not implemented. No continuous translation, training, webcam loop, LLM, interface, authentication or cloud services are included.

**Status: partially complete; real pipeline demonstrated on the target Mac.** The revised 20-class provisional training target is met. Remaining evidence gaps are listed in PHASE1_RESULTS.md; this is not an independent recognition benchmark or permission to describe all Phase 1 criteria as verified.

Measured on the user's Mac: 404 decodable videos, 24 canonical classes, 20 selected training classes with **309** unique non-test recordings. All 20 labels exist in the unchanged 263-class checkpoint map. Seven original training clips produced six correct predictions, with no processing failures. These are different sets: the diagnostic does not evaluate the full target vocabulary.

## Existing Mac installation — quick demonstration
Run from your existing signbridge directory. Keep your current videos, weights and working environment. No reinstall or dataset download is needed.

```bash
source .venv-recognition/bin/activate
python scripts/finalize_phase1.py
python scripts/run_offline.py "data/raw/Adjectives/36. light/MVI_9711.MOV" --out reports/demo_light.json
```

The first command audits saved evidence and does not run inference. The second performs real CPU inference. Light is a previously tested demonstration clip; it is not in the selected 20-class training target. See DEMO_GUIDE.md for interpretation and results.

## Fresh installation only
Python 3.12 is the documented runtime. Do not install the old indexing requirements into the recognition environment.

```bash
python3.12 -m venv .venv-recognition
source .venv-recognition/bin/activate
python -m pip install -r requirements-recognition-mac.txt
python scripts/setup_recognition.py
python -m pytest -q
python scripts/check_environment.py --out reports/current_environment.json
```

Mac ARM64 CPU execution is evidenced by the supplied diagnostic (Python 3.12.10, 8 GiB RAM). Intel Mac support is untested. MPS arithmetic worked, but model acceleration was not verified; the adapter only permits CPU. For Linux Python 3.12, install requirements-recognition.txt, then torch==2.6.0 using the official Linux CPU wheel index https://download.pytorch.org/whl/cpu. That index is not a macOS source.

The recognition pins use NumPy 1.26.4, opencv-contrib-python 4.11.0.86, MediaPipe 0.10.21 and PyTorch 2.6.0. Dependency constraints retain the previously resolved runtime. requirements.txt and requirements-dev.txt are historical indexing environments, not the recognition setup. A virtual environment prevents global package changes.

## Dataset import and indexing
Acquire selected INCLUDE archives from https://zenodo.org/records/4010759 after reviewing the CC BY 4.0 terms and attribution in DATASET_REPORT.md. Browser downloads worked on the target Mac when command-line connections timed out. Compare archive checksums with the official record. Alternatively scripts/fetch_asset.py accepts URL, destination, and --checksum with the official algorithm:value; it resumes where the server supports ranges. Never disable HTTPS verification.

Extract under data/raw, preserving category/class/video paths. Import metadata with relative_video_path and original_label; preserve official_split and known signer/source IDs. Unknown identities stay null. The packaged data/manifests/all_downloaded.csv refers to the user's existing 404 recordings. It is not a full-dataset index.

```bash
python scripts/inspect_dataset.py data/raw --metadata data/manifests/all_downloaded.csv --out reports/target20
```

configs/dataset.yaml sets target_classes: 20 and minimum_unique_development_samples: 5. Explicit aliases retain original labels and IDs. Do not use flat class-folder mode for INCLUDE's nested layout. Final test recordings are excluded from vocabulary selection and diagnostics. Indexing can read test files for integrity checks, not model tuning.

## Model acquisition and inference
scripts/setup_recognition.py downloads only the pinned official ST-GCN checkpoint and required heavy pose asset, verifies hashes, strict state keys, graph adjacency, extractor resources and synthetic output shape. Setup is not accuracy evidence. Weights and videos are excluded from this package. Exact URLs, hashes and input specifications are in configs/model_stgcn.yaml and MODEL_COMPATIBILITY.md.

run_offline.py accepts an existing video path, --out, --device cpu, and optional --cache-dir data/cache/stgcn. Structured output contains labels, uncalibrated model scores, duration, timings, device, provenance, and errors. Missing/corrupt input or unusable landmarks produces an error rather than a forced prediction. Cached landmarks are checked against source and extractor configuration. The 20-class target never replaces the checkpoint's 263 labels.

To reproduce the earlier seven-class diagnostic (original files must still exist):
```bash
python scripts/evaluate_diagnostic.py reports/downloaded_subset_fixed/manifest.json --root data/raw --allow-training-overlap --limit 7 --out reports/repeated_seven_class_diagnostic.json
```
Do not interpret --limit 20 on the full 24-class manifest as evaluation of precisely the selected 20 classes. The tool uses round-robin class selection and includes eligible train data only with explicit opt-in. Final test rows are never selected.

## Modules and limitations
- environment: OS, architecture, RAM, package versions and device arithmetic; does not load the selected checkpoint.
- dataset_index / dataset_validation: full sequential decoding, metadata, file hashes, exact duplicates and known-identity leakage.
- labels / vocabulary / splits: explicit aliases, stable IDs, sample-supported target, source/signer grouping. Unknown signers block new signer-independent splits.
- landmarks / preprocessing: temporally ordered body and both hands, exact 27-joint subset, shoulder normalization, explicit missing-data policy.
- model_loader / stgcn_model / vendor: pinned assets, strict original architecture and 263-output label mapping.
- inference / diagnostics: CPU predictions, separate model-load and per-video times, development-only diagnostics.
- scripts/finalize_phase1.py: reproducible consistency audit of saved reports; no claim of fresh video/weight validation.

See TESTING.md for checks and PHASE1_RESULTS.md for the readiness decision. Reports outside target20 and phase1_closeout.json are earlier evidence; their timestamps/scopes matter. Reports contain the user's local filesystem paths but no recordings.

## Troubleshooting
Activate .venv-recognition if pytest or torch is missing. Do not mix NumPy 2 or two OpenCV distributions into it. A missing dataset root means you must use an actual extracted path. Run inspection before reading its output folder. TLS hostname mismatch requires fixing the connection, proxy or network; installing a CA bundle alone does not fix a wrong-host certificate. Never use an unverified TLS context. Move a checksum-mismatched asset aside before retrying setup. MediaPipe feedback warnings were nonfatal in the recorded runs, but inspect structured status and missing-landmark warnings. High scores can be wrong (Radio → Market). No unknown-sign rejection is implemented.

''' bash
cd ~/Downloads/signbridge
source .venv-recognition/bin/activate
unzip -n ~/Downloads/SignBridge_Demo_Update.zip -d ~/Downloads
python ~/Downloads/SignBridge_Demo_Update/apply_demo_update.py --project .
python scripts/run_demo.py
'''
