# urbanbirds25-Code
# Code and results: sensitivity of urban bird detection to bounding-box area error

This archive contains the code, saved model outputs and result tables behind the manuscript
"Sensitivity of species-level urban bird detection to systematic and random bounding-box area error".
All numbers, tables and figures in the manuscript can be regenerated from these files without retraining.

The image annotations (tight bounding boxes for 12,500 iNaturalist photographs of 25 species, with
photo URLs, licences and attributions) are archived separately:
Yucedag, M.C., 2026. Urban Birds 25: tight bounding boxes for 25 urban bird species. Zenodo.
https://doi.org/10.5281/zenodo.23220531

The bird count data used for the city analyses are from Alba et al. (2025):
https://doi.org/10.5281/zenodo.15343591

Code comments are partly in Turkish; file and function names follow the description below.

## Environment

Python 3.14, Ultralytics 8.4.171, PyTorch 2.14, NumPy, pandas, SciPy, matplotlib.
Exact package versions: `requirements_lock.txt`. Training used one NVIDIA GeForce RTX 4050 Laptop GPU.

## Folder layout

Scripts expect to be run from the archive root, with the dataset from the Zenodo record above placed in
`dataset/` (`dataset/images/{train,val,test}`, `dataset/labels/{train,val,test}`).

| Path | Content |
|---|---|
| `common.py` | shared helpers (species list, YOLO label I/O, dataset YAML) |
| `species.csv` | the 25 study species and class IDs |
| `dataset/split_manifest.csv`, `split_summary.csv` | train / validation / test assignment of every image |
| `refs/city_shares_25.csv` | species shares in the six cities, derived from Alba et al. (2025) |
| `results_main/` | results of the 21 systematic box levels (both architectures, all seeds) |
| `results_random/` | results of the random-error condition |
| `runs_main/<model>_<level>_s<seed>/` | per model: training arguments (`args.yaml`), training log (`results.csv`) and all test-set predictions (`test_predictions.csv.gz`, confidence ≥ 0.001) |
| `runs_random/` | the same for the three random-error models |
| `tables/` | statistical test outputs (`stats_*`) and city analyses (`city_*`) |
| `kalibrasyon/` | calibration analysis (code, validation-set predictions of every model, results) |
| `rastgele_cozum/` | analyses of the random-error loss (code, results) |
| `annotator_test/` | blind annotation test with three independent annotators (code, selected images, drawn boxes) |

Level names: `m003` = +3% box area, `mn003` = −3%, `m000` = original boxes, `rnd010` = random error.

## Manuscript section → script

| Manuscript | Script | Output |
|---|---|---|
| 2.4 Systematic box manipulation | `04_make_margins.py` | `datasets_margin/<level>/` |
| 2.7 Training and test evaluation (tight, species-agnostic, expanded protocols) | `05_train_eval.py` | `runs_main/`, `results_main/results.csv`, `counts.csv`, `confusion.csv` |
| 2.8, 3.1–3.3 Friedman, Wilcoxon (Holm), Page trend, symmetry, TOST, Kendall's W, Spearman | `analiz/numbers_rebuild.py` | values reported in Sections 3.1–3.3 |
| 2.8, 3.4 Predicted/tight box-area ratio | `analiz/box_ratio_rebuild.py` | `results_main/box_ratio.csv` |
| 2.8, 3.4, 3.5, 3.7 Rescaling predictions by 1/k inside the evaluation tool | `analiz/val_olcekle.py` | `results_main/shrink_back_val.csv` |
| 2.8 Rescaling from saved predictions (check) | `analiz/shrink_back_rebuild.py` | `results_main/shrink_back.csv` |
| 2.8, 3.4 Calibration: predictions on the validation set | `kalibrasyon/val_tahmin.py` | `kalibrasyon/cikti_val/<model>.csv.gz` |
| 2.8, 3.4 Calibration with n tightly annotated validation images, evaluated on the full test set | `kalibrasyon/kalibrasyon_val.py` (uses helpers in `kalibrasyon/kalibrasyon.py`) | `kalibrasyon/cikti_val/kalibrasyon_val_sonuclari.csv` |
| 2.8, 3.5 Predicting box deviations; correlation between models | `rastgele_cozum/sapma_tahmin.py` | `rastgele_cozum/cikti/sapma_tahmin.csv` |
| 2.8, 3.5 Test-time augmentation | `rastgele_cozum/tta.py` | `rastgele_cozum/cikti/tta_sonuclari.csv` |
| 3.5, Table 3 AP of random versus systematic error | values read from `results_main/results.csv` and `results_random/` | Table 3 |
| 3.6 Species sensitivity slopes, reliability, variance decomposition | `analiz/tur_bolumu_sayilar.py`, `tur_egim.py`, `guvenilirlik.py`, `seed_varyans.py`, `tur_tutarlilik.py` | values in Section 3.6 |
| 3.7 YOLOv8n versus YOLOv10n | `analiz/mimari_sayilar.py` | values in Section 3.7 |
| 2.9, 3.8 City-weighted performance, bootstrap, Wald test | `analiz/sehir_wald.py` | values in Section 3.8, `tables/city_*` |
| 2.3, 3.9, Table 4 Blind annotation test | `annotator_test/blind_test/analiz_arkadas.py` (matching rules in `annotator_test/analiz.py`) | `annotator_test/blind_test/results/*_comparison.csv` |
| Fig. 1 | `figures/fig_pipeline_v6.py` | |
| Fig. 2 | `figures/fig_box_levels.py` | |
| Fig. 3 | `analiz/fig_tur_isi.py` | |
| Figs. 4, 5 and 8 | `analiz/fig_sonuclar.py` | |
| Fig. 6 | `kalibrasyon/sekil_kalibrasyon_val.py` | |
| Fig. 7 | `analiz/fig_mimari.py` | |

Retraining all 66 models takes several weeks on a single laptop GPU; every analysis above runs from the
saved predictions and result tables in this archive.

## Annotator test

`annotator_test/blind_test/secilen_50.json` lists the 50 test images (two per species) used in the test.
`paket_yap.py` builds the single-file, offline box-drawing page in which the images were shown without the
reference boxes. The drawn boxes of the three annotators are in `results/boxes_annotator_{A,B,C}.json`
(format: `{"annotator", "created", "n_images", "n_done", "boxes": {image: [[x1, y1, x2, y2], ...]}}`,
coordinates normalised to 0–1). To compare them with the reference boxes:

    python annotator_test/blind_test/analiz_arkadas.py annotator_test/blind_test/results/boxes_annotator_A.json

## Not included

Trained model weights (66 models), because of their size; available from the author on request.

## Licence

Code: MIT licence. Result tables and predictions: CC BY 4.0.

## Citation

If you use this archive, please cite the article (citation to be added on publication) and this record.
