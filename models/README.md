# DartFeed AI test models

Both files are the **same network** (`DartTipHeatmapNet`, 177,745 parameters) exported with the **same recipe**
(`torch.onnx.export`, opset 17, input `input` `[1,3,416,320]`, output `heatmap` `[1,1,104,80]`, single self-contained file).
Only the weights differ. Calibration, board geometry, preprocessing, decode (NMS radius 4, max 3 darts,
confidence threshold 0.7) and scoring are shared and unchanged by the model selector.

| key | label in page | file | bytes | sha256 |
|---|---|---|---:|---|
| `original` (default) | Original v0.4 | `v0.4_best.onnx` | 713,511 | `3f40407bef9ded0e32e245c4b5a3edaf6f417a0e4e906c3a2de35b0023641937` |
| `syn600` | Synthetic 600 seed 22 | `dartfeed_syn600_seed22.onnx` | 713,511 | `a4d91face26ff57f3322ef42428d037e3e3b01efbf7ff3863cae2fe1b373b2f4` |

## Synthetic 600 seed 22

* Training: v0.4 recipe (Adam 1e-3, batch 6, weighted heatmap loss, v0.4 augmentation, early stopping patience 30,
  checkpoint chosen on the real 16-image validation split) on the 74 real v0.4 training photos **plus 600 synthetic
  composites** (condition "C", seed 22). 97 epochs, best epoch 66.
* Source checkpoint: `training/ab_experiment/checkpoints/C_seed22.pt`,
  sha256 `19a375d98123e9450b8315ff5353f099d5d43fde9790da87a71028d46d395f09`.
* Held-out real result (103 unseen real darts): 84.5% recall, 19.7 px median tip error (3024-px scale),
  11 false positives, 0 empty-board phantoms. This was one seed of three; the three seeds of that condition were
  81.6 / 84.5 / 78.6% against 76.7 / 78.6 / 76.7% for the real-only baseline, so treat it as an experimental model.
* ONNX check against the PyTorch checkpoint (457 identical input tensors): max abs logit difference 2.8e-5,
  decoded tips identical on every image.
* Export recipe check: re-exporting the original v0.4 checkpoint with the same script reproduces `v0.4_best.onnx`
  byte-for-byte.

## Versioning / cache-busting

`index.html` keeps a registry of models with a version and the expected SHA-256 of each file. A non-default model is
requested as `…onnx?v=<first 12 hex of sha256>`, fetched with `cache: 'no-store'`, and its bytes are hashed in the
browser and compared with the registry before the network is created, so a stale browser or CDN cache can never
silently serve the wrong weights. When publishing a new model, add a **new file name** and a new registry entry;
never overwrite an existing `.onnx`.
